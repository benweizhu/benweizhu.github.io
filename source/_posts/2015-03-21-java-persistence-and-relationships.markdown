---
layout: post
title: "JPA中的多对多关系"
date: 2015-03-21 10:44:43 +0800
comments: true
categories: 
---
在关系型数据库中，表与表之间的关系是通过外键来实现，而其中最常见的关系有两种：一对多和多对多。    

在面向对象的世界，对象与对象之间的关系是通过从源对象到目标对象的对象引用实现。关系是一个对象含有的其他对象或者对象的集合，而在关系型数据库中，关系要么通过定义另一张表的外键建立，要么通过中间表建立。

在Java或者JPA的世界里，所有的关系都是单向性的，一个源对象保有目标对象的索引，并不能保证目标对象和源对象一样包含源对象的索引。这个关系型数据库就不一样了，在关系型数据库中，表之间的关系通过外键来实现，查询语句通过这个外键，可以进行正向和反向的查询。

ManyToMany多对多关系是关系型数据库中一种非常常见的表关系。

在JPA中，如果要实现ManyToMany关系，常常是利用的一种方式是中间表。所有的ManyToMany关系都需要一个JoinTable，在这个JoinTable中，通过JoinColumns定义了两个外键，一个外键来指向源表的主键，另一个外键指向目标表的主键，通常JoinTable的主键是两个外键的组合。

{% codeblock lang:java %}
@Entity
public class Employee {
  @Id
  @Column(name="ID")
  private long id;
  ...
  @ManyToMany
  @JoinTable(
      name="EMP_PROJ",
      joinColumns={@JoinColumn(name="EMP_ID", referencedColumnName="ID")},
      inverseJoinColumns={@JoinColumn(name="PROJ_ID", referencedColumnName="ID")})
  private List<Project> projects;
  .....
}
{% endcodeblock %}

双向ManyToMany

虽然在数据库中ManyToMany总是双向的，但是在对象模型中定义双向关系时，需要定义映射的发起方（管理方）和映射的接收方。发起方需要定义JoinTable，接收方则需要定义mappedBy。如果不使用mappedBy，则Jpa的实现者会认为，这种多对多关系是两个独立定义的关系，就会有重复的两行记录插入到JoinTable中。

{% codeblock lang:java %}
@Entity
public class Project {
  @Id
  @Column(name="ID")
  private long id;
  ...
  @ManyToMany(mappedBy="projects")
  private List<Employee> employees;
  ...
}
{% endcodeblock %}


双向关系的共同毛病

需要应用程序自己去维护它们（对象）之间的双向关系，否则就不同步（out of sync）。在写set方法或者，多对多关系中的add方法时，需要注意将自己的引用设置到包含的集合对象中。

{% codeblock lang:java %}
public class Employee {
    private List phones;
    ...
    public void addPhone(Phone phone) {
        this.phones.add(phone);
        if (phone.getOwner() != this) {
            phone.setOwner(this);
        }
    }
    ...
}
 
public class Phone {
    private Employee owner;
    ...
    public void setOwner(Employee employee) {
        this.owner = employee;
        if (!employee.getPhones().contains(this)) {
            employee.getPhones().add(this);
        }
    }
    ...
}
{% endcodeblock %}

多对多关系的管理方的选择

多对多关系的管理方的定义取决于JoinTable的定义方，至于由哪一边作为管理方，取决于业务的关系，所以并不固定。但是必须要理解的时，它们之间建立关系的方式（即谁管理谁）。举个例子，User和Group，这是一个常见的多对多关系，当你将管理方定义在Group这边时，你要做的是将User添加到Group，然后保存Group。而不能将Group添加给User，然后保存User。


参考资料：    
1.http://en.wikibooks.org/wiki/Java_Persistence/ManyToMany     
2.http://stackoverflow.com/questions/4935095/jpa-hibernate-many-to-many-cascading



