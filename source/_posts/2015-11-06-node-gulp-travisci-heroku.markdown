---
layout: post
title: "在Node下通过TravisCI部署由Gulp启动服务的应用到云平台Heroku"
date: 2015-11-06 08:06:16 +0800
comments: true
categories: 
---
这篇博客的起源比较有意思，客户给我们出了一道前端的开发题目，实现一个满足某种需求的web应用程序，算是某种程度的面试或者能力检测。

开发的技术栈采用我比较熟悉的：

环境：Node    
脚手架，构建和依赖管理：Yoman，Gulp，Bower，NPM（Node包管理也算依赖管理吧）    
开发框架：AngularJS，Bootstrap    
测试框架和Runner：Karma，Protractor，Jasmine，Webdriver

当然还有些七七八八的JavaScript类库，这里就不罗列了。

开发时间大概用了不到一天，考虑到这些环境我都没安装，所以下载还是花了点时间的，基本的本地运行，单元测试，功能测试都完善了，本来想着已经差不多了，但作为一个在以构建，持续集成和持续交付自豪的公司（Build and CI is in our DNA）里工作的开发人员，好像还差点什么。

没错，就是**持续集成和部署到PreProduction环境或者Production环境**，之前没怎么用过Heroku，所以彻底完成还是足足花费了一天的时间，也就是从开发到上线用了两天，不过这其中踩了无数的坑。

关于开发以及构建的部分，我就不详细说明了，这与标题也不符合。

