---
layout: post
title: "我的JavaScript之旅——从Scope Chain到Closure"
description: "Javascript里每个函数(Function)都是运行在一个作用域(Scope)里，而且一个函数的作用域连着外部函数的作用域，像个链条一样，最终指向全局作用域。javascript会沿着这个链条由下而上地去找变量。也就是内部函数可以使用外部函数定义的变量,这就是闭包（Closure）的实现机制。"
category: Javascript
tags: [Javascript,Closure,Scope Chain]
---
{% include JB/setup %}

**(迁移自[博客园](http://www.cnblogs.com/CaiAbin/archive/2010/09/05/1818493.html))**

```javascript
a = 1;
function Outer(x){
      function Inner(y){return x + y;}
      return Inner
}
var inner = Outer(1);
inner(2);
```

执行上面这段代码的过程中，有哪些事情发生？Inner函数为什么可以引用Outer函数的参数x？closure是怎么实现的？本文试图回答这些问题。

 
##术语

本文虽然所讲理论并不复杂，但用到不少名词，初读时相对比较晦涩，下面列出术语和简短解释，便于阅读时随时查看。

* __global__：engine预先创建好的一个object，里面有所有built-in objects的属性。
* __globalContext__：本文术语，用作表示全局的execution context。
* __globalScopeChain__：本文术语，用作表示全局的execution context所拥有的Scope Chain，里面只有一个对象为global，用代码表示为 [global]
* __functionContext__：本文术语，用作表示执行函数代码时，进入的新的execution context。
* __VariableObject__：ECMAScript术语，在globalContext中即为global，在functionContext中是被创建的一个对象。在进入context时，被放到scope chain的最前方。
* __outerVariable__：本文术语，表示进入OuterFunctionContext时被创建的Variable Object。
* __innerVariable__：本文术语，表示进入InnerFunctionContext时被创建的Variable Object。
* __outerFunctionContext__：本文术语，用作表示执行Outer这个函数时，进入的execution context。
* __outerScopeChain__：本文术语，用作表示outerFunctionContext所拥有的Scope Chain。可用[outerVariable, global]表示。
* __innerFunctionContext__：本文术语，用作表示执行Inner这个函数时，进入的execution context。
* __innerScopeChain__：本文术语，用作表示innerFunctionContext所拥有的Scope Chain。可用[innerVariable, outerVariable, global]表示。
 

##JS代码种类

JS代码分三种：

1. Global code，全局代码
2. Functioncode，函数内的代码。
3. Eval code，为简单计，不在本文说明。
 

##Execution context

任何一句JS代码，都是执行在一个特定的__execution context__下面。

执行Global code时，JavaScript engine将会创建一个全局的context，为表述简单，我们把它叫做__globalContext__。

而每次进入Functioncode时，将会创建一个新的context，在函数返回（或有未捕获的异常发生）时，退出这个新的context，本文把它叫做__functionContext__。

```javascript
a = 1; //进入globalContext

function Outer(x){
      function Inner(y){return x + y;}
      return Inner
}        //在globalContext中创建Outer这个Function

var inner = Outer(1); //执行Outer函数时进入新创建的outerFunctionContext上下文。
                      //然后退出，回到globalContext，把Outer(1)的返回值赋给inner这个变量。
inner(2);  //进入InnerContext，执行Inner函数的return x + y，然后退出，回到globalContext
```

##Scope Chain

每个execution context都有一个关联的Scope Chain。所谓Scope Chain，其实就是一个List，里面有若干个object。

 
##global

globalContext所关联的Scope Chain，这里不妨称之为globalScopeChain，这个chain里面只有一个object，就是global，global是一个engine事先创建好的对象，所有的built-in Object（比如Function()、Object()、Math）都会作为这个global对象的属性。

 

##Function型对象的[[Scope]]属性

在[第一篇](/javascript/2013/08/24/javascript-1-prototype/ "我的Javascript之旅——对象的原型链是如何实现的")创建Function型对象的步骤里，第5步说了，会为这个Function型对象创建一个[[Scope]]属性，不过当初没有提到，这个属性的值是当前context的Scope Chain。

Outer函数是在globalContext下创建起来的，因此Outer.[[Scope]] = globalScopeChain，也就是[global]。而Inner函数是在执行Outer函数时，也就是在outerFunctionContext下创建起来的，因此Inner.[[Scope]] = OuterContext的ScopeChain，是什么呢，往下看。

 

##Entering execution context

每次进入一个context（不管是globaContext还是functionContext）时，都会有一系列的事情发生。

1. 上面说到，每个context都有一个关联的Scope Chain，这个Scope Chain就是在此时会被创建起来的。
2. 确定或创建一个Variable Object（ECMAScript术语），并把它放到Scope Chain的最前面。
对于globalContext，这个Variable Object就是global，被放到globalScopeChain里（也是globalScopeChain里唯一的一个对象）；
而如果进入到一个functionContext，则会创建一个Variable Object起来，也放到Scope Chain的最前面，并且还会额外再做一件事——就是把当前Function的[[Scope]]里所有object，放到Scope Chain里面。因此执行Outer函数时，Scope Chain是这样的：[outerVariable, global];上面知道，创建Inner函数时，这个Chain将作为Inner函数的[[Scope]]属性，因此进入Inner函数的执行时，它的Scope Chain就是[innerVariable, outerScopeChain]，也就是[innerVariable, outerVariable, global]。
3. 实例化Variable Object，就是为Variable Object创建一些属性。
..1. 首先，如果是functionContext，则把函数的参数作为Variable Object的属性；
..2. 其次，把声明的函数作为Variable Object的属性；这里的属性将覆盖上面的同名属性。
..3. 再次，把声明的变量作为Variable Object的属性，属性的初始值均为undefined，只有在执行赋值语句后，才会有值。这边的属性不会覆盖上面的同名属性。
4. 为当前context确定this，this在context中是不变的。
详细见下面的注解。
 

```javascript
//在执行一切代码之前，进入globalContext，global对象也已经创建好。

//1.然后创建Scope Chain
globalContext.ScopeChain = [];  

//2.确定variable object为global，并加入到scope chain中
variable = global;
globalContext.ScopeChain.push(global); 

//3.实例化variable object，创建a、Outer和inner三个属性，分如下步骤

//3.1  Outer是“函数声明”，因此此时就会被创建起来。 
Outer = new Function('', '' [global]) //创建Outer函数，传入当前的scope chain，即[global]
Outer.[[Scope]] = [];  
Outer.[[Scope]].push(global); //为Outer.[[Scope]]赋值
variable.Outer = Outer  //把创建起来的函数作为variable的属性。
//3.2  对于变量a和inner，也创建为variable的属性，不过初始值为null
variable.a = null;
variable.inner = null;

//4.确定this，在globalContext中为global。
this = global;

//以上是进入globalContext时所做的事情
 
//以下开始执行代码。
a = 1;  //variable.a 此时才被赋值。
function Outer(x){
    function Inner(y){return x + y;}
    return Inner;
}      
var inner = Outer(1); //这段代码用伪代码表示如下：
//Outer(1)，会执行Outer函数，因此进入新创建的outerFunctionContext上下文
//1.创建ouerFunctionContext的Scope Chain，并放入Outer函数的[[Scope]]里所有的object
outerFunctionContext.ScopeChain = [];
outerFunctionContext.ScopeChain.push(global) //global是[[Scope]]里唯一的对象。
//2.创建Variable Object属性，并放到Scope Chain的最前方。
outerVariable= {arguments: xxx} //创建的variable有arguments等属性
outerFunctionContext.ScopeChain.push(variable)
//3.实例化variable object
outerVariable.x = 1
outerVariable.Inner = new Function('y', 'return x + y', [outerVariable, global]) 
 //注意上句创建Inner函数时，会传入当前的Scope Chain，即[outerVariable, global]
 //4.确定Outer函数体内的this参数，就是Outer函数对象。

//最后回到globalContext中，把新建的Inner函数对象，返回给inner变量。

inner(2);  //最后执行的这句代码，将创建并进入InnerContext。
```

 

##初步结论

现在已经知道，执行Outer函数时，对应的outerScopeChain的图如下，注意global对象忽略了指向所有built-in object的属性：

![outerScopeChain](/uploads/20130901/outerScopeChain.png)
　　

执行Inner函数时，对应的innerScopeChain的图如下：
![innerScopeChain](/uploads/20130901/innerScopeChain.png)　

##Scope Chain的作用

Scope chain的图出来了，那么它用来干嘛呢？执行inner函数的return x + y，会发现，我们需要两个变量，x和y。那么JavaScript将循着Scope Chain来查找，与\_\_proto\_\_链配合，也就是首先在innerVariable（以及其\_\_proto\_\_链）找x，没找到，则到outerVariable中找x，找到为1。 找y时类似。这就是Inner函数体中，可以访问得到Outer函数中定义的参数x的原理所在，不难想象，如果Outer函数中定义了局部变量z，那么z也会出现在outerVariable对象中，因此同样可以被Inner函数访问。内部函数可以引用外部函数的参数以及变量，这就是JavaScript传说中的__闭包(Closure)__。

 

[下篇](/javascript/2013/09/01/javascript-4-the-creation-of-closure/ "“闭包”是什么时候创建的")将对closure进一步阐述，待续。