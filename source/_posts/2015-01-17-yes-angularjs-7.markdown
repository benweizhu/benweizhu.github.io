---
layout: post
title: "开始！AngularJS!（七）- 过滤器"
date: 2015-01-17 13:38:17 +0800
comments: true
categories: 
---
过滤器，按照一般的理解是，按照某个标准（或者阈值），将输入的内容中不满足要求的排除，得到最后的满足要求的输出。

但过滤器还有一个更宽泛的理解，就是数据变换处理器。

在AngularJS中，过滤器就是一个数据变换函数，将输入的数据进行处理，得到对应输出结果。

##AngularJS内置过滤器

####格式化过滤器

currency：用两个小数位和一个货币符号来格式化数字   
date：根据指定的数据格式来格式化日期，模型包含的日期可表达为Date对象或者字符串（这时字符串会在格式化前被解析为Date对象）    
number：用参数指定的小数位数量格式化输入   
lowercase和uppercase：转换大小写   
json：主要用于调试，它能打印出漂亮的JavaScript对象   

####数组变换过滤器

limitTo：它将数组收缩到参数指定的长度，可以从集合的头或尾开始保留其中的元素（如果是尾部，则参数必须是负数）    
Filter：提供通用的过滤功能，它非常灵活，支持很多可以精确从集合中选择元素的选项      
orderBy：此排序过滤器根据给定的条件对数组中的元素进行排序     

##过滤器的使用

###在模板中使用

最简单的过滤使用方式: { { 表达式 | 过滤器名 } }
{% codeblock lang:html %}
<body ng-app>
    <div>
        <input type="text" ng-model="words"/>全部转换成大写
        { {words | uppercase} }
    </div>
</body>
{% endcodeblock %}

过滤器可以应用在另外一个过滤器的结果上，叫做“链式”调用: { { 表达式 | 过滤器1 | 过滤器2 | ... } }
{% codeblock lang:html %}
<body ng-app>
    <div>
        <input type="text" ng-model="words"/>
         { {words | uppercase | lowercase} }
    </div>
</body>
{% endcodeblock %}

过滤器可以拥有（多个）参数: { { 表达式 | 过滤器:参数1:参数2:... } }
{% codeblock lang:html %}
<body ng-app>
    <div>
        <input type="text" ng-model="num"/>最多显示到小数点后四位
        { {num | number:4} }
    </div>
</body>
{% endcodeblock %}

内置过滤器是全局的命名函数，在视图中通过管道(|)符号调用，接收用冒号(:)分割的参数。

###控制器和服务中使用过滤器

过滤器又可以链式的使用又可以传递参数，那它的本质是什么？在前面其实已经讲过，是数据的变换函数，本质是函数。

所以，你同样可以在控制器和服务中使用过滤器。注入过滤器有一种简单的方法，在控制器或者服务中添加以“<过滤器名>Filter”为名的依赖。例如，使用"uppercaseFilter"为依赖时，会相应的注入uppercase过滤器。

看下面的例子：在控制器中注入"uppercaseFilter"，然后将输入的小写转换成大写

{% codeblock lang:html %}
<body ng-app="filterApp">
    <div ng-controller="filterController">
        <input type="text" ng-model="value" />
        <input type="button" value="To Upper Case" ng-click="changeToUpperCase()" />
    </div>
</body>
{% endcodeblock %}

{% codeblock lang:js %}
angular.module('filterApp', [])

.controller('filterController', ['uppercaseFilter', '$scope', function (uppercaseFilter, $scope) {
    $scope.changeToUpperCase = function () {
        $scope.value = uppercaseFilter($scope.value);
    };
}]);
{% endcodeblock %}


##自定义过滤器

除了使用AngularJS内置的过滤器，我们还可以定义自己的过滤器

{% codeblock lang:html %}
<body ng-app="filterApp">
    <input type="text" ng-model="value" />{{ value | subString:3}}
</body>
{% endcodeblock %}

{% codeblock lang:js %}
angular.module('filterApp', [])

.filter('subString', function () {
    return function (input, number) {
        return input.slice(0,3);
    };
});
{% endcodeblock %}

##优点和缺点

过滤器的优点就是，它们不需要在作用域上注册函数，而且与规整的函数调用相比，通常具有更简洁的语法

缺点是，过滤器的调用是频繁的，过滤器是函数，当模型发生变换时，使用了过滤器的表达式就是进行一次求值，所以被调用的次数非常的平凡。
