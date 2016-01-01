---
layout: post
title: "JavaScript的构造函数、new、原型"
date: 2015-12-31 21:38:21 +0800
comments: true
categories:
---

在JavaScript中，对象是一系列的键值对，ECMA-262把对象（object）定义为“属性的无序集合，每个属性存放一个原始值、对象或函数”。

JavaScript是面向对象的语言（而且比C++/Java更加的面向对象），因为所有东西都是对象，包括函数，但是在JavaScript没有类的概念。既然没有类，根据对之前对C++/Java这样的基于类的面向对象语言的理解，应该如何理解JavaScript中构造函数的概念，因为在基于类的面向对象中，构造函数只存在于类当中。

###JavaScript构造函数、JavaScript中的constructor属性

在JavaScript中，每个具有原型的对象都会自动获得constructor属性。除了arguments、Enumerator、Error、Global、Math、RegExp、Regular Expression等一些特殊对象之外，其他所有的JavaScript内置对象都具备constructor属性。例如：Array、Boolean、Date、Function、Number、Object、String等。所有主流浏览器均支持该属性。

现在，请暂且不要去思考“具有原型的对象”中原型的意思。

对象的constructor属性返回创建该对象的函数的引用，无论直译或者按照基于类的面向对象语言的理解，我们且把该函数称为“构造函数”。下面是一些例子：

{% codeblock lang:javascript %}
// 字符串：String()
var str = "张三";
document.writeln(str.constructor); // function String() { [native code] }
document.writeln(str.constructor === String); // true

// 数组：Array()
var arr = [1, 2, 3];
document.writeln(arr.constructor); // function Array() { [native code] }
document.writeln(arr.constructor === Array); // true

// 数字：Number()
var num = 5;
document.writeln(num.constructor); // function Number() { [native code] }
document.writeln(num.constructor === Number); // true

// 自定义对象：Person()
function Person(){
    this.name = "CodePlayer";
}
var p = new Person();
document.writeln(p.constructor); // function Person(){ this.name = "CodePlayer"; }
document.writeln(p.constructor === Person); // true

// 字面量对象：Object()
var o = { "name" : "张三"};
document.writeln(o.constructor); // function Object() { [native code] }
document.writeln(o.constructor === Object); // true

// 自定义函数：Function()
function foo(){
    alert("CodePlayer");
}
document.writeln(foo.constructor); // function Function() { [native code] }
document.writeln(foo.constructor === Function); // true

// 函数的原型：bar()
function bar(){
    alert("CodePlayer");
}
document.writeln(bar.prototype.constructor); // function bar(){ alert("CodePlayer"); }
document.writeln(bar.prototype.constructor === bar); // true
{% endcodeblock %}
代码中的[native code]，表示这是JavaScript的底层内部代码实现，无法显示代码细节。

你会发现连那些看似是基本类型的数字“5”和字符串“张三”（当然Java里面有String类型）都具有constructor属性。特别是定义一个函数foo，foo也有constructor属性，而且指向名字是Function的函数（暂且不管为什么如此）。

通过var str = "张三";，我创建了一个String对象，通过var num = 5;，我创建了一个Number对象。

在Java或者C++中，通过new关键字调用某个类的构造函数，例如：new SomeConstructor()，来创建对象，知识SomeConstructor和类名一样。

那么在JavaScript，如何自定一个具有类型对象呢？上面的Person例子已经给出了答案。

p对象的constructor是它：function Person(){ this.name = "CodePlayer"; }

那么要创建一个新的p对象，只需要var p = new Person();。表面上的理解和C++或者Java相似。

剩下的疑问是创建这个对象的时候使用了new关键字。它是干什么的？new关键字很容易让你想到C++和Java中通过new来创建新的对象（分配一段内存空间）。

在《JavaScript高级编程》里对new操作符的解释：

new操作符会让构造函数产生如下变化：    
1.创建一个新对象    
2.将构造函数的作用域赋给新对象（因此this就指向了这个新对象）    
3.执行构造函数中的代码（为这个新对象添加属性）   
4.返回新对象

MDN上也有介绍new关键字： https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new

