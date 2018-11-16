---
title: How Minitest Works Part 1
date: 2016-02-20 19:02:42
categories:
  - Ruby
  - Test
tags:
  - Test
  - Minitest
---

> 注：本文以 minitest 的最新master 分支 [baf6010](https://github.com/seattlerb/minitest/tree/baf6010053279f75f561f6a599d8837151327588) ，版本为`5.8.4`为基础。
所有代码可以在[minitest_source](https://github.com/minitest_source) 找到.

# 一个简单的测试

```ruby
# 代码 1.1
# dog.rb
class Dog
  def spark
    'Spark!'
  end
end

# dog_test.rb
require 'minitest/autorun'
require_relative './dog'

class DogTest < Minitest::Test
  def setup
    @dog = Dog.new
  end

  def test_dog_should_spark
    assert_respond_to @dog, :spark
    assert_equal 'Spark!', @dog.spark
  end

  def ordiary_method
    assert true
  end
end
```

如何执行测试呢？

```bash
$ ruby dog_test.rb
Run options: --seed 24057

# Running:
.

Finished in 0.001039s, 962.5723 runs/s, 1925.1446 assertions/s.

1 runs, 2 assertions, 0 failures, 0 errors, 0 skips

```

如果我们把`require 'minitest/autorun'` 这一行注释掉，然后再执行`ruby dog_test.rb`，程序是否还
正常执行呢？让我们执行下：

<!-- more -->

```bash
$ ruby dog_test.rb
```
 这一次什么都没有输出。Why? 这一行有什么魔法呢？为什么有了它之后，可以执行测试代码，
 还可以输出测试结果？

 那我们看一下`Minitest` 的代码，就可以了然了。

如果列位有兴趣，可以继续往下看。

# Minitest  的工作原理

## Minitest 主要代码的结构

    minitest
    │   ├── assertions.rb   # 定义 assert_*方法
    │   ├── autorun.rb      # 自动执行测试
    │   ├── benchmark.rb    # Benchmark 相关方法
    │   ├── expectations.rb # 使用`must` 代替`assert`
    │   ├── hell.rb         # 并行执行测试
    │   ├── mock.rb         # Mock 的 expect 相关实现
    │   ├── parallel.rb     # 多线程执行测试
    │   ├── pride.rb        # Report 的一种，以颜色展示结果
    │   ├── pride_plugin.rb # Pride plugin 的具体实现
    │   ├── spec.rb         # spec 实现，本质上是一种语法糖
    │   ├── test.rb         # Minitest 具体执行部分
    │   └── unit.rb         # test/unit 的 Minitest 实现
    └── minitest.rb         # Minitest 的抽象层实现

Minitest 的代码行数统计数据:

    -------------------------------------------------------------------------------
    Language                     files          blank        comment           code
    -------------------------------------------------------------------------------
    Ruby                            13            646           1061           1593
    -------------------------------------------------------------------------------
    SUM:                            13            646           1061           1593
    -------------------------------------------------------------------------------

> 以上是使用[cloc](http://cloc.sourceforge.net/) 统计得出。

Minitest 用了不到1600行的代码，集扩展性强、兼容性好、可读性强于一身。

从上面的代码，我们可以得到以下结论：

* 测试文件需要加载`minitest/autorun` 文件
* 测试类要继承自`Minitest::Test` 或者其子类
* 测试方法需要以`test_` 开头。(可以看到上面只有2个 assert 执行了，里面其实有3个 assert 语句)
* 测试结束会输出执行的结果（是否通过、失败、跳过以及执行时间和速度）
* Minitest 有钩子的存在(比如setup)

那么我们会有以下疑问：

* `minitest/autorun` 到底做了什么？
* 继承`Minitest::Test`的目的何在，它内部有什么特殊方法？
* 为什么以`test_` 开头的方法执行了，而普通的方法没有执行？里面肯定有一个"惊天的阴谋"
* Minitest 的结果是何时，如何打出来的？
* Minitest有哪些钩子，调用顺序几何？

以上问题，我们在下一节讨论。

参考文献:

* [Minitest github repository](https://github.com/seattlerb/minitest)
* [How minitest works](http://mednoter.com/minitest-part-I-autorun.html)
* [The Minitest Cookbook](http://chriskottom.com/minitestcookbook)
