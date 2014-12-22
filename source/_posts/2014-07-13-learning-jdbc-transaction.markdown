---
layout: post
title: "再次了解JDBC（下）- 事务"
date: 2014-07-13 13:37
comments: true
categories: 
---
事务就是将一条或者多条语句作为一个单元一起执行，要么全部执行，要么全都不执行。

如果你读过数据库的书，肯定知道数据库事务有四个基本特性：原子性，一致性，隔离性，持久性。

在JDBC中，事务的操作建立在Connection对象上，Connection对象提供了与事务相关的操作函数，例如：setAutoCommit()，commit()，rollback()等。


##**autocommit**

JDBC Connection默认的情况是AutoCommit模式，即每一条SQL语句在执行完之后都会提交到数据库中。对于简单的应用是可以的，但是关闭自动提交模式，由自己管理实务是有必要的，提高执行效率，保证业务处理过程完整性，使用分布式事务。

事务可以让你控制对数据库的改变，它将一条或多条SQL语句作为一个逻辑单元，如果任何一条语句失败，则整个事务失败。

关闭自动提交模式的方法很简单，调用setAutoCommit()方法：

{% codeblock lang:java %}
connection.setAutoCommit(false);
{% endcodeblock %}

##**commit & rollback**
一旦你完成了对数据库的改变操作，你需要执行commit()方法来提交改变，当执行过程中出现异常，你需要执行rollback()方法来回滚该事物。

{% codeblock lang:java %}
connection.commit();
{% endcodeblock %}

{% codeblock lang:java %}
connection.rollback();
{% endcodeblock %}

调用rollback方法会终结一个事务，并返回到数据被修改之前的值。如果你尝试在一个事务中去执行一条或多条语句，结果得到一个SQLException，调用rollback去结束一个事务，然后重新开始事务。这是唯一知道什么被提交，什么没有被提交的办法。因为捕获到SQLException会告诉你什么样的错误发生了，但是不会告诉你什么已经提交，什么没有。回滚是唯一的可靠办法。

##**完整性**
除了将语句分组以一个单元统一执行，事务还帮助保证一张表中数据的完整性。事务会提供某种程度的保护，以防止两个用户同时访问数据时，造成的冲突。

DBMS会使用锁机制，来防止其他用户对已经被事务访问的数据进行访问。一旦加锁，它会强制保证不变直到事务被提交。锁机制的目的是防止用户读到脏数据，也就是读到一些还没有被永久保存的数据（访问一个被更新但是还没有被提交的值，被认为是访问到脏数据，因为这个值很有可能会被回滚到以前的结果，那么你读到的值就是无效的）。

锁是如何被设置的是由一个叫做事务隔离级别决定的。举例来说，如果事务隔离级别被设置为TRANSACTION_READ_COMMITTED，那么它就不会允许数据被访问，直到事务提交。换句话说，DBMS不允许读取脏数据的事件发生。

TRANSACTION_NONE JDBC 驱动不支持事务  
TRANSACTION_READ_UNCOMMITTED 允许脏读，不可重复读和幻读。  
TRANSACTION_READ_COMMITTED 禁止脏读，但允许不可重复读和幻读。  
TRANSACTION_REPEATABLE_READ 禁止脏读和不可重复读，单运行幻读。  
TRANSACTION_SERIALIZABLE 禁止脏读，不可重复读和幻读。  

不可重复读的场景发生在事务A读取一行数据，事务B后续的改变了这一行，当事务A再次去读时，两次读取的事务就不一致了。

幻读的场景是事务A读取到满足一定条件的一部分数据，事务B后续插入或者更新了一行数据，但是同样满足该条件，此时A再去读取发现多了一行数据。

通常，你不需要对事务隔离级别做任何操作，只要使用默认的即可，但默认值取决于DBMS。例如，对于Java数据库，默认值是TRANSACTION_READ_COMMITTED。JDBC允许你获取和改变该级别，方法是getTransactionIsolation()和setTransactionIsolation()。


参考资料：

http://docs.oracle.com/javase/tutorial/jdbc/basics/transactions.html

http://www.tutorialspoint.com/jdbc/jdbc-transactions.htm

http://blog.csdn.net/chenyongsuda/article/details/5641412