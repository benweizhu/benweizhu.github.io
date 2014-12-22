---
layout: post
title: "什么是Spring IoC？"
date: 2013-09-22 15:33
comments: true
categories: 
---
在直接进入主题之前，先引入一些基本概念。  
>概念1：什么是容器？  

日常生活领域内的容器，用来包装或装载物品的贮存器(如箱、罐、罈)或者成形或柔软不成形的包覆材料。--维基百科。  
简而言之，容器是存放东西的东西。  

Java中，Java的类库有一系列的基本类来保存对象（正确的说是保存对象的引用），包括List，Set，Queue和Map，它们被称作集合类。  
Bruce Eckel（《Thinking in Java》的作者）使用了一个范围更广的术语来称呼它们：“容器”，容器提供了完善的方法来保存和管理对象。  

当然，这里只是为了简单介绍“容器”这个宽泛术语本身的含义。我们常说的容器并不是指Java的内部容器类。  

那么我们常说的容器是指的什么呢？举个例子，*Servlet容器*。  
什么是Servlet？这里又扯远了，引用孙鑫老师的说法（《Servlet/JSP深入详解：基于Tomcat的Web开发》的作者），Java Servlet（Java服务器小程序）是一个基于Java技术的Web组件，运行在服务器端。Servlet是平台独立的Java类，编写一个Servlet，实际上就是按照Servlet规范编写一个Java类。  

Servlet容器又称为Servlet引擎，它是Java Web服务器（例如：Apache Tomcat）的一部分，用来提供动态网页的服务。  
我们知道，Servlet没有main方法，它是不能独立运行的。它必须被部署到Servlet容器中，由容器来实例化和调用Servlet中的方法。因此，Servlet容器的作用是在Servlet的生命周期内包容和管理Servlet。

用户通过单击某个链接或者直接在浏览器的地址栏中输入URL来访问Servlet，Java Web服务器接收到该请求后，并不是将请求直接交给Servlet，而是交给Servlet容器。Servlet容器实例化Servlet，调用Servlet的一个特定方法对请求进行处理，并产生一个响应。这个响应由Servlet容器返回给Web服务器，Web服务器包装这个响应，以HTTP响应的形式发送给Web浏览器。  

Ok，容器的概念就讲到这里，下一个依赖注入。  
>概念2：依赖注入  

