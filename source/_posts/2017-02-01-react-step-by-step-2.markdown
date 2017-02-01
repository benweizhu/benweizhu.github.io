---
layout: post
title: "一步一步做React(二) - CSS Modules"
date: 2017-02-01 22:44:09 +0800
comments: true
categories:
---
基于上次的内容 [《一步一步做React(一) - Hello React》](http://benweizhu.github.io/blog/2017/01/17/react-step-by-step-1/)，我们继续搭建一个React项目的工程实践 - CSS Modules。

### CSS Modules概念

Github文档 - [CSS Modules](https://github.com/css-modules/css-modules)

**CSS Modules — Local Scope 局部作用域**

CSS Modules里面最基础也是最重要的概念 - 局部作用域。

```
.backdrop {}
.prompt {}
.pullquote {}
```
定义在style.css中的样式都存在于一个组件内的局部作用域，不会污染全局的样式。
```
import styles from "./style.css"

const Component = props => {
  return (
    <div className={styles.backdrop}>
      <div className={styles.prompt}>
      </div>
      <div className={styles.pullquote}>
      </div>
   </div>
  )
}
```
怎么解释？如何不会污染全局的样式？答案是唯一的命名。现在，我们先来看是怎么配置的。

### 用webpack处理样式

**style-loader | css-loader | sass-loader**

给webpack.config.js中的module添加一个新的loaders配置，如下：

```
module: {
  loaders: [
    {
      test: /\.jsx?/,
      include: APP_DIR,
      loader: 'babel',
    },
    {
      test: /\.scss$/,
      loaders: ["style", "css?modules&importLoaders=1&localIdentName=[name]__[local]___[hash:base64:5]", "sass"],
    },
  ],
},
```
webpack的loader是用来预处理由require()或者import加载文件，比如：

```
import styles from "./style.scss"
```
在上面的webpack配置中，针对*.scss文件，配置了一个loader的管道，顺序是：

sass-loader => css-loader => style-loader

sass-loader顾名思义用来编译SCSS到CSS

css-loader和style-loader用来将样式嵌入到Webpack打包后的JS文件中，从而实现样式的模块化管理。

针对css-loader的配置，你会看到它的格式是如下的：

```
css?modules&importLoaders=1&localIdentName=[name]__[local]___[hash:base64:5]
```
其实，这是webpack loader传递参数的一种方式（URL方式），css-loader的参数表请进入[传送门](https://github.com/webpack-contrib/css-loader#options-1)。

localIdentName是用于指定局部className的格式，比如：

```
[name]__[local]___[hash:base64:5]
ComponentName__LocalName___HashCode
Component__pullquote___SDFe34IIE
```
### 如何抽离出独立的css文件？Extract Text Plugin

正如前面所强调的，style-loader和css-loader，将样式嵌入到Webpack打包后的JS文件。

这当然不是最好的方式，我们需要将样式抽离到独立的css文件中，但就是[Extract Text Plugin](https://github.com/webpack-contrib/extract-text-webpack-plugin)。

```
yarn add extract-text-webpack-plugin --dev
```

完整的webpack.config.js文件如下：

```
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

const BUILD_DIR = path.resolve(__dirname, 'public');
const APP_DIR = path.resolve(__dirname, 'src');

const extractCSS = new ExtractTextPlugin(`${BUILD_DIR}/styles.css`);

const config = {
  entry: `${APP_DIR}/index.jsx`,
  output: {
    path: BUILD_DIR,
    filename: 'bundle.js',
  },
  module: {
    loaders: [
      {
        test: /\.jsx?/,
        include: APP_DIR,
        loader: 'babel',
      },
      {
        test: /\.scss$/,
        loader: extractCSS.extract(['css?minimize&modules&importLoaders=2&localIdentName=[name]__[local]', 'postcss', 'sass']),
      },
    ],
  },
  resolve: {
    extensions: ['', '.js', '.jsx'],
  },
  plugins: [
    extractCSS,
  ],
};

module.exports = config;

```
