---
layout: post
title: "Gradle深入与实战（六）Gradle的背后是什么？"
date: 2015-03-31 23:20:24 +0800
comments: true
categories: [Gradle深入与实战]
---

##理解DSL（领域特定语言）

DSLs come in two main forms: external and internal. An external DSL is a language that's parsed independently of the host general purpose language: good examples include regular expressions and CSS. External DSLs have a strong tradition in the Unix community. Internal DSLs are a particular form of API in a host general purpose language, often referred to as a fluent interface. ---- Martin Fowler

Martin Fowler将DSL分为两类：外部DSL和内部DSL。外部DSL是一种独立的可解析的语言，举一个最常见的是例子，SQL，它专注于数据库的操作。内部DSL是通用语言所暴露的用来执行特定任务的API，它利用语言本身的特性，将API以特殊的形式（或者格式）暴露出来的。比如，Martin Fowler给出了关于流接口（fluent interface）。

总结一下，外部DSL是一种特定的独立语言，内部DSL是通用语言为实现特殊目的提供的API。

我们可以看下Martin Fowler提供的关于流接口的一个例子。流接口的实现方式同样是通用语言的写法，但相对于连续的调用set方法或者add方法，可读性方面要好很多。

{% codeblock lang:java %}
private void makeFluent(Customer customer) {
        customer.newOrder()
                .with(6, "TAL")
                .with(5, "HPK").skippable()
                .with(3, "LGV")
                .priorityRush();
}
{% endcodeblock %}

##Gradle的DSL

Gradle为了很好的描述构建，它提供了一套DSL，它是一种内部DSL。这套语言基于Groovy，但增加了一点点的特殊处理，以便在利用Groovy语言特性的之外，让它更像一个构建语言。

所以，Gradle的构建脚本和Groovy之间的关系是大于(>)。换句话说，如果你在Gradle的构建脚本中，写纯粹的Groovy代码是绝对没有问题的，这一点在官方文档的用户手册中也单独拿出一章来说明。

##构建脚本
我们先暂且不去验证这一点，也不要着急着用Groovy去写一个功能强大且复杂的task，从构建脚本中跳出来，站在一个更高的位置来俯视Gradle。

在我们写Gradle脚本的时候，我们一般会写两种类型的，一种脚本，默认命名是build.gradle，另一种脚本，命名是settings.gradle。它们各自的作用，相信不用我在这里过多的介绍。在Gradle的世界里面，它们都叫做配置脚本。举个很简单的例子，在settings中配置，多项目结构有哪些模块，或者在build.gradle中，配置Java的sourceset。

那可能就有人问了，那在构建脚本中写一个task也算配置吗？我并没有配置什么东西，而是在用语言实实在在的写一系列的动作。这个算配置吗？

我想，这取决于对“配置”这个概念的理解。如果，我只是应用Java插件，写一个sourceset，或者写一个dependencies，这个配置就有点类似，传统意义上理解的“键值对”的配置。

如果涉及到写一个task的时候，我们就要把它想象成“对某一个对象的配置”，想象成一系列的set或者add操作（即便是sourceset，你也应该这么去理解，因为这才是本质）。

##基于Groovy的本质
在前面介绍DSL概念的时候，我们了解到Gradle是一种内部DSL，是一系列的API，它基于Groovy。Groovy是什么？它是一种面向对象的编程语言，它的核心概念是对象。

Gradle基于Groovy但大于Groovy，它是经过“定制”的Groovy，是经过“定制”的面向对象语言，所以，由始至终，Gradle都离不开对象这个概念。

如果你明白了这个本质，那么就明白了写Gradle脚本，就和写Java代码是一个道理，写Gradle构建脚本就是写代码调用Gradle的API，只不过因为一些特性和特殊处理，让他看上去不太像一个标准的类C的编程语言（我更倾向于说类Java，因为C语言不是面向对象的），那么接下来，你所需要知道的就是Gradle提供的API长啥样。

##Gradle的对象

既然我们知道了Gradle的本质是经过“定制”的**面向对象**语言（Groovy语言），那么我们就来看看Gradle里面有些什么对象。

如果你有读过Gradle的用户手册，那么，第六章，Build Script Basics，肯定是你必读的一章节，即便你当时看不太明白，只是依葫芦画瓢。现在你可以回过头来看下，该章节在一开始就进入主题，介绍了Gradle中两个的核心概念，project和task。

从组成关系上来看，我们知道，或者文档是这么说的，一个构建是由多个project组成，一个project是由多个task组成，task之间的依赖关系，构成了整个构建的流水线。

