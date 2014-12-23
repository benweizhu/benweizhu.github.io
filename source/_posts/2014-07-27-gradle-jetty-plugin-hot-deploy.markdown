---
layout: post
title: "Gradle Jetty和Gradle Watch插件实现热部署"
date: 2014-07-27 14:53
comments: true
categories: 
---

##**Jetty插件**

Jetty插件提供两个重要方法：jettyRun和jettyRunWar。

jettyRun会将一个已暴露（解包的）的web应用部署到嵌入式Jetty Web容器中。它不需要将web应用打包成一个war文件，目的是为了节省部署时间。

jettyRunWar正好相反，是将一个War包部署到Web容器中。

**jettyRun的好处是，你可以改变静态文件和JSP文件，而不需要重新启动服务器。**

但是即便如此，对于日常开发还是不方便，因为开发过程中改动最多的其实是Java文件和资源配置文件，所以真正需要的是**热部署**。

jettyRun的Gradle API文档中有这么一句话：Once started, the web container can be configured to run continuously, scanning for changes in the project and automatically performing a hot redeploy when necessary. This allows the developer to concentrate on coding changes to the project using their IDE of choice and have those changes immediately and transparently reflected in the running web container, eliminating development time that is wasted on rebuilding, reassembling and redeploying.

这句话简单总结就是Jetty提供实现热部署的特性，开发人员只需要专注于编写代码，减少重新构建，重新组装和重新部署所浪费的时间。

但问题是，官方文档上写了这句话后，就不了了之了，没有说怎么做。我们都试过，默认配置是不会实现热部署的，那么应该怎么做呢？

###**两个属性：**

**reload**	The reload mode, which is either "automatic" or "manual".

**scanIntervalSeconds**	 The interval in seconds between scanning the web app for file changes. If file changes are detected, the web app is reloaded. Only relevant if reload is set to "automatic". Defaults to 0, which disables automatic reloading.

读完上面两段，说明默认scanIntervalSeconds的配置是不支持自动重新载入变化文件的。

试一把，把它改为支持：

{% codeblock lang:groovy %}
apply plugin: 'jetty'
apply plugin: 'idea'

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.springframework:spring-webmvc:4.0.6.RELEASE'
}

jettyRun {
    reload = "automatic"
    scanIntervalSeconds = 1
}
{% endcodeblock %}

然后运行gradle jettyRun启动jetty容器，修改Java类，然后去页面验证变化，结果是没有变化。为什么？

原来，jetty监听的是build目录下的class文件变化，而不是源代码变化，也就说源代码内容改变了，但class文件没有变化，那么不会自动触发jetty重载变化文件，那么该怎么办？最简单粗暴的解决方案就是另起一个命令行窗口，手动运行一次gradle compileJava命令。

没错，这个方法是行得通的。但仍然不是最好的解决方案。我查了下，到目前为止，官方没有给出正式的解决方案，但是该特性是在GradleWare的To-Do-List上的，预计以后应该会有。

##**Gradle Watch**

那么，唯一的办法只有借助第三方的插件来协助Jetty插件，一起实现热部署了，gradle-watch（日本人写的，因为上面的饿提交记录全是日文的）。 https://github.com/bluepapa32/gradle-watch-plugin

gradle watch的作用是监听某种类型的文件的变化，包括添加，删除和修改，然后执行预定义的任务。

使用起来很简单：

{% codeblock lang:groovy %}
apply plugin: 'jetty'
apply plugin: 'idea'

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.springframework:spring-webmvc:4.0.6.RELEASE'
}

jettyRun {
    reload = "automatic"
    scanIntervalSeconds = 1
}

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.bluepapa32:gradle-watch-plugin:0.1.3'
    }
}

apply plugin: 'watch'

watch {
    java {
        files files('src/main/java')
        tasks 'compileJava'
    }
}
{% endcodeblock %}

配置watch闭包，什么文件发生变化后就执行什么任务（好像它没有提供默认配置，所以需要手动显示的配置）。

在启动了gradle jettyRun之后，开启另一个窗口运行gradle watch。一次Java文件变化的输出如下：
{% codeblock lang:js %}

:watch  
Starting............ OK  

--------------------------------------------------------------------------------
Sun Jul 27 15:36:51 CST 2014
 
File "src/main/java/me/zeph/springmvc/jrebel/controller/HelloJRebelController.java" was changed.

-----------------------------------------------------
:compileJava

BUILD SUCCESSFUL

Total time: 0.954 secs  
Building > :watch
{% endcodeblock %}

可见，成功的更新了Java的class文件到build目录。刷新一次页面，就可以查看变化了。

那么对于资源文件呢？比如，我使用了Spring，需要改变Spring的Bean配置文件，同样可以。

{% codeblock lang:groovy %}

watch {
    java {
        files files('src/main/java')
        tasks 'compileJava'
    }
    resources {
        files fileTree(dir: 'src/main/resources', include: '**/*.xml')
        tasks 'processResources'
    }
}
{% endcodeblock %}

输出结果如下：

{% codeblock lang:js %}

--------------------------------------------------------------------------------
Sun Jul 27 15:56:39 CST 2014
 
File "src/main/resources/applicationContextService.xml" was changed.

--------------------------------------------------------------------------------
:processResources

BUILD SUCCESSFUL

Total time: 0.501 secs
{% endcodeblock %}

是不是很爽，这样是不是就再也不用重启服务器了，开发速度瞬间提升好几万战斗力。

加一句，该插件同样支持properties文件的改变，与XML一样配置。

好吧，就到这里，我觉得热部署也算的上某种自动化开发的一部分，至少他们的目的一样，提升开发效率，得到快速反馈，希望这篇文章对大家有所帮助。

参考资料：

1.http://forums.gradle.org/gradle/topics/hot_deploy_with_jetty_plugins_jettyrun

2.http://www.gradle.org/docs/current/dsl/org.gradle.api.plugins.jetty.JettyRun.html

3.https://github.com/bluepapa32/gradle-watch-plugin
