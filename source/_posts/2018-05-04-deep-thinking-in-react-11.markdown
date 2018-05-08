---
layout: post
title: "React的思考（十一）- 组织state和reducer（3）"
date: 2018-05-04 23:09:38 +0800
comments: true
categories:
---

以React为首的“数据驱动”的开发方式，让我们从过去的DOM操作，慢慢转化成对应用状态和数据的管理，这些状态和数据在应用的生命周期中被持久化，被应用管理和使用，怎么样有效的在SPA中管理这些数据变的特别重要。

## 前后端不同的数据模型

在后端开发中，我们设计数据库或者对象模型，通常会根据领域模型来建立不同的数据表和对象，以反映我们对客观现实的抽象，这种抽象在MVC的世界里通常由Model表示。

而在前端开发中，我们一般会从UI的角度出发，去设计前端展示所需要的视图对象模型，我们也称之为View Model。

而往往在大多数情况下，后端的数据模型Model和前端的数据模型View Model是不对等的。

## 来自后方的数据

当一个应用的生命周期启动，应用会调用后台的API获取后端数据，然后以某种方式转换成前端所需数据模型，最后展示在应用的页面上。

作为一个后端的开发人员，通常以什么样的方式暴露后端的资源给别的应用使用呢？

如果需要API的通用度比较高，一般最简单直接的方法是开放领域资源的CRUD操作（你可以是RESTful API也可以是别的方式）。

```
https://api.example.com/v1/zoos
https://api.example.com/v1/animals
https://api.example.com/v1/employees

GET /zoos：列出所有动物园
POST /zoos：新建一个动物园
GET /zoos/ID：获取某个指定动物园的信息
PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
```
我们目前就暂且以这种方式为例，目测这种方式暴露的API比较常见，后面我们再讨论其他的。

## combineReducer下的拆分

让我们再回到Redux中，基于Redux的应用程序中，比较常见的state结构是一个简单的JavaScript对象，它最外层的每个key中拥有特定域的数据，这其实是Redux官方文档上的一句话，我在其他的一些博客上也看到了采用基于业务领域的方式组织state结构。

```JavaScript
{
  "zoos": {
    ...
  },
  "animals": {
    ...
  },
  "employees": {
    ...
  }
}
```

在combineReducer的帮助下，构建上面这个结构，其实就是拆分成多个reducer，拆分之后的reducer独立负责管理该特定切片state的更新。好处是，它提供了一种非常直接的代码逻辑拆分管理的方式，职责独立，物理位置独立（文件独立）。

## 响应同一个操作

有了这样一层结构，我们该如何操作它呢？假设这样一个操作，它既需要更新zoos下面的数据，又需要更新animals下面的数据。如果放在后端代码中，你会怎么做？

我的思路会是写一个函数A，在这个函数里面调用zoos和animals的service或者repository的方法完成更新操作，那么，真正使用的时候，只需要调用这个函数A即可，思路很明朗直接。

在combineReducer和redux结合情况，我们需要转换一下思路，不同业务领域下的数据被放置到了不同的reducer，而你能做的是发送action。

combineReducer神奇的地方就是，被发送的action会被所有的reducer接收到。（有点像发布订阅模式）

这样一个过去直觉上同步有序的操作过程，在redux中，被分发到多个拆分之后的多个reducer中，每个reducer都去响应这个action，在需要的情况下独立的更新他们自己的切片state，最后组合成新的state。

## 计算衍生（derived）数据

后端的业务数据被存储在了redux的store中，然而它是以后端model的形式保存在那。我们的前端页面需要的数据模型，一般和后端model不完全一样，也许是多个后端model组合在一起才能使用的view model，这种情况很常见。

这个时候，我们就需要从后端数据模型中计算衍生数据，得到我们最终需要的View Model。

一般的做法是在mapStateToProps中进行，mapStateToProps中的state是根节点上的state，所以可以拿到所有的领域数据，此时我们就能根据它衍生出我们需要的视图模型。

而在这里一般都会推荐使用reselect库来做，好处是：

第一，能够借机将这个计算衍生数据的逻辑拆分到另一个模块中       
第二，它能帮你记住之前的计算过的数据，避免二次计算，同时避免无意义的重新渲染。

```JavaScript
import { createSelector } from 'reselect'

const getVisibilityFilter = (state) => state.visibilityFilter
const getTodos = (state) => state.todos

export const getVisibleTodos = createSelector(
  [ getVisibilityFilter, getTodos ],
  (visibilityFilter, todos) => {
    switch (visibilityFilter) {
      case 'SHOW_ALL':
        return todos
      case 'SHOW_COMPLETED':
        return todos.filter(t => t.completed)
      case 'SHOW_ACTIVE':
        return todos.filter(t => !t.completed)
    }
  }
)
```

## 总结

这么分析下来，每个部分的职责都很清晰，开发的模式也比较明确，然而理想和设计在业务不复杂的时候都很美好，现实往往比它们骨感很多，这一篇文章只是给了一个简单的例子，没有分析复杂的场景。

在下一篇，我们可以在此基础上分析开发中的复杂场景，一起思考下怎么样管理是合适的。同时，我们也来思考另一种新的前端架构BFF下的Redux应该怎么管理，当后端API不再是资源的CRUD，而是面向业务的API操作时，Redux又应该怎么管理state。
