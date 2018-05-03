---
layout: post
title: "React的思考（九）- 组织state和reducer（上）"
date: 2018-05-01 22:47:38 +0800
comments: true
categories:
---
## 前提条件

在进行深入探讨之前，我先确保大家的理解是一致的，因为这部分是客观的，而之后的内容是相对主观和有争议的。

1.当组件的某个操作dispatch了一个action，所有的reducer都会接收到：[All reducers will be invoked when an action is dispatched?][2f5b8732]，[combineReducers用法中也有讲到][3d72e6a7]

  [2f5b8732]: https://stackoverflow.com/questions/33590579/all-reducers-will-be-invoked-when-an-action-is-dispatched "All reducers will be invoked when an action is dispatched?"

  [3d72e6a7]: https://cn.redux.js.org/docs/recipes/reducers/UsingCombineReducers.html "combineReducers用法中也有讲到"

2.当combineReducers发现有任意一个reducer返回了新的state，会通知所有和redux关联（connect）的组件准备更新，请检查自己是否要更新

3.每一个通过connect构建的组件，其mapStateToProps中的state，是combineReducers合并的state，也就是每个组件都能拿到所有reducer中的state（曾经有遇到过有人误以为是跟它action相关的reducer的state）

## Redux的上帝视角

Redux的三大设计原则之一，单一数据源，定义了整个应用的state被储存在一棵object tree中，并且这个object tree只存在于唯一一个store中。这样一个顶层的状态树，会拥有一个全局的视角，掌握着整个应用的状态。

单一数据源的设计原则在许多程序设计的领域都是准确的，但仅仅用一个JavaScript对象来存储整个应用的状态，总会让人感觉某一天这个对象一定会特别臃肿，上帝实在太忙，要关注的东西太多。

很自然的，我们就会去思考，物尽其用，到底什么样的数据应该放在Redux中，什么样的数据应该放在别处来管理。

## 应用的拥有的数据和状态

大多数应用会处理多种类型的数据和状态，通常可以分为以下三类：领域数据（Domain data），应用状态（App state），UI状态（UI state）

领域数据也就是业务相关数据，一般和你的后台业务系统数据相关联，是领域数据的数据来源，但他们不一定直接对等。

应用状态和UI状态有时候不容易分清，应用状态是描述应用无限循环的生命周期中的某一种存在（中间）状态，而这样一个应用状态可能会导致一个或者多个UI的状态变化。比如：用户登录，是一个应用状态，它可能导致导航栏的UI状态改变。

UI的状态，自然是描述UI的改变，但不一定是由应用状态的变化导致。比如：页面上tab的切换。

## 应用数据和状态的存储位置和影响范围

根据上面的应用数据和状态的分类，好像让我们对这样一个问题有些头绪。不过在你下任何判断之前，我们在从另外一个维度继续思考一下。

从前面我们就了解到，Redux存储的数据，在任何一个与之关联的组件中都能拿到，也就是说，Redux存储的是一个全局的数据。

反之，React本身也有一个state，而它所关注的只是组件本身以及它的子组件，兄弟和父组件它都不关心。

除了Redux和React，就没有别的位置保存数据了？当然不是，比如：cookie，local storage。这些也是保存数据的关键位置，毕竟当页面刷新后，Redux和React中的state就丢失了，还需要从后台重新加载。

## React组件的state存什么？

现在还不是时候讨论Redux里面存什么，我们反向推理，用排除法，先看看React里面应该存在什么。

React中的state存在于组件当中，那我们就需要思考组件的特性，能独立，也许能自治，必然高可重用，这是它的部分典型特性。基于它这些特性，也就决定了它的state也必须满足这些要求，组件的state是服务于组件本身的，这些state能够不收外部干预下就自我管理，这些state当组件被用在任何位置时，都能适应。一个典型的例子：UI状态。

当然，上面是我对React组件state理解的一个抽象描述，能够一定程度下知道我的思考。还有什么原则，能够帮助决策什么样的state可以放在组件中，Dan Abramov在他的Twitter发了这样一张图片也具有不错的指导意义：

![](https://camo.githubusercontent.com/5e85994aa142e7699548e2f5a1e74583229ebd10/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f436d654273477a57384151705f61762e6a7067 =500x)

## 总结

今天先写到这里，后续还会继续讨论redux的state和reducer的设计思考。
