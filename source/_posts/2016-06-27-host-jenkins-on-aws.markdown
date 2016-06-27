---
layout: post
title: "在AWS上部署Jenkins"
date: 2016-06-27 20:57:25 +0800
comments: true
categories:
---
##注册AWS账号
https://aws.amazon.com

##通过信用卡验证
付款1刀

##选择Region
美国东部（弗吉尼亚北部）   
美国西部（俄勒冈）   
美国西部（加利福尼亚北部）   
欧洲（爱尔兰）   
欧洲（法兰克福）   
亚太区域（新加坡）   
亚太区域（东京）   
亚太区域 (首尔)   
亚太区域（悉尼）  
南美洲（圣保罗）

##AWS Identity and Access Management (IAM)
* 对您 AWS 账户的共享访问权限
* 精细权限
* 在 Amazon EC2 上运行的应用程序针对 AWS 资源的安全访问权限
* 等等

###创建用户

https://console.aws.amazon.com/iam/home#home

AWS分为根账号用户和IAM用户

在账户中创建与组织中的用户对应的各IAM用户，而不是与他人共享您的根账户凭证。IAM用户不是单独的账户；它们是您账户中的用户。每个用户都可以有自己的密码以用于访问AWS管理控制台。您还可以为每个用户创建单独的访问密钥，以便用户可以发出编程请求以使用账户中的资源。

可以将IAM用户组织为IAM组，然后将策略附加到组。这种情况下，各用户仍有自己的凭证，但是组中的所有用户都具有附加到组的权限。

1.我创建了一个Administrators的组，并赋予AdministratorAccess权限。   
2.创建一个用户，用户名：someuser，勾选：为每个用户生成访问密钥，用户需要访问密钥，以确保向 AWS 服务 API 提交安全的 REST 或查询协议请求。  
3.给用户分配用户组   
4.设置用户的安全证书管理，即创建用户名密码访问权限和第二验证策略   
5.下载用户密码安全凭证

要使用AWS管理控制台的用户必须通过特定于您账户的登录页面登录到您的AWS账户。您需要为您的用户提供他们用于访问登录页面的URL地址。凭证中包含URL： https://My_AWS_Account_ID.signin.aws.amazon.com/console/

###创建EC2实例（我等穷人只能用免费的）

Amazon Elastic Compute Cloud (Amazon EC2) 在 Amazon Web Services (AWS) 云中提供可扩展的计算容量。使用 Amazon EC2 可避免前期的硬件投入，因此您能够快速开发和部署应用程序。通过使用 Amazon EC2，您可以根据自身需要启动任意数量的虚拟服务器、配置安全和网络以及管理存储。Amazon EC2 允许您根据需要进行缩放以应对需求变化或流行高峰，降低流量预测需求。

1.用新用户登录： https://My_AWS_Account_ID.signin.aws.amazon.com/console/   
2.创建一个EC2实例  
3.选择一个Amazon系统映像（AMI），我选择Ubuntu的免费版   
4.选择一个实例类型（计算资源），免费版只有1G内存，实在不够用   
5.配置实例详细信息（我采用默认配置）   
6.选择存储大小，最大免费30G   
7.标签实例（我留空了）   
8.配置安全组，这里要注意了，只要配置SSH和TCP的8080端口（jenkins用）   
9.下载SSH证书

SSH证书是用于SSH登录服务器的：

{% codeblock lang:sh %}
ssh -i somepem.pem user@serveraddress
{% endcodeblock %}

##安装Jenkins

SSH登录到服务器上，然后运行下面的脚本：

{% codeblock lang:sh %}
wget -q -O - https://jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
{% endcodeblock %}

注意jenkins默认是8080端口，这就是上面安全组要配置的TCP的8080端口。

然后，可以通过ip地址+8080端口。

参考资料：    
1. http://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/concepts.html    
2. http://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/introduction.html   
3. http://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/AMIs.html   
4. https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu   
5. http://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html
