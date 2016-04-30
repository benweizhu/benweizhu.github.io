---
layout: post
title: "JavaScript事件流和事件代理/委托"
date: 2016-04-27 21:05:57 +0800
comments: true
categories: 
---

HTML元素都位于另一些。如果鼠标移动到某一个链接，或者点击了链接，同样鼠标也移动到了它的父元素上，并点击了那个父元素。

当你点击某个元素时，产生了一个点击事件，从而产生了一个事件流。

###事件流分为两种：

事件冒泡：事件从最具体的节点开始向外传播到最宽泛的节点。这是事件流的默认类型，被绝大多数浏览器所支持。

事件捕获：事件从最宽泛的节点开始向内传播到最具体的节点。这种方式在IE8和更早版本的IE中不被支持。

![Alt text](/images/js_event_bubble.png =500x "事件流")


<p data-height="424" data-theme-id="0" data-slug-hash="dMgMox" data-default-tab="js,result" data-user="benweizhu" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/benweizhu/pen/dMgMox/">Bubbling Event</a> by Benwei (<a href="http://codepen.io/benweizhu">@benweizhu</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

看一个codepen的例子

什么时候事件冒泡或者说事件流会变得如此重要？

当代码在一个元素和其祖先元素或者后代元素上都有事件处理程序时，事件流就会变得非常重要。

###事件代理/委托

为大量的元素创建事件监听器会造成页面速度下降，事件流允许你在父元素上监听事件。

如果用户可以和页面中的大量元素进行交互，比如：大量的按钮，很长列表，表格中每一个单元格。如果想这些元素分别添加事件监听器就会使用大量的内存，从而降低性能。

事件可以影响到容器元素（祖先元素），因此可以将事件处理程序放置在一个元素容器上，然后使用事件对象的target属性找到它的后代中是哪一个发生了事件，因此只需要响应一个元素上的事件（而不是在每一个子元素上分别响应事件）。

看一个codepen的例子

<p data-height="266" data-theme-id="0" data-slug-hash="WwaqyP" data-default-tab="js,result" data-user="benweizhu" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/benweizhu/pen/WwaqyP/">WwaqyP</a> by Benwei (<a href="http://codepen.io/benweizhu">@benweizhu</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>


