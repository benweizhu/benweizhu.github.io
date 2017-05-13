---
layout: post
title: "React应用在产品环境代码下性能优化"
date: 2017-05-12 18:21:55 +0800
comments: true
categories:
---

只有10%~20%的最终用户响应时间花在了下载HTML文档上，其余的80%~90%时间花在了下载页面中的所有组件上。   - 性能黄金法则，Steve Souders

Steve Souders在2007年提出这样的“性能黄金法则”，我猜测当他看到React这样一项技术之后，一定会觉得自己的这个法则居然如此的准确，甚至觉得这个比例不够极致。（虽然此组件非React组件，但是我还是忍不住想笑）

所以，今天我们就来聊聊，React应用在产品环境下的性能优化问题。

## Bundle大小分析

在开始做任何的优化之前，你需要知道痛点在什么地方？最直观和最有效的方式之一，就是了解项目的模块组成。

**1.Webpack运行时的输出**

在没有任何外部力量帮助的情况下，我们可以直接阅读Webpack的输出
![react ouput](/images/react-production/webpack-output.jpg =400x)

缺点是，信息不够清晰直观，晦涩难懂，关键是官方文档也没有给出详细的说明。

**2.bundle-size-analyzer**

[webpack-bundle-size-analyzer][32444442]是我个人比较喜欢的模块大小分析工具，使用起来非常简单，输出也非常清晰。

![webpack-analyzer](/images/react-production/webpack-analyzer.png =400x)

  [32444442]: https://github.com/robertknight/webpack-bundle-size-analyzer "webpack-bundle-size-analyzer"

**3.webpack-bundle-analyzer**

[webpack-bundle-analyzer][d064a1cf]在github上star人数更多，功能也相对更加齐全（fancy）。
![webpack-bundle-analyzer.gif](/images/react-production/webpack-bundle-analyzer.gif =400x)

  [d064a1cf]: https://github.com/th0r/webpack-bundle-analyzer "webpack-bundle-analyzer"

## 代码分离（Code Splitting）

**1.Vendor代码分离**

代码分离是Webpack核心功能之一，典型的做法是将第三方依赖代码从应用代码中抽离出来，这样可以利用浏览器的缓存来提高性能（减少下载次数）。

Webpack官方文档有非常详细的介绍： [code-splitting-libraries][7d15232d]，我就不在这里赘述，下面是一个简单代码样例：

```javascript
var webpack = require('webpack');
var path = require('path');

module.exports = function(env) {
    return {
        entry: {
            main: './index.js',
            vendor: 'react', 'react-dom', 'react-redux', 'babel-polyfill']
        },
        output: {
            filename: '[name].[chunkhash].js',
            path: path.resolve(__dirname, 'dist')
        },
        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                name: 'vendor' // Specify the common bundle's name.
            })
        ]
    }
}
```
[7d15232d]: https://webpack.js.org/guides/code-splitting-libraries/ "code-splitting-libraries"

**2.CSS代码**

也许你还会想要的就是将CSS文件分离，原因是一样的。官方文档也给出了非常详细的介绍：[code-splitting-css][69ce9a0b]，所以同样也不赘述了。
  [69ce9a0b]: https://webpack.js.org/guides/code-splitting-css/ "code-splitting-css"

```javascript
const extractCSS = new ExtractTextPlugin('styles.css');
module: {
  rules: [
    {
      test: /\.scss$/,
      use: extractCSS.extract(['css-loader', 'postcss-loader', 'sass-loader'])
    }
  ]
},
plugins: [
  extractCSS
]
```
**3.React-Router按需分离**

当应用逐渐变得复杂后，你会发现，仅仅将代码分离为vendor和app两个bundle，远远是不够的，要么vendor.js文件特别大，要么app.js文件特别大，这个时候你一定会想到，要按需加载（异步加载）。

