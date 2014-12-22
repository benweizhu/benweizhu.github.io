---
layout: post
title: "Spring Web Service By Example"
date: 2014-05-11 17:05
comments: true
categories: 
---
Web Service是一种独立的、低耦合的Web服务，它由几个核心技术组成：XML、XSD、WSDL、SOAP、UDDI。我们根据这些知识点，来一个个的学习。

##**XML**

呵呵，这个太简单了，这个没什么好谈的，跳过。

##**XSD**

XSD（XML Schema Definition）是DTD（Document Type Definition）的替代品，作用是定义XML文档的合法构建模块。

它能够做的事情有很多：

1. 定义可出现在文档中的元素
2. 定义可出现在文档中的属性
3. 定义哪个元素是子元素
4. 定义子元素的次序
5. 定义子元素的数目
6. 定义元素是否为空，或者是否可包含文本
7. 定义元素和属性的数据类型
8. 定义元素和属性的默认值以及固定值

....

{% codeblock lang:xml %}
<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           targetNamespace="http://benweizhu.github.io"
           elementFormDefault="qualified">
    <xs:element name="SearchBookRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="name" type="xs:string"/>
                <xs:element name="author" type="xs:string" minOccurs="0"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

    <xs:element name="SearchBookResponse">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="id" type="xs:int"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

</xs:schema>
{% endcodeblock %}
我们来分析一下这个schema的含义：

（1）xmlns:xs= "http://www.w3.org/2001/XMLSchema"，它表示在这个schema中使用的元素，例如element，complexType，sequence等，都来自于命名空间"http://www.w3.org/2001/XMLSchema"，并且在使用这些元素时，要以xs最为前缀。

（2）targetNamespace= "http://benweizhu.github.io"，它表示在这个schema中定义的元素，例如SearchBookRequest，name，author等，都是来自于命名空间"http://benweizhu.github.io"。它表示在基于该schema构建XML时，XML中使用的元素来自于该命名空间。

{% codeblock lang:xml %}
<z:SearchBookRequest xmlns:z="http://benweizhu.github.io">
    <z:name>SpringWebService</z:name>
    <z:author>Spring</z:author>
</z:SearchBookRequest>
{% endcodeblock %}

（3）xmlns="http://benweizhu.github.io"，指定了一个默认的命名空间，什么意思呢？就是说如果schema中使用的元素，没有xs这样前缀，则都是来自于这个默认的命名空间的。

（4）elementFormDefault="qualified"，指出任何在XML实例文档所使用的元素，只要在此schema中声明过，就必须被命名空间限定。

（5）<xs:element name="xxx" type="yyy"/>，此处 xxx 指元素的名称，yyy 指元素的数据类型。XML Schema 拥有很多内建的数据类型。最常用的类型有：

   * xs:string
   * xs:decimal
   * xs:integer
   * xs:boolean
   * xs:date
   * xs:time

（6）复杂元素，定义复杂元素的方式有两种，例子已经在上面给出，一种是直接在元素内部定义，一种是通过type引用该复杂类型。

（7）属性，element元素包含有很多的属性，例如，minOccurs="0"，它表示出现最少次数为0，就是说这个元素是optional的，可选的。

##**SOAP**

SOAP（Simple Object Access Protocol，简单对象访问协议），是交换数据的一种协议规范，它是一种通信协议，基于XML，用于发送和接受消息，它是真正在WebService的服务端和客户端传送的信息。

SOAP使用应用层协议作为其传输协议。SMTP以及HTTP协议都可以用来传输SOAP消息，但是由于HTTP在如今的因特网结构中工作得很好，特别是在网络防火墙下仍然正常工作，所以被广泛采纳。

SOAP基于XML，所以归根到底，SOAP发送的消息就是一个XML格式的文档消息。

一条SOAP消息的基本结构如下：

{% codeblock lang:xml %}
<?xml version="1.0"?>
<soap:Envelope
xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">
<soap:Header>
  ...
</soap:Header>
<soap:Body>
  ...
  <soap:Fault>
    ...
  </soap:Fault>
</soap:Body>
</soap:Envelope>
{% endcodeblock %}

