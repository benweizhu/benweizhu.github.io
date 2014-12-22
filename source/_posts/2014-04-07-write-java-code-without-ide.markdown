---
layout: post
title: "丢掉IDE，回到Java的第一堂课"
date: 2014-04-07 13:56
comments: true
categories: 
---

最近，《实现领域驱动设计》的译者，ThoughtWorks的[“滕老板”](http://www.davenkin.me/)（腾云）给我们上了一堂基础课，就是没有IDE的情况下，去编译和运行用文本编辑器编写的Java代码。

这本应该是基础中的基础，我记得应该是Java第一堂课就应该讲到的东西。但是我们这群用惯了IDE的程序员，在没有了IDE之后，就开始不知所措了。虽然不知道这些知识，对于普通开发似乎是没有影响，但毕竟是Java的重要基础知识，了解这些知识可以帮助你理解IDE帮你做了什么，可以更快速的定位问题。

还是举例子，滕老板的例子：

目录结构

src

--client

----Client.java

--service

----Service.java

{% codeblock lang:java %}
package service;

public class Service {
	public String run(){
		String str = new String("as your service");
		return str;
	}
}
{% endcodeblock %}

{% codeblock lang:java %}
package client;

import service.*;

public class Client {
	public static void main(String args[]){
		Service service = new Service();
		System.out.println(service.run());
	}
}
{% endcodeblock %}
ok，代码写完了，你知道该怎么运行吗？

命令行：进入到src目录下
{% codeblock lang:powershell %}
javac client/Client.java
java client.Client
{% endcodeblock %}

这样就会输出正确结果：as your service。

再来看一下目录结构

src

--client

----Client.java

----Client.class

--service

----Service.java

----Service.class

我们知道javac命令编译的是.java文件，而java命令运行是.class文件。

但是这里还是有几个疑问：

1.javac client/Client.java时，它怎么知道去哪里找Service类？

2.对于上一个疑问，你的答案可能是import service.\*;告诉它去service包下找，那么第二疑问，为什么是import service.\*;而不是import src.service.*?

3.第三个疑问和第一个疑问类似，答案也类似，java client.Client，怎么知道去哪里找service.class文件？

面对这三个问题，答案是，要引入两个概念。

sourcepath和classpath。

classpath大家听得比较多，用来告诉JVM或者Java编译器到哪里去寻找用户定义的类和包。

sourcepath和classpath一样，只不过它是用来告诉编译器到哪里去寻找源文件。

在上面的javac和java命令中，既没有显示的指定sourcepath，也没有显示的指定classpath。但是它们都有默认值，就是将当前目录作为各自的路径。

比如：我现在从当前路径src，进入到它的一个子目录中src/sub。

然后我运行：javac ../client/Client.java命令

它提示我找不到Service类。 那是因为当前的sourcepath是src/sub，在这个路径下没有Service.java。我们改一下，显示的设置它sourcepath。

运行命令：javac -sourcepath ../ ../client/Client.java

编译成功。

那么运行呢？

直接在src/sub路径下，运行：java client.Client。能行吗？肯定不行。

Error: Could not find or load main class client.Client

必须要显示的指定它的classpath为上一级目录：

java -classpath ../ client.Client

ok，成功。

现在再来让我们回到刚才的第二个问题。为什么import的不是src.service.\* ，因为我们设置它的sourcepath，它就成为一个相对路径。

这不禁让我想到一个问题。package和import到底有什么用？为什么package名字和目录结构一致？

这不是白痴问题吗？

package是作为命名空间，因为不同开发商，开发出来的类库可能会有相同的类名。为了区分，防止冲突，用不同的包名区分。

import是根据你要使用的类，从classpath下找到你要使用的类，并引入到你的类中。

我做了一个实验：

将Service类的package定义，改为service.something，这样包名和目录结构就不一致了。

{% codeblock lang:java %}
package service.something;

public class Service {
	public String run(){
		return "as your service";
	}
}
{% endcodeblock %}

然后还是照常编译。

	client/Client.java:7: error: cannot access Service
		Service service = new Service();
		^
		bad source file: ./service/Service.java
		file does not contain class service.Service
	Please remove or make sure it appears in the correct subdirectory of the sourcepath.
	1 error
	
第四行，bad source file: ./service/Service.java表明他在编译Client.java文件时，通过import找到了该文件，说明import是通过文件结构去找到的。

但是他又说file does not contain class service.Service，说明在这个Service.java中找不到对应的Service类。

为什么呢？因为我的包名是service.something，但是他要找的是service。

所以包的作用是和类的名字组成完整的类名，这里的Service类的全名应该是service.something.Service。但是他去找到的是service.Service。所以它找不到。

那是不是将import改为import service.something.*;就可以呢？

不行，前面说了，import是根据目录结构去找到的，根本就没有service/something这个目录结构，所以不可能找到。

	client/Client.java:3: error: package service.something does not exist
	import service.something.*;
	^
	client/Client.java:7: error: cannot find symbol
		Service service = new Service();
		^
		symbol:   class Service
		location: class Client
		client/Client.java:7: error: cannot find symbol
		Service service = new Service();
		                      ^
		symbol:   class Service
		location: class Client
		
你看，第一行：client/Client.java:3: error: package service.something does not exist

回过头来想，这也是为什么，有时候，当同时导入两个不同包中名字相同的类时，其中有一个要用完全限定名。因为类的名字本来就是由包的名字和类的自身名字组成。

总而言之，这堂课的收获还是挺大的，重点要掌握上面提到的两个概念。

课上本来还有一点关于，打jar包和如何引用jar包的内容，以及通过java -d命令，将class文件编译到指定的目录下。

这里我就不多说了，万变不离其宗。

javac -classpath ".:service.jar" client/Client.java //linux下面是冒号，windows下面是分号

java -classpath ".:service.jar" client.Client

只不过要注意一点的是，打jar包时，一定要按照package定义的目录结构去打。因为在将jar包加入到classpath时和将一个目录下的class文件加入是一样的，相对路径。

java -d out client/Client.java，会将class文件编译到out目录下，且内部目录结构与包的结构一致。

