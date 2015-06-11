---
layout: post
title: "Spring Boot 深入浅出系列（二） - 用Gradle启动应用"
date: 2015-06-11 12:56:12 +0800
comments: true
categories: 
---
在前面介绍的《用Spring Boot开发Spring项目 快速上手》上介绍过：

“通过Spring Boot创建的Java应用可以直接通过java -jar启动（即便它是Web应用）。也就是说，在这里，首先运行gradle assemble，得到jar文件，然后运行java -jar gs-spring-boot-0.1.0.jar。”

但是，我们肯定不能每次这样去启动SpringBoot的应用。好在是，官方提供了与构建相关的插件Spring Boot Gradle plugin，插件中提供了对应的task给你使用：

Application tasks     
-----------------     
bootRun - Run the project with support for auto-detecting main class and reloading static resources.    
distTar - Bundles the project as a JVM application with libs and OS specific scripts.    
distZip - Bundles the project as a JVM application with libs and OS specific scripts.    
installApp - Installs the project as a JVM application along with libs and OS specific scripts.   
run - Runs this project as a JVM application.

##在构建中引入Spring Boot Gradle plugin

首先，你需要在构建中加入该插件：

{% codeblock lang:groovy %}
buildscript {
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.2.4.RELEASE")
    }
}
apply plugin: 'spring-boot'
{% endcodeblock %}

##省略依赖的版本号

SpringBoot插件会注册一个定制的依赖解析策略，允许你省略对依赖版本的配置。

{% codeblock lang:groovy %}
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.thymeleaf:thymeleaf-spring4")
    compile("nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect")
}
{% endcodeblock %}

那么问题来了，这个省略的版本号从哪里来呢？它有Spring Boot Plugin的版本号决定，比如，当前定义的插件是org.springframework.boot:spring-boot-gradle-plugin:1.2.4.RELEASE，版本号是1.2.4，那么对应的spring-boot-starter-web的版本号，也是1.2.4。

当然，如果你想显示的指定版本号也是可以的：

{% codeblock lang:groovy %}
dependencies {
    compile("org.thymeleaf:thymeleaf-spring4:2.1.1.RELEASE")
}
{% endcodeblock %}

##打包可执行的jar包或者war包

一旦使用了Spring boot插件，它就会用bootRepackage任务改写archive的过程。

你可以在配置选项中指定main class，或者在Manifest添加Main-Class，如果你不指定，它会去搜索含有public static void main(String[] args)方法的类。

{% codeblock lang:groovy %}
bootRepackage {
    mainClass = 'demo.Application'
}
{% endcodeblock %}

如果你想要打成War包，并部署到外部容器里面，除了要使用war插件，还需要将embedded container的依赖放在providedRuntime里。如下：

{% codeblock lang:groovy %}
...
apply plugin: 'war'

war {
    baseName = 'myapp'
    version =  '0.5.0'
}

repositories {
    jcenter()
    maven { url "http://repo.spring.io/libs-snapshot" }
}

configurations {
    providedRuntime
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    providedRuntime("org.springframework.boot:spring-boot-starter-tomcat")
    ...
}
{% endcodeblock %}

参考资料：   
1.http://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins-gradle-plugin.html