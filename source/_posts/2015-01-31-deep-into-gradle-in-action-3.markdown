---
layout: post
title: "Gradle深入与实战（三）依赖管理工具"
date: 2015-01-31 18:41:46 +0800
comments: true
categories: 
---
大部分的项目都不是自包含的，也就是说，需要使用到其他项目的构建结果，比如一些Jar文件。它们作为输入文件，必须存在于项目的ClassPath下，程序才能编译和运行。这些输入文件有一个很表意的名字，叫做依赖。

Gradle允许你告诉它项目的依赖是什么，然后它就会负责找到这些依赖。这些依赖会从Maven或者Ivy的远程仓库下载下来（大部分情况），并缓存在本地的某个路径，这个过程叫做依赖解析。

Maven和Gradle一样也提供了类似的功能，而Ant没有，你只能告诉Ant依赖文件的相对或者绝对路径，让它去加载。

常常一个依赖自己也存在依赖，我们称为传递依赖，依赖管理工具又具有解析传递依赖的能力。

###Gradle的依赖管理

那么如何在Gradle中定义依赖呢？看个最简单的例子。

{% codeblock lang:groovy %}
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    testCompile 'junit:junit:4.11'// testCompile group: 'junit', name: 'junit', version: '4.11'
}
{% endcodeblock %}

项目使用了Java的插件，在repositories块中告诉Gradle使用maven的远程仓库作为依赖下载地址，在dependencies块定义了一个junit的依赖，并说明了分组(Maven中的Scope)，后面注释中有一个表意更完整的依赖定义，说明了依赖声明使用的三个坐标group，name，version。

整个看起来是那么的表意，使用过Maven更会觉得是无缝转换，甚至更简洁。

###Dependency configurations

在Gradle中，依赖都被会分配到某一个具体的configuration中（这里我不倾向于翻译成配置，我觉得布局，或者分组更适合）。Configuration代表着一个或多个构件及构件所需依赖的一个分组。

Java插件已经预定义了一些configuration，比如，compile，runtime，testCompile，testRuntime等。

compile 放在这个configuration下的依赖是在编译产品代码时所使用的，但它作为一个分组，包含产品代码和编译所需的依赖。    
runtime 产品代码在运行时需要的依赖，默认，也会包含compile中的依赖。    
testCompile 编译测试代码时所需要的依赖，默认，被编译的产品代码和产品代码需要的编译依赖也属于该分组。    
testRuntime 运行测试时需要的依赖。默认，包含compile，runtime和testCompile的分组的构建和依赖。

使用过Maven的都应该知道分组的含义，这里讲解给不明白的同学，依赖之所以要分组，是因为，每个阶段对依赖的需要不一样，最明显的是产品代码和测试代码，比如junit在产品代码中就不需要。

那么，为什么产品代码的编译阶段和运行阶段也分组，一般编译阶段需要的依赖，在运行阶段也需要，但是反过来就不一定了。比如，你通过反射去load一个class，这时该class就不一定需要在编译阶段存在。

一个更常见的例子，做web开发时需要servlet的依赖，但是只是编译阶段，运行时servlet依赖由servlet容器来提供。所以Gradle的War插件也提供了两个configuration，分别是providedCompile和providedRuntime，它们对依赖的使用范围定义和compile以及runtime一致，只不过依赖的Jar包不会被加到War包里面。

###依赖的多种定义方式

除了通过远程仓库和依赖坐标来定义依赖，Gradle还提供了另外两种常用的依赖定义方式，对本地文件的依赖，对某个项目的依赖。

####对文件的依赖

这种情况看起来是不是很奇葩，都有依赖管理了和Maven仓库了还要什么文件依赖。其实不然，使用这种定义方式，最常见场景是项目构建工具的迁移，从Ant到Gradle。无论任何项目，迁移过程都是小步前进，Gradle提供文件依赖的配置，就是为了解决这些特殊性。

{% codeblock lang:groovy %}
dependencies {
    runtime files('libs/a.jar', 'libs/b.jar')
    runtime fileTree(dir: 'libs', include: '*.jar')
}
{% endcodeblock %}

####对另一个工程的依赖

项目中划分子模块是很平常的事情，前端Controller和数据层Dao分离管理就是一个例子，那么在进行前端Controller模块构建时，就需要将数据层模块作为依赖。定义方式如下：

{% codeblock lang:groovy %}
dependencies {
    compile project(':shared')
}
{% endcodeblock %}

###依赖版本冲突

依赖冲突是所以依赖管理中最头痛的问题，这常常出现在传递依赖中。Gradle对解决传递依赖提供了两种策略，使用最新版本或者直接导致构建失败。默认的策略是使用最新版本。虽然这样的策略能够解决一些问题，但是还是不够。常见的一种情况是，NoSuchMethond或者ClassNotFound。这时候，你可能需要一些特殊手段，比如排除不想要的传递依赖。

####排除传递依赖

排除传递依赖有多种原因，远程仓库中不存在，运行时不需要，或者版本冲突。排除传递依赖的方式有两种：1.直接在configuration中排除 2.在具体的某个dependency中排除

{% codeblock lang:groovy %}
configurations {
    compile.exclude module: 'commons'
    all*.exclude group: 'org.gradle.test.excludes', module: 'reports'
}

dependencies {
    compile("org.gradle.test.excludes:api:1.0") {
        exclude module: 'shared'
    }
}
{% endcodeblock %}

####通过命令行查看依赖关系

当出现依赖冲突时，最主要的还是要分析依赖冲突的原因，Gradle提供了两个任务来帮助你分析依赖关系

dependencies - Displays all dependencies declared in root project 'projectReports'.   
dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.

#####**Tips：输出依赖关系图到文件**

在命令行中直接使用gradle dependencies可以打印出依赖图，但是在命令行中查看始终不太方便，我们可以将结果输出到一个文件中，如下：
{% codeblock lang:python %}
gradle dependencies > dependencies.txt
{% endcodeblock %}
dependencies.txt保存在项目的根目录

Gradle的官方文档中关于Gradle的依赖管理的内容还有很多，比如，如何访问需要用户名密码授权的Maven仓库等等。等多内容，可以参考官方文档：http://gradle.org/docs/current/userguide/dependency_management.html

下一节，利用前三节学到的知识，编写集成测试任务，并单独划分SourceSet。

参考资料：   
1.Gradle官方文档









