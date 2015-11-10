---
layout: post
title: "通过DaoCloud配置Coding平台的持续集成服务"
date: 2015-11-07 13:14:19 +0800
comments: true
categories: 
---
昨天无意间搜索到一篇文章： http://blog.daocloud.io/daocloud_coding/ DaoCloud完成与Coding对接

这让我感到很兴奋，原因是我搜索的关键字是“coding.net 持续集成”。

有一个项目放在coding平台的私有库中，一直没有做持续集成，如果是放在github的公有库，倒是可以用TravisCI做，但是这个项目不是开源的，一直希望找到一个国内的类似TravisCI的服务，直到昨天。

##DaoCloud

![Alt text](http://blog.daocloud.io/wp-content/uploads/2015/06/Logo_2.jpg)

DaoCloud 是业界领先的企业级容器云平台和解决方案提供商，致力于以 Docker 为代表的容器技术，为企业打造面向下一代互联网应用的交付和运维平台，帮助客户实现云端持续创新，....，等等。就不帮别人打广告了。

对我来说最重要的是：    
DaoCloud 对接 GitHub、Coding、GitCafe 等国内外代码托管库，采用云端 SaaS 化服务，帮助开发者实现自动化持续集成测试和 Docker 容器镜像构建。DaoCloud 镜像构建服务基于全球分布式网路，构建速度极快，提供私有镜像存储空间，为容器化交付和跨团队合作奠定了基础。

###如何使用

其实它提供的持续集成服务和TravisCI有点类似，至少从使用者的角度，是这么理解的。

目前支持如下语言和服务：

语言：Golang、Python、Ruby、Java、Javascript（NodeJS）、PHP、C（gcc）   
服务：MySQL、Redis、MongoDB

服务指的意思是，比如：构建需要集成测试或者功能测试，那么它可以提供三种数据库供你使用。

和TravisCI类似，需要你提供一个daocloud.yml，格式如下：
{% codeblock lang:yml %}
image: daocloud/ci-golang:1.4

services:
    - mongodb
    - mysql
    - redis

env:
    - MYENV = "hello"

install:
    - echo $MYENV
    - echo "This is an install segment"
    - echo "Here, we usually run scripts to setup a base environment"
    - echo "For customized base image, you need to install git here unless you have git installed in your base image"
    - echo "e.g., apt-get install -y git-core"

before_script:
    - echo $MYENV
    - echo "This is an before_script segment"
    - echo "Here, we usually run scripts to prepare our test"

script:
    - echo $MYENV
    - echo "This is an script segment"
    - echo "Run test cases here"
    - echo ""
    - echo "Below shows how to use services, mongodb/mysql/redis are the hostnames of services"
    - ping -c 2 mongodb
    - ping -c 2 mysql
    - ping -c 2 redis
{% endcodeblock %}

image是指定构建运行的环境镜像，除了上面的go语言，还有其他node，java，php，ruby等，在文章底部的参考资料中可以找到。

###服务

service也就是你需要的数据库服务了，举个例子，假设，你指定了mysql。那么它提供的mysql的配置如下：
{% codeblock lang:properties %}
Version：MySQL 5.5
Docker Link Alias: mysql
Host: mysql
Port: 3306
UserName: root
Password: 不设密码
Default Instance: test
{% endcodeblock %}

所以，项目中的测试在连接数据库的时候，也需要对应配置，否则就连接不上。

###执行顺序

设置环境变量。   
执行 install 脚本。   
克隆源代码，切换到对应的提交。   
执行 before_script 脚本。   
执行 script 脚本。  

script脚本，也就是你的构建执行命令了，比如：./gradlew clean build

配置完成后，提交代码，就可以触发CI构建。


##微信提醒功能

DaoCloud还可以和微信绑定，提供构建状态提醒，这样就不需要Build TV，直接手机微信提醒。

##结束语

我的项目花费了大概半天的时间，就配置成功了，当然这包括项目本身的一些配置，与平台无关。

参考资料：    
1.http://help.daocloud.io/features/continuous-integration/daocloud-yml.html