ES2015 Loader spec中定义了一个import()方法来在运行时动态加载ES2015的模块，代码如下：
```
function determineDate() {
  import('moment').then(function(moment) {
    console.log(moment().format());
  }).catch(function(err) {
    console.log('Failed to load moment', err);
  });
}
determineDate();
```
Webpack会将import()方法看做一个“代码分离点”，将被加载的模块放在一个单独的文件块中。

那么，如果你的应用采用了React-Router，我们就可以根据路由，按需加载所使用的组件。

```javascript
function errorLoading(error) {
  throw new Error(`Dynamic page loading failed: ${error}`);
}

function loadRoute(cb) {
  return module => cb(null, module.default);
}

<Router history={history} queryKey="false">
  <Route path="/user" name="UserPage" getComponent={(location, cb) => {
    System.import('./components/UserPage').then(loadRoute(cb, false)).catch(errorLoading)}}
  />
  <Route path="/data" name="DataPage" getComponent={(location, cb) => {
    System.import('./components/DataPage').then(loadRoute(cb, false)).catch(errorLoading)}}
  />
  <Route path="/about" name="AboutPage" getComponent={(location, cb) => {
    System.import('./components/AboutPage').then(loadRoute(cb, false)).catch(errorLoading)}}
  />
</Router>
```

## 运行webpack -p

Webpack的官方文档有详细的说明，对于产品环境的构建应该运行webpack -p：[production-build][8eea20d6]。
  [8eea20d6]: https://webpack.js.org/guides/production-build/ "production-build"

```
webpack --optimize-minimize --define process.env.NODE_ENV="'production'"
```
此时，Webpack会做几件事情，我们也需要根据这些事情做相关的配置：

**1.对JS代码进行压缩**

--optimize-minimize 标签会对JS代码用UglifyJsPlugin做压缩，并根据Webpack中配置的devtool配置SourceMap

**2.SourceMap**

即便在产品环境下，仍然建议使用SourceMap，方便产品环境的bug定位，但是对于开发环境和产品环境，我们需要使用不同力度的SourceMap，才能既方便开发也兼容产品环境性能。

```javascript
const config = {
  // webpack config
}
if (process.env.NODE_ENV === 'production') {
  config.devtool = 'cheap-source-map';
} else {
  config.devtool = 'cheap-module-inline-source-map';
}
```
官方文档有更详细的关于devtool的配置，详见 [devtool][4d00ee4f]。
  [4d00ee4f]: https://webpack.js.org/configuration/devtool/ "devtool"

**3.Node环境变量production**

将redux的中间件和开发环境使用的devtool通过变量分离
--define process.env.NODE_ENV="'production'" 标签会以下面的方式使用DefinePlugin：
```javascript
const webpack = require('webpack');
module.exports = {
  /*...*/
  plugins:[
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify('production')
    })
  ]
};
```
这个时候，就可以在产品代码里面获取到此环境变量。这个时候我们要做的就是根据环境变量的不同，来进行不同的配置，比如：这样写log
```javascript
if (process.env.NODE_ENV !== 'production') console.log('...')
```
对于产品环境就等价于：
```javascript
if (false) console.log('...')
```
此时UglifyJS插件就会将它去除掉。

又比如：在react-redux开发中，我们一般都会配置开发插件DevTool，或者log中间件，但其实，在产品环境中，我们不需要，这个时候就需要根据环境变量来动态配置：

```javascript
import { applyMiddleware, compose, createStore } from 'redux';
import thunkMiddleware from 'redux-thunk';
import { createLogger } from 'redux-logger';
import rootReducer from './reducers';
import promiseMiddleware from 'redux-promise-middleware';

export default () => {
  let finalCreateStore;
  if (process.env.NODE_ENV === 'production') {
    finalCreateStore = compose(
      applyMiddleware(promiseMiddleware(), thunkMiddleware)
    )(createStore);
  } else {
    finalCreateStore = compose(
      applyMiddleware(promiseMiddleware(), thunkMiddleware, createLogger()),
      window.devToolsExtension ? window.devToolsExtension() : f => f
    )(createStore);
  }
  return finalCreateStore(rootReducer);
};

```
## Babel对React代码的优化

