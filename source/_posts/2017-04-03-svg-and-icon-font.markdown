---
layout: post
title: "前端不止系列 - 请告诉我，你要什么样的图标"
date: 2017-04-03 09:47:12 +0800
comments: true
categories:
---
####一个图标的生命周期（工作流程）

![Alt text](/images/svg/icomoon-svg.png =265x "图标的生命周期")     
图标库(选择阶段) -> 图标使用（开发阶段）

![Alt text](/images/svg/ai-ps-svg.png =400x "图标的生命周期")     
图标设计（设计阶段） -> 图标导出（沟通阶段） -> 图标使用（开发阶段）

第一种方式是一般是小公司或者独立开发者的工作流程。而对于大型组织或公司，因为拥有更完善的团队和资源，一般是第二种方式，能够获得更多自主权和建立企业VI（Visual Identity，企业视觉识别）的能力。

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

使用SVG（可缩放矢量图形），W3C标准，**最看好的Web端图形解决方案**。它能提供如裁剪路径、Alpha通道、滤镜效果等复杂渲染能力，具备传统图片没有的矢量功能，还可以被记事本等阅读器、搜索引擎访问。

设计师可以轻松的在设计绘图软件（AI，PS）的帮助下导出SVG格式的图标/图片。

但目前，国内svg还并没有被非常广泛的使用，原因是它的兼容性，不能够很好的兼容旧的IE版本和一些Android原生浏览器。

![Alt text](/images/svg/svg-support.png =400x "svg support")    
Can I use svg?  

![Alt text](/images/svg/baidu.jpg =400x "baidu tong ji")      
百度2017年前三个月的浏览器使用统计，目前国内还有超过20%的用户仍在使用IE8，9甚至是IE7。

3.IconFont

IconFont是目前最为流行的图标解决方案，顾名思义，它就是字体文件，你可以用任何一个字体编辑工具打开它，如果你打开某一个查看，你会发现它就是一些路径，这些路径可以用AI，PS，Sketch等软件来绘制。

![Alt text](/images/svg/font-awesome.png =300x "font awesome")  

IconFont的优点在于能够用CSS控制样式，无限缩放而不失真，支持IE7+，兼顾屏幕阅读器，不过缺点是不能支持彩色（拥有多种颜色的图标）图标。获得IconFont的方式也很简单，设计师将图标通过AI/PS转成SVG文件，然后由开发人员通过工具（在线或者本地）转换为IconFont，比如：国外的[icomoon.io][714f8918]，国内的[iconfont.cn][b5df4e92]，开源构建工具插件有[gulp-iconfont][679293f7]等等。

  [714f8918]: https://icomoon.io/ "icomoon.io"
  [b5df4e92]: http://iconfont.cn/ "iconfont.cn"
  [679293f7]: https://github.com/nfroidure/gulp-iconfont "gulp-iconfont"

####产生适合Web开发的图标

“产生适合Web开发的图标”是我们今天要关注的重点。

1.使用图片的方式

如果开发人员直接使用图片，则相对简单，设计师只需要针对普通屏幕和Retina屏幕准备两套图（单倍图和两倍图）。

以国内某著名的中文小说阅读网站为例，会针对不同的设备使用不同倍数的logo图片，以保证在如Retina屏幕下的清晰度。

{% codeblock lang:css %}
.logo-wrap .logo a {
    display: block;
    width: 219px;
    height: 52px;
    background: url(/qd/images/logo.0.2.png) no-repeat;
}

@media (min--moz-device-pixel-ratio:1.3),(-o-min-device-pixel-ratio: 2.6 / 2),(-webkit-min-device-pixel-ratio:1.3),(min-device-pixel-ratio:1.3),(min-resolution:1.3dppx) {
  .logo-wrap .logo a {
      background: url(/qd/images/logo3x.0.2.png) no-repeat;
      background-repeat: no-repeat;
      background-size: 217px;
  }
}
{% endcodeblock %}

2.使用SVG

关于转换成SVG，这里就要引荐一下Sara Soueidan在Generate London 2015 Conference上的演讲[《Sara Soueidan: SVG for Web Designers (and Developers)》][2f145803]（YouTube视频需要翻墙），如果不方便，Sara Soueidan有一篇博客[《Tips for Creating and Exporting Better SVGs for the Web》][13043805]更详细的讲解关于SVG导出的内容，当然，还有一篇国内的翻译文章[《创建和导出SVG的技巧》][8092bc6d]，最后在推荐一篇Adobe工程师michael chaize写的关于AI导出SVG的文章[《Export SVG for the web with Illustrator CC》][a6335300]。

  [2f145803]: https://www.youtube.com/watch?v=q4QI9iOeyPo "《Sara Soueidan: SVG for Web Designers (and Developers)》"
  [13043805]: https://sarasoueidan.com/blog/svg-tips-for-designers/ "《Tips for Creating and Exporting Better SVGs for the Web》"
  [8092bc6d]: http://www.w3cplus.com/svg/svg-tips-for-designers.html "《创建和导出SVG的技巧》"
  [a6335300]: http://creativedroplets.com/export-svg-for-the-web-with-illustrator-cc/ "《Export SVG for the web with Illustrator CC》"

