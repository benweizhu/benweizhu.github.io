---
layout: post
title: "再次了解JDBC（上）- 从Class.forName到DataSource"
date: 2014-07-06 18:21
comments: true
categories: 
---
JDBC（Java Data Base Connectivity，Java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。

##**API**
先来了解JDBC中几个非常重要的API（以下内容来自JDBC API中文文档，但未完全拷贝，建议还是阅读一下）：

**java.sql.DriverManager（类）**：管理一组 JDBC 驱动程序的基本服务。（注：DataSource 接口是 JDBC 2.0 API 中的新增内容，它提供了连接到数据源的另一种方法。使用 DataSource 对象是连接到数据源的首选方法。）作为初始化的一部分，DriverManager 类会尝试加载在 "jdbc.drivers" 系统属性中引用的驱动程序类。这允许用户定制由他们的应用程序使用的 JDBC Driver。应用程序不再需要使用 Class.forName() 显式地加载 JDBC 驱动程序。当前使用 Class.forName() 加载 JDBC 驱动程序的现有程序将在不作修改的情况下继续工作。在调用getConnection方法时，DriverManager会试着从初始化时加载的那些驱动程序以及使用与当前applet或应用程序相同的类加载器显式加载的那些驱动程序中查找合适的驱动程序。

**java.sql.Connection（接口）**：与特定数据库的连接（会话）。在连接上下文中执行 SQL 语句并返回结果。（注：在配置 Connection 时，JDBC 应用程序应该使用适当的Connection方法，比如setAutoCommit或setTransactionIsolation。在有可用的JDBC方法时，应用程序不能直接调用 SQL 命令更改连接的配置。默认情况下，Connection对象处于自动提交模式下，这意味着它在执行每个语句后都会自动提交更改。如果禁用了自动提交模式，那么要提交更改就必须显式调用commit方法；否则无法保存数据库更改。）

**java.sql.Statement（接口）**：用于执行静态SQL语句并返回它所生成结果的对象。

**java.sql.ResultSet（接口）**：表示数据库结果集的数据表，通常通过执行查询数据库的语句生成。ResultSet对象具有指向其当前数据行的光标。最初，光标被置于第一行之前。next方法将光标移动到下一行；因为该方法在ResultSet对象没有下一行时返回false，所以可以在while循环中使用它来迭代结果集。默认的ResultSet对象不可更新，仅有一个向前移动的光标。因此，只能迭代它一次，并且只能按从第一行到最后一行的顺序进行。ResultSet 接口提供用于从当前行获取列值的获取方法（getBoolean、getLong 等）。可以使用列的索引编号或列的名称获取值。一般情况下，使用列索引较为高效。列从1开始编号。为了获得最大的可移植性，应该按从左到右的顺序读取每行中的结果集列，每列只能读取一次。对于获取方法，JDBC 驱动程序尝试将底层数据转换为在获取方法中指定的Java类型，并返回适当的Java值。JDBC规范有一个表，显示允许的从SQL类型到ResultSet获取方法所使用的Java类型的映射关系。

**java.sql.Driver（接口）**：每个驱动程序类必须实现的接口。Java SQL框架允许多个数据库驱动程序。每个驱动程序都应该提供一个实现 Driver 接口的类。DriverManager会试着加载尽可能多的它可以找到的驱动程序，然后，对于任何给定连接请求，它会让每个驱动程序依次试着连接到目标URL。强烈建议每个Driver类应该是小型的并且是单独的，这样就可以在不必引入大量支持代码的情况下加载和查询Driver类。在加载某一Driver类时，它应该创建自己的实例并向DriverManager注册该实例。这意味着用户可以通过调用以下程序加载和注册一个驱动程序Class.forName("foo.bah.Driver")。

来看一段比较老式风格的JDBC代码：
{% codeblock lang:java %}
package me.zeph.jdbc.example.dao;

import me.zeph.jdbc.example.model.Book;

import java.sql.*;

public class BookDaoWithDM {

	private final String url;
	private final String username;
	private final String password;
	private final String driverName;

	public BookDaoWithDM(String url, String username, String password, String driverName) {
		this.url = url;
		this.username = username;
		this.password = password;
		this.driverName = driverName;
	}