除了从产品环境模块架构上优化，Babel也在编译阶段优化React应用性能作出了巨大贡献。

1.transform-react-constant-elements插件

[transform-react-constant-elements][175dea69]，自从React0.14版本，我们可以将React元素以及他们的属性对象当做普通的值对象看待。这时候我们就可以重用那些输入是immutable的React元素。
  [175dea69]: https://babeljs.io/docs/plugins/transform-react-constant-elements/ "transform-react-constant-elements"
```javascript
const Hr = () => {
  return <hr className="hr" />;
};
```
```javascript
const _ref = <hr className="hr" />;

const Hr = () => {
  return _ref;
};
```
从而减少对React.createElement的调用。

2.transform-react-inline-elements插件

[transform-react-inline-elements][bc37bd17]，自从React0.14版本，可以将React元素内联为对象，Babel将React.createElement方法替换成babelHelpers.jsx来转换元素为对象。
```javascript
<Baz foo="bar" key="1"></Baz>;
```
```javascript
babelHelpers.jsx(Baz, {
  foo: "bar"
}, "1");
```
{type: Baz,props:{foo:"bar"},key:"1"}

  [bc37bd17]: https://babeljs.io/docs/plugins/transform-react-inline-elements/ "transform-react-inline-elements"

除了以上两个官方插件，在开源世界还有许多其他Babel插件可以优化代码，也不再此追溯，留给大家自己去看： [transform-react-remove-prop-types][6e3b3780]， [transform-react-pure-class-to-function][0678e86b]， [babel-react-optimize][fa60633b]（综合所有优化的插件，此处应该有掌声）

  [6e3b3780]: https://github.com/oliviertassinari/babel-plugin-transform-react-remove-prop-types "transform-react-remove-prop-types"
  [0678e86b]: https://github.com/thejameskyle/babel-react-optimize/tree/master/packages/babel-plugin-transform-react-pure-class-to-function "transform-react-pure-class-to-function"
  [fa60633b]: https://github.com/thejameskyle/babel-react-optimize "babel-react-optimize"

## 代码本身的优化

除了利用工具和构建来提高产品环境下的代码性能，最最基础的还是开发在平时写代码的需要注意的一些基础原则

1.只导入需要的包

比如：lodash，如果你只用到isEqual，那么就不要把整个lodash都引入
```
import isEqual from 'lodash/isEqual';
```

2.使用ESLint

合理的使用ESLint，除了帮助团队指导代码风格，也可以告诉你如何正确的写React应，比如，当组件是纯presentional组件时，就应该使用PureComponent或者纯函数组件，这些eslint都会告诉你。

3.利用React官方的[Perf工具][5fc674cc]

  [5fc674cc]: https://facebook.github.io/react/docs/perf.html "Perf工具"

```javascript
Perf.start()
// ...
Perf.stop()
```

## 服务器端优化

使用Gzip压缩倒不是React应用才有的性能优化策略，但还是要提一下，因为确实有用。

1.Nginx服务器端配置

我猜测大部分的情况下，都会用Nginx来部署静态资源，斗胆提供一个nginx的gzip配置。

```
gzip on;//仅仅配置这一行是不会起作用的
gzip_types  text/plain application/javascript application/x-javascript text/javascript text/xml text/css;
gzip_proxied    no-cache no-store private expired auth;
gzip_min_length 1000;
```

2.手动压缩

另外一种方式，就是我们自己手动压缩，这样可以减少Nginx编码带来的性能消耗，Webpack插件
[compression-webpack-plugin][797cb956]可以做到。

  [797cb956]: https://github.com/webpack-contrib/compression-webpack-plugin "compression-webpack-plugin"

## 服务器端渲染


## preload
