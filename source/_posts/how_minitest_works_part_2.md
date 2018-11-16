---
title: How Minitest Works Part 2
date: 2016-02-21 10:40:42
categories:
  - Ruby
  - Test
tags:
  - Test
  - Minitest
---

> 注：本文以 minitest 的最新master 分支 [baf6010](https://github.com/seattlerb/minitest/tree/baf6010053279f75f561f6a599d8837151327588) ，版本为`5.8.4`为基础。
所有代码可以在[minitest_source](https://github.com/minitest_source) 找到.

在上一节我们留下了以下几个问题，本节我们透过对Minitest源码的分析来一探究竟:

* `minitest/autorun` 到底做了什么？
* 继承`Minitest::Test`的目的何在，它内部有什么特殊方法？
* 为什么以`test_` 开头的方法执行了，而普通的方法没有执行？里面肯定有一个"惊天的阴谋"
* Minitest 的结果是何时，如何打出来的？
* Minitest有哪些钩子，调用顺序几何？

<!-- more -->

我们首先看`minitest/autorun`文件，里面只有简单几行：

```ruby
begin
  require "rubygems"
  gem "minitest"
rescue Gem::LoadError
  # do nothing
end

require "minitest"
require "minitest/spec"
require "minitest/mock"

Minitest.autorun
```
## Minitest.autorun

它做了什么事情呢？加载 minitest 相关文件，最后调用`Minitest.autorun`方法，即Minitest的module方法`autorun`。
让我们继续顺藤摸瓜。它在哪里定义的呢？在 [minitest.rb#L45](https://github.com/seattlerb/minitest/blob/baf6010053279f75f561f6a599d8837151327588/lib/minitest.rb#L45):

```ruby
def self.autorun
  at_exit {
    next if $! and not ($!.kind_of? SystemExit and $!.success?)

    exit_code = nil

    at_exit {
      @@after_run.reverse_each(&:call)
      exit exit_code || false
    }

    exit_code = Minitest.run ARGV
  } unless @@installed_at_exit
  @@installed_at_exit = true
end
```

它的主要作用是在程序进程结束前注入Minitest，并执行。

咦，`at_exit`是什么鬼？相信很多人很少见到它甚至是在阅读`Minitest`源码时第一次见到它。查询Ruby的文档，有下面这样的描述：

> Converts block to a Proc object (and therefore binds it at the point of call) and registers it for execution when the program exits. If multiple handlers are registered, they are executed in reverse order of registration.

它将代码块转为`Proc`对象，在程序退出时call这个Proc对象；如果注册了多个`at_exit`代码块，它会逆序执行。

我们写一个测试代码:

```ruby
# at_exit.rb
puts "Into program."

at_exit do
  puts "I'm executed at the end. start..."
  at_exit { puts "I'm executed at last." }
  puts "I'm executed at the end. end..."
end

at_exit { puts "I'm executed after the 'Exit program'." }

puts "Exit program."
```

执行结果：

```bash
$ ruby at_exit.rb
Into program.
Exit program.
I'm executed after the 'Exit program'.
I'm executed at the end. start...
I'm executed at the end. end...
I'm executed at last.
```

通过上面测试代码的执行结果，我们知道，`Minitest.autorun`会先后调用`Minitest.run`和Module变量`@@after_run`里的Proc对象。

它们又分别做了什么呢？

## Minitest.run

`@@after_run`保存的是在所有test执行结束后执行的代码块, 我们可以调用通知程序将测试完成通知给其他程序 或者发送邮件等等。

`Minitest.run`加载Minitest的插件；初始化reporter；执行测试，输出结果；最后返回test的执行结果给上面的`exit_code`

### Minitest.load_plugins

Minitest的插件都是以`_plugin.rb`结尾，放在`minitest`目录下。比如在Minitest中就有`pride_plugin.rb`，它就是Minitest默认的 插件。每个Minitest的插件都可以有(不是必须有)一个以该插件名命名的初始化方法`plugin_[插件名]_init`。 比如`pride_plugin.rb`的插件初始化方法为`plugin_pride_init`。Minitest的参数是用`optparse`解析的，它的插件也有一个支持`optparse`的方法: `plugin_pride_options`来做一些扩展。

### Reporter

这期间还会初始化CompositeReporter、SummaryReporter和ProgressReporter 3 个reporter，并赋值给Minitest的reporter属性，它用来展示测试的结果；它只在`init_plugins`中可用，在初始化完plugin后就被置为空了。所以如果想要在测试结束后调用reporter相关的操作，可以自己编写plugin（后续文章我们会涉及）。

上面说了3中Reporter。那么这三者有什么区别和联系呢？

Reporter的继承结构是这样的:

    AbstractReporter
    |__Reporter
    |   |__ProgressReporter
    |   |__StatisticsReporter
    |      |__SummaryReporter
    |__CompositeReporter

所有的Reporter都是`AbstractReporter`子类，`AbstractReporter`定义了作为一个reporter应该有的方法，它们是`start`(在启动后开始记录测试结果),`record`(输出测试的结果；如果测试没有通过，会记录单个测试的结果),`report`(输出测试的概况),`passed?`(测试是否通过)。

`Reporter`默认将标准输出作为默认的输出。

`ProgressReporter`是一个很简单的reporter，他将测试用**点**打出.

`StatisticsReporter`收集单个测试的统计信息，并没有任何IO相关的操作。如果想定制输出类型（比如CI，HTML等等），可以通过修改这个类的一些方法来做到。该类因为是统计测试结果的，所以它里面包含了测试的数量、assert的数量、开始时间、总时间、失败的测试、报错的测试和跳过的测试数量。

`SummaryReporter`是`StatisticsReporter`的子类，分别在测试开始时和测试结束的时候打印参数信息和标题、概况、失败的细节信息,类似：

```bash
# At the beginning
Run options: --seed 13908

# Running:

# At the end
Finished in 0.003004s, 665.8359 runs/s, 665.8359 assertions/s.

2 runs, 2 assertions, 0 failures, 0 errors, 1 skips

You have skipped tests. Run with --verbose for details.
```
最后一句只有test中有`skip`的结果才会输出。


`CompositeReporter`可以调度多个repoter,将调用`passed?`、`start`、`record`、`report`的方法代理给所有的reporter。可以认为它是所有reporter的顶级代理类。

## Minitest.__run

在初始化完reporter和plugin后，开始跑测试。具体执行每个继承了`Mintest::Test`/`Minitest::Spec`/`Minitest::Benchmark`的子类。比如之前距离代码中的'DogTest'。

但是我们发现继承了`Minitest::Test`的子类只有以`test_`开头的方法执行了，魔法在哪里？我们继续往下看。

## Runnable

`Runnable`是什么鬼？它表示任何`runnable`的父类，任何它的子类都会自动注册到`Runnable.runnables`。为甚么它会**自动**注册呢？因为它有一个Ruby的钩子：`inherited`,它会在任何继承了该类的时候调用，让我们来看看它的代码：

```ruby
def self.inherited klass
  self.runnables << klass
  super
end
```

Runnable的run方法

### runnable.run

它可以按照用户输入的`--name`参数只跑符合对应正则的方法。里面定义了一个`runnable_methods`，它里面就会只保留满足子类定义的正则（不是用户输入的）的方法来一个个执行。比如`Minitest::Test`是`Runnable`的子类，它要求可执行方法是以`test_`开头；同样的`Minitest::Benchmark`要求方法以`bench_`开头.

保留的测试方法，会逐个执行，调用`Minitest.run_one_method klass, method_name, reporter`。
它将调用Runnable的`initialize`方法，将`method_name`作为参数，作为要调用的方法名。

### runnable.new.run

以`Minitest::Test`为例，执行前它会先后调用`before_setup`、`setup`、`after_setup`几个钩子方法，然后调用上面作为参数传入的`method_name`。以实例代码为例，就是调用了`DogTest.new. test_dog_should_spark`，它里面就执行了具体的assert_方法，同样asert的数量就是在此时增加的。方法执行结束后，会调用`before_teardown`、`teardown`和`after_teardown`这几个钩子，你可以做一些你想在测试结束后想做的事情。

这里衍生出个问题：它是如何判断我是Skip还是报错的？

如果正常执行，它会根据`test_`方法实际的`assert_`方法的数量增加asserts属性的值。那么如果报错了呢？比如抛了一个异常，如何保证程序不终止，而是继续执行其他的方法呢？

答案是`rescue`，是的，对代码做保护,详见`lib/minitest/test.rb:L204`:

```ruby
def capture_exceptions # :nodoc:
  yield
rescue *PASSTHROUGH_EXCEPTIONS
  raise
rescue Assertion => e
  self.failures << e
rescue Exception => e
  self.failures << UnexpectedError.new(e)
end
```

Asesrtion的继承关系如下图:

    Exception
    |__Assertion
       |__Skip
       |__UnexpectedError

如果某个方法是被`skip`了，那么它会抛出Skip异常，由Assertion捕获；如果方法执行代码错误，则被Exception捕获，并用UnexceptedError包一下。

## 总结

通过以上的分析，我们现在可以把整个的调用层级重新整理下：

    Minitest.autorun
      Minitest.run(args)
        Minitest.__run(reporter, options)
          Runnable.runnables.each
            runnable.run(reporter, options)
              self.runnable_methods.each
                self.run_one_method(self, runnable_method, reporter)
                  Minitest.run_one_method(klass, runnable_method)
                    klass.new(runnable_method).run

用我们案例代码，表示为:

    Minitest.autorun
      Minitest.run(args)
        Minitest.__run(reporter, options)
          Runnable.runnables.each
            DogTest.run(reporter, options)
              [test_dog_should_spark].each
                DogTest.run_one_method(DogTest, test_dog_should_spark, reporter)
                  Minitest.run_one_method(DogTest, test_dog_should_spark)
                    DogTest.new(test_dog_should_spark).run

参考文献:

* [Minitest github repository](https://github.com/seattlerb/minitest)
* [How minitest works](http://mednoter.com/minitest-part-I-autorun.html)
* [The Minitest Cookbook](http://chriskottom.com/minitestcookbook)
