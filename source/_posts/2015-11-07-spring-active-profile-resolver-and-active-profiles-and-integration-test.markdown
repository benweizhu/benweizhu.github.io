---
layout: post
title: "Spring4.0中通过ActiveProfiles和SpringActiveProfileResolver为不同环境下的集成测试指定不同的Properties配置"
date: 2015-11-07 17:46:18 +0800
comments: true
categories: 
---
##场景

在运行集成测试的时候，很可能会遇到这样一种情况：

本地的测试环境和持续集成服务器上的运行环境不同，最可能不同的的一种场景就是数据库服务器的配置不同，比如：host，端口，instance。

##解决方案

这个时候，我希望，不同的环境下，使用不同的application.properties文件配置。

如果你熟悉Spring Boot中profile的概念，那么，你一定会想到指定不同的profile，于是就有了两个不同的配置文件application-local.properties和application-ci.properties。

如果是启动Spring Boot，可以通过--spring.active.profile=dev来指定不同的profile。

如果是测试呢？通过注解@ActiveProfiles。
{% codeblock lang:java %}
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
@WebAppConfiguration
@Transactional
@ActiveProfiles("test")
public abstract class AbstractControllerIntegrationTest {

    protected MockMvc mockMvc;

    @Autowired
    private WebApplicationContext webApplicationContext;

    @Before
    public void setUp() throws Exception {
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    }
}
{% endcodeblock %}

可是，这样做还有一个问题，通过@ActiveProfiles("test")，是硬编码该测试使用test这个profile，那本地环境和集成测试环境要使用不同的profile，应该怎么办呢？

Spring 4中提供了一个接口：ActiveProfilesResolver，以可编程的方式提供active profile的解析。

{% codeblock lang:java %}
import org.springframework.test.context.ActiveProfilesResolver;

public class SpringActiveProfileResolver implements ActiveProfilesResolver {
    @Override
    public String[] resolve(Class<?> testClass) {
        final String activeProfile = System.getProperty("spring.profiles.active");
        return new String[]{activeProfile == null ? "test" : activeProfile};
    }
}
{% endcodeblock %}

使用方式很简单，在ActiveProfiles中指定resolver。

{% codeblock lang:java %}
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
@WebAppConfiguration
@Transactional
@ActiveProfiles(resolver = SpringActiveProfileResolver.class)
public abstract class AbstractControllerIntegrationTest {

    protected MockMvc mockMvc;

    @Autowired
    private WebApplicationContext webApplicationContext;

    @Before
    public void setUp() throws Exception {
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    }
}
{% endcodeblock %}

根据上面ActiveProfileResolver的实现方式，你只需要在运行集成测试时，指定一个系统变量spring.profiles.active。

以Gradle为例：
如果运行集成测试的任务是test，那么你只需要这样写：
{% codeblock lang:java %}
test {
    systemProperty "spring.profiles.active", "ci"
}
{% endcodeblock %}

但是如果，你希望通过命令行参数传递，来决定active的profile，那么你还需要获取Gradle的命令参数：
{% codeblock lang:java %}
test {
    if (project.hasProperty('profile')) {
        systemProperty "spring.profiles.active", "$profile"
    }
}
{% endcodeblock %}

运行时的命令是：
{% codeblock lang:java %}
./gradlew test -Pprofile=ci
{% endcodeblock %}

##结束语

通过这样的配置，在集成测试环境中，配置执行的命令就可以通过传递参数来指定激活的profile，而开发环境使用默认值（即不同在命令行指定参数）。

参考资料：    
1.http://stackoverflow.com/questions/20551681/spring-integration-tests-with-profile 作答人：Sam (author of the Spring TestContext Framework)
