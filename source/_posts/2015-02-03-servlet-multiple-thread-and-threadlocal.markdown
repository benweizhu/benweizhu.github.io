---
layout: post
title: "Servlet多线程安全问题和LocalThread"
date: 2015-02-03 21:13:53 +0800
comments: true
categories: 
---

##Servlet线程不安全

####以下内容摘录自Java™ Servlet Specification，在开始阅读本文章之前，请仔细阅读：

2.1 Request Handling Methods    
...    
The handling of concurrent requests to a Web application generally requires that the
Web Developer design servlets that can deal with multiple threads executing within
the service method at a particular time.    
Generally the Web container handles concurrent requests to the same servlet by
concurrent execution of the service method on different threads.

2.2 Number of Instances     
...    
For a servlet not hosted in a distributed environment (the default), the servlet
container must use only one instance per servlet declaration. However, for a servlet
implementing the SingleThreadModel interface, the servlet container may
instantiate multiple instances to handle a heavy request load and serialize requests
to a particular instance.
...

2.3.3.1 Multithreading Issues     
...  
Although it is not recommended, an alternative for the Developer is to implement
the SingleThreadModel interface which requires the container to guarantee that
there is only one request thread at a time in the service method. A servlet container
may satisfy this requirement by serializing requests on a servlet, or by maintaining a
pool of servlet instances. If the servlet is part of a Web application that has been
marked as distributable, the container may maintain a pool of servlet instances in
each JVM that the application is distributed across.

For servlets not implementing the SingleThreadModel interface, if the service
method (or methods such as doGet or doPost which are dispatched to the service
method of the HttpServlet abstract class) has been defined with the synchronized
keyword, the servlet container cannot use the instance pool approach, but must
serialize requests through it. It is strongly recommended that Developers not
synchronize the service method (or methods dispatched to it)

默认情况下，非分布式系统，Servlet容器只会维护一个Servlet的实例，当多个请求到达同一个Servlet，Servlet容器会启动多个线程分配给不同请求来执行同一个Servlet实例中的服务方法。为什么这么做？有效利用JVM允许多个线程访问同一个实例的特性，来提高服务器性能。因为，无论是同步线程对Servlet的调用，还是为每一个线程初始化一个Servlet实例，都会带来巨大的性能问题。

这也就是为什么Servlet会存在多线程安全问题。

大部分线程安全问题出现的原因都是Servlet实现者在不经意间创建了一个Servlet的实例变量（成员变量），而导致多个线程公有这个实例变量，存在不同阶段对该变量的读写操作。

###预防它很简单：就是避免这样写，用方法中的本地变量替代它。

“Java虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧用于存储局部变量表、操作栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。”     
---深入理解Java虚拟机

##ThreadLocal

那如果你希望定义一个变量，让每一个线程都拥有不同的拷贝，应该怎么办？答案是ThreadLocal。

ThreadLocal是Java语言包提供的一个实现类，与其命名ThreadLocal，叫它thread-local variables更合适。和普通变量不同，通过该对象的set和get方法，可以给每一个调用它的线程保存一个独立的变量的拷贝。什么意思？也就是说，该变量保存下来的变量和当时调用该方法的线程是绑定的，不同线程的值是不一样的。

在Java Doc中介绍过：定义该变量的典型方法是private static
{% codeblock lang:java %}
private static final ThreadLocal<Integer> threadId = new ThreadLocal<Integer>();
{% endcodeblock %}

当要保存时，调用它的set方法，获取时，调用get方法。
{% codeblock lang:java %}
threadId.set(10);
threadId.get();
{% endcodeblock %}

下面看一个例子，单例类中定义了两个变量，实例变量和ThreadLocal变量，多线程读写，并随机等待一段时间，得到的结果会是普通实例变量和time的值不一致，而threadLocal是一致的：
{% codeblock lang:java %}
package me.zeph.relations;

import java.util.Random;

public class Singleton {

	private int value;

	private static Singleton instance;

	private static ThreadLocal<Integer> threadLocal = new ThreadLocal<>();

	private Singleton() {
	}

	public void set(int value) {
		threadLocal.set(value);
	}

	public int get() {
		return threadLocal.get();
	}

	public static synchronized Singleton getInstance() {
		if (instance == null) {
			instance = new Singleton();
		}
		return instance;
	}

	public int getValue() {
		return value;
	}

	public void setValue(int value) {
		this.value = value;
	}

	public static void main(String args[]) {
		final Singleton singleton = Singleton.getInstance();
		for (int i = 0; i < 10; i++) {
			Thread thread = new Thread() {
				@Override
				public void run() {
					int time = new Random(System.currentTimeMillis()).nextInt(2000);
					singleton.set(time);
					singleton.setValue(time);
					try {
						Thread.sleep(time);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println(time + "---ThreadLocal---" + singleton.get());
					System.out.println(time + "---NonThreadLocal-----" + singleton.getValue());
				}
			};
			thread.start();
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
{% endcodeblock %}


###了解到它的作用和如何使用之后，你肯定在想，它是怎么实现的？

一种办法是：在这个对象的里面存放一个map对象，map对象的key就是Thread.currentThread()的一些信息，value就是对应的值。这是最显而易见而直接的实现方式。

**但是，它不是这么实现的！！！是反过来的！！！**

在当前的线程对象里面存放一个Map，Map的key是当前的ThreadLocal对象，value是对应的值。

那么当程序中有多个ThreadLocal是就不是每个threadLocal对象维护一个线程的map，而是每个线程有一个map来维护所有的ThreadLocal。

###这么做有什么好处？

我的猜测是，资源释放问题，如果是第一种方式，线程已经完成了它的任务，但是ThreadLocal仍然保存它的引用，那么线程资源就不会立刻释放（根据不同的垃圾回收策略，可能不同）。

以上只是Servlet线程安全问题中一种常见情况，Servlet线程安全问题还有很多，比如Session的访问，但重点是，需要大家意识到Servlet是线程不安全的，于是在编写代码的时候一定要多思考，这样写是否存在线程安全问题。

参考资料：    
1.Servlet Specification    
2.深入理解Java虚拟机














