---
title: Feign Code Analysis and Best Practice 
categories:
  - java
  - spring
tags:
  - java
  - feign
  - feign best practice
  - feign 代码解析
date: 2019-07-20 13:43:03
---

在日常开发过程中，程序员需要调用第三方或者第二方的接口。有的第三方或者第二方提供了供其他人使用的 client libarary, 还有一部分只是提供了接口声明和返回，需要调用者自己去根据接口约定去发起请求。在spring 程序中，Feign[^footnote_feign]是我们经常用到的一个库。本文主要讲解 Feign 的代码解析和在日常工作中的最佳实践。

## 什么是 Feign

Feign |/**feɪn**/| 的意思是假装。它假装了什么呢？顾名思义，它让我们调用 HTTP 请求像调用本地方法一样。

它简化了HTTP 请求的调用，是一种声明式的 Web 服务客户端，灵感来源于[Retrofit](https://github.com/square/retrofit)、[JAXRS-2.0](https://jax-rs-spec.java.net/nonav/2.0/apidocs/index.html)和[WebSocket](http://www.oracle.com/technetwork/articles/java/jsr356-1937161.html)。

## Feign 的概念：

在讲解 Feign 概念之前，我们先不去关心 Feign。假想一下，如果让我们去设计一个这样的库，我们需要哪些因素。

请求应答的基本要素有：
1. 请求的 URL：知道请求发往哪里。
2. 请求的 method： 知道请求使用什么 Http Method。
3. 请求的参数: server 需要客户端传哪些参数。
4. 请求的应答：server 的应答如何处理。应答又分为两种：成功请求应答和失败请求应答。

有了以上的要素，就可以基本确定一个请求的所有信息了。但是，这还不够：

声明了接口，请求还是得需要代码处理才能发出去。代码处理的第一步：它需要知道我们的声明和意图，即能够解析我们的声明。所以一个约定的解析器(Contract)是必需的。

代码懂了我们的意图，还需要行动，将请求发出去。请求的发出者，相对于服务提供者就是一个 Client。由 Client 具体执行，按照我们的意图，把请求发出去。

有了上面的信息，一个完整的请求就够了。但是，这还不够完美？

1. 加入我希望一个请求在 1S 内就能收到反馈，而实际服务提供者，需要 3S 钟才能有应答。这时候该如何声明我的期望，如何处理请求超时？
    
    我们作为客户端将请求发出去，必须要考虑我们的应用希望客户端多久可以返回。相信大家都不相等太久，所以我们要声明我们对请求的时间去期待，即超时时间。如果我们设置了超时时间，就需要有能力定制万一超时了怎么办: 要么重试或者直接报错返回。

1. 如果服务需要客户端带上 token，或者作为服务中的一环，我希望可以追踪这次请求，怎么办？

    有时候服务端的请求需要我们带上 token，或者我们想在请求的头中加上用于追踪请求的 Http Header，要是有个请求拦截器就好了。

到这里，由我们“设计”的声明式请求库就出炉了。回到我们前面的话题，Feign 是如何实现的呢？它有哪些概念？与我们上面的构思有何异同？

* Contract: 契约；用于解析接口声明，生成HTTP请求所需基本参数。
    默认契约在 feign 的 github 主页有介绍。spring-cloud-feign 中使用SpringMvcContract 解析 Spring 的注解，我们可以使用统一的注解，定义请求的基本参数。
* Encoder: 编码器；用于对请求内容作编码。
* Decoder: 解码器：解析请求应答；一般只解析成功返回的应答。
* ErrorDecoder: 错误解码器。一般 400~5xx 的错误由此解析。
* Target: 目标: 即请求所对应的目标类及请求的url.
* Client: 请求具体客户端。可以由用户定制，默认为 HttpUrlConnection; 还可以使用 HttpClient、OkHttp、WebClient 等。
* Retryer: 重试器；在网络 IO 有错误（比如超时、服务不可用等)，触发重试机制，具体重试的规则，在这个里面声明。
* Logger: 日志；请求参数、url、应答、重试等日志的输出具体实现。
* Options: 请求配置，一般用于定义请求的连接、读取应答的超时时间、是否响应 302 跳转。
* RequestInterceptor: 请求拦截器，在请求发出前，对请求作修改。比如添加请求追踪，requestId 等。

