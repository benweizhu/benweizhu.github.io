---
layout: post
title: "Spring Restful Web Service By Example"
date: 2014-04-06 12:06
comments: true
categories: 
---

REST是[Roy Fielding](http://en.wikipedia.org/wiki/Roy_Fielding)在他的2000年博士论文《Architectural Styles and the Design of Network-based Software Architectures》中提出的概念，全称是Representational State Transfer。

Roy Fielding是互联网一个极为重要的人物，他是HTTP协议（1.0版和1.1版）的主要设计者，Apache服务器软件的作者之一，Apache基金会的第一任主席。

他在他的论文中提到一点：“网络研究主要关注系统之间通信行为的细节、如何改进特定通信机制的表现，常常忽视了一个事实，那就是**改变应用程序的互动风格比改变互动协议，对整体表现有更大的影响**。”。

REST是个名字，加上ful之后，变成了形容词“具有REST风格的”。

在看过一些资料之后，我觉的REST越来越火的主要原因之一，就是想在的互联网应用已经不仅仅是内容的资源，还包括计算资源。而资源的消费者也不仅仅是浏览器（人在消费），还有其他应用（应用程序在消费），所以资源的表现形式在变化。

##**Web Service**

Web Service是一种服务导向架构的技术，通过标准的Web协议提供服务，目的是保证不同平台的应用服务可以互操作。而它的技术通常是根据SOAP协议进行传递XML格式消息。而基于Java的主流WEB服务开发框架往往需要通过WSDL生成客户端的源代码。

PS：这里没有任何想要对比基于SOAP和基于REST的Web Service的区别或者优缺点的意思。

##**Restful Web Service**

Restful Web Service又称为Restful Web API。Rest本身其实不是一种技术，而是一种风格，它基于HTTP协议，具有统一的URL资源，通过HTTP协议提供的方法来表示对数据的操作，资源的表现形式多样化，具体表现形式由资源的消费者决定等等。

说了这么多，还是回到标题，如何实现？

如果你的项目采用Spring开发，那么Spring提供了一套基于Spring MVC的Restful方案，可以非常方便的通过Controller提供想要的Web Service，又可以通过RestTemplate实现客户端对Web Service操作。


PS：本例子是针对Spring 3.2以后，3.1版本有所不同。

首先来实现一个基于Controller的Restful Web Service：

首先是Domain，这个很简单一个Car类，包含名字和价格。
{% codeblock lang:java %}
package me.zeph.spring.restful.demo.model;

public class Car {
	private String name;
	private int price;

	public void setName(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}

	public void setPrice(int price) {
		this.price = price;
	}

	public int getPrice() {
		return price;
	}
}
{% endcodeblock %}

然后是Controller，按照普通写RequestMapping一样方式，但这里返回的是Domain对象。
这里有两个位置不同，一个是注解@ResponseBody用来说明返回的结果是HttpResponse的body部分。
另一个参数produces用来说明，这里只接受Http请求中head里说明了Accept类型是Application/Json。
{% codeblock lang:java %}
package me.zeph.spring.restful.demo.controller;

import me.zeph.spring.restful.demo.model.Car;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import static org.springframework.http.MediaType.APPLICATION_JSON_VALUE;
import static org.springframework.web.bind.annotation.RequestMethod.GET;

@Controller
@RequestMapping(value = "car", method = GET)
public class CarController {

	public static final String VIEW_NAME = "car";

	@RequestMapping
	public String getCarByView() {
		return VIEW_NAME;
	}

	@RequestMapping(produces = {APPLICATION_JSON_VALUE})
	@ResponseBody
	public Car getCarByJSON() {
		return getCar();
	}

	private Car getCar() {
		Car car = new Car();
		car.setName("BenZ");
		car.setPrice(300000);
		return car;
	}
}
{% endcodeblock %}

这样基本就算完成了，可能有人会问，为什么上面的getCarByView方法没有写produces。其实，我也看到许多网上例子，将返回HTML格式View的方法添加了produces。比如，添加Text/Http等。

为什么这里不加，原因是，我们知道，该方法是提供给浏览器请求，用于返回基于HTML的页面。而对于不同的浏览器而言，发出的HTTP请求的head的是不一样，以Chrome和IE为例。

Chrome发出的head是：“text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,\*/\*;q=0.8”

IE发出的head是：“\*/\*”

所以如果添加了produces，那么有可能返回结果不是HTML。我就遇到过，添加了Text/Http之后，在IE中返回给我的是JSON数据，而不是HTML页面。

如果不写，那么就是默认的选择，在不满足其他RequestMapping，会去走默认那条Mapping路线。你可以在RestClient中验证，URL是localhost:8080/spring-restful-demo/car，但是head不同，那么它返回给你的view格式就不同了。

其实，走到这一步，已经就完成了。

##**但是如果你在页面上和RestClient上做一个实验，就会发现它远远不止如此！！！**

在浏览器上输入localhost:8080/spring-restful-demo/car.html，会返回给你html页面。

在浏览器上输入localhost:8080/spring-restful-demo/car.json，会返回给你给你json数据。

明明通过浏览器发送请求的Head是Text/Html，为什么会返回给我Json数据。如果通过debug的方式，你会看到确实进入的是getCarByJSON方法。

在**RestClient**去试一把，输入地址，localhost:8080/spring-restful-demo/car.html，设置head为accept=Application/Json，返回给我的是HTMl页面。

##**默认规约**

原来Spring在选择映射的时候，有一定的规约，加入了检查条件，URL后缀，URL参数。并且其默认的检查顺序是后缀，参数，最后才是Accept。

也就是，如果你的URL是以HTML结尾，那么无论你的head是什么，Spring都会认为是返回HTML页面，因为你显示的指明了。

对于那些希望显示指明返回类型的应用，Spring的默认规约已经帮助它们实现。

但是如果，我不希望呢？

无论你的后缀还是参数是什么，我都希望Spring根据我head中的accept值来决定。

##**Content Negotiation（内容协商）**

这里就要引入另外一个东西：Spring MVC Content Negotiation（内容协商）。

它是用来帮助你定义在返回多种表现形式时，应该遵守的规约，默认我们是不需要定义的，那么它就会像上面所提到的去做。但如果你不想要默认的规则，那么你就需要在Spring的Context中定义这个Bean。

这里用我的demo中的例子，简单说明一下：
{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc
                    http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
                    http://www.springframework.org/schema/beans
                    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
                    http://www.springframework.org/schema/context
                    http://www.springframework.org/schema/context/spring-context-3.2.xsd">

    <context:component-scan base-package="me.zeph.spring.restful.demo"/>

    <mvc:annotation-driven content-negotiation-manager="contentNegotiationManager"/>

    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/view/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <bean id="contentNegotiationManager"
          class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
        <property name="favorPathExtension" value="false" />
        <property name="favorParameter" value="false" />
        <property name="defaultContentType" value="text/html" />
    </bean>
    
    <bean id="restfulTemplate" class="org.springframework.web.client.RestTemplate"/>

</beans>
{% endcodeblock %}

在Spring 3.2以后，实现内容协商的bean的名字叫做ContentNegotiationManager，需要通过他的工厂类ContentNegotiationManagerFactoryBean去定义规约。在这里，我修改了三个属性，favorPathExtension，favorParameter和defaultContentType，defaultContentType简单，就是默认的表现内容形式，favorPathExtension表示是否采用路径扩展作为最高优先级的判断，同理favorParameter表示是否采用参数作为第二优先级的判断。这里，我将它们两个都设置为false。即不采用它们作为判断条件。最后，在annotation-driven的标签里，添加属性contentNegotiationManager，说明使用@Controller注解的Controller都采用这种规约。

然后，我们再来是测试，在RestClient上，设置请求Url为localhost:8080/spring-restful-demo/car.html，但是呢，将head设为Accept=Applicaiton/Json。你猜结果怎样？乖乖的返回Json数据。

关于Content Negotiation就说到这里，关于它功能还有很多，以后有机会在补充，那么Web Service就差不多说完了。

##**RestTemplate实现客户端**

在来看看怎么实现Client端的代码。如果不用Spring，可以通过Apache的HttpClient作为客户端，当然写的代码要多一点。

如果使用Spring，可以通过Spring提供的RestTemplate，目的应该和Spring的JDBCTemplate类似，帮你封装了具体操作。

用起来很简单，我写了另一个Controller来在页面显示调用该Web Service的结果。
{% codeblock lang:java %}
package me.zeph.spring.restful.demo.controller;

import me.zeph.spring.restful.demo.common.URLs;
import me.zeph.spring.restful.demo.model.Car;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.client.RestTemplate;

@Controller
public class DisplayCarController {

	public static final String VIEW_NAME = "displayCarJson";
	private RestTemplate restTemplate;

	@Autowired
	public DisplayCarController(RestTemplate restTemplate) {
		this.restTemplate = restTemplate;
	}

	@RequestMapping(value = VIEW_NAME)
	public String getCar(Model model) {
		model.addAttribute("car", getCarByJson());
		return VIEW_NAME;
	}

	private Car getCarByJson() {
		return restTemplate.getForObject(URLs.REST_CAR_URL, Car.class);
	}

}
{% endcodeblock %} 

如果你需要特定指定请求的Head。可以使用RestTemplate的exchange方法，不过使用起来要麻烦一些。
{% codeblock lang:java %}
private Car getCarByJson() {
		HttpHeaders headers = new HttpHeaders();
		headers.add("accept", "application/json");
		HttpEntity httpEntity = new HttpEntity("", headers);
		ResponseEntity<Car> exchange = restTemplate.exchange(URLs.REST_CAR_URL, HttpMethod.GET, httpEntity, Car.class);
		return exchange.getBody();
}
{% endcodeblock %} 

不过第二种方式更能说明是指定了具体head来请求哪种资源。

这里使用Spring来发出请求，都是后端的，如果要用前端请求，固然是通过Ajax，不过那是另外一部分内容。

总而言之，在使用Spring Web Service的这个过程当中还是遇到了不少的坑。

首先，你需要知道有Content Negotiation这个东西，其次，不同的Spring版本，默认的规约不同。Spring 3.2和3.1就是一个分水岭，使用的时候要注意你使用的是哪个版本。

参考资料：

https://spring.io/guides/gs/rest-service/

https://spring.io/blog/2013/05/11/content-negotiation-using-spring-mvc

http://spring.io/blog/2013/06/03/content-negotiation-using-views/
