---
layout: post
title: "React的思考（四）- componentDidMakeSense之生命周期面试调侃"
date: 2018-04-21 20:51:50 +0800
comments: true
categories:
---

看到一个老外的博客用了这样一个标题《componentDidMakeSense — React Component Lifecycle Explanation》，我只能说：老哥你太有才了，本来都不太想写这篇文章，冲你这个标题，我再啰嗦下生命周期。

###没有水平的面试官老喜欢问的问题

“你能说一下React的生命周期函数调用过程吗？”

“大哥，是不是我背出来，你就录用我？不是的话，你给我10秒钟，网不卡的话，我立马Google给你答案”

手贱，给你们扩皮了一份。
**Mounting**

constructor()    
static getDerivedStateFromProps()    
componentWillMount() / UNSAFE_componentWillMount()    
render()    
componentDidMount()

**Updating**

componentWillReceiveProps() / UNSAFE_componentWillReceiveProps()    
static getDerivedStateFromProps()    
shouldComponentUpdate()    
componentWillUpdate() / UNSAFE_componentWillUpdate()    
render()   
getSnapshotBeforeUpdate()    
componentDidUpdate()    

**Unmounting**

componentWillUnmount()

**Error Handling**

componentDidCatch()

###你不如这样问

你那么问别人问题，既不实际，也很尴尬，别人还觉得你没水平。不如我们结合实际问些问题：

1.组件需要做一次网络请求来获取数据，请问应该怎么写？组件有一些事件订阅放在哪个位置比较合适？为什么？

2.在哪些生命周期函数里面我不应该调用setState？如果调用了，会导致什么样的问题？

3.如果我这么写constructor来初始化对象会有什么问题？
```JavaScript
constructor(props) {
  super(props);
  this.state = {
    color: props.color
  };
}
```

4.你以前的经历中，用到了哪些生命周期函数？遇到过什么奇怪的问题没有？

等等，这样会显得你比较有水平，如果朋友们还问过其他类型的问题？请不惜赐教！我会好好收藏的。

###其他参考资料

官方文档其实就是不错的和最准确的 [the-component-lifecycle][32fd71f6]

  [32fd71f6]: https://reactjs.org/docs/react-component.html#the-component-lifecycle "the-component-lifecycle"

另外推荐看 https://reactarmory.com/guides/lifecycle-simulators

另外，放心，我这些面试问题都没有给答案，不会被套路的，大不了，我换个方式问。
