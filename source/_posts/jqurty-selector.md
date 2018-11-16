---
title:        jquery 选择器总结
date:         2015-02-14 20:00:00
categories:
  - Tech
  - Javascript
tags:
  - jquery
  - selector
---

# Jquery选择器简介

　　(1) Jquery中的选择器完全继承了CSS的风格，利用Jquery选择器，可以非常便捷和快速的找出特定的Dom元素，然后为他们添加相应的行为，而无需担心浏览器是否支持这一选择器，学会使用选择器是学习Jqeury的基础，Jquery的行为规则都必须在获取到元素后才能生效。

# jquery选择器的优势

　　(1) 简洁的写法，$()函数

　　(2)支持CSS1到CSS3选择器

　　(3)完善的处理机制

# 下面我们主要来说一下Jquery中所有的选择器

<!-- more -->

*  基本选择器：通过元素id,class和标签名等来查找Dom元素


　　1). $("#id") 根据给定的ID匹配一个元素，返回单个元素  $("#name")选取Id为name的元素

　　2). $(".class") 根据给定的类名匹配元素 返回集合元素  $(".class")选取所有class为class的元素

　　3). $("element") 根据给定的元素名匹配元素，返回集合元素  $("input")选取所有的input元素

　　4). $("\*") 匹配所有的元素，返回集合元素，$("*")选取所有的元素

　　5). $("selector1,selector2,...,selectorN") 将每一个选择器匹配到的元素合并后返回集合元素， $("div,span,p.myClass")选取所有&lt;div>,&lt;span>和拥有class为myClass的&lt;p>标签的一组元素

* 层次选择器：如果想通过Dom元素之间的层次关系来获取特定元素，例如后代元素，子元素，相邻元素和同辈元素

　　1). $("ancestor descendant") 获得ancestor元素里面的所有descendant(后代)元素，$("div span")选取&lt;div>里的所有的&lt;span>元素

　　2). $("parent>child") 选取parent元素下的child(子)元素，返回集合元素 $("div span")选取&lt;div>元素下元素名为&lt;span>的子元素

 >  注解：和$("ancestor descendant")有区别，$("ancestor descendant")选择的是后代元素

　　3). $("prev+next") 选取紧接在prev元素后的next元素，返回集合元素，$(".one+div")选取class为one的下一个&lt;div>同辈元素

　　4). $("prev~siblings") 选取prev元素之后的所有siblings元素，$("#two~div")选取Id为two的元素后面的所有&lt;div>同辈元素

 >  注解:可以使用next()方法来替代$('prev+next')选择器  $(".one").next("div");可以使用nextAll()方法来替代$("prev~siblings")选择器 $("#two").nextAll(div)

　　(3)过滤选择器：主要通过特定的过滤规则来筛选出所需要的Dom元素，按照不同的过滤规则，过滤选择器可以分为基本过滤，内容过滤，可见性过滤，属性过滤，子元素过滤和表单对象属性过滤选择器
　　

####  1). 基本过滤选择器

1). $(":first") 选取第一个元素，$("div:first")选取所有&lt;div>元素中第一个&lt;div>元素。

2). $(":last")选取最后一个元素，$("div:last")选取所有&lt;div>元素中最后一个&lt;div>元素。

3). $(":not(selector)")去除所有与给定选择器匹配的元素，$("input:not(.myClass)")选取class不是myClass的&lt;input>元素。

4). $(":even")选取索引是偶数的所有元素，索引从0开始，$("input:even")选取索引是偶数的&lt;input>元素
	
5). $(":odd")选取索引是奇数的所有元素，索引从0开始，$("input:odd")选取索引是奇数的&lt;input>元素
	
6). $(":eq(index)")选取索引等于index的元素，(index从0开始)，$("input:eq(1)")选取索引等于1的&lt;input>元素
	
7). $(":gt(index)")选取索引大于index的元素，(index从0开始)，$("input:gt(1)")选取索引大于1的&lt;input>元素(大于1，而不包括1)
	
