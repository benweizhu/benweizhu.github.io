---
layout: post
title: "开始！AngularJS!（六）- 依赖注入"
date: 2015-01-04 13:24:06 +0800
comments: true
categories: 
---
不说废话，开始学习AngularJS的依赖注入，如果你对什么是依赖注入还不明白的话，可以看看Martin Fowler的一篇关于依赖注入的文章
[Inversion of Control Containers and the Dependency Injection pattern](http://martinfowler.com/articles/injection.html "Inversion of Control Containers and the Dependency Injection pattern")，这里也有[译文](http://benweizhu.github.io/blog/2013/09/22/inversion-of-control-containers-and-the-dependency-injection-pattern-translate/ "译文")。

依赖注入就是，在需要此依赖的地方等待被依赖对象注入（传入）进来，而不是通过new关键字去构造，或者去查找某个依赖。

看一个AngularJS依赖注入的例子：

{% codeblock lang:html %}
<body ng-app="diApp">
    <div ng-controller="diController">
        <input type="text" ng-model="alertValue" />
        <input type="button" ng-click="alertMe()" value="clickMe" />
    </div>
</body>
{% endcodeblock %}

{% codeblock lang:js %}
angular.module('diApp', [])

.controller('diController', function ($scope, $window) {
    $scope.alertMe = function () {
        $window.alert($scope.alertValue);
    };
});
{% endcodeblock %}
在jsfiddle中查看，http://jsfiddle.net/n5sknpe9/


在定义控制器diController时，在构造函数中传入一个对象$scope和一个服务$window。$scope将控制器作用域中的模型alertValue传递进来，而$window则提供alert()方法。

用起来看着确实很简单，那么AngularJS是怎么做到的呢？看下面一个例子。

{% codeblock lang:html %}
<body ng-app="diApp">
    <div ng-controller="diController">
        { {value} }
    </div>
</body>
{% endcodeblock %}

{% codeblock lang:js %}
angular.module('diApp', [])

.factory('diService', function () {
    return {
        getValue: function(){
            return 1 + 1;
        }
    };
})

.controller('diController', function($scope, diService){
    $scope.value = diService.getValue();
});
{% endcodeblock %}

每个Angular应用都有一个injector对象。这个injector是一个服务定位器，负责创建和查找依赖，当你的app的某处声明需要用到某个依赖时，Angular 会调用这个依赖注入器去查找或是创建你所需要的依赖，然后返回来给你用。

为了看得更清楚，手动的去调injector来获取该依赖，就下面这样做：

{% codeblock lang:js %}
angular.module('diApp', [])

.factory('diService', function () {
    return {
        getValue: function(){
            return 1 + 1;
        }
    };
});

var injector = angular.injector(['diApp', 'ng']);

var diService = injector.get('diService');

console.log(diService.getValue());
{% endcodeblock %}

而当你在声明说需要的依赖时，AngularJS帮你做了上面这件事情。

依赖注解(Annotation)的方式

那么$inject服务，怎么知道应该将什么依赖注入给你呢？

如果从$inject服务的内部来看，有下面三种方式：

{% codeblock lang:js %}
// inferred (only works if code not minified/obfuscated)
$injector.invoke(function(serviceA){});

// annotated
function explicit(serviceA) {};
explicit.$inject = ['serviceA'];
$injector.invoke(explicit);

// inline
$injector.invoke(['serviceA', function(serviceA){}]);
{% endcodeblock %}

那么，换成在注册控制器或者服务时，对应也是三种方式：

直接指定，最简单的获取依赖的方法是让你的函数的参数名直接使用依赖名，也就是之前的那些例子一样。

{% codeblock lang:js %}
function myController($scope, myService) {
    ...
}
{% endcodeblock %}

$inject服务的官方例子也说了，这种用法只适合于js不需要压缩和混乱的情况下。

而为了让在压缩版的js代码能中重命名过的参数名能够正确地注入相关的依赖服务。函数需要通过$inject属性进行标注，这个属性是一个存放需要注入的服务的数组。

{% codeblock lang:js %}
var myController = function(renamed$scope, renamedMyService) {
    ...
}
myController['$inject'] = ['$scope', 'myService'];
{% endcodeblock %}

但是，这种方式是不是看的很累赘。

还有最后一种方法，内联（inline）

{% codeblock lang:js %}
angular.module('myApp',[]).controller('myController',['$scope','myService',function($scope, myService) {
    ...
}]);
{% endcodeblock %}

这样是不是好多了，这是AngularJS官方推荐的方法，我之前写的例子，都不算是最佳实践，我们应该参考这种方式去实现自己控制器和服务。

总算，将依赖注入的部分介绍完了，下一节，一起来了解AngularJS为View Model（视图模型）提供的强大的过滤器功能。

参考资料：   
1.http://www.ngnice.com/docs/guide/di    
2.http://www.ngnice.com/docs/api/auto/service/$injector    
3.http://www.ngnice.com/docs/tutorial/step_05  
