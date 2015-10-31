---
layout: post
title: "了解RactiveJS"
date: 2015-10-28 14:13:53 +0800
comments: true
categories: 
---
> Ractive is a next-generation DOM manipulation library for creating reactive user interfaces, optimised for developer sanity. It was originally developed to create interactive news applications at theguardian.com.

>Ractive.js is a template-driven UI library, but unlike other tools that generate inert HTML, it transforms your templates into blueprints for apps that are interactive by default.

>Two-way binding, animations, SVG support and more are provided out-of-the-box – but you can add whatever functionality you need by downloading and creating plugins.

看看Ractive的介绍，你能够看明白，RactiveJS是干嘛的吗？

1.为用户界面交互提供的下一代DOM操作类库（这是要干掉jQuery的节奏吗？）    
2.模板驱动的UI库（模板的概念可以看看我一篇关于《JavaScript模板引擎》的文章），但不同于其他工具产生可以插入的HTML代码，它将模板变成了用户交互的蓝图。   
3.双向绑定，动画，和SVG支持等开箱即用的功能。

花费60秒去看一下，官方的《60 second setup》， http://www.ractivejs.org/60-second-setup

你会发现，这不就是JavaScript模板引擎吗？只不过不需要你，手动将编译得到的HTML片段添加到指定的元素上。

但实际上，还远远不止如此。

##模板引擎
就像它自己介绍的和你在《60 second setup》中看到了那样，Ractive和模板引擎有很强的关联性，它要求所使用的HTML模板片段是严格符合HTML标准的（不会像浏览器那样，允许一些未关闭的标签）。

**引用**

在这个模板引擎中有一个“引用”的概念，其实指的就是模板中的占位符变量，比如《60 second setup》中的name变量。

因为在定义的Ractive实例中有一个data属性，包含了所有的“引用”变量，在这里又引入一个context上下文的概念。通过定义上下文（比如messages和user），你就可以不需要messages.total来指定占位符，而是这样：

{% codeblock lang:html %}
{ {#user} }
  <p>Welcome back, { {name} }!
    { {#messages} }
      You have { {unread} } unread of { {total} } total messages.
    { {/messages} }
  </p>
{ {/user} }

data: {//data长这个样
    user: {
      name: 'Jim',
      messages: {
        total: 10,
        unread: 3
      },
      lastLogin: 'Wednesday'
    }
 }
{% endcodeblock %}

**表达式**
表达式允许你在模板中定义逻辑，比如：
{% codeblock lang:html %}
<div class='bar-chart'>
  <div style='width: {{ value * 100 }}%;'>{{ i + 1 }}</div>
</div>
{% endcodeblock %}

*Partial*

Partial是一个模板代码片段，它可以被插入到模板当中，或者其他的partial中。主要作用是通过抽象（或者抽取）来简化模板的内容。

{% codeblock lang:html %}
<!-- the main template -->
<div class='gallery'>
  { {#items} }
    { {>thumbnail} }
  { {/items} }
</div>
<!-- the partial -->
<figure class='thumbnail'>
  <img src='assets/thumbnails/{{id}}.jpg'>
  <figcaption>{{description}}</figcaption>
</figure>
{% endcodeblock %}

{% codeblock lang:javascript %}
ractive = new Ractive({
  el: myContainer,
  template: myTemplate,
  partials: { thumbnail: myThumbnailPartial },
  data: { items: myItems }
});
{% endcodeblock %}
##双向绑定

数据双向绑定的概念在后端MVC中用的挺多的，比如，SpringMVC。而实现并标榜这一特性的前端框架也很多，例如Ember.js，AngularJS以及KnockoutJS。

默认情况下，如果你有使用input，textarea或者select等元素，Ractive实例会基于用户的输入更新它的内部模型。

如果你不想在某一个元素上使用双向绑定，你可以通过设置twoway: false属性，将该特性disable。

{% codeblock lang:html %}
<input value="{ {foo} }">

<input value="{ {foo} }" twoway="false">
{% endcodeblock %}

**双向绑定对contenteditable元素的支持**

contenteditable是html5的属性，任何一个元素，比如div，加上contenteditable，该元素就可以被编辑了。

{% codeblock lang:html %}
<div contenteditable="true" value="{{content}}"></div>
{% endcodeblock %}






