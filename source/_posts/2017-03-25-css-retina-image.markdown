---
layout: post
title: "前端不止HTML/JS/CSS系列 - Retina屏幕下两倍图"
date: 2017-03-25 16:40:09 +0800
comments: true
categories:
---
###所见不一定即所得

> 眼睛是心灵的窗户，也是蒙蔽你的一种途径。

假设，我给你一张图片，你觉得肉眼可以观察到全部的细节吗？

###屏幕上一张清晰的图片

肉眼在屏幕上看到图片的清晰度由三个因素决定，一是图片像素本身是否精细，二是屏幕分辨率，三是屏幕大小。

我们来逐步分析它们之间的关系：

####屏幕分辨率

屏幕分辨率也就是设备分辨率，设备像素，它是物理的像素，比如，新的iPhone7，屏幕分辨率是1334 x 750像素分辨率，326 ppi。

![Alt text](/images/iphone7-screen.png =400x "iPhone7分辨率")

####图像大小

如果你学过《数字图像处理》这门课，那你对下面的解释就是非常熟悉了。

位图是由像素（Pixel）组成的，像素是位图最小的信息单元，存储在图像栅格中。每个像素都具有特定的位置和颜色值。按从左到右、从上到下的顺序来记录图像中每一个像素的信息，如：像素在屏幕上的位置、像素的颜色等。位图图像质量是由单位长度内像素的多少来决定的。单位长度内像素越多，分辨率越高，图像的效果越好。

![Alt text](/images/tw-logo.png =200x "ThoughtWorks Logo")

假设，以上这个logo的图像大小是1334 x 750像素和iPhone7屏幕分辨率一样，那么,一位图像素对应的就是一个设备像素，这就是会是一个完全保真的显示。因为一个位置像素不能进一步分裂，我想这一点应该大家非常容易理解，也就是一个萝卜一个坑。

####屏幕分辨率和屏幕尺寸

![Alt text](/images/resolution.png =400x "Windows分辨率")

相信大部分人对上面这个设置肯定特别熟悉，有些人可能对XP，甚至98系统的样式更熟悉（一不小心暴露了年龄），在Windows系统下，提高屏幕分辨率一般都需要提高屏幕尺寸。

因为在固定屏幕的情况下，提高屏幕分辨率（如上图），图像和文字显示目标会相应缩小，原因是系统并不会自动根据屏幕尺寸和分辨率关系相应的调整文字和图标的大小，这是Windows系统自身的行为。

我相信，如果家里有年长的人使用电脑，肯定屏幕分辨率调的很低，因为这样文字和图标才会比较大，我家06年买的台式机就是这样。

也因此，我们很容易有一个错觉，那就是屏幕越大，分辨率就能越大（在单位面积内像素数量固定的情况下，尺寸越大，单个屏幕拥有的像素就越多，分辨率自然就越大）。

**直到，苹果Retina屏幕的出现，原来小屏幕也可以拥有大分辨率。**

####PPI的概念

PPI，像素密度，即每英寸所拥有的像素数目（比如：上面iPhone 7的PPI是326），PPI数值越高，代表显示屏能够以越高的密度显示图像，画面的细节就会越丰富。

以Retina屏幕为例，它并不是像普通显示器那样通过增大尺寸来增加分辨率，而是靠提升屏幕单位面积内的像素数量，即像素密度来提升分辨率，这样就有了高像素密度屏幕。

根据上面的分析，分辨率提升了，那么图标和文字尺寸就会变小，但是Mac的操作系统不同，它自动采取相应的模式（如Mac下的HiDPI）进行适配，将缩小后的字体（苹果一直采用矢量字体）和图标重新放大，这样苹果用了更多的像素数来显示同样的内容，所以显示尺寸仍然不变。

苹果将“高像素密度屏幕”的概念营销出一个专业的术语“Retina”，将其称为双密度显示，声称人类的肉眼将无法区分单个像素。

当一个显示屏像素密度超过300ppi时，人眼就无法区分出单独的像素。这也是讲：显示设备清晰度已达到人视网膜可分辨像素的极限。因此，行动电话显示器的像素密度达到或高于300ppi就不会再出现颗粒感，而手持平板类电器显示器的像素密度达到或高于260ppi就不会再出现颗粒感，苹果电脑Mac的Retina显示器像素密度只要超过200ppi就无法区分出单独的像素。

