---
layout: post
title: "Living style guide - 缩小设计和开发的沟通鸿沟"
date: 2016-10-27 10:04:27 +0800
comments: true
categories:
---
本文作者：朱本威，杨松

###什么是Style Guide（风格指南）？

如果你做过Web开发，我打赌你肯定听说过“风格指南”!

风格指南是什么？它是一系列关于书写和设计标准的文档，比如一些常见的标准有：字体，颜色，Logo，间距等等。

风格指南最早出自于印刷行业，比如：出版社就是风格指南的常见用户。

<p style="text-align: center;">
  <img src="/images/style-guide/duzhe99.png" width="500" title="1999年的《读者》" alt="Alt text">
  <img src="/images/style-guide/duzhe2016.png" width="294" title="2016年的《读者》" alt="Alt text">
</p>
<p style="text-align: center;">从1999年到2016年，《读者》的封面风格都没有改变过</p>

好的品牌，通常都有一个非常棒的风格指南，无论是哪个行业，比如：KFC。我们也将KFC的这种风格指南成为VI（Visual Identity，企业视觉设计）。

<p style="text-align: center;">
  <img src="/images/style-guide/KFC.png" width="500" title="KFC Style Guide" alt="Alt text">
</p>
<p style="text-align: center;">KFC老爷爷大家都认识</p>

对于我们的WEB开发，也需要风格指南，目的是保证最终的交付产物和最初设计的一致性，打造出一致的核心用户体验。
<p style="text-align: center;">
  <img src="/images/style-guide/optus.png" width="500" title="OPUTS Style Guide" alt="Alt text">
</p>
<p style="text-align: center;">Optus澳大利亚电讯公司</p>

###风格指南过去的工作方式

在过去，一种常用的工作方式，设计与开发是分离开了。设计师在这儿，前端工程师在那儿，中间有一条无形的巨大的鸿沟。

唯一的沟通渠道就是这样一个PDF材料。设计师最开始把PDF给你，然后和工程师各自为阵开始忙自己的，打死不相往来。

沟通不及时就不说了，有时候设计还要埋怨你，说你的实现怎么和他最初的设计不一样。

每一个工程师都应该都会有一段被设计师逼着改实现的经历，也许有的设计师们根本不理解我们修复IE问题时的痛苦，也不一定明白PDF文档沟通的效率是有限的。

“我怎么能知道点击每一个按钮时候的行为呢？”

答案是我根本不知道，等我看完了一百页文档，还不如你直接给我解释来得简单直接。

<p style="text-align: center;">
  <img src="/images/style-guide/ui-dev.png" width="500" title="UX 和 UIDev" alt="Alt text">
</p>
<p style="text-align: center;">UX，UI Dev和Backend Dev</p>

而每一次修改都需要很长的反馈周期，中间浪费了不少时间。

然而，当第二版，第三版，第四版来的时候，我的PDF和PSD文件已经堆积成山，我也不清楚哪一个才是最新的版本，没有版本管理工具，即便有，这种二进制文件也很难立刻查看它和上一个版本的区别在哪。

而且我相信大部分像我一样没有强迫症的人，一定是文件下载或者拷贝随意放置在桌面或者什么位置，然后常常根据时间排序去找最新的。久而久之，就不知道在哪了，然后又找设计师要一版。

经历了以上种种的痛点之后，我们是怎么做的？

###Living Style Guide
<p style="text-align: center;">
  <img src="/images/style-guide/living-style-guide-logo.png" width="500" title="UX 和 UIDev" alt="Alt text">
</p>

我们忍受不了这种不够敏捷的工作方式，尝试着用新的方式去解决这个问题，我们构建一个Living Style Guide。它是一个在线站点，同时也是一种工作方式，一种实践。我们尝试通过这种实践去解决这个问题。

**Living Style Guide和Style Guide的区别：**

一个直观的变化就是，曾经的PDF文档，代码化了。比如，最开始只存在于PDF文件中的调色板，字体，在网站上就可以直接查看，而不用担心是否过期，找不到等等。

同时，也是一种工作方式的改变，我们的设计师与工程师需要更加紧密地工作在一起了。让Living Style Guide成了一个钮带，连接着设计师与工程师。

设计看到的文档即代码，开发人员看到的代码即文档。不会出现设计师所认为的设计和实际开发出来的结果不一致的情况，我们做到Single source of truth(唯一真实来源)。同时，你看到的是动态，可交互的一系列组件。不需要考虑怎么用文档去描述一种行为，你所看到的就包含自身行为。

敏捷宣言中有这样一句话：工作的软件 高于 详尽的文档，Living Style Guide在努力实现这一点，尝试将设计与前端的闭环缩小。

###聊聊工程师们关心的干货

**选择合适的技术栈**

技术栈的选择还是要回归到项目需求上。我们从大的方向上考虑了这么几点：组件化，数据驱动，易定制和可测试。

Living Style Guide是一个供设计师和前端开发工程师使用的平台，并可以将开发的内容打包使用在其他不同的Web应用平台，对通用性和易用性都有较高的要求，我们没有做一个大而全的框架，而是想要做到以组件为单位，灵活发布，并能灵活使用在各种平台。

<p style="text-align: center;">
  <img src="/images/style-guide/tech-stack.png" width="500" title="我们的技术栈" alt="Alt text">
</p>
<p style="text-align: center;">我们的技术栈</p>

上面这张图涵盖了我们大部分的技术栈内容，具体内容我就不单独介绍了。不过，我突然想起来前段时间有一篇非常火的文章《在2016年学JavaScript是一种什么样的体验?》，它写的就是我们这个feel。

**关于代码组织结构方式**

组件化是我们的核心价值，所以在代码结构上，我们将“自包含”属性作为一个很重要的特性。每一个组件包含了它自己所需要的相关文件：测试文件（单元测试，功能测试），样式文件（Sass），组件和文档，而公共的部分，我们抽离到其他位置，从而尽量做到高内聚，低耦合，同时提高代码重用性。

<p style="text-align: center;">
  <img src="/images/style-guide/folder-structure.png" width="300" title="代码组织结构" alt="Alt text">
</p>
<p style="text-align: center;">我们项目的代码组织结构</p>

**需要注意的点**

组件化的好处是相互隔离，又可被重用。实现组件化一定要需要合理的分类和管理好组件，比如，区分业务组件和通用组件，保证被重用的组件拥有足够的可扩展性。

**最后**

Living Style Guide是我们在缩小设计与开发沟通鸿沟的一种尝试，努力将闭环缩小，以提高效率，降低成本，这是一种实践，也是设计师和工程师工作方式的改变。

转载请注明： http://benweizhu.github.io/blog/2016/10/27/living-style-guide/

参考资料：   
1. https://asinthecity.com/2011/11/10/the-difference-between-a-ux-designer-and-ui-developer/   
2. 敏捷宣言（ http://agilemanifesto.org/iso/zhchs/ ）
