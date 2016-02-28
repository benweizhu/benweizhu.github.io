---
layout: post
title: "用Webpack构建和打包模块（JS，SCSS，HTML等）基础（译）"
date: 2016-02-28 10:40:36 +0800
comments: true
categories:
---
本文翻译自 http://blog.madewithlove.be/post/webpack-your-bags/ 的部分内容

译者：原文作者很幽默写了许多幽默的语句，但是为了节省时间，我就不翻译那些营养不多的句子，捡核心有用的内容翻译。

##什么是webpack？

Webpack是构建系统还是模块bundler。Well，两者都是，但不是说它两件事情都做，而是将两件事情合并在一起，一起做。Webpack没有构建你的assets，然后在单独打包你的模块，而是把assets也当做模块。

更准确的说，它没有构建Sass文件，优化图片，并把它们放在一边后，再打包你的模块，然后把模块放在另一边，相反，你将拥有这些：

{% codeblock lang:javascript %}
import stylesheet from 'styles/my-styles.scss';
import logo from 'img/my-logo.svg';
import someTemplate from 'html/some-template.html';

console.log(stylesheet); // "body{font-size:12px}"
console.log(logo); // "data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5[...]"
console.log(someTemplate) // "<html><body><h1>Hello</h1></body></html>"
{% endcodeblock %}

所有的你的assets都会被当做是一种模块，然后被导入，修改，操作，并最终打包到最后的bundle文件中。

当然为了实现上面说的这些事情，需要在Webpack的配置中注册loader。Loader是一些小的插件，基本上就是说“当你遇到某种类型的文件时，做这个操作”。比如：
{% codeblock lang:javascript %}
{
  // When you import a .ts file, parse it with Typescript
  test: /\.ts/,
  loader: 'typescript',
},
{
  // When you encounter images, compress them with image-webpack (wrapper around imagemin)
  // and then inline them as data64 URLs
  test: /\.(png|jpg|svg)/,
  loaders: ['url', 'image-webpack'],
},
{
  // When you encounter SCSS files, parse them with node-sass, then pass autoprefixer on them
  // then return the results as a string of CSS
  test: /\.scss/,
  loaders: ['css', 'autoprefixer', 'sass'],
}
{% endcodeblock %}

最终在食物链的底层，所有的loader都会返回String。这样Webpack会把他们wrap到JavaScript模块中。比如每个Sass文件在被loader给transform之后，它可能看上去像这样：

{% codeblock lang:javascript %}
export default 'body{font-size:12px}';
{% endcodeblock %}

##为什么要这样做？

一旦你知道了Webpack做了什么，第二问题一定是：这样做有什么好处？“Images and CSS? In my JS? What the hell man?”。Well，这样想：长期以来，我们都被告知要将所有的东西都集中到一起，减少我们的HTTP请求。

这样也导致一个不好的情况，今天大部分人都讲所有的assets打包到一个app.js文件中，然后在所有的页面都加载它。这意味着对于任何一个页面在大部分时间都在加载一大堆根本不需要的assets。如果你不这样做，那么你就需要手动的在某个页面加入这些东西，这样会导致要维护一大堆依赖，哪个页面需要这些依赖?

上面两个方法没有一个是正确的或者错误的。Webpack采取了一种折中方式（middleground） - 它不仅仅是构建系统或者打包工具，他是一个顽皮聪明的模块打包系统。一旦被合理配置，他会比你更了解你的技术栈，比你更清楚如何最优化它们。

##简单试用

我们来创建一个项目，并用Webpack构建/打包它。

{% codeblock lang:python %}
$ npm init
$ npm install jquery --save
$ npm install webpack --save-dev
{% endcodeblock %}

src/index.js
{% codeblock lang:javascript %}
var $ = require('jquery');
$('body').html('Hello');
{% endcodeblock %}

创建一个webpack.config.js文件。Webpack的配置文件就是JavaScript，但是需要export这个对象。

webpack.config.js
{% codeblock lang:javascript %}
module.exports = {
    entry:  './src',
    output: {
        path:     'builds',
        filename: 'bundle.js',
    },
};
{% endcodeblock %}

这里，entry告诉Webpack哪些文件作为应用程序的入口。它们是你的主要文件，是坐落在项目依赖的最顶部。然后，我们告诉它编译，并输出到build目录下的bundle.js中。接着，在index.html中配置引入bundle.js文件。

index.html
{% codeblock lang:HTML %}
<!DOCTYPE html>
<html>
<body>
    <h1>My title</h1>
    <a>Click me</a>

    <script src="builds/bundle.js"></script>
</body>
</html>
{% endcodeblock %}

运行webpack命令，如果一切顺利，你应该看到下面这个信息：

{% codeblock lang:python %}
$ webpack
Hash: 65ea4aab7f0755f7fc8e
Version: webpack 1.12.2
Time: 350ms
    Asset    Size  Chunks             Chunk Names
bundle.js  257 kB       0  [emitted]  main
   [0] ./src/index.js 53 bytes {0} [built]
    + 1 hidden modules
{% endcodeblock %}

这里你看到Webpack告诉你在bundle.js文件中包含入口文件(index.js)，已经一个隐藏的模块，jQuery。默认情况下，Webpack会隐藏不属于你的模块。如果你想显示它们，使用--display-modules标识符。

