---
layout: post
title: "学好JavaScript（二）- 被我忽视的东西(2)对象"
date: 2015-10-06 10:02:04 +0800
comments: true
categories: 
---
原型

每个JavaScript对象（null除外）都和另一个对象相关联。“另一个”对象就是我们熟知的原型，每一个对象都从原型继承属性。

所有通过对象直接量创建的对象都具有同一个原型对象，并可以通过JavaScript代码Object.prototype获得对原型对象的引用。通过关键字new和构造函数调用创建的对象的原型就是构造函数的prototype属性的值。因此，同使用{}创建对象一样，通过new Object()创建的对象也继承自Object.prototype。同样，通过new Array()创建的对象的原型就是Array.prototype，通过new Date()创建的对象的原型就是Date.prototype。

没有原型的对象为数不多，Object.prototype就是其中之一。他不继承任何属性。其他原型对象都是普通对象，普通对象也有原型。所有的内置的构造函数都具有一个继承自Object.prototype的原型。例如，Date.prototype的属性继承自Object.prototype，因此由new Date()创建的Date对象的属性同时继承自Date.prototype和Object.prototype。这就是所谓的“原型链”。

关联数组

object.property
object["property"]

第二种语法使用方括号和一个字符串，看起来更像数组，只是这个数组是通过字符串索引而不是数字索引，这种数组就是所说的关联数组。

在C++、Java等强类型语言中，对象只能拥有固定数目的属性，并且这些属性名称必须提前定义好。由于JavaScript是弱类型语言，因此不必遵循这个条规定，在任何对象中程序都可以创建任意数量的属性。但当通过点运算符访问对象的属性时，属性名用一个标识符来表示。标示符必须直接出现在JavaScript程序中，它们不是数据类型，因此程序无法修改它们。

反过来讲，当通过[]来访问对象的属性时，属性名通过字符串来表示。字符串是JavaScript的数据类型，在程序运行时可以修改和创建它们。

for(i = 0; i < 4; i++)
	addr += customer["address" + i];


