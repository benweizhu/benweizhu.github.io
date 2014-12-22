---
layout: post
title: "使用jasmine为javascript写单元测试"
date: 2013-10-14 14:38
comments: true
categories: 
---

写这一篇关于Jasmine文章，是因为在正式加入ThoughtWorks之前，做过的一个偏前端的web项目。

项目的内容是对用户提供的数据进行计算，并得到一个报表，提供打印。

当时希望能够用github page作为空间，但我们都知道，github page是只支持前端，即页面和javascript。

所以在项目中，我使用js进行计算，使用xml作为数据库，使用cookie来确保用户意外退出浏览器时，不会丢失尚未完成计算的输入数据。

项目中最难的部分是对计算公式的选择，需要根据计算参数的不同条件选择出不同的计算公式。这里我给出一部分较为简单的公式选择逻辑作为说明。C和AF是条件也就是参数，PI是对应的选择公式。

	#   C		AF		PI
	1.	q   	q		1/q
	2.	pq		q		1/2q
	3.	q		qr		1/2q
	4.	pq		pq		(p+q)/4pq
	5.	pq		qr		1/4q

如果这个是一个Java或者C的项目，Ok，即使没有测试，我们也可以非常方便的通过控制台查看计算结果（如果内容多了，还是要写测试，因为要重构）。

但对于是js项目，如果没有测试帮助判断逻辑的正确性，将是非常痛苦的事情。

![Jasmine](http://pivotal.github.io/jasmine/images/jasmine_logo.png)

Jasmine是一种基于行为驱动的Javascript测试框架。它不依赖于其他的Javascript框架。

下面是一个简单的且常用的单元测试模式。

{% codeblock a junit test for js lang:javascript %}
describe("Test method", function() {
  it("should return true when condition is pp qq", function() {
    expect(someMethod('p','p','q','q')).toBe(true);
  });
});
{% endcodeblock %}

describe是对一系列单元测试的大描述。

it中的描述是这一个单元测试的描述。例如，我们可以在describe中写多个单元测试，例如：
{% codeblock a junit test for js lang:javascript %}
describe("Test method", function() {
  it("should return true when condition is pp qq", function() {
    expect(someMethod('p','p','q','q')).toBe(true);
  });
  it("should return false when condition is pp pp", function() {
    expect(someMethod('p','p','p','p')).toBe(true);
  });
});
{% endcodeblock %}

>下面进入正题，如何在项目中引入Jasmine。
jasmine库中一共包含4个重要文件。

jasmine.css   jasmine.js   jasmine-html.js   SpecRunner.html

剩下的是你自己的Javascript文件和测试文件。例如，假设js文件是
{% codeblock justdoit.js lang:javascript %}
function add(a, b) {
    return a + b;
}
{% endcodeblock %}

那么你对应的测试文件是 
{% codeblock justdoitSpec.js lang:javascript %}
describe("just do it", function () {
    it("should return 10 when 5 plus 5", function () {
        expect(add(5, 5)).toBe(10);
    })
})
{% endcodeblock %}

在写完测试代码之后，需要将对应的js文件引入在SpecRunner.html文件中。
{% codeblock justdoitSpec.js lang:javascript %}
<!-- include source files here... -->
<script type="text/javascript" src="src/justdoit.js"></script>

<!-- include spec files here... -->
<script type="text/javascript" src="spec/whatervername.js"></script>
{% endcodeblock %}

运行测试的方式是在浏览器中打开这个SpecRunner.html页面。

大致给出的内容就是：
	
	Passing 1 spec

	just do it

		should return 10 when 5 plus 5

在我的这个项目中，起初没有使用jasmine，由于逻辑复杂，手动在页面测试非常麻烦，且不易于代码重构。

后期引入jasmine之后，覆盖了所有的测试情况，这样可以在重构代码时，能够非常迅速的知道重构代码是否正确。

Jasmine还有很多更高级的用法，就目前项目中暂时还没有用到，如果用到，在以后的时间继续更新。

