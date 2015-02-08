---
layout: post
title: "了解Spring Test对单元测试和集成测试的支持"
date: 2015-02-07 23:20:45 +0800
comments: true
categories: 
---
在敏捷开发和测试驱动开发中，自动化测试一直被认为是让软件开发人员能够有信心的进行软件开发的力量源泉。

所以，软件开发人员不仅要关注写出高质量的产品代码，同时也要关注写出高质量的测试代码。

有些人说，对于开发人员，自动化测试不过是利用Junit，Mockito，Selenium等测试框架写出的另一段程序代码而已。虽然听起来很不爽，但我也不否认这一点，这是事实，但我们也是按照写产品代码的要求来写出高质量的测试代码。

回到正题，今天的主题是了解Spring对基于Spring开发的单元测试和集成测试的支持。

如果你正在使用Spring做开发，那么在写测试代码的时候，无论是单元测试还是集成测试，如果只是用到Junit，Mockito，EasyMock这些测试框架，你一定会发现，它们是不够的。Spring框架的优点就在于，它的策略是试图涵盖Java开发的所有部分，测试也不例外。Spring提供了一套API来支持基于Spring的单元测试和集成测试。下面，我们来看看Spring在测试方面都提供一些什么样的优秀特性。

##反射测试工具 org.springframework.test.util.ReflectionTestUtils

依赖注入是Spring框架提供的最主要的特性之一，在Spring上下文管理范围内，Spring提供三种注入方式，Field注入，构造器注入，Setter注入。我们都知道在写单元测试时，只会关注被测试类本身的逻辑，一般我们都会将类中的依赖进行mock。

在没有Spring提供的反射测试工具的时候，我们一般都倾向于构造器注入，Setter注入的方式，因为这样在写测试代码时候，可以将mock的依赖传入到被测试类。但实际上，在Field上注入更符合Spring风格，或者更容易理解，我需要Spring给我注入这个对象，我就在它上面加一个@Autowired注解。所以构造器注入和Setter注入多少都是为了方便测试，不得已而为之。更有甚者，其实采用的是Field注入，但是为了测试，不得以添加一个setter方法。这就违反了简洁代码的原则，无用的方法，或者只被测试用到方法。

还好，Spring提供的ReflectionTestUtils拯救了我们。它提供这样的一个方法，ReflectionTestUtils.setField()，通过反射的方式将想要的依赖设置到对应的field上。

{% codeblock lang:java %}
@param target the target object on which to set the field
@param name the name of the field to set
@param value the value to set

ReflectionTestUtils.setField(xxService, "xxDao", mockedXxDao.class);
{% endcodeblock %}

##Spring MVC测试的Mock对象：MockHttpServletRequest，MockHttpSession等。

如果你正在测试Controller，而Controller又恰好对HttpServletRequest和HttpSession有操作，那么Spring提供的MockHttpServletRequest，MockHttpSession就派上用场了。

{% codeblock lang:java %}
MockHttpServletRequest mockHttpServletRequest = new MockHttpServletRequest();
MockHttpSession mockHttpSession = new MockHttpSession();
mockHttpSession.setAttribute("username", "user");
mockHttpServletRequest.setSession(mockHttpSession);
{% endcodeblock %}

当然，针对上面这种做法，你采用mockito也是可以实现的，而且代码量也差不多。
{% codeblock lang:java %}
HttpServletRequest mockHttpServletRequest = mock(HttpServletRequest.class);
HttpSession mockHttpSession = mock(HttpSession.class);
when(mockHttpSession.getAttribute("username")).thenReturn("user");
when(mockHttpServletRequest.getSession()).thenReturn(mockHttpSession);
{% endcodeblock %}

但是，如果你的Controller类中，使用了某个静态工具类方法，它对HttpServletRequest和HttpSession做了各种各样的操作，来维护当前用户的某些状态。从本质上，你应该将静态方法mock，但这并不容易实现。于是，你就必须mock HttpServletRequest和HttpSession中各种各样的依赖，来保证静态工具类方法不会抛空指针，而其中这些会抛空指针的操作与你期待的测试的行为无关。最直接的结果就是测试中的mock逻辑会特别的多，关键还不一定正确。

Spring的MockHttpServletRequest和MockHttpSession就可以解决这类问题，它默认初始化好大部分HttpServletRequest和HttpSession需要的依赖，所以不会出现空指针问题，也就简化了mock的过程，而其他你期待的返回结果都可以通过setter配置进去。

