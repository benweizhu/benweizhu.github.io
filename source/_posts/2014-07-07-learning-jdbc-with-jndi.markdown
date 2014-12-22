---
layout: post
title: "再次了解JDBC（中）- 引入JNDI"
date: 2014-07-07 13:56
comments: true
categories: 
---

##**什么是命名服务？什么是目录服务？什么是命名服务的上下文？**

命名服务NS（Naming Service）是命名系统提供的服务功能，通过名字访问命名系统中的对象。

目录服务DS(Directory Service)是命名服务的扩展，它不仅把名字与对象对应在一起，并且把名字与对象属性(Attribute) 联系在一起，因此不仅可以通过名字还可以通过属性来搜索对象。

对象名与对应的对象构成的集合叫对象上下文（Context）。
例如，在文件命名系统中，一个目录就是一个Context，其内容是文件名（名）和对应的文件的集合。

##**那什么是JNDI？**

Java Naming and Dirctory Interface - Java命名和目录接口，它是sun公司提出的方便应用程序访问命名和目录服务的的API。

和其他设计一样，JNDI是接口API，它独立于任何命名或者目录服务的具体实现。这样，你就可以用相同的API去访问多种不同类型的命名和目录服务。

根据它们作用的不同，典型应用场景也就分为两个部分：

（1）将Java应用程序连接到外部的目录服务。

（2）允许Java的Servlet在web容器中寻找定义配置信息。

JNDI的架构是有一套API和一套SPI（Service Provider Interface）接口组成。Java应用程序使用JNDI API来访问不同的命名和目录服务。SPI则让不同的命名和目录服务可以透明和无缝插入，这样Java应用程序才能使用JNDI API来访问它们的服务。

{% img center /../images/jndi/arch.png %}

JNDI是包含在Java SE平台中。要使用JNDI，你必须有JNDI的类库和一个到多个服务提供商。JDK本身包含一些服务提供商：

Lightweight Directory Access Protocol (LDAP)  
Common Object Request Broker Architecture (CORBA) Common Object Services (COS) name service  
Java Remote Method Invocation (RMI) Registry  
Domain Name Service (DNS)

JNDI API是访问任何命名或者目录服务的通用API。实际访问一个命名或目录服务需要在JNDI下插入一个服务提供商。

服务提供商是一个映射到JNDI API能实际调用命名或目录服务器的软件。一般情况，服务提供商的角色和命名或目录服务器的角色是不一样的。从C/S软件角度说，JNDI和服务器提供商是JNDI的客户端，命名或者目录服务器是服务端。

客户端和服务器端交互的方式有很多。一种比较常用的方式是，使用网络协议。而服务器通常支持不同的客户端，不仅仅是JNDI的客户端，只是它们要遵循不同协议。JNDI也不规定JNDI客户端和服务端交互的方式。

##**JDNI的Context（上下文）**

上下文这个概念在Java的开发中经常出现，比如，ServletContext，Spring的Context，Android中也有Context，JNDI也不例外。

JNDI的上下文，依赖于一个重要的接口Context和一个重要的类InitialContext

Context接口 它表示一个命名上下文，由一组名称到对象的绑定组成。它提供了查找，绑定，重命名，创建和销毁子上下文的接口。

InitialContext类 所有命名操作都相对于某一上下文，它是JNDI提供的，执行命名和目录操作的初始上下文，是根上下文，为命名和目录服务提供起点。一旦你有了初始化上下文，你就可以去查找其他的上下文和对象。

##**JNDI的环境变量**

JNDI需要定义许多环境变量来说明访问什么样的命名和目录服务。

而为了简化设置JNDI应用需要的环境变量，应用程序组件和服务提供商会和资源文件一起发布。JNDI的资源文件就是常用的properties文件格式，包含的是键值对。

JNDI的资源文件有两种类型：provider和application

每个服务商都有一个可选的资源文件：[prefix/]jndiprovider.properties，这个prefix前缀是context实现类的包名。例如：

com.sun.jndi.ldap.LdapCtx对应的资源文件是com/sun/jndi/ldap/jndiprovider.properties

JNDI会使用ClassLoader.getResources()方法在应用程序的所有资源文件中查找一个叫jndi.properties的文件。该文件中定义的所有属性都会放到InitialContext里面，而其他的Context会继承自该InitialContext。

当InitialContext被构建时，它的环境会被初始化，要么通过传递进来的HashMap参数，要么通过定义的Java应用properties文件。而IntialContext的具体实现是在运行时决定的，默认的策略是使用环境变量“java.naming.factory.initial”定义的InitContextFactory（工厂类）。

**回到我们的例子当中：**

