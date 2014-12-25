---
layout: post
title: "开始！AngularJS!（二）- 入门：视图，模板，模型，控制器和作用域"
date: 2014-12-25 22:40:58 +0800
comments: true
categories: 
---

在AngularJS中，视图是模型在模板中的映射的结果（模型（Model）和视图模型的绑定（View Model））。这意味当模型发生变化时，视图上的绑定内容会对应更新。

**模型**

看第一个的例子，模型与模板：

{% codeblock lang:html %}
<body ng-app>
    <input type="text" ng-model="name"/> 
    { {name} }
</body>
{% endcodeblock %}
在jsfiddle中查看，http://jsfiddle.net/benweizhu/j6b5271r/

在这个例子中，使用了一个HTML本身没有的语法ng-model="name"，它是AngularJS扩展的HTML语法，叫做ng-model指令，还用到了{{name}}，AngularJS的数值表达式。

我们知道在大部分传统模板系统中，对模板的渲染都是线性单向的过程：模型和模板混合在一起产生标记的计算结果。当任何模型发生改变后都需要模板重新计算。

在AngularJS中，当输入框中的内容改变时，对应的模型name会发生改变，AngularJS会**检测到模型发生改变**，重新渲染模板中与该模型绑定的视图部分。

这是AngularJS提供的最基本也是最重要的功能之一，**数据的双向绑定**。

**控制器**

再来第二个例子，模型和控制器：
{% codeblock lang:html %}
<body ng-app>
    <div ng-controller="textController">
         <input type="text" ng-model="name"/>
    </div>
</body>
{% endcodeblock %}

{% codeblock lang:js %}
var textController = function($scope){
    $scope.name="benwei";
}
{% endcodeblock %}
在jsfiddle中查看，http://jsfiddle.net/benweizhu/m5gue0zs/

在这里，用到了AngularJS中的另一个指令ng-controller，在这个Controller的范围内，定义了一个模型name，我们通过模板域模型$scope对象得到controller范围内定义的模型name，并赋值。

在例子中，我们通过$scope对象将模型数据name传递给Controller，从而进行数据的初始化。

虽然，一般情况下，我们不会用这样的方式去建立controller，但是我们可以清晰的看到，在AngularJS中，一个Controller其实就是JavaScript中的一个函数。所以，Controller可以做事情是很多，它并通过$scope对象与模型关联，从而可以实现对模型的赋值以及其他相关逻辑行为的定义。

**作用域**

这里你看到了另一个重要对象$scope，**作用域（Scope）是AngularJS的重要概念，它可以看做一种粘合剂，让模板，模型和控制器工作在一起。AngularJS通过作用域，以及模板中包含的信息，数据模型，控制器，让模型和视图保持位置的分离，内容的同步。任何对模型的改变都会反映到视图，任何视图的改变也会映射到模型中。**

大部分Web应用都是基于MVC的模式，也衍生出许多中MVC的变种（如MVVM（Model View ViewModel）），但AngularJS宣称自己是MVW（Model View Whatever）。

在本节，最重要的是了解视图，模板，模型，控制器之间的关系，以及对作用域对象$scope有一个基本的了解。我们可以在之后一起去了解一下这个Whatever到底是什么含义。

参考资料：

1.《精通AngularJS》 