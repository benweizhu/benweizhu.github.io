---
layout: post
title: "Spring Validation 深入浅出"
date: 2014-07-19 19:12
comments: true
categories: 
---
在Web开发中，一个重要的特性，那就是对表单提交的内容进行合法性校验。

校验的方式根据方式的不同，分为两类：前端校验和后端校验。

前端校验的优点是，不用与服务器交互，由JavaScript直接对页面元素中的内容进行校验，速度更快。但这种方式的安全性不高，对于了解web开发的人可以通过修改页面内容的方式绕过前端验证。

后端验证需要表单提交，一般情况下会刷新页面，对于用户体验不是太理想。

所以，一般前端验证会和后端验证会配合在一起。

今天我们一起来学习一下，在Spring MVC中，是如何进行后端的校验，它又是如何通过Spring提供的Form标签，将错误警告信息传递到前端来显示。

##**基本了解**

我们先来简单的写一个demo，这种HelloWorld式例子最容易让人有信心去理解和学习，要实现它，你只需要了解下面五样或者四样东西。

**一个重要的包：**

Spring框架中有一个非常重要的包，org.springframework.validation，它提供了两个重要的特性：数据绑定和校验。

数据绑定允许用户的输入动态的绑定到应用的领域模型，实现字符串类型和其他类型的转换。

在Spring MVC，数据绑定机制允许你使用任何的命令对象或者表单对象-你不需要实现一个框架指定的接口或者类。

**一个重要接口：**

Errors：用于存储和暴露于某个对象相关的数据绑定和错误校验信息。

**一个重要的类：**

BeanPropertyBindingResult：Spring中，Errors和BindingResult接口的默认实现类，主要给JavaBean对象的绑定错误进行注册和评估。

**一个重要的Form标签：**

<form:errors>:这个标签会将对象field的错误渲染成一个HTML的span标签。它能够访问由Controller或者与Controller相关的的Validator创建出来的errors对象。（关于Validator，后面会介绍）

**两个重要的函数：**

那么，要在Controller中实现一个最简单的校验，并将校验结果显示在前端就非常简单了，你只需要在了解Errors接口的两个函数：

reject():使用给定的错误描述信息，给这个目标对象注册一个全局的错误信息

rejectValue()：使用给定的错误描述信息，注册一个field的错误到当前对象的某个指定field上（或者该对象的某个成员变量的field上，比如customer.name.firstName）。这个field名字也可以是null或者空串来说明是指定该对象而不是该对象的一个field（不过这样容易导致一些错误）。

看下面的例子：

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc
                    http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd
                    http://www.springframework.org/schema/beans
                    http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
                    http://www.springframework.org/schema/context
                    http://www.springframework.org/schema/context/spring-context-3.1.xsd">

    <context:component-scan base-package="me.zeph.springview.demo"/>

    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/view/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- bind your messages.properties -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="messages" />
    </bean>

    <mvc:annotation-driven/>

</beans>
{% endcodeblock %}
这里唯一特殊的一点就是配置了一个ResourceBundle，它指向一个messages.properties文件，里面存放这校验错误码对应的信息。

{% codeblock lang:html %}
NotEmpty.customer.name=Name is required
NotEmpty.customer.password=Password is required
{% endcodeblock %}

{% codeblock lang:java %}
package me.zeph.springview.demo.controller;

import me.zeph.springview.demo.domain.Customer;
import org.springframework.stereotype.Controller;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.RequestMapping;

import static org.springframework.util.StringUtils.isEmpty;
import static org.springframework.web.bind.annotation.RequestMethod.GET;
import static org.springframework.web.bind.annotation.RequestMethod.POST;

@Controller
public class ErrorController {
	public static final String ERROR = "error";
	public static final String ERROR_SUCCESS = "error-success";

	@RequestMapping(value = "error", method = GET)
	public String view(Customer customer) {
		return ERROR;
	}

	@RequestMapping(value = "error", method = POST)
	public String commit(Customer customer, Errors errors) {
		if (isEmpty(customer.getName())) {
			errors.rejectValue("name", "NotEmpty.customer.name", "name is empty");
		}
		if (isEmpty(customer.getPassword())) {
			errors.rejectValue("password", "NotEmpty.customer.password", "password is empty");
		}
		//Just for demonstrating reject the object itself
		errors.reject("customer.invalid", "Customer is invalid");
		return errors.hasErrors() ? ERROR : ERROR_SUCCESS;
	}
}
{% endcodeblock %}
在Controller里面，通过参数让Spring帮我们注入Errors对象（注意Errors作为参数的位置一定要在领域对象（表单对象）的后面，猜测原因是Spring需要知道Errors对象是与哪个对象绑定的）。这里自己写校验逻辑，然后调用errors的rejectValue方法，将错误信息保存起来。如果你在这里打断点，你会看到，Errors真正的类型是BeanPropertyBindingResult。

{% codeblock lang:html %}
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Spring MVC View Demo</title>
</head>
<body>
<h1>Spring MVC View Demo</h1>
<form:form commandName="customer" method="post" action="error">
	<form:errors path="*"/>
    <div>
        <label for="name">name :</label>
        <form:input path="name" id="name"/> 

    </div>
    <div>
        <label for="password">password :</label>
        <form:password path="password" id="password"/>
    </div>
    <input type="submit" value="submit">
