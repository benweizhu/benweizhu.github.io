---
layout: post
title: "重温SASS基础"
date: 2015-10-27 14:18:48 +0800
comments: true
categories: 
---
最早因为Bootstrap接触到了LESS，但LESS随着时间，使用的人越来越少，大家都开始转向SASS，至于为什么，原因很多，大家可以去网上搜索，有很多人都对比了LESS和SASS。今天主要是来回顾SASS的基础知识。

CSS很好，但是随着stylesheets的内容增多，变得越来越复杂，也越来越难以维护。此时，CSS预处理器横空出世，SASS就是其中一种。

SASS允许你做许多在CSS中做不了的事情，比如：变量，网状结构（接近DOM结构），mixin，继承等。

###变量

Sass使用$符号定义变量，就和其他编程语言一定，它可以被赋值和重复使用。
{% codeblock lang:css %}
$font-stack:    Helvetica, sans-serif;
$primary-color: #333;

body {
  font: 100% $font-stack;
  color: $primary-color;
}
{% endcodeblock %}

###网状结构

Sass比CSS强大的一点是，可读性好，CSS有后代选择器和权重的知识，它们与DOM结构息息相关，但是CSS本身却没有很好的语法结构来反映这种结构，Sass做到了这一点。

Sass让你的css选择器符合真实的HTML层次结构，当然Sass官方也提示如果过度的使用网状结构，也会导致CSS很难维护。

{% codeblock lang:css %}
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  li { display: inline-block; }

  a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
  }
}
{% endcodeblock %}
上面的例子中，ul和li是在nav结构之中的。

###通过Partial和Import来做模块化

CSS本身也是有import功能的，但是缺点是，它会在页面中单独发送一个Http请求来获取这个import进来的css。

Sass是一个CSS预处理器，它的import是建立在编译过程中，将两个或者多个文件合并，所以机制完全不同。

比如，有_reset.scss和base.scss两个文件，注意，定义Partial的sass文件是以下划线开头的。
{% codeblock lang:css %}
// _reset.scss

html,
body,
ul,
ol {
   margin: 0;
  padding: 0;
}
{% endcodeblock %}
{% codeblock lang:css %}
// base.scss

@import 'reset';

body {
  font: 100% Helvetica, sans-serif;
  background-color: #efefef;
}
{% endcodeblock %}

###Mixins

Mixins是一个非常有用的功能，Mixins允许你将一组css定义在不同的位置重用，而且还可以像函数一样传递变量。

{% codeblock lang:css %}
@mixin border-radius($radius) {
  -webkit-border-radius: $radius;
     -moz-border-radius: $radius;
      -ms-border-radius: $radius;
          border-radius: $radius;
}

.box { @include border-radius(10px); }
{% endcodeblock %}

###Extend/Inheritance继承

通过@extend关键字，你可以让一系列的在某个选择器中css属性，在另一个css选择器中继承，也是重用css定义的一种模式。

{% codeblock lang:css %}
.message {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}

.success {
  @extend .message;
  border-color: green;
}

.error {
  @extend .message;
  border-color: red;
}

.warning {
  @extend .message;
  border-color: yellow;
}
{% endcodeblock %}

###Operators操作符/运算符

Sass可以轻易的进行数学运算，+, -, *, /, %

{% codeblock lang:css %}
.container { width: 100%; }

article[role="main"] {
  float: left;
  width: 600px / 960px * 100%;
}

aside[role="complimentary"] {
  float: right;
  width: 300px / 960px * 100%;
}
{% endcodeblock %}

###Data Types有用的数据类型

SassScript支持七种主要的数据类型

numbers (e.g. 1.2, 13, 10px)   
strings of text, with and without quotes (e.g. "foo", 'bar', baz)   
colors (e.g. blue, #04a3f9, rgba(255, 0, 0, 0.5))   
booleans (e.g. true, false)   
nulls (e.g. null)   
lists of values, separated by spaces or commas (e.g. 1.5em 1em 0 2em, Helvetica, Arial, sans-serif)   
maps from one value to another (e.g. (key1: value1, key2: value2)) 

比如：map
{% codeblock lang:css %}
$map: (key1: value1, key2: value2, key3: value3);
{% endcodeblock %}


参考文献：   
1.SASS官方文档