一条SOAP消息包含下列元素：

1. 必需的Envelope元素，它是用来将一个XML文档标识为一条SOAP消息。
2. 可选的Header元素，包含头部信息
3. 必需的Body元素，包含所有的调用和响应信息
4. 可选的Fault元素，提供有关在处理此消息所发生错误的信息

所有以上的元素均被声明为，命名空间 http://www.w3.org/2001/12/soap-envelope 中的元素，以及针对SOAP编码和数据类型的命名空间： http://www.w3.org/2001/12/soap-encoding 。

关于 xmlns:soap="http://www.w3.org/2001/12/soap-envelope" 应该比较容易理解，在上面的XSD介绍中已经解释过了，用于说明该XML的命名空间，相当于该SOAP的XSD中的targetNamespace。

可选的SOAP Header元素可包含有关SOAP消息的应用程序专用信息（比如认证、支付等）。如果Header元素被提供，则它必须是Envelope元素的第一个子元素。

{% codeblock lang:xml %}
<soap:Header>
<m:Trans xmlns:m="http://www.w3school.com.cn/transaction/" soap:mustUnderstand="1">234</m:Trans>
</soap:Header>
{% endcodeblock %}

必需的SOAP Body元素包含打算传送到消息最终端点的实际SOAP消息。
{% codeblock lang:xml %}
<soap:Body>
  <z:SearchBookRequest xmlns:z="http://benweizhu.github.io">
    <z:name>SpringWebService</z:name>
    <z:author>Spring</z:author>
  </z:SearchBookRequest>
</soap:Body>
{% endcodeblock %}

而可选的SOAP Fault元素用于指示错误消息。

##**WSDL**

WSDL指网络服务描述语言(Web Services Description Language)，也就是WebService的描述语言。

WSDL是一种使用XML编写的文档。这种文档用于描述某个web service，规定服务的位置，以及此服务提供的操作（或方法）。 

一个WSDL文档是由如下几种元素组成：
元素定义:\<types>, \<message>, \<portType>, \<binding>

WSDL类型 

\<types>元素定义web service使用的数据类型。为了最大程度的平台中立性，WSDL使用XML Schema语法来定义数据类型。

WSDL消息

\<message>元素定义一个操作的数据元素。每个消息均由一个或多个部件组成。可以把这些部件比作传统编程语言中一个函数调用的参数。

WSDL端口

\<portType>元素是最重要的WSDL元素。它可描述一个web service、可被执行的操作，以及相关的消息。可以把\<portType>元素比作传统编程语言中的一个函数库（或一个模块、或一个类）。

WSDL绑定协议

\<binding> 元素为每个端口定义消息格式和协议细节。 

{% codeblock lang:xml %}
<definitions>
    <types>
        definition of types........
    </types>
    <message>
        definition of a message....
    </message>
    <portType>
        definition of a port.......
    </portType>
    <binding>
        definition of a binding....
    </binding>
</definitions>
{% endcodeblock %}

**这里要说明一点，WSDL是WebService的描述语言，只是用来描述WebService，当客户端想要使用该WebService时，根据这个描述，可以知道该WebService的服务端口有哪些，服务内容有什么。但对于WebService本身的功能没有任何影响，换句话说，如果是你自己写的WebService，提供给自己使用，你根本不需要关心WSDL是什么，因为你知道自己提供的服务内容和端口。**

关于这点，我会在后面的Spring WebService代码中证明。


##**Spring WebService**

OK，到这里几个重要的概念都谈到了，应该开始正题，如果上面的内容没看懂，特别是WSDL，没关系，接下来，我们根据Spring WebService的例子，来一个个分析。

###Contract First 契约优先

首先实现WebService的消息通信协议，也就是提供什么样的服务，接收什么消息，返回什么消息。

根据前面的知识：要根据XSD来约束发送和接收的XML长什么样子。

{% codeblock lang:xml %}
<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           targetNamespace="http://benweizhu.github.io"
           elementFormDefault="qualified">
    <xs:element name="SearchBookRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="name" type="xs:string"/>
                <xs:element name="author" type="xs:string" minOccurs="0"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

    <xs:element name="SearchBookResponse">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="id" type="xs:int"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>
{% endcodeblock %}

