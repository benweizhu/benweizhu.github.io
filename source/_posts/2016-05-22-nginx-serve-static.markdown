---
layout: post
title: "用Nginx服务静态文件"
date: 2016-05-22 11:58:09 +0800
comments: true
categories:
---

使用Nginx服务静态文件的原因很简单：快。

上一篇文章中 http://benweizhu.github.io/blog/2016/05/21/use-nginx-as-reverse-proxy-for-tomcat/ ，我们使用Nginx作为Tomcat服务器的反向代理服务器，这样比在Tomcat直接配置80端口更加方便。

但是，却没有很好的利用Nginx服务器的特点，即 NGINX | High Performance Load Balancer, Web Server, & Reverse等。

我们可以使用Nginx来服务器静态文件，使用Tomcat来处理JSP的动态文件。配置非常简单，分别使用下面两个配置：

##403错误：添加Nginx访问文案件的权限
nginx配置中有一个user选项，默认，nginx配置的是nobody。 http://nginx.org/en/docs/ngx_core_module.html#user
{% codeblock lang:sh %}
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;
{% endcodeblock %}
需要将它改为可以访问Tomcat服务器上文件的用户和用户组
{% codeblock lang:sh %}
user  yourusername yourusergroup; #比如 benweizhu stuff
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;
{% endcodeblock %}

##404错误：使用Alias来配置Static文件路径

{% codeblock lang:sh %}
location /static/ {
  autoindex  on;
  alias /Documents/servers/tomcat/apache-tomcat-8.0.30/webapps/someapp/static; #完整路径
}
{% endcodeblock %}


参考资料：     
1. https://nu2ruby.wordpress.com/2010/04/13/nginx-getgrnam-presmini-failed/     
2. http://stackoverflow.com/questions/16808813/nginx-serve-static-file-and-got-403-forbidden     
3. http://stackoverflow.com/questions/1474374/nginx-doesnt-serve-static     
