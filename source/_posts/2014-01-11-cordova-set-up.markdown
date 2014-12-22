---
layout: post
title: "Cordova探索之旅系列（一）"
date: 2014-01-11 16:47
comments: true
categories: 
---
最早接触PhoneGap平台是在1年多之前，能够使用HTML、CSS和JavaScript跨平台来编写Android或者IOS设备程序，并且应用的核心代码不需要多少修改就可以移植，确实让我感觉的到它应该是未来移动设备开发的趋势。Web程序员，特别是前端程序员，能够轻松的通过Web技术来编写移动设备软件。

**但是当时给我带来的感觉是应用的响应速度太慢，提供的API不全，能够实现的功能不多。PhoneGap经过1年多的沉淀，如今过头再来看PhoneGap，它又是什么样的呢？有哪些进步呢？**

##Cordova是什么？

Apache Cordova是PhoneGap贡献给Apache后的开源项目，是从PhoneGap中抽出的核心代码，是驱动PhoneGap的核心引擎。

Cordova是一个设备API的集合，它允许手机开发者通过JavaScript去访问设备原生功能，例如相机，重力感应等。结合UI框架，例如jQuery Mobile，Dojo Mobile或者Sencha Touch，可以让开发者通过HTML，CSS和JavaScript开发手机应用。

当使用Cordova的API时，应用可以在没有任何原生代码（Java，Object-C等）的情况下构建。并且，虽然使用着Web开发技术，但是该应用却是在本机运行，而不是远程的Web应用）。

并且因为提供的JavaScript的API在多个设备平台都保持一致性并且基于web标准，所以，应用可以在几乎没有任何修改的情况下应用到各个不同的设备平台。

使用Cordova开发的应用仍然是使用平台的SDK打包，可以放置到每个设备平台的应用商店中。

Cordova提供了一套统一的JavaScript库，其背后使用平台对应的代码来驱动设备。Cordova支持的平台有：IOS，Android，Blackberry，Windows Phone，Palm WebOS，Bada和Symbian。

##如何开始Cordova编程?

**1.下载并安装node.js（如果你已经安装过，就不用了）**

http://nodejs.org/

**2.安装Cordova**

{% codeblock lang:bash %}
sudo npm install -g cordova
{% endcodeblock %}

**3.创建应用程序**

{% codeblock lang:bash %}
cordova create hello com.example.hello HelloWorld
{% endcodeblock %}

**4.添加应用平台**

这里添加的是Android应用程序

你可能会得到如下信息：

Error: ERROR : executing command ‘ant’, make sure you have ant installed and added to your path.

说明你需要安装ant，方法如下：

{% codeblock lang:bash %}
brew update
brew install ant
{% endcodeblock %}

然后添加应用平台

{% codeblock lang:bash %}
cd hello
{% endcodeblock %}
首先进入到hello目录下
{% codeblock lang:bash %}
cordova platform add android
{% endcodeblock %}

这里也可以删除一个应用平台
{% codeblock lang:bash %}
cordova platform rm android
{% endcodeblock %}

通过ls命令
{% codeblock lang:bash %}
cordova platforms ls
{% endcodeblock %}
可以查看你安装关于平台的信息，例如，我的是:

Installed platforms: android 3.3.0

Available platforms: blackberry10, firefoxos, ios

**5.构建应用程序**

在进行构建之前，先确保你的Android SDK配置好了。

需要下载Android的SDK，并设置Path到系统路径下。

安装Android SDK（Mac平台，下载解压缩就行）

配置SDK的Path到系统路径

{% codeblock lang:bash %}
touch ~/.bash_profile; open ~/.bash_profile
{% endcodeblock %}

在profile文件中加一句话：

export PATH=${PATH}:/Users/twer/Downloads/adt-bundle-mac-x86_64-20131030/sdk/platform-tools:/Users/twer/Downloads/adt-bundle-mac-x86_64-20131030/sdk/tools

记得要指定你自己的路径。

最后还要执行profile将它更新到你的系统path下。
{% codeblock lang:bash %}
source ~/.bash_profile
{% endcodeblock %}

然后你就可以开始构建了:
{% codeblock lang:bash %}
cordova build
{% endcodeblock %}

当然你也可以正对某一个平台构建：
{% codeblock lang:bash %}
cordova build android
{% endcodeblock %}

**6.在模拟器上运行**

{% codeblock lang:bash %}
cordova emulate android
{% endcodeblock %}

此时，模拟器会启动，并自动安装应用，效果应该如下：

![Jasmine](http://cordova.apache.org/docs/zh/3.1.0/img/guide/cli/android_emulate_install.png)

**7.实体机上运行**

将你的设备插到电脑上，然后运行命令：

{% codeblock lang:bash %}
cordova run android
{% endcodeblock %}

会将应用安装到你的手机上。


