---
layout: post
title: "全民安全站Let's Encrypt配置NGINX"
date: 2016-05-29 17:40:12 +0800
comments: true
categories:
---
ThoughtWorks2016年4月份最新发布的技术雷达对Let’s Encrypt项目的介绍：

从2015年，12月开始，Let’s Encrypt项目从封闭测试阶段转向公开测试阶段，也就是说用户不再需要收到邀请才能使用它了。Let’s Encrypt为那些寻求网站安全的用户提供了一种简单的方式获取和管理证书。Let’s Encrypt也促使安全和隐私前进了一大步，而这一趋势已经随着ThoughtWorks和我们许多使用其进行证书认证的项目开始了。

Let's Encrypt发布最新数据，至今该项目已经颁发了超过300万份证书——300万这个数字是在5月8日-9日之间达成的。Let's Encrypt是为了让HTTP连接做得更加安全的一个项目，所以越多的网站能够加入进来，则整个互联网也会变得更加安全。

本文是一个简单的Tutorial，告诉你怎样在NGINX服务器配置SSL实现网站的https：

##登录到你的服务器上
保证你申请SSL的域名和服务器的IP是一致的，即域名确实是解析到你的服务器上的，可以使用nslookup命令查询。
{% codeblock lang:sh %}
nslookup www.yourwebsite.com
{% endcodeblock %}
Let’s Encrypt在给你分配证书时，会检查你所在的服务器是否和域名解析的服务器一致。

##配置基本的Nginx设置：
{% codeblock lang:sh %}
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  yourwebsite.com;

    location ^~ /.well-known/acme-challenge/ {
       default_type "text/plain";
       root     /var/www/letsencrypt;
    }

    location = /.well-known/acme-challenge/ {
       return 404;
    }
    ... 其他配置，例如
    location / {
      proxy_pass http://localhost:8080;
    }
}
{% endcodeblock %}
这里location配置了一个/.well-known/acme-challenge/路径，里面host了简单文件，我这里host了一个简单的html文件。原因是你必须证明，你拥有所请求的证书的域名。因为 Let's Encrypt要求你host一些文件。

##使用certbot申请证书

克隆certbot仓库：https://github.com/certbot/certbot

{% codeblock lang:sh %}
sudo apt-get install -y git
sudo git clone https://github.com/certbot/certbot /opt/letsencrypt
/opt/letsencrypt/letsencrypt-auto
{% endcodeblock %}

运行certbot提供的脚本获取证书

{% codeblock lang:sh %}
export DOMAINS="yourdomain.here,www.yourdomain.here"
export DIR=/var/www/letsencrypt
/opt/letsencrypt/letsencrypt-auto certonly --server https://acme-v01.api.letsencrypt.org/directory -a webroot --webroot-path=$DIR -d $DOMAINS
{% endcodeblock %}
注意这里指定了一个webroot-path，他应该和上面well-known配置的root一样。

运行成功之后，你会看到下面这个提示
{% codeblock lang:sh %}
IMPORTANT NOTES:
- Congratulations! Your certificate and chain have been saved at
  /etc/letsencrypt/live/letsecure.me/fullchain.pem. Your cert will
  expire on 2016-XX-XX. To obtain a new version of the certificate in
  the future, simply run Let's Encrypt again.
- Your account credentials have been saved in your Let's Encrypt
  configuration directory at /etc/letsencrypt. You should make a
  secure backup of this folder now. This configuration directory will
  also contain certificates and private keys obtained by Let's
  Encrypt so making regular backups of this folder is ideal.
- If you like Let's Encrypt, please consider supporting our work by:

Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
{% endcodeblock %}

##配置https证书

{% codeblock lang:sh %}
ssl_certificate /etc/letsencrypt/live/yourdomain.here/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/yourdomain.here/privkey.pem;
{% endcodeblock %}

完整配置
{% codeblock lang:sh %}
server {
    listen 443 ssl;
    server_name yourdomain.here www.yourdomain.here;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    ssl_prefer_server_ciphers On;
    ssl_certificate /etc/letsencrypt/live/yourdomain.here/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.here/privkey.pem;
    ssl_session_cache shared:SSL:128m;
    add_header Strict-Transport-Security "max-age=31557600; includeSubDomains";
    ssl_stapling on;
    ssl_stapling_verify on;
    location / {
      proxy_pass http://localhost:8080;
    }
}
{% endcodeblock %}

配置80端口跳转：
{% codeblock lang:sh %}
return 301 https://$server_name$request_uri;
{% endcodeblock %}
完整配置
{% codeblock lang:sh %}
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  yourwebsite.com;

    location ^~ /.well-known/acme-challenge/ {
       default_type "text/plain";
       root     /var/www/letsencrypt;
    }

    location = /.well-known/acme-challenge/ {
       return 404;
    }

    location / {
      return 301 https://$server_name$request_uri;
    }
}
{% endcodeblock %}

##证书90天过期
Let’s Encrypt证书会在90天后过期，需要配置脚本自动更新证书。
{% codeblock lang:sh %}
#!/bin/sh
# This script renews all the Let's Encrypt certificates with a validity < 30 days

if ! /opt/letsencrypt/letsencrypt-auto renew > /var/log/letsencrypt/renew.log 2>&1 ; then
    echo Automated renewal failed:
    cat /var/log/letsencrypt/renew.log
    exit 1
fi
nginx -t && nginx -s reload
{% endcodeblock %}

开启定时任务Cron
{% codeblock lang:sh %}
sudo crontab -e
{% endcodeblock %}

编辑任务内容
{% codeblock lang:sh %}
@daily /path/to/renewCerts.sh
{% endcodeblock %}

参考资料：    
1.https://letsecure.me/secure-web-deployment-with-lets-encrypt-and-nginx/    
2.https://community.letsencrypt.org/t/how-to-nginx-configuration-to-enable-acme-challenge-support-on-all-http-virtual-hosts/5622     
3.https://letsencrypt.org/getting-started/
