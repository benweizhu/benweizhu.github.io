---
layout: post
title: "在Node中使用Express"
date: 2015-10-27 09:43:04 +0800
comments: true
categories: 
---
Node.js是基于Chrome V8引擎的JavaScript运行时。

https://nodejs.org/en/

Express是Node.js上最小的，灵活的web应用框架，给web和移动应用提供了一系列有用的特性。

http://expressjs.com/

##如何安装

首先确保你已经安装好Node.js，下面是mac环境下，安装node的一些简单步骤（安装路径，根据不同的系统有所不同，注意安装结束时的说明）：

{% codeblock lang:groovy %}
Go to https://nodejs.org/en/, and download the latest version of node.

install it and,

Node.js was installed at
   /usr/local/bin/node

npm was installed at
   /usr/local/bin/npm

Make sure that /usr/local/bin is in your $PATH.

Then, try

node -v
npm -v

to verify the node has been successfully installed on your PC.
{% endcodeblock %}

新建一个myapp文件夹，从命令行进入目录，运行 npm init 。

npm init命令会创建一个package.json的文件，关于这个文件里面具体每个字段的含义，请参考： https://docs.npmjs.com/files/package.json 。

npm init命令会提示你输入一段内容，不必管它，一路回车（enter）。只是要注意一点，entry point: (index.js)会提示你输入【入口点文件】，默认是index.js，你也可以改为app.js（我这里改为了app.js，之后都采用app.js作为入口文件）。

现在来安装express，输入：
{% codeblock lang:groovy %}
npm install express --save
{% endcodeblock %}

如果你有node相关的知识，或者看过我的这篇文章《用Grunt做JavaScript的构建》（ http://benweizhu.github.io/blog/2015/01/09/use-grunt-to-build-javascript/ ）中，对node和npm部分进行讲解。

应该知道，express的模块会安装到当前目录的node_modules文件夹中， express会保存到package.json的dependencies中。

express的安装部分现在就完成了。

##Hello World 向世界说你好

简单的让express跑起来，验证上面的步骤都是正确的，新建一个app.js文件，用编辑器打开它，然后拷贝下面这段代码：

{% codeblock lang:javascript %}
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

var server = app.listen(3000, function () {
  var host = server.address().address;
  var port = server.address().port;

  console.log('Example app listening at http://%s:%s', host, port);
});
{% endcodeblock %}
不用我解释，看到这段代码，你应该猜测的出来，它要干什么，这段代码会启动一个server，监听3000端口，接受一个“/”的uri作为请求路径，返回‘Hello World!’的字符串。

运行方式应用的方式：
{% codeblock lang:javascript %}
node app.js
{% endcodeblock %}

这个里面有用到一个require函数来或者express对象（函数对象），关于require，在《用Grunt做JavaScript的构建》也有简单介绍Node的模块系统。

##Express的应用生成器（generator）

无论是前端，后端还是构建工具，任何成熟的应用框架都不会让你从头写应用，都会提供一套基本的脚手架（scaffold），比如maven的archetype，前端的yeoman等等。

Express也有自己的generator来做一些初始化的工作。

安装generator：
{% codeblock lang:python %}
npm install express-generator -g
{% endcodeblock %}

-g 参数表示express-generator模块会安装到npm的全局环境，而不是当前目录，这样，所有的node应用都可以重用，而不需要重新下载。运行express -h可以查看帮助。

和之前helloworld中不同，使用generator就不需要手动运行npm init命令和建立app.js文件，直接运行express myapp，如下：

{% codeblock lang:python %}
express myapp

   create : myapp
   create : myapp/package.json
   create : myapp/app.js
   create : myapp/public
   create : myapp/public/javascripts
   create : myapp/public/images
   create : myapp/routes
   create : myapp/routes/index.js
   create : myapp/routes/users.js
   create : myapp/public/stylesheets
   create : myapp/public/stylesheets/style.css
   create : myapp/views
   create : myapp/views/index.jade
   create : myapp/views/layout.jade
   create : myapp/views/error.jade
   create : myapp/bin
   create : myapp/bin/www
{% endcodeblock %}

