---
layout: post
title: "开始！AngularJS!（八）- 路由"
date: 2015-01-23 22:18:10 +0800
comments: true
categories: 
---
传统的Web应用，以Java栈为例，无论是纯Servlet还是MVC框架，比如，SpringMVC，Struts。在页面上的导航都比较简单，只需要在浏览器地址栏输入URL，一个URL用来指向服务器上一个单一的物理资源（页面文件）。当页面加载后，就可以点击链接跳转到其他资源，或者使用前进后退按钮跳转到已访问的页面。

通过修改DOM，动态生成页面的Web应用改变了这一现状，因为对页面内容的改变是直接对页面DOM元素的修改，而不是向服务器发出请求。于是，前进后退按钮在这种情况受到了影响。

##URL

那么，对于单页应用，如何能够修改浏览器的URL，让浏览器可以前进后退，但是又不会向服务器发出请求？

###HashBang URL

通过修改URL定制中#符号后面的部分，而不会触发当前页面重新加载。比如， http://localhost:3000/#/user/123 ，浏览器会选取URL中#号后面的差异部分来提供前进后退。

###另一种方式，HTML5的historyAPI

这里首先要说明浏览器的前进和后退都依赖于浏览器的history堆栈（window.history对象），保证浏览器history堆栈的记录正确，前进后退的按钮就可以正常工作。HTML5中的history正好提供了方法来将URL推送到浏览器的history堆栈。然后只要监听window.onpopstate事件就可以修改应用的状态。

但是当直接在服务器上输入URL时，还是会向服务器发送请求，这个时候还需要在服务器端做些事情，让服务器始终返回应用首页。所以这种方式会相对麻烦一些。

当看到#号，你可能会想到另一个东西，HTML中的锚点，定位页面的位置。在HashBang方式下，需要借助AngularJS的一个服务$anchorScroll以及两个#来实现锚点。比如 http://localhost:3000/#/user/123#name

##使用AngularJS的路由服务

AngularJS内置了$route服务来处理Web应用的路由，并通过ng-view指令来显示匹配路由的内容，看下面的例子：

{% codeblock lang:html %}
<body ng-app="routeApp">
    <a href="#/admin">admin</a>
    <div ng-view></div>
</body>
{% endcodeblock %}

被插入的模板，动态渲染部分
{% codeblock lang:html %}
<div ng-controller="adminController">
   I am admin.
</div>
{% endcodeblock %}

{% codeblock lang:javascript %}
angular.module('routeApp', ['ngRoute'])
.config(['$routeProvider', function($routeProvider) {
    $routeProvider.when('/admin', {
        templateUrl: 'pages/admin/admin.html'
    });
}])
.controller('adminController', ['$scope',
        function ($scope) {}
]);
{% endcodeblock %}

这里引入了.config方法，需要穿插介绍一点知识：模块的生命周期。AngularJS将模块的生命周期分为两个阶段，配置阶段和运行阶段。其中模块的.config方法可以用来注册一些需要在模块加载时候执行的动作或者进行的配置，比如这里的路由配置。

AngularJS的路由服务在angular-route.js中，所以首先需要引入它，然后引入子模块ngRoute，在config方法中注入$routeProvider服务，然后剩下的就很清楚了，和Java技术栈中道理一样，URL和资源文件的Mapping。

ng-view指令可以通过当前匹配的路由找到要显示的内容。

###URL查询参数

http://localhost:3000/#/user?id=123 或者 http://localhost:3000/#/user/123

几乎所有Web应用，遇到这样的URL是太平常的事情了，那么这种动态的URL应该怎么匹配呢？在AngularJS中，URL中任何以冒号（:）开头的字符串都会作为通配符。无论是这里的$routeProvider.when(/user/id=:userid)还是之后会介绍的与后端restful api通信的$resource服务在定义URL的时候。

那么如何在controller中获取该参数呢？$routeParams服务。我们可以在controller中注入该服务，看下面的例子。

{% codeblock lang:javascript %}
angular.module('routeApp', ['ngRoute'])
.config(['$routeProvider', function($routeProvider) {
    $routeProvider.when('/user/:userid', {
        templateUrl: 'pages/user/user.html'
    });
}])
.controller('userController', ['$scope, '$routeParams',
       function ($scope, $routeParams) {
        	$scope.userid = $routeParams.userid;
    	}
]);
{% endcodeblock %}

###多个控制器重用模块

在上面的例子中，url对应的页面文件是这样的：

{% codeblock lang:html %}
<div ng-controller="adminController">
   I am admin.
</div>
{% endcodeblock %}

模板的作用域固定为adminController的，如果不同路由下的其他controller也想用该模块，那么就得重新建一个新的html文件。

还有一种办法，就是将controller的声明放在路由的定义中。

{% codeblock lang:html %}
<div>
   I am admin.
</div>
{% endcodeblock %}

{% codeblock lang:javascript %}
angular.module('routeApp', ['ngRoute'])
.config(['$routeProvider', function($routeProvider) {
    $routeProvider
    .when('/admin', {
        templateUrl: 'pages/admin/adminOrSuperUser.html',
        controller: 'adminController'
    })
    .when('/superUser', {
        templateUrl: 'pages/admin/adminOrSuperUser.html',
        controller: 'superUserController'
    });
}])
.controller('adminController', ['$scope',
        function ($scope) {}
])
.controller('superUserController', ['$scope',
        function ($scope) {}
]);
{% endcodeblock %}

这样多个控制器就可以使用相同的模板。

###指定默认路由

就和编程语言中的switch一样，在$routeProvider还提供了otherwise方法来设置默认路由，很明显它只能是一个，一般的做法是通过redirectTo属性，跳转到一个已有的路由上。

{% codeblock lang:javascript %}
config(['$routeProvider', function($routeProvider) {
  $routeProvider.otherwise({redirectTo: '/home'});
}]);
{% endcodeblock %}

下一节，学习通过$Resource与后端restful的API通信。

参考资料：

1. 精通AngularJS