![Alt text](/images/retina-display.jpg =500x "retina-display")

好，说了这么多，都是谈屏幕的问题，貌似和前端开发没有什么关系，我又不是要买新手机（呵呵），那么现在，我们现在来谈谈前端的问题。

####Web中的像素（CSS像素）

CSS像素是一个抽象概念，设备无关像素，简称-“DIPS”，device-independent像素，主要使用在浏览器上，用来精确的度量（确定）Web页面上的内容。

在标准情况下一个CSS像素对应一个设备像素。

{% codeblock lang:css %}
.box {
  width: 200px;
  height: 300px;
  font-size: 12px;
}
{% endcodeblock %}

上面的代码，将会在显示屏设备上绘制一个200x300像素的盒子，在标准屏幕下，它占据的就是200x300设备像素。但是在Retina屏幕下，相同的div却使用了400x600设备像素，保持相同的物理尺寸显示，导致每个像素点实际上有4倍的普通像素点。

![Alt text](/images/retina-web.jpg =500x "retina-web")

对于图片来说也是如此：

![Alt text](/images/retina-web-bitmap.jpg =500x "retina-web bitmap")

这个时候，屏幕会怎么处理呢？其实，有点类似图像软件的放大图片功能，采用自有的算法（图像处理算法）计算放大方式。只不过，这里是苹果Retina屏幕的计算方法，一个CSS像素点实际分成了四个，造成颜色肯定会存在偏差（非全保真的显示），于是，我们看上去就变得模糊了（特别是图片，非常的明显）。

开发当中遇到这样的事情，我们应该怎么处理呢？这时，我们需要引出devicePixelRatio的概念。

#####devicePixelRatio设备像素比

window.devicePixelRatio是设备上物理像素和设备独立像素(device-independent pixels (dips))的比例。

公式表示就是：window.devicePixelRatio = 物理像素 / dips

* 普通密度桌面显示屏的devicePixelRatio=1
* 高密度桌面显示屏(Mac Retina)的devicePixelRatio=2
* 主流手机显示屏的devicePixelRatio=2或3

举例说明，一张100x100的图片，通过CSS设置它{ width:100px; height:100px }。在普通密度桌面显示屏的电脑上打开，没有什么问题，但假设在手机/或者Retina屏幕的Mac，按照逻辑分辨率来渲染，他们的devicePixelRatio=2，那么就相当于拿4个物理像素来描绘1个电子像素。这等于拿一个2倍的放大镜去看图片，图片可能因此变得模糊。

![Alt text](/images/devicePixelRatio.png "devicePixelRatio")

###代码如何解决呢？

原理我们明白了，那么从代码层面，我们应该如何实现呢？

一个常见的做法是把图片换成200x200的，CSS宽高不变，仍然是{ width:100px; height:100px }，这样，CSS宽高换算成物理像素是200x200，图片也是200x200，就不会变糊了。可以采用媒体查询和JS操作的方式

####CSS Media Queries

{% codeblock lang:css %}
#element { background-image: url('hires.png'); }

@media only screen and (min-device-pixel-ratio: 2) {
    #element { background-image: url('hires@2x.png'); }
}

@media only screen and (min-device-pixel-ratio: 3) {
    #element { background-image: url('hires@3x.png'); }
}
{% endcodeblock %}

####JS查询
**retinajs库**

http://imulus.github.io/retinajs/

![Alt text](/images/caniuserdevicePixelRatio.png "devicePixelRatio")

###是不是适配Retina屏幕所有的图片都需要切换呢？

不是，一般情况下，不需要针对网站上的所有图片都提供两个版本（非Retina屏幕和Retina屏幕），大部分图片缩放并不会太多的影响用户的体验。

不过有一个需要注意的是，网站的logo，因为logo本身图像大小比较小，在Retina上物理像素放两倍显示就会出现模糊情况，这个时候，你就需要通过媒体查询或者JS操作来替换图片。

*最后：眼睛是心灵的窗户，也是蒙蔽你的一种途径，带上知识的眼镜，将世界看个清楚*

参考资料：    
1. http://www.w3cplus.com/css/towards-retina-web.html   
2. http://www.jianshu.com/p/bb76c606f0b4    
3. https://developer.mozilla.org/zh-CN/docs/Mobile/Viewport_meta_tag   
4. http://caniuse.com/#search=devicePixelRatio    
5. https://www.web-tinker.com/article/20590.html
