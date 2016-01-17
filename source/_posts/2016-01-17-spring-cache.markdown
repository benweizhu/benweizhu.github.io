---
layout: post
title: "了解Spring Cache"
date: 2016-01-17 09:26:45 +0800
comments: true
categories:
---
自从Spring 3.1，Spring框架提供了透明式给Spring应用添加缓存。类似于对事物的支持，抽象缓存机制允许一致的方式来实现不同的缓存解决方案，以满足对代码的最小影响。

Spring 4.1后，抽象缓存机制有了重大的改进，包含支持JSR-107注解和更多的定制选项。

##Cache vs Buffer

术语“buffer”和“cache”倾向于交换的使用。然而，它们代表着不同的东西。buffer常常被作为快慢实体之间（比如：发送方很快，接收方较慢）的中间临时存储。乙方必须等待另一方，因为执行效率（速率）的不同，buffer通过允许数据在移动时，以一整块数据为单位而不是小块形式来调解这种速率不一致的问题。数据在buffer中只会读写一次。更进一步，至少对于一方来说buffer是可见的。

另一方面，Cache就定义上而言，对于任何一方都是隐藏。它也会提升性能，但是仅限于同样地数据以快速的方式读取多次。

Cache的核心是将抽象层应用到Java方法上，减少方法执行的次数，因为信息在Cache中可以获取到。那就是，每次目标方法被触发时，抽象层就会执行一个cache行为，来检测方法是否已经针对指定的参数执行过一次。如果是，那么就会cache结果，而不会执行实际方法；如果没有，那么就执行相应的方法，方法执行的结果会被cache，下次同样的方法和参数执行的时候，就会返回cache的结果。这样，对于消耗更大的方法（无论是CPU还是IO）都只会针对给定的参数执行一次。Cache的逻辑完全是透明的，不会对触发调用产生任何干扰。

该抽象层也提供了其他Cache相关的操作的，允许更新Cache的内容或者删除一部分内容（条目）。这对于需要在应用程序运行阶段改变Cache的情况非常有用。

就像其他Spring框架的其他服务，Cache服务只是一个抽象层，需要实际的存储方式来存储数据。抽象层的关键类是org.springframework.cache.Cache和org.springframework.cache.CacheManager接口。

有几种开箱即用的实现：JDK java.util.concurrent.ConcurrentMap based caches, EhCache, Gemfire cache, Guava caches， JSR-107 compliant caches。

要使用cache抽象机制，需要负责两个方面：
cache声明 - 指定要被cache的方法以及策略
cache配置 - 背后的cache存储在哪里，从哪里读取

##关键注解

{% codeblock lang:java %}
@Cacheable triggers cache population
@CacheEvict triggers cache eviction
@CachePut updates the cache without interfering with the method execution
@Caching regroups multiple cache operations to be applied on a method
@CacheConfig shares some common cache-related settings at class-level
{% endcodeblock %}

@Cacheable用来声明方法可以被缓存（cache）

{% codeblock lang:java %}
@Cacheable("books")
public Book findBook(ISBN isbn) {...}
{% endcodeblock %}

其中，books是该cache的名字。     
在大部分情况下，只有一个cache被声明，注解允许多个名字被指定，所以就可以有多个cache被使用。在这种情况下，每个cache都会在方法执行之前被检测，如果至少有一个cache命中，那么相关的值就会被返回：

{% codeblock lang:java %}
@Cacheable({"books", "isbns"})
public Book findBook(ISBN isbn) {...}
{% endcodeblock %}

cache本质上是键值对存储，每一个cache的方法在被触发时，都会转换成一个合适的key来访问cache。开箱即用的是，有一个简单地KeyGenerator基于下面这个简单算法：

如果没有参数，就返回SimpleKey.EMPTY     
如果有一个参数，就返回该实例     
如果有多个参数，就返回一个SimpleKey包含所有的参数

只要参数包含自然键值，并且实现了合法的hashCode和equals方法，这种算法就适用于所有场景。

默认情况下，Cache抽象层会使用一个简单地CacheResolver来获取cache。

##有条件的cache

有些时候，一个方法并不适合在任何时候都cache，cache注解支持通过SpEL来指定条件，只有满足条件的参数来会被cache。
{% codeblock lang:java %}
@Cacheable(cacheNames="book", condition="#name.length < 32")
public Book findBook(String name)
{% endcodeblock %}

cache抽象不仅允许存储缓存，也允许收回（eviction）缓存。这个操作对于删除陈旧无用的缓存非常有用。和注解@Cacheable相反，@CacheEvict指定方法执行cache收回，也就是方法执行时会从cache中删除数据。和@Cacheable需要指定一个或者多个cache的名字。还提供一个额外allEntries属性，用来指定是否需要一个更广范围的cache需要收回（所有条目（无参数的，一个参数的，多个参数的））。
{% codeblock lang:java %}
@CacheEvict(cacheNames="books", allEntries=true)
public void loadBooks(InputStream batch)
{% endcodeblock %}

##CacheConfig

如果一个类中的所有方法操作都需要Cache，那么就可以使用@CacheConfig注解。
{% codeblock lang:java %}
@CacheConfig("books")
public class BookRepositoryImpl implements BookRepository {
    @Cacheable
    public Book findBook(ISBN isbn) {...}
}
{% endcodeblock %}

##EnableCaching

仅仅声明上面你需要cache的方法或者类是不足够的，和Spring的其他服务一样，该特性需要被启动（Enable）。

{% codeblock lang:java %}
@Configuration
@EnableCaching
public class AppConfig {
}
{% endcodeblock %}

Cache正如它在对于Buffer时所解释的那样，用来解决对重复获取的数据，减少对数据库的重复操作，从而提高数据获取速度。

参考资料：

1.http://docs.spring.io/spring/docs/current/spring-framework-reference/html/cache.html
