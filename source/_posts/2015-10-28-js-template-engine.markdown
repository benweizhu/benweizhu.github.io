---
layout: post
title: "了解JavaScript模板引擎"
date: 2015-10-28 08:36:18 +0800
comments: true
categories: 
---

模板引擎这个概念，相信对大家并不陌生，如果做过Java EE的开发，或者Rails开发，那么Jsp，Erb，Haml都是一种模板引擎，也许你还听说过Apache的velocity，只不过它们是后端的模板引擎，模板的渲染由服务器端完成。

比如这样的代码：
{% codeblock lang:html %}
//Jsp With EL
<div>user name: ${userName}</div>

//Haml
%div = "user name: #{userName}"
{% endcodeblock %}
JavaScript模板引擎，顾名思义，它也是一种模板引擎，只不过由JavaScript实现，是一种前端或者说是客户端的模板引擎。

###这个时候，也许你要问了，为什么需要前端模板引擎，它有什么作用？

其实，它的作用和后端的模板引擎作用相似，所以还是回归到了模板引擎的作用。

不知道，大家知不知道JavaEE(J2EE)的发展历程，基本过程是CGI-Servlet-JSP-Model1-Model2，再到之后的MVC。

如果你感兴趣，可以看我的这篇文章： http://benweizhu.github.io/blog/2013/12/21/web-mvc-by-example-with-spring-mvc/

###关于后端模板引擎(JSP)

在纯Servlet的开发年代，这样的web开发流程都在实现一个servlet类的实例，也就是无论是业务逻辑还是前端显示，都是放在Servlet类中来完成，通过实现doGet和doPost的方法，来完成前端参数的获取和视图的渲染。

这样做的缺点是表现逻辑、控制逻辑和业务逻辑全部写在了Java类中，导致逻辑非常混乱。

JSP的出现改变了这一现状，它是一种后端模板引擎，它由Sun和许多公司参与共同创建的一种使软件开发者可以响应客户端请求，而动态生成HTML、XML或其他格式文档的Web网页的技术标准。

JSP将前端的表现逻辑（用户界面）与后端的业务逻辑相分离，于是，你就可以像这样，动态渲染前端显示内容：

{% codeblock lang:html %}
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>helloWorldConfirm</title>
</head>
<body>
 hi,${user.name}
</body>
</html>
{% endcodeblock %}

**将前端表现逻辑和后端业务逻辑分离是模板引擎出现的最主要的原因。**

###那什么情况或者说场景下，我们要使用到JavaScript模板引擎呢？

1.通过ajax获取数据，再封装成视图展现到前端   
2.经常遇到字符串拼接   
3.需要抽取动态的公共模块，以实现重用   
等等

那么，目前有哪些比较常见的JavaScript模板引擎呢？

答案是好多，百度或者谷歌搜索，你会得到很多答案。举一个，最近在看的类库的例子，handlebarsjs：

{% codeblock lang:html %}
<div id="replace"></div>
<script id="entry-template" type="text/x-handlebars-template">
  <div class="entry">
    <h1>{ {title} }</h1>
    <div class="body">
      { {body} }
    </div>
  </div>
</script>
{% endcodeblock %}

{% codeblock lang:javascript %}
(function() {
  var source = $("#entry-template").html();//获取到模板内容
  var template = Handlebars.compile(source);//编译模板
  var context = {
    title: "My New Post",
    body: "This is my first post!"
  };
  var html = template(context);//传入Json对象，替换模板中的内容
  $("#replace").html(html);
}());
{% endcodeblock %}

<p data-height="268" data-theme-id="0" data-slug-hash="rOvyde" data-default-tab="result" data-user="benweizhu" class='codepen'>See the Pen <a href='http://codepen.io/benweizhu/pen/rOvyde/'>handlebarsjs example</a> by Benwei (<a href='http://codepen.io/benweizhu'>@benweizhu</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

你肯定会有些疑问，比如，在HTML中下面这段代码是干什么的？

{% codeblock lang:javascript %}
<script type = “text/template”> … </script>
{% endcodeblock %}

这段代码最常用的位置是实现客户端模板功能，通过将类型设置成为“text/template”，它变成了一段浏览器不能解释的脚本代码，所以浏览器就会忽略这段脚本。这样，就允许你在这段脚本片段中存放任何东西，存放的内容可以再之后由JavaScript抽取提供给一个模板库来生产HTML片段。

至于，剩余的函数是干什么的已经在代码中注释了，这里就不多解释。

问题又来了，Handlebars是来源于另外一个有名的模板引擎Mustache，

Mustache声明自己是一个logic-less（无逻辑或轻逻辑）语法模板。那么，什么是logic-less，有什么好处？其实答案在stackoverflow有人回答。参考： http://stackoverflow.com/questions/3896730/whats-the-advantage-of-logic-less-template-such-as-mustache

简单来说，比如在JSP中，提供了很多的taglib，实现了如if，loop（循环）等逻辑标签。这样就导致，在纯粹的前端显示代码中，仍然存在许多的逻辑判断和操作。

Mustache从设计上就不允许这样的操作，逼迫你在前端页面不要有任何逻辑的代码。但其实，这一点很难做到。

即便是基于Mustache的Handlebars，也提供了if和loop的语法(代码块中的两个大括号加了空格，只是为了博客中可以显示)：

{% codeblock lang:html %}
{ {permalink} }
{ {#each comments} }
  { {../permalink} }

  { {#if title} }
    { {../permalink} }
  { {/if} }
{ {/each} }
{% endcodeblock %}

关于模板引擎的框架还有很多，大家可以自己去搜索，我也可以给一个链接作为参考：http://www.imooc.com/article/1219 。当然谁好谁坏，众说风云，只有你用了才能知道适不适合你。

参考资料    
1.http://handlebarsjs.com









