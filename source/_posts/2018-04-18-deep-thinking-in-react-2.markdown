---
layout: post
title: "React的思考（二）- 逃不开的生命周期函数之构造函数"
date: 2018-04-18 21:23:11 +0800
comments: true
categories:
---
这一定是一个老生常谈的话题，你们就别多想了，跟我一起回顾一遍，看我说的有没有道理。

```JavaScript
class Timer extends React.Component {
  constructor(props) {
    super(props);
    this.state = { seconds: 0 };
  }

  render() {
    return (
      <div>
        Seconds: {this.state.seconds}
      </div>
    );
  }
}

ReactDOM.render(<Timer />, mountNode);
```

###Constructor

构造函数（或者构造器），这个概念对于熟悉基于类的面向对象语言的朋友们肯定烂熟于心，但是对于JavaScript而言，这个概念往往容易让人困惑。

在JavaScript的世界里，构造函数和普通函数没有什么区别，你一样的可以像普通函数一样调用它，但如果通过new关键字来调用函数，该函数就成为了构造函数，this指针就会指向新创建的对象。

比如：

```JavaScript
function Person(){
  this.name = "CodePlayer";
  console.log(this)
}
Person()
$ Window {external: Object, chrome: Object, document: document, WPCOMSharing: Object, RecaptchaTemplates: Object…}

var b = new Person()
$ Person {name: "CodePlayer"}
```
更多具体的解释请参考我以前的一篇博客：[JavaScript渐入佳境 - 构造函数、new、原型][bafdb4d3]。

  [bafdb4d3]: http://benweizhu.github.io/blog/2015/12/31/javascript-contructor-new-prototype/ "JavaScript渐入佳境 - 构造函数、new、原型"

### ES6的Class、extends、super

上面是一个ES5的例子，然而，当我们写React代码时，我们会用到ES6语法，会用到class，constructor以及super关键字，他们的作用是什么？

我们先看下面一个跟React无关的例子：
```JavaScript
//ES6
class Rectangle {
    constructor (width, height) {
        this.width  = width;
        this.height = height;
    }
}

//翻译成ES5
var Rectangle = function (width, height) {
    this.width = width;
    this.height = height;
};

//ES6
class Shape {

}
class Rectangle extends Shape {
    constructor (id, x, y, width, height) {
        super(id, x, y);
        this.width  = width;
        this.height = height;
    }
}

//概念性翻译成ES5
var Share = function(){

}
var Rectangle = function (id, x, y, width, height) {
    Shape.call(this, id, x, y);
    this.width  = width;
    this.height = height;
};
Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;
Rectangle.__proto__ = Shape;
```
简单来说，class的作用就是定义Shape和Rectangle两个function（这就是被人用烂的词，语法糖），extends的作用是定义函数Rectangle的prototype和__proto__属性来实现原型链的继承，super的作用是在Rectangle函数中执行Share函数，并绑定this指针。

建议查看Babel的编译结果，它是更准确的ES5转义：[babel链接][04dbe491]

  [04dbe491]: https://babeljs.io/repl/#?babili=false&browsers=&build=&builtIns=false&code_lz=MYGwhgzhAEDKAWYAOBTaBvAUJgvp0kMASisAC5gB2A5iGigB5kqUAmMCyaW0v0wAe0oQyAJwCu5AaOgAKAJasANNAYqAnioDuisvBXwU86vDIBKDJj7XoEcalELlqjWYDcVm7z3yIAOh1WPV4AXmhAvQ8vb3hfP0NjU2gwhJMyKL48PCA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&lineWrap=true&presets=es2015&prettier=false&targets=&version=6.26.0&envVersion= "babel链接"

> 如果以后有人问我，JavaScript和Java有什么关系，我不会说它们没关系，我会告诉那个人，ES6抄袭Java的语法范式。

### React中的constructor

我们再来回到React上，看ES6和Babel编译后的结果：

