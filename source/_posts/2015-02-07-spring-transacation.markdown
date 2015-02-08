---
layout: post
title: "了解Spring Transaction事务管理"
date: 2015-02-07 17:33:36 +0800
comments: true
categories: 
---
提供一种统一、抽象的编程模型来管理不同的事务API，如，JavaTransactionAPI(JTA)，JDBC，Hibernate，Java Persistence API (JPA)以及Java Data Objects (JDO)，是选择Spring Transaction做事务管理最直接的理由。

###本地事务和全局事务

全局事务让你可以和多个事务资源工作在一起，比如，关系型数据库，消息队列。

而本地事务则是与某个指定的事务资源联系在一起，比如，与JDBC连接相关的事务。本地事务相对于全局事务更容易使用，但不能跨多个事务资源。管理JDBC连接所写的事务代码不能够在全局事务中使用。

###Spring Transaction

Spring Transaction使得开发人员在任何一个环境中都可以使用相同的编程模型。只要写一次代码，就可以在不同环境下的不同的事务策略中使用。最重要的是Spring提供了声明式的事务管理方式，可以通过配置的方式实现事务管理。

Spring事务抽象中一种关键的概念是：事务策略。

一个事务策略，由PlatformTransactionManager接口所定义：
{% codeblock lang:java %}
public interface PlatformTransactionManager {

	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

	void commit(TransactionStatus status) throws TransactionException;

	void rollback(TransactionStatus status) throws TransactionException;
}
{% endcodeblock %}

PlatformTransactionManager是事务管理的抽象层，Spring根据这个抽象层提供许多不同的具体实现。无论是声明式还是编程式的进行事务管理，你都必须正确的定义PlatformTransactionManager的实现。

在使用PlatformTransactionManager的具体实现时，通常都需要一些与对应工作环境的相关知识，比如：JDBC，JTA，Hibernate。

下面是一个JDBC的例子：
{% codeblock lang:xml %}
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
</bean>

<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
</bean>
{% endcodeblock %}

下面是一个Hibernate的例子：
{% codeblock lang:xml %}
<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="mappingResources">
            <list>
                <value>org/springframework/samples/petclinic/hibernate/petclinic.hbm.xml</value>
            </list>
        </property>
        <property name="hibernateProperties">
            <value>
                hibernate.dialect=${hibernate.dialect}
            </value>
        </property>
</bean>

<bean id="txManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
</bean> 
{% endcodeblock %}

###声明式的事务管理

大部分Spring框架的使用者都会采用声明式的事务管理，因为这样做对产品代码的侵入性是最低的。这种声明式的事务管理，使得它可以和Spring的切面编程（AOP）结合在一起，不过即便你不了解AOP，仍然可以使用，因为它几乎是模板式的配置。

这种声明式的事务管理可以允许你在方法级别上指定事务行为。

####理解Spring声明式事务的实现

要理解Spring声明式事务的实现，你需要知道的最重要的概念就是，Spring声明式事务的实现是通过Spring的AOP代理。

下面是一个通过XML方式配置事务的例子，方便你理解事务的实现方式。
{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx" xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- this is the service object that we want to make transactional -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>
    <!-- the transactional advice (what 'happens'; see the <aop:advisor/> bean below) -->
    <tx:advice id="txAdvice" transaction-manager="txManager"> <!-- the transactional semantics... -->
        <tx:attributes>
            <!-- all methods starting with 'get' are read-only -->
            <tx:method name="get*" read-only="true"/>
            <!-- other methods use the default transaction settings (see below) -->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>
    <!-- ensure that the above transactional advice runs for any execution
        of an operation defined by the FooService interface -->
    <aop:config>
        <aop:pointcut id="fooServiceOperation" expression="execution(* x.y.service.FooService.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceOperation"/>
    </aop:config>
    <!-- don't forget the DataSource -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
        <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
        <property name="username" value="scott"/>
        <property name="password" value="tiger"/>
    </bean>
    <!-- similarly, don't forget the PlatformTransactionManager -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- other <bean/> definitions here -->
</beans>
{% endcodeblock %}

对于x.y.service.FooService包中的任何一个服务类的任何一个方法，我都需要它被txManager所管理，任何以get开头的方法都让它在只读事务的上下文中执行，其他的则在默认事务的上下文中执行。

Note：如果DataSourceTransactionManager的bean name，定义为transactionManager，则<tx:advice>中的transaction-manager不用指定。

####注解的方式

在真正的开发当中，除了这种xml的方式来指定事务的配置，通过注解的方式来配置事务相对更简单一些。
{% codeblock lang:java %}
@Transactional
public class DefaultFooService implements FooService {
	Foo getFoo(String fooName);

	Foo getFoo(String fooName, String barName);

	void insertFoo(Foo foo);

	void updateFoo(Foo foo);
}
{% endcodeblock %}

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx" xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!-- this is the service object that we want to make transactional -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>
    <!-- enable the configuration of transactional behavior based on annotations -->
    <tx:annotation-driven transaction-manager="txManager"/>
    <!-- a PlatformTransactionManager is still
     required -->
    <bean id="txManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager"> <!-- (this dependency is defined somewhere else) -->
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- other <bean/> definitions here -->
</beans>
{% endcodeblock %}

Note：同样的道理，如果DataSourceTransactionManager的bean name，定义为transactionManager，则<tx:annotation-driven>中的transaction-manager不用指定。

要真正启动事务管理，仅仅配置@Transactional是不够的，一定要配置<tx:annotation-driven>来启动事务管理行为。

Note：如果你使用Java的配置方式，只需要在@Configuration的类上添加@EnableTransactionManagement来启动事务管理。

Note：@EnableTransactionManagement和<tx:annotation-driven/>在查找@Transactional时，只会在它们定义位置的上下文中查找。意味着，如果你把它们放在了WebApplicationContext中，那么它们只会在Controller中，而不是Service中查找@Transactional。


参考资料：     
1.Spring Framework Reference Document, Transaction Management


