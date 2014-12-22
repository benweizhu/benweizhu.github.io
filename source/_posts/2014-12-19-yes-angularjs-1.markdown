---
layout: post
title: "开始！AngularJS!（一）"
date: 2014-12-19 21:54
comments: true
categories: 
---
首先，看这样一个例子：

{% codeblock lang:html %}
<body ng-app>
    <div ng-controller="textLengthLimitController">
        <textarea ng-model="text"></textarea> <span>{{remaining()}}</span>
        <input type="button" ng-disabled="!hasValidLength()" value="send" />
    </div>
</body>
{% endcodeblock %}

{% codeblock lang:js %}
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