</form:form>
</body>
</html>
{% endcodeblock %}

**path="*" - 显示全部的内容**     
**path="name" - 显示与name这个field相关的错误信息**  
**如果path不填 - 只显示与表单对象本身相关的错误信息**  

到这里，你就可以去试一试，看能不能正常显示出你想要的信息。但是是不是到这里就为止了呢？远远没有。这只是Spring Validation最根本和最简单的实现，这种方式并不被Spring推荐，我们继续。

##**JSR-303**

在Spring 3.0之后，Spring Validation的功能增强了，支持结合使用JSR-303标准进行校验。

先讲讲什么是JSR-303，它是Java标准中Bean Validation的1.0版本。

它可以让你通过注解的方式来对对象模型添加限制，它提供的开箱即用的常用限制注解，同时也允许你写自己的限制规则。

不过就像之前学习的，它只是标准，具体实现由第三方提供，默认的参考实现是Hibernate Validator。

结合JSR-303，对表单的对象的校验就是成为了简单的给领域对象添加限定规则的注解。

来看看怎么写

首先给领域对象的field添加你需要的注解

{% codeblock lang:java %}
package me.zeph.springview.demo.domain;


import org.hibernate.validator.constraints.NotEmpty;

public class User {

	@NotEmpty
	private String name;
	@NotEmpty
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
}
{% endcodeblock %}

接下来，Controller就简单了，没有写任何校验逻辑。

{% codeblock lang:java %}
package me.zeph.springview.demo.controller;

import me.zeph.springview.demo.domain.User;
import org.springframework.stereotype.Controller;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.validation.Valid;

import static org.springframework.web.bind.annotation.RequestMethod.GET;
import static org.springframework.web.bind.annotation.RequestMethod.POST;

@Controller
public class ValidController {

	public static final String VALID = "valid";
	public static final String VALID_SUCCESS = "valid-success";

	@RequestMapping(value = "valid", method = GET)
	public String view(User user) {
		return VALID;
	}

	@RequestMapping(value = "valid", method = POST)
	public String commit(@Valid User user, Errors errors) {
		return errors.hasErrors() ? VALID : VALID_SUCCESS;
	}
}
{% endcodeblock %}

看下面的错误码和消息的写法，你应该知道JSR-303生成错误码的规则。

{% codeblock lang:java %}
NotEmpty.user.name=Name is required
NotEmpty.user.password=Password is required
{% endcodeblock %}

如果你觉得它提供的注解不够你用，你还可以写自己的注解，具体的内容就不在这里谈了，这是JSR-303的内容。如果你想了解，可以参考Hibernate Validation的指南： http://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-customconstraints

还没讲完呢，继续深入。

##**深入理解**

{% codeblock lang:java %}
public String commit(@Valid User user, Errors errors)
{% endcodeblock %}
看到没，在我们的领域对象User上有一个注解@Valid，它是JSR-303标准API提供的。

在Spring3.0之后，Spring MVC提供了一种能力可以让Controller输入参数进行自动的校验，之前后需要我们自己去触发校验逻辑。方法就是在输入参数加一个@Valid注解。

关于Spring Validation与JSR-303的结合使用，暂时到这里，我们再来看点别的。如果你也在学习Spring的Validation，那么你肯定会看到Spring的文档的第六章（ http://docs.spring.io/spring/docs/current/spring-framework-reference/html/validation.html ）。

它没有像我这样讲解Errors对象，讲解form怎么显示错误信息，而是介绍了一个Validator接口。那它是什么呢？

如果你有看它的API的解释，它会告诉你，它只是一个单纯的接口，期待将校验逻辑与web层，数据访问层解耦，它提供两个方法supports和validate。

根据它在指南中给的信息，我写出来下面的这段代码。

{% codeblock lang:java %}
package me.zeph.springview.demo.controller;

import me.zeph.springview.demo.domain.Account;
import me.zeph.springview.demo.validator.AccountValidator;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.validation.Valid;

import static org.springframework.web.bind.annotation.RequestMethod.GET;
import static org.springframework.web.bind.annotation.RequestMethod.POST;

@Controller
public class ValidatorController {

	@Autowired
	private AccountValidator accountValidator;

	public static final String VALIDATOR = "validator";
	public static final String VALIDATOR_SUCCESS = "validator-success";

	@RequestMapping(value = "validator", method = GET)
	public String view(Account account) {
		return VALIDATOR;
	}

	@RequestMapping(value = "validator", method = POST)
	public String commit(Account account, Errors errors) {
		accountValidator.validate(account, errors);
		return errors.hasErrors() ? VALIDATOR : VALIDATOR_SUCCESS;
	}
}
{% endcodeblock %}

{% codeblock lang:java %}
package me.zeph.springview.demo.validator;

import me.zeph.springview.demo.domain.Account;
import org.springframework.stereotype.Component;
import org.springframework.validation.Errors;
import org.springframework.validation.ValidationUtils;
import org.springframework.validation.Validator;