对于task的概念相对比较好理解，因为在命令行中，我们通过gradle build，进行Java的构建，这是一个看得见，摸得着的概念。

那project是什么？从你学习Gradle开始，到应用Java插件，实现Java的构建，好像从头到尾都没有直接接触过project这个概念，至少没有像task这样如此真实的接触。我们需要了解它吗？如果你只是依葫芦画瓢，参考Gradle的文档，进行构建的“配置”，那么你不用。

如果你想知道写的那些配置在本质上是什么？那么就有必要。

##Project对象和build.gradle

为了不深究Gradle的实现原理（就是去读源代码），又要让大家觉得有据可依。我通过引用官方文档的一些描述来帮助大家理解Project对象。

For each project in the build, Gradle creates an object of type Project and associates this Project object with the build script. (Chapter 13. Writing Build Scripts)     
构建中的每一个project，Gradle都会创建一个Project对象，并将这个对象与构建脚本相关联。

There is a one-to-one relationship between a Project and a "build.gradle" file. (Interface Project API)     
Project对象与build.gradle是一对一的关系。

First, Gradle scripts are configuration scripts. As the script executes, it configures an object of a particular type. For example, as a build script executes, it configures an object of type Project. This object is called the delegate object of the script. (Gradle Build Language Reference)     
Gradle的脚本是配置脚本，当脚本执行时，它是在配置某一个特殊类型的对象。比如一个构建脚本的执行，它就是在配置一个Project类型的对象。这个对象叫做脚本的代理对象。

读完这三句话，应该可以清楚的明白build.gradle的本质，简单的说，build.gradle是对一个Project对象的配置。

##深入理解

如果你还没明白，你可以仔细考量这三句话。因为这里，我们要进一步深入探讨上个部分引出的另一个概念：代理。

这个概念并不来自于Gradle，如果你熟悉Groovy，你肯定会立刻想到这是Groovy中很重要的一个概念。

在Groovy中，Object对象提供了一个重要的方法with，这个方法在JavaScript中也是存在的。with方法可以在一个闭包内辅助实现委托调用，在with的作用域内调用的任何方法，都被定向到该上下文对象上，这样就去掉了对该实例的多余引用，举个例子：

{% codeblock lang:groovy %}
def list = [1, 2]
list.add(3)
list.add(4)
println(list.size())
println(list.contains(2))

def listWith = [1, 2]
listWith.with {
	add(3)
	add(4)
	println(size())
	println(contains(2))
}
{% endcodeblock %}

在with中，省去了对 list对象的引用。

而build.gradle和project对象，虽然从解析的角度不一定是通过with方式实现，但是它们之间就是这样的一个关系。

闭包内的内容就是build.gradle对project对象的操作。

这里我引用Gradle用户手册第十三章的内容来进一步说明，

Any method you call in your build script which is not defined in the build script, is delegated to the Project object.
Any property you access in your build script, which is not defined in the build script, is delegated to the Project object.（Chapter 13. Writing Build Scripts）

##通过现象看本质

我们从理论上讲了这么多关于Gradle本质的东西，而且好像还有点道理，但我们还是要验证一下，透过现象来看本质。通过实践，进一步加强我们的理解。

####一个小小的task

举个例子，我们在build.gradle中，写一个简单的task

{% codeblock lang:groovy %}
task helloWorld << {
    println 'helloWorld'
}
{% endcodeblock %}

如果你熟悉Groovy，并且知道它是针对project为上下文的一段代码，你会怎么看上面这段代码。是不是会有几个疑问？

问题一：是否有一个project的方法叫做task？答案：是，Project.task(String name)，返回一个Task对象。

问题二：helloWorld是一个参数吗？答案：是，它被解析为一个String类型的实参变量

问题三：符号“<<”是什么意思？答案：Groovy的强大特性，操作符重载。Task.leftShift(Closure action)，用来给task的action列表中添加一个action。

如果我用Groovy的写法，把它写成下面这样，是否就更好理解一些呢？

{% codeblock lang:groovy %}
task("helloWorld").leftShift({
    println 'hello world'
})
{% endcodeblock %}

##总结

当我们透过现象看到本质之后，你对Gradle的理解是不是不再是冷冰冰的闭包配置。是不是觉得Gradle其实没有那么神秘，不需要为Gradle中的奇怪的DSL感到困惑，它只是个API，读下API文档就好了。

最后总结，Gradle is Groovy but more than Groovy。


参考资料：

1.http://gradle.org/docs/current/userguide/writing_build_scripts.html    
2.http://gradle.org/docs/current/javadoc/     
3.http://docs.groovy-lang.org/latest/html/documentation/index.html#_delegation_strategy     
4.Groovy程序设计






