那么在[上一篇](http://benweizhu.github.io/blog/2014/07/06/learning-jdbc/ "再次了解JDBC（上）")中，DataSource API文档里提到的“实现DataSource接口的对象通常在基于JavaTM Naming and Directory Interface(JNDI) API的命名服务中注册。”就是JNDI的第二种应用方式。

我们使用的Apache Common的DBCP作为DataSource的实现，而DBCP也是Tomcat的数据库连接池组建，所以针对它，我么可以使用Tomcat实现的JNDI服务。

首先需要把上一篇中的例子改为一个Java Web应用。

定义Context.xml，位置在webapp/META-INF/context.xml
{% codeblock lang:xml %}
<?xml version='1.0' encoding='utf-8'?>
<Context>
    <Resource name="jdbc/mysql/bookshelf" auth="Container" type="javax.sql.DataSource"
              maxActive="100" maxIdle="30" maxWait="10000"
              username="root" password="" driverClassName="com.mysql.jdbc.Driver"
              url="jdbc:mysql://localhost:3306/bookshelf"/>
</Context>
{% endcodeblock %}
在JDNI中，对象是一种资源，Tomcat指定资源的入口出在<Context>元素中，有两个位置可以定义，一个是在$CATALINA_BASE/conf/server.xml，一个是在每个web应用需的META-INF/context.xml中。前一种方式，Tomcat容器中所有的Web应用都可以使用，算是一种全局的资源。不过一般第二种方式会更好。

我在使用第一种方式的时候，遇到了MySQL的Driver文件找不到的问题，应该是需要将MySQL的驱动拷贝的Tomcat的lib下。

定义web.xml文件，位置在webapp/WEB-INF/web.xml
{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
        http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

    <servlet>
        <servlet-name>helloDataSource</servlet-name>
        <servlet-class>me.zeph.jdbc.example.servlet.HelloDataSourceServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>helloDataSource</servlet-name>
        <url-pattern>/helloDataSource</url-pattern>
    </servlet-mapping>

    <resource-ref>
        <description>DB Connection</description>
        <res-ref-name>jdbc/mysql/bookshelf</res-ref-name>
        <res-type>javax.sql.DataSource</res-type>
        <res-auth>Application</res-auth>
    </resource-ref>

</web-app>
{% endcodeblock %}
光定义资源还不行，web应用必须要有个办法知道资源有哪些？所以需要在web.xml定义资源的引用。

此时，获取Connection的方式就可以换成JNDI的方式了。
{% codeblock lang:java %}
package me.zeph.jdbc.example.dao;

import me.zeph.jdbc.example.model.Book;

import javax.naming.Context;
import javax.naming.InitialContext;
import javax.naming.NamingException;
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class BookDaoWithDS {

	public Book findBookByISBN(int isbn) {
		Book book = null;
		Statement statement = null;
		Connection connection = null;
		try {
			connection = getConnection();
			statement = connection.createStatement();
			book = getBook(statement.executeQuery(getQuerySqlFor(isbn)));
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			try {
				if (statement != null) {
					statement.close();
				}
				if (connection != null) {
					connection.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
		return book;
	}

	private Connection getConnection() throws SQLException {
		DataSource dataSource = null;
		try {
			InitialContext initialContext = new InitialContext();
			Context envContext = (Context) initialContext.lookup("java:/comp/env");
			dataSource = (DataSource) envContext.lookup("jdbc/mysql/bookshelf");
		} catch (NamingException e) {
			e.printStackTrace();
		}
		return dataSource.getConnection();
	}

	private String getQuerySqlFor(int isbn) {
		return "select * from book where isbn = " + isbn + ";";
	}

	private Book getBook(ResultSet resultSet) throws SQLException {
		Book book = null;
		while (resultSet.next()) {
			book = new Book();
			book.setIsbn(resultSet.getInt(1));
			book.setName(resultSet.getString(2));
			book.setPrice(resultSet.getDouble(3));
			book.setAuthor(resultSet.getString(4));
		}
		return book;
	}
}
{% endcodeblock %}

这里你可以将java.naming.factory.initial和java.naming.factory.url.pkgs打印出来，结果是：org.apache.naming.java.javaURLContextFactory和org.apache.naming。

java.naming.factory.url.pkgs的作用是告诉JNDI去哪个包下面，找满足java.javaURLContextFactory的类。

你应该还看到一点有点疑惑，我的资源名字命名就是：jdbc/mysql/bookshelf，为什么前面还有java:/comp/env。

java:/comp/env是环境命名上下文，是针对Java EE组件中使用JNDI引入的，目的是为了防止冲突。Java EE环境下，被访问的系统或者用户定义的对象都是存储在java:comp/env的环境命名上下文中。

到此，我们实现了JNDI的引入，可以通过JNDI来配置DataSource，此时，如果我们希望从MySQL迁移到Oracle就不需要修改Java代码，只需要更改一下配置文件即可，这也就是JNDI的好处。

再下一篇，我们继续讨论，JDBC的事务。

参考资料：

http://docs.oracle.com/javase/tutorial/jndi/overview/index.html

http://tomcat.apache.org/tomcat-7.0-doc/jndi-resources-howto.html

http://docs.oracle.com/javase/jndi/tutorial/

http://docs.oracle.com/javase/8/docs/api/javax/naming/Context.html