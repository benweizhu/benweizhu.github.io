---
layout: post
title: "React的思考（三）- 总结下shouldComponentUpdate"
date: 2018-04-20 11:10:09 +0800
comments: true
categories:
---

Google来，百度去，原来网上已经有一大堆讲解shouldComponentUpdate的文章，差点就打算放弃了，为了学习精神，那我就集百家之长，小总结一下。

### 它的作用

首先，简单说一下shouldComponentUpdate的作用（如果你已经知道，请不要跳过，帮助我审查下有没有描述错误）

```JavaScript
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```
extends React.Component和写Functional Stateless Component（它不能复写shouldComponentUpdate），shouldComponentUpdate默认都是返回true。

这意味着，当props或者state更新时，该组件一定会调用render方法。

也就意味着，React一定会去对比该组件节点上的VisualDOM，但是，不一定会去更新真实DOM，因为reconciliation的结果可能是相等的。（一致性对比，对比新返回的元素是否和之前的元素是否一样）

如果，你将shouldComponentUpdate复写，返回false，那么componentWillUpdate()，render()和componentDidUpdate()都不会被调用，那么该组件不会更新。

当shouldComponentUpdate返回true，这个过程是向叶子节点传递的，比如：父节点返回true，它有两个叶子节点A和B，那么A和B会被要求执行mount或者update的生命周期，如果A的shouldComponentUpdate返回false，B返回true，那么只有B更新。

React官方的[ShouldComponentUpdate In Action][60462522]讲解的很清楚。

  [60462522]: https://reactjs.org/docs/optimizing-performance.html#shouldcomponentupdate-in-action "ShouldComponentUpdate In Action"

### 逃生出口

听起来貌似很有道理，谁不希望减少无谓的计算，提高性能。

然而，我又看到这样一句话：React team called shouldComponentUpdate an "escape hatch"（逃生出口）instead of "turbo button"（涡轮增压按钮）。在 github issue [Stateless functional components and shouldComponentUpdate][0cd4d7bb]上也有人这么说。
  [0cd4d7bb]: https://github.com/facebook/react/issues/5677 "Stateless functional components and shouldComponentUpdate"

听上去总结起来，有两个原因：

1.维护自定义的shouldComponentUpdate成本太高，有可能加了一个新的prop，但是忘记更新shouldComponentUpdate的代码，导致bug     
2.也许shouldComponentUpdate的比较计算逻辑比起直接重新render更加浪费性能

### 尴尬了

那这就尴尬了，这到底是写还是不写呢？当被问到这个问题的时候，永远都有一个正确但不受人欢迎的答案：“Well, it depends”（好吧，看情况）。

### React.Component，PureComponent和Function

与其思考这个没有人能够给出准确答案的问题，不如我们思考怎么样结合对shouldComponentUpdate的理解，合理的写组件，设计合理的状态结构树。

大家对React.Component和Functional Stateless Component比较熟，一个就是extends React.Component，一个就是函数，前面也说了，它们的shouldComponentUpdate永远返回true。

React里还有一个顶级的组件API：[PureComponent][61ff2738]。
  [61ff2738]: https://reactjs.org/docs/react-api.html#reactpurecomponent "PureComponent"

这个PureComponent，对shouldComponentUpdate有一个默认实现，官方称为shallow的prop和state对比。啥意思呢？就是帮你对比this.props和this.state上的第一层叶子节点的引用。

如果，某一个叶子节点里面深层的一个元素改变了，而该叶子节点本身的引用没变，shouldComponentUpdate是检查不出来的。

什么类型的组件比较适合写成PureComponent呢？比如：基础组件Button，它本身的属性就相对简单，完全可以和普通HTML的button元素相似，这样就可以将组件的属性拍平一层展现，用PureComponent正好满足shallow对比。

### 总结

一切不分析性能瓶颈而做的性能优化，都是无用功，shouldComponentUpdate不一定是你的性能瓶颈，但是，我们在这里讨论shouldComponentUpdate，为React组件的更新的问题开了一个头，后面介绍Redux和Object.assign还会在提到组件（不）更新的问题。

周五了，祝大家周末愉快。
