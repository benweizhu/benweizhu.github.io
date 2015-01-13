---
layout: post
title: "用Grunt做JavaScript的构建"
date: 2015-01-09 21:22:27 +0800
comments: true
categories: 
---

就像Grunt官方网站上说的“为何要用构建工具？”

”一句话：自动化。对于需要反复重复的任务，例如压缩（minification）、编译、单元测试、linting等，自动化工具可以减轻你的劳动，简化你的工作。当你正确配置好了任务，任务运行器就会自动帮你或你的小组完成大部分无聊的工作。”

在Java世界里面，自动化工具或者说构建工具，我们见得的不少，并且已经很成熟，比如，Ant，Maven，Gradle。而在JavaScript的世界，我们同样需要一个工具，来帮助我们做一些重复并且需要频繁做的事情，比如，压缩（minification）、编译、单元测试、linting。

对于Web开发来说，在Single Page的Web应用开始流行之前，我们使用Ant，Maven，Gradle，同样也可以做压缩（minification）、编译、单元测试、linting，因为官方和社区提供了各种各样的插件。

但如果你只是在开发JavaScript的类库，或者静态站点，又或者现在流行的Single Page的Web应用，那么一个纯粹的针对JavaScript而开发的构建工具就拥有了它的存在价值。

所以，才有了Grunt作为JavaScript世界的构建工具出现在我们的视野中。

##Node

Grunt的第一版发布在2012年的3月，作者将它形容为“针对JavaScript项目的基于任务的命令行构建工具”

Grunt和Grunt插件是通过npm安装和管理的。那npm是什么？node的包管理器（a package manager for node）。那Node又是什么？

引入node.js官方的介绍：

"Node.js is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices."

没错，它不是一个JavaScript的类库或者应用，node.js是一个基于Google Chrome V8 JavaScript引擎的JavaScript运行平台（环境）。

看到这里，我们大概可以明白，要开始使用Grunt，首先要安装node.js，然后明白如何使用npm，但是要学会使用Grunt，你并不需要了解node.js的全部知识。

所以，首先要来安装node.js，移步到官方网站 http://nodejs.org/ ，下载node.js，当前官网的最新版本是v0.10.35。安装完成之后，可以用node -v命令验证你是否安装成功。

node.js本身就自带npm，所以你不需要单独安装npm，运行命令npm -v就可以查看npm的版本。

###Node的模块系统

在正式开始了解npm之前，我们需要了解一个关于node.js的技术知识：模块系统

node.js的模块系统是CommonJS标准的实现，它描述了一种简单的语法让JavaScript请求（导入）其他JavaScript程序进入到当前请求模块的上下文中。

而在node.js中，**每一个JavaScript文件都可以被看做是独立的模块**。

理解几个关键概念（对象或者函数），就可以很快理解CommonJS如何被使用。

**module** – 一个代表模块自身的对象。该模块对象包含一个关键的exports对象。就node.js，它还包括一些元信息，比如id，parent，children。

**exports** – 一个纯粹的JavaScript对象，它可以被扩展来向其他模块暴露方法。该exports对象，将会作为require函数被调用后的返回结果。

**require** ­– 用来引入模块的函数，返回相关的exports对象。

math.js
{% codeblock lang:js %}
exports.add = function() {
    var sum = 0, i = 0, args = arguments, l = args.length;
    while (i < l) {
        sum += args[i++];
    }
    return sum;
};
{% endcodeblock %}

increment.js
{% codeblock lang:js %}
var add = require('math').add;
exports.increment = function(val) {
    return add(val, 1);
};
{% endcodeblock %}
当require('math')执行，它会返回math.js模块对应的exports对象，于是就可以调用add方法。

program.js
{% codeblock lang:js %}
var inc = require('increment').increment;
var a = 1;
inc(a); // 2
 
module.id == "program";
{% endcodeblock %}
上面同理

更多关于CommonJS的内容，请移步 http://javascript.ruanyifeng.com/nodejs/commonjs.html

##NPM

基本了解过Module系统的概念之后，我们再回过头来看npm。

对于npm而言，它并不是为构建而存在，它是node的包管理工具，只是它的存在恰恰是一个成功构建工具必须要解决的一个问题, 提供真正价值的插件。npm可以将node.js的包发布到npm的仓库里，同样的，仓库里的一个包，也可以被任何知道它名字的应用安装和使用。

###NPM中包和模块的关系

接下来的问题是，怎么样才算是一个包呢，它可以被上传和下载？

“What is a package?

A package is:

a) a folder containing a program described by a package.json file

b) a gzipped tarball containing (a)

c) a url that resolves to (b)”

根据上面的解释，一个包存在三种表现形式。

