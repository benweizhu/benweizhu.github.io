---
layout: post
title: "React的思考（八）- Redux的Middleware（下）异步的世界"
date: 2018-04-27 13:38:01 +0800
comments: true
categories:
---
## 有序而独立的同步世界

没有异步的情况下，Redux配合React很容易理解的（Action->Reducer->CombineReducers->React-Redux->Component），简单回顾下：

1.在组件里面dispatch（发出）一个action对象（带上类型和数据）    
2.action对象被传递到reducer的入口，reducer根据类型给到不同的switch分支，然后根据带入的数据操作state，返回新的state     
3.redux发现有新的state，配合React-Redux，通知所有component，告诉组件请注意你自己要不要更新，然后各自判断各自更新

一个被传递到组件里的disptach
```JavaScript
const mapDispatchToProps = dispatch => {
  return {
    onAddTodo: text => {
      dispatch(addTodo(text))
    }
  }
}
const addTodo = (text) => {
  type: 'ADD_TODO',
  text
}

//在组件里面点击时触发
handleClickAdd() {
  this.props.onAddTodo(this.state.todoText)
}

```

action被传递到reducer的入口
```JavaScript
function todoApp(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO://命中这个switch case
      return Object.assign({}, state, {//返回新的state
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    default:
      return state
  }
}
```

组件被通知请查看你是否需要更新，connect发现todos变了，所以要更新这个connect嵌入的组件
```JavaScript
const mapStateToProps = state => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
```

## 没有Redux的异步世界

异步世界其实没什么可怕的（又不是异世界），看下面一个React里面用fetch实现的数据异步加载：

```JavaScript
class ExampleComponent extends React.Component {
  constructor(props){
    super(props);
    this.state = { name: '', loading: false, errorMessage: '' };
    this._getRandomName = this.getRandomName.bind(this);
  }

  render() {
    const { name, errorMessage } = this.state;
    return(
      <div>
        <h1>{name}</h1>
        <button onClick={this._getRandomName}>PRESS ME!</button>
        { errorMessage && <div>{errorMessage}</div> }
      </div>
    );
  }

  getRandomName() { // 重点是这么一段代码
    this.setState({loading: true})
    fetch('https://randomuser.me/api/')
      .then(response => response.json())
      .then(data => {
        const person = data.results[0];
        this.setState({ name: `${person.name.first} ${person.name.last}`, loading: false })
      })
      .catch(reason => {
        this.setState({errorMessage:'get name failed', loading: false})
      });
  }
}
```

过程很简单，我们需要考虑三种页面状态：请求开始和进行中，请求成功，请求失败，然后分别设置组件的state。

## 如果没有Redux异步中间件

按照“没有Redux的异步世界”的思想，在Redux里面，我们仍然可以依葫芦画瓢的进行异步的Redux的操作。

首先，一个异步请求都需要dispatch至少三种action，对应至少三个不同的状态：

1.通知reducer请求开始     
2.通知reducer请求成功     
3.通知reducer请求失败

```JavaScript
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { name: 'Redux'} }
```

如果没有Redux异步中间件，那么你的做法和没有Redux时是类似的，你需要在mapDispatchToProps那传入三个disptach，将异步的fetch逻辑放在组件里面实现，Redux本身仍然是处理同步的state操作。

```JavaScript
class ExampleComponent extends React.Component {
  ...
  getRandomName() {
    this.props.fetchRequest(true)
    fetch('https://randomuser.me/api/')
      .then(response => response.json())
      .then(data => {
        const person = data.results[0];
        this.props.fetchSuccess(`${person.name.first} ${person.name.last}`, false)
      })
      .catch(reason => {
        this.props.fetchFailure('get name failed', false)
      });
  }
}
```

## 使用Redux-Thunk的不同点在哪？

因为Redux官网推荐，我们就以Redux-Thunk为例。

Redux-Thunk刚刚引入的时候，往往容易让使用者有些感到混乱，一个原因是函数式编程的嵌套写法，第二个是和Redux之前dispatch函数做的事情不一样了。

其实，没有Redux-Thunk我们已经可以处理异步请求，只不过异步逻辑不在Redux里面，而是在组件里面，如果我们加入Redux-Thunk会有什么不同呢？

我在[上面篇文章][4076f240]讲过，中间件的作用是在dispatch的附近做一些额外的操作，让Redux拥有不同的能力，Redux-Thunk中间件的能力，可以让action creater，不用返回一个action对象，而是一个函数，这个action创建的函数就成为一个thunk。

  [4076f240]: http://benweizhu.github.io/blog/2018/04/25/deep-thinking-in-react-7/ "上面篇文章"

（关于Thunk函数的含义：编译器的"传名调用"实现，往往是将参数放到一个临时函数之中，再将这个临时函数传入函数体。这个临时函数就叫做Thunk函数。）

这个函数并不需要保持纯净，它还可以带有副作用，包括执行异步API请求。这个函数还可以dispatch action。

```JavaScript
//actions.js
export function fetchRandmonName() {
  return function (dispatch) {
    dispatch(fetchRequest(true))
    return fetch('https://randomuser.me/api/')
      .then(response => response.json())
      .then(data =>{
        const person = data.results[0];
        dispatch(fetchSuccess(`${person.name.first} ${person.name.last}`, false))
      })
      .catch(reason => {
        dispatch(fetchFailure('get name failed', false))
      })
  }
}

const mapDispatchToProps = dispatch => {
  return {
    fetchRandmonName: () => {
      dispatch(fetchRandmonName())
    }
  }
}

//ExampleComponent.js
getRandomName() {
  this.props.fetchRandmonName();
}
```

你看其实代码差不多，只不过，因为Redux-Thunk，**你可以将异步的处理逻辑，从组件里面拿出来，将它放在一个和Redux其他代码更加的内聚的位置**，也许是action的存放位置actions.js，而组件里面只需要dispatch一个thunk。

## 总结

关于Redux的异步世界，暂时更新到这里，Redux里面处理异步的中间件有好多，我就不一个个分析了，你肯定很关心这个，[《Redux异步方案选型》][6f63588d]

  [6f63588d]: http://react-china.org/t/redux/8761 "《Redux异步方案选型》"。推荐看React-China论坛上的这篇文章，分析还比较全面。
