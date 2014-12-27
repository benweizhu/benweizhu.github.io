---
layout: post
title: "开始！AngularJS!（四）- 了解控制器"
date: 2014-12-27 19:14:15 +0800
comments: true
categories: 
---

只要你做过Web开发的，控制器的概念对你来说，实在是太熟悉了。在Spring MVC中，我认为控制器的职责应该只是告诉我，请求从哪里来，带来什么，要到哪里去，带走了什么。

至于要做什么事情，那是服务层的事情。

**在AngularJS中，一般情况下，我们使用控制器做两件事：**

1.初始化$scope对象  
2.为$scope对象添加行为

在前面的章节中，我们已经知道控制器和作用域$scope的关系紧密。当某个控制器通过ng-controller指令被添加到DOM中时，AngularJS会调用该控制器的构造函数来生成一个控制器对象，这样，就创建了一个新的子级作用域。在这个构造函数中，作用域会作为$scope参数注入其中，并允许用户代码访问它。

**初始化$scope对象**

为Angular的$scope对象设置初始状态，通过在$scope对象上添加属性即实现。这些属性就是在视图中展示的视图模型（view model），与此控制器相关的模板中均可以访问到它们。

**创建控制器**

Angular允许我们在全局作用域下(window)定义控制器函数，就像在第二节中介绍的那样，定义一个JavaScript的全局函数函数。但不建议这种方式，推荐在Angular模块 下通过.controller为你的应用创建控制器，如第三节中那样。

{% codeblock lang:html %}
<body ng-app="controllerSampleApp">
    <div ng-controller="sampleController">
        first name <input type="text" ng-model="firstName" /><br/>
        last name<input type="text" ng-model="lastName" /><br/>
        my name is {{fullName()}}.
    </div>
</body>
{% endcodeblock %}

{% codeblock lang:js %}
angular.module("controllerSampleApp", [])

.controller("sampleController", function ($scope) {
    $scope.fullName = function () {
        return $scope.firstName + " " + $scope.lastName;
    };
});
{% endcodeblock %}
在jsfiddle中查看，http://jsfiddle.net/benweizhu/3whwfj3n/


**明白控制器的作用，正确的使用控制器**

就像上面所说的，通常情况下，控制器不应被赋予太多的责任和义务，它只需要负责一个单一视图所需的业务逻辑。最好的保持控制器干净的办法是将那些不属于控制器的逻辑都封装到服务（services）中，然后在控制器中通过依赖注入调用相关服务，这部分会在下一节介绍。

注意，下面的场合千万不要用控制器：（引用官方文档）

**任何形式的DOM操作**：控制器只应该包含业务逻辑。  
DOM操作则属于应用程序的表现层逻辑操作，向来以测试难度之高闻名于业界。把任何表现层的逻辑放到控制器中将会大大增加业务逻辑的测试难度。ng提供数据绑定（数据绑定）来实现自动化的DOM操作。如果需要手动进行DOM操作，那么最好将表现层的逻辑封装在指令中。   
**格式化输入**：使用angular表单控件代替   
**过滤输出**：使用angular过滤器代替   
**在控制器间复用有状态或无状态的代码**：使用angular服务代替   
**管理其它部件的生命周期**（如手动创建service实例）


无论对于何种语言，单一职责一直都是写出漂亮代码的首要原则之一，下一节，我们来了解AngularJS的服务及依赖注入。

参考资料：

1.http://www.ngnice.com/docs/guide/controller