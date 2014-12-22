---
layout: post
title: "了解war包和ear包"
date: 2014-05-03 17:06
comments: true
categories: 
---

上一次，讲到了如何如何手动编译Java，简单的介绍打Jar包（提示：在classpath的基础上，一定将包的目录结构打进去）。Jar包的目的，将编译过的class文件有效合理的组装起来，方便管理和使用。这一次，滕老板继续跟我们讲解了如何打War包。

在本篇文章中，将分别介绍War包和Ear包（在项目中遇到的Ear包的模块，不清楚是什么，所以随便一起弄清楚）。

##**War包**

只要做过Java开发的同志，肯定知道War包，至少听说过。

War包（Web application ARchieve）也是一种Jar包文件，它被用来描述由JSP，Servlet，Java类，XML文件，tag库，静态web页面等资源组成的集合，它们组合在一起成文一个web应用。

从命令上说，打War包的命令和打Jar包的命令一样，比较简单，主要是要了解一个War包的目录结构：

<image src="/../images/ear/war.png" height="50%" width=50%" style="text-align: center"/>


MANIFEST.MF是一个可选文件，用来描述额外的元数据信息。

WEB-INF目录包含了War中的私有文件，也就是说，当web应用被部署之后，该目录下的文件不能够由Web客户端（浏览器）直接访问的。

WEB-INF/lib/用来放置你代码中需要使用的第三方的jar文件。

WEB-INF/classes/用来放置你自己编译的class文件。

WEB-INF/web.xml是web部署描述器，JavaEE配置web模块的标准描述器，这里不详细解释。

最后是公共的静态文件。

那么在打包的时候，按照这个目录结构打包，然后将War包放置到tomcat的webapp目录下，tomcat在运行时就会自动帮你解包并运行，或者你也可以直接将包含该目录结构的目录直接拷贝到tomcat的webapp下，一样可以运行，打包只不过是一个封装和压缩过程。

##**那什么是Ear包呢？**

Ear(Enterprise ARchieve)用于在Java EE中将一个或者多个模块封装到一个文件中，这样，多个不同模块在应用服务器上的部署就可以同时并持续的进行。

Java EE应用以Jar文件，War文件和Ear文件形式呈现。War或者Ear文件都是标准的Jar文件，只不过扩展名是.war和.ear。通过Jar，War和Ear等文件或模块的方式，使得用一些相同组件，来构建不同的JavaEE应用成为可能。不需要额外的编码，只需要将不同的JavaEE模块打包到不同的Java EE的Jar包，War包或Ear包的文件中。

一个Ear文件是由Java EE模块和可选的部署描述器组成。部署描述器是一个带有.xml扩展名的XML文档，描述了一个应用，模块或者组件的部署设置。因为部署描述其的信息是声明式的，所以可以直接修改它，与我们的源代码没有关系。在运行时中，Java EE的服务器会读取部署描述器的内容，并根据描述对应用，模块或者组件做相对应的操作。

<image src="/../images/ear/ear.png" height="50%" width=50%" style="text-align: center"/>
<image src="http://docs.oracle.com/javaee/6/tutorial/doc/figures/overview-applicationModule.gif" style="text-align: center"/>
上面两个图，分别摘自Jboss at Work(A practice guide)和JavaEE6 Tutorial。


一个Java EE模块是由一个或者多个为同一个容器类型准备的Java EE组件组成，当然还包含一个可选的部署描述器文件。

Java EE模块有以下几种类型：

EJB模块，它包含企业级别的bean类文件，一个EJB的部署描述器（可选），EJB模块会以Jar包形式组装。

Web模块，它包含了servlet类文件，web文件，其他相关class文件，图片，html静态文件，和一个web.xml部署文件（可选），并最终以war包形式组装。

应用客户端模块，包含了class文件，应用客户端部署描述器文件（可选）并以Jar包形式组装。

资源适配模块，包含所有的Java接口，类，原生库，部署描述器（可选）。组合在一起，实现JavaEE的Connector架构。同样是以Jar包方式组装，但是文件以.rar扩展名结尾。 

就和War包包含一个web.xml的部署描述器，一个Ear包文件包含一个名字为application.xml的文件。它是一个必要的打包列表，告诉Java EE服务器Ear包种包含什么东西，以及去哪里找。

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<application xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee             http://java.sun.com/xml/ns/j2ee/application_1_4.xsd"
             version="1.4">
    <display-name>JBossAtWorkEAR</display-name>
    <module>
        <web>
            <web-uri>webapp.war</web-uri>
            <context-root>jaw</context-root>
        </web>
    </module>
    <module>
        <java>common.jar</java>
    </module>
</application>
{% endcodeblock %}

application.xml中的元素都应该是自解释的。在告诉应用程序服务器每一个Jar包的名字，以及它的作用（功能）。

例如：
{% codeblock lang:xml %}
<module>
    <web>
        <web-uri>webapp.war</web-uri>
        <context-root>jaw</context-root>
    </web>
</module>
{% endcodeblock %}

<context-root>用来告诉你web应用站点名字，如果你是直接打成War包，那么War包的名字就是这个站点名。

总而言之，Ear包是由Java EE多个模块组成，相互配合，完成各自职责，形成完整的Java EE应用。

参考资料：

http://docs.oracle.com/javaee/6/tutorial/doc/bnaby.html

Jboss at work A practice guide(Chapter 3)(http://oreilly.com/catalog/jboss/chapter/ch03.pdf)

