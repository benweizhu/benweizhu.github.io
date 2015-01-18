---
layout: post
title: "用Bower做JavaScript类库依赖管理"
date: 2015-01-15 19:29:54 +0800
comments: true
categories: 
---

使用Grunt做JavaScript的构建，npm为它解决了最大的问题，插件（提供各种各样的任务）。但是如果你是做前端的Web应用，那么就还需要很多东西，比如，MVX框架，JavaScript工具类库等，这些东西，我们在构建中称为依赖。而这时，我们就要借助另一个工具，**Bower**。  

Bower做的事情：   

*“Bower works by fetching and installing packages from all over, taking care of hunting, finding, downloading, and saving the stuff you’re looking for.” --- Bower*

而且，Bower针对前端应用做了优化。它拥有扁平的依赖关系树，对于每个包之请求一个版本，将页面加载的内容减到最低。

##如何使用

###安装

Bower处在NodeJs的生态圈里，它是Node的一个模块（包）。所以安装方法和普通node包安装方法一样。

{% codeblock lang:python %}
npm install -g bower
{% endcodeblock %}

其实，我也在纠结是安装在全局还是本地，网上众说风云。

###命令工具

bower下载下来之后，你会发现它和grunt cli很像，除了模块部分，还有命令文件，因为它还是一个命令行工具。

使用bower install安装需要的依赖：

{% codeblock lang:python %}
# registered package
$ bower install jquery

# GitHub shorthand
$ bower install desandro/masonry

# Git endpoint
$ bower install git://github.com/user/package.git

# URL
$ bower install http://example.com/script.js
{% endcodeblock %}

###bower.json

就和node一样，node有一个package.json来存储模块相关的信息，包括依赖。

bower也有自己的一个json文件，bower.json，通过命令bower init，可以在bower的提示下，完成对bower.json的填写。当运行bower init命令后，bower会询问你下面这些问题，来帮助你完成bower.json文件的初始化：

{% codeblock lang:python %}
? name: (hello_grunt)
? version: (0.0.0)
? description: hello
? main file:
? what types of modules does this package expose?: (Press <space> to select)
❯◯ amd
 ◯ es6
 ◯ globals
 ◯ node
 ◯ yui
? keywords: hello
? authors: (benweizhu <xxxx@gmail.com>)
? license: (MIT)
? homepage:
? set currently installed components as dependencies?: (Y/n)
? add commonly ignored files to ignore list?:
? would you like to mark this package as private which prevents it from being accidentally 
published to the registry?: (y/N) Y

{
  name: 'hello_grunt',
  version: '0.0.0',
  authors: [
    'benweizhu <xxxx@gmail.com>'
  ],
  description: 'hello',
  moduleType: [
    'node'
  ],
  keywords: [
    'hello'
  ],
  license: 'MIT',
  private: true,
  ignore: [
    '**/.*',
    'node_modules',
    'bower_components',
    'test',
    'tests'
  ],
  dependencies: {
    angularjs: '~1.3.8'
  }
}

? Looks good?: (Y/n)
{% endcodeblock %}

bower.json中要填写的信息包括：

1.**name (required):** The name of your package; please see Register section for how to name your package.    
2.**version: **A semantic version number (see semver).    
3.**main string or array: **The primary acting files necessary to use your package.   
4.**ignore array:** An array of paths not needed in production that you want Bower to ignore when installing your package.    
5.**keywords array of string: **(recommended) helps make your package easier to discover.   
6.**dependencies hash:** Packages your package depends upon in production. Note that you can specify ranges of versions for your dependencies.   
7.**devDependencies hash:** Development dependencies.   
8.**private boolean: **Set to true if you want to keep the package private and do not want to register the package in the future.

###安装并存储到bower.json

就和Grunt一样，bower也允许你将依赖下载下来，并存储到bower.json文件中
{% codeblock lang:python %}
bower install angularjs --save
{% endcodeblock %}

###bower中的有用命令

{% codeblock lang:python %}
bower list        --列出本地的包和可能的更新
bower cache clean --清除bower的依赖缓存
bower cache list  --列出bower的依赖缓存
bower update      --根据bower.json的内容将安装的包升级到最新版本
bower install     --安装bower.json中定义的依赖包
bower uninstall   --卸载已安装的依赖包
{% endcodeblock %}
更多内容，请查看：http://bower.io/docs/api/

###将bower和grunt集成

目前，github上面有很多grunt的插件可以将bower集成到grunt中。你可以去 https://www.npmjs.com/ 上搜索bower和grunt的关键字。

我这里使用的是：grunt-bower-task

安装很简单：npm install grunt-bower-task --save-dev

{% codeblock lang:json %}
{
    "name": "relations-front-end",
    "version": "0.0.1",
    "devDependencies": {
        "grunt": "~0.4.5",
        "grunt-bower-task": "^0.4.0",
        "grunt-contrib-jshint": "~0.4.0"
    },
    "scripts": {
        "test": "grunt"
    }
}
{% endcodeblock %}

{% codeblock lang:javascript %}
module.exports = function (grunt) {

    grunt.initConfig({
        jshint: {
            files: ['Gruntfile.js', 'src/**/*.js', 'test/**/*.js', '!src/js/libs/**/*.js']
        },
        bower: {
            install: {
                options: {
                    targetDir: './src/js/libs'
                }
            }
        }
    });

    grunt.loadNpmTasks('grunt-bower-task');
    grunt.loadNpmTasks('grunt-contrib-jshint');

    grunt.registerTask('default', ['bower:install', 'jshint']);

};
{% endcodeblock %}

targetDir: './src/js/libs'的作用是告诉task将bower下载的目标文件拷贝一份到哪个路径下

请参考：http://bower.io/docs/tools/





