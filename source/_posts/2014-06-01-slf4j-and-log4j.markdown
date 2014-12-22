---
layout: post
title: "logger（SLF4j和log4j）"
date: 2014-06-01 12:33
comments: true
categories: 
---

##**SLF4j**

Simple Logging Facade for Java (SLF4J)

SLF4J，简单日志门面（Facade），是各个不同日志框架的抽象，它不是具体的日志解决方案，只服务于各种各样的日志系统，例如java.util.logging, log4j等，它允许最终用户在部署其应用时使用其所希望的日志系统。

**什么意思？**SLF4J就像一个接口，是一个门面，在写代码时，我们只需要根据这个接口提供的方法去记录日志。这个接口可以和不同的日志框架绑定，比如Java自带的java.util.logging，或者大家常听到的log4j。这样，你可以在不修改代码的情况下，去替换不同的日志框架。

基本概念明白之后，看下面这个例子：

{% codeblock lang:java %}
import org.slf4j.logger;
import org.slf4j.loggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    logger logger = loggerFactory.getlogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
{% endcodeblock %}

{% codeblock lang:groovy %}
apply plugin: 'idea'
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.slf4j:slf4j-api:1.7.7'
}
{% endcodeblock %}

输出的结果是：

SLF4J: Failed to load class "org.slf4j.impl.StaticloggerBinder".  
SLF4J: Defaulting to no-operation (NOP) logger implementation  
SLF4J: See http://www.slf4j.org/codes.html#StaticloggerBinder for further details.  

原因是你没有将SLF4j和某一个具体的日志框架绑定。怎么绑定呢？不需要你做任何事情，简单的加一个依赖就可以了。

{% codeblock lang:groovy %}
apply plugin: 'idea'
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.slf4j:slf4j-api:1.7.7'
    compile 'org.slf4j:slf4j-simple:1.7.7'
}
{% endcodeblock %}

输出是：[main] INFO log.HelloWorld - Hello World

**SLF4J支持对很多不同框架的绑定：**

slf4j-log4j12-1.7.7.jar  
slf4j-jdk14-1.7.7.jar  
slf4j-nop-1.7.7.jar  
slf4j-simple-1.7.7.jar  
slf4j-jcl-1.7.7.jar

**那么有个问题，SLF4J支不支持对多个日志框架的绑定呢？答案是不支持，也感觉一般这是不必要需求。**

{% codeblock lang:groovy %}
apply plugin: 'idea'
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.slf4j:slf4j-api:1.7.7'
    compile 'org.slf4j:slf4j-simple:1.7.7'
    compile 'org.slf4j:slf4j-log4j12:1.7.7'
}
{% endcodeblock %}

输出：

SLF4J: Class path contains multiple SLF4J bindings.  
... 

**在SLF4J的官方文档上也说明了这点：**

“SLF4J does not rely on any special class loader machinery. In fact, each SLF4J binding is hardwired at compile time to use one and only one specific logging framework. For example, the slf4j-log4j12-1.7.7.jar binding is bound at compile time to use log4j. In your code, in addition to slf4j-api-1.7.7.jar, you simply drop one and only one binding of your choice onto the appropriate class path location. Do not place more than one binding on your class path.”

OK，关于SLF4J的内容就到这里，该进入重头戏--log4J。

##**log4j**

log4j相对于SLF4j就复杂许多了，毕竟SLF4j只是接口，log4j则是具体的实现了。

log4j由三个重要的组件组成：

1. 类型和级别(logger)  
2. 输出目的地(Appender)
3. 输出格式(Layout)

###**logger**

**logger hierarchy（logger的层次级别）**

**Named Hierarchy（以名字分层）**

logger的名字是大小写敏感的，并且根据名字的不同，logger之间存在父子层次关系。举例来说，com.foo是com.foo.bar的父logger，com是com.foo.bar的祖先logger。

在这些logger之间，存在一个root logger，位于这个层次的顶层，它是永久存在，并且不能够通过名字获取，要获取它，只能通过logger.getRootlogger()方法获取。而这样一个层次关系，也决定了logger和logger之间存在着一些继承关系。

**Level Inheritance（级别继承）**

子类继承父类的级别：如果子类级别没有显示的指定，子类的级别等于第一个级别非空的父类logger的级别，直到追溯到根logger。

前面提到logger的级别，它们分别是TRACE, DEBUG, INFO, WARN, ERROR和FATAL。

这些级别存在高低关系：DEBUG < INFO < WARN < ERROR < FATAL。

如果你以前有用过一点点log4j，这些级别映射到你的记忆中，一定是反映成logger的各个方法。没错，在logger这个对象中，你会看到很多的打印方法，debug, info, warn, error, fatal和log。

**回过头来想想，如果你给一个logger指定了级别，但是你打印日志时，又不是调用对应级别的方法，会发生什么事情呢？**