我们看一下 Feign 的基本声明和使用代码：
```java
interface GitHub {
  @RequestLine("GET /repos/{owner}/{repo}/contributors")
  List<Contributor> contributors(@Param("owner") String owner, @Param("repo") String repo);

  @RequestLine("POST /repos/{owner}/{repo}/issues")
  void createIssue(Issue issue, @Param("owner") String owner, @Param("repo") String repo);

}

public static class Contributor {
  String login;
  int contributions;
}

public static class Issue {
  String title;
  String body;
  List<String> assignees;
  int milestone;
  List<String> labels;
}

public class MyApp {
  public static void main(String... args) {
    GitHub github = Feign.builder()
                         .decoder(new GsonDecoder())
                         .target(GitHub.class, "https://api.github.com");
  
    // Fetch and print a list of the contributors to this library.
    List<Contributor> contributors = github.contributors("OpenFeign", "feign");
    for (Contributor contributor : contributors) {
      System.out.println(contributor.login + " (" + contributor.contributions + ")");
    }
  }
}
```

## 设计思路

可见 Feign 的设计与我们之前的“设计”非常接近；作为一个框架，它提供了很好的扩展能力，这些扩展能力的设计，也是我们需要着重学习的。

Feign 把很多概念定义为接口或者抽象类，方便我们扩展和定制已有的实现，也可以定义自己的私有实现。比如拍拍贷团队基于 Feign开发的 Raptor库，它拓展了 Feign 的实现，支持 google 的 protobuf。

### 代码解析

以下代码解析基于 Feign 10.2.3，不同版本实现细节可能会有差异。

#### 解析

1. ParseHandlersByName.apply 使用 contract(契约)解析接口类上的注解，获取到相关的 meta 信息，将方法与BuildTemplateByResolvingArgs(用于构建 RequestTemplate)及SynchronousMethodHandler做绑定。
2. ReflectiveFeign 对 1 的结果进行处理，新建一个 java 代理，非默认方法使用代理类处理；默认方法绑定到代理方法上。
3. 返回代理类。

#### 执行

执行的过程本质是java 代理类的调用过程，实际调用的是代理类对应的 MethodHandler。

1. 新建 MethodHandler 实例
1. 将 RequestTemplate转为 Request，发起 HTTP 请求。
    这个过程会对请求编码，如果有拦截器，则会使用拦截器对请求做处理。
2. 根据返回的 Response的状态，判断正常 decode 还是调用 ErrorDecoder。如果超时的话，会调用重试器实例进行处理，由它决定继续重试还是抛出异常。
3. 如果正常返回，调用 Decoder 反序列化为方法声明的返回类型。

更为详细的代码解析，大家可以参考拍拍贷技术博客的一篇文章 [^footnote_paipaidai]

### Feign 涉及到的设计模式

#### 代理模式

Feign 本质上通过 Java 的代理，将声明与实现进行解耦。调用者不用关心具体的实现，只需要假装调用本地方法一样来调用 HTTP 请求。

#### Builder

Feign.Builder 调用者不需要关心 Feign 的实现细节，构建一个代理类。比如前面例子中的 **GitHub**接口，在被初始化之后就成为了实现了 GIthub 接口的代理类。

#### 策略模式

服务声明接口中定义的默认接口方法和普通接口方法，分别使用 DefaultMethodHandler 和 SynchronousMethodHandler处理。前者相当于调用接口的普通默认方法，后者为实际代理方法。

#### 简单工厂方法

InvocationHabndlerFactory 根据传入的 Target 和方法 Handler 映射构建 InvodationHandler 类。

## Feign 有哪些不足

1. 没有提供细粒度地控制接口超时时间的注解

我们不能具体在接口层面约定请求的超时时间，只能将不同超时时间的接口，分别放在不同的文件中。当然，这个也不会是大问题，有赖于 Feign 的良好扩展性，我们可以添加自己的注解和扩展已有的契约(Contract)，使它支持更精细粒度的控制。

## 最佳实践

1. 必须声明接口的超时时间

    默认情况下，Feign 的连接超时时间为 10S，读取超时时间为 60S。不论是当前的微服务条件下，还是作为单独一个应用，默认的超时时间肯定不可以直接用。否则会降低服务的吞吐量，也影响着客户端的使用体验。
    
1. 一个服务内，统一 ErrorDecoder

    从应用的角度来讲，我们给客户端的应答的 body 结构应该是统一的，类似：
    ```json
    {
      "errors": [
        {
          "code": "error.system.remote_service_call",
          "detail": "获取天气信息失败"
        }
      ]
    }
    ```
    对于 RPC的异常情况，错误的 code可以统一，错误的 detail 有利于我们理解到底发生了什么。使用 ErrorDecoder，解析错误的应答，可以帮助我们实现之。

## 参考链接

[^footnote_cloudfeign]: https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-feign.html
[^footnote_feign]: https://github.com/OpenFeign/feign
[^footnote_paipaidai]: http://techblog.ppdai.com/2018/05/14/20180514/