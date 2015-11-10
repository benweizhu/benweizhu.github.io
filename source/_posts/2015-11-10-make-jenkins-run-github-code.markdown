---
layout: post
title: "配置Jenkins运行Github的仓库代码构建"
date: 2015-11-10 13:38:27 +0800
comments: true
categories: 
---

用了这么久的CI服务应用，Jenkins， Go pipeline，还没有自己尝试搭建一个。今天花点时间在本地搭建了Jenkins。

##安装

以Mac版本为例：

打开： https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins ，下载Mac版本。

安装完成之后，会直接启动 http://localhost:8080 ，也就是说，Jenkins会默认启动8080端口作为服务端口。

在mac下，如果你想要切换端口，你需要这么做：

{% codeblock lang:python %}
sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
sudo defaults write /Library/Preferences/org.jenkins-ci httpPort 8443
sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist
{% endcodeblock %}
如果切换成https，如下：
{% codeblock lang:python %}
sudo defaults write /Library/Preferences/org.jenkins-ci httpPort 8443
sudo defaults write /Library/Preferences/org.jenkins-ci httpsKeyStore /path/to/your/keystore/file
sudo defaults write /Library/Preferences/org.jenkins-ci httpsKeyStorePassword <keystore password>
{% endcodeblock %}

你会发现，它们都有下面的两个命令，它们用来在mac中启动和关闭Jenkins服务：
{% codeblock lang:python %}
sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist
sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
{% endcodeblock %}

##安装Github插件和配置项目第一个项目

安装完成之后，你可以点击new item来新建一个构建项目，选择Freestyle project。

在Source Code Management中，你会发现，它CVS和Subversion的支持。没错，Jenkins默认并不支持Git配置。你需要安装Github Plugin。

回到Jenkins服务器的首页，打开Manage Jenkins，里面有Manage Plugins。在Available Plugin中搜索GitHub plugin，安装并重启Jenkins（页面上又重启的checkbox，点击一下即可）。

安装完成后，再次新建item，可以看GitHub project字段，Source Code Management中多了Git，Build Triggers中多了Build when a change is pushed to GitHub(但简单配置这个，还不能实现自动trigger，后面讲)。

添加在Build那一栏，选择添加Build Step，在本例中选择python。

我的项目是一个NodeJS项目，所以第一步是NPM install。

保存项目，回到项目栏，点击Schedule a build。

##构建执行失败和Jenkins用户

正如这个小标题，构建执行失败了，你会发现失败的原因是command npm not found。

NPM这个命令不存在，原因是Jenkins在执行那个脚本的时候是以Jenkins这个用户身份去执行，所以命令的PATH配置是不正确的。

## Prepare Env Before Run

在运行前配置Jenkins运行命令的环境，有两种方式：

1.直接在Prepare an environment for the run中配置     
2.安装Environment Inject插件，在Inject environment variables to the build process中配置

{% codeblock lang:python %}
PATH=$PATH:/usr/local/bin:/usr/local/Cellar/node/0.12.5/libexec/npm/bin/
{% endcodeblock %}

##A Blue Ball Appear

配置完成之后，再次执行，构建成功。但是构建的提示是一个蓝颜色的球。关于为什么是蓝色球： http://jenkins-ci.org/content/why-does-jenkins-have-blue-balls

这个当然不太习惯，好在要换成绿色球也很简单，安装插件：Green Ball Plugin。

##Pipeline插件

根据现代软件的开发方式，我们更习惯于构建CI以pipeline的方式呈现，pipeline中有不同的step，可以自动的trigger，也可以手动trigger，比如：部署到Test或者Production环境。

这个时候，你需要安装Jenkins的pipeline插件。安装完成之后，回到首页，点击tab上的加号，添加一个tab，你就可以看到pipeline选项。

新建一个pipeline的tab，需要你填写一些信息，比如：在pipeline页面一次显示多少个构建的pipeline，配置完成之后，你就可以看到pipeline页面了。

在pipeline页面，你可以添加一个step，其实也就是新建一个item，选项可以是free style的，也可以从别现有项目copy生成。

配置方式和新建一个item一样。

配置完成之后，你可以在前一个项目的Post-build Actions中添加Build Other Projects，选择Trigger only if build is stable或者其他。

这样就可以自动触发后续的step。

##Clone Workspace

你肯定会发现，既然新的step就是一个新的item，那么不是要重新check out一次代码，而且之前build的archive也不在了。

没错，这个时候，你需要安装Clone Workspace SCM Plugin，安装完成之后，你需要做两件事情：

1.在upstream的项目中的Post-Build Actions中添加Achieve for Clone Workspace SCM   
2.在downstream的项目的Source Code Management中选择Clone Workspace

##关于Build when a change is pushed to GitHub

简单的添加这个选项还不行，需要你在github的webhook中进行配置，方法在下面的链接中，但是前提是，需要配置的jenkins有url，如果是像我这样在本地配置的，可能就会有问题。
http://thepracticalsysadmin.com/setting-up-a-github-webhook-in-jenkins/