{% codeblock lang:python %}
$ webpack --display-modules
bundle.js  257 kB       0  [emitted]  main
   [0] ./src/index.js 53 bytes {0} [built]
   [1] ./~/jquery/dist/jquery.js 248 kB {0} [built]
{% endcodeblock %}

你也可以运行webpack --watch，他会自动watch你的文件，并在有需要时重新编译。

##配置你的第一个loader

还记得我们谈到过Webpack可以导入css和html等所有的东西？怎么做到的呢？Well，如果你和最近几年Web Component的步调一致(Angular 2, Vue, React, Polymer, X-Tag, etc.)。也许你已经听说过这个idea，与相互链接庞大的UI不同，Web Component是一系列可维护和自包含的，小的，可重用的UI片段（piece）。为了让组件可以真正的自包含，需要将所有它所需要的东西都打包进去。想象一个按钮组件：有一些HTML代码，需要一些JS代码来让组件可以交互，也许还需要一个样式。如果这些东西只有在需要的时候才被加载，就更好了。

让我们来写一个Button。首先，我假设大部分人现在都更习惯与用ES2015语法，我们来添加第一个loader：Babel。在Webpack中添加添加一个loader，你需要做两件事情：npm install {whatever}-loader，然后在Webpack的配置的module.loaders部分添加该loader。所以：

{% codeblock lang:python %}
$ npm install babel-loader --save-dev
{% endcodeblock %}

更新我们的配置：我们想要什么？想要Babel运行所有的以.js结尾的文件，但是Webpack会递归遍历所有的依赖，我们需要Babel避免运行在第三方代码中，比如jQuery，所以需要过滤一下。loader可以有include和exclude规则，可以是字符串，正则，回调等等。在我们的例子中，我们希望Babel仅仅在我们自己的文件上运行，所以我们只需要include我们的源文件目录即可：

{% codeblock lang:JavaScript %}
module.exports = {
    entry:  './src',
    output: {
        path:     'builds',
        filename: 'bundle.js',
    },
    module: {
        loaders: [
            {
                test:   /\\.js/,
                loader: 'babel',
                include: \_\_dirname + '/src',
            }
        ],
    }
};
{% endcodeblock %}

现在来重写index.js，用ES6的语法。

{% codeblock lang:JavaScript %}
import $ from 'jquery';

$('body').html('Hello');
{% endcodeblock %}

##第一个自包含的组件

现在来写一个小的Button组件，它会包含SCSS样式，HTML模板，和一些行为，所以我们会需要安装一些需要的东西。首先，我们需要Mustache，它是一个轻量级的模板引擎（译者：如果你不知道JavaScript模板引擎，可以查看这篇文章 http://benweizhu.github.io/blog/2015/10/28/js-template-engine/ ），这时我们也需要Sass和HTML的loader。而且，最终需要load的内容会从一个loader流向另一个loader，我们需要一个CSS loader来吹了Sass loader的处理结果，现在一旦我们有了自己的CSS，就有很多方法来处理它们，现在我们会使用一个叫做style-loader的loader，它会取出CSS片段，并把它动态的插入页面。

{% codeblock lang:python %}
$ npm install mustache --save
$ npm install css-loader style-loader html-loader sass-loader node-sass --save-dev
{% endcodeblock %}

现在有序的告诉Webpack让load的内容从一个loader流入另一个，我们简单的传入一系列loader，从右到左，用一个!(感叹号)分开。或者你也可以通过一个数组，并使用loaders属性而不是loader属性：
{% codeblock lang:javascript %}
{
    test:    /\.js/,
    loader:  'babel',
    include: \_\_dirname + '/src',
},
{
    test:   /\\.scss/,
    loader: 'style!css!sass',
    // Or
    loaders: ['style', 'css', 'sass'],
},
{
    test:   /\\.html/,
    loader: 'html',
}
{% endcodeblock %}

现在来写我们的Button组件：

src/Components/Button.scss
{% codeblock lang:css %}
.button {
  background: tomato;
  color: white;
}
{% endcodeblock %}
src/Components/Button.html

{% codeblock lang:html %}
<a class="button" href="{{link}}">{{text}}</a>
{% endcodeblock %}

src/Components/Button.js
{% codeblock lang:javascript %}
import $ from 'jquery';
import template from './Button.html';
import Mustache from 'mustache';
import './Button.scss';

export default class Button {
    constructor(link) {
        this.link = link;
    }

    onClick(event) {
        event.preventDefault();
        alert(this.link);
    }

    render(node) {
        const text = $(node).text();

        // Render our button
        $(node).html(
            Mustache.render(template, {text})
        );

        // Attach our listeners
        $('.button').click(this.onClick.bind(this));
    }
}
{% endcodeblock %}

无论何时被导入，在哪个上下文中运行，Button.js是100%自包含的，它会包含所有需要的工具，并正确的渲染，现在我们只需要直接在页面上渲染我们的Button即可。

{% codeblock lang:javascript %}
import Button from './Components/Button';

const button = new Button('google.com');
button.render('a');
{% endcodeblock %}

参考：http://blog.madewithlove.be/post/webpack-your-bags/
