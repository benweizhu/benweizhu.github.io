---
layout: post
title: "Cordova探索之旅系列（二）"
date: 2014-01-18 15:28
comments: true
categories: 
---
在Cordova中有一个很重要的概念：**插件**。

插件会提供访问Cordova核心API的接口。

插件是一些附加的代码，它能够提供访问原生组件的接口。一般情况下，你都需要添加一些插件以启动Cordova设备级别的特性。

插件由官方和社区提供，可以在plugins.cordova.io上找到，当然还可以在命令行中去搜索插件。

从3.0之后，Cordova将所有设备的API都作为插件，并默认设置为是不启动的。

那么，如何添加插件呢？两种方式。

**第一种是使用CLI命令行。**

**第二种是使用更低级别的命令行Plugman。**

两个的区别在于，Plugman只能一次添加一个平台的插件，而**CLI命令行会添加所有平台的插件**。所以如果你只在单个平台上工作，使用Plugman就显得更合理。

###使用CLI命令行添加插件：

**1.添加插件**

{% codeblock lang:bash %}
cordova plugin add org.apache.cordova.camera
{% endcodeblock %}

**2.删除插件**

{% codeblock lang:bash %}
cordova plugin rm org.apache.cordova.camera
{% endcodeblock %}

**3.查看当前已有插件**

{% codeblock lang:bash %}
cordova plugin ls
{% endcodeblock %}

**4.根据关键字搜索插件**

{% codeblock lang:bash %}
cordova plugin search bar code
{% endcodeblock %}

**5.社区中会提供很多插件，但是如果它没有注册到registry.cordova.io，你可以通过仓库地址添加**

{% codeblock lang:bash %}
cordova plugin add https://github.com/apache/cordova-plugin-console.git
{% endcodeblock %}

###使用plugman添加插件：

首先你需要安装plugman

{% codeblock lang:bash %}
npm install -g plugman
{% endcodeblock %}

**1.添加一个插件**

{% codeblock lang:bash %}
plugman --platform <ios|amazon-fireos|android|blackberry10|wp7|wp8> --project <directory> --plugin <name|url|path> [--plugins_dir <directory>] [--www <directory>] [--variable <name>=<value> [--variable <name>=<value> ...]]
{% endcodeblock %}

通过plugman命令行将一个插件加到一个Cordova工程中。你至少要指定平台信息、Cordova项目位置。

**name**：插件的目录名称。它必须是存在于plugins_dirpath路径下或者在Cordova中有注册。

**url**：以"http://"或者"git://"开头的url，指向一个合法的git可克隆仓库，仓库中应该包含一个plugin.xml的文件，仓库中的内容可以被拷贝到plugins_dir路径下。

**path**：一个指向包含合法插件（包含plugin.xml文件）目录的路径。该路径下的内容可以被拷贝到plugins_dir。
其他参数：

**plugin_dir**：默认是指向<project>/cordova/plugins，当然也可以是任何包含每一个已获取插件的子目录。

**www**：默认指向项目的www目录，但是也可以指向Cordova项目的应用web asserts目录。

**variable**：允许在安装时指定某些变量，对于某些插件需要API key或者其他用户定义参数是有必要的。

下面是添加电池状态插件的安装命令行

{% codeblock lang:bash %}
plugman -d --platform android --project myProject –plugin org.apache.cordova.battery-status
{% endcodeblock %}

-d或者—debug参数，会帮助你打印出内部调试信息，帮助你跟踪具体信息。

**2.删除一个插件**

{% codeblock lang:bash %}
plugman --uninstall --platform <ios|amazon-fireos|android|blackberry10|wp7|wp8> --project <directory> --plugin <id> [--www <directory>] [--plugins_dir <directory>]
{% endcodeblock %}