---
layout: post
title: "我的Javascript之旅——对象的原型链是如何实现的"
description: "Javascript没有类，只有对象，对象的继承是基于原型链(Scope Chain)的，那么在javascript内部，原型链是怎么实现的呢？"
category: Javascript
tags: [Javascript]
---
{% include JB/setup %}

**（迁移自[博客园](http://www.cnblogs.com/CaiAbin/archive/2010/08/25/1808001.html)）**

以问题开始：

```javascript
function Base(){	
};
var base = new Base();
```

上面两行代码会创建几个对象（object）？


要回答这个问题，先明确一下Javascript里object的概念。


##Objects

在Javascript里，几乎一切都是object(Arrays、Functions、Numbers、Objects……)，而没有C#里的class的概念。object的本质是一个name-value pairs的集合，其中name是string类型的，可以把它叫做“property”，value包括各种objects（string，number，boolean，array，function…），指的是property的值。

 

##typeof

既然object包含Arrays、Functions、Numbers、Objects……，那怎么区分这些呢？答案是typeof。 typeof返回一个字符串，如typeof(Null) = “object”,typeof(false) = “Boolean”, typeof(1) = “number”。既然总是返回字符串，那么对于typeof (typeof x)，不管x是什么，总是返回”string”。

![typeof](/uploads/201308/typeof.png "typeof (typeof xxx) === string")
 

##Constructor

JS里没有class，也就没有class里的构造函数，那么object是怎么被创建的呢？用构造器：constructor。constructor其实就是Function，因此本身也是object。开头的function Base(){}就是一个构造器，var b = new Base()就是用这个构造器（通过关键字new）创建了一个叫b的object。至此我们可以得出结论，开头的两行代码至少创建了2个object：一个是Base,类型为function的object，一个是base，类型为object的object。

 

##Function()和Object()

这是两个重要的预定义好的构造器。一切function（比如开头的Base()）都是由Function()构造出来的；而Object的prototype将会被所有object继承，下面会讲到。

　　　　
![Function](/uploads/201308/Function.png)
 

##Function的创建过程

当执行function Base(){this.a = 1}时，相当于var Base = new Function(“this.a = 1”)，也就是说，这行代码本身，将使用预定义好的Function() constructor，来构造一个function型object(即Base)出来。在这个创建过程中，js将做哪些事呢？

1. 首先当然会创建一个object起来，Base指向这个object。typeof 这个object = “function”

  ![Function1](/uploads/201308/function_1.png)

2. 给Base附上\_\_proto\_\_属性，让它等于Function这个构造器的prototype（也是预定义好的）。这是很重要的一步，也是规律性的一步。（规律：）在执行任意类似varx = new X()时，都会把X的prototype赋给x的\_\_proto\_\_，也就是说，x.\_\_proto\_\_和X.prototype此时会指向同一个对象。

  ![Function2](/uploads/201308/function_2.png)

3. 为Base创建call属性，该属性是个function。因此我们可以这样写：Base.Call()

  ![Function3](/uploads/201308/function_3.png)
  
4. 为Base创建Construct属性，该属性也是个function。在执行var base = new Base()时，即会调用这个Construct属性。
5. 为Base创建Scope，Length等属性，略。
6. 为Base创建prototype属性：先用new Object()创建一个对象，为这个对象创建一个属性叫constructor，该属性值设置为Base。再把Base的prototype设置为这个新创建的对象。伪代码如下：

```javascript
var x = new Object();
x.constructor = Base;
Base.prototype = x;
```

先把关注点放到2和6。
 

##\_\_proto\_\_和“prototype”

从2可以看出来，任意一个用构造器构造出来的object（包括Objects和Functions），都会有\_\_proto\_\_属性，指向该构造器的prototype属性。注意\_\_proto\_\_是个私有属性，在IE上是看不到的，我用的是chrome，可以看到。

从6可以看出，任意一个用new Function()构造出来的functions，都会有prototype属性，该属性是用new Object()构建出来的，初始公开属性只有一个constructor。

![prototype](/uploads/201308/prototype.png)

 

##原型链

再来分析下第6步的伪代码，也就是为function创建prototype的这一步：

```javascript
var x = new Object();  //  参见2中的规律，会有x.__proto__= Object.prototype。
x.constructor = Base;
Base.prototype = x;
```

此时我们用Base()构造一个对象出来：

```javascript
var base= new Base(); // 参见2中的规律，会有base.__proto__ = Base.prototype，也就是 = x。
                       // 因此有base.__proto__.__proto__ = x.__proto__
                      // 而x.__proto__ = Object.prototype(见上一个代码片段)　　
                      // 所以，base.__proto__.__proto__ = Object.prototype.
```

\_\_proto\_\_.\_\_proto\_\_，这就是传说中JS对象的原型链！由于用Function()创建构造器时的关键的第6步，保证了所有object的原型链的顶端，最终都指向了Object.prototype。

![prototypechain](/uploads/201308/prototypechain.png)
 

##Property Lookup

而我们如果要读某个object的某个属性，JS会怎么做呢？

比如有个object叫xxx，我们执行alert(xxx.a)，也就是读取xxx的a属性，那么JS首先会到xxx本身去找a属性，如果没找到，则到xxx.\_\_proto\_\_里去找a属性，由此沿着原型链往上，找到即返回（没找到，则返回undefined）。可以来看个例子：

![hasOwnProperty](/uploads/201308/hasOwnProperty.png)

上图得知：base本身是没有constructor属性的，但是base.constructor确实能返回Base这个函数，原因就在于base.\_\_proto\_\_有这个属性。（base.\_\_proto\_\_是啥？就是Base.prototype，上面构建Function的第6步的伪代码里，为Base.prototype.constructor赋值为Base本身。）

 

##Object作为“基类”

另外，由于任意object的原型链的顶端都是Object.prototype。所以，Object.prototype里定义的属性，就会通过原型链，被所有的object继承下来。这样，预定义好的Object，就成了所有对象的“基类”。这就是原型链的继承。

![inheritence](/uploads/201308/prototypeInheritence.png)　　　

看上图，Object.prototype已经预定义好了一些属性，我们再追加一条属性叫propertyA，那么这个属性和预定义属性一样，都可以从base上读到。

 

##原型继承

已经得知，

对于 var xxx =new Object(); 有xxx.\_\_proto\_\_= Object.prototype;

对于 var xxx =new Base(); 有xxx.\_\_proto\_\_.\_\_proto\_\_= Object.prototype;

看上去很像什么呢？从c#角度看，很像Base是Object的子类，也就是说，由Base()构造出来的object，比由Object()构造出来的object，在原型链上更低一个层级。这是通过把Base.prototype指向由Object()创建的对象来做到的。那么自然而然，如果我想定义一个继承自Base的构造器，只需把改构造器的prototype指向一个Base()构造出来的对象。

```javascript
function Derived(){}
var base = new Base();
Derived.prototype = base;
var d = newDerived();  //很容易推算出:d.__proto__.__proto__.__proto__ = Object.prototype.
```

推算过程：d.\_\_proto\_\_指向Derived.prototype，也就是base；则\_\_proto\_\_.\_\_proto\_\_指向base.\_\_proto\_\_，也就是Base.prototype，也就是某个new object()创建出来的东东，假设是o；则\_\_proto\_\_.\_\_proto\_\_.\_\_proto\_\_指向o.\_\_proto\_\_，也就是Object.prototype。

 

##回答开头的问题，以及几个新的问题

那两行代码至少创建了三个对象：Base、base、Base.prototype。顺便说说，base是没有prototype属性的，只有function类型的object，在被构建时才会被创建prototype属性。

![onlyFunctionHasPrototype](/uploads/201308/onlyFunctionHasPrototype.png)
 

d.constructor会返回什么呢？

构造器Base()和Derived（）里都是空的，如果有代码，将会怎么被执行呢？

......

[待续。](/javascript/2013/08/31/javascript-2-new/ "我的Javascript之旅——new以及构造器是如何工作的")