谈到依赖注入，必须提Martin Fowler的经典文章[IoC容器和Dependency Injection模式](http://martinfowler.com/articles/injection.html)。这篇文章有对应的中文版翻译（[熊节](http://gigix.thoughtworkers.org/)译），原文上有链接，但是由于时间太久了，链接已经不可用了。我把它转载到我的博客上做了记录，可以在这里访问。我相信如果有看完这篇文章的，应该不用听我在这里啰嗦了。
依赖注入的直接好处就是解耦，消除组件之间的直接依赖。这种优势能够非常好的体现在对组件的测试。

>进入正题

在Martin的那篇文章中已经提到，依赖注入有三种方式，分别是接口注入，构造器注入和设值注入。其中比较常用的是构造器注入和设值方法注入。在Spring中，配置依赖的方式有两种，一种是比较常用的xml方式配置，另一种是通过注解的方式配置（Spring2.5之后）。  

>首先介绍基于xml的配置方式：  

现在来看一下，如何通过xml来配置一个bean
{% codeblock bean-config.xml lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:annotation-config/>
    <context:component-scan base-package="foo.bar"/>
    <beans>
        <bean id="dataBaseService" class="foo.bar.DatabaseService">
            <constructor-arg index="0" value="username"/>
            <constructor-arg index="1" value="password"/>
            <constructor-arg index="2" value="database_url"/>
        </bean>
        <bean id="justService" class="foo.bar.JustService">
            <constructor-arg value="word" type="java.lang.String"/>
            <constructor-arg value="10" type="int"/>
        </bean>
        <bean id="justReferenceService" class="foo.bar.JustReferenceService">
            <property name="justService" ref="justService"/>
        </bean>
        <bean id="calculatorService" class="foo.bar.CalculatorService">
            <property name="a" value="1"/>
            <property name="b" value="1"/>
        </bean>
        <bean id="useCalculatorService" class="foo.bar.UseCalculatorService" autowire="byName"/>
    </beans>
</beans>
{% endcodeblock %}
每一个Bean都有对应的id用来唯一表示它，可以理解为Spring容器中，该Bean的名字。  

在这个Bean的配置文件中，介绍了几种配置Bean的方法。 

dataBaseService采用构造器方式注入，由于构造器参数都是String类型，所以这里采用index来指定对应的参数。  
justService同样采用构造器方式注入，但构造器参数类型不同，所以这里可以根据类型指定参数。  
justReferenceService采用的是设值方式注入，并且注入的参数为一个引用类型，name为类中对应的变量名，ref是xml中的某一个Bean。  
calculatorService同样采用设值方式注入。  
useCalculatorService同样采用设值方法注入，并声明autowire自动装配是byName，它会根据类中成员变量的名字，在这些Bean中找到对应id等于成员变量名字的Bean。

在xml中配置好这些Bean之后，就可以通过ApplicationContext去得到这些Bean。

{% codeblock helloworld.java lang:java %}
package foo.bar;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class HelloWorld {
	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext("spring-config.xml");

		DatabaseService databaseService = context.getBean(DatabaseService.class);
		databaseService.connect();

		JustService justService = context.getBean(JustService.class);
		justService.justPrint();

		JustReferenceService justReferenceService = context.getBean(JustReferenceService.class);
		justReferenceService.justPrint();

		UseCalculatorService useCalculatorService = context.getBean(UseCalculatorService.class);
		System.out.println(useCalculatorService.calculate());
	}
}
{% endcodeblock %}  

>另一种配置Bean的方式是直接在类文件中加入注解  

{% codeblock bean-config.xml lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:annotation-config />
    <context:component-scan base-package="foo.bar"/>
</beans>
{% endcodeblock %}
上面是一个没有任何Bean的xml配置文件，里面有两个标签，分别是<context:annotation-config />和<context:component-scan />。现在来解释一下这两个标签的作用：    

<context:annotation-config />:隐式地向Spring容器注册
AutowiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor、
PersistenceAnnotationBeanPostProcessor 以及 RequiredAnnotationBeanPostProcessor 这 4 个BeanPostProcessor。这样我们就可以使用@Required、@Autowired以及JSR 250的@PostConstruct、@PreDestroy和@Resource (if available）等等这些注解。如果不使用这个标签，那么如果要使用这些注解，就必须在xml文件中配置这些processor的Bean。  

<context:component-scan base-package="foo.bar"/>:它会扫描classpath下被注解过的类，并注册为Spring容器中的bean。Spring默认提供的注解有@Component、@Repository、@Service和@Controller。

有了这两个标签，我们就可以简单的在类中添加注解实现Bean向Spring容器的注册，如下，是对JustReferenceService的配置：
{% codeblock JustReferenceService.java lang:java %}
package foo.bar;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class JustReferenceService {

	@Autowired
	private JustService justService;

	public JustService getJustService() {
		return justService;
	}

	public void setJustService(JustService justService) {
		this.justService = justService;
	}

	public void justPrint() {
		justService.justPrint();
	}
}
{% endcodeblock %}
使用@Autowired注解进行装配，只能是根据类型进行匹配。@Autowired注解可以用于Setter方法、构造函数、字段，甚至普通方法，前提是方法必须有至少一个参数。  

当容器中存在多个Bean的类型与需要注入的相同时，注入将不能执行，我们可以给@Autowired增加一个候选值，做法是在@Autowired后面增加一个@Qualifier标注，提供一个String类型的值作为候选的Bean的名字，见下例。

在xml配置中，有两个同类型的Bean，它们拥有不同的id，因此，需要在类中添加注解告诉它需要装配哪一个。

{% codeblock UseCalculatorService.java lang:java %}
package foo.bar;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component
public class UseCalculatorService {

	@Autowired
	@Qualifier("anotherCalculatorService")
	private CalculatorService calculatorService;

	public CalculatorService getCalculatorService() {
		return calculatorService;
	}

	public void setCalculatorService(CalculatorService calculatorService) {
		this.calculatorService = calculatorService;
	}

	public int calculate(){
		return calculatorService.calculate();
	}
}
{% endcodeblock %}
*如果希望根据name执行自动装配*

那么应该使用JSR-250提供的@Resource注解，而不应该使用@Autowired与@Qualifier 的组合。@Resource使用byName的方式执行自动封装。@Resource标注可以作用于带一个参数的Setter方法、字段，以及带一个参数的普通方法上。@Resource注解有一个name属性，用于指定Bean在配置文件中对应的名字。如果没有指定name属性，那么默认值就是字段或者属性的名字。@Resource和@Qualifier的配合虽然仍然成立，但是@Qualifier对于@Resource而言，几乎与name属性等效。如果@Resource没有指定name属性，那么使用byName匹配失败后，会退而使用byType继续匹配，如果再失败，则抛出异常。
{% codeblock UseCalculatorService.java 使用@Resource lang:java %}
package foo.bar;

import org.springframework.stereotype.Component;

import javax.annotation.Resource;

@Component
public class UseCalculatorService {

	@Resource(name = "anotherCalculatorService")
	private CalculatorService calculatorService;

	public CalculatorService getCalculatorService() {
		return calculatorService;
	}

	public void setCalculatorService(CalculatorService calculatorService) {
		this.calculatorService = calculatorService;
	}

	public int calculate() {
		return calculatorService.calculate();
	}
}
{% endcodeblock %}
*使用 @Required 进行 Bean 的依赖检查*  

依赖检查的作用是，判断给定Bean的相应Setter方法是否都在Bean被实例化的时候被调用了，而不是判断字段是否已经存在值了。Spring进行依赖检查时，只会判断属性是否使用了Setter注入。如果某个属性没有使用Setter注入，即使是通过构造函数已经为该属性注入了值，Spring仍然认为它没有执行注入，从而抛出异常。另外，Spring只管是否通过Setter执行了注入，而对注入的值却没有任何要求，即使注入的null，Spring也认为是执行了依赖注入。  

@Required 注解只能标注在Setter方法之上。因为依赖注入的本质是检查Setter方法是否被调用了，而不是真的去检查属性是否赋值了以及赋了什么样的值。如果将该注解标注在非setXxxx()类型的方法则被忽略。

*使用 @Configuration 和 @Bean 进行 Bean 的声明*

Spring3.0新增了另外两个实现类：AnnotationConfigApplicationContext和AnnotationConfigWebApplicationContext，它们是为注解而生，直接依赖于注解，作为容器配置信息来源的IoC容器初始化类。AnnotationConfigApplicationContext搭配上@Configuration和@Bean注解，自此，XML配置方式不再是 Spring IoC容器的唯一配置方式。Spring对标注Configuration的类有如下要求：

(1)配置类不能是 final 的；(2)配置类不能是本地化的，亦即不能将配置类定义在其他类的方法内部；(3)配置类必须有一个无参构造函数。
{% codeblock BeanConfiguration.java lang:java %}
package foo.bar;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class BeanConfiguration {

	@Bean
	public DatabaseService dataBaseService() {
		return new DatabaseService("username", "password", "database_url");
	}

	@Bean
	public JustService justService() {
		return new JustService("justService", 10);
	}
}
{% endcodeblock %}

上面这种配置bean的方式与下面的xml配置方式一样。

{% codeblock spring-config.xml lang:xml %}
<bean id="dataBaseService" class="foo.bar.DatabaseService">
       <constructor-arg index="0" value="username"/>
       <constructor-arg index="1" value="password"/>
       <constructor-arg index="2" value="database_url"/>
</bean>
<bean id="justService" class="foo.bar.JustService">
       <constructor-arg value="word" type="java.lang.String"/>
       <constructor-arg value="10" type="int"/>
</bean>
{% endcodeblock %}
Spring IoC是将Bean的管理交给Spring容器，开发人员只需要配置，当要使用时，告诉Spring，让它注入给你。这样，在写程序时，可以大大降低耦合。


参考资料：

http://www.ibm.com/developerworks/cn/opensource/os-cn-spring-iocannt/

http://www.springbyexample.org

孙鑫《Servlet/JSP深入详解：基于Tomcat的Web开发》