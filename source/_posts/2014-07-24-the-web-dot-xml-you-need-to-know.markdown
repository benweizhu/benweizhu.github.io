---
layout: post
title: "Spring，Gradle，Web.xml和Intellij"
date: 2014-07-24 21:38
comments: true
categories: 
---

标题取得不是太好哈，但是看这标题，就知道，这篇文章不是什么高大上的内容。四个看似完全搭不上边的东西，把它们结合在一起使用的时候，对于大部分Java新人来说，却绝对是个头疼的问题。

好，先把问题摆出来，看例子：

我有一个Spring Web MVC的小例子，项目结构是这样的：

{% img center /../images/springgradlewebintellij/project-structure.png %}


configuration目录中，配置有一些Spring的Bean，比如Service类，Dao类等等。比如：

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" class="me.zeph.springmvc.service.HelloService"/>
</beans>
{% endcodeblock %}

项目中的xx-servlet.xml很简单，如下：

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd 
       http://www.springframework.org/schema/context 
       http://www.springframework.org/schema/context/spring-context.xsd 
       http://www.springframework.org/schema/mvc 
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="me.zeph.springmvc"/>

    <bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">
        <property name="prefix" value="/WEB-INF/view/"/>
        <property name="suffix" value=".jsp"/>
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    </bean>

    <mvc:annotation-driven/>

</beans>
{% endcodeblock %}

问题是，我应该在怎么写web.xml和build.xml？才能够让它跑起来。

大部分进入项目的人，包括我在内，都很少有机会能够在项目开始时就加入，那么你就没有机会参与到项目结构的配置过程中，这样对于大部分人都很难有这方面的经验，除非你去做项目的迁移工作。

最常见的问题就是，跑一个gradle jettyRun，报了一个异常，XXXBean Not Define，然后就不知所措了。

今天，我们就通过这个很小的例子来了解一下，如何通过Gradle来合理配置Spring的Bean定义文件？

要弄清楚这个内容，你需要有几项基本知识：

1.ClassPath  
2.Gradle的SourceSet  
3.web.xml中<context-param>的含义  
4.contextConfigLocation  
5.ContextLoaderListener  
6.web.xml中classpath: 符号  

关于前两项：

关于ClassPath的基本知识，可以自学，或者看我的这篇文章：http://benweizhu.github.io/blog/2014/04/07/write-java-code-without-ide/     （丢掉IDE，回到Java的第一堂课）

SourceSet，请阅读http://www.gradle.org/docs/current/userguide/java_plugin.html  ，基本概念就是，它是Gradle的Java插件引入的一个概念，用于告诉Gradle，项目哪些目录是源文件，需要Gradle在打包的时候将这些文件加入。

后面的，我们一边看答案，一边了解。

先来看web.xml该怎么写？

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
        http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

    <servlet>
        <servlet-name>webapp</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>webapp</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:spring/applicationContextService.xml,
            classpath:spring/applicationContextDao.xml,
            classpath:spring/applicationContextJMS.xml
        </param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

</web-app>
{% endcodeblock %}

除了配置servlet，这里还有两项配置，分别是context-param和listener。

**context-param**

web.xml的配置中<context-param>配置作用：（为了不赘述，引用文章，http://www.cnblogs.com/hzj-/articles/1689836.html ）

1.启动一个WEB项目的时候,容器(如:Tomcat)会去读它的配置文件web.xml.读两个节点: "<listener>" 和 "<context-param>"

2.紧接着,容器创建一个ServletContext(上下文),这个WEB项目所有部分都将共享这个上下文.

3.容器将<context-param></context-param>转化为键值对,并交给ServletContext.

4.容器创建<listener></listener>中的类实例,即创建监听.

5.在监听中会有contextInitialized(ServletContextEvent args)初始化方法,在这个方法中获得ServletContext = ServletContextEvent.getServletContext();context-param的值 = ServletContext.getInitParameter("context-param的键");

