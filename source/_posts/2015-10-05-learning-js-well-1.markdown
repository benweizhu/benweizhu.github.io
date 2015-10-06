---
layout: post
title: "学好JavaScript（一）- 被我忽视的东西(1)"
date: 2015-10-05 10:06:40 +0800
comments: true
categories: 
---
###JavaScript普通对象

JavaScript中除了数字、字符串、布尔值、null和undefined之外的就是对象了。对象是属性的集合，每个属性都由“名/值对”构成，值可以是原始值，比如数字、字符串，也可以是对象。普通的JavaScript对象就是“命名值”的无序集合。（JavaScript同样定义了一种特殊对象-数组，表示带编号的值的有序集合）

###可选的分号

JavaScript使用分号将语句分开。但如果语句各自独占一行，通常可以省略语句之间的分号。是否省略分号是一种风格，对于不知道什么时候可以省略分号的开发，在任何时候都采用分号分割是一种常见风格。

###布尔值

任意JavaScript的值都可以转换为布尔值，下面这些值会被转换为false，   
undefined    
null    
0    
-0   
NaN   
""   
所有其他值，包括所有对象（数组）都会转换成true，false和上面6个可以转换成false的值有时称作“假值”，其他值称作“真值”。JavaScript期望使用一个布尔值的时候，假值会被当做false，真值会被当做true。

###null和undefined

null是JavaScript语言的关键字，它表示一个特殊值，常用来描述“空值”。对null执行typeof运算，结果返回字符串“object”，也就是说，可以将null认为是一个特殊的对象值，含义是“非对象”。但实际上，通常认为null是它自由类型的唯一一个成员，它可以表示数字，字符串和对象是“无值”的。

JavaScript还有第二值来表示值的空缺。用未定义的值表示更深层次的“空值”。它是变量的一种取值，表示变量没有初始化，如果要查询对象属性或者数组元素的值时，返回undefined则说明这个属性或者元素不存在。如果函数没有任何返回值，则返回undefined。

undefined是预定义的全局变量，它和null不一样，它不是关键字，它的值就是“未定义”。

尽管null和undefined是不同的，但它们都表示“值的空缺”，两者往往可以互换。判断相等运算符“==”认为两者是相等的，所以需要使用严格相等运算符“===”来区分它们。

如果你想将它们赋值给变量或者属性，或将它们作为参数传入函数，最佳的选择是使用null。

###全局对象

全局对象的属性是全局定义的符号，JavaScript程序可以直接使用。当JavaScript解释器启动时（或者任何Web浏览器加载新页面的时候），它将创建一个新的全局对象，并给它一组定义的初始属性：全局属性，undefined，全局函数，isNaN()，构造函数，Date()，全局对象，Math。

全局对象的初始属性并不是保留字，但它们应该当做保留字来对待。

在代码的最顶级——不在任何函数内的JavaScript代码——可以使用JavaScript关键字this来引用全局对象：
{% codeblock lang:JavaScript %}
var global = this; // 定义一个引用全局对象的全局变量
{% endcodeblock %}
在客户端JavaScript中，在其表示的浏览器窗口中的所有JavaScript代码中，Window对象充当了全局对象。这个全局对象Window对象有一个属性window引用其自身，它可以代替this来引用全局对象。

当初次创建的时候，全局对象定义了JavaScript中所有的预定义全局值。这个特殊对象同样包含了为程序定义的全局值。如果代码声明了一个全局变量，这个全局变量就是对象的一个属性。

{% codeblock lang:JavaScript %}
var a = 'a';
window.a; //输出"a"
{% endcodeblock %}

###包装对象

我们看到字符串也同样具有属性和方法，字符串既然不是对象，为什么它会有属性呢？    
只要引用了字符串的属性，JavaScript就会将字符串通过调用new String("some string")的方法转换成对象。

同字符串一样，数字和布尔值也具有各自的方法：通过Number()和Boolean()构造函数创建一个临时对象。

JavaScript会在必要时将包装对象转换成原始值。“==”等于运算符将原始值和其他包装对象视为相等，但“===”全等运算符将它们视为不等。

###相等和不相等运算符

“==”和“===”运算符用于比较

详细内容待输入

###in运算符

in运算符希望它的左操作数是一个字符串或可以转换为字符串，希望它的右操作数是一个对象。如果右侧的对象拥有一个名为做操作数值的属性名，那么表达式返回true。
{% codeblock lang:JavaScript %}
a = {text:"hi"}
"text" in a
{% endcodeblock %}

for/in语句也使用for关键字，但它是和常规的for循环完全不同的一类循环。for/in循环语句的语法如下：
{% codeblock lang:JavaScript %}
for (variable in object)
	statement
{% endcodeblock %}

variable通常是一个变量名，也可以是一个可以产生左值的表达式或者一个通过var语句声明的变量，总之必须是一个适用于赋值表达式左侧的值。object是一个表达式，这表达式的计算结果是一个对象。

for/in循环则是用来更方便地遍历对象属性成员：
{% codeblock lang:JavaScript %}
for (var p in o)
	console.log(o[p]);
{% endcodeblock %}
在执行for/in语句的过程中，JavaScript解释器首先计算object表达式。如果表达式为null或者undefined，JavaScript解释器将会跳出循环并执行后续的代码。如果表达式等于一个原始值，这个原始值将会转换为与之对应的包装对象。否则，expression本身已经是对象了。JavaScript会依次枚举对象的属性来执行循环。

###"use strict"
"use strict"是ECMAScript5引入的一条指令。指令不是语句。

它不含任何语言的关键，指令仅仅是一个包含一个特殊字符串直接量的表达式，对于那些没有实现ECMAScript5的JavaScript解释器来说，它只是一条没有副作用的表达式语句，它什么也没做。

它只能出现在脚本代码的开始或者函数体的开始，任何实体语句之前。但他不必一定要出现在脚本的首行或者函数体首行，因为“use strict”指令之后或者之前都可能有其他字符串直接量表达式语句，并且JavaScript的具体实现可能将它们解析为解释器自有的指令。在脚本或者函数体第一条常规语句之后字符串直接量表达式语句只当做普通表达式语句对待；它们不会当做指令解析，它们也没有任何副作用。

使用“use strict”指令的目的是说明（脚本或函数中）后续的代码将会解析为严格代码。

严格代码以严格模式执行。ECMAScript5中的严格模式是该语言的一个受限制的子集，它修正了语言的重要缺陷，并提供健壮的查错功能和增强的安全机制。