在这个例子中，我提供了一个图书搜索的服务，传给我书名name和作者author，我返回对应书的id。

根据这个XSD文件，发送出去的请求的XML的应该是长类似下面这个样子：

{% codeblock lang:xml %}
<z:SearchBookRequest xmlns:z="http://benweizhu.github.io">
    <z:name>SpringWebService</z:name>
    <z:author>Spring</z:author>
</z:SearchBookRequest>
{% endcodeblock %}


###JAXB

根据这个定义的XSD文件，可以生成对应的Java类文件，这一步很简单，在Eclipse或者IntellIj中都有提供XSD到Java文件的转换插件。你只需要转换一下，就会生成的两个类SearchBookRequest和SearchBookResponse，它们会在下面的Endpoint中被使用。

###Endpoint

实现Endpoint，也就是，这个请求消息给服务端后，有谁来处理，并返回响应结果。

Spring提供了注解的方式来配置一个Endpoint。

{% codeblock lang:java %}
package me.zeph.spring.ws.example.endpoint;

import jaxbgen.SearchBookRequest;
import jaxbgen.SearchBookResponse;
import org.springframework.ws.server.endpoint.annotation.Endpoint;
import org.springframework.ws.server.endpoint.annotation.PayloadRoot;
import org.springframework.ws.server.endpoint.annotation.RequestPayload;
import org.springframework.ws.server.endpoint.annotation.ResponsePayload;

@Endpoint
public class BookEndpoint {

	@PayloadRoot(localPart = "SearchBookRequest", namespace = "http://benweizhu.github.io")
	public @ResponsePayload SearchBookResponse searchBook(@RequestPayload SearchBookRequest searchBookRequest) {
		SearchBookResponse searchBookResponse = new SearchBookResponse();
		searchBookResponse.setId(1);
		return searchBookResponse;
	}
}
{% endcodeblock %}

第一个注解@Endpoint不用我解释了，就是说明这个类是一个WebService的Endpoint。
第二个注解@PayloadRoot里面有两个参数，localPart和namespace，它们分别匹配到XML中的根元素名字和命名空间，也就是说，如果发送过来消息的满足这两个条件，就交给该函数处理。这样就形成了消息到Endpoint的映射。
第三和第四个注解@RequestPayload和@ResponsePayload就简单了，分别用来指明请求和响应的对象。

###Spring的XML配置

配置比较简单，提供组件扫描和对Endpoint注解的支持即可。

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:sws="http://www.springframework.org/schema/web-services"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
      http://www.springframework.org/schema/web-services
      http://www.springframework.org/schema/web-services/web-services-2.0.xsd
      http://www.springframework.org/schema/context
      http://www.springframework.org/schema/context/spring-context.xsd">

    <sws:annotation-driven/>

    <context:component-scan base-package="me.zeph.spring.ws.example"/>

</beans>
{% endcodeblock %}

###web.xml

