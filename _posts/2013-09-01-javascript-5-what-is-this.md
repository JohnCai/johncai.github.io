---
layout: post
title: "我的JavaScript之旅——this到底是什么？"
description: ""
category: Javascript
tags: [Javascript, this]
---
{% include JB/setup %}

**(迁移自[博客园](http://www.cnblogs.com/CaiAbin/archive/2010/09/25/1834797.html))**

下图是在ASP.NET中为button挂上客户端onclick事件的两种办法：图中的2和3/1。 结果发现两种方式调用同样一个函数clickMe，this却不一样。　
![asp.net](/uploads/20130901/aspnet.png)

如果采用3或1的做法，那么点击button1后将alert出[object DOMWindow]；而采用2的做法，将alert出 [object HTMLInputElement]（在chrome下测试。）

显然，在1的做法中，this指向DOMWindow，也就是全局的object——global；而2的做法中，this指向Button1这个元素。

为什么呢？

(注：对于3这种通过Attribute来声明处理程序的方式，button的onclick不是指向clickMe函数，而是指向一个被自动创建的匿名函数，该匿名函数以Attribute的值（也就是"clickMe();"）作为函数体——也就是等价于1。)



###简单的答案

　　基于类的OO语言（比如C#）中，函数总是声明在一个类中，函数内部的this指向该类的当前实例。而JS中，函数是第一等公民（它不会声明为别的东西的一部分），所以this跟函数是如何声明的无关，跟函数是怎么被调用的有关。

　　方式1和2对clickMe的调用，不同之处在于：1中clickMe是被button1.onclick所指向的函数调用的，2中clickMe是作为button1.onclick属性直接调用的。因此对于clickMe函数，1中的this指向“拥有”clickMe函数的对象——global（DOMWindow），而2中的this指向“拥有”onclick属性的对象——button1。

　　可惜对像我这种写JS写的不多的人来说，总是记不住这个简单答案，因为它只告诉我what，没有告诉我why。本文试图从ECMAScript官方文档出发，从原理上说明：__在不同的场合中，函数的this到底是什么？__

 

###call和apply

　　首先明确一点，在最正常的情况下，我们这样调用：Func()，这时this是由JavaScript来确定的，这也是本篇要研究的主题。而如果用Func.call(thisArg,  arg1, arg2, ...)或者Func.apply(thisArg, [arg1, arg2, ...])来调用时，this是我们自己传进去的（作为call或apply的第一个参数）。如果我们不传this进去，或者传null进去，会怎样？这时this将会是global object。

![this1](/uploads/20130901/this1.png)
　　

###函数作为构造器时的this

先从简单的说起。所谓构造器，就是用new关键字来调用函数，在这篇[关于new关键字的玄机的文章中](/javascript/2013/08/31/javascript-2-new)有说到。看下面的注释，可以知道，A函数里的this这时就是a。

```javascript
function A() {this.x = 1;}

var a = new A();  //这行代码大概等价于下面两行
 
var a = {};   // a指向一个新创建的对象。
A.call(a);       //再把a作为第一个参数传给A.call函数。
```
 

###普通调用时的this

看下图，对于在全局定义的函数A，this就是global。

![this1](/uploads/20130901/this2.png)

这个好理解，A作为顶级函数，是顶级对象global（也就是window）的属性，所以this === global。但看下图：

![this1](/uploads/20130901/this3.png)

　　

Outer()返回inner函数，Outer()()就是是对inner的调用，在inner内部，this同样等于global。可是inner函数是执行Outer时，在Outer的context下创建起来的，inner并不是global的属性（见下图）。为什么inner里的this仍旧是global？一个函数被调用时，this究竟是怎么确定的？

![this1](/uploads/20130901/this4.png)

　　
###官方文档对this的解释

上面第1个例子，当执行A()时，[这里说明了函数被调用时发生的事情](http://bclary.com/2004/11/07/#a-11.2.3 "Function Calls")，简述如下：

1. 得到A这个引用。

2. 求A的值，就是A这个函数对象。

3. A的值必须是object，并且必须有[[Call]]方法。（当然，它是函数型object，函数被创建时都有[[Call]]方法。）

4. 如果A是个引用（是），那么通过GetBase()得到A所在的对象，就是包含有A属性的对象（在我们例子中就是global）

5. 检查上面步骤中A所在的对象。如果是个activation object（就是variable object，scope chain里的对象），则把null作为thisArg传给A.call()；否则就把该对象作为thisArg。

重点就是这第5步（需要对variable object熟悉，否则请先阅读[这篇文章](/javascript/2013/09/01/javascript-3-from-scopechain-to-closure "从原型链到闭包")），执行A()时，global里有A属性，而global是global context下的activation object，所以将会把null作为thisArg参数传给A.call()。而上文说了，如果传的thisArg参数为null，则会把global作为this。所以gloabl === this还不是天生就这样，还转了下弯。

　　对于第2个例子，Outer()()，也就好解释了：因为Outer()返回的inner函数是在OuterContext中创建为OuterVariableObject的属性，所以第4步得到的对象是OuterVariableObject；那么根据第5步逻辑，仍旧把null作为thisArg传给inner.call()。因此inner里的this也同样是global。

 
###把函数作为对象的属性值

　　所以，如果直接调用函数，无论这个函数是在哪个执行上下文被创建，this都指向global。除非“拥有”这个函数的对象不是一个variable object，它才会被当做this。那么很简单，我们把函数作为我们自己创建的对象(非variable object)的属性值，然后再调用，那么this就应该是这个我们创建的对象了。验证如下：

![](/uploads/20130901/this5.png)


回过头看文章开头的问题：

__方式1__是这样调用的：Button1.onclick(clickMe();)，clickMe是global的属性，因此clickMe里的this指向global。

__方式2__是这样调用的：Button1.onclick()，onclick是Button1的属性，因此里面的this指向Button1。

简言之，onclick和clickMe都是对真正的函数object的引用，只不过通过GetBase()得到的“所属对象”不同，this自然也就不同了。

 

###阶段性结语

　　我的JavaScript之旅先告一段落了（快写吐了），5篇文章写下来，至少自己收获不少，下次面人的时候，终于可以问点JS的东西了。

　　this, scope chain, prototype chain, constructor, closure，这些都是学习JavaScript必须懂的东西，每本JS的书都会对这些概念进行描述，我看过几篇，都是讲what，就是要你死记硬背，或者多写JS，熟能生巧才行。没有看到一篇讲why的。而我把这5篇写why的博文写完，再看那些书籍上的描述，真有一览众山小的感觉，哈哈。

　　至于对这些概念的运用，就需要多写多看JS代码了，不想写那样的文章了。JS告一段落，要研究别的东西去了。FubuMVC、Ruby on Rails、CQRS……