6.得到这个context-param的值之后,你就可以做一些操作了.注意,这个时候你的WEB项目还没有完全启动完成.这个动作会比所有的Servlet都要早.换句话说,这个时候,你对<context-param>中的键值做的操作,将在你的WEB项目完全启动之前被执行.

**listener**

在这里，ContextLoaderListener的作用是为Servlet初始化Spring的Web应用上下文，而上下文的内容就是上面<context-param>中配置的contextConfigLocation的内容。

你只需要知道他们的作用是什么，如何配置的，不用太深入的知道，Spring到底是怎么使用它们的。

于是，有了这些基本了解之后，再来看怎么写contextConfigLocation的内容，以及为什么这么写？

首先，classpath:spring/applicationContextService.xml是什么意思？它是说在当前应用的classpath下去寻找spring目录下的applicationContextService.xml文件。

就以Web开发为例，web应用程序是以War包的形式存在，它的classpath，就是War中的WEB-INF/class目录。只要你的applicationContextService.xml在war包的class目录下，它就在classpath路径下的。那么针对上例，你需要在打包的时候将applicationContextService.xml放在War包中的WEB-INF/class/spring目录下。

道理很简单吧，那么问题来了，怎么样让Gradle帮我把它打到War包中呢？

答案是自定义SourceSet，告诉Gradle applicationContextService.xml是我的源文件，我希望你帮我打到War包中。

如果你有去了解SourceSet，你应该知道，它有默认的规约，就是遵循Maven的项目布局模式。main/java，main/sources等。

在这个例子里，configuration下所有的东西，我都希望打到war包的classpath下，于是你就需要自定义SourceSet，告诉Gradle，我的源文件在哪个位置。

{% codeblock lang:groovy %}
apply plugin: 'jetty'
apply plugin: 'idea'

repositories {
    mavenCentral()
}

sourceSets {
    main {
        resources {
            srcDir 'configuration'
        }
    }
}

dependencies {
    compile 'org.springframework:spring-webmvc:4.0.6.RELEASE'
}
{% endcodeblock %}

当以上这些你都配置好之后，你可以运行 gradle war命令，打一个war包，然后将它解压缩，看看里面class目录的内容，应该是包含spring/applicationContext**等相关文件。

现在你应该清楚Gradle，Web.xml和Spring是怎么相亲相爱的在一起了吧。

**我们再来看一个更有趣的内容，Intellij和它们有什么关系呢？**

这是很多初学Java和使用Intellij的人容易犯的错误。

你肯定遇到过这个问题，为什么我的Intellij的xml配置文件中一些路径的配置老是红的，比如classpath:applicationXX.xml，而一些又是好的。

为什么我的配置没有红，而且它可以通过Ctrl + B可以跳转到源文件，但是启动jettyRun的时候还是提示我找不到Bean呢?

那是因为Intellij和你启动Gradle打包和部署一点关系都没有。所以答案就是：没有关系。

之所以会红，是因为，xml文件所在目录，你并没有在Intellij中Mark为SourceRoot（会变成绿色文件夹那个）。所以Intellij不认为那是你的源代码，所以你在classpath：时，它就认为你指定的不对，如果你将该目录mark为SourceRoot，Intellij不仅不红，还是很智能的提示你，帮你补全。

相反，那为什么他都帮我补全，而且可以Ctrl + B 跳过去了，但是Gradle jettyRun却提示找不到Bean呢？那是因为，你在Intellij中设置了Source目录，但是Gradle并不知道啊！！！你是否还会天真的以为，明明找得到啊，都Ctrl + B跳过去了，为什么还是不对呢？

好吧，这是很多新人，包括以前的我在内，都不太了解的基本知识。

希望今天这篇文章对大家有帮助，帮助大家解答所有的疑惑。不早了，晚安。


参考资料：

1.http://www.cnblogs.com/hellojava/archive/2012/12/28/2835730.html

2.http://www.gradle.org/docs/current/userguide/java_plugin.html

3.http://lyfei022.blog.163.com/blog/static/82558312009112943635741/

4.http://www.cnblogs.com/hzj-/articles/1689836.html
