---
layout: post
title: "JS 改变默认行为 e.preventDefault() e.returnValue e.stopPropagation e.cancelBubble return false"
date: 2016-05-02 09:31:27 +0800
comments: true
categories: 
---

事件对象有一些方法可以改变一个元素的默认行为，以及它的祖先元素如何对这个事件作出响应。

有一些事件，比如点击链接或提交表单，会把用户导向另一个页面。为了阻止这类元素的这种默认行为，比如在用户点击链接或提交表单之后还是停留在当前页面，而不是导向新的页面，可以使用事件对象e的preventDefault()。

IE5~IE8有一个同样功能的属性returnValue，如果将其设置为false，就能达到同样的效果。

{% codeblock lang:javascript %}
if (e.preventDefault) {
	e.preventDefault();
} else {
	e.returnValue = false;
}
{% endcodeblock %}

处理完某个元素上的事件之后，可能需要阻止这个事件向其祖先元素继续冒泡传播。这时候，可以使用e.stopPropagation()方法。

IE8或更早的IE版本中，使用拥有同样功能的属性e.cancelBubble，将其设置为true可以达到同样的效果。

{% codeblock lang:javascript %}
if (e.stopPropagation) {
	e.stopPropagation();
} else {
	e.cancelBubble = true;
}
{% endcodeblock %}

如果你想要同时实现，preventDefault和stopPropagation，可以使用return false。

{% codeblock lang:javascript %}
$("a").click(function() {
   $("body").append($(this).attr("href"));
   return false;
}
{% endcodeblock %}

参考资料：   
1.《JavaScript&jQuery交互式Web前端开发》