{% codeblock lang:javascript %}
function Person(){ this.name = "CodePlayer"; console.log(this) }
var p = Person()
$ Window {external: Object, chrome: Object, document: document, WPCOMSharing: Object, RecaptchaTemplates: Object…}

var p = new Person()
$ Person {name: "CodePlayer"}
{% endcodeblock %}

JavaScirpt中的构造函数和普通函数没有什么区别，你一样的可以像普通函数一样调用它，如上例，那么this指向函数执行时的当前对象（关于this指针会在之后的文章中详解）。

但如果通过new关键字来调用函数，该函数就成为了构造函数，this指针就会指向新创建的对象。这就是JavaScript的构造函数。

###通过new和构造函数创建对象的问题

new的方式创建对象看上去非常好用，而且和C++或者Java语言很相似，比较容易理解它在创建一个新的对象。但是构造函数方法创建对象存在一个浪费内存的问题。

以下摘抄自阮一峰的文章： http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_encapsulation.html

“请看，我们现在为Cat对象添加一个不变的属性"type"（种类），再添加一个方法eat（吃老鼠）。那么，原型对象Cat就变成了下面这样：”

{% codeblock lang:javascript %}
function Cat(name,color){
  this.name = name;
  this.color = color;
  this.type = "猫科动物";
  this.eat = function(){alert("吃老鼠");};
}

var cat1 = new Cat("大毛","黄色");
var cat2 = new Cat ("二毛","黑色");
alert(cat1.type); // 猫科动物
cat1.eat(); // 吃老鼠

alert(cat1.eat == cat2.eat); //false
{% endcodeblock %}

“表面上好像没什么问题，但是实际上这样做，有一个很大的弊端。那就是对于每一个实例对象，type属性和eat()方法都是一模一样的内容，每一次生成一个实例，都必须为重复的内容，多占用一些内存。这样既不环保，也缺乏效率。能不能让type属性和eat()方法在内存中只生成一次，然后所有实例都指向那个内存地址呢？回答是可以的。”

解决这个问题，阮一峰在他的文章中介绍了Prototype模式。我猜测对JavaScript面向对象感兴趣的同志已经无数次看到这样的代码了。

##JavaScript原型、Prototype和\_\_proto\_\_

prototype英文翻译过来“原型”

什么是原型？原型是一个从其他对象继承属性的对象。

是不是任何对象都可以是原型？是的。

哪些对象有原型？每个对象都有一个默认的原型。原型本身就是对象，每一个原型本身也存在一个原型。（只有一个例外，默认的对象原型在每条原型链的顶端，其他的原型在原型链的后面）

当我看到上面的回答时，加上我对prototype英文含义的理解，我会认为每个对象都有一个prototype. 但当我写({}).prototype的时候，或者"".prototype，我却得到了undefined，是不是疯了？

忘记你所理解的关于prototype属性的理解（它其实只是函数对象的一个属性，比如Object.prototype，Function.prototype，Array.prototype，在浏览器控制台查看Object，Function，Array这些值，它们都是函数） - 这个名字很可能是迷惑的根源。

一个对象真正的prototype是内部[[Prototype]]属性. ECMA5介绍了标准的访问方法，Object.getPrototypeOf(object)。这个最新的实现已被Firefox, Safari, Chrome and IE9所支持。另外，除了IE，所有的浏览器都支持非标准的访问方法\_\_proto\_\_，它指向当对象被实例化的时候，用作原型的对象。

那么我想知道prototype属性到底是干什么的?比如Object.prototype，Function.prototype，Array.prototype等。特别是上面的阮一峰介绍的Prototype模式。

Object.prototype属性表示对象Object的原型对象，它是所有对象原型链的根节点。

那其他的呢？好吧，首先，在前面介绍构造函数时已经知道，JavaScript不区分构造函数和其它普通函数，所以每个函数都有prototype属性。反而任何不是方法的，都没有这样的属性。

{% codeblock lang:javascript %}
//永远不是构造函数的方法，无论如何都是有prototype属性的
Math.max.prototype; //[object Object]

//构造函数也有prototype属性
var A = function(name) {
    this.name = name;
}
A.prototype; //[object Object]

//Math不是一个方法，所以没有prototype属性
Math.prototype; //null
{% endcodeblock %}