@Component
public class AccountValidator implements Validator {
	@Override
	public boolean supports(Class<?> clazz) {
		return Account.class.equals(clazz);
	}

	@Override
	public void validate(Object target, Errors errors) {
		ValidationUtils.rejectIfEmpty(errors, "name", "NotEmpty.account.name");
		ValidationUtils.rejectIfEmpty(errors, "password", "NotEmpty.account.password");
	}
}
{% endcodeblock %}

和我写的第一个例子比较，是不是很简洁，将校验逻辑和Controller分离。

等等，你这只是分了一个Service层来专门做校验，你在Controller层还是手动触发了校验逻辑，哪里像Validator API中说的那么理想，完全解耦了。

那是因为我写错了，正确的写法应该是这样的。

{% codeblock lang:java %}
package me.zeph.springview.demo.controller;

import me.zeph.springview.demo.domain.Account;
import me.zeph.springview.demo.validator.AccountValidator;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.validation.Errors;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.validation.Valid;

import static org.springframework.web.bind.annotation.RequestMethod.GET;
import static org.springframework.web.bind.annotation.RequestMethod.POST;

@Controller
public class ValidatorController {

	@Autowired
	private AccountValidator accountValidator;

	public static final String VALIDATOR = "validator";
	public static final String VALIDATOR_SUCCESS = "validator-success";

	@InitBinder
	protected void initBinder(WebDataBinder binder) {
		binder.setValidator(accountValidator);
	}

	@RequestMapping(value = "validator", method = GET)
	public String view(Account account) {
		return VALIDATOR;
	}

	@RequestMapping(value = "validator", method = POST)
	public String commit(@Valid Account account, Errors errors) {
		return errors.hasErrors() ? VALIDATOR : VALIDATOR_SUCCESS;
	}
}
{% endcodeblock %}

上面的代码说明了两件事情：

1.我在Account的前面加了@Valid，导致自动触发校验过程  
2.校验逻辑是由我写的Validator实现

你先不要去想@InitBinder是干什么，简单的说，它是Controller中的一个回调函数，Controller容器会在Controller的生命周期的某个阶段调用它。

WebDataBinder到是可以看看，它是web请求参数转换为JavaBean对象时，使用的特殊的数据绑定（DataBinder）对象。因为我们这里不涉及到数据绑定的内容，所以不细聊，但是你可以想象，该数据绑定的过程，必然在进入Controller的某个请求方法之前完成的（因为Spring必须在给我们注入领域对象啊）。

###**回头想一个问题**

好了，如果这里你想清楚了，可以再回头想一个问题，前面使用JSR-303方式定义校验，怎么没有定义一个Validator呢？

原因在这个对象：LocalValidatorFactoryBean，它是Spring为支持JSR-303标准专门实现的一个类，为了同时支持Spring的Validation机制和JSR-303，它同时实现了Spring的Validator接口和JSR-303中Validator接口（这两个不一样哦）。

那么，如果我要使用JSR-303标准，需要显示的在容器中声明这个bean吗？需要也不需要。

在你的xxx-servlet.xml中定义的<mvc:annotation-driven/>标签会帮你声明。

所以如果你在initBinder方法那打个断点，你可以看到WebDataBinder中的Validator对象就是LocalValidatorFactoryBean。

现在是不是比较清楚了？

##**再讲一个重要的东西**

还需要再讲一个东西：MessageCodesResolver和DefaultMessageCodesResolver

MessageCodesResolver是从Validation的ErrorCode映射到MessageCode的策略接口，被DataBinder使用，给ObjectErrors和FieldErrors构建MessageCode列表。

DefaultMessageCodesResolver是MessageCodesResolver提供的一个默认实现。

策略是：

针对object的错误，会去找两种错误信息码：

1.code + "." + object name  
2.code

针对field的错误，会去找三种错误信息码：

1.code + "." + object name + "." + field  
2.code + "." + field  
3.code + "." + field type  
4.code

举例来说：

{% codeblock lang:java %}
//field, error code, default message
errors.rejectValue("name", "NotEmpty", "name is empty");
errors.rejectValue("password", "NotEmpty", "password is empty");
errors.reject("Invalid", "Customer is invalid");

//messages.properties
NotEmpty=Name is required
NotEmpty.password=Password is required
Invalid=Customer is invalid
{% endcodeblock %}

所以，如果你回过头去，看我写的第一个例子，实际上，它是使用的最后一个策略。

好了，关于Spring Validation的基本内容到此为止，应该比较清楚了，如果你要正确使用它，应该没有任何问题。

其实，我很早就想写这篇文章，一直拖到在项目上遇到问题，才促使我完成，实在惭愧。希望这篇文章对大家有所帮助


参考资料：

1.http://docs.spring.io/spring/docs/current/spring-framework-reference/html/validation.html

2.http://docs.spring.io/spring/docs/current/spring-framework-reference/html/view.html

3.http://docs.spring.io/spring/docs/2.5.x/api/org/springframework/validation/package-summary.html

4.http://docs.huihoo.com/spring/3.0.x/en-us/ch05s07.html


