---
layout: post
title: "Spring Web Security 实战 (一) - 最少配置启动"
date: 2015-06-12 21:12:38 +0800
comments: true
categories: 
---

你大概应该知道对于应用的安全性（无论是Web应用还是非Web应用），都包含两个方面“身份认证”（authentication）和“权限认证”（authorization，access-control）。身份认证是确立“访问者”所声称的“当事人”（principal）的一个过程（当事人可以是一个用户，设备，或者其他任何要在该应用操作的系统）。权限认证指的是确定该“当事人”是否可以在应用执行某一个操作的过程。

Spring Security能提供的功能亦围绕着这两个核心概念展开。

在“身份认证”方面，Spring Security能提供什么样的支持呢？如下：

HTTP BASIC authentication headers(an IETF RFC-based standard)     
HTTP Digest authentication headers(an IETF RFC-based standard)     
HTTPX.509 client certificate exchange(an IETF RFC-based standard)     
Form-based authentication(for simple user interface needs)     
...
更多类型，请参考Spring Security官文。

好吧，概念性的东西就介绍到这，本篇文章以实战为主，下面我们还是来聊聊大家更关心的实现：

在Spring Security 3.0中，相应的功能代码被分到了不同的Jar包中，以清晰的区分不同的功能和第三方依赖。下面介绍三个主要的Jar包：

Core - spring-security-core.jar

包含核心的“身份认证”和“访问控制”类和接口。任何Spring Security的应用都需要这个包，支持独立应用，远程客户端，方法级别（服务层）的安全，以及JDBC，主要包含下面几个顶级包：        
org.springframework.security.core    
org.springframework.security.access    
org.springframework.security.authentication      
org.springframework.security.provisioning    

Web - spring-security-web.jar

包含对应的filter和web security相关的基础架构代码。任何与Servlet API相关的依赖。如果你需要Spring Security对web身份认证和URL级别的访问控制的支持，就需要它。里面主要包含的包是：org.springframework.security.web。

Config - spring-security-config.jar

包含security命名空间解析代码。如果你要使用XML方式的配置Security，那么你就会需要它。主要包含的包：org.springframework.security.config。


以web应用和xml配置为例，要开始使用Spring Security，需要至少下面的两个依赖，web和config，web会自动依赖于core，所以不用显示的指定core。

{% codeblock lang:groovy %}
dependencies {
    compile 'org.springframework.security:spring-security-web:3.2.7.RELEASE'
    compile 'org.springframework.security:spring-security-config:3.2.7.RELEASE'
}
{% endcodeblock %}

为了能够支持更多的Spring使用，我们以XML方法来配置Security框架。

下面介绍下在Security命名空间下，都有哪些重要的元素节点。

• Web/HTTP Security - 设置filter和相关的Service Bean，以应用框架的用户认证机制，安全化URL，渲染登录，错误页面等。

• Business Object(Method) Security - 用来保证业务层的安全。

• Authentication Manager - 处理身份认证请求

• Access Decision Manager - 为web和method提供访问决策。框架提供了一个默认实现，你也可以自定义一个，语法和普通Bean定义一样。

• AuthenticationProviders - 与Authentication Manager相对应的身份认真机制提供方，提供了多种标准选择，同样也支持自定义。

• UserDetailsService - 与authentication providers联系紧密，也会被其他Bean使用。

接下来，我们开始正式写代码。

首先，你需要在web.xml中定义一个名字是springSecurityFilterChain的filter。它提供了一个“钩子“，来启动Spring Security Web的基础架构。DelegatingFilterProxy是Spring框架中一个类，它代理着一个以Spring Bean方式定义在Spring上下文中的filter。在本例中，bean的名字是springSecurityFilterChain，它是Spring提供的内部基础架构Bean，所以，我们定义的一些自定义的Bean的时候，就不可以使用这个名字springSecurityFilterChain。代码如下：

{% codeblock lang:xml %}
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>

<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
{% endcodeblock %}

springSecurityFilterChain是由Spring框架定义。
{% codeblock lang:java %}
package org.springframework.security.config.annotation.web.configuration;
public class WebSecurityConfiguration{
	@org.springframework.context.annotation.Bean(name = {"springSecurityFilterChain"})
	public javax.servlet.Filter springSecurityFilterChain() throws java.lang.Exception { /* compiled code */ }	
}
{% endcodeblock %}

做完上面的”钩子”配置，你就可以正式开始Spring上下文的定义。首先来配置Web/HTTP Security节点部分“http”：

{% codeblock lang:xml %}
<http use-expressions="true">
    <intercept-url pattern="/**" access="hasRole('ROLE_USER')"/>
    <form-login/>
    <logout/>
</http>
{% endcodeblock %}

上面是使得Spring Security工作的最小http节点配置。安全拦截所有应用中的URL，要求访问者（“当事人”）含有ROLE_USER权限，要求使用带有用户名和密码的表单登录，并提供一个登出的URL。上面的pattern可以使用正则表达式来匹配对应路径，access中的hasRole必须配合use-expressions="true"才能使用（4.0以上有所不同）。如果提供多种权限，ROLE_USER后面可以以逗号分隔以允许多种权限（如果是4.0以上版本，Spring会自动给hasRole中的值加上ROLE_作为前缀，所以需要注意）。

这里允许定义多个intercept-url以应对不同的url，比如，css，js的访问，自定义登陆页面的访问，等。由于该intercept-url会根据定义的先后顺序来解析输入的url，所以，越明确的url模式，越应该放在前面。

note：如果你对filter比较熟悉，你应该能明白intercept-url其实是在配置FilterChainProxy所使用的filter。

接下来，需要配置Authentication Manager：

{% codeblock lang:xml %}
<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="ben" password="ben" authorities="ROLE_USER"/>
        </user-service>
    </authentication-provider>
</authentication-manager>
{% endcodeblock %}

authentication-provider和user-service分别创建了两个bean，DaoAuthenticationProvider和InMemoryDaoImpl。它们作为authentication-manager的子元素存在，而authentication-manager则创建了一个ProviderManager的Bean，并将authentication-provider注册到该ProviderManager。理解这段代码的含义很简单，一个用户ben，密码ben，拥有ROLE_USER这个权限。

你还可以通过user-service上的properties属性来指定一个properties文件作为用户定义的输入。

到此，你就完成了一个最简单的Spring Web Security的配置。拦截所有url，需要输入用户名和密码登录访问。

参考资料：     
1. Spring Security Reference。
