---
layout: post
title: "Gradle深入与实战（二）"
date: 2015-01-31 13:57:55 +0800
comments: true
categories: 
---
没有介绍Gradle的基础知识，直接开始实战，目的是为了更快的让大家开始使用Gradle做构建，快速上手，当需要实现的自动化需求更复杂时，在深入学习基础知识。

这一篇，我们直接开始Java插件的使用。

###应用Java插件

Gradle是一个通用构建工具，也就是说，它不单是为Java而生。比如，还可以做Groovy，Scala的构建。这取决于你使用什么样的插件。

大部分Java项目的基本步骤都非常类似，编译Java源代码，运行单元测试，拷贝生成的class文件到目标目录，打包Jar文件（或者War包，Ear包），而这些重复且约定俗成的任务，如果可以不用写一行构建代码就实现，是多么的棒！Maven就做到这一点，采用约定由于配置的思想，预先定义常用的任务，并定义它们的执行顺序。

Gradle吸收了Maven的这个优点，通过插件，实现预定义任务和任务之间依赖关系的导入，这样就可以在一行代码都不写的情况下（如果应用插件，你觉得也算一行的话，那就写一行吧），直接使用已经定义的任务。

{% codeblock lang:groovy %}
apply plugin: 'java'
{% endcodeblock %}

###SourceSet和项目布局

就和Maven一样，在默认的情况下，项目的目录结构是固定的Java世界的标准项目目录布局，只不过Maven的不可以改，但是Gradle可以改。

{% codeblock lang:groovy %}
src {//目录结构而已，不是代码
	 main {
		 java
		 resources
	 }
	 test {
		 java
		 resources
	 }
}
{% endcodeblock %}

Java插件引入了一个概念叫做SourceSets，它代表了一组源文件，通过修改SourceSets中的属性，可以指定哪些源文件（或文件夹下的源文件）要被编译，哪些源文件要被排除。Gradle就是通过它实现Java项目的布局定义。

Java插件默认实现了两个SourceSet，main和test。每个SourceSet都提供了一系列的属性，通过这些属性，可以定义该SourceSet所包含的源文件。比如，java.srcDirs，resources.srcDirs。Java插件中定义的其他任务，就根据main和test的这两个SourceSet的定义来寻找产品代码和测试代码等。

在构建脚本中，怎么样定义或者修改SourceSet呢？Gradle提供了一系列的DSL，可以让你方便的定义或者修改配置。比如，sourceSets的DSL。

{% codeblock lang:groovy %}
sourceSets {
    main {
        java {
            srcDir 'src/java'
        }
        resources {
            srcDir 'src/resources'
        }
    }
}
{% endcodeblock %}
上面的这个例子，在sourceSets中，修改了Java插件中已经定义的SourceSet main，修改了它的java.srcDir和resources.srcDir。于是，项目的目录结构就改变了。

改变Java插件中预定义的项目目录结构，不是我们最终的目的，因为它是目前Java世界，标准的项目布局，或者说大家都遵守的项目布局。

sourceSets最主要的作用是增加新的目录约定，比如，你想要定义一个新的SourceSet来管理集成测试的源文件，这样可以将单元测试和集成测试分开管理。

至于，关于具体如何为集成测试写一个新的SourceSet会在后面介绍依赖管理时举例说明。

###Java插件提供的任务

Java插件提供了一系列的任务给你使用，包括编译，运行测试，打包等等。当你在项目中应用Java插件时，就已经将这些任务集成到你的项目中了。

在命令行中，运行gradle tasks命令，可以查看当前项目下主要的task。

{% codeblock lang:python %}
Build tasks
assemble - Assembles the outputs of this project.   
build - Assembles and tests this project.   
buildDependents - Assembles and tests this project and all projects that depend on it.   
buildNeeded - Assembles and tests this project and all projects it depends on.    
clean - Deletes the build directory.   
jar - Assembles a jar archive containing the main classes.   

Documentation tasks
javadoc - Generates Javadoc API documentation for the main source code.

Verification tasks
check - Runs all checks.  
test - Runs the unit tests.
{% endcodeblock %}

你可以对比Java插件应用前和应用后该命令的输出，Java插件提供的任务有很多，至于每个任务是做什么，这里就不赘述了。

Java插件除了为你预定义这些任务，该预定义了这些任务之间的依赖关系。如下图：

{% img http://gradle.org/docs/current/userguide/img/javaPluginTasks.png %}

你也可以通过命令gradle tasks --all来查看每个task各自有什么依赖。

当然，这里还是重点提下，Java插件中四个重要和常用的任务，assemble，check，build，clean。

**assemble**   
All archive tasks in the project, including jar. Some plugins add additional archive tasks to the project.   
**check**	
All verification tasks in the project, including test. Some plugins add additional verification tasks to the project.   
**build**	
check and assemble     
**clean**    
Deletes the project build directory.

assemble被用来产生Jar文件，输出目录在build/libs下。

check用来运行所有的验收任务，包括test任务，以及其他验收任务，比如checkstyle。

###Tips：在命令行中运行单个测试

JAVA插件中的test任务提供了一个filter属性，可以帮助指定运行test任务时，什么测试源文件要包含，什么要排除。

{% codeblock lang:groovy %}
test {
    filter {
        //include specific method in any of the tests
        includeTestsMatching "*UiCheck"

        //include all tests from package
        includeTestsMatching "org.gradle.internal.*"

        //include all integration tests
        includeTestsMatching "*IntegTest"
    }
}
{% endcodeblock %}

当然一般情况下，你不会这么去做。

但重点是，你可以通过**命令行传递的参数**来指定这个matching规则，这样你就可以通过命令行来指定跑某一类测试，或者单个测试。你一定遇到过，某个测试在命令行中可以运行，在IDE中不能运行，或者反过来。这时，你可以不会想要跑全部的测试来验证某一个测试。于是，你就可以通过命令行来运行某一个测试：

{% codeblock lang:groovy %}
gradle test --tests org.gradle.SomeTest.someSpecificFeature
gradle test --tests *IntegTest
{% endcodeblock %}

到目前为止，你已经了解了Java插件提供的一些核心功能和有用小技巧。虽然还未涉及到Jar任务和uploadfile任务（这些任务当需要时，再去看就行了），但是就启动项目而言，对Java插件的使用所需要了解的知识已经足够了。

下一节，讲解依赖管理和为集成测试添加新的SourceSet

参考资料：   
1.Gradle官方文档




