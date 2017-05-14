---
layout: post
title: "React应用在产品环境下的性能优化"
date: 2017-05-12 18:21:55 +0800
comments: true
categories:
---

只有10%~20%的最终用户响应时间花在了下载HTML文档上，其余的80%~90%时间花在了下载页面中的 **所有组件** 上。   - 性能黄金法则，Steve Souders

Steve Souders在2007年提出这样的“性能黄金法则”，我猜测当他看到React这样一项技术之后，一定会觉得自己的这个法则居然如此的准确，可能甚至觉得这个比例不够极致。（虽然此组件非React组件，但是我还是忍不住想笑）

所以，今天我们就来聊聊，React应用在产品环境下的性能优化问题。

## Bundle大小分析

在开始做任何的优化之前，你需要知道痛点在什么地方？既然Steve说80%~90%时间花在了下载页面中的 **所有组件** 上，那么就从了解项目的模块组成开始。

**1.Webpack运行时的输出**

在没有任何外部力量帮助的情况下，我们可以直接阅读Webpack的输出
![react ouput](/images/react-production/webpack-output.png =500x)
```
webpack --display-chunks
```
可以查看模块在哪个分块中出现，帮助我们查找分块中重复的依赖。

**2.bundle-size-analyzer**

[webpack-bundle-size-analyzer][32444442]是我个人比较喜欢的模块大小分析工具，使用起来非常简单，输出也非常清晰。

![webpack-analyzer](/images/react-production/webpack-analyzer.png =500x)

  [32444442]: https://github.com/robertknight/webpack-bundle-size-analyzer "webpack-bundle-size-analyzer"

**3.webpack-bundle-analyzer**

[webpack-bundle-analyzer][d064a1cf]在github上star人数更多，功能也相对更加齐全（fancy）。
![webpack-bundle-analyzer.gif](/images/react-production/webpack-bundle-analyzer.gif =500x)

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
  devtool: isProd ? 'cheap-source-map' : 'cheap-module-inline-source-map',
}
```
官方提供了7种Devtool，而且有更详细的关于devtool的配置，请详见 [devtool][4d00ee4f]。
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
##Tree Shaking

[Webpack TreeShake][0915e9b0]

清理无用的JS代码，真实导入有用的模块。配置起来非常简单：
```
"presets": [["es2015", {"modules": false}], "react", "stage-0"]
```
  [0915e9b0]: https://webpack.js.org/guides/tree-shaking/ "Webpack TreeShake"

![tree-shake.png](/images/react-production/tree-shake.png)

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
gzip on; //仅仅配置这一行是不会起作用的
gzip_types  text/plain application/javascript application/x-javascript text/javascript text/xml text/css;
gzip_proxied    no-cache no-store private expired auth;
gzip_min_length 1000;
```

2.手动压缩

另外一种方式，就是我们自己手动压缩Gzip，这样可以减少Nginx编码带来的性能消耗，Webpack插件
[compression-webpack-plugin][797cb956]可以做到。

  [797cb956]: https://github.com/webpack-contrib/compression-webpack-plugin "compression-webpack-plugin"

##还有什么别的提高性能的办法呢？

**1.服务器端渲染如何**

有人会说，**服务器端渲染如何？** 这个要看情况。服务器端渲染一般主要用来处理首屏渲染性能（注意是首次加载）和搜索引擎爬虫问题。如果你的JS文件特别大，那么服务器端渲染能够，让用户在加载完HTML和CSS之后立刻看到页面。如果不是首次加载，那么其实JS是可以缓存在客户端的，所以即便不用服务器端渲染，之后也不会很慢。

更多关于[是否应该进行服务器端渲染][0319f9fe]以及[如何进行服务器端渲染][1fa7be4c]？请查看相关文章。

  [1fa7be4c]: http://redux.js.org/docs/recipes/ServerRendering.html "如何进行服务器端渲染"
  [0319f9fe]: http://andrewhfarmer.com/server-side-render/ "是否应该进行服务器端渲染"

**2.ServiceWork**

渐进式 Web 应用程序思想（PWA）最近可火了，2017年[ThoughtWorks技术雷达][b36f1256]将[“Progressive Web Applications”][f62dc182]放在了试验阶段。

  [f62dc182]: https://www.thoughtworks.com/radar/techniques/progressive-web-applications "“Progressive Web Applications”"
  [b36f1256]: https://www.thoughtworks.com/radar "ThoughtWorks技术雷达"

简单介绍什么是service worker:

> 在2014年，W3C公布了service worker的草案，service worker提供了很多新的能力，使得web app拥有与native app相同的离线体验、消息推送体验。
> service worker是一段脚本，与web worker一样，也是在后台运行。
> 作为一个独立的线程，运行环境与普通脚本不同，所以不能直接参与web交互行为。native app可以做到离线使用、消息推送、后台自动更新，service worker的出现是正是为了使得web app也可以具有类似的能力。

Github： [Google sw-toolbox][e4b610ae], [sw-precache-webpack-plugin][16ad9fe5]和[offline-plugin][71bd14a9]

  [16ad9fe5]: https://github.com/goldhand/sw-precache-webpack-plugin "sw-precache-webpack-plugin"
  [71bd14a9]: https://github.com/NekR/offline-plugin "offline-plugin"
  [e4b610ae]: https://github.com/GoogleChrome/sw-toolbox "Google sw-toolbox"

**3.Preload**

Preload 作为一个新的web标准，旨在提高性能和为web开发人员提供更细粒度的加载控制。Preload使开发者能够自定义资源的加载逻辑，且无需忍受基于脚本的资源加载器带来的性能损失。
```html
<link rel=“preload”>
```
作为新的标准，浏览器兼容性是你有必要考虑的一个方面：

![preload](/images/react-production/preload.png!web)

github: [preload-webpack-plugin][881bf37b]

  [881bf37b]: https://github.com/GoogleChrome/preload-webpack-plugin "preload-webpack-plugin"

##最后  

文章内容有点长，但我相信这些都是干货是值得一读的，前端产品环境性能优化确实是一个说不完的话题，前端技术更新迭代也没有多少其他计算机技术能够匹敌的，这也对前端开发工程师（全栈开发工程师）的技术敏感度和追求新技术的态度有很高的要求。

作者：Benwei，ThoughtWorks高级咨询师，全栈开发工程师，《实战Gradle》译者