```JavaScript
//ES6
class Rectangle extends React.Component{

}

//不完全Babel编译代码
function Rectangle() {
  _classCallCheck(this, Rectangle);

  return _possibleConstructorReturn(
    this,
    (Rectangle.__proto__ || Object.getPrototypeOf(Rectangle))
      .apply(this, arguments) // 看这里
}

//ES6
class Rectangle extends React.Component{
    constructor (props) {
      super(props);
    }
}

//不完全Babel编译代码
function Rectangle(props) {
   _classCallCheck(this, Rectangle);

   return _possibleConstructorReturn(
     this,
     (Rectangle.__proto__ || Object.getPrototypeOf(Rectangle))
       .call(this, props) // 看这里
}
```

当我们创建一个组件时，如果不需要在constructor里面做任何初始化的操作时，我们是不需要复写constructor的，因为Babel编译后，会将整个arguments都绑定上this指针后传递给被Rectangle的原型（React.Component），并执行，它替我们将constructor中super()的操作做了，如上所示。

如果有需要在constructor中做初始化的操作时，那么必须带上super(props)并放在最前面，因为它的作用是调用Rectangle的原型（React.Component），并绑定this指针。

那么，哪些初始化的操作可以在constructor里面做呢？原则上只有一个，那就是初始化state。

```JavaScript
class Timer extends React.Component {
  constructor(props) {
    super(props);
    this.state = { seconds: 0 };
  }
}
```
有三个问题：

1.为什么不用this.setState()来执行？

原因：1.this.state够直接了，你还要怎样？2.this.setState()是一个异步执行的函数，执行完之后，组件的响应式重新渲染（render），你这第一次渲染都还没开始呢。3.在这里this.setState()是一个空指令，这么写，不会任何起作用，不信你可以试试。

2.那么能不能在constructor里面执行网络请求来初始化数据？

我问过许多来面试的候选人，你的网络请求会放置在什么时候执行，我印象中确实有人回答我说在constructor中。听起来，在constructor中获取数据来初始化挺合理的。而且确实有人问：[Stack Overflow][6fdaa67d]

  [6fdaa67d]: https://stackoverflow.com/questions/39338464/reactjs-why-is-the-convention-to-fetch-data-on-componentdidmount "Stack Overflow"

然而，我们在官方文档上看到这样一句话：避免在构造函数中引入任何有副作用的代码（比如data fetching或者DOM manipulation）或执行订阅的操作，如果有，请在componentDidMount里执行。

关于这个位置的理解，我常常这样解释，就像你用jQuery写代码一样，你一定会等到document ready之后，才开始操作DOM或执行网络请求（也是为了操作DOM），否则，很有可能遇到undefined的情况。虽然，React这里也许和jQuery不一样，但我认为它的理由是相似的。（如果不对，请纠正我）。

3.那么可不可以传递props到state呢？非常像Java的写法。

```JavaScript
constructor(props) {
  super(props);
  this.state = {
    color: props.initialColor
  };
}

//Java
public Shape {
  Shape(String color){
    this.color = color;
  }
}
```
这么写，没有说完全错误，但是你需要注意的是，要同时实现getDerivedStateFromProps()在React 16中，或者16之前的componentWillReceiveProps，来保证当父组件更新时，props能有传递到state，因为constructor只会执行一次。

如果出现这种使用场景，我们需要思考一下，能否将state的控制向上提升，将Shape组件仅仅作为Presentional Component，这样减少在不同的两个位置（父组件和它自己）来控制组件的状态。

#### this.handleClick = this.handleClick.bind(this);

官方文档说，在constructor里面只做两件事情，初始化state，和绑定event的handler函数的this指针到组件对象本身。既然是this指针这么困惑的话题，我再啰嗦一句这里做了什么：

当函数（这里指这个组件）就成为了构造函数，该函数中this指针就会指向新创建的对象，也就是constructor里面的this就是指向的它自己（该组件的实例），那么this.handleClick = this.handleClick.bind(this);就能保证在handleClick函数里面的this指针，无论handleClick被传递到了哪里，可能被基础DOM元素button使用，也可能被子组件传递到别的位置，handleClick里面的this指针都能指向该组件的（如果是下面的例子就是Toggle），这样里面的this.setState才能起作用。

```JavaScript
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}
```

### 总结

别小看一个constructor函数，这里面的知识点可多了。总结一下，就是只干这些事情，其他的别干：
```JavaScript
constructor(props) {
  super(props);
  this.state = {isToggleOn: true};
  this.handleClick = this.handleClick.bind(this);
}
```
