---
layout: post
title: "将Nginx作为Tomcat的反向代理服务器"
date: 2016-05-21 21:15:08 +0800
comments: true
categories:
---
本例子，以mac作为主机：

##安装Nginx

通过HomeBrew安装Nginx。

{% codeblock lang:sh %}
$ brew install nginx
{% endcodeblock %}

然后，运行启动nginx。如果启动遇到问题，使用brew doctor查看下，有可能是没有link（brew link nginx），或者没有文件执行权限（chmod去改）。
{% codeblock lang:sh %}
$ nginx
{% endcodeblock %}

nginx启动默认是8080端口，所以到 http://localhost:8080 上测试下。mac上nginx.conf的位置在/usr/local/etc/nginx/nginx.conf，也可以通过brew info nginx查看。
因为我们要使用80端口，所以需要修改配置，如下：
{% codeblock lang:sh %}
server {
    listen       8080;
    server_name  localhost;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }
{% endcodeblock %}

改为：
{% codeblock lang:sh %}
server {
    listen       80;
    server_name  localhost;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }
{% endcodeblock %}
这个时候，需要sudo去启动nginx了。

{% codeblock lang:sh %}
$ sudo nginx
{% endcodeblock %}

访问 http://localhost 进行测试，可以看到Nginx的主页。

##安装Tomcat

https://tomcat.apache.org/download-80.cgi

运行bin下面的./startup.sh。同样如果没有执行权限，用chmod修改。
{% codeblock lang:sh %}
$ ./startup.sh
{% endcodeblock %}

默认是8080，访问 http://localhost:8080 可以看到Tomcat的主页。

##修改Nginx配置，通过proxy_pass转发80请求到8080
{% codeblock lang:sh %}
server {
        listen       80;
        server_name  localhost;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass http://127.0.0.1:8080;
            root   html;
            index  index.html index.htm;
        }
{% endcodeblock %}

此时，重启Nginx服务器:
{% codeblock lang:sh %}
sudo nginx -s stop
sudo nginx
{% endcodeblock %}

再次访问 http://localhost 进行测试，就会看到Tomcat的主页了。

参考资料：  
1. http://learnaholic.me/2012/10/10/installing-nginx-in-mac-os-x-mountain-lion/  
2. https://devtidbits.com/2015/12/08/nginx-as-a-reverse-proxy-to-apache-tomcat/
