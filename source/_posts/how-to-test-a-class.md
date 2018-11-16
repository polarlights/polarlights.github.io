---
title: how_to_test_class_and_module
date: 2016-03-23 22:32:49
categories:
  - ruby
  - minitest
tags:
  - ruby
  - minitest
---

前面我们简要介绍了Minitest的运行原理。知其然而知其所以然，如果了解了
别人好的代码是如何设计的，对于自己编码技术的提升会有促进作用。

好了，我们接下来继续了解如何使用Minitest测试我们的类、模块、model及钩子、控制器和试图。
后面还会涉及mock和使用种子数据、定制minitest等内容。

本文主要讲如何测试我们写的类。

Ruby是一种面向对象语言非常高的语言，因为即使像数字、纯字符串等都是对象，都有属于它的方法。
现实是复杂的，为了方便理解和处理我们遇到的事物、问题、概念，我们会把它抽象为对象，再高级一些
就是类。对象是具体的某个事物，类，泛指一类事物。

假定我们有下面的一个类：

```ruby
class MyString < String
  def palindrome?
    self.reverse.eql? self
  end
end
```
如何测试它呢？

```ruby
require 'minitest/autorun'

class MyStringTest < Minitest::Test
  def test_should_be_palindrome
    ms = MyString.new('')
    assert ms.palindrome?
    ms = MyString.new('mom')
    assert ms.palindrome?
  end

  def test_should_not_be_palindrome
    ms = MyString.new('name')
    refute ms.palindrome?
  end
end
```

需要requre Minitest的autorun文件，测试类一般带有`Test`字样，而且必须继承自Minitest::Test
类。具体原因我们在前面分析Minitest的原理时，有谈到。里面的测试方法需要以`test_`开头，否则
不会执行该方法。


