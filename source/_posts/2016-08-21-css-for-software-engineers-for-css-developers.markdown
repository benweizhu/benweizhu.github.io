---
layout: post
title: "CSS和软件工程中的设计原则"
date: 2016-08-21 18:10:13 +0800
comments: true
categories:
---

原视频： https://vimeo.com/177216958

PPT如下：

<script async class="speakerdeck-embed" data-id="087e1c8ae1c4452f82ae6fd5e6215a9a" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

作者提到CSS开发过程的很多原则：
  • DRY/Single Source of Truth   
  • The Single Responsibility Principle   
  • The Separation of Concerns   
  • Immutability   
  • Cylcomatic Complexity   
  • The Open/Close Principle   
  • Orthogonality

这些原则在其他软件开发领域应用非常广泛，然而绝大多数人在写CSS的时候却又显得相当随意。
可以参考一下作者的观点，每一原则他都给出了对应的应用场景。

同时对于非常喜欢写嵌套规则的同学，强烈建议去了解[Cylcomatic Complexity](圈复杂度)，尝试着去减少嵌套，减少以后的维护成本。

个人感觉，作者对于这些设计原则本身的说明，占用了太多的篇幅，并没有告诉你如果要实现这个设计，应该怎么做？但作者讲解了一些反例，并印证它不满足那些原则。我们应该反向思考，我们在css中过于随意的css选择器，定义出scope正确的合理的css。
