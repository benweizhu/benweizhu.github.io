---
layout: post
title: "Promise/Q和AngularJS中的resolve"
date: 2015-04-18 10:02:50 +0800
comments: true
categories: 
---
JavaScript是一种单线程的语言。这意味着运行一个有较长处理时间的代码A，会阻塞所有其他代码的执行，直到该代码A执行完。UI元素无响应，动画暂停，其他代码都不能运行。

解决这个问题的办法就是尽量避免同步执行。一种解决办法就是让这种需要较长处理时间的代码后执行。在JavaScript中，所有类似这样的操作都是通过回调函数实现。比如，JavaScript的事件处理器，当事件触发时，才被调用。

异步编程会让代码变得复杂难懂，许多JavaScript的API严重依赖于回调函数，这样就导致了回调的嵌套。比如，下面的ajax调用：
{% codeblock lang:javascript %}
ajax({
    url: url1,
    success: function(data) {
        ajax({
            url: url2,
            data: data,
            success: function() {}
        });
    }
});
{% endcodeblock %}

如果上面的代码，再进行一次回调就更难以阅读了。

为了解决这个问题，许多JavaScript库（jQuery，AngularJS）提供了一个Promise对象来让异步编程模式同步化。看下面的一个例子：

{% codeblock lang:javascript %}
myWebService.get("http://www.example.com")
    .then(function (result) {
        return myDb.add(result);
    })
    .then(function () {
        console.log('data successfully saved');
    }, function (error) {
        console.log('an error occurred while saving:');
        console.dir(error);
    });
{% endcodeblock %}

myWebService.get("http://www.example.com")返回一个promise对象。该promise对象提供一个重要的方法then，并接受一个或者两个回调函数（success callback，failure callback）。

重点是，then方法调用之后，会再次返回一个promise对象，该promise对象是什么，取决于回调函数返回什么，还是什么都不返回。这样，你就可以将多层回调通过链接方式连接起来。这样就可以用同步化的方式来进行异步化编程。

在AngularJS中，通常使用$resource服务来进行restful的Ajax请求，所以对应需要success callback和failure callback。看下面的例子：

{% codeblock lang:javascript %}
var User = $resource('/user/:userId', {userId:'@id'});
var user = User.get({userId:123}, function() {
	user.abc = true;
});
{% endcodeblock %}

Calling these methods invoke an $http with the specified http method, destination and parameters. When the data is returned from the server then the object is an instance of the resource class. 

调用resource上的get方法，会触发一个$http请求，当请求数据从服务器端回来时，它是resource类的是一个实例。该实例有一个很重要的属性$promise，该属性返回给你对应的promise对象。于是上面那段通过回调方式编写的代码，就可以用同步的方式编写，如下。

{% codeblock lang:javascript %}
var User = $resource('/user/:userId', {userId:'@id'});
var userPromise = User.get({userId:123}).$promise;
userPromise.then(function() {
	user.abc = true;
});
{% endcodeblock %}

理解这部分后，我们进行下一个话题：Angular的resolve。

在AngularJS路由进入一个页面时，对应的Controller可能会进行一些异步调用，比如，去服务器端获取一些需要在页面现实的数据。这种异步调用可能会花费较长的时间，这样就很可能导致页面抖动，页面在渲染完成后，现实数据还没有返回，甚至页面显示不正确。

为了解决这个问题，angular的路由提供了一个重要的属性resolve，允许在进入页面之前，进行一些必要的数据准备，然后将准备好的数据注入到Controller中，如下。

{% codeblock lang:javascript %}
.when('/', {
    templateUrl: 'views/main.html',
    controller: 'mainCtrl',
    resolve: {
    	user: function() {
    		return {name:'benweizhu'};
    	}
	}
})
{% endcodeblock %}

resolve提供了一个重要的特性，如果返回的是一个promise对象，那么路由会等到这个promise对象resolve（成功或者失败的）后，再初始化Controller。这样，我们就可以在Controller初始化前，进行一些异步调用，比如resource的Ajax请求，这样就可以防止页面都会，或者渲染错误。

{% codeblock lang:javascript %}
.when('/', {
    templateUrl: 'views/main.html',
    controller: 'mainCtrl',
    resolve: {
    	user: function(User) {
    		return User.get({userId:123}).$promise;
    	}
	}
})
{% endcodeblock %}

参考资料：    
1.AngularJS官方网站   
2.http://andyshora.com/promises-angularjs-explained-as-cartoon.html     
3.https://thinkster.io/a-better-way-to-learn-angularjs/promises     
4.https://msdn.microsoft.com/en-us/library/windows/apps/hh700330.aspx   