我们再来看一下，node.js中一个模块是怎么定义的？怎么样它就是一个模块？

“What is a module?

A module is anything that can be loaded with require() in a Node.js program. The following things are all examples of things that can be loaded as modules:

a) A folder with a package.json file containing a main field.

b) A folder with an index.js file in it.

c) A JavaScript file."

如果你仔细观察，两个解释中的定义a是非常相似的。一个包可以是一个含有package.json文件的文件夹，一个模块可以是一个含有package.json文件或者index.js文件的文件夹。所以，为了让别人在它的程序中使用你的包，它必须使用require函数来导入，顾名思义，你的包也必须是一个模块。

###npm install

那么如何用npm来下载一个模块或者包呢？

npm intall命令，它的目的只有一个就是从npm仓库下载模块。

安装一个node模块时，有两种安装方式，一种是全局方式（global），一种是本地方式（local）。如果被下载模块是在另一个模块或者应用中使用，就应该下载到本地，如果这个模块是命令行工具（例如，grunt cli，这个之后会介绍），那么就可以放在全局。

全局方式，就跟全局函数一样，意味着这个模块在安装之后，在任何位置都可以使用它。

比如：

{% codeblock lang:python %}
npm install -g grunt-cli
{% endcodeblock %}

如果是mac系统，被下载的模块会放在usr/local/lib/node_modules 如果是Windows系统，则在C:\Users\AppData\Roaming\npm

你可以通过命令，修改存储的位置，毕竟你不会太希望缓存库放在用户文件夹下:

{% codeblock lang:python %}
npm config set prefix C:\Dev\Software\npm-repository\npm --global

npm config set cache C:\Dev\Software\npm-repository\npm-cache --global
{% endcodeblock %}

本地方式，顾名思义，是给指定模块或者应用使用，比如npm install grunt-contrib-uglify，那么uglify的存放路径到底在哪？它会存放在命令执行的工作路径上吗？不一定，它是有些规则的：

这个新下载的模块会被放置在它认为的当前node包的node_modules文件中，那它怎么决定哪个是当前的node包呢？

npm会从当前工作路径开始向上遍历，寻找模块描述文件package.json。如果找到了，则包含该描述文件的文件夹就会被当做包的根目录。如果向上遍历没有找到，它就会认为还没有package.json文件被创建，那么当前文件夹就会被当做包的根目录，并将模块下载到node_modules文件夹中。

这种判断在什么位置存放node_modules文件夹的模式是和node中模块系统的require函数寻找导入模块的策略是相匹配的。

当我们想要使用一个新安装的node模块时，我们通过require函数导入，传入的参数是模块的名字，而不是文件的名字。require函数会在当前路径寻找node_modules目录，如果没有找到，则会去它的父目录寻找。它会一直搜索，知道到达文件系统根目录。

##Grunt

谈了这么多，还没有到今天的主题Grunt，那么Grunt和Node，npm是什么关系呢？答案是：Grunt是Node.js中的一个模块，可以通过npm下载并安装。

所以，如果你要安装Grunt，就和安装其他node模块一样，npm install grunt。

###Grunt的插件与Node模块

在前面提到，npm为Grunt成为一个成功的构建系统做了很大的贡献，它是一个构建系统的插件仓库。

比如说，你希望构建的过程中，做JavaScript的CheckStyle。你需要安装：grunt-contrib-jshint插件（时刻记住，它就是node模块）

比如说，你希望构建的过程中，做JavaScript的文件压缩。你需要安装：grunt-contrib-uglify插件(node模块)

等等

难道手动的去一个个install吗？当然不是。

我们的项目，既是一个Grunt的工程，也是一个node模块，所以其中的package.json是有用的。

模块中所有的依赖模块（在grunt中，就是需要的插件），都可以在package.json中声明，而你只需要一个命令npm install，就可以全部下载并安装。

比如，在package.json中这么写：

{% codeblock lang:js %}
{
  "name": "my-project-name",
  "version": "0.1.0",
  "devDependencies": {
    "grunt": "~0.4.1",
    "grunt-contrib-jshint": "~0.6.0",
    "grunt-contrib-nodeunit": "~0.2.0",
    "grunt-contrib-uglify": "~0.2.2"
  }
}
{% endcodeblock %}

然后，你运行npm install即可。

###Grunt CLI

那么，为了看到Grunt跑起来，看到最后的build success字样，还差什么呢？构建脚本，Gruntfile.js，这个是当然。但，即便我不知道怎么写，我可以拷贝一个过来。可是，好像还是差点什么东西，Ant，Maven，Gradle都分别有对应运行脚本的命令ant，mvn和gradle（或者gradlew）。那么Grunt的是什么？

