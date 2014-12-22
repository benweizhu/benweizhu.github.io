---
layout: post
title: "Spring AOP by Example"
date: 2014-02-23 16:08
comments: true
categories: 
---
OOP编程中以对象为核心，系统软件是由相互依赖、相互协作的对象组成。对象在程序中被抽象为类，它们拥有一组属性和行为。类与类之间可以通过继承和组合实现分层和协作，它可以将具有相同属性或者相同功能的代码抽象到一个分明的层次，从而有效的管理代码，降低复杂度。

但随着业务复杂度的增加，客户不断提出新的需求。有些特定的需求虽然可以采用OOP解决，但却不是最好的方式，说的直观一点，并不是最节省代码编写工作量的。

例如：假设你有一个图形类Graphic，它里面有很多的set方法。新的需求是在每一次set之后，就直接将结果绘画出来（假设你有一个方法Canvas.paint()）。普通的做法是在set方法的末尾，添加一行调用paint()方法的代码。如果该Graphic类中只有两三个set方法，那到没什么问题，但是如果它有100个呢？（有点极限）而且如果你添加了一个新的set方法，还要记住不能忘记添加paint()方法调用。

AOP（aspect-oriented programming，面向方面/切面编程）能够很好的解决这个问题，你只需要告诉它，在每一个set方法之后，调用一次paint()方法。

AOP能够很好地来处理一些具有横切性质的系统级服务，如事务管理、安全检查、日志服务等。

**AOP中的一些概念**：

切面（Aspect）：一个关注点的模块化，该关注点会横切多个对象。在Spring AOP中有两种方式定义一个Aspect，通过在xml中配置，或者在Java类中使用@AspectJ的注解。

连接点（Joint point）：在程序执行过程中某个特定的点，比如某个方法调用的时候或者异常处理的时候。在Spring AOP中，一个连接点通常表示一个方法的执行。

通知（Advice）：在切面（Aspect）的某个特定的连接点（Joint point）上执行的动作。通知的类型有很多种，例如：“before”和“after”等等。

切入点（Point cut）：匹配连接点（Joint point）的断言（匹配连接点的公式）。通知（Advice）会和一个切入点（Point cut）表达式关联，并在满足这个切入点（Point cut）的连接点（Joint point）上运行。切入点如何和连接点匹配是AOP的核心。

在Spring AOP中的术语实在是太多了，往往其实不太让人好理解，也成为我们学习的绊脚石。

这里尝试用简单的话总结：

**通知就是要执行的动作，连接点就是程序中某个要插入动作（通知）的特定点，可能在该特定点之前插入（before），可能在它之后插入（after），或者在它抛出异常后插入（after throwing），切入点定义了如何找到该连接点的匹配模式，它们组合在一起，定义一个完整的切面。**

这里主要关注的如何使用Spring AOP，关于实现的方式，这里简单的提一下：

AOP实现的方式有三种：**静态代理，JDK动态代理，CGLib动态里**。Spring AOP采用的是动态代理方式实现AOP。

如果深入了解可以参考：http://developer.51cto.com/art/201309/410861_all.htm

通过Spring编写AOP代码方式有两种：**1.通过@AspectJ的注解实现。2.通过在XML中配置**

这里看一个通过XML配置方式实现AOP的例子：

{% codeblock build.gradle lang:groovy %}
apply plugin: 'java'
apply plugin: 'idea'

repositories {
 mavenCentral()
}

dependencies {
 compile 'org.springframework:spring-context:3.2.8.RELEASE'
 compile 'org.aspectj:aspectjweaver:1.7.4'
 testCompile group: 'junit', name: 'junit', version: '4.+'
}
{% endcodeblock %}
这里需要添加aspectjweaver的依赖，否则会报class not found的异常。

{% codeblock HelloTransaction.java lang:java %}
package me.zeph.aop.beans;

import org.springframework.stereotype.Component;

@Component
public class HelloTransaction {
	
   private boolean flag;
	
   public void setFlag(boolean flag) {
		this.flag = flag;
   }

   public void startTransaction() throws Exception {
		System.out.println(process());
   }

   private String process() throws Exception {
		if (flag) {
			return "start transaction";
		} else {
			throw new Exception("transaction exception");
		}
   }

}
{% endcodeblock %}
这里定义了“连接点类”，要被插入动作或者要被增强的类。

{% codeblock TransactionLogger.java lang:java %}
package me.zeph.aop.aspect;

public class TransactionLogger {

   public void beforeTransaction(){
		System.out.println("before transaction");
   }

   public void afterTransaction(){
		System.out.println("after transaction");
   }

   public void transactionThrowException(){
		System.out.println("transaction throw exception");
   }

}
{% endcodeblock %}
“通知类”，定义了要在连接点执行的动作。

{% codeblock bean.xml lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">

   	<bean id="helloTransactionSuccess" class="me.zeph.aop.beans.HelloTransaction">
        <property name="flag" value="true"/>
    </bean>

   	<bean id="transactionLogger" class="me.zeph.aop.aspect.TransactionLogger">
    </bean>

   	<aop:config>
        <aop:aspect id="transactionAspect" ref="transactionLogger">
            <aop:pointcut id="startTransaction" expression="execution(public * me.zeph.aop.beans.*.startTransaction(..))"/>
            <aop:before pointcut-ref="startTransaction" method="beforeTransaction"/>
            <aop:after-throwing pointcut-ref="startTransaction" method="transactionThrowException"/>
            <aop:after pointcut-ref="startTransaction" method="afterTransaction"/>
        </aop:aspect>
    </aop:config>

</beans>
{% endcodeblock %}
这里重点看定义切面的xml配置文件，首先定义了对应的两个Bean，然后在<aop:config>标签定义中定义切面。在<aop:config>标签下有三种标签：

<aop:pointcut>：用来定义切入点（用来找到连接点的匹配的模式），该切入点可以重用； 

<aop:advisor>：用来定义只有一个通知和一个切入点的切面； 

<aop:aspect>：用来定义切面，该切面可以包含多个切入点和通知，而且标签内部的通知和切入点定义是无序的；它和advisor的区别就在此，advisor只包含一个通知和一个切入点。

在<aop:aspect>标签下有<aop:pointcut>切入点标签，和一堆通知标签，例如before和after。

<aop:aspect>中的ref用于指定“通知类”，指定要添加的行为来自于哪个类。

<aop:pointcut>中的expression用于定义寻找连接点的匹配模式。

<aop:before>等通知标签中的pointcut-ref用于指定要在哪个连接点插入（增强）通知（行为）。

通过AOP，我并不需要改变HelloTransaction.startTransaction()中的业务逻辑代码，就可以给它增加例如，日志，事务管理等功能。

这就是AOP强大的位置。
