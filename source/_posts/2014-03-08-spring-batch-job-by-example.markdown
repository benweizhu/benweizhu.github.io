---
layout: post
title: "一起学学Spring Batch（一）"
date: 2014-03-08 17:52
comments: true
categories: 
---

批处理应用（Batch Application）主要是在没有人工干预的情况下处理大量数据。比如，不同系统之间数据文件的导入和导出，数据的计算，定期生成财务报告等等。

Spring Batch是一个轻量级的，复杂的Batch框架，能够让你开发针对企业级系统日常操作的Batch应用。

**Spring Batch提供的有用特性：**

1.	Spring框架中的基础，依赖注入，切面编程，企业级别的支持
2.	面向批处理的运行时，有效的驱动批处理应用的流程
3.	有效处理数据，以最佳策略读写数据
4.	可用的现成组件，提供能够定位到不同批处理场景的组件

Spring框架以轻量级容器闻名，提供配置方式来完成应用程序组件的组装。

Spring Batch对Spring框架进行了扩展，提供了专用的xml的命名空间来配置批处理的过程。 

**有效处理数据**

以一种经典场景为例：从一个数据系统读取数据存储到另一个数据系统中。如果一次将整个数据系统中的数据读入到内存，JVM随时可能Out of Memory。当然将所有数据读入到内存肯定不是最好的办法。

Spring Batch使用一种更加有效的办法，叫做Chunk Processing（块处理方式）：以数据流的方式读取输入的源数据，并处理一定数量（这一块）的记录，并写入到对应的组件。

这个块的大小可以改变，所以仍然可以让批处理一条一条的处理数据。

**Ready-to-use Component**

Spring Batch提供了基础设施来执行块处理，并代理I/O到专用的组件，并命名为reader和writer。

Spring为一些通用的批处理场景提供可用的组件，这些组件可以让你更加专注于业务逻辑。

Spring Batch对reader和writer场景的支持技术。

数据类型	 | 技术格式
-------  | ------------- 
Database | JDBC
Database | Hibernate
Database | JPA
Database | iBatis
File | Flatfile
File | XML

当然Spring Batch支持的不止这些技术。当然如果没有你需要的实现，你也可以自己是实现对应的组件。

**Spring Batch不是Scheduler**

Spring Batch可以驱动批处理流程但是不会提供启动它们的支持，特别是基于时间的启动。Spring Batch一般会讲这些工作代理给其他Scheduler工具，例如Quartz或者Cron。

**下面是一个Spring Batch Job的Hello World Example**
{% codeblock build.gradle lang:groovy %}
apply plugin: 'java'
apply plugin: 'idea'

repositories {
 mavenCentral()
}

dependencies {
 compile 'org.springframework.batch:spring-batch-core:2.2.5.RELEASE'
 compile 'org.springframework:spring-context:3.2.8.RELEASE'
 testCompile group: 'junit', name: 'junit', version: '4.+'
}
{% endcodeblock %}

在这里定义一个具体的task。
{% codeblock tasklet.java lang:java %}
package me.zeph.spring.batch.tasklet;

import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;

import static org.springframework.batch.repeat.RepeatStatus.FINISHED;

public class HelloTasklet implements Tasklet {

   @Override
	public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
		System.out.println("Hello Spring Batch!");
		return FINISHED;
	}

}
{% endcodeblock %}

{% codeblock beans.xml lang:xml %}
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:batch="http://www.springframework.org/schema/batch"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
           http://www.springframework.org/schema/batch
           http://www.springframework.org/schema/batch/spring-batch-2.2.xsd">

   <bean id="transactionManager" class="org.springframework.batch.support.transaction.ResourcelessTransactionManager"/>

   <bean id="jobRepository" class="org.springframework.batch.core.repository.support.MapJobRepositoryFactoryBean">
        <property name="transactionManager" ref="transactionManager"/>
   </bean>

   <bean id="jobLauncher" class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
        <property name="jobRepository" ref="jobRepository"/>
   </bean>   
</beans>
{% endcodeblock %}

在xml中配置一个具体batch job，这里只有一步，那就是执行上面定义的task。
{% codeblock batch-job.xml lang:xml %}
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:batch="http://www.springframework.org/schema/batch"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
           http://www.springframework.org/schema/batch
           http://www.springframework.org/schema/batch/spring-batch-2.2.xsd">

   <import resource="spring-batch-beans.xml"/>

   <bean id="helloTasklet" class="me.zeph.spring.batch.tasklet.HelloTasklet"/>

   <batch:job id="helloJob">
        <batch:step id="helloStep">
            <batch:tasklet ref="helloTasklet"/>
        </batch:step>
   </batch:job>

</beans>{% endcodeblock %}

在main函数中，通过JobLauncher来运行一个Job。
{% codeblock main.java lang:java %}
package me.zeph.spring.batch.tasklet;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TaskletRunner {
	public static void main(String args[]) throws Exception {
		String[] configLocations = {"spring-batch-beans.xml", "spring-batch-jobs.xml"};
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(configLocations);
		JobLauncher jobLauncher = applicationContext.getBean(JobLauncher.class);
		Job job = applicationContext.getBean(Job.class);
		JobExecution jobExecution = jobLauncher.run(job, new JobParameters());
		System.out.println("JOB EXECUTION STATUS:" + jobExecution.getExitStatus().getExitCode());
	}
}
{% endcodeblock %}



