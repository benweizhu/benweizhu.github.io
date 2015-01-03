---
layout: post
title: "开始！AngularJS!（五）- 模块化"
date: 2015-01-03 10:47:01 +0800
comments: true
categories: 
---
我们都知道JavaScript很容易就写出全局函数，所以无论是用jQuery还是纯JavaScript，我们都会使用模块化的策略避免写出来的函数污染全局。

而模块化的办法一般都是利用函数对象立即执行。

{% codeblock lang:js %}
var feature = (function() {
 
    // Private variables and functions
    var privateThing = "secret";
    var publicThing = "not secret";
 
    var changePrivateThing = function() {
        privateThing = "super secret";
    };
 
    var sayPrivateThing = function() {
        console.log( privateThing );
        changePrivateThing();
    };
 
    // Public API
    return {
        publicThing: publicThing,
        sayPrivateThing: sayPrivateThing
    };
})();
 
feature.publicThing; // "not secret"
 
// Logs "secret" and changes the value of privateThing
feature.sayPrivateThing();
{% endcodeblock %}

###模块与ng-app指令

Angular也有模块的概念，但是它不只是为了解决全局函数的问题。

看一个最简单的例子：

{% codeblock lang:html %}
<body ng-app="textApp">
    <div ng-controller="textController">
        <input type="text" ng-model="name" />
    </div>
</body>
{% endcodeblock %}

{% codeblock lang:js %}
angular.module('textApp', [])

    .controller('textController', function ($scope) {
        $scope.name = 'Benwei';
    });
{% endcodeblock %}
在jsfiddle中查看，http://jsfiddle.net/benweizhu/73kn66kd/


在前面我们已经无数次的看到ng-app指令。它的作用是自动启动一个AngularJS应用，ng-app指令一般指派在应用的根元素上，比如，body或者html标签。在每一个HTML文档中，只能有一个AngularJS应用可以被自动启动，在HTML文档中第一个被找到定义在根元素上的ng-app指令将会作为自动启动的应用。

可以理解它为AngularJS应用启动的触发点。

那我们在js代码中定义的模块和ng-app有什么关系呢？很明显，它是告诉AngularJS应用在启动时加载指定的模块，假设这里ng-app只是放一个纯标签，而不给它赋值。那么它就不知道这里该加载什么模块，于是，它也不认识在模块中定义的textController控制器。

但是，赋值与否和启动一个AngularJS的应用无关：

{% codeblock lang:html %}
<body ng-app>
     <input type="text" ng-model="name" />{{name}}
</body>
{% endcodeblock %}

这样也是可以启动AngularJS应用，并实现name模型的绑定。

###模块化的目的

模块化除了像Java中分包，分类那样来划分职责，还带来什么好处呢？（以下摘自官方文档）

整个过程是声明式的，更容易理解  
在单元测试中，没有必要加载所有模块，这样有利于单元测试的代码书写  
在场景测试中，额外的模块可以被加载进来，进而重写一些配置，这样有助于实现应用的端到端的测试  
第三方代码可以很容易被打包成可重用的模块  
模块可以用任意顺序或并行顺序加载（得益于模块执行的延迟性）


###子模块

对于大型应用，根据不同职责，建议把它像这样分成多个模块（参考官方文档）：

服务模块  
指令模块  
过滤器模块  
一个应用的模块，依赖于上述的三个模块，而且包含应用的初始化及启动代码

**模块依赖**

模块声明时可以列出它所需要依赖的其它模块。一个模块依赖某模块，意味着这个被依赖的模块需要在模块被加载之前加载完毕。更具体些，假设模块A依赖于模块B，那么模块A的配置代码块的执行，必须发生在模块B的配置代码块之后；模块A的执行代码块亦同理，也在模块B的执行代码块之后被执行。每个模块只能被加载一次，即使有多个别的模块依赖它。


**创建模块，获取模块，如何实现模块的依赖**

使用 angular.module('myModule', []) 将创建名为 myModule 的模块并重写已有的同名模块。而使用 angular.module('myModule') 则只会获取已有的模块实例。

你会注意到，创建和获取的区别，在于module函数的第二参数，该参数的作用是定义创建模块时的所依赖的模块（子模块）。

这样做的好处是，不同的服务或者一组服务被放置在不同的可重用模块，那么应用模块只需要声明应用所需要的全部依赖模块即可。

来看一个模块依赖的例子：

{% codeblock lang:js %}
angular.module('restApp', ['ngResource'])
    .factory('personRest', function ($resource) {
        return $resource('/app/person');
    });
    
angular.module('myApp', ['restApp'])
    .controller('parentsController', function ($scope, personRest) {
        personRest.get({}, function (data) {
            $scope.person = data.person;
        });
    });
{% endcodeblock %}

我需要写一个Rest的客户端模块restApp，需要使用到AngularJS提供的$resource服务，那么首先我需要引入AngularJS的ngResource服务模块，最后在我的应用模块，引入我自己定义的restApp模块然后，我就可以注入我自己定义personRest服务，继而使用它，这是一个典型的模块依赖例子。

本来上一节，说好了要讲解依赖注入，但是我发现先介绍模块和模块化更好，一个是尽快帮助学习写更规范的AngularJS应用，二个也会有助于之后对理解依赖注入。下一节，我们来讨论AngularJS中的依赖注入。





