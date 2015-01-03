---
layout: post
title: "开始！AngularJS!（一）- 刚刚开始"
date: 2014-12-19 21:54
comments: true
categories: AngularJS
---
看这样一个例子：

{% codeblock lang:html %}
<body ng-app>
    <div ng-controller="textLengthLimitController">
        <textarea ng-model="text"></textarea> <span>{{remaining()}}</span>
        <input type="button" ng-disabled="!hasValidLength()" value="send" />
    </div>
</body>
{% endcodeblock %}

{% codeblock lang:javascript %}
var MAX_LENGTH = 10;

function textLengthLimitController($scope) {

    $scope.remaining = function () {
        return MAX_LENGTH - $scope.text.length;
    };

    $scope.hasValidLength = function () {
        return $scope.remaining() >= 0;
    };

}
{% endcodeblock %}

代码的功能是告诉用户还可输入多少字符，当字符超过限制，将发送按钮灰掉。在jsFiddle里运行该代码，http://jsfiddle.net/benweizhu/cbcn995m/

这里做了几件事情：  
1.定义页面（HTML页面）  
2.定义期待行为（remaining()和ng-disabled）  
3.定义行为发生的逻辑（MAX_LENGTH - $scope.text.length和$scope.remaining() >= 0）  

剩下最复杂的事情：对DOM元素的操作，交给了AngularJS来做。

**引用AngularJS官方文档对AngularJS的介绍：**

“AngularJS是一个开发动态Web应用的框架。它让你可以使用HTML作为模板语言并且可以通过扩展的HTML语法来使应用组件更加清晰和简洁。它的创新之处在于，通过数据绑定和依赖注入减少了大量代码，而这些都在浏览器端通过JavaScript实现，能够和任何服务器端技术完美结合。”

就像文档介绍的那样，HTML本身是静态文档，是很好的声明式语言，但对于构建动态的web应用，却无能为力。

针对这类问题，通常的解决办法是：通过JavaScript对HTML的DOM结构进行修改。比如，jQuery。

而AngularJS另辟蹊径，通过扩展HTML的语法来拉近静态Web和动态Web之间的距离。

**"Angular是建立在这样的信念之上的：即声明式的代码用在构建用户界面和组装软件组件时更好，而命令式的代码更擅长展现业务逻辑。"**

AngularJS将应用逻辑与DOM操作解耦，让开发人员不用去担心去前端视图模型（View Model）改变，从而大大提高代码的可测试性。

AngularJS试图去实现一个完整的前端解决方案：

1.构建一个CRUD应用时可能用到的所有技术：数据绑定、基本模板指令、表单验证、路由、深度链接、组件重用、依赖注入  
2.可测试性：单元测试、端到端测试、模拟对象（mocks）、测试工具

在一个以用户体验，多平台（Web和移动）为核心的现代应用时代，单页应用以前端+API成为了Web应用的开发的趋势：

1.对于内容的改动不需要加载整个页面   
2.对服务器压力很小，消耗更少的带宽，与面向服务的架构更好地结合  
3.多平台使用相同的API，减少后台逻辑的重复开发

AngularJS给现代Web应用开发带来曙光，让我们一起认认真真学习AngularJS。

参考资料：

1.《精通AngularJS》  
2.http://www.ngnice.com/docs/guide/introduction   
3.http://www.csdn.net/article/2012-12-10/2812658-Single-Page-Applications