##Spring在集成测试方面的支持

###SpringJUnit4ClassRunner，@ContextConfiguration，@WebAppConfiguration，MockMvc

SpringJUnit4ClassRunner：它是在Junit下启动Spring集成测试的基础，为Junit提供Spring TestContext Framework所拥有的功能。   
@ContextConfiguration：用来决定根据什么样的配置为集成测试加载和配置ApplicationContext。   
@WebAppConfiguration：用来声明集成测试加载的ApplicationContext应该是一个WebApplicationContext。    
MockMvc：是Spring在服务器端对Spring MVC测试的支持，可以在对Controller的集成测试中，模拟对Controller某个Request的调用，是非常实用的集成测试组件。

看下面一个例子，如何使用上面4个组件来实现Controller的集成测试：

{% codeblock lang:java %}
import ...

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = WebContextConfiguration.class)
@WebAppConfiguration
public class xxxIntegrationTest {

	private MockMvc mockMvc;

	@Autowired
	private WebApplicationContext webApplicationContext;

	@Before
	public void setUp() throws Exception {
		mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
	}

	@Test
	public void shouldReturnKits() throws Exception {
		mockMvc.perform(get("/form"))
     		.andExpect(status().isOk())
     		.andExpect(content().mimeType("text/html"))
     		.andExpect(forwardedUrl("/WEB-INF/layouts/main.jsp"));
	}
}
{% endcodeblock %}

###测试类的事务管理 @TransactionConfiguration，@Transactional，@BeforeTransaction，@AfterTransaction

如果你正在对Dao层或者说数据库做集成测试，包括了CRUD所有基本操作的测试，那么你肯定会希望在单个测试运行结束之后，可以将数据库的状态回滚，这样除了可以重复的运行测试，更重要的是不会影响到其他测试，不会弄脏数据。那么，Spring提供的事务管理功能，除了可以实现产品代码中的事务管理，还可以实现测试代码的事务管理。

@TransactionConfiguration是为集成测试提供的类似@EnableTransactionManagement的功能的注解，用来显示的为集成测试指定某个TransactionManager和Rollback策略。但它并不是必须的，如果在Spring的上下文中，只有一个TransactionManager，且bean的名字是transactionManager，并且你认为的默认策略是Rollback，那么就可以不必配置@TransactionConfiguration。

@Transactional就和产品代码中一样，你需要某个测试类具有事务功能，就在该测试类上加上@Transactional注解。

{% codeblock lang:java %}
import ...

@TransactionConfiguration(transactionManager = "txManager")
@Transactional
public class xxxDaoIntegrationTest {
...
}
{% endcodeblock %}

@BeforeTransaction，@AfterTransaction，顾名思义，就是在事务前和事务后做的对应操作。


###@TestPropertySource，@DirtiesContext

@PropertySource用来以声明式的方式将Properties加载到Spring的Environment变量中，@TestPropertySource拥有比@PropertySource更高的优先级，可以用来加载专门为测试提供的Properties。

{% codeblock lang:java %}
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {
     @Autowired
     Environment env;

     @Bean
     public TestBean testBean() {
         TestBean testBean = new TestBean();
         testBean.setName(env.getProperty("testbean.name"));
         return testBean;
     }
}
{% endcodeblock %}

{% codeblock lang:java %}
@ContextConfiguration
@TestPropertySource("/test.properties") 
public class MyIntegrationTests {
    // class body...
}
{% endcodeblock %}

@DirtiesContext用来说明在某个测试执行之后，会导致Spring Context被污染，比如，修改了某个执行策略，改变了某个单例对象的状态。此时，该Context应该被关闭，之后的测试会使用新的Context。

@DirtiesContext可以被用在测试类上，也可以用在测试方法上，具体行为如下：     
* after the current test, when declared at the method level     
* after each test method in the current test class, when declared at the class level with class mode set to AFTER_EACH_TEST_METHOD     
* after the current test class, when declared at the class level with class mode set to AFTER_CLASS


这里只是介绍一些常用的比较重要的特性，除了以上这些，Spring Test Framework还提供许多其他特性，有效利用它们，可以让你写出方便的进行基于Spring的应用的测试，同时也是保证写出高质量测试的基础。

参考资料：    
1.Spring Reference Document, Spring Test


