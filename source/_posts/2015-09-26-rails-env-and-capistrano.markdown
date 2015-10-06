---
layout: post
title: "Rails环境和Capistrano部署"
date: 2015-09-26 10:09:49 +0800
comments: true
categories: 
---

默认情况下，Rails 提供了三个环境：开发（development），测试（test）和生产（production）。这三个环境能满足大多数需求，但有时需要更多的环境。

假设有个服务器镜像了生产环境，但只用于测试。这种服务器一般叫做“交付准备服务器”（staging server）。要想为这个服务器定义一个名为“staging”的环境，新建文件 config/environments/staging.rb 即可。请使用 config/environments 文件夹中的任一文件作为模板，以此为基础修改设置。

在持续集成开发的场景下，除了开发环境（提供给Dev的验证环境），测试环境（提供QA的测试环境），一般还会有一个UAT环境（用户验收测试环境 - User Acceptance Test），它是离生产环境最近的一个预备环境，就像上面所说的staging。

通过Rails Server命令启动环境，默认是development。

{% codeblock lang:python %}
=> Booting Puma
=> Rails 4.2.0 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
Puma 2.11.1 starting...
* Min threads: 0, max threads: 16
* Environment: development
* Listening on tcp://localhost:3000
{% endcodeblock %}

切换不同的环境启动可以通过参数-e

{% codeblock lang:python %}
rails server -e production -p 4000
{% endcodeblock %}
