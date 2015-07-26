---
layout: post
title: "Spring Web Security 实战 (四) - 了解Security基础架构"
date: 2015-06-17 21:41:24 +0800
comments: true
categories: 
---

在了解了Spring Security的基本使用之后，还是需要更深入的了解Spring Security的基础架构，核心组件等。

Spring Security是以自包含的角度来实现安全设计，所以使用Spring Security没有任何侵入性。因为Spring Security的目标是自己容器内管理， 所以不需要为你的Java运行环境进行什么特别的配置。

##核心组件
###SecurityContextHolder, SecurityContext和Authentication对象

SecurityContextHolder，就如该类的名字所描述的那样，它用来存放当前应用程序的当前安全上下文中的细节，当然也包括“当事人Principal”的细节。SecurityContextHolder将上下文信息存储在ThreadLocal当中，这意味着，安全环境在同一个线程执行的方法一直是有效的。这种情况下使用ThreadLocal是非常安全的，只要记得在处理完当前主体的请求以后，把这个线程清除就行了。当然，Spring Security自动帮你管理这一切了，你就不用担心什么了。

下面这段代码很清晰的说明了上面的实现策略：

{% codeblock lang:java %}
final class ThreadLocalSecurityContextHolderStrategy implements SecurityContextHolderStrategy {
	private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<SecurityContext>();

	public SecurityContext getContext() {
	    SecurityContext ctx = contextHolder.get();

	    if (ctx == null) {
	        ctx = createEmptyContext();
	        contextHolder.set(ctx);
	    }

	    return ctx;
	}
}
{% endcodeblock %}

当你从SecurityContextHolder中拿到SecurityContext后，你就可以通过下面的这段代码取到“当事人”或者说用户的信息：

{% codeblock lang:java %}
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();

if (principal instanceof UserDetails) {
	String username = ((UserDetails)principal).getUsername();
} else {
	String username = principal.toString();
}
{% endcodeblock %}

看完上面的代码，就要引入第二个关键组件Authentication：

Authentication是一个接口，代表着一种“认证”令牌，它里面存放着“当事人”的基本信息和是否认证的标志信息。框架在接受到认证请求时，会将新创建的Authentication对象会传递给AuthenticationManager.authenticate(Authentication)，由它来决定“认证”是否成功。

而一旦“认证”请求通过，Authentication通常都会存储在SecurityContext中，由SecurityContextHolder来管理。

除了主体，另一个Authentication提供的重要方法是getAuthorities()。 这个方法提供了GrantedAuthority对象数组。 毫无疑问，GrantedAuthority是赋予到主体的权限。这些权限通常使用角色表示，比如ROLE_ADMINISTRATOR或ROLE_HR_SUPERVISOR。

SecurityContextHolder，提供访问SecurityContext的方式。   
SecurityContext，保存Authentication信息，和请求对应的安全信息。    
Authentication，作为认证的令牌。    
GrantedAuthority，反应在应用程序范围内，赋予的权限。   

##Spring Security的认证(Authentication)过程

让我们考虑一种标准的验证场景，每个人都很熟悉的那种。   
一个用户想使用一个账号和密码进行登陆。       
系统（成功的）验证了密码对于这个用户名是正确的。   
这个用户对应的信息获取（他们的角色列表以及等等）。   
为用户建立一个安全环境。   
用户会执行一些操作，这些都是潜在被权限控制机制所保护的，通过对操作的授权，使用当前的安全环境信息。   

前三个项目执行了验证过程，所以我们可以看一下Spring Security的作用。   
用户名和密码被获得，并进行比对，在一个UsernamePasswordAuthenticationToken的实例中（它是Authentication接口的一个实例，我们在之前已经见过了）。   
这个标志被发送给一个AuthenticationManager的实例进行校验。   
AuthenticationManager返回一个完全的Authentication实例，在成功校验后。   
安全环境被建立，通过调用SecurityContextHolder.getContext().setAuthentication(...)，传递到返回的验证对象中。   

事实上，Spring Security并不介意你如何将Authentication对象放到SecurityContextHolder里。唯一关键的一点是 在AbstractSecurityInterceptor需要验证一个用户操作之前，SecurityContextHolder必须包含了一个表示了主体的Authentication。


待续。。。

