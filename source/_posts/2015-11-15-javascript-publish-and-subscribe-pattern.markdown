---
layout: post
title: "JavaScript发布订阅模式"
date: 2015-11-15 11:21:36 +0800
comments: true
categories: 
---
发布者和订阅者的目的就是为了对象之间的解耦，或者更大一点是JavaScript组件之间的解耦。

>Subscribe/Publish模式使用了一个主题/事件通道，这个通道介于订阅者和发布者之间。该事件系统允许代码定义应用程序的特定事件，该事件可以传递自定义参数，自定义参数包含订阅者所需要的值。其目的是避免订阅者和发布者产生依赖关系。 - 《Javascript设计模式》

下面是它的一种实现方式：

{% codeblock lang:javascript %}
var EventBus = {
    topics: {},

    subscribe: function (topic, listener) {
        // create the topic if not yet created
        if (!this.topics[topic]) this.topics[topic] = [];

        // add the listener
        this.topics[topic].push(listener);
    },

    publish: function (topic, data) {
        // return if the topic doesn't exist, or there are no listeners
        if (!this.topics[topic] || this.topics[topic].length < 1) return;

        // send the event to all listeners
        this.topics[topic].forEach(function (listener) {
            listener(data || {});
        });
    }
};

EventBus.subscribe('foo', alert);
EventBus.publish('foo', 'Hello World!');
{% endcodeblock %}

订阅者，在EventBus中注册事件/主题，并传递对应事件/主题的回调函数。   
发布者，发布对应的事件/主题，并传递需要的回调函数所需要的数据对象。

下面的这个写法，从函数命名上看更倾向于以注册事件和触发事件的方式理解，并且对事件的命名和回调函数的约束更加明确，也就是更鲁邦。

{% codeblock lang:javascript %}
(function() {
    var eventService = this.EventService = {};
    var _listeners = {};

    eventService.add = function (topic, func) {
        if (!func instanceof Function) {
            console.warn(topic + " must add a function.");
        }

        if (!_listeners[topic]) {
            _listeners[topic] = [];
        }
        _listeners[topic].push(func)
    };

    eventService.remove = function (topic, func) {
        if (_listeners[topic]) {
            var index = _listeners[topic].indexOf(func);

            if (index !== -1) {
                _listeners[topic].splice(index, 1);
            }
        }
    };

    eventService.fire = function (topic, data) {
        var listeners;

        if (typeof topic !== 'string') {
            console.warn('EventDispatcher', 'First params must be an event type (String)')
        } else if(_listeners[topic] == undefined)
            console.warn('EventDispatcher', 'No Event Register');
        else {
            _listeners[topic].forEach(function (func) {
                func(data);
            });
        }
    };

    return eventService;
}.call(this));
{% endcodeblock %}


参考资料：    
1.http://dev.housetrip.com/2014/09/15/decoupling-javascript-apps-using-pub-sub-pattern/