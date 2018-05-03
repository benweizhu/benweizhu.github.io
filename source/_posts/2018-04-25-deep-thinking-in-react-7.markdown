---
layout: post
title: "React的思考（七）- Redux的Middleware（上）- 中间件的概念"
date: 2018-04-25 21:14:18 +0800
comments: true
categories:
---

Redux的Middleware（中间件）是Redux中相对比较神秘的部分。

## 怎么理解middleware这个概念呢？

middle这个词很重要，它是指，这个件（ware），被放在（穿插）于某一个已存在操作的过程当中（middle）。

我这么平淡直白无水准的解释，你应该能get到吧，如果你熟悉Java Web开发，你可能第一时间会想到Java Servlet Filters，当然也许你比较年轻，你对Spring熟悉，你可能会立刻想到AOP（面向切面编程），它们可以完成日志，审计，authentication, authorization等。

那么，再往后面走，如果你使用过Express或者Koa等服务端框架，在这类框架中，middleware是指可以被嵌入在框架“接收请求到产生响应过程之中”的代码。例如，Express或者Koa的middleware可以完成添加CORS headers、记录日志、内容压缩等工作。

## Redux的中间件是做什么呢？

回过头来，看Redux的中间件，同样，它肯定是指穿插在Redux的某一个过程当中，那么问题来了，这个ware可以在哪里穿插，或者哪里有穿插操作的需要呢？我们来逐一分析，以下内容，参考阮一峰的文章：

（1）Reducer：纯函数，只承担计算State的功能，不合适承担其他功能，也承担不了，因为理论上，纯函数不能进行读写操作。

（2）View：与State一一对应，可以看作State的视觉层，也不合适承担其他功能。

（3）Action：存放数据的对象，即消息的载体，只能被别人操作，自己不能进行任何操作。

其实，和其他服务端框架类似，它被嵌入到Redux“接收请求到产生响应过程之中”，位于action被发起之后，到达reducer之前，也就是store.dispatch()附近。

## 举个例子：如何在dispatch的时候log state和action

我在看官方文档的时候，看到一个非常有趣的概念，猴子补丁（monkey patching），大概的意思是指在运行时动态修改模块、类或函数，通常是添加功能或修正缺陷。

通过这种方式，可以在代码的运行过程中，给store.dispatch打补丁，增加额外的功能，比如log state和action，代码如下：

```JavaScript
let next = store.dispatch;
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action);
  next(action);
  console.log('next state', store.getState());
}
```
通过上面这样一段代码，我们就在Redux原来的store.dispatch附近添加了我们自己代码，当我们再次调用store.dispatch，它就会打印log了，不过monkey patching本质上是一种hack，“将任意的方法替换成你想要的”。

如果，我们想给dispatch加另一个补丁，那就在它的前面，或者后面，在加上一段类似的代码呗。

## Redux applyMiddlewares()

真实情况下，我们肯定不用这样写代码，那Redux提供了applyMiddlewares的方式，所以就不需要我们像上面那样写，那它是怎么做的呢？

```JavaScript
const store = createStore(
  reducer,
  applyMiddleware(thunk, promise, logger)
);
```

```JavaScript
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    var store = createStore(reducer, preloadedState, enhancer);
    var dispatch = store.dispatch;
    var chain = [];

    var middlewareAPI = { //注意这里：getState和dispatch都传入了middleware
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    };
    chain = middlewares.map(middleware => middleware(middlewareAPI));//注意这里：这些middleware都是函数
    dispatch = compose(...chain)(store.dispatch);

    return {...store, dispatch}
  }
}
```

所有中间件被放进了一个数组chain，通过compose，将多个中间件合并，从右到左执行。中间件可以拿到getState和dispatch这两个方法。

注：compose: 将多个函数合并成一个函数，从右到左执行。例如：

```JavaScript
R.compose(Math.abs, R.add(1), R.multiply(2))(-4) // 7
```
-4 * 2 + 1 再求绝对值

## 总结

仔细看下来之后，中间件就没有那么神秘了，下一篇文章，我们来介绍下Redux的异步中间件，比如：Redux-Thunk，看懂这一个，其他的都差不多。