8). $(":lt(index)")选取索引小于index的元素，(index从0开始)，$("input:lt(1)")选取索引小于1的&lt;input>元素(小于1，而不包括1)
	
9). $(":header")选取所有的标题元素，例如：h1,h2,h3等等，$(":header")选取网页中的所有的&lt;h1>,&lt;h2>,&lt;h3>...
	
10). $(":animated")选取当前正在执行动画的所有元素，$("div:animated")选取正在执行动画的&lt;div>元素。
	
11). $(":focus")选取当前获取焦点的元素，$(":focus")选取当前获取焦点的元素
	
#### 2). 内容过滤选择器：主要体现在它所包含的子元素或者文本内容上面
	
1). $(":contains(text)")选取含有文本为"text"的元素，$("div:contains('我')")选取含有文本"我"的&lt;div>元素

2). $(":empty")选取不包含子元素或者文本的空元素，$("div:empty")选取不包含子元素(包括文本元素)的&lt;div>空元素

3). $(":has(selector)")选取含有选择器所匹配的元素的元素，$("div:has(p)")选取含有&lt;p>元素的&lt;div>元素

4). $(":parent")选取含有子元素或者文本的元素,$("div:parent")选取拥有子元素(包括文本元素)的&lt;div>元素
	
#### 3). 可见性过滤选择器：根据元素的可见和不可见状态来选择相应的元素
	
1). $(":hidden")选取所有不可见的元素，$(":hidden")选取所有不可见的元素，包括&lt;input type="hidden" />，&lt;div style="display:none;">和&lt;div style="visibility:hidden;">等元素。如果只想选取&lt;input>元素，可以使用$("input:hidden")
	
2). $(":visible")选取所有可见的元素，$("div:visible")选取所有可见的&lt;div>元素
	
### 4)属性过滤选择器：通过元素的属性来获取相应的元素

1).$("[attribute]")选取拥有此属性的元素，$("div[id]")选取拥有属性Id的&lt;idv>元素。

2).$("[attribute=value]")选取属性的值为value的元素，$("div[title=test]")选取属性title为"test"的&lt;div>元素

3).$("[attribute!=value]")选取属性的值不等于value的元素，$("div[title!=test]")选取属性title不等于"test"的&lt;div>元素(注意：没有属性title的&lt;div>元素也会被选取)

4).$("[attribute^=value]")选取属性的直以value开始的元素，$("div[title^=test]")选取属性title以"test"开始的&lt;div>元素

5).$("[attribute$=value]")选取属性的值以value结束的元素，$("div[title$=test]")选取属性title以"test"结束的&lt;div>元素

6).$("[attribute*=value]")选取属性的值含有value的元素，$("div[title*=test]")选取属性title含有"test"的&lt;div>元素

7).$("[attribute|=value]")选取属性等于给定字符串或以给字符串为前缀(该字符串后跟一个连字符"-")的元素,$("div[title|="en"]")选取属性title等于en或者以en为前缀(给字符串后跟一个"-")的元素

8).$("[attribute~=value]")选取属性用空格分隔的值中包含一个给定值得元素，$("div[title~='uk']")选取属性title用空格分隔的值中包含字符uk的元素

9).$("[attribute1][attribute2][attribute3]")用属性选择器合并成一个复合属性选择器，满足多个条件，每选择一次，缩小一次范围

$("div[id][title$='test']")选取拥有属性id，并且属性title以"test"结束的&lt;div>元素

### 5)子元素过滤选择器

> 注解：子元素过滤选择器的过滤规则相对于其他的选择器稍微有些复杂，只要将元素的父元素和子元素区分清楚，使用起来还是相当简单的

#### 1). $(":nth-child(idenx/even/odd/equation)")选取每个父元素下的第index个子元素或者奇偶元素(idnex从1算起)

