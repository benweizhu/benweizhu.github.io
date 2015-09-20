---
layout: post
title: "Gradle深入与实战（四）自定义集成测试任务"
date: 2015-01-31 21:47:45 +0800
comments: true
categories: [Gradle深入与实战]
---
由于本小节，涉及到自定义任务，所以穿插一点自定义任务的知识。

##Gradle Task

在前面已经介绍过Gradle和Ant相似，由任务驱动，以任务依赖的方式形成任务链，从而实现构建生命周期。所以，任务是Gradle中一个完整的可执行单元。

如何定义任务：
{% codeblock lang:groovy %}
task hello {
    println 'hello Gradle'
}
{% endcodeblock %}
执行该任务，只需要输入命令gradle hello。定义task的方式有很多种：
{% codeblock lang:groovy %}
task myTask
task myTask { configure closure }
task myType << { task action }
task myTask(type: SomeType)
task myTask(type: SomeType) { configure closure }
{% endcodeblock %}

其中有一种定义方式，传入了一个参数type，作用是预定义该task的类型，指定类型之后，在传入的闭包中就可以使用该类型task提供的特殊变量或函数。

比如一个拷贝类型的task
{% codeblock lang:groovy %}
task copyDocs(type: Copy) {
    from 'src/main/doc'
    into 'build/target/doc'
}
{% endcodeblock %}
更过关于Task的内容，在以后的章节中再介绍。

##自定义集成测试任务

**现在我们开始写一个集成测试的task，需求是这样的：**

作为一个Java的程序员，我想要将单元测试和集成测试分离

1.我想要 将单元测试全部放在src/test/unit目录中，将集成测试全部放在src/test/intgetaion中     
2.我想要 能够单独运行我的集成测试    
3.我想要 在运行build命令时，同时跑单元测试和集成测试   


**根据这样的一个需求，划分几步来做：**   
1.建立目录    
2.目录结构已经和原来的默认规约不同，所以要更改Java插件提供的SourceSet test，来映射单元测试目录结构    
3.需要新建一个SourceSet intTest，来映射集成测试目录结构    
4.Java插件会给新建的SourceSet intTest定义两个Configuration，分别是intTestCompile和intTestRuntime，那么就需要给这两个分组指定构件内容和依赖    
5.定义一个名字叫做integrationTest的测试的task

**那么我们从第二步和第三步开始，修改Java插件提供的SourceSet test和新建SourceSet intTest：**

{% codeblock lang:groovy %}
// 定义一些常量，在其他位置使用
ext {
    unitJavaSrcDir = 'src/test/unit/java'
    unitResourcesSrcDir = 'src/test/unit/resources'
    intJavaSrcDir = 'src/test/integration/java'
    intResourcesSrcDir = 'src/test/integration/resources'
}

sourceSets {
    test {
        java {
            srcDir unitJavaSrcDir
        }
        resources {
            srcDir unitResourcesSrcDir
        }
    }
    intTest {
        java {
            srcDir intJavaSrcDir
        }
        resources {
            srcDir intResourcesSrcDir
        }
    }
}
{% endcodeblock %}

**第三步，给intTestCompile和intTestRuntime指定指定构件内容（产品代码）和依赖**
{% codeblock lang:groovy %}
dependencies {
    testCompile 'junit:junit:4.11'
    testCompile 'org.mockito:mockito-core:1.9.5'

    intTestCompile sourceSets.main.output // 将sourceSets.main中的输出class指定到intTestCompile中
    intTestCompile configurations.testCompile // 将configurations.testCompile的依赖拿过来
}
{% endcodeblock %}

**最后一步，定义一个test类型的task，并让check任务依赖于它**
{% codeblock lang:groovy %}
task integrationTest(type: Test) {
    testClassesDir = sourceSets.intTest.output.classesDir
    classpath = sourceSets.intTest.runtimeClasspath
}

check.dependsOn integrationTest
{% endcodeblock %}

然后，你就可以在命令行中运行gradle integrationTest。

完整版本如下：

{% codeblock lang:groovy %}
apply plugin: 'java'
apply plugin: 'idea'

ext {
    unitJavaSrcDir = 'src/test/unit/java'
    unitResourcesSrcDir = 'src/test/unit/resources'
    intJavaSrcDir = 'src/test/integration/java'
    intResourcesSrcDir = 'src/test/integration/resources'
}

sourceSets {
    test {
        java {
            srcDir unitJavaSrcDir
        }
        resources {
            srcDir unitResourcesSrcDir
        }
    }
    intTest {
        java {
            srcDir intJavaSrcDir
        }
        resources {
            srcDir intResourcesSrcDir
        }
    }
}

repositories {
    mavenCentral()
}

dependencies {
    testCompile 'junit:junit:4.11'
    testCompile 'org.mockito:mockito-core:1.9.5'

    intTestCompile sourceSets.main.output
    intTestCompile configurations.testCompile
}

task integrationTest(type: Test) {
    testClassesDir = sourceSets.intTest.output.classesDir
    classpath = sourceSets.intTest.runtimeClasspath
}

check.dependsOn integrationTest

idea {
    module {
        testSourceDirs += file(unitJavaSrcDir)
        testSourceDirs += file(unitResourcesSrcDir)
        testSourceDirs += file(intJavaSrcDir)
        testSourceDirs += file(intResourcesSrcDir)
    }
}
{% endcodeblock %}

参考资料：   
1.Gradle官方文档   
2.http://selimober.com/blog/2014/01/24/separate-unit-and-integration-tests-using-gradle/