##TravisCI
![Alt text](https://travis-ci.com/img/travis-mascot-200px.png)

>Test and Deploy with Confidence
>Easily sync your GitHub projects with Travis CI and you’ll be testing your code in minutes!

TravisCI是一个免费的，可以和Github项目同步的持续集成服务器，对持续集成这个概念不懂的同学，请参考我司（我厂）高级咨询师腾云的翻译的Martin Fowler的文章《持续集成》 http://www.cnblogs.com/CloudTeng/archive/2012/02/25/2367565.html 。

其实，要使用TravisCI是非常简单的，假设你是Java的项目，且采用了Maven或者Gradle做构建，那么只需要在项目中添加一个.travis.yml的文件，在里面写上language: java，提交，并在TravisCI上将项目Sync打开，就可以开始构建了。可以参考资料： http://docs.travis-ci.com/user/languages/java/ 

但是在本例子中，采用的是Node环境，所以相对的配置就需要有所改变。基本配置和Java环境类似：
{% codeblock lang:yml %}
language: node_js
node_js:
  - "4.1"
  - "4.0"
  - "0.12"
  - "0.11"
  - "0.10"
  - "0.8"
  - "0.6"
  - "iojs"
{% endcodeblock %}
这里要注意的是Node的版本，如果你使用4.0以上版本，很有可能在TravisCI上会遇到，导致npm install失败：
{% codeblock lang:python %}
../node_modules/nan/nan.h:41:3: error: #error This version of node/NAN/v8 requires a C++11 compiler
{% endcodeblock %}

官方文档上并没有给出解决这个问题的办法，唯一的临时解决办法就是使用低于4的稳定版本，比如我使用是0.12。

对于Node项目，TravisCI默认执行：npm test命令来运行你的测试（官方翻译：测试套件）。

如果你查看了官方文档，项目采用Gulp做构建，它会告诉你还需要在.travis.yml文件中添加：
{% codeblock lang:yml %}
before_script:
  - npm install -g gulp
script: gulp
{% endcodeblock %}
如果添加script: gulp，TravisCI会运行gulp，而不会运行npm test命令，所以这里取决于你的项目构建（测试）运行方式。我这里采用的npm test，因为需要同时运行单元测试和功能测试，在我的配置中，gulp任务只是最优化打包应用，所以在.travis.yml我并没有这些配置。

官方参考文档： http://docs.travis-ci.com/user/languages/javascript-with-nodejs/#Using-Gulp

我的配置全部在package.json的Script中，主要原因是为了方便Heroku部署，这里之所以需要在npm install之后运行bower install是为了功能测试能够正常运行。

{% codeblock lang:javascript %}
"scripts": {
    "test": "gulp test & gulp protractor",
    "start": "./node_modules/.bin/gulp serve:dist",
    "postinstall": "./node_modules/.bin/bower install"
}
{% endcodeblock %}

到目前为止，TravisCI的配置就结束了，项目可以正常的在持续集成服务器（CI Server）上运行。

##Heroku
![Alt text](https://upload.wikimedia.org/wikipedia/en/thumb/a/a9/Heroku_logo.png/220px-Heroku_logo.png)

>Heroku (pronounced her-OH-koo) is a cloud application platform – a new way of building and deploying web apps.


Heroku是国外有名的云应用平台，旗下的产品有：   
Heroku Platform   
Heroku Postgres   
Heroku Redis   
Heroku Connect   
Heroku Enterprise

###注册和安装Toolbelt

首先，你需要注册Heroku的账号，然后安装Heroku的Toolbelt工具。     
可以参考： https://devcenter.heroku.com/articles/getting-started-with-nodejs#set-up

安装参考资料的提示，登录：

{% codeblock lang:python %}
heroku login
Enter your Heroku credentials.
Email: zeke@example.com
Password:
...
{% endcodeblock %}

###在Travis中配置Heroku
登录Heroku创建一个应用程序，名字你自己取（得小写字母）

进入到应用，在Deploy的tab里面，你会看到一个Connect to Github，你可以选择将哪个repository和该应用关联来实现自动部署或手动部署，但TravisCI的自动部署跟它没有关系，所以你不用管它。

你要做的是看这里： http://docs.travis-ci.com/user/deployment/heroku/

在.travis.yml中配置：
{% codeblock lang:yml %}
deploy:
  provider: heroku
  api_key:
    secure: "YOUR ENCRYPTED API KEY"
...    
{% endcodeblock %}
虽然，你看得懂文档，但不建议手动配置，建议Travis和Heroku的客户端都安装，然后在项目目录下运行：travis setup heroku，来自动配置.travis.yml文件。

可以参考的文档： http://blog.travis-ci.com/2013-07-09-introducing-continuous-deployment-to-heroku/

配置完成之后，.travis.yml文件大概和下面的相似：
{% codeblock lang:yml %}
language: node_js
node_js:
- '0.12'
deploy:
  provider: heroku
  api_key:
    secure: QFSD0pnNddlsdZ6Wm/...
  app: yourapplicationname
  on:
    repo: benweizhu/yourApplicationRepoName
{% endcodeblock %}

提交代码之后，TravisCI就会开始在构建完成之后，开始执行部署到Heroku。

这样就完了吗？错！！！前面已经踩过一些坑，但还不够坑。

##现在正式开始采坑

执行完上面的步骤，你会发现构建是绿的，并且显示：
{% codeblock lang:python %}
-----> Compressing... done, 65.3MB
-----> Launching... done, v28
       https://yourapplicationname.herokuapp.com/ deployed to Heroku
{% endcodeblock %}

当你通过URL打开应用时，就会出现Application Error的页面。这个时候，就要开始troubleshooting了。

首先，你需要在TravisCI上构建和部署的log，这个就不用我教了。

如果TravisCI上没有问题，那么，你就需要查看Heroku服务器上的log，方法如下：
{% codeblock lang:python %}
heroku logs --tail --app appname
{% endcodeblock %}
所有的出现问题/导致失败的原因都可以在log中看到。

**常见问题**

问题1：Heroku的Node环境启动时，运行npm start，所以，你需要配置好，package中的script命令来正确的启动服务器。你也可以配置Procfile文件，那么它就会执行文件中的命令：
{% codeblock lang:python %}
web: node node_modules/.bin/gulp serve:dist
{% endcodeblock %}

问题2：Heroku没有在全局（global）下安装gulp，所以项目的gulp需要安装在本地，在npm start的命令中也要有相应的配置，比如：gulp命令是执行本地的bin目录。

问题3：Heroku会先运行npm install，所以如果项目使用了BowerJS，那么在postInstall要进行bower install。

问题4：确保package中，dependencies的配置是正确的，很多情况下，我们都把依赖放在了devDependencies中，但在产品环境下，应该在dependencies下。

另外还有一些问题：可能出在Heroku的配置上，具体请参考**Heroku官方的troubleshooting**： https://devcenter.heroku.com/articles/troubleshooting-node-deploys#start-with-a-blank-slate

##结束语

整个项目是一个完整的JavaScript全栈项目，从需求，到开发，最后部署，花费两天时间，虽然辛苦，但是学到不少东西。


参考资料：   
1.http://docs.travis-ci.com/user/languages/javascript-with-nodejs/    
2.http://docs.travis-ci.com/user/deployment/heroku/    
3.http://www.sitepoint.com/deploying-heroku-using-gulp-node-git/
4.http://www.hygkui.com/2015/03/13/%E4%BD%BF%E7%94%A8Gulp-Node-Git%E9%83%A8%E7%BD%B2%E9%A1%B9%E7%9B%AE%E5%88%B0Heroku%E4%B8%8A/ 上面的中文版    
5.http://blog.travis-ci.com/2013-07-09-introducing-continuous-deployment-to-heroku/    
6.https://devcenter.heroku.com/articles/troubleshooting-node-deploys#start-with-a-blank-slate







