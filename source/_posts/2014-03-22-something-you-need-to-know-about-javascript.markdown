---
layout: post
title: "JavaScript中你需要了解的基本知识（一）"
date: 2014-03-22 14:16
comments: true
categories: 
---

对于初学JavaScript的人很容易去关注如何快速使用它，而忘记学好一门语言，应该首先掌握的基本知识，我就是这样。但是基础知识才是真正掌握一门语言最关键的部分，特别是JavaScript这个看上去好像和C++、Java相同，但实际上完全不是一回事的语言。

##**作用域**

在编程语言中，作用域控制着变量与参数的可见性及生命周期，是一个很基本的问题。

但是和类C语言不同，JavaScript没有块级别作用域。

所谓块级作用域，即在一个代码块中定义的所有变量在代码块的外部都是不可见的，定义在代码块中的变量在代码块执行结束后会被释放掉。

Java中print方法中的a肯定是未定义的变量，编译器会标红，因为a是if块中的局部变量。
{% codeblock lang:java %}
public static void main(String args[]) {
	if(true){
		int a = 0;
	}
	System.out.print(a);
}
{% endcodeblock %}

但是在JavaScript中，a变量就可以被访问。
{% codeblock lang:javascript %}
if(true){
	var a = 1;
}
console.log(a);
{% endcodeblock %}

JavaScript是由函数限定变量的作用域，意味着定义在函数中的参数和变量在函数外部是不可见的，而函数内可以访问函数外部的变量。
{% codeblock lang:javascript %}
var carName = 'byd';

function visitName() {
	console.log(carName);
}
visitName();
{% endcodeblock %}

JavaScript允许延迟声明变量，但是对于JavaScript缺少块级别作用域，所以这是个悲剧，推荐的做法是在函数的开头声明所有需要使用的变量。

##**词法作用域和作用域链**

JavaScript是一种解释性语言。在解释过程中，JavaScript引擎是严格按照作用域来执行的。JavaScript语法采用的是词法作用域，也就是说JavaScript的变量和函数作用域是在定义时决定的，而不是执行时决定的，由于词法作用域决定于代码结构，所以JavaScript解释器只需要通过静态分析就能确定每个变量、函数的作用域，这种作用域也称为静态作用域。

JavaScript解释器通过作用域链把多个嵌套的作用域串联在一起，并借助这个链条帮助JavaScript解释器检索变量的值。这个作用域链相当于一个索引表，通过编号来存储它们的嵌套关系。当JavaScript解释器检索变量的值，会按照这个索引编号进行快速查找，直到找到全局对象为止，如果没有找到值，则传递一个undefined。

看下面这个比较经典的例子
{% codeblock lang:javascript %}
var carName = 'byd';

function visitName() {
	console.log(carName);
    var carName = 'bmw';
    console.log(carName);
}
visitName();
{% endcodeblock %}
第一个输出结果：undefined

第二个输出结果：bmw

原因就是它的作用域由静态分析决定，他在visitName中找到carName。所以当遇到第一个console.log(carName)，它并没有到外层的作用域去寻找。

##**全局变量和全局对象**

JavaScript中，在所有函数之外声明的变量为全局变量，而在函数内部声明的变量（通过var关键字）为局部变量。

全局变量在所有的作用域中都是可见的。这样一种特殊的变量，就算不是在JavaScript语言中，也是不受欢迎的。

但是在JavaScript中实在太容易就创建一个全局变量，所以要格外小心。

JavaScript中定义全局变量的方式有三种：

1.在任何函数之外定义一个var语句变量。

2.直接给全局对象添加一个属性，全局对象是全局变量和全局函数的容器。

3.没有使用var关键字声明的变量，它在任何作用域中被使用的时候，都会被当做全局变量（与上面介绍的作用域链相关）。

关于第3种情况：JavaScript有一种特性叫做隐式全局变量，不管一个变量有没有用过，JavaScript解释器反向遍历作用域链来查找该变量的var声明，如果没有找到，解释器则假定该变量是全局变量，如果该变量用于了赋值操作的话，之前如果不存在的话，解释器则会自动创建它。

上面提到了一个全局对象的概念，在Web浏览器中，全局对象的名称是windows。全局对象是预定义的对象，作为JavaScript的全局函数和全局属性的占位符。通过使用全局对象，可以访问所有其他所有预定义的对象（例如，document（浏览器容器下））、函数（例如，parseInt()）和属性。

