---
layout: post
title: "JavaScript中你需要了解的基本知识（二）"
date: 2014-03-29 22:44
comments: true
categories: 
---
##**表达式**

表达式是一组可以计算出一个数值的有效的代码的集合，是一个单纯的运算过程。

##**函数立刻调用**

当你声明类似function foo(){}或var foo = function(){}函数的时候，通过在后面加个括弧就可以实现函数的执行，例如foo()。

在一个表达式后面加上括号()，该表达式会立即执行，但是在一个语句后面加上括号()，是完全不一样的意思，他的只是分组操作符。

我们只需要用大括弧将代码的代码全部括住就行了，因为JavaScript里括弧()里面不能包含语句，所以在这一点上，解析器在解析function关键字的时候，会将相应的代码解析成function表达式，而不是function声明。

{% codeblock lang:javascript %}
var runImmediately = (function () {
    var value = 100;
    return {
        getValue: function () {
            return value;
        }
    };
}());

console.log(runImmediately.getValue());
{% endcodeblock %}

关于函数立即执行有两种写法，(function f(){}())和(function f(){})()，我个人认为第二种更容易理解，加入括号()，语句变成表达式，然后再加一个括号()，表达式执行。但是我看到大部分例子中都倾向于前一种写法，可能更容易说明他是一个整体。

##**闭包**

我在这里给出多个解释，因为有时候，我感觉从一种解释中不一定能够获得完全理解。

在计算机科学中，闭包（Closure）是词法闭包（Lexical Closure）的简称，是引用了自由变量的**函数**。这个被引用的自由变量将和这个函数一同存在，**即使已经离开了创造它的环境也不例外**。所以，有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的实体。闭包在运行时可以有多个实例，不同的引用环境和相同的函数组合可以产生不同的实例。-------维基百科

闭包可以用来在一个函数与一组“私有”变量之间建立关联关系。在给定函数被多次调用的过程中，这些私有变量能够保持其持久性。变量的作用域仅限于包含它们的函数，因此无法从其它程序代码部分进行访问。-------维基百科

闭包是能够指向独立自由变量的函数。换句话说，定义在闭包中的函数能够“记住”定义它的环境。-------mozilla社区

闭包是一种特殊的对象。它由两部分构成：函数，以及创建该函数的环境。环境由闭包创建时在作用域中的任何局部变量组成。-------mozilla社区

**JavaScript的作用域是让JavaScript拥有闭包特性的根本。**

看下面这段代码：

闭包是一个函数和该函数所引用的自由变量组成。在下面的代码中isBigEnough函数和它所引用的自由变量threshold组成了一个闭包。该闭包传递给了list对象的filter函数。filter函数会反复调用这个函数。

{% codeblock lang:javascript %}
var list = [12, 5, 8, 130, 44];

function getNumberBigThan(list,threshold) {
    return list.filter(function isBigEnough(element) {
        return element >= threshold;
    });
}

var filteredList = getNumberBigThan(list,10);
console.log(filteredList);
{% endcodeblock %}

上面那个例子还不够清楚，再看下面这个：
{% codeblock lang:javascript %}
function f1(n) {　　　　
    function f2() {
        n++;
        return n;
    }
    return f2;
}
var result = f1(999);
console.log(result()); // 1000

var result2 = f1(499);
console.log(result2()); // 500
{% endcodeblock %}
闭包f2由f2函数和它引用的自由变量n组成。在f1运行之后，将f2返回给了result对象。此时运行result就是运行f2，得到结果是1000。说明，闭包f2保留了创建它的环境，变量n并没有在离开函数f1之后被销毁。而且根据创建的环境不同，闭包有不同的实例。
最后看一个例子，来自：http://bonsaiden.github.io/JavaScript-Garden/zh/#function.closures
{% codeblock lang:javascript %}
function Counter(start) {
    var count = start;
    return {
        increment: function() {
            count++;
        },

        get: function() {
            return count;
        }
    }
}

var foo = Counter(4);
foo.increment();
foo.get(); // 5
{% endcodeblock %}
这里，Counter函数返回两个闭包，函数increment和函数get。这两个函数都维持着对外部作用域Counter的引用，因此总可以访问此作用域内定义的变量count。

##**匿名函数立即执行**

我们知道定义函数有两种方式：直接定义和函数表达式（函数字面量）

在定义函数字面量时，我们可以不写函数的名字，通过该在函数对象加上括号来执行该函数。

我们可以在定义该函数字面量时，直接将函数表达式执行。

{% codeblock lang:javascript %}
var funRun = (function () {
	var a = 1;
	var b = 2;
	var c = a + b;
	return c;
}());

console.log(funRun);
{% endcodeblock %}

如果不return任何东西，则funRun为undefined。

通过匿名函数立即执行，可以实现匿名闭包。
{% codeblock lang:javascript %}
var funRun = (function () {
	var a = 1;
	var b = 2;
	return function(){
		return a + b;
	};
}());
console.log(funRun());
{% endcodeblock %}

##**模块化**

JavaScript最讨厌的特性就是它的全局性，随便创建一个方法或者变量，就是全局的，因为它没有像Java或者C++这种语言性上，类方式的模块划分，所以很容易冲突。

那JavaScript为了实现模块化只有借助于实现模式：模块模式。

利用JavaScript的作用域特性，以函数作为模块，利用闭包特性，定义不会污染全局的局部变量和方法（我这描述的有点抽象），直接看怎么写。

通过匿名函数的立即执行返回两个闭包add和sub。这样定义出来的方法add和sub不会暴露到全局中，同时还可以访问在函数内部定义的“私有变量”。
{% codeblock lang:javascript %}
var module = (function () {

    var fact = 10;

    function add(a, b) {
        return (a + b) * fact;
    }

    function sub(a, b) {
        return (a - b) * fact;
    }
    return {
        add: add,
        sub: sub
    };
}());

console.log(module.add(1, 2));
console.log(module.sub(1, 2));
{% endcodeblock %}

关于模块化，还更多深入和高级的知识，例如导入全局变量（要用jQuery的时候）等。
可以查看这边译文。http://www.oschina.net/translate/javascript-module-pattern-in-depth