最后再来看web.xml怎么配置。

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
         http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
    <servlet>
        <servlet-name>spring-ws</servlet-name>
        <servlet-class>org.springframework.ws.transport.http.MessageDispatcherServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>spring-ws</servlet-name>
        <url-pattern>/services/*</url-pattern>
    </servlet-mapping>
</web-app>
{% endcodeblock %}

和SpringMVC一样，WebService也需要一个前端的请求分发的Servlet，MessageDispatcherServlet。你可能会想为什么是Servlet？

其实，这点，我在前面已经提到过，WebService的协议是绑定在Soap上面的，而Soap又是以http协议作为应用层协议的载体，发出去的Request固然是http请求。不配置对应Servlet，不配置对应的URL，那应该怎么做呢？

到此为止，一个Spring Web Service的服务端代码就写完了。你肯定会想，说好的WSDL呢？它去哪了？我这里刻意没有配置WSDL，就是要证明WSDL本身对于WebService没有影响，所以WSDL可以不用配。

我们先把WSDL放在一边，先把客户端代码完成。

###Spring WebService Template 客户端

这里的客户端代码使用Spring提供的WebServiceTemplate完成。

在applicationContext中配置WebServiceTemplate，如下：

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="webServiceTemplate" class="org.springframework.ws.client.core.WebServiceTemplate">
        <property name="defaultUri" value="http://localhost:8080/spring-ws/services/"/>
        <property name="marshaller" ref="marshaller"/>
        <property name="unmarshaller" ref="marshaller"/>
    </bean>

    <bean id="marshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
        <property name="packagesToScan">
            <list>
                <value>jaxbgen</value>
            </list>
        </property>
    </bean>
</beans>
{% endcodeblock %}

defaultUri用来说明该Template会向哪个URL发出请求。关于这个URL，因为在web.xml配置时，该MessageServlet是接受spring-ws/services/*的任何请求，而Spring WebService的Endpoint映射并不是根据URL来判断，所以这里你可以写任何满足该模式的URL，例如： http://localhost:8080/spring-ws/services/ssss 。

Marshaller用于实现XML到Java和Java到XML的转换。在Template需要配置marshaller和unmarshaller，这里它们用的同一个Marshaller对象。

客户端Java代码：

{% codeblock lang:java %}
package integration;

import jaxbgen.SearchBookRequest;
import jaxbgen.SearchBookResponse;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.ws.client.core.WebServiceTemplate;

import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.assertThat;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class BookClientIntegrationTest {

	@Autowired
	private WebServiceTemplate webServiceTemplate;

	@Test
	public void shouldGetResponseFromWebService() {
		//given
		SearchBookRequest searchBookRequest = new SearchBookRequest();

		//when
		SearchBookResponse searchBookResponse = (SearchBookResponse) webServiceTemplate.marshalSendAndReceive(searchBookRequest);

		//then
		assertThat(searchBookResponse.getId(), is(1));
	}
}
{% endcodeblock %}

OK，到此为止，Spring WebService的服务端和客户端代码完成。

所以，现在我们再回过头来，看看WSDL。

###WSDL

#####Automatic WSDL exposure 自动生成WSDL

在刚才的Spring XML配置中（/WEB-INF/[servlet-name]-servlet.xml）中加一段XML配置：

{% codeblock lang:xml %}
<sws:dynamic-wsdl id="book" portTypeName="SearchBookRequest"
                      locationUri="http://tobereplacedbyws:8080/spring-ws/services/">
        <sws:xsd location="classpath:book.xsd"/>
</sws:dynamic-wsdl>
{% endcodeblock %}
要注意位置有两点：

1. 属性id，除了唯一指定一个元素之外，它还有个更重要的作用，是说明该wsdl的名字。当你要访问该wsdl时，访问的url是 http://localhost:8080/spring-ws/services/book.wsdl 。
2. 属性locationUri，前面已经看到WSDL跟WebService的功能性服务没有半毛钱关系，所以这里的locationUri属性配置也不会影响到WebService的功能，它只是在你在wsdl中看到的一个字段，用来描述服务端的服务端口（URL）是什么。如果你看上面例子中的locationUri，你会发现不是写的localhost，而是tobereplacedbyws。为什么呢？因为MessageDispatcherServlet提供了一个特性，可以在部署时，根据主机的名字，替换这个位置的内容。如何开启这个特性呢？

在web.xml中配置一个initparam即可。

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
         http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

    <servlet>
        <servlet-name>spring-ws</servlet-name>
        <servlet-class>org.springframework.ws.transport.http.MessageDispatcherServlet</servlet-class>
        <init-param>
            <param-name>transformWsdlLocations</param-name>
            <param-value>true</param-value>
        </init-param>
    </servlet>

    <servlet-mapping>
        <servlet-name>spring-ws</servlet-name>
        <url-pattern>/services/*</url-pattern>
    </servlet-mapping>

</web-app>
{% endcodeblock %}

这个时候，你访问 http://localhost:8080/spring-ws/services/book.wsdl ，就可以看到对应的WSDL信息，然后你就可以用SoapUI等类似软件去测试你的Webservice。

参考资料：

(1) spring webservice reference

(2) http://www.w3school.com.cn/ws.asp