也许你会发现安装了express-generator，运行express时，还是会提示命令找不到，如果是mac环境，你需要查看$PATH中，npm的bin目录是否包含在里面，方法如下：

{% codeblock lang:python %}
echo $PATH

touch ~/.bash_profile; //如果以前没有创建过

open ~/.bash_profile

//将npm的bin目录放在原始的${PATH}后面
export PATH=${PATH}:/usr/local/Cellar/node/0.12.5/libexec/npm/bin/

source ~/.bash_profile
{% endcodeblock %}

可以参考： http://hathaway.cc/post/69201163472/how-to-edit-your-path-environment-variables-on-mac

安装成功之后，进入myapp目录，运行npm install，安装对应的依赖，运行DEBUG=myapp npm start命令，运行package.json中配置的start脚本命令来启动应用，进入 http://localhost:3000/ 就可以。

##关于路由

路由这个概念就很容易理解了，无论是前端的AngularJS还后端的SpringMVC或者Ruby on Rails，它们都有自己的一套请求和对应handler（Controller）的映射方式。

Express也不例外，用来指定应用程序如何响应客户端的特定请求，包含URI，请求的HTTP方法（GET，POST）。

Express中路由定义是通过app.METHOD(PATH, HANDLER)结构实现，app是express的一个实例对象，METHOD是HTTP请求方法，PATH就是请求的相对URI，HANDLER是一个回调函数来处理请求内容，具体例子看下面：

{% codeblock lang:javascript %}
/ respond with "Hello World!" on the homepage
app.get('/', function (req, res) {
  res.send('Hello World!');
});

// accept POST request on the homepage
app.post('/', function (req, res) {
  res.send('Got a POST request');
});

// accept PUT request at /user
app.put('/user', function (req, res) {
  res.send('Got a PUT request at /user');
});

// accept DELETE request at /user
app.delete('/user', function (req, res) {
  res.send('Got a DELETE request at /user');
});
{% endcodeblock %}

##静态文件处理

静态文件也就是，常说的“非服务器程序代码”，比如：图片，CSS，字体，JavaScript文件等。

Express有一套自己的内置中间件来处理它们，express.static，使用方式如下：

{% codeblock lang:javascript %}
app.use(express.static('public'));
{% endcodeblock %}

将目录的名字传递过去，如果它是作为static assets的目录，express.static就会把它们host起来。比如，你将图片，CSS和JavaScript放在名字是publish的目录。

这样，你就可以直接通过浏览器访问它们了。
{% codeblock lang:python %}
http://localhost:3000/images/kitten.jpg
http://localhost:3000/css/style.css
http://localhost:3000/js/app.js
http://localhost:3000/images/bg.png
http://localhost:3000/hello.html
{% endcodeblock %}

当然，如果你想使用多个目录，也是可以的：
{% codeblock lang:javascript %}
app.use(express.static('public'));
app.use(express.static('files'));
{% endcodeblock %}

如果你想使用自定义的路径，而不是默认的真是文件路径作为url的路径（我想你看了代码应该明白我的意思），可以像下面这样做：

{% codeblock lang:javascript %}
app.use('/static', express.static('public'));
{% endcodeblock %}

那么，你通过浏览器访问它们，就成这样了。
{% codeblock lang:python %}
http://localhost:3000/static/images/kitten.jpg
http://localhost:3000/static/css/style.css
http://localhost:3000/static/js/app.js
http://localhost:3000/static/images/bg.png
http://localhost:3000/static/hello.html
{% endcodeblock %}

路径问题一直都是所有Web框架最头痛的问题，所以express.static的路径定义，也取决于你在哪个位置启动Node进程。如果，你在另一个位置启动node，最好还是给对应的你想要载入的文件目录（比如，publish目录），加一个绝对路径。

{% codeblock lang:javascript %}
app.use('/static', express.static(__dirname + '/public'));
{% endcodeblock %}

参考资料：    
1.http://expressjs.com/starter/installing.html







