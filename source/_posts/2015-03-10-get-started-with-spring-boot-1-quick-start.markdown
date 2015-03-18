---
layout: post
title: "用Spring Boot开发Spring项目（一） - 快速上手"
date: 2015-03-10 16:44:18 +0800
comments: true
categories: 
---

Spring Boot是Spring团队提供的全新框架，其设计目的是用来简化新的Spring应用的初始搭建以及开发过程。

Spring Boot的主要目的是：（参考：https://github.com/spring-projects/spring-boot）

1.Provide a radically faster and widely accessible getting started experience for all Spring development    
2.Be opinionated out of the box, but get out of the way quickly as requirements start to diverge from the defaults    
3.Provide a range of non-functional features that are common to large classes of projects (e.g. embedded servers, security, metrics, health checks, externalized configuration)    
4.Absolutely no code generation and no requirement for XML configuration

快速启动Spring的开发，快速响应变化，提供非功能的特性，无需XML配置，这是Spring Boot所带来的优势。

下面通过一个例子，来快速的开发一个基于Spring Boot的Rest应用

{% codeblock build.gradle lang:groovy %}
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.2.2.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'idea'
apply plugin: 'spring-boot'

jar {
    baseName = 'gs-spring-boot'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web") {
        exclude module: "spring-boot-starter-tomcat"
    }
    compile("org.springframework.boot:spring-boot-starter-jetty")
    testCompile("junit:junit")
}

task wrapper(type: Wrapper) {
    gradleVersion = '1.11'
}
{% endcodeblock %}
在Gradle中，应用了Spring Boot Gradle插件，你可以看到dependencies中并没有指定Spring Boot依赖的版本，这是因为已经在插件的定义中已经指定了版本：1.2.2.RELEASE，所以这里就可以省略。（关于省略版本这一点，在之后的文章中会解释）。

而且，依赖spring-boot-starter-web会一站式的帮你将与web开发相关的所有相关依赖下载，而不需要你一个个复制粘贴。这也是所有其他Spring Boot Starter的主要目的。


{% codeblock HelloController.java lang:java %}
package me.zeph.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
	@RequestMapping(value = "/")
	public String hello() {
		return "hello spring boot";
	}
}
{% endcodeblock %}

这里定义了一个HelloController使用RestController注解，@RestController代表着@Controller和@ResponseBody，可以简化写Restful类型Controller的流程。

{% codeblock Application.java lang:java %}
package me.zeph;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
	public static void main(String args[]) {
		SpringApplication.run(Application.class, args);
	}
}
{% endcodeblock %}

最后一步，定义Spring boot应用的main方法，@SpringBootApplication等价于另外三个注解@Configuration，@EnableAutoConfiguration，@ComponentScan。

运行Gradle的命令: gradlew bootRun就可以启动应用程序。

通过Spring Boot创建的Java应用可以直接通过java -jar启动（即便它是Web应用）。也就是说，在这里，首先运行gradle assemble，得到jar文件，然后运行java -jar gs-spring-boot-0.1.0.jar。


参考资料：   
1.https://github.com/spring-projects/spring-boot   
2.http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started-first-application




