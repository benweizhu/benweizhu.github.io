---
layout: post
title: "Spring Boot 深入浅出系列（一） - 习惯使用注解"
date: 2015-06-10 22:19:19 +0800
comments: true
categories: 
---

Spring Boot从一开始就告诉你，它更喜欢基于Java的配置，即注解的方式。所以它提供了你一大堆注解，并让你习惯使用注解。

@Bean

Indicates that a method produces a bean to be managed by the Spring container.

@Configuration

Indicates that a class declares one or more @Bean methods and may be processed by the Spring container to generate bean definitions and service requests for those beans at runtime

@EnableAutoConfiguration

Enable auto-configuration of the Spring Application Context, attempting to guess and configure beans that you are likely to need. Auto-configuration classes are usually applied based on your classpath and what beans you have defined.

@ComponentScan

Configures component scanning directives for use with @Configuration classes. Provides support parallel with Spring XML's <context:component-scan> element.

##指定main application class的位置

SpringBoot建议你将主应用class（main application class）放在包根路径上，即其他子包之上。@EnableAutoConfiguration通常放在你的main class上，这样也隐含的指定了对某些配置项的搜索路径。比如，对@Entity的搜索。

在主应用class上指定@ComponentScan，同样也隐式的指定了扫描时basePackage的路径。

{% codeblock lang:java%}
@Configuration 
@EnableAutoConfiguration 
@ComponentScan
public class Application {
	public static void main(String[] args) { 
		SpringApplication.run(Application.class, args);
	} 
}
{% endcodeblock %}

如果你的main application class的位置确实在包的根路径上，上面的三个注解，可以用@SpringBootApplication这一个注解代替。

##多种方式加载Bean

你必然不会在main application class定义很多的@Bean，Spring提供两种方式将定义在另外一个带有@Configuration的类中的Bean加载，第一种，在Application类中使用@Import指定该类，第二种，让@ComponentScan扫描到该类。大部分情况都会选择第二种。

##加载XML的配置

如果你必须使用XML的配置，你可以使用@ImportResource来加载指定的XML配置。

##Bean的自动配置

SpringBoot有一个非常神秘的注解@EnableAutoConfiguration，官方的解释已经在上面的部分给出，简单点说就是它会根据定义在classpath下的类，自动的给你生成一些Bean，并加载到Spring的Context中。

它的神秘之处，不在于它能做什么，而在于它会生成什么样的Bean对于开发人员是不可预知（或者说不容易预知）。举个例子：

要开发一个基于Spring JPA的应用，会涉及到下面三个Bean的配置，DataSource，EntityManagerFactory，PlatformTransactionManager。

{% codeblock lang:java%}
@Configuration
@EnableJpaRepositories
@EnableTransactionManagement
public class ApplicationConfig {
	@Bean
	public DataSource dataSource() {
		...
	}

	@Bean
	public EntityManagerFactory entityManagerFactory() {
		..
		factory.setDataSource(dataSource());
		return factory.getObject();
	}

	@Bean
	public PlatformTransactionManager transactionManager() {
		JpaTransactionManager txManager = new JpaTransactionManager();
		txManager.setEntityManagerFactory(entityManagerFactory());
		return txManager;
	}
}
{% endcodeblock %}

@EnableJpaRepositories会查找满足作为Repository条件（继承父类或者使用注解）的类。

@EnableTransactionManagement的作用：Enables Spring's annotation-driven transaction management capability, similar to the support found in Spring's <tx:*> XML namespace。

但是，如果你使用了@EnableAutoConfiguration，那么上面三个Bean，你都不需要配置。在classpath下面只引入了MySQL的驱动和SpringJpa。

{% codeblock lang:java%}
compile 'mysql:mysql-connector-java:5.1.18'
compile 'org.springframework.boot:spring-boot-starter-data-jpa'
{% endcodeblock %}

在Application类中写下下面这段代码，可以查看SpringBoot给你生成了这些Bean：
{% codeblock lang:java%}
ConfigurableApplicationContext ctx = SpringApplication.run(Application.class, args);
System.out.println("Let's inspect the beans provided by Spring Boot:");

Object dataSource = ctx.getBean("dataSource");
Object transactionManager = ctx.getBean("transactionManager");
Object entityManagerFactory = ctx.getBean("entityManagerFactory");
System.out.println(dataSource);
System.out.println(entityManagerFactory);
System.out.println(transactionManager);
System.out.println(((JpaTransactionManager)transactionManager).getDataSource());
System.out.println(((JpaTransactionManager)transactionManager).getEntityManagerFactory());
{% endcodeblock %}

输出如下：
{% codeblock lang:java%}
org.apache.tomcat.jdbc.pool.DataSource@4f0e94db{ConnectionPool[defaultAutoCommit=null; defaultReadOnly=null; defaultTransactionIsolation=-1; defaultCatalog=null; driverClassName=com.mysql.jdbc.Driver; maxActive=100; maxIdle=100; minIdle=10; initialSize=10; maxWait=30000; testOnBorrow=true; testOnReturn=false; timeBetweenEvictionRunsMillis=5000; numTestsPerEvictionRun=0; minEvictableIdleTimeMillis=60000; testWhileIdle=false; testOnConnect=false; password=********; ... }

org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@5109d386

org.springframework.orm.jpa.JpaTransactionManager@5c1e2bfa

org.apache.tomcat.jdbc.pool.DataSource@4f0e94db{ConnectionPool[defaultAutoCommit=null; defaultReadOnly=null; defaultTransactionIsolation=-1; defaultCatalog=null; driverClassName=com.mysql.jdbc.Driver; maxActive=100; maxIdle=100; minIdle=10; initialSize=10; maxWait=30000; testOnBorrow=true; testOnReturn=false; timeBetweenEvictionRunsMillis=5000; numTestsPerEvictionRun=0; minEvictableIdleTimeMillis=60000; testWhileIdle=false; testOnConnect=false; password=********;... }

org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@5109d386
{% endcodeblock %}
Bean中的URL，username和password是在属性文件中配置的：

{% codeblock lang:java%}
#Database     
spring.datasource.url=jdbc:mysql://localhost:3306/xxxx      
spring.datasource.username=root    
spring.datasource.password=    
{% endcodeblock %}

##Disable自动配置

如果你发现自动转配的Bean不是你想要的，你也可以disable它。比如说，我不想要自动装配Database的那些Bean

{% codeblock lang:java%}
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})
public class MyConfiguration {
	
}
{% endcodeblock %}
此时，就会报下面的错了
{% codeblock lang:java%}
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type [javax.sql.DataSource] found for dependency
{% endcodeblock %}

习惯使用和正确使用上面这些注解，是正确使用Spring Boot的重要起步。


参考资料：   
1.Spring Boot Reference   
2.Spring JPA Reference   


