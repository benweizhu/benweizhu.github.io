---
layout: post
title: "Web MVC By Example"
date: 2013-12-21 17:03
comments: true
categories:
---
早期的web应用主要是静态页面的浏览。静态页面使用HTML语言来编写，放在服务器上。用户使用浏览器通过HTTP协议请求服务器上的Web页面，服务器上的Web服务器软件接收到用户发送的请求后，读取请求URI所标识的资源，加上消息报头发送给客户端的浏览器；浏览器解析响应中的HTML数据，从而向用户呈现多姿多彩的HTML页面。

###**CGI->Servlet->JSP->Model1->Model2**

早期使用的Web服务器扩展机制是CGI，它允许用户调用Web服务器上的CGI程序。CGI的全称是Common Gateway Interface，即公共网关接口。用户通过单击某个链接或者直接在浏览器的地址栏中输入URL来访问CGI程序，Web服务器接收到请求后，发现这个请求是给CGI程序的，于是就启动并运行这个CGI程序，对用户请求进行处理。CGI程序解析请求中的CGI数据，处理数据，并产生一个响应（通常是HTML页面）。这个响应被返回给Web服务器，Web服务器包装这个响应（例如添加消息报头），以HTTP响应的形式发送给Web浏览器。

Java Servlet（**Java服务器小程序Servlet=Server+Applet**）是一个基于Java技术的Web组件，运行在服务器端，由Servlet容器所管理，用于生成动态的内容。Servlet是平台独立的Java类，编写一个Servlet，实际上就是按照Servlet规范编写一个Java类。

Servlet不能独立运行，它必须被部署到Servlet容器（例如：Tomcat）中，由容器来实例化和调用Servlet的方法，Servlet容器在Servlet的生命周期内包容和管理Servlet。

用户通过单击某个链接或者直接在浏览器的地址栏中输入URL来访问Servlet，Web服务器接收到该请求后，并不是将请求直接交给Servlet，而是交给Servlet容器。Servlet容器实例化Servlet，调用Servlet的一个特定方法对请求进行处理，并产生一个响应。这个响应由Servlet容器返回给Web服务器，Web服务器包装这个响应，以HTTP响应的形式发送给Web浏览器。

下面是用servlet编写的get和post请求页面例子：
{% codeblock web.xml lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
		  http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         version="2.5">

   <servlet>
       <servlet-name>hello</servlet-name>
       <servlet-class>me.zeph.springview.demo.HelloWorldServlet</servlet-class>
   </servlet>

   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello</url-pattern>
   </servlet-mapping>

</web-app>
{% endcodeblock %}

{% codeblock servlet.java lang:java %}
package me.zeph.springview.demo;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class HelloWorldServlet extends HttpServlet {
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html");
		PrintWriter writer = resp.getWriter();
		writer.write("<h1>HelloWorld</h1>");
		writer.write("<form action='hello' method='post'>");
		writer.write("<input type='text' name='value'>");
		writer.write("<input type='submit' value='submit'>");
		writer.write("<form/>");
	}

   @Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html");
		String value = req.getParameter("value");
		PrintWriter writer = resp.getWriter();
		writer.write("Hi, " + value);
	}
}

{% endcodeblock %}
这样的web开发流程都在实现一个servlet类的实例。也就是无论是业务逻辑还是前端显示，都是放在Servlet类中来完成，通过实现doGet和doPost的方法，来完成前端参数的获取和视图的渲染。

但这样做的缺点是**表现逻辑、控制逻辑和业务逻辑全部写在了Java类中，导致逻辑有些混乱**。

之后，JSP技术出现，它由Sun和许多公司参与共同创建的一种使软件开发者可以响应客户端请求，而动态生成HTML、XML或其他格式文档的Web网页的技术标准。

它以Java语言作为脚本语言的，使Java代码和特定的预定义动作可以嵌入到静态页面中。

Sun这样做提供了非常好的前端显示，**前端开发工程师也可以更好地改善页面体验**，但实际上，它仍然没有解决前端显示和控制逻辑以及业务逻辑的混乱问题。

**Servlet是将前端显示放在了Java类中，而JSP是将控制和业务逻辑放在了JSP中。**

JSP+JavaBean是纯JSP的增强版被称为Model1。特点是使用<jsp:useBean>标准动作，自动将请求参数封装为JavaBean组件。

这样可以将部分业务逻辑封装到JavaBean中。

Model1的架构中，JSP要负责控制逻辑、表现逻辑和业务对象（JavaBean）的调用，所以实际上仍然不理想。

**Model2架构其实可以认为就是我们所说的Web MVC模型，只是控制器采用Servlet、模型采用JavaBean、视图采用JSP**。

下面是一个Model2的例子：

web.xml配置和上面一样配置一个Servlet映射。但是Servlet只负责控制逻辑，不进行前端操作，而是直接将前端渲染分发给jsp页面进行。
{% codeblock HelloWorldServlet.java lang:java%}
package me.zeph.springview.demo;

import me.zeph.springview.demo.domain.User;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class HelloWorldServlet extends HttpServlet {

   public static final String GET_PATH = "WEB-INF/view/hello-world.jsp";
   public static final String POST_PATH = "WEB-INF/view/hello-world-confirm.jsp";

   @Override
   protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
	   req.getRequestDispatcher(GET_PATH).forward(req, resp);
   }

   @Override
   protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
   	   User user = getUserFromRequest(req);
	   String path = null;
	   if (user.validate()) {
	      req.setAttribute("user", user);
	      path = POST_PATH;
	   } else {
		  req.setAttribute("error", true);
		  path = GET_PATH;
	   }
	   req.getRequestDispatcher(path).forward(req, resp);
   }

   private User getUserFromRequest(HttpServletRequest req) {
	   User user = new User();
	   user.setName(req.getParameter("name"));
	   user.setPassword(req.getParameter("password"));
	   return user;
   }
}

{% endcodeblock %}
{% codeblock hello-world.jsp lang:html%}
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>hello world</title>
</head>
<body>
<form action="helloWorld" method="post">
    <label for="name">name:</label><input type="text" id="name" name="name">
    <label for="password">password:</label><input type="password" id="password" name="password">
    <input type="submit" value="submit">
    <c:if test="${error}">
        login failed
    </c:if>
</form>
</body>
</html>
{% endcodeblock %}
{% codeblock hello-world-confirm.jsp lang:html%}
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>helloWorldConfirm</title>
</head>
<body>
hi,${user.name}
</body>
</html>
{% endcodeblock %}

{% codeblock User.java lang:java %}
package me.zeph.springview.demo.domain;

public class User {
	public static final String NAME = "zeph";
	public static final String PASSWORD = "123456";
	private String name;
	private String password;

   public String getName() {
		return name;
	}

   public void setName(String name) {
		this.name = name;
	}

   public String getPassword() {
		return password;
	}

   public void setPassword(String password) {
		this.password = password;
	}

   public boolean validate() {
		return NAME.equals(name) && PASSWORD.equals(password);
	}
}

{% endcodeblock %}

总结，Model2模式已经很好的将控制逻辑，业务逻辑和显示逻辑分开，但其中也有些位置还可以进一步改进。

例如：**从参数到业务模型的转换（数据绑定），视图显示技术不容易更换（View解析）。**

不过，Web MVC框架替我们解决的了这些问题（流行的Web MVC框架有：Spring MVC，Struts）。

关于Spring MVC框架的使用将留在下一篇介绍。