	public Book findBookByISBN(int isbn) {
		Book book = null;
		Statement statement = null;
		Connection connection = null;
		try {
			connection = getConnection();
			statement = connection.createStatement();
			book = getBook(statement.executeQuery(getQuerySqlFor(isbn)));
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
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

	private Connection getConnection() throws ClassNotFoundException, SQLException {
		Class.forName(driverName);
		return DriverManager.getConnection(url, username, password);
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

{% codeblock lang:java %}
package me.zeph.jdbc.example.dao;

import me.zeph.jdbc.example.model.Book;
import org.junit.Before;
import org.junit.Test;

import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.CoreMatchers.nullValue;
import static org.junit.Assert.assertThat;

public class BookDaoWithDMTest {

	private final String url = "jdbc:mysql://localhost:3306/bookshelf";
	private final String username = "username";
	private final String password = "";
	private final String driverName = "com.mysql.jdbc.Driver";

	private BookDaoWithDM bookDaoWithDM;

	@Before
	public void setUp() throws Exception {
		bookDaoWithDM = new BookDaoWithDM(url, username, password, driverName);
	}

	@Test
	public void shouldFindBookFromDatabaseWhenISBNIs12() {
		//when
		Book book = bookDaoWithDM.findBookByISBN(12);

		//then
		assertThat(book.getName(), is("benwei"));
	}

	@Test
	public void shouldNotFindBookFromDatabaseWhenISBNIs1() {
		//when
		Book book = bookDaoWithDM.findBookByISBN(1);

		//then
		assertThat(book, is(nullValue()));
	}
}
{% endcodeblock %}

{% codeblock lang:java %}
package me.zeph.jdbc.example.model;

public class Book {
	private int isbn;
	private String name;
	private double price;
	private String author;

	public void setIsbn(int isbn) {
		this.isbn = isbn;
	}

	public int getIsbn() {
		return isbn;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}

	public void setPrice(double price) {
		this.price = price;
	}

	public double getPrice() {
		return price;
	}

	public void setAuthor(String author) {
		this.author = author;
	}

	public String getAuthor() {
		return author;
	}
}
{% endcodeblock %}

##**Class.forName()**

上面这个例子是很普通的JDBC编程，这里有一点要讲的，那就是DriverManager会加载注册到它自己的Driver。但是，在上面的例子中，好像没有看到哪里有显示的注册Driver到DriverManager。为什么DriverManager能够找到MySQL的Driver呢？

回头看API文档中的一句话：

“这意味着用户可以通过调用以下程序加载和注册一个驱动程序Class.forName("foo.bah.Driver")。”。

我们知道Class.forName()方法是将一个指定名字的Class加载到JVM中。这和注册Driver有什么关系呢？看下MySQL实现的Driver源码你就明白了。

{% codeblock lang:java %}
package com.mysql.jdbc

import java.sql.SQLException;

public class Driver extends NonRegisteringDriver implements java.sql.Driver {
	static {
		try {
			java.sql.DriverManager.registerDriver(new Driver());
		} catch (SQLException E) {
			throw new RuntimeException("Can't register driver!");
		}
	}
	public Driver() throws SQLException {
		
	}
}
{% endcodeblock %}

原来在这个Driver实现类中一段静态代码块，在类加载时，会被执行，此时就实现了将自己注册到DriverManager。

##**DataSource**

OK，继续，前面的API文档中提到，目前一种更好的连接到数据源也就是获得Connection的方式是使用DataSource。我们来看下DataSource是什么？同样，先来读一下DataSource的API文档。

**javax.sql.DataSource（接口）**：该工厂用于提供到此DataSource对象所表示的物理数据源的连接（A factory for connections to the physical data source that this DataSource object represents.）。作为 DriverManager工具的替代项，DataSource对象是获取连接的首选方法。实现DataSource接口的对象通常在基于JavaTM Naming and Directory Interface(JNDI) API的命名服务中注册。

DataSource接口由驱动程序供应商实现。共有三种类型的实现：

基本实现 - 生成标准的Connection对象  
连接池实现 - 生成自动参与连接池的Connection对象。此实现与中间层连接池管理器一起使用。  
分布式事务实现 - 生成一个Connection对象，该对象可用于分布式事务，大多数情况下总是参与连接池。此实现与中间层事务管理器一起使用，大多数情况下总是与连接池管理器一起使用。

DataSource对象的属性在必要时可以修改。例如，如果将数据源移动到另一个服务器，则可更改与服务器相关的属性。其优点在于，由于可以更改数据源的属性，所以任何访问该数据源的代码都无需更改。

通过DataSource对象访问的驱动程序本身不会向DriverManager注册。通过查找操作获取DataSource对象，然后使用该对象创建Connection对象。使用基本的实现，通过DataSource对象获取的连接与通过DriverManager设施获取的连接相同。

——————————————————————————————————————————————————————

上面提到了几点：

1.使用JNDI方式注册DataSource，然后在使用时根据名字将DataSource取出。  
2.通过DataSource对象访问的驱动程序本身不会向DriverManager注册。

JNDI(Java Naming and Directory Interface，Java命名和目录接口)是一组在Java应用中访问命名和目录服务的API。命名服务将名称和对象联系起来，使得我们可以用名称访问对象。不过在本文中暂时不去涉及这个内容。

DataSource是一个接口，那么它的实现者有哪些呢？有一个比较常用的，Apache Commons的DBCP（Database Connection Pool），它是Apache Commons提供的一种数据库连接池组件，也是tomcat的数据库连接池组件。（回头看一下API文档上说的三种实现类型的第二种）。

将上面例子中使用DriverManager创建连接的方式，换成使用DataSource非常简单，替换上面的getConnection方法。


{% codeblock lang:java %}
private Connection getConnection() throws SQLException {
		BasicDataSource dataSource = new BasicDataSource();
		dataSource.setUrl(url);
		dataSource.setUsername(username);
		dataSource.setPassword(password);
		return dataSource.getConnection();
}
{% endcodeblock %}

从这段代码可以看出，映射了上面我说的的第二点，不需要向DriverManager注册，所以这里没有传入MySQL的ClassName。

OK，到此，我们回顾了JDBC最基础的几个接口：DriverManager，Driver，Connection，Statement，ResultSet，以及JDBC推荐的数据库连接方式DataSource。在下一章，我们会继续了解与JDBC相关的核心技术和概念，包括事务，JNDI等。
