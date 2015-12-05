---
layout: post
title: "鱼和熊掌的故事 - CSS Modules还是BEM"
date: 2015-12-05 11:39:48 +0800
comments: true
categories:
---
首先还是最基础的两个问题：什么是BEM？什么是CSS Modules？

##BEM(Block Element Modifer)

https://en.bem.info/

![Alt text](http://segmentfault.com/img/bVbN4T)

BEM的意思就是块（block）、元素（element）、修饰符（modifier），是由Yandex团队提出的一种**前端命名方法论**。这种巧妙的命名方法让你的CSS类对其他开发者来说更加透明而且更有意义。BEM命名约定更加严格，而且包含更多的信息，它们用于一个团队开发一个耗时的大项目。

{% codeblock lang:css %}
.block{}
.block__element{}
.block--modifier{}
{% endcodeblock %}

.block 代表了更高级别的抽象或组件。   
.block\_\_element 代表.block的后代，用于形成一个完整的.block的整体。   
.block--modifier代表.block的不同状态或不同版本。    
.block\_\_element--modifier代表.element的不同状态或不同版本。    

我们用一个搜索栏为例子：
{% codeblock lang:html %}
<form class="site-search full">
  <input type="text" class="field">
  <input type="Submit" value ="Search" class="button">
</form>  
{% endcodeblock %}

上面的这个CSS类名真是太不精确了，比如field，并不能告诉我们足够的信息。尽管我们可以用它们来完成工作，但它们确实非常含糊不清。用BEM记号法就会是下面这个样子：
{% codeblock lang:html %}
<form class="site-search site-search--full">
  <input type="text" class="site-search__field">
  <input type="Submit" value="Search" class="site-search__button">
</form>
{% endcodeblock %}

我们能清晰地看到有个叫.site-search的块，它的内部是一个叫.site-search\_\_field的元素。并且.site-search还有另外一种形态叫.site-search--full。如果搜索栏还有验证，那么它还有一个error状态，可以用site-search\_\_field--error。

虽然上面是个简单地例子，但相信你已经可以看到BEM带来的好处是命名独立，有意义，且可以清晰的表示模块结构。

BEM（或BEM的变体）是一个非常有用，强大，简单的命名约定，以至于让你的前端代码更容易阅读和理解，更容易协作，更容易控制，更加健壮和明确而且更加严密。

##CSS Modules

在React中，我们以Web Component的方式实现应用。组件（Component）的概念中有一个很重要特性：完整和自包含，而对于一个完整的Web Component，包含HTML，JAVASCRIPT和CSS。

React通过JSX实现了在JavaScript中写HTML，但是还缺少一个重要的元素：CSS。

在2014年11月的NationJS上Christopher Chedeau谈到了“CSS in JS”的话题。给许多人带了思想上的一个冲击，也让React实现完整Web Component带来的曙光。

现在，已经有了三种最新的，最明智和最可行的实现React样式的方式。

React Style： https://github.com/js-next/react-style     
JSX Style： https://github.com/petehunt/jsxstyle     
Radium： https://github.com/FormidableLabs/radium    

####一般情况下，如果项目中有大量的CSS，会出现什么问题？如图

![Alt text](http://glenmaddern.com/assets/images/7_problems_css.jpg)

Christopher指出，如果你将样式都挪到JavaScript中，这些问题都有很好地解决方案，这点说的没错，但是会引入复杂度，以及不太习惯的CSS使用方式。CSS Modules团队觉得可以让CSS还是保持以前的样子，但是构建时，以style-in-JS的方式实现。

CSS Modules的项目地址： https://github.com/css-modules/css-modules

###默认就是局部命名空间
在CSS Modules中每个文件都是独立编译，你可以使用更为简单地类命名方式，而不用担心它会污染全局。

我们以不同状态的Button为例，在没有CSS Modules的情况下，你可能会以Suit/BEM的方式来写CSS。

{% codeblock lang:css %}
/* components/submit-button.css \*/
.Button { /* all styles for Normal \*/ }
.Button--disabled { /* overrides for Disabled \*/ }
.Button--error { /* overrides for Error \*/ }
.Button--in-progress { /* overrides for In Progress \*/}
{% endcodeblock %}

{% codeblock lang:html %}
<button class="Button Button--in-progress">Processing...</button>
{% endcodeblock %}

使用CSS Modules的方式，你就可以用下面这种方式来写：

{% codeblock lang:css %}
/* components/submit-button.css \*/
.normal { /* all styles for Normal \*/ }
.disabled { /* all styles for Disabled \*/ }
.error { /* all styles for Error \*/ }
.inProgress { /* all styles for In Progress \*/
{% endcodeblock %}

你应该发现，class的前缀button没有了，为什么？因为这个css文件的名字已经叫做submit-button.css，CSS Modules已经知道它的前缀，不需要再次定义了。

{% codeblock lang:javascript %}
/* components/submit-button.js \*/
import styles from './submit-button.css';
buttonElem.outerHTML = `<button class=${styles.normal}>Submit</button>`
{% endcodeblock %}

以JavaScript的方式使用CSS。

然后，你再看看生成的最终页面的样子：
{% codeblock lang:html %}
<button class="components_submit_button__normal__abc5436">
  Processing...
</button>
{% endcodeblock %}
最终生成的class的名字是Component Name + Class Name + Base64，保证了层次结构和唯一性。

##鱼和熊掌的问题

对比一下BEM和CSS Modules，它们最终的目的都是给你带来独立，唯一和有意义的类命名。CSS Modules允许你以变量的方式将class获取并使用到元素上，并且省略了类名中Component Name前缀（也就是BEM中的B）。

在React中，Component一般会由多个元素组成（除了像Button这样比较简单地组件），就以上面的搜索框为例，在React中以CSS Modules的方式写，就有可能如下：

{% codeblock lang:javascript %}
import styles from './site-search.css';
//省略React代码
<form className={styles.full}>
  <input type="text" className={styles.field}>
  <input type="Submit" value ="Search" class={styles.button}>
</form>  
{% endcodeblock %}

对比一下BEM方式

{% codeblock lang:javascript %}
import from './site-search-bem.css';//以BEM方式写的样式表
//省略React代码
<form className="site-search site-search--full">
  <input type="text" className="site-search__field">
  <input type="Submit" value="Search" className="site-search__button">
</form>
{% endcodeblock %}

CSS Modules好像挺完美的，变量的方式传递class，在JavaScript的语境下似乎更有道理，但从本质上来说，与BEM相比，也就只是省略了Block这一个前缀，在这个看似好像很不错的方案下，有没有存在的问题？

##问题

####测试

在React的环境下，会使用Jest框架对组件进行测试，import styles from './site-search.css';这一个语句，并不是JavaScript的标准，只是CSS Modules的实现者，帮助你进行了编译，但Jest并不会编译。

所以这个时候，我们会在Jest的PreProcessor中处理这句话，这样才不会有语法解析错误。

{% codeblock lang:javascript %}
var babel = require('./babel-jest');

module.exports = {
	process: function (src, filename) {
		if (/\.(css|scss)$/.test(filename)) {
			return '';
		} else {
			return babel.process(src, filename);
		}
	}
};
{% endcodeblock %}

但与此同时，styles会变成空对象，由于我们使用了变量的方式，在测试中，并不能测试className的改变（因为state改变，去改变className）。相反，BEM因为是纯粹的字符串，所以它使可测试的。

##在React中引入classnames模块
我们知道，在React中，通过state的改变来决定组件的样式变化，在没有外部力量帮助的情况下，你的代码就会如下：

{% codeblock lang:javascript %}
/* components/submit-button.jsx \*/
import { Component } from 'react';
import styles from './submit-button.css';

export default class SubmitButton extends Component {
  render() {
    let className, text = "Submit"
    if (this.props.store.submissionInProgress) {
      className = styles.inProgress
      text = "Processing..."
    } else if (this.props.store.errorOccurred) {
      className = styles.error
    } else if (!this.props.form.valid) {
      className = styles.disabled
    } else {
      className = styles.normal
    }
    return <button className={className}>{text}</button>
  }
}
{% endcodeblock %}

这个时候，可以引入classnames模块，例子来自classnames样例：

{% codeblock lang:javascript %}
var classNames = require('classnames');

var Button = React.createClass({
  // ...
  render () {
    var btnClass = classNames({
      'btn': true,
      'btn-pressed': this.state.isPressed,
      'btn-over': !this.state.isPressed && this.state.isHovered
    });
    return <button className={btnClass}>{this.props.label}</button>;
  }
});
{% endcodeblock %}
问题是，classNames中传入的JavaScript对象，不可以用styles.normal作为key，如果要在CSS Modules的环境下，使用classnames，你需要像下面这样写：

{% codeblock lang:javascript %}
import styles from './site-search.css';
var classNames = require('classnames');

var Button = React.createClass({
  // ...
  render () {
    var btnClass = classNames({
      [styles['btn']]: true,
      [styles['btn-pressed']]: this.state.isPressed,
      [styles['btn-over']]: !this.state.isPressed && this.state.isHovered
    });
    return <button className={btnClass}>{this.props.label}</button>;
  }
});
{% endcodeblock %}

如果你不使用classnames，在使用CSS Modules也是可以用styles['normal']来代替styles.normal。

##无意中的发现，结合CSS Modules和BEM方式

通过用styles['normal']来代替styles.normal，在定义CSS的类名时，你依旧可以使用你习惯的CSS类命名方式，比如：BEM。上面的搜索框的例子，就可以写成：

{% codeblock lang:javascript %}
import styles from './site-search.css';
import classNames from 'classnames';
//省略React代码

var fieldClass = classNames({
  [styles['field']]: !this.state.isError,
  [styles['field__error']]: this.state.isError,
});

<form className={styles[full]}>
  <input type="text" className={fieldClass}>
  <input type="Submit" value="Search" className={styles['submit']}>
</form>
{% endcodeblock %}

这种方式，既保证了CSS Modules的使用，也保留了BEM通过三种基本元素来描述组件样式的方式。

##总结
对比两种实现方式，从本质上来讲，并没有区别，否则它们也不可能结合在一起，但是CSS Modules带来的致命伤害是作为一个变量，并不能直接测试（想要测试，可以让Jest的路径指向已经编译过的JavaScript路径）。而纯粹的BEM因为是字符串可以很好地和classnames结合使用，也可以进行测试，所以在选择上占据了很大的优势。

参考资料：    
1.http://glenmaddern.com/articles/css-modules    
2.http://segmentfault.com/a/1190000000391762    
3.https://github.com/css-modules/css-modules    
4.https://www.npmjs.com/package/classnames     
