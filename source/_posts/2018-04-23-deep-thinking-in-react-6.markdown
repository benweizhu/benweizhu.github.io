---
layout: post
title: "React的思考（六）- 不可变性"
date: 2018-04-23 22:29:51 +0800
comments: true
categories:
---
## 为什么immutable在React中那么重要？

我尝试用简（kuo）单（hao）易（li）懂（mian）的词解释

1.可以给你的React应用带来性能提升（不用深度对比）   
2.简单的编程和调试体验，少出bug（不会操作共享对象）    
3.数据更容易追踪，推导（保留previewState可以对比）

Talk is shit, show me your money.

光说还是差了点，虽然很通俗了，但是还是看下面一段代码，我用一个React初学者，特别容易犯的一个错误来说明这三个问题，请看题板：

```JavaScript
updateState(event) {
  const {value} = event.target;
  let user = this.state.user;
  user.name = value;
  this.setState({user});
}
```

```JavaScript
updateState(event) {
  const {value} = event.target;
  let user = Object.assign({}, this.state.user, {name: value});
  this.setState({user});
}
```

请问，第一种写法有什么问题？你有20秒的时间思考并作答，对不起，由于我没法遮挡住答案，请自觉闭眼思考。

答案：就是上面说的三点，举个例子：shouldComponentUpdate(nextProps, nextState)，需要对比nextState和this.state的区别来决定是否渲染，但此时this.state已经不是以前的state了状态了，特别是PureComponent的shallow compare，直接导致组件不重新渲染，会出bug

## 为什么immutable在Redux中那么重要？

因为不用就会出bug，因为你就会问“为什么UI不更新”，因为Redux和React-Redux都使用了浅比较。

具体行为体现在两处，一个在按照Domain区分的combineReducers那，一个在使用数据被connect包裹的组件那：

**1.Redux的combineReducers方法浅比较它调用的reducer的引用是否发生变化**

比如:

```JavaScript
combineReducers({ todos: myTodosReducer, counter: myCounterReducer })
```

combineReducers会遍历所有这些键值对，判断每一个reducer执行返回结果后的引用是否发生变化（所以是浅比较），如果其中一个变化了，就会设置一个标志位hasChanged为true，当遍历结束，如果hasChanged为true，则返回新的conbineReducers合并的总的state，否则返回旧的那个state（这个新旧的state在React-Redux中会用到）。

这就是为什么在reducer当中要使用Object.assign({},state,{..}}来返回一个新的state，如果你直接操作state，并返回它，combineReducers会认为它没有改变。

```JavaScript
function myTodosReducer(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```

**2.React-Redux的state和mapStateToProps**

React-Redux的connect方法生成的组件，会假设包装的组件是一个“纯”（pure）组件，即给定相同的props和state，这个组件会返回相同的结果。做出这样的假设后，React-Redux就只需检查根state对象或mapStateToProps的返回值是否改变。如果没变，包装的组件就无需重新渲染。

我们回忆一下，上面说道，combineReducers会决定是否返回新的根state，而每次调用React-Redux提供的connect函数时，它之前储存的根state对象的引用，会与当前传递给store的根state对象之间进行浅比较。如果相等，说明根state对象没有变化，也就无需重新渲染组件，甚至无需调用mapStateToProps。

如果不相等，则connect会调用mapStateToProps来，并查看最后传给组件的props是否被更新。同样，这里也是一个浅比较，它要对比的是这个对象的第一层引用是否变化。

```JavaScript
function mapStateToProps(state) {
  return {
    todos: state.todos, // prop value
  }
}

export default connect(mapStateToProps)(TodoApp)
```
即，这个对象的todos是否变化。
```JavaScript
{
  todos: state.todos, // prop value
}
```

比如，如果在todos的reducer返回的state没有变化，那么这里的todos也就是没有变化，因此组件就不需要渲染。

mapStateToProps因为它特殊的作用，很容易出现一种反模式，我们需要注意，就是像下面这样写：

```JavaScript
function mapStateToProps(state) {
  return {
    todos: {
      all: state.todos,
      visibleTodos: getVisibleTodos(state)
    }
  }
}

export default connect(mapStateToProps)(TodoApp)
```
todos对象永远拿到的都是新的对象{}，而不是直接由reducer里面返回的对象，所以组件一定会重新渲染，无论state是否变化。

**React-Redux这样设计是有道理的**

因为Redux本身的CombineReducers只会决定最根节点的state有没有变化，也就是这个{ todos: myTodosReducer, counter: myCounterReducer }里面的存不存在变化，然后决定返回新的或者旧的，而每一个connect的组件拿到这个新的根state，首先判断state有没有变化，然后判断这个变化和它想要的数据（一个或者多个reducer）有没有关，如果没有关系，当然就不用重新渲染。

## 总结

React和Redux之所以要求使用Immutable，原因的初衷是性能，避免深度对比。对于我们使用React和Redux的开发人员，除了关注性能，更是因为需要遵循React和Redux的编程规范来写出正确的代码。