现在可以定义:   
一个方法的prototype属性是当这个方法被用作构造函数来创建实例时，赋给该实例的原型（内部Prototype）的对象。非常重要的一点是，要理解方法的prototype属性和实际的prototype（原型）没有任何关系。看下面一段代码和解释：

{% codeblock lang:javascript %}
//构造器，this作为新对象返回并且它内部的[[prototype]]属性将被设置为构造器默认的prototype属性
var Circle = function(radius) {
    this.radius = radius;
    //next line is implicit, added for illustration only
    //this.__proto__ = Circle.prototype;
}

//扩充 Circle默认的prototype对象的属性因此扩充了每个由它新建实例的prototype对象的属性
Circle.prototype.area = function() {
   return Math.PI*this.radius*this.radius;
}

//创建Circle的两个示例，每个都可以使用相同的真正prototype所拥有属性area
var a = new Circle(3), b = new Circle(4);
a.area().toFixed(2); //28.27
b.area().toFixed(2); //50.27
{% endcodeblock %}

那么，根据上面的定义，下面的内容非常有趣：
{% codeblock lang:javascript %}
Function.prototype
$ function() {}
var func = new Function()
func.__proto__
$ function() {}
var functionA = function (){}
functionA.__proto__
$ function() {}
var myFunction = new Function('users', 'salary', 'return users * salary');
myFunction.__proto__
$ function() {}
Array.prototype
$ []
var arr = new Array()
arr.__proto__
$ []
var array = []
array.__proto__
$ []
Cat.prototype
$ Cat {}
{% endcodeblock %}

##原型链和instanceof

###原型链又是什么？

因为每个对象和每个原型(本身)都有一个原型，我们可以想象，一个接一个的对象连接在一起形成一个原型链。原型链的终端总是默认对象（object）的原型，即Object.prototype。你可以在自定义的对象上调用\_\_proto\_\_方法就来自于Object.prototype.\_\_proto\_\_上。

{% codeblock lang:javascript %}
function Cat(){}
var cat = new Cat()
cat.__proto__
$ Cat {}
cat.__proto__.__proto__
$ Object {}
Object.prototype
$ Object {}
cat.__proto__.__proto__ === Object.prototype
$ true
{% endcodeblock %}

原型继承机制是内在且隐式实现的。当对象a要访问属性foo时，Javascript会遍历a的原型链（首先从a自身开始），检查原型链的每一个环节中存在的foo属性。如果找到了foo属性就会将其返回，否则返回undefined值。

{% codeblock lang:javascript %}
var A = function() {};
A.prototype.constructor == A; //true

var a = new A();
a.constructor == A; //true (a 的constructor属性继承自它的原型)
{% endcodeblock %}

###instanceof与prototype有啥关系？
如果A的prototype属性出现在a的原型链中，则表达式a instanceof A会返回true。这意味着我们可以欺骗instanceof，让它失效。
{% codeblock lang:javascript %}
var A = function() {}

var a = new A();
a.__proto__ == A.prototype; //true - so instanceof A will return true
a instanceof A; //true;

//mess around with a's prototype
a.__proto__ = Function.prototype;

//a's prototype no longer in same prototype chain as A's prototype property
a instanceof A; //false
{% endcodeblock %}

##总结

看完这篇文章，你需要记住一下几点：

构造函数和普通函数没有区别，只有在结合new关键字时，有特定的作用，可以创建一个对象实例，而该对象实例有一个constructor属性指向创建它的函数，即构造函数。每个对象都有原型，不要误解Prototype属性，它是函数对象的属性（比如Object，Array，Function），真正的原型通过Object.getPrototypeOf(object)和\_\_proto\_\_获取。因为构造函数隐式的执行this.\_\_proto\_\_ = Circle.prototype，所以Prototype模式可以实现对象方法是定义（对象中的属性只是引用，指向的是构造函数Prototype属性上定义的一个方法）。

参考文献：     
1.http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_encapsulation.html    
2.http://www.oschina.net/translate/understanding-javascript-prototypes    
3.http://yehudakatz.com/2011/08/12/understanding-prototypes-in-javascript/   
