---
layout: post
title: "Gradle对资源的过滤处理"
date: 2017-11-23 13:01:08 +0800
comments: true
categories: 
---

两种场景需要动态的处理资源：

1.对resource下的资源进行处理    
2.对war包中其他资源，比如：jsp文件进行处理

```java
import org.apache.tools.ant.filters.*

processResources {
    filter ReplaceTokens, tokens: [
        "application.version": project.property("application.version")
    ]
}
```

```java
import org.apache.tools.ant.filters.*

war {
    filter(ReplaceTokens, tokens:['versionDate': "${new Date().format('yyyyMMdd')}".toString()])
    filter(ReplaceTokens, tokens:['copyright': "${new Date().format('yyyy')}".toString()])
}
```

比较简单，一眼就看完了，就不多介绍了。