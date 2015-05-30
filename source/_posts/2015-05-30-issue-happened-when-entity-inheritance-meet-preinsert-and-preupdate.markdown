---
layout: post
title: "当Entity继承遇到Hibernate的@PrePersist和@PreUpdate"
date: 2015-05-30 17:03:04 +0800
comments: true
categories: 
---

上周做项目的时候遇到的关于实现审计日志方式的问题，这里记录一下。

假设你的数据库是这样设计的：
{% codeblock lang:json %}
{
    "Customer": {
        "id": 1,
        "name": "benwei"
    },
    "AdvancedCustomer": {
        "customerId": 1,
        "level": 1
    }
}
{% endcodeblock %}

一个Customer表和一个AdvancedCustomer表，AdvancedCustomer表中含有CustomerId作为外键。

在Java中的Entity实现是这样的：AdvancedCustomer继承自Customer，父类定义的继承策略是Joind。

{% codeblock lang:java %}
@Entity
@Table
@Inheritance(Strategy=InheritanceType.JOINED)
public class Customer {
	
}

@Entity
@Table
public class AdvancedCustomer extends Customer {
	
}
{% endcodeblock %}

Joind策略的含义是：A strategy in which fields that are specific to a subclass are mapped to a separate table than the fields that are common to the parent class, and a join is performed to instantiate the subclass.

通用的属性定义在父类中表，特殊的属性映射到另一个独立的表。

此时，你想要给应用添加一个审计功能。

你给Customer表添加LastModifiedBy和LastModifiedDate两个字段。

{% codeblock lang:json %}
{
    "Customer": {
        "id": 1,
        "name": "benwei",
        "lastModifiedBy": "benwei",
        "lastModifiedDate": "21/6/2015 23:01:11"
    },
    "AdvancedCustomer": {
        "customerId": 1,
        "level": 1
    }
}
{% endcodeblock %}


做法是使用EntityListener，在Listener中使用Hibernate的@PrePersist和@PreUpdate来监听事件的发生。

{% codeblock lang:java %}
public class AuditListener {
    
    @PrePersist
    public void prePersist(Object entity) {
    	if(entity instanceOf AdvancedCustomer) {
    		((AdvancedCustomer)entity).setLastModifiedDate(Datetime.now());	
    	}
    }

    @PreUpdate
    public void preUpdate(Object entity) {
    	...
    }
}

@Entity
@Table
@EntityListeners([AuditListener.class])
public class AdvancedCustomer extends Customer {
	
}
{% endcodeblock %}

###问题来了，这个prePersist或者preUpdate方法会在什么时候触发呢？当修改AdvancedCustomer中的任意变量时，比如name，level，都会触发prePersist或者preUpdate。

但是，你到数据库中去查看审计事件变化时会发现，当创建一个新的Customer，或者更新Customer的名字字段都没有问题。

当update子类中的变量level的时候，lastModifiedDate并没有发生变化。这是为什么？

###要找到原因，必须打开showSql属性，来查看Hibernate到底产生的SQL语句是什么。

你会发现，当修改level变量时，Hibernate只产生了一条update语句来更新AdvancedCustomer这张表。而创建Customer会同时更新AdvancedCustomer和Customer两张表，更新name字段，会更新Customer这张表。

也就是说，在PreUpdate触发之前，Hibernate在策略上已经决定了只更新AdvancedCustomer。即便之后改变了Customer中的lastModifiedDate，也没有改变它的行为。这里并不是说PreUpdate没有起到作用，而是Hibernate之决定更新一张表，至于更新什么内容，要等到PreUpdate之后决定（这一点可以从更新name时，lastModifiedDate发生了改变来证明）。

##如何解决：

目前，我们没有完美的解决方案可以在仍然使用PreUpdate的情况下，保证审计信息更新正确。

出现这个问题的主要原因是因为我们的实现受到框架实现机制的限制。

所以，我们改变了实现的策略，既然受到框架本身实现策略的限制，我们就脱离框架，在还未计入框架管理范围之内，就将审计信息写入Entity内，那么可行的一种方式就是AOP。在触发Hibernate的save方法之前，将审计信息写入Entity。







