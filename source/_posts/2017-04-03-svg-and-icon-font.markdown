---
layout: post
title: "前端不止系列 - 请告诉我，你要什么样的图标"
date: 2017-04-03 09:47:12 +0800
comments: true
categories:
---

####一个图标的生命周期（工作流程）

![Alt text](/images/svg/icomoon-svg.png =265x "图标的生命周期")

图标库 -> 图标使用（开发阶段）

![Alt text](/images/svg/ai-ps-svg.png =400x "图标的生命周期")

图标设计（设计阶段） -> 图标导出（沟通阶段） -> 图标使用（开发阶段）

第一种方式是一般是小公司或者独立开发者的工作流程。而对于大型组织或公司，因为他们拥有更完善的团队和资源，一般是第二种方式，能够获得自主权和建立企业VI（Visual Identity，企业视觉识别）的能力。

但无论是哪种方式，都包括两个角色：设计师和Web开发，只是第一种工作方式中，设计师是不可见的。

####图标的设计和使用

设计阶段通常是由不了解Web开发的设计师们来完成的，他们会根据产品的需要，绘画出满足需求的图标。

![Alt text](/images/svg/ThoughtWorksContactUSIcon.jpg =400x "ThoughtWorksContactUSIcon")    
ThoughtWorks官网Contact with us图标

然后交给Web开发人员使用，为什么要先介绍图标的使用，而一笔跳过导出过程呢？原因很简单，因为我们需要先知道服务的对象是谁，才知道如何正确的为它服务。

#####常见的三种使用图标的方式

1.使用图片

直接将设计师画好的图标，以PNG格式的图片一个个分离导出，这是最直观的图标打包方式。

![Alt text](/images/svg/taobao.png =300x "iconfont cn taobao icon")   
1688DPL中台图标库 

它的优点是：（1）能够使用彩色的图标（2）能够支持大部分浏览器；缺点是：（1）图标大小是固定的（不能根据场景自由缩放）（2）Retina屏幕需要两倍图。

开发人员拿到这样的图标，通常会需要先合成为一张图片，以方便制作[雪碧图][d11d6a59]，这个过程可以由开发人员自己完成，也可以由设计师（设计师可以根据源文件中心导出一张包含所有图标的PNG文件）。

  [d11d6a59]: https://css-tricks.com/css-sprites/ "雪碧图"

制作雪碧图的工具有很多，我比较常用的在线雪碧图工具是：[Sprite Cow][dd89d0bc]，或者NodeJS平台下的构建工具插件，如：[webpack-spritesmith][bb32f427]。

  [dd89d0bc]: http://www.spritecow.com/ "Sprite Cow"
  [bb32f427]: https://github.com/mixtur/webpack-spritesmith "webpack-spritesmith"

2.直接使用svg

使用SVG（可缩放矢量图形），W3C标准，最看好的Web端图形解决方案。它能提供如裁剪路径、Alpha通道、滤镜效果等复杂渲染能力，具备传统图片没有的矢量功能，还可以被记事本等阅读器、搜索引擎访问。

设计师可以轻松的在设计绘图软件（AI，PS）的帮助下导出SVG格式的图标/图片。

但目前，国内svg还并没有被非常广泛的使用，原因是它的兼容性，不能够很好的兼容旧的IE版本和一些Android原生浏览器。

![Alt text](/images/svg/svg-support.png =400x "svg support")    
Can I use svg?  

![Alt text](/images/svg/baidu.jpg =400x "baidu tong ji")      
百度2017年前三个月的浏览器使用统计，目前国内还有超过20%的用户仍在使用IE8，9甚至是IE7。

3.IconFont

IconFont是目前最为流行的图标解决方案，顾名思义，它就是字体文件，你可以用任何一个字体编辑工具打开它。

![Alt text](/images/svg/font-awesome.png =300x "font awesome")  

IconFont能够用CSS控制样式，无限缩放而不失真，支持IE7+，兼顾屏幕阅读器。而获得IconFont的方式也很简单，设计师将图标通过AI/PS转成SVG文件，然后由开发人员通过工具（在线或者本地）转换为IconFont。
