---
layout: post
title: "我的Javascript之旅——new以及构造器是如何工作的"
description: "Javascript里的Function（函数）,可以普通调用，也可以用new关键字来调用（这时这个函数旧事一个构造器），两种方式有什么不同？当用new关键字创建一个对象时，都发生了什么？"
category: Javascript
tags: [Javascript]
---

**(迁移自[博客园](http://www.cnblogs.com/CaiAbin/archive/2010/08/25/1808285.html))**

[接上](/javascript/2013/08/24/javascript-1-prototype/ "我的Javascript之旅——对象的原型链是如何实现的")
先看张对老手不新鲜但对菜鸟很有趣的图：

![1](/uploads/20130831/1.png)

What the heck is that? 简直是luan lun。 

##new

抛开上面的图，先看看上篇文章留下的第二个问题，让我们在构造器的函数体内加点东西，看会发生什么。
 
···javascript
function A(){this.p =1}
var a =new A()
```

会得到如下结果：
![1](/uploads/20130831/2.png)
　　　　

为什么用new关键字构造出来的a，会获得p这个属性？new A()这行代码做了什么事情？根据上篇文章中Function的创建过程第4步，A这个对象会有一个Construct属性（注意不是constructor，Consturct是ECMAScript标准里的属性，好像对外不可见），该属性的值是个函数，new A()即会调用A的这个Construct函数。那么这个Construct函数会做些啥呢？

1. 创建一个object，假设叫x。

2. 如果A.prototype是个object（一般都是），则把A.prototype赋给x.\_\_proto\_\_；否则（不常见），请大老板Object出马，把Object.prototype赋给x.\_\_proto\_\_。

3. 调用A.call(x)，第一个参数传入我们刚刚创建的x。这就妥了，A的函数体里this.p = 1，这个this，就成了x。因此x就有了p这个属性，并且x.p = 1。

4. 一般情况下，就返回x了，这时a就是x了。但也有__特殊情况__，如果A的函数体里返回的东西，它的类型(typeof)是个object。那么a就不是指向x了，而是指向A函数返回的东西。

伪代码如下：

```javascript
var x =new Object(); //事实上不一定用new来创建，我也不清楚。
x.__proto__ = A.prototype 
var result = A.call(x)
if (typeof(result) =="object"){
return result;
}
return x;
```

在我们的例子里，A函数返回undefined（因为没有return字眼），所以a就是x。但我们举个例子，验证下上面第4步里的特殊情况：

![1](/uploads/20130831/3.png)　

果然。

 

##对象的constructor属性

再看看上篇文章留下的第一个问题

```javascript
function Base(){}
Base.prototype.a =1
var base =new Base();

function Derived(){}
Derived.prototype = base;
var d =new Derived()
```

执行完上面的代码，base.constructor很容易猜到是Base，那么d.constructor呢？是Derived吗？

![1](/uploads/20130831/4.png)　　　　　

不对，也是Base，怎么回事？很简单，复习下上篇的内容就知道：由于d本身没有constructor属性，所以会到d.\_\_proto\_\_上去找，d.\_\_proto\_\_就是Derived.prototype，也就是base这个对象，base也没constructor属性，于是再往上，到base.\_\_proto\_\_上找，也就是Base.prototype。它是有constructor属性的，就是Base本身。事实上，就我目前所知，只有构造器（function类型的object）的prototype，才真正自己拥有constructor属性的对象，且“构造器.prototype.constructor === 构造器”。


##Instanceof

那么，instanceof怎么样？

![1](/uploads/20130831/5.png)　　　

从图中可以看出，d是Base、Derived和Object的实例。很合理，但这是怎么判断的呢？是这样的：对于x instanceof constructor的表达式，如果constructor.prototype在x的原型(\_\_proto\_\_)链里，那么就返回true。很显然，__d的\_\_proto\_\_链往上依次是：Derived.prototype, Base.prototype, Object.prototype__，得到图中结果就毫无疑问了。所以，instanceof跟对象的constructor属性无关。


##Function and Object

最后解答一下文章开头的图。

Function和Object本身也是function类型的对象，因此可以说都是Function()构造出来的东西（自己构造自己，我不知道具体是不是这样，但就这么认为，挺合理的。）

也就是说，可以设想如下代码：

```javascript
var Function =new Function();
var Object =new Function();
``` 

根据上篇文章的规律，会有Function.\_\_proto\_\_ === Function.prototype，以及Object.\_\_proto\_\_ === Function.prototype，验证一下：

![1](/uploads/20130831/6.png)
　　　　

Function instanceof Object，这是显然为true的，万物归Object管，Function的\_\_proto\_\_链依次指向：Function.prototype，Object.prototype。

Object instanceof Function，因为Function.prototype在Object的\_\_proto\_\_链中，所以也为true。

 

【update】

5楼的评论引用的图片，被chrome报有威胁，因此我移到这里来了。
  
Object 是对象的祖先
Function 是函数的祖先
函数可以做构造器
对象是函数new出来的
构造器的prototype是对象
对象的\_\_proto\_\_指向构造器的prototype
　　　　
![1](/uploads/20130831/7.jpg)

这样就都串起来了