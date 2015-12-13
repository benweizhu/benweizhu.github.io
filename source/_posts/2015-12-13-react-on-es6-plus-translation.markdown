---
layout: post
title: "翻译 React on ES6+"
date: 2015-12-13 10:32:12 +0800
comments: true
categories:
---
原文地址： http://babeljs.io/blog/2015/06/07/react-on-es6-plus/

###文章翻译有些不准确，敬请见谅

当我们正在从内到外的重新设计Instagram Web的时候，我们非常享受使用许多ES6+的特性来编写React组件。这让我有机会去说明这些新的语言特性可以改变你写React应用的方式，让它变得更简单也更有趣。

##Classes

到目前为止，最明显的变化就是当我们选择使用ES6+中的类定义语法时，如何来编写React组件。相对于使用React.createClass方法来定义一个组件，我们可以使用真正的ES6类来继承React.Component：

{% codeblock lang:javascript %}
class Photo extends React.Component {
  render() {
    return <img alt={this.props.caption} src={this.props.src} />;
  }
}
{% endcodeblock %}

立马，你就会注意到一个微妙的不同 - 当定义类时，使用一个更加简洁的语法：

{% codeblock lang:javascript %}
// The ES5 way
var Photo = React.createClass({
  handleDoubleTap: function(e) { … },
  render: function() { … },
});
// The ES6+ way
class Photo extends React.Component {
  handleDoubleTap(e) { … }
  render() { … }
}
{% endcodeblock %}

很明显，我们丢掉了两个括号和一个分号，而且每一个方法声明忽略了一个冒号，一个function关键字和一个逗号。

当使用新的类语法时，所有的生命周期方法（除了一个）都可以像你所期望的那样定义。类的构造函数现在的角色，之前是由componentWillMount来扮演：

{% codeblock lang:javascript %}
// The ES5 way
var EmbedModal = React.createClass({
  componentWillMount: function() { … },
});
// The ES6+ way
class EmbedModal extends React.Component {
  constructor(props) {
    super(props);
    // Operations usually carried out in componentWillMount go here
    // 所有componentWillMount的操作都放在这里
  }
}
{% endcodeblock %}

##属性初始化

在ES6+的类世界，属性类型和默认值都是作为类自己的静态属性。同样，Component的状态初始化可以使用ES7的属性初始化：
{% codeblock lang:javascript %}
// The ES5 way
var Video = React.createClass({
  getDefaultProps: function() {
    return {
      autoPlay: false,
      maxLoops: 10,
    };
  },
  getInitialState: function() {
    return {
      loopsRemaining: this.props.maxLoops,
    };
  },
  propTypes: {
    autoPlay: React.PropTypes.bool.isRequired,
    maxLoops: React.PropTypes.number.isRequired,
    posterFrameSrc: React.PropTypes.string.isRequired,
    videoSrc: React.PropTypes.string.isRequired,
  },
});
// The ES6+ way
class Video extends React.Component {
  static defaultProps = {
    autoPlay: false,
    maxLoops: 10,
  }
  static propTypes = {
    autoPlay: React.PropTypes.bool.isRequired,
    maxLoops: React.PropTypes.number.isRequired,
    posterFrameSrc: React.PropTypes.string.isRequired,
    videoSrc: React.PropTypes.string.isRequired,
  }
  state = {
    loopsRemaining: this.props.maxLoops,
  }
}
{% endcodeblock %}

ES7属性初始化操作在类的构造函数中，这里指向的是类实例的构造，所以state的初始化可以设置为依赖于this.props。值得注意的是，我们不在需要，针对getter方法，定义prop的默认值，和初始化state对象。

##箭头函数

React.createClass方法用来在组件实例方法上执行一些额外的绑定工作来保证，在他们里面，this关键字可以被考虑指向组件的实例。

{% codeblock lang:javascript %}
// Autobinding, brought to you by React.createClass
var PostInfo = React.createClass({
  handleOptionsButtonClick: function(e) {
    // Here, 'this' refers to the component instance.
    this.setState({showOptionsModal: true});
  },
});
{% endcodeblock %}

既然我们不使用React.createClass方法了，当我们用ES6+的类语法定义组件时，似乎我们就需要在我们想要这些行为时，手动的绑定实例方法：

{% codeblock lang:javascript %}
// Manually bind, wherever you need to
class PostInfo extends React.Component {
  constructor(props) {
    super(props);
    // Manually bind this method to the component instance...
    this.handleOptionsButtonClick = this.handleOptionsButtonClick.bind(this);
  }
  handleOptionsButtonClick(e) {
    // ...to ensure that 'this' refers to the component instance here.
    this.setState({showOptionsModal: true});
  }
}
{% endcodeblock %}

幸运的是，通过结合两个ES6+的特性 - 箭头方法和属性初始化 - 选择性的绑定到组件实例变得轻而易举：

{% codeblock lang:javascript %}
class PostInfo extends React.Component {
  handleOptionsButtonClick = (e) => {
    this.setState({showOptionsModal: true});
  }
}
{% endcodeblock %}

The body of ES6 arrow functions share the same lexical this as the code that surrounds them, which gets us the desired result because of the way that ES7 property initializers are scoped. Peek under the hood to see why this works.

##动态属性名和模板字符串

对对象字面量的一个增强是，拥有给一个衍生而来的属性名赋值的能力。以前，我们可能需要像下面这样做来设置state对象：
{% codeblock lang:javascript %}
var Form = React.createClass({
  onChange: function(inputName, e) {
    var stateToSet = {};
    stateToSet[inputName + 'Value'] = e.target.value;
    this.setState(stateToSet);
  },
});
{% endcodeblock %}

现在，我们可以构建那些属性名由JavaScript表达式在运行时决定的对象。这里，我们用模板字符串来决定将哪个属性设置到state上：
{% codeblock lang:javascript %}
class Form extends React.Component {
  onChange(inputName, e) {
    this.setState({
      [`${inputName}Value`]: e.target.value,
    });
  }
}
{% endcodeblock %}

##析构和JSX扩展属性（spread attributes)

Often when composing components, we might want to pass down most of a parent component's props to a child component, but not all of them. In combining ES6+ destructuring with JSX spread attributes, this becomes possible without ceremony:
常常当我们组合组件时，我们也许想要将父组件的大部分属性传递到子组件中，但是并不是全部。结合ES6+的destructuring和JSX spread attributes，这变成可能且不太复杂：

{% codeblock lang:javascript %}
class AutoloadingPostsGrid extends React.Component {
  render() {
    var {
      className,
      ...others,  // contains all properties of this.props except for className
    } = this.props;
    return (
      <div className={className}>
        <PostsGrid {...others} />
        <button onClick={this.handleLoadMoreClick}>Load more</button>
      </div>
    );
  }
}
{% endcodeblock %}

我们可以将JSX扩展属性和正常的属性相结合，利用简单地优先级规则来实现复写和默认值指定。下面这个元素会获得className “override”，即便className属性已经在this.props中定义：
{% codeblock lang:javascript %}
<div {...this.props} className="override">
  …
</div>
{% endcodeblock %}
这个元素会正常的拥有className “base”，除非在this.props中存在一个className属性复写了它：
{% codeblock lang:javascript %}
<div className="base" {...this.props}>
  …
</div>
{% endcodeblock %}
