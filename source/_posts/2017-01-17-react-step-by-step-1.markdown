---
layout: post
title: "一步一步做React(一) - Hello React"
date: 2017-01-17 21:08:15 +0800
comments: true
categories:
---
什么都不说，先照着做

#### 安装合适的node版本和包管理工具Yarn

```
node -version v6.9.4(用nvm)

brew update（如果是mac的）

brew install yarn
```

https://yarnpkg.com/en/docs/install


#### 安装webpack

```
yarn add --dev webpack
```

#### 安装babel

```
yarn add --dev babel-core babel-loader babel-preset-es2015 babel-preset-react
```
#### 安装React
```
yarn add react react-dom
```
#### 创建和编辑.babelrc
```
touch .babelrc   
vim .babelrc  
```
```
{
  "presets" : ["es2015", "react"]
}
```
#### 创建和编辑webpack.config.js
```
touch webpack.config.js
vim webpack.config.js
```
```
var webpack = require('webpack');
var path = require('path');

var BUILD_DIR = path.resolve(__dirname, 'public');
var APP_DIR = path.resolve(__dirname, 'src');

var config = {
	entry: APP_DIR + '/index.jsx',
	output: {
		path: BUILD_DIR,
		filename: 'bundle.js'
	},
	module: {
		loaders: [
			{
				test: /\.jsx?/,
				include: APP_DIR,
				loader: 'babel'
			}
		]
	}
};

module.exports = config;
```

#### 创建和编辑src/index.jsx
```
mkdir src
cd src
touch index.jsx
vim index.jsx
```
```
import React from 'react';
import {render} from 'react-dom';

class App extends React.Component {
	render () {
		return <p> Hello React!</p>;
	}
}

render(<App/>, document.getElementById('app'));
```

#### 在html中引用
```
cd src
touch index.html
vim index.html
```
```
<!doctype html>

<html lang="zh">
<head>
    <!-- The first thing in any HTML file should be the charset -->
    <meta charset="utf-8">
    <!-- Make the page mobile compatible -->
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="mobile-web-app-capable" content="yes">
    <title>17High</title>
</head>
<body>
<div id="app"/>
<!-- Display a message if JS has been disabled on the browser. -->
<noscript>If you're seeing this message, that means <strong>JavaScript has been disabled on your browser</strong>, please <strong>enable JS</strong> to make this app work.</noscript>

<script src="../public/bundle.js" type="text/javascript"></script>
</body>
</html>
```
#### 运行webpack命令，生成打包文件bundle.js
```
./node_modules/.bin/webpack -d
./node_modules/.bin/webpack -d --watch //监听文件变化
```

打开index.html看效果
