---
layout: post
title: "React的思考（一）- 官网首页的信息量就挺大"
date: 2018-04-17 21:11:23 +0800
comments: true
categories: 
---
没错，这个标题有些大的，也挺抽象，给自己画了一个大饼，就看能不能给圆上。

##从官网的首页开始

### 先看小标题

我们就从[React][12f86ce9]官方网站的首页开始我们的思考。先看它的小标题：
  [12f86ce9]: https://reactjs.org/ "React"

> A JavaScript library for building user interfaces。

React从一开始就将自己到底是一个什么样的存在，定义的非常的清楚。看清楚，我不是一个框架，我就是一个JavaScript的库，那我是干什么用的呢？构建用户的界面（UI），其他的乱七八糟的事情我不管。

有人可能会问，其他乱七八糟事情的是什么？比如：页面的路由，网络请求，逻辑控制器（Controller），服务等等，是不是听起来挺像前几年某A打头的框架做的事情，这里我就不点名了，大家心里都清楚，没有谁对谁错，此一时彼一时的。

为什么React是这样的一个定义呢？关于这一点，我们可以在Pete Hunt在2013年5月份写的一篇博客[Why did we build React?][b1fcf350]中看到一些insight。比如：
  [b1fcf350]: https://reactjs.org/blog/2013/06/05/why-react.html "Why did we build React?"，
  
简单摘录一段：

*React isn’t an MVC framework.*    

*React is a library for building composable user interfaces. It encourages the creation of reusable UI components which present data that changes over time.*

鄙人简单理解和翻译：我不是MVC框架。React是一个用于构建可组合用户界面的库。它鼓励创建可重用的UI组件，以呈现随时间变化的数据。Pete Hunt将React的目的说的很透彻。

React官网也通过这样一句话，给自己了一个清晰的定位，并且在这个清晰的定位下，给出下面三个基本特性：Declarative，Component-Based和Learn Once, Write Anywhere。我们一个个来看：

#### Declarative

Declarative，声明式的，嗯呐嗯呐，这是什么意思？相信大家对“声明”这个词比较了解，比如：声明一个变量，声明一个函数。

要理解它，首先要引入另外一个东西，叫做Imperative Programming（命令式编程）。声明式编程和命令式编程，都是一种编程范式，那么他们的区别是什么？简单来说就是what和how的区别。

*命令式编程：命令“机器”如何去做事情(how)，这样不管你想要的是什么(what)，它都会按照你的命令实现。*    

*声明式编程：告诉“机器”你想要的是什么(what)，让“机器”想出如何去做(how)。*

命令式编程应该大家都比较好理解，比如：操作几个变量，最后计算出你想要的结果，这里的重点在于你通过指令操作它们得到结果。那么声明式呢？举个例子，比如，SQL语句：

```SQL
Select * from JSLibrary where name = 'React'
```

你没告诉SQL该怎么去搜索，只是告诉它要找到名字是React的库，对吧？

React就是采用声明式的编程范式思想，你只需要设计在不同状态下，组件应该是长什么样子，React自己会帮助完成组件的更新。它的最直接明白的对比（反面教材），就是通过jQuery操作DOM来更新UI。

React的这种开发模式和有限状态机的思想是一致的，在预知所有状态的条件下，去规划你的代码，也因此衍生了Redux, MobX这样的状态管理库。

#### Component-Based
基于组件的，这个相对比较好理解，组件是什么？对数据和方法的简单封装。它应该具备具有独立性，封装性，可重用性，职责单一，有自己的状态等等。React组件就封装了自己的状态来构建复杂的UI的组件。

*Since component logic is written in JavaScript instead of templates, you can easily pass rich data through your app and keep state out of the DOM.*

官方网站上说：组件的逻辑是用JavaScript编写，而不是模板，所以你轻松的传递数据到应用，并且让状态不和DOM打交道。

如果你十分好奇，这里的“而不是模板”是什么意思？Pete Hunt的那篇文章其实说的很清楚，传统的Web应用是通过HTML或者模板引擎（比如后端模板引擎：JSP，HAML等，前端模板引擎：handlebar,ejs等）来构建UI的，而React使用有完整功能编程语言来渲染师徒。

其实，我是不太认同的，JSX不算模板，VueTemplate不算模板？不过JSX允许你用JavaScript的方式做一些逻辑的处理，而不像JSP需要些JSTL和ctag的的逻辑标签，如果我的理解有误，请务必纠正我。

#### Learn Once, Write Anywhere
这个，看看就行，官方网站这里特指的是React Native可以开发移动端的应用，不过大家也不要太天真，你理解成React和React-Native的思想和语法是融会贯通的就行了，不要真的以为可以很轻松的将组件在React和RN之间移植，否则你会被鄙视的。

另外提一点，在ElectronJS的帮助下，可以通过React开发桌面应用，这个倒是真可行。

###总结

你看，首页的信息量其实挺大吧，认证阅读和思考，其实收获不少，总结下来就是：我是一个用来实现基于状态的UI组件的JavaScript库（妈呀，有点绕）。


  