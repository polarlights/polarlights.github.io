---
title: JPA stream query with java8
date: 2018/11/11 16:00:00
categories:
  - JPA
  - Java
tags:
  - Java
  - JPA
---

最近有小伙伴在做批量导出数据的功能，原来是这样写的：
{% asset_img 15417517310823.jpg %}

后来在看其他代码的时候，有个类似功能，突然意识到那个代码在数据量很大的时候，会存在性能问题（最坏会OOM，导致不可用)。后面在交流过程中，小伙伴突然想到上面那个代码也会有类似问题。

在 Rails 的 ActiveRecord 中有`find_in_batches`功能，它会生成`select ... from XX where id > ? limit N offset M`类似的 SQL，实现批量查询。

在 Java 中，首先想到的是使用 Pagable 翻页查找，但是这个不是很优雅，要自己去处理总页数、当前页等数据。

有没有更优雅一些的做法呢？搜索之后，发现了两篇文章：
1. [Streaming MySQL Results Using Java 8 Streams and Spring Data JPA](http://knes1.github.io/blog/2015/2015-10-19-streaming-mysql-results-using-java8-streams-and-spring-data.html)
2. [What’s new in JPA 2.2 – Stream the result of a Query execution](https://vladmihalcea.com/whats-new-in-jpa-2-2-stream-the-result-of-a-query-execution/)

原来 JPA 早就支持了 Java 8 的 stream。使用 Pageable 翻页方法，与 stream 的相比，多了 DB offset 的操作；后者实质上使用的是数据库的游标。

第一篇文章的内容结合自己业务数据，对文章内容作了一次验证：

测试数据库的 Post 表有 56W 条数据。
测试机器信息：
> CPU: i7 2.3GHz
> Memory: 8GBx2 1600MHz DDR3

测试代码:
* case 1:

```java
# PostRepository.java 
@Query("select p from Post p")
Stream<Post> streamAll();
```

* case 2:
```java
# PostRepository.java case 1:
@QueryHints(value = @QueryHint(name = HINT_FETCH_SIZE, value = "" + Integer.MINI_VALUE))
@Query("select p from Post p")
Stream<Post> streamAll();
```

* case 3:
```java
# PostRepository.java case 1:
@QueryHints(value = @QueryHint(name = HINT_FETCH_SIZE, value = "200")
@Query("select p from Post p")
Stream<Post> streamAll();
```

需要注意的是，在调用 Repository 的地方，要显示声明为只读事务，否则会抛出异常。

## 没有堆(Heap)大小限制的情况下：

* case 1:

正常工作

{% asset_img 15417527402090.jpg %}

* case 2:

正常工作

{% asset_img 15417527637481.jpg %}

* case 3:

正常工作

{% asset_img 15417527764295.jpg %}

以上3个时间相差不大。

## 将堆大小做了限制

参数：`-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 -Xmx1080m -Xms512m`

在几个测试案例中，只有case 2可以正常工作，且比前面的速度快了1/3。其它两种 case 全部不能正正常工作，最终 OOM，而且 GC 时间非常多。

* case 1:

{% asset_img 15417529616240.jpg %}

* case 2:

{% asset_img 15417529776586.jpg %}

* case 3:

{% asset_img 15417529893759.jpg %}

## 结论

使用 stream 查询大量数据的时候，务必要添加`@QueryHints(value = @QueryHint(name = HINT_FETCH_SIZE, value = "" + Integer.MIN_VALUE))`注解，它可以在保证速度的同时，内存可以得到很好的控制。