不过，我觉得看视频更直观，顺便领略一下这位优秀的 **阿拉伯女性前端开发工程师（兼自由作家和演讲人）** 的风采。

博客和视频中谈到了多个点导出SVG需要注意的地方，篇幅限制，这里简单描述三个tip：

**（1）选择适合绘画的画板**。

你有在网页上嵌入过SVG吗，给它指定一个高度和宽度，然后发现它其实比你指定的尺寸要小？开发人员常常会遇到这样的问题。

大多数情况下，这是因为SVG视窗中有一定大小的白色空白的空间。视窗是按照你在样式表中指定的尺寸显示的，但是它里面有额外的空白——在图形周围——使得你的图片看起来好像“缩水”了，因为这块空白是占空间的，在视窗里面。为了避免这种情况，你需要确保你的画板是刚刚好放下里面的图像的，不要大太多。

画板的尺寸就是导出的SVG的视窗的尺寸，所有画板上的空白都会最终变成视窗中的白色空白。

![Alt text](/images/svg/fit-artboard.png =400x "fit artboard")  

*对于没有AI工具的开发，可以在下面的SVGO优化选项中选择“Prefer viewBox to width/height”。*

**（2）选择合适的导出选项**

![Alt text](/images/svg/export-options.png =400x "保存")     
上面的图片中展示的选项是推荐的生成适合Web使用的SVG的。如果你不想使用Web字体，可以选择把文本转换成轮廓。

![Alt text](/images/svg/output-fewer.png =400x "output-fewer")  
如果SVG中包含大量的文字，这个选项output fewer tspan elements可以很大程度降低svg的大小。

**（3）优化SVG**

通常是建议在把SVG从图形编辑器中导出后，再用单独的优化工具来进行优化。比如：删除无用Comments和Metadata，简化代码，简化单个路径等。推荐的第三方工具：NodeJS工具[svgomg][ca74c2fc]，AI插件[SVG-NOW][86db84bd]，Sketch插件[Svgo-compressor][bc537040]等，请参考Sara Soueidan的文章[《Useful SVGO[ptimization] Tools》][5046bb9d]。

  [ca74c2fc]: https://jakearchibald.github.io/svgomg/ "svgomg"
  [86db84bd]: https://github.com/davidderaedt/SVG-NOW "SVG-NOW"
  [bc537040]: https://github.com/BohemianCoding/svgo-compressor "Svgo-compressor"
  [5046bb9d]: https://sarasoueidan.com/blog/svgo-tools/ "《Useful SVGO[ptimization] Tools》"

![Alt text](/images/svg/svgomg.png =400x "优化svg")   

3.IconFont

前面提到IconFont一般是由SVG通过工具转换而来，而如果开发最终需要使用IconFont展示图标，则对于导出的SVG有一些特殊要求。我在本文的前面一小节，已经介绍了几款IconFont的转换工具，每一款工具其实都有详细的文档说明SVG绘制的规则，尽管不尽相同，但有一些基本原则是一致的：

（1）将文字转换为路径    
（2）不可以使用图片（字体只是路径）   
（3）修剪画板（trimming to art boundaries）（前面已经介绍过）   
（4）将描边转化为闭合图形   
（5）简化无用的节点    
等等

更多关于IconFont的绘画规则，请参考：[Iconfont.cn文档][8d1e0301]，[Icomoon文档][de1f5b96]，[gulp-iconfont文档][ae177e48]，[fontello文档][c0bfea1f]。

  [8d1e0301]: http://iconfont.cn/plus/help/detail?helptype=draw "Iconfont.cn文档"
  [de1f5b96]: https://icomoon.io/#docs/importing "Icomoon文档"
  [ae177e48]: https://github.com/nfroidure/gulp-iconfont#preparing-svgs "gulp-iconfont文档"
  [c0bfea1f]: https://github.com/fontello/fontello/wiki/How-to-use-custom-images "fontello文档"

####尽早的沟通

无论是开发还是设计师，最重要的还是沟通，借用Sara Soueidan的一句“设计师和开发者应该成为好朋友”。