:eq(index)只匹配一个元素，而:nth-child将为每一个父元素匹配子元素，并且:nth-child(index)的index是从1开始的，

而:eq(index)是从0开始的

#### 2). $(":first-child")选取每个父元素的第一个子元素，:first只返回单个元素，而:first-child选择符将为每个父元素匹配第一个

子元素，例如:$("ulli:first-child");选取每个&lt;ul>中的一个&lt;li>元素

#### 3). $(":last-child")选取每个父元素的最后一个子元素,和上面一样，:last只返回当个元素，而:last-child选择符将为每个符永

元素匹配最后一个子元素，例如：$("ulli:first-child");选取每个&lt;ul>中的最后一个&lt;li>元素。

#### 4). $(":only-child")如果某个元素是它父元素中唯一的子元素，那么将会被匹配，如果父元素中含有其他元素，这不会被匹配

$("ulli:only-child")在&lt;ul>中选取是唯一子元素的&lt;li>元素

#### 5). :nth-child()选择器是很常用的子元素过滤器，详细功能如下：

1). $(:nth-child(even))能选取每个父元素下的索引值是偶数的元素

2). $(:nth-child(odd))能选取到每个父元素下的索引值是奇数的元素

3). $(:nth-child(2))能选取到每个父元素下索引值等于2的元素

4). $(:nth-child(3n))能选取到每个父元素下索引值是3的倍数的元素(n从1开始)

5). $(:nth-child(3n+1))能选取每个父元素下索引值是(3n+1)的元素(n从1开始)

### 6). 表单对象过滤选择器：对所选择的表单进行过滤

1). $(:enabled)选取所有可用元素，$("#form:enabled");选取id为"form"的表单内的所有可用元素

2). $(:disabled)选取所有不可用的元素，$("#form:enabled");选取id为"form"的表单内的所有不可用元素

3). $(:checked)选取所有被选中的元素(单选框，复选框)，$("input:checked")选取所有被选中的&lt;input>元素

4). $(:selected)选取所有被选中的选项元素(下拉列表)，$("selectoption:selected")选取所有被选中的选项元素

### (4)表单选择器：方便的获取到表单的某个或者某类型的元素

1). $(":input")选取所有的&lt;input>,&lt;textarea>,&lt;select>,&lt;button>元素

2). $(":text")选取所有的单行文本框

3). $(":password")选取所有的密码框

4). $(":radio")选取所有的单选框

5). $(":checkbox")选取所有的多选框

6). $(":submit")选取所有的提交按钮

7). $(":image")选取所有的图像按钮

8). $(":reset")选取所有的重置按钮

9). $(":button")选取所有的按钮

10). $(":file")选取所有的上传域

11). $(":hidden")选取所有的不可见元素

### 4.选择器中的一些注意事项

#### (1)选择器中含有特殊符号的注意事项

1). 选择器中含有".","、","#","(","]"等特殊字符

根据w3c的规定，属性中是不能含有这些特殊字符的，但在实际项目中偶尔会遇到表达式中含有"#","."等特殊字符，如果按照普通的方式去处理的话可能会出现错误，解决此类错误是使用转义符转义

例如：&lt;divid="id#b">韩迎龙&lt;/div>&lt;divid="id[1]">韩迎龙&lt;/div>,这时的取法是这样的$("#id\\#b"),$("#id\\[1\\]")

2). 属性选择器的@符号问题

在Jquery升级版本的过程中，jquery在1.3.1版本中彻底放弃了1.1.0版本遗留下来的@符号，加入你使用的是1.3.1以上的版本，

那么你不需要再属性前添加@符号，例如：

$("div[@title='test']")，正确的写法就是去掉@符号，$(div[title='test'])

(2). 选择器中含有空格的注意事项

选择器中的空格是不容忽视的，多一个空格或者少一个空格也许会得到截然不同的结果
　　
### 出处：http://www.cnblogs.com/hanyinglong     　　
