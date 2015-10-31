---
layout: post
title: "SEO实战密码：URL静态化"
date: 2015-10-31 19:01:24 +0800
comments: true
categories: 
---
URL静态化一直以来都是最基本的SEO要求之一，但近年来搜索引擎技术进步，抓取动态url已经不是问题，SEO行业对是否一定要做静态化也有了一些观念上的改变。

##为什么要做静态化？

现在的网站绝大多数都是数据库驱动，页面由程序实时生成，而不是真的在服务器上有一个静态HTML文件存在。当用户访问一个网址时，程序根据URL中的参数调用数据库，实时生成页面内容。因此动态页面相对应的URL原始状态也是动态的，包含问号，等号及参数。

搜索引擎在发展初期，一般不愿意爬行和收录动态URL，主要原因是可能陷入无限循环或收录大量重复内容，造成资源极大的浪费。最典型的无线循环就是某些网站上出现的万年历。万年历会使得蜘蛛可以无限点击下去。

真实用户一眼就能看出这是一个万年历，但是搜索引擎蜘蛛面对的只是一串代码，不一定能判断它。

有时候就算不是无限循环，动态URL也可能造成大量重复页面。比如：想用的URL和参数（也就是相同的页面），但是参数的顺序不同（有多个参数）。

更麻烦的是，有时候某些参数完全可以是任意值，服务器都能够正常的返回某个固定页面。

这就是为什么搜索引擎对动态URL敬而远之的原因。

##如何静态化URL呢？

最常见的方式是使用服务器URL重写模块，LAMP（Linux+Apache+MySQL+PHP）服务器上一般使用mod_rewrite模块，Windows服务器上也有类似的功能。

举例：

http://www.domain.com/products.php?id=123

静态化为:

http://www.domain.com/products/123

需要请用服务器模块mod_rewrite，然后在.htaccess文件写入代码：

{% codeblock lang:python %}
RewriteRule /products/([0-9]+)/products.php?id=$1
{% endcodeblock %}
URL重写基于正则表达式，每个网站的动态URL结构都不同，所以写起来就很麻烦。

严格的说，这里所说的URL静态化应该称为“伪静态化”，也就是说服务器上还是不存在相应的HTML文件，用户访问时还是动态生成页面，只不过通过URL重写技术使网站看起来像是静态的。当然，也有CMS系统可以实现真正静态化，系统会自动真实生成静态的HTML文件。但对于搜索引擎来说，真正的静态化和伪静态化没有区别。

##到底是否需要URL静态化？

近年来搜索引擎对URL的抓取有了很大进步。一般来说，URL中有两三个参数，对于收录不会造成任何影响。权重高的域名，再多几个问号也不是问题。不过一般还是建议将URL静态化，即能提高用户体验，又能降低收录难度。

2008年9月，Google站长博客发表了一篇关于动态网址还是静态网址的帖子，颠覆了SEO界的观念。

Google的帖子有几个要点。

一是Google完全有能力抓取动态网址，多少个问号也不是问题。这一点基本靠谱。

第二，动态网址更有助于Google蜘蛛读懂URL含义，并进行鉴别，因为网址中的参数有提示性。比如Google举了这个例子：

www.example.com/article/bin/answer.foo?language=en&answer=3&sid=98971298178906&query=URL

URL里的参数都有助于Google理解URL及网页内容。比如language后面跟的参数是提示语言，answer后面跟的是文章编号，sid后面的肯定是session ID。其他常用的包括color后面跟的参数指的是颜色，size后面跟的参数是尺寸等。有了这些参数的帮助，Google更容易理解网页。

而将网址静态化后，这些参数的意义通常就变得不明显了。比如这个URL：

www.example.com/shoes/red/7/12/men/index.html

就可能使Google不知道哪个是产品序列号，哪个是尺寸等。

第三，网址静态化很容易弄错，那就更得不偿失了。比如通常动态网址的参数调换顺序，所得到的页面其实是相同的，比如这两个网址很可能就是同一个页面：

www.example.com/article/bin/answer.foo?language=en&answer=3

www.example.com/article/bin/answer.foo?answer=3&language=en

保留动态网址，Google还比较容易明白这是一样的网页。而经过静态化后，这样两个网址Google就不容易判断是不是同一个页面，从而可能引起复制内容：

www.example.com/shoes/men/7/red/index.html

www.example.com/shoes/red/7/men/index.html

再一个容易搞错的是session ID，也可能被静态化进URL：

www.example.com/article/bin/answer.foo/en/3/98971298178906/URL

这样网站将产生大量URL不同，但其实内容相同的页面。

所以，Google建议不要静态化URL。

##虽然如此

但是《SEO实战密码》的作者建议还是做静态化。他认为Google的这个帖子解释的有些极端和牵强，也没有从其他搜索引擎的角度出发。

参考资料：   
1.《SEO实战密码》