答案是：对这些方法的调用，不光是打印日志信息，也是对logger对象级别改变的请求。

但是，并不是你调用它，级别就一定会改变，对级别改变的请求还是取决于logger对象本身的级别（无论是被显示的指定，还是继承）。如果请求的级别低于该logger当前的级别，那么改变就不会成功。

###**Appender**

Appender的作用是指出日志信息的输出位置，比如控制台，文件。而且好处是，你可以给一个logger指定多个appender，比如同时在控制台和文件中输出日志。

####**Appender Additivity(附加性)**

每一个Appender都有一个属性，叫做additivity flag，默认是true。什么意思呢？

子类会继承它直接父类的appender（包括该父类自己继承的appender），如果设置为false，则不继承其父类的appender。

**这句话应该好理解，那下面这种情况呢？**

Logger | Appender | Additivity
:----------- | :-----------: | :-----------:
root         | A1        | null
appenderFather| A2        | false
appenderFather.appenderChild| A3|true

appenderFather的flag设置为了false，所以他拥有的appender的结果应该比较明显，就是A2。

appenderFather.appenderChild的结果应该是什么？答案是A2和A3。因为它只是继承它直接父类的appender。

**log4j提供的Appender有哪些呢？**

org.apache.log4j.ConsoleAppender（控制台）

org.apache.log4j.FileAppender（文件）

org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件）

org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生新文件，用的较多，限制文件大小）

org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）

###**Layout**

日志信息布局（格式）是log4j提供的另外一个非常有用的特性。

**log4j提供四种Layout方式：**

org.apache.log4j.HTMLLayout（以HTML表格形式布局），

org.apache.log4j.PatternLayout（可以灵活地指定布局模式，比较常用），

org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串），

org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）

**log4j采用类似C语言中的printf函数的打印格式格式化日志信息，打印参数如下：**

%m 输出代码中指定的消息 　

%p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL 　　

%r 输出自应用启动到输出该log信息耗费的毫秒数 　　

%c 输出所属的类目，通常就是所在类的全名 　　

%t 输出产生该日志事件的线程名 　　

%n 输出一个回车换行符，Windows平台为“rn”，Unix平台为“n” 　　

%d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921 　　

%l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10)

%x: 输出和当前线程相关联的NDC(嵌套诊断环境),尤其用到像java servlets这样的多客户多线程的应用中。

%%: 输出一个”%”字符 %F: 输出日志消息产生时所在的文件名称

%L: 输出代码中的行号

%m: 输出代码中指定的消息,产生的日志具体信息

%n: 输出一个回车换行符，Windows平台为”\r\n”，Unix平台为”\n”输出日志信息换行 可以在%与模式字符之间加上修饰符来控制其最小宽度、最大宽度、和文本的对齐方式。

Ok，基本知识就到这里，看一个例子。

##**Example**

{% codeblock lang:groovy %}
apply plugin: 'idea'
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.slf4j:slf4j-api:1.7.7'
    compile 'org.slf4j:slf4j-log4j12:1.7.7'
}
{% endcodeblock %}
在配置依赖时，将SLF4J和log4j绑定

{% codeblock lang:java %}
package log;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {

	private final Logger logger = LoggerFactory.getLogger(HelloWorld.class);

	public void say() {
		logger.info("start time");
		System.out.println("Hello, world!!!");
		System.out.println(logger.getName());
		logger.info("end time");
	}

	public static void main(String args[]) {
		HelloWorld helloWorld = new HelloWorld();
		helloWorld.say();
	}
}
{% endcodeblock %}
通过SLF4J的LoggerFactory.getLogger()方法得到一个logger的实力。logger的名字就是类名（全限定的类名）。

{% codeblock lang:properties %}
log4j.rootLogger=info, console, file

log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%p %l %d{yyy MMM dd HH:mm:ss,SSS} %m %n

log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
log4j.appender.file.File=logger.log
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%p %l %d{yyy MMM dd HH:mm:ss,SSS} %m %n
{% endcodeblock %}
在log4j的properties文件中，指定了根logger的级别，以及它的两个appender，分别是console和file。然后定义这两个appender，并指定它们的layout使用PatternLayout，让它们输出：级别，日志输出的位置，时间，指定的信息和换行。

输出结果如下：

INFO log.HelloWorld.say(HelloWorld.java:11) 2014 Jun 01 16:57:54,807 start time   
Hello, world!!!  
INFO log.HelloWorld.say(HelloWorld.java:13) 2014 Jun 01 16:57:54,809 end time  
log.HelloWorld  

总结，log不算是功能代码，但是他对于开发人员和维护人员，非常有用了，往往是解决问题的唯一线索，特别是对于后台系统，往往只能通过log来分析问题。

参考资料：

1.http://logging.apache.org/log4j/1.2/manual.html  
2.http://www.slf4j.org/manual.html  
3.http://www.cnblogs.com/dennisit/archive/2013/01/01/2841603.html