你可以打开当前工程的node_modules目录，看看里面grunt的文件夹，里面除了js文件就是json等描述性文件。好像这些都不是可以运行的命令文件。

还差点什么？

在前面，我们有引入一个命令npm install -g grunt-cli来安装grunt的命令行工具，你可以发现它是在全局范围内装了一个名字叫grunt-cli的模块。

没错，就是它了。Grunt在0.4版本以后，被分割为三部分：grunt、grunt-cli和grunt-init。

grunt-cli就是为了可以让你可以使用grunt命令，因此，当你在下载grunt-cli模块时，除了在全局的node_modules里面有grunt-cli模块，在npm文件夹下还有一个新下载的命令文件（例如，Windows是grunt.cmd），而且全局的node目录是放在系统path下的，因此你可以在任何位置使用这个命令。

那么，除了是为了提供grunt命令给你使用，它还有一个特别的意义。

Grunt CLI另一个很简单目的：运行离某个Gruntfile.js文件（Grunt里的build脚本）最近的某个版本的Grunt。换句话说，Grunt CLI就是类似Gradle中的Wrapper，是一个包装器。允许你在一台机器上给不同的应用使用不同的Grunt版本。

而Grunt CLI又是安装在全局下的，所以你在任何一个位置都可以运行grunt命令，它会去找里当前构建文件Gruntfile.js最近的Grunt。

我甚至怀疑它是不是就是借鉴的Gradle的包装器，只不过，Gradle的包装器在创建之前，需要创建的人先装一个支持包装器的Gradle版本，然后生成Gradlew，提交代码及对应的Gradlew命令，那么后面的人，就可以直接check out并运行，而不需要先安装Gradle，而这里，需要大家都安装Grunt CLI，它就是包装器，不需要某个人先创建。

###最后一百米，如何完整的跑一次Grunt构建

有了它，剩下来的就是编写脚本文件Gruntfile.js，我个人觉得，它是整个Grunt生态系统中最容易理解的，因为它是配置。

看一个最简单的完整例子：

package.json
{% codeblock lang:js %}
{
  "name": "hello-grunt",
  "version": "0.1.0",
  "devDependencies": {
    "grunt": "~0.4.1",
    "grunt-contrib-jshint": "~0.6.0"
  }
}
{% endcodeblock %}

Gruntfile.js
{% codeblock lang:js %}
module.exports = function(grunt) {

  // Project configuration.
  grunt.initConfig({
    jshint: {
      files: ['gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
      options: {
        globals: {
          jQuery: true,
          console: true,
          module: true
        }
      }
    }
  });

  grunt.loadNpmTasks('grunt-contrib-jshint');

  grunt.registerTask('default', ['jshint']);

};
{% endcodeblock %}

files: ['gruntfile.js', 'src/**/*.js', 'test/**/*.js'] 定义了所有需要做lint的的文件

options 用来配置JSHint(文档在这里http://www.jshint.com/docs/)

grunt.loadNpmTasks('grunt-contrib-jshint') 用来加载包含 "jshint" 任务的插件。

grunt.registerTask('default', ['jshint']) 定义默认被执行的任务列表，即直接运行grunt命令，默认的任务是什么。

现在你可以运行一下。

首先运行npm install，安装在package.json中的模块。

然后运行grunt命令，你也可以显示的指定运行某个任务 grunt jshint

运行结果：

{% codeblock lang:python %}
Running "jshint:files" (jshint) task
>> 1 file lint free.

Done, without errors.
{% endcodeblock %}

Grunt官方提供了许多插件来满足一般JavaScript项目或者Web项目前端部分需要的任务，比如jshint，less，sass等。

当你在grunt.initConfig中配置完对应的task之后，你就可以load和register对应的task到grunt中。

当然，你也可以写自己的task。下面摘自官方的首页例子：

{% codeblock lang:javascript %}
module.exports = function(grunt) {

  // A very basic default task.
  grunt.registerTask('default', 'Log some stuff.', function() {
    grunt.log.write('Logging some stuff...').ok();
  });

};
{% endcodeblock %}

本篇关于Grunt的文章到这里就结束了，目的已经达到，如果你对更多关于如何写package.json和Gruntfile.js。可以去nodejs和grunt官方网站查看更多文档。

总结，学习Grunt，本身不难，因为它是Cofiguation Over Code，这个的原则类似Maven。但是需要首先理解它的生态系统node.js。

PS：我知道，现在，越来越多的人也在讨论到底Gulp（一个新的JavaScript构建工具，同样基于Node.js，提倡Code Over Configuration）该不该替代Grunt，或者它们的优缺点。但这对于你去了解一个构建工具并不影响。


参考资料：

1.http://www.gruntjs.net/docs/getting-started/     
2.Book: Getting started with Grunt: The JavaScript Task Runner
