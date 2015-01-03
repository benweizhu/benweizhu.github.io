---
layout: post
title: "开始！AngularJS!（三）- 深入作用域"
date: 2014-12-26 22:58:34 +0800
comments: true
categories: AngularJS
---
###什么是作用域？它做了些什么事情？

**作用域是一个存储应用数据模型的对象**   
没错，它是概念也是一个对象

**作用域为表达式提供了一个执行上下文**   
作用域为表达式提供执行环境，比如像表达式{ {name} }，必须在一个拥有该属性的作用域中才能执行

**作用域的层级结构对应于DOM树结构**   
一个应用可以有多个作用域，当新作用域被创建的时候，他们会被当成子作用域添加到父作用域下，这使得作用域会变成一个和相应DOM结构一个的树状结构。   

**作用域可以监听表达式的变化并传播事件**    
作用域提供了($watch)方法监听数据模型的变化  
作用域提供了($apply)方法把不是由Angular触发的数据模型的改变引入Angular的控制范围内（如控制器，服务，及Angular事件处理器等）

**作用域是Web应用的控制器和视图之间的粘结剂**  
控制器和指令都持有作用域的引用，于是我们可以这样理解它们之间的传递关系：  
控制器 --> 作用域 --> 视图（DOM）  
指令 --> 作用域 --> 视图（DOM）

###何时会产生一个作用域？

于是，你肯定会想要知道，一个作用域的范围是什么？即何时会产生一个作用域？

1.创建控制器时会产生作用域（这个很明显，控制器的构造函数会传入$scope对象）   
2.某些指令也会产生作用域   

###作用域层级

一般情况下，当新的作用域被创建时，它是以子级的形式被创建，嵌入在当前父级作用域，这样就形成了与其所关联的DOM树相对应的一个作用域的树结构。

###继承

作用域中定义的属性对于所有子作用域都是可见的，只要子作用域没有定义同名属性。

换句话说，当AngularJS执行表达式{ {name} }，它首先会在与当前节点相关的作用域中查找叫做name的属性。如果没找到，则继续向上层作用域搜索，直到根作用域$rootScope。在Javascript中，这被称为原型类型的继承，子作用域以原型的形式继承自父作用域。

上面引入了一个新的概念和对象根作用域$rootScope，正如上面所介绍的，作用域的结构对应于DOM结构，那么最顶层，就和DOM树有根节点一样，每个Angular应用有且仅有一个 根作用域$rootScope。

我们来看个一个具体的例子，贯穿一下上面所提到的概念：


{% codeblock lang:html %}
<body ng-app="listApp">
    <div ng-controller="listController">
        my name is <input type="text" ng-model="name"/>
        <ul ng-repeat="friend in friends">
            <li>Hi { {friend} }, { {introduce()} }</li>
        </ul>
    </div>
</body>
{% endcodeblock %}

{% codeblock lang:js %}
angular.module("listApp", [])

.controller("listController", function ($scope) {
    $scope.friends = ["Lily","Kate","Mike"];
    $scope.introduce = function() {
        return "this is " + $scope.name + ".";
    };
});
{% endcodeblock %}
在jsfiddle中查看，http://jsfiddle.net/benweizhu/wLufL8p1/1/

在这个例子中，你可能会看到和上一节定义控制器的方式不同。没错，上一节已经说过，以直接定义函数的方式定义控制器并不是常用的方式，只是为了让你知道控制器的本质是函数。

这里你不用关心定义控制器的方式，之后的章节会讲解。你需要知道的是，这里定义了一个app叫做listApp，里面定义了一个控制器listController。在这个控制器中定义了两个模型(Model)对象name，数组friends和一个函数introduce()，在模板(HTML)上使用了ng-repeat指令来迭代数组friends里面的结果，通过表达式来解析遍历结果和调用函数。

在控制器中初始化了friends，映射了这样的传递关系：控制器 --> 作用域 --> 视图（DOM）

有一点要注意的，指令ng-repeat在执行时，会在每一次遍历都创建一个名字是friend的变量，但是你从结果中看到，页面上friend的值并没有重复。那是因为，AngularJS将每个friend都放在了不同的作用域中，这就是上面提到的，某些指令会产生作用域。

于是乎，在这个app中，就不止一个作用域了，有四个作用域，其中三个作用域是同级的，它们都嵌入在父作用域listController。根据继承的关系，父作用域中的属性对子作用域是可见的，所以{ {introduce()} }表达式可以调用函数introduce。


如果你希望更深入的了解作用域的层级和产生，可以运用Chrome的插件Batarang，它可以帮助你分析AngularJS层级结构。


参考资料：


1.《精通AngularJS》  
2.http://www.ngnice.com/docs/guide/scope   
3.http://www.angularjs.cn/A00y



