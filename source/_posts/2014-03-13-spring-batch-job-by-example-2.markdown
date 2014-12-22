---
layout: post
title: "一起学学Spring Batch（二）"
date: 2014-03-13 14:31
comments: true
categories: 
---
所有的Batch处理都可以以一种最简单的方式描述，那就是**读取大量的数据，然后进行某种方式的计算或者变形，最后输出写入到某个存储系统中**。Spring Batch提供了三个简单的接口来帮助执行大量数据的读写，分别是**ItemReader、ItmeWriter和ItermProcessor**，你可以说它们是抽象出来的一种简单概念，它们同样也是针对不同类型数据的输入、输出和处理的方法。

我们最常用到的数据类型有三种：**数据库、XML和Flat文件**。为了能够清晰的描述Batch处理的流程，我们采用Flat这种文件作为输入和输出数据源，排除数据库操作和XML文件解析的带来的理解上的复杂度。

“Flat文件是一种包含没有相对关系结构的记录的文件。这个类型通常用来描述文字处理、其他结构字符或标记被移除了的文本。在任何事件中，许多用户把保存成“纯文本(text only)”类型的Microsoft Word文档叫做“Flat File(flat file)”。Flat File的另一种形式是包含了用ASCII码记录的，每个表单元由逗号分隔，用行表示记录组的文件。这种文件页叫做用逗号分隔数值(CSV)的文件.”

**这里给出一个例子**，从一个csv格式的flat文件中读取英语、数学、文学的分数，然后计算平均分，将结果写入另一个csv格式文件。

首先是两个Model，分别是输入的数据模型ExamScore和输出的数据模型AverageScore。

{% codeblock ExamScore.java lang:java %}
package me.zeph.spring.batch.model;

public class ExamScore {
	private int id;
	private int english;
	private int math;
	private int literature;

	public int getId() {
		return id;
	}
	
	public void setId(int id) {
		this.id = id;
	}

	public int getEnglish() {
		return english;
	}

	public void setEnglish(int english) {
		this.english = english;
	}

	public int getMath() {
		return math;
	}

	public void setMath(int math) {
		this.math = math;
	}

	public int getLiterature() {
		return literature;
	}
	
	public void setLiterature(int literature) {
		this.literature = literature;
	}
	
}
{% endcodeblock %}

{% codeblock AverageScore.java lang:java %}
package me.zeph.spring.batch.model;

public class AverageScore {

	private int id;
	private double score;

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public double getScore() {
		return score;
	}

	public void setScore(double score) {
		this.score = score;
	}

}
{% endcodeblock %}

然后是Processor，这里实现ItemProcessor接口。在process方法中传入输入数据模型，返回输出数据模型。

{% codeblock ScoreProcessor.java lang:java %}
package me.zeph.spring.batch.processor;

import me.zeph.spring.batch.model.AverageScore;
import me.zeph.spring.batch.model.ExamScore;
import org.springframework.batch.item.ItemProcessor;

public class ScoreProcessor implements ItemProcessor<ExamScore, AverageScore> {

	public static final double COUNT = 3.0;

	@Override
	public AverageScore process(ExamScore examScore) throws Exception {
		AverageScore averageScore = new AverageScore();
		averageScore.setId(examScore.getId());
		averageScore.setScore(calculateAverageScore(examScore));
		return averageScore;
	}

	private double calculateAverageScore(ExamScore examScore) {
		return (examScore.getEnglish() + examScore.getLiterature() + examScore.getMath()) / COUNT;
	}
}
{% endcodeblock %}

和上篇文章的一样，在spring-batch-beans.xml中定义了关于Sping Batch的一些基础Bean。

{% codeblock spring-batch-beans.xml lang:xml %}
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">

    <bean id="transactionManager" class="org.springframework.batch.support.transaction.ResourcelessTransactionManager"/>

    <bean id="jobRepository" class="org.springframework.batch.core.repository.support.MapJobRepositoryFactoryBean">
        <property name="transactionManager" ref="transactionManager"/>
    </bean>

    <bean id="jobLauncher" class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
        <property name="jobRepository" ref="jobRepository"/>
    </bean>

</beans>
{% endcodeblock %}

