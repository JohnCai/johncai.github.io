---
layout: post
title: "我的JavaScript之旅——Closure（闭包）是什么时候创建的"
description: "Javascript中什么时候创建的闭包（Closure）？据说它容易引起内存泄露？为什么？垃圾回收（garbage collection）为什么不管它？"
category: Javascript
tags: [Javascript, closure]
---
{% include JB/setup %}
**(迁移自[博客园](http://www.cnblogs.com/CaiAbin/archive/2010/09/14/1826287.html))**

```javascript
function Outer(){
    var x = 1;
    function Inner(y) {return x + y}; 
    return Inner;
}
```

对于这样一个简单的闭包函数，下面两种调用方式有什么不一样的地方？

```javascript
//方式1
var inner1 = Outer();
var result = inner1(2); //3
```
 
```javascript
//方式2
var result = Outer()(2); //3
```

此篇试图解答这个问题。先复习一下：

[上篇文章](/javascript/2013/09/01/javascript-3-from-scopechain-to-closure/ "从Scope Chain到Closure")说到，每次执行一个function时，就会进入一个新的“执行上下文”（execution context）。context的几个重要属性：

1. 有一个对应的variable object；在global context中就是global object。
2. 有一个对应的scope chain，这个scope chain的第一个object就是variable object；
3. 有一个不变的this变量。

其中第2条，scope chain是JS实现闭包（closure）的关键所在，这篇对“闭包”展开描述，以加深印象。后续文章将对this专门探讨。

 

##Variable object的实例化三部曲

我们已经知道，当JS执行时碰到一个变量，它会到scope chain里递归去找，而scope chain是由variable object和global object组成的一个object chain，global object包含JS预定义好的所有object，function的variable object则会包含函数内声明的所有东西，包括function的参数、内部函数、局部变量。创建Variable object的过程有三步，上篇有写过，这边是[官方的文档](http://bclary.com/2004/11/07/#a-10.1.3)。简述如下：

1. 为variable object创建与函数参数同名的属性；属性值为传入的参数值。

2. 对于“function declaration(见下节)”，首先创建函数，然后为variable object创建属性；属性值即为该函数实例。覆盖1的同名属性。

3. 为variable object创建各变量的属性；属性值为undefined。不覆盖1,2的同名属性。

 

##function declaration vs. function statement

下面就是一个function declaration:　　

```javascript
function A(){
}
```
 

下面是一个function statement:

```javascript
var A = function A(){
}
```

这是创建函数的三种方式之二，区别在上面的三部曲中就可以看出来：

1. 在进入execution context时（还没执行任何代码），function declaration就已经在第2步创建函数实例起来了；而function statement属于第3步，而且初始值是undefined，要到执行这行代码时，函数才会被创建起来。

2. function statement不会覆盖同名的function declaration。

验证一下：

![1](/uploads/20130901/1.png)

　　如图：A可以先用再声明；B则不行。B在执行到最后一句之前，都是undefined。但是B这个属性是一开始就在的：

![1](/uploads/20130901/2.png)
　　

　　（this是什么？因为上面的代码是执行在global context中，B属性应该创建到global object身上，也就是this。下篇会详述this。）
 

三部曲中说了，function declaration会覆盖同名的1里的参数，所以它是varaible object里的第一等公民：

![1](/uploads/20130901/3.png)
　　

而function statement不会。

![1](/uploads/20130901/4.png)


##闭包的创建时机

对于开头的这段代码：

```javascript
function Outer(){
    var x = 1;
    function Inner(y) {return x + y}; 
    return Inner;
}
```
 

然后执行 

```javascript
var inner1 = Outer();
``` 

回忆一下这时会发生什么？

会进入一个新的“执行上下文”（Outer Context），创建OuterVariableObject（有x和Inner属性），放到OuterScopeChain的最前方。而且会创建Inner函数，__创建时把当前Scope Chain作为Inner函数的[[Scope]]属性。__这是重点。

创建起来的Inner函数被inner1变量引用，Inner有[[Scope]]属性，引用了OuterScopeChain，即[OuterVariableObject, global object]，而OuterVariableObject又引用了局部变量x。__所以inner1变量就对Outer函数体内的局部变量x有间接的引用。内部函数对外部函数的变量有了引用关系——闭包就是这时产生的。每次对外部函数的调用，都会产生一次闭包。__



##garbage collection

很多人都听过闭包容易引起内存泄露。为什么呢？因为如上所述，inner1变量对x有间接引用，而inner1是声明在global context下的一个变量，它在global context下随时可以被用，那么JS的垃圾回收器就不会回收它（inner1），当然也就不会回收它所引用的x——直到退出global context，也就是我们关掉网页的时候。

这就是文章开头两种调用方式的区别：方式1，x在关掉网页前一直不能被回收；而方式2，x会被回收。

 
[后续文章](/javascript/2013/09/01/javascript-5-what-is-this/ "this到底是什么")将介绍闭包的用处，和this关键字（比我想象的复杂）。