看下面这个例子，通过三种方式创建全局变量。

{% codeblock lang:javascript %}
var carName = 'byd';

function defineGlobalVariable() {
    window.carPrice = 90000;
    carCount = 2;
}
defineGlobalVariable();

function print() {
    console.log(carName);
    console.log(carPrice);
    console.log(carCount);
}

print();
{% endcodeblock %}

##**对象**

在JavaScript中，只有对象，一切都是对象。

首先要明确一点，JavaScript是面向对象的语言，但是它与C++、Java的面向对象不同。JavaScript不是基于类的面向对象，而是基于原型的面向对象，在JavaScript中没有类的概念。

在基于类的面向对象中，要创建对象，首先要有类。类是对象的抽象描述。类为它们的实例定义了严格不变的结构（属性）和严格不变的行为（方法）。

在基于原型的面向对象中，是动态可变的对象。他们可以很容易地改变（添加，删除，修改）自己的属性。在对对象的属性赋值的时候，如果某些属性不存在，则创建它并且将赋值与它进行初始化，如果它存在，就更新它。

JavaScript里的对象是无类型的，它是属性的容器，是一系列属性的集合，而每一个属性包含名字和值（键值对）。（是不是有点像Map）

属性的名字和属性的值没有限制，它可以是Number、String、Boolean或者另一个对象或者一个函数。

为什么属性可以是一个函数。原因是在JavaScript中，函数也是对象。

##**JavaScripty原生对象**

Object|Number|String|Boolean|Date|Array|RegExp

Function
{% codeblock lang:javascript %}
new Function ([arg1[, arg2[, ... argN]],] functionBody)
{% endcodeblock %}
在JavaScript中，一个函数实际上是一个Function对象。

{% codeblock lang:javascript %}
var fn = new Function("arg1", "arg2", "return arg1 * arg2;");
alert(fn(2, 3));//6
{% endcodeblock %}
##**对象字面量**

在JavaScript中创建对象有三种方式：

1.通过Object创建

2.通过自定义对象构造器创建

3.对象字面量

{% codeblock lang:javascript %}
var audiCar = new Object();
audiCar.name = 'audi';
console.log(audiCar.name);

function bmwCar(name) {
    this.name = name;
}
var myBmwCar = new bmwCar('bmw');
console.log(myBmwCar.name);

//字面量模式
var bydCar = {
    name: 'byd'
};
console.log(bydCar.name);
{% endcodeblock %}

从上面的代码可以看出，字面量写法的一个明显优势是，它的代码更少。

JavaScript中的字面量模式更加简洁、有表现力，而且在定义对象时不容易出错。

如果调用Object()创建对象，解析器需要顺着作用域链从当前作用域开始查找，直到找到全局Object()构造函数为止，如果你在某个域中创建了一个Object()方法就会有问题。

这里有一篇关于JavaScript创建对象的三种方式的介绍，是翻译自JavaScript Pattern。里面有更多关于为什么使用字面量定义对象更好。
https://github.com/TooBug/javascript.patterns/blob/master/chapter3.markdown

##**函数**

因为函数是对象，所以它们可以像任何其他的值一样被使用。函数可以保存在变量、对象和数组（数组也是对象）中。函数可以被当做参数传递给其他函数，函数也可以在返回函数。而且，因为函数是对象，所以函数可以拥有方法。

##**函数字面量（函数表达式）**

定义函数有两种常用方法：

1.函数声明

2.函数字面量（函数表达式）

{% codeblock lang:javascript %}
function getCarName() {
    return "byd";
}
console.log(getCarName());

var getCarPrice = function () {
    return 90000;
};
console.log(getCarPrice());
{% endcodeblock %}

这两种写法相差无几，实际项目中都是可行的，我们可能也没有发现什么错误。但是，他们是有区别的，JavaScript解析器对函数声明和函数表达式并不是一视同仁的。

对于函数声明，JavaScript解析器会在预解析阶段优先读取函数声明的代码，以确保函数能够被引用到；而对于函数表达式，只有在执行到相应的语句时才进行解析。

在实际中，具体表现在：当使用函数声明的形式来定义函数时，可将调用放在函数声明之前，而使用函数表达式，这样做的话会报错。如下例：

{% codeblock lang:javascript %}
runCar();

function runCar() {
    console.log("run car");
}

runTrain();

var runTrain = function () {
    console.log("run train");
};
{% endcodeblock %}