---
layout: post
title: "在AngularJS环境下写单元测试：module，inject和$httpBackend"
date: 2015-05-21 17:37:38 +0800
comments: true
categories: 
---

##Angular测试基础：module和inject

先来最简单的样例代码，Controller端代码

{% codeblock lang:javascript %}
angular.module('angularGruntExampleApp')
    .controller('MainCtrl', function ($scope) {
        $scope.awesomeThings = [
            'HTML5 Boilerplate',
            'AngularJS',
            'Karma'
        ];
    });
{% endcodeblock %}

测试代码

{% codeblock lang:javascript %}
describe('Controller: MainCtrl', function () {

    beforeEach(module('angularGruntExampleApp'));

    var MainCtrl, scope;

    beforeEach(inject(function ($controller, $rootScope) {
        scope = $rootScope.$new();
        MainCtrl = $controller('MainCtrl', {
            $scope: scope
        });
    }));

    it('should attach a list of awesomeThings to the scope', function () {
        expect(scope.awesomeThings.length).toBe(3);
    });
});
{% endcodeblock %}

beforeEach()是Jasmine提供的全局方法，在每个测试方法执行之前，调用一次传入的回调函数。

module()方法是由angular-mocks提供，用来加载给定的Angular模块。

$rootScope.$new()创建了一个scope对象，并且在$controller获取MainCtrl时，将scope对象注入。

angular.mock.inject函数接受一个回调函数，回调函数的参数，是需要注入的外部依赖，可以是angular提供的服务，比如，这里的$controller和$rootScope，也可以是你想要测试的自定义服务，如下：

{% codeblock lang:javascript %}
// Defined out reference variable outside
var myService;

// Wrap the parameter in underscores
beforeEach(inject(function(_myService_){
  myService = _myService_;
}));

// Use myService in a series of tests.
it('makes use of myService', function() {
  myService.doStuff();
});
{% endcodeblock %}

在上面的代码中，注入的_myService_，带有下划线，这是inject方法提供的一个特性，因为，我们总是希望在describe这个作用域下定义的变量名可以和真实的Service名字一致，所以inject允许你在注入的参数中加入下划线以区分注入的参数和定义的变量。


##$httpBackend

在单元测试中，我们希望单元测试可以快速的运行，并且没有外部依赖，所以，我们不希望真正的发送HTTP请求到真正的服务器。我们想要的是验证请求已发送，然后将预先定义的请求返回。

$httpBackend就是这样一个提供fake响应的服务器端mock对象实现。通过$httpBackend.expect和$httpBackend.when来制定响应结果和条件。

Flushing HTTP requests

在产品环境中，代码中对http服务器端的请求都是异步，但是在单元测试中，我们不太容易实现异步的测试。httpBackend提供的flush方法允许测试立即flush等待的请求，这样就可以让异步请求同步化，这样就可以在单元测试中同步的测试http请求。

使用$httpBackend非常的简单：

{% codeblock lang:javascript %}
$httpBackend = $injector.get('$httpBackend'); //注入$httpBackend服务

$httpBackend.when('GET', '/customer/1').respond({customerId: '1',name:'benwei'});
scope.getCustomer('1'); // 调用scope的方法发出http请求
$httpBackend.flush(); // 让http请求立刻执行

expect(scope.customer).toEqual({customerId: '1',name:'benwei'});
{% endcodeblock %}


参考资料：    
1.http://docs.ngnice.com/api/ngMock    
1.http://docs.ngnice.com/api/ngMock/service/$httpBackend







