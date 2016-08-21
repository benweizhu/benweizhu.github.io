---
layout: post
title: "SVG和IconFont"
date: 2016-07-12 15:28:07 +0800
comments: true
categories:
---
在大型企业中，它们都拥有自己的VI，有自己的设计团队，他们来开发设计一些企业特有的图标和标志。而这些图标一般以矢量图的方式提供给我们。

矢量图，也称为面向对象的图像或绘图图像，在数学上定义为一系列由线连接的点。矢量文件中的图形元素称为对象。每个对象都是一个自成一体的实体，它具有颜色、形状、轮廓、大小和屏幕位置等属性。

SVG（Scalable Vector Graphics）是什么？

大部分人第一次见到它，肯定认为是一种矢量图图形格式，但严格来说它也是一种开放标准的矢量图形语言，是使用XML来描述二维图形和绘图程序的语言，它是一门语言，DSL（领域特定语言）。

{% codeblock lang:xml %}
<?xml version="1.0" standalone="no"?>

<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
"http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<svg width="100%" height="100%" version="1.1"
xmlns="http://www.w3.org/2000/svg">

<circle cx="100" cy="50" r="40" stroke="black"
stroke-width="2" fill="red"/>

</svg>
{% endcodeblock %}

上面这段代码得到的svg矢量图，如下：

![Alt text](https://rawgit.com/benweizhu/5036fcaf39960a28ce9b0aca17a78caa/raw/31797b14c698431c9a6f5435a2f5a28dda2ce1bd/circle.svg)

我们在响应式网页设计的工作中经常用到SVG，因为一般的图片没有办法进行适当的缩放，普通图片的缩放会导致图像的失真，同时，图片的文件大小也可能和你希望的图片不一致（大图被缩小使用，导致图片文件大小仍然很大）。

我相信大部分前端开发工程师都用过IconFont，比如：http://fontawesome.io/icons/ 和bootstrap中的glyphicons http://v3.bootcss.com/components/#glyphicons 。

关于为什么要转换为Icon Font？CSS-Tricks中有一篇文章详细的对比了SVG和ICON Font，见： https://css-tricks.com/icon-fonts-vs-svg/

简单版，一个最主要的原因必须使用IconFont，因为要兼容IE8。

一个比较有用的gulp插件，可以将SVG文件转化为font字体文件，并自动生成对应的scss文件（包含font-familly和content）。

https://github.com/nfroidure/gulp-iconfont

当然，我们也可以使用一些在线工具来将SVG文件转化成字体文件比如： https://icomoon.io/ 。
