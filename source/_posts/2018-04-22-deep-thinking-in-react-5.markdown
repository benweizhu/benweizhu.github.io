---
layout: post
title: "React的思考（五）- Reconciliation"
date: 2018-04-22 12:33:42 +0800
comments: true
categories:
---

### Reconciliation的含义本身

Reconciliation有的人翻译成“协调算法”，有的人翻译成“一致性对比”，在没有官方答案之前，我认为直译可能会比较准确，它的作用是React用来区分一棵节点树和另一棵节点树的算法，以确定哪些部分需要更改。

Reconciliation是通常被理解为“虚拟DOM”背后的算法。

简单描述就是：当您渲染React应用时，会生成描述该应用的节点树并将其保存在内存中。然后将该节点树刷新到渲染环境 - 例如，在浏览器应用程序的情况下，它会转换为一组DOM操作。当应用程序更新（比如，通过setState）时，会生成一棵新树。对比得到新树与前一棵树的区别，以计算需要更新渲染应用的操作。

### React 16+的Reconciler - React Fiber

React团队在2016年7月公开发布React Fiber，React新的核心算法，React Fiber的目标是提高其对动画，布局和手势等领域的适用性。它的特征是增量渲染：能够将渲染工作分割成块并将其分散到多个帧中。

按照官方文档的说法，Fiber Reconciler的主要目标：

- 能够将可中断的任务拆分成块。
- 能够对进程中的工作划分优先级、重新设定基址（Rebase）、恢复。
- 能够在父子之间来回反复，借此为React的Layout提供支持。
- 能够通过render()返回多个元素。
- 为错误边界提供了更好的支持。

这个项目持续的2年之久，蕴含着过去多年来Facebook不断改进的工作成果。该架构可向后兼容，彻底重写了React的协调（Reconciliation）算法。他们还专门做了一个网站叫：http://isfiberreadyyet.com/

![Alt text](/images/isfiberready.jpg =500x "isfiberreadyyet")      
27 September 2017

我们都知道DOM只是React可以渲染的渲染环境之一，另外一个就是React Native。（这就是为什么“虚拟DOM”有点用词不当）。

它可以支持如此多环境的原因是因为React的设计使是Reconciliation和渲染是分开的阶段。Reconciler执行计算树的哪些部分已经改变; 渲染器然后使用该信息实际更新呈现的应用。这种分离意味着React DOM和React Native可以在共享由React核心提供的相同Reconciler的同时使用它们自己的渲染器。

### React 15以及以前的Reconciler - Stack Reconciler

Fiber在React 16中首次登场，发布时间是2017年9月26号，那么意味着，在这之前，有另外一套Reconciliation的算法。React现在把它命名为Stack Reconciler。它存在于React 15及更早版本的实现中。

**Stack Reconciler犯了一个单线程或者存在UI主线程环境下的“禁忌” - 用同步的方式来处理整个组件树。**

Virtual DOM diff会一次性处理整个组件树，重点在于，Stack Reconciler始终会一次性地同步处理整个组件树。因为整个过程都是在内存中完成，所以当组件树比较小的时候的，不会感觉到问题，但是，当组件树比较庞大的时候，就会出现卡顿（掉帧）的情况。

![Alt text](/images/Stack.jpg =500x "Stack Reconciler")     
单线程进入到栈中，要从栈从退出来，才能响应其他用户操作

![Alt text](/images/Fiber.jpg =500x "Fiber Reconciler")     
按照时间片段的方式执行

参考：[Lin Clark - A Cartoon Intro to Fiber - React Conf 2017][406aeeee]

  [406aeeee]: https://www.youtube.com/watch?v=ZCuYPiUIONs "Lin Clark - A Cartoon Intro to Fiber - React Conf 2017"

### 会不会有影响（参考官方文档）

首先，不变的地方是，diff节点或者说判断两个节点是否相同的方式没有变：

1.不同的元素类型

每当根元素具有不同类型时，React就会销毁旧的树并从头开始构建新树。从a到img ，或者从Article到Comment，从Button到div – 这些都将导致全部重新构建。

```HTML
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```
2.相同DOM元素类型，有不同的属性

当比较两个相同类型的React DOM元素时，React检查它们的属性（attributes），保留相同的底层DOM节点，只更新发生改变的属性（attributes）。

```HTML
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

3.子元素递归

默认情况下，当递归一个DOM节点的子节点时，React只需同时遍历所有的孩子基点同时生成一个改变当它们不同时。

例如，当给子元素末尾添加一个元素，在两棵树之间转化中性能就不错:

```HTML
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```
React 会比较两个 li.first 树与两个 li.second 树，然后插入 li.third 树。

如果在开始处插入一个节点也是这样简单地实现，那么性能将会很差。例如，在下面两棵树的转化中性能就不佳。

```HTML
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```
React 将会改变每一个子节点而没有意识到需要保留 li.Duke 和 li.Villanova 两个子树。这种低效是一个问题。

为了解决这个问题，React 支持一个 key 属性（attributes）。当子节点有了 key ，React 使用这个 key 去比较原来的树的子节点和之后树的子节点。例如，添加一个 key 到我们上面那个低效的例子中可以使树的转换变高效：

```
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```
现在 React 知道有'2014' key 的元素是新的， key为'2015' 和'2016'的两个元素仅仅只是被移动而已。

**关于key的使用，这个key需要在它的兄弟节点中是唯一的就可以了，不需要是全局唯一。而且必须唯一可以表示该节点，比如：不能使用index。**

据说，会影响到生命周期的调用，就目前我从官方网站上看到的，还没有正式提出哪些变化，所以可以暂时不用慌张。

### 总结

本来只是想借助Reconciliation来讲最后一点关于key的使用，不知道怎么写成了介绍Reconciliation的历史过程，所以Copy Paste的比较多。不过，从整体看来，Fiber引入时间片的异步更新，确实改进不少页面渲染的性能问题。

另外，[Dan Abramov: Beyond React 16 | JSConf Iceland 2018][1744a8f7]有简单介绍关于这个Fiber的异步更新功能启用后的页面渲染速度演示，但目前这个功能还没有开启。

  [1744a8f7]: https://www.youtube.com/watch?v=nLF0n9SACd4&list=PL37ZVnwpeshEO7qXEbjG4riQD7SzydLEO&index=2 "Dan Abramov: Beyond React 16 | JSConf Iceland 2018"