重点是下面一个xml文件spring-batch-jobs.xml，在这个里面定义所有关于Batch Job的具体内容。

{% codeblock spring-batch-jobs.xml lang:xml %}
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:batch="http://www.springframework.org/schema/batch"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
           http://www.springframework.org/schema/batch
           http://www.springframework.org/schema/batch/spring-batch-2.2.xsd">

    <import resource="spring-batch-beans.xml"/>

    <bean id="csvFileReader" class="org.springframework.batch.item.file.FlatFileItemReader">
        <property name="resource" value="file:/Users/twer/Documents/gradle-project/spring-batch-example/src/main/resources/reader_file.csv"/>
        <property name="lineMapper" ref="lineMapper"/>
    </bean>

    <bean id="lineMapper" class="org.springframework.batch.item.file.mapping.DefaultLineMapper">
        <property name="lineTokenizer" ref="lineTokenizer"/>
        <property name="fieldSetMapper" ref="fieldSetMapper"/>
    </bean>

    <bean id="lineTokenizer" class="org.springframework.batch.item.file.transform.DelimitedLineTokenizer">
        <property name="delimiter" value=","/>
        <property name="names" value="id,english,math,literature"/>
    </bean>

    <bean id="fieldSetMapper" class="org.springframework.batch.item.file.mapping.BeanWrapperFieldSetMapper">
        <property name="targetType" value="me.zeph.spring.batch.model.ExamScore"/>
    </bean>

    <bean id="csvFileProcessor" class="me.zeph.spring.batch.processor.ScoreProcessor"/>

    <bean id="csvFileWriter" class="org.springframework.batch.item.file.FlatFileItemWriter">
        <property name="resource" value="file:/Users/twer/Documents/gradle-project/spring-batch-example/src/main/resources/writer_file.csv"/>
        <property name="lineAggregator" ref="lineAggregator"/>
    </bean>

    <bean id="lineAggregator" class="org.springframework.batch.item.file.transform.DelimitedLineAggregator">
        <property name="delimiter" value=","/>
        <property name="fieldExtractor" ref="fieldExtractor"/>
    </bean>

    <bean id="fieldExtractor" class="org.springframework.batch.item.file.transform.BeanWrapperFieldExtractor">
        <property name="names" value="id,score"/>
    </bean>

    <batch:job id="fileTransfer">
        <batch:step id="step">
            <batch:tasklet>
                <batch:chunk reader="csvFileReader" processor="csvFileProcessor" writer="csvFileWriter" commit-interval="1"/>
            </batch:tasklet>
        </batch:step>
    </batch:job>

</beans>
{% endcodeblock %}

让我们来一起分析一下上面这个xml配置。

**FlatFileItemReader**是Flat文件的读取类，可以按行读取输入文件。这里传入两个参数，LineMapper和Resource。

Resource顾名思义，用来指定数据源。

LineMapper用于将文件中的每行数据映射到对应的数据模型，这里使用DefaultLineMapper。LineMapper中传入两个参数，分别是行分词器LineTokenizer和域映射器FieldSetMapper。

对应的LineTokenizer采用基于分隔符的分词器，而对应的FieldSetMapper需要传入待映射的数据模型类型。

对于**FlatFileItemWriter**，同样要配置对应的输出源Resource和行聚合器lineAggregator。

对于lineAggregator这里同样采用基于分隔符的聚合器DelimitedLineAggregator，需要给聚合器配置对应的分隔符（比如“,”）和域提取器BeanWrapperFieldExtractor。

在域提取器BeanWrapperFieldExtractor中配置要被提取的值。

最后就是定义具体的Batch Job。

{% codeblock spring-batch-jobs.xml lang:xml %}
<batch:job id="fileTransfer">
        <batch:step id="step">
            <batch:tasklet>
                <batch:chunk reader="csvFileReader" processor="csvFileProcessor" writer="csvFileWriter" commit-interval="1"/>
            </batch:tasklet>
        </batch:step>
</batch:job>
{% endcodeblock %}
运行结果：上面是输入，下面是输出

	1,100,80,80
	2,100,70,80
	3,100,80,70

	1,86.66666666666667
	2,83.33333333333333
	3,83.33333333333333
