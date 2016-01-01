---
layout: post
title: "JavaScript this指针"
date: 2016-01-01 19:04:16 +0800
comments: true
categories:
---
在JavaScript中，随着函数使用场合的不同，this的值会发生变化。但是有一个总的原则，那就是this指的是，调用该函数的那个对象。this关键字在Javascript中和执行环境，而非声明环境有关。

举个例子：
{% codeblock lang:javascript %}
var someone = {
    name: "Bob",
    showName: function(){
        alert(this.name);
    }
};

var other = {
    name: "Tom",
    showName: someone.showName
}

other.showName();　　//Tom
{% endcodeblock %}
这里其实应该好理解，other的showName属性指向了someone的showName所声明的函数，他等价于：
{% codeblock lang:javascript %}
var other = {
    name: "Tom",
    showName: function(){
        alert(this.name);
    }
}

other.showName();　　//Tom
{% endcodeblock %}
所以this指针理所当然的指向other自己。

{% codeblock lang:javascript %}
var name = "Tom";

var Bob = {
    name: "Bob",
    show: function(){
        alert(this.name);
    }
}

var show = Bob.show;
show();　　//Tom
{% endcodeblock %}

上面这个show为什么显示Tom，它的执行对象是什么？Window对象。

Window对象是所有客户端JavaScript特性和API的主要接入点。它表示Web浏览器的一个窗口，并且可以用标示符window（小写）来引用。

{% codeblock lang:javascript %}
window
$ Window {external: Object, chrome: Object, document: document, \_ASYNC_START: 1451649155228, \_chrome_37_fix: undefined…}
Window
$ function Window() { [native code] }
{% endcodeblock %}

Window对象定义了一些属性，比如，指代Location对象(location)的location属性。
{% codeblock lang:javascript %}
window.location === location
$ true
{% endcodeblock %}
Window对象还定义了一些方法，比如alert()和setTimeout()，使用时，我们都没有显示的使用window属性。Window对象是全局对象，处于作用域链的顶部，它的属性和方法实际上全是全局变量和全局函数。
通过window变量可以引用到Window对象本身，但是如果要使用全局窗口对象的属性，并不需要使用window。

我们常说，JavaScript很容易就创建一个全局变量或者全局函数。比如，直接var a_global_variable = 'a global variable'。它不仅使全局变量，它还是window的一个属性。如下：

{% codeblock lang:javascript %}
var a_global_variable = 'a global variable'
a_global_variable
$ "a global variable"
window.goodtest
$ "a global variable"
window.a_global_variable === a_global_variable
$ true
{% endcodeblock %}

理解了setTimeout是Window的属性之后，理解下面这段代码应该比较容易了：
{% codeblock lang:javascript %}
var name = "Bob";  
var nameObj ={  
    name : "Tom",  
    showName : function(){  
        alert(this.name);//此时this指向的是window  
    },  
    waitShowName : function(){  
        setTimeout(this.showName, 1000);//这里的this指向的是nameObj.showName
    }  
};
nameObj.waitShowName();// Bob
{% endcodeblock %}
和下面这段代码相似
{% codeblock lang:javascript %}
var name = "Bob";  
setTimeout(function(){  
    alert(this.name);
}, 1000);
{% endcodeblock %}

##new关键字

new关键字指向创建的对象，在上一篇文章中已经介绍的很清楚了。 http://benweizhu.github.io/blog/2015/12/31/javascript-contructor-new-prototype/

{% codeblock lang:javascript %}
function Person(name){
    this.name = name; //这个this指向用该构造函数构造的新对象，这个例子是Bob对象
}
Person.prototype.show = function(){
    alert(this.name);
}
var Bob = new Person("Bob");
Bob.show();        //Bob
{% endcodeblock %}

##apply和call改变this指向的对象

apply和call能够强制改变函数执行时的当前对象，让this指向其他对象。
{% codeblock lang:javascript %}
var name = "window";

var someone = {
    name: "Bob",
    showName: function(){
        alert(this.name);
    }
};

var other = {
    name: "Tom"
};    

someone.showName();   //Bob
someone.showName.apply();    //window
someone.showName.apply(other);    //Tom
{% endcodeblock %}

参考资料：    
1.http://www.cnblogs.com/justany/archive/2012/11/01/the_keyword_this_in_javascript.html
