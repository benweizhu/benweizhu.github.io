---
layout: post
title: "Gradle深入与实战（五）自定义插件"
date: 2015-03-15 14:18:10 +0800
comments: true
categories: 
---
利用Gradle做构建，必然逃不掉Gradle的插件的使用，即便是最简单的Java或Groovy的应用都需要使用Java插件或者Groovy插件。

Gradle插件的作用就是将会被重复利用的逻辑打包，这样就可以在不同的项目中重复的使用。比如在上一节中实现的集成测试任务，就可以打包到插件中，然后在其他的工程中使用，而不需要重复的写相同的任务。

**Gradle提供了三种写插件的方式：**

1.直接在build.gradle文件中写插件，然后直接使用，不好的地方很明显，不能在该build.gradle脚本之外的位置（其他脚本或者工程）中使用。

2.将插件写在项目的rootProjectDir/buildSrc/src/main/groovy包下，Gradle会负责编译和放置到classpath，虽然可以多个gradle脚本中使用，但是不能在其他工程中使用。

3.一个独立的插件工程，很明显，这是最常见的实现方式，因为可以被任何脚本或者工程使用。

为了节省时间，我们直接进入到最常见的实现方式：**实现一个独立的插件工程**。

##这是一个Groovy工程
我们知道，Gradle项目是基于Groovy语言开发的，所以插件功能必然是一个Groovy工程，构建创建一个build.gradle文件。

{% codeblock build.gradle lang:groovy %}
apply plugin: 'idea'
apply plugin: 'groovy'

dependencies {
    compile gradleApi()
    compile localGroovy()
}

task wrapper(type: Wrapper) {
    gradleVersion = '1.11'
}
{% endcodeblock %}

按照Groovy插件推荐的目录结构建立好下面结构的目录，你可以忽略java那一级

src/main/java	  	
src/main/resources     
src/main/groovy	   
src/test/java	  
src/test/resources   		
src/test/groovy	

##利用Plugin接口实现插件

然后，我们写一个Groovy类，让它实现Plugin接口，如下：

{% codeblock lang:groovy %}
package me.zeph.gradle.plugin

import me.zeph.gradle.extension.HelloExtension
import org.gradle.api.Plugin
import org.gradle.api.Project

class HelloPlugin implements Plugin<Project> {

    @Override
    void apply(Project project) {
        project.task('hello') << {
            println 'hello plugin'
        }
    }
}
{% endcodeblock %}

这里，我们通过project的task方法实现了一个名字是hello的task，里面打印了一句话。这个任务很简单，现在我们来增加一点点复杂度。记不记得大部分插件在使用之后，除了提供一些列的task，还提供了许多的closure（闭包），可以通过这些闭包传递一些参数进去。那么，这是怎么是实现的呢？很简单，利用project提供的扩展。

##扩展的使用

定义一个名字是HelloExtension的Groovy类（名字其实无所谓叫什么，而且居然不需要实现任何的接口）：

{% codeblock lang:groovy %}
package me.zeph.gradle.extension

class HelloExtension {
    String message;
}
{% endcodeblock %}

改变一些插件的实现：

{% codeblock build.gradle lang:groovy %}
package me.zeph.gradle.plugin

import me.zeph.gradle.extension.HelloExtension
import org.gradle.api.Plugin
import org.gradle.api.Project

class HelloPlugin implements Plugin<Project> {

    @Override
    void apply(Project project) {
        project.extensions.add('hello', HelloExtension)
        project.task('hello') << {
            println project.hello.message
        }
    }
}
{% endcodeblock %}

project.extensions.add('hello', HelloExtension)，这段代码将HelloExtension添加到project的extensions中，于是task就可以通过project.hello.message来获取。是不是很简单？

##告诉别人这是个插件：插件id

那么，功能部分都写完了，怎么样让其他构件脚本知道这是一个插件能？配置META-INF。

在resources目录下建立这样一个目录结构：/resources/META-INF/gradle-plugins

然后在这里建立一个名字是me.zeph.hello.properties的Property文件，文件里的内容是：
{% codeblock me.zeph.hello.properties lang:properties %}
implementation-class=me.zeph.gradle.plugin.HelloPlugin
{% endcodeblock %}

这个Property文件的命名并不是随意定义的，名字的作用是定义该插件的id，什么意思？说白了就是apply时使用的名字。如下：
{% codeblock lang:groovy %}
apply plugin: 'me.zeph.hello'
{% endcodeblock %}

至于里面的内容，我就不解释了，一眼就看明白了。

##使用生成的插件

到这里，一个独立的插件工程就完成了，实验一把！！

运行gredlew clean assemble，将生成的jar文件，拷贝到其他的项目目录中（这里没有upload到仓库，所以直接文件形式引入依赖）。

{% codeblock lang:groovy %}
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath fileTree(dir: 'libs', include: '*.jar')
    }
}

apply plugin: 'me.zeph.hello'

hello {
    message = 'hello gradle plugin'
}
{% endcodeblock %}

然后运行gradlew hello，就可以看到hello任务的执行。

总结，其实实现一个Gradle的独立插件工程，从建立工程的角度还是比较简单的，关键在如何通过Groovy实现插件，已经理解插件的api。

参考资料：

1.https://gradle.org/docs/current/userguide/custom_plugins.html













