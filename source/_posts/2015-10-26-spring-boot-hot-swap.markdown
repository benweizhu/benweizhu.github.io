---
layout: post
title: "基于Gradle和Intellij的Spring Boot热交换"
date: 2015-10-26 12:19:43 +0800
comments: true
categories: 
---

在去年的一篇《Gradle Jetty和Gradle Watch插件实现热部署》中，谈到Java热部署带来的好处，文章地址在： http://benweizhu.github.io/blog/2014/07/27/gradle-jetty-plugin-hot-deploy/ 。

但是现在越来越多的Spring应用直接使用Spring Boot作为框架，那么这个Jetty插件的配置就不起作用了，好在Spring官方针对热部署问题，提供了解决方案：Spring Reloaded。

项目地址在： https://github.com/spring-projects/spring-loaded

这里，我就不多说废话了，直接告诉大家怎么用？

{% codeblock lang:groovy %}
buildscript {
    repositories { jcenter() }
    dependencies {
        classpath "org.springframework.boot:spring-boot-gradle-plugin:1.2.7.RELEASE"
        classpath 'org.springframework:springloaded:1.2.4.RELEASE'
    }
}

apply plugin: 'idea'

idea {
    module {
        inheritOutputDirs = false
        outputDir = file("$buildDir/classes/main/")
    }
}
{% endcodeblock %}

注：IntelliJ必须配置跟命令行Gradle任务相同的Java版本，并且springloaded必须作为一个buildscript依赖被包含进去。

官方文档的springloaded版本是1.2.0.RELEASE，这个版本有问题，会出现：

org.springsource.loaded.jvm.JVM : Problems copying method. Incompatible JVM? 报错

依赖下载完成之后，正常启动Spring Boot Run。

如果想将Spring Loaded和Gradle，IntelliJ结合起来，那你需要付出代价。默认情况下，IntelliJ将类编译到一个跟Gradle不同的位置，这会导致Spring Loaded监控失败，所以使用idea模块修改编译输出位置和Gradle一样。


{% codeblock lang:java %}
./gradlew bRun
{% endcodeblock %}

此时，如果你在应用启动的时候修改了Java代码，只需要点击Intellij的编译按钮，重新编译代码即可。

又或者，将我在上篇文章中介绍的Watch引入，监测文件变化，自动运行compileJava等Gradle命令，也是可以的。

参考文献：   
1.http://docs.spring.io/spring-boot/docs/current/reference/html/howto-hotswapping.html

