---
layout: post
title: "Spring Web Security 实战 (二) - 自定义登录页面"
date: 2015-06-14 15:45:34 +0800
comments: true
categories: 
---
上一章节结尾，我们实现了Spring web security的最小实现，拦截所有url，采用表单登录，用户名和密码采用inMemory方式保存。

你可能在想，这个登录页面从哪里来的，我们并没有直接去实现一个页面。事实上，正是因为我们没有设置登录页面，Spring Security就自动生成了一个，该页面是的Spring Security处理登录的标准页面。

不过，我们肯定是不会想要使用Spring Security给我们提供的标准页面，除非项目刚启动，处于demo阶段，否则，肯定采用自定义页面，那么如何做到呢？

在form-login标签提供了一个属性login-page，允许指定用户自定义的登录页面，于是，实现方式如下：

首先定义一个登录页面JSP文件，里面的form表单和标准页面一样：

{% codeblock lang:html %}
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Spring MVC Security</title>
</head>
<body>
<form action="/spring-mvc-security/j_spring_security_check" method="POST">
    <table>
        <tbody>
        <tr>
            <td>User:</td>
            <td><input type="text" name="j_username" value=""></td>
        </tr>
        <tr>
            <td>Password:</td>
            <td><input type="password" name="j_password"></td>
        </tr>
        <tr>
            <td colspan="2"><input name="submit" type="submit" value="Login"></td>
        </tr>
        </tbody>
    </table>
</form>
</body>
</html>
{% endcodeblock %}


然后定义一个对应的RequestMapping：

{% codeblock lang:java %}
@Controller
public class LoginController {
	@RequestMapping(value = "/login", method = GET)
	public String login() {
		return "login";
	}
}
{% endcodeblock %}

最后一步，就是配置http标签的内容，有两点，第一，设置form-login的属性login-page为登录页面的url，第二，设置login页面的访问权限为IS_AUTHENTICATED_ANONYMOUSLY，必须允许非授权用户可以访问，否则怎么登录呢。代码如下：

{% codeblock lang:xml %}
<security:http>
    <security:intercept-url pattern="/login" access="IS_AUTHENTICATED_ANONYMOUSLY"/>
    <security:intercept-url pattern="/**" access="ROLE_USER"/>
    <security:form-login login-page="/login"/>
    <security:logout/>
</security:http>
{% endcodeblock %}

当然，你也可以这么写，3.1之后，允许多个http标签的定义。不过这么做，相当于完全绕过Spring Security的filterChain，不在Spring Security相关管理之下。
{% codeblock lang:xml %}
<http pattern="/css/**" security="none"/> 
<http pattern="/login" security="none"/>
{% endcodeblock %}

##匿名登录

你肯定会问，这和IS_AUTHENTICATED_ANONYMOUSLY有什么区别？这里参考：http://www.mossle.com/docs/auth/html/ch107-anonymous.html

”匿名登录，即用户尚未登录系统，系统会为所有未登录的用户分配一个匿名用户，这个用户也拥有自己的权限，不过他是不能访问任何被保护资源的。

设置一个匿名用户的好处是，我们在进行权限判断时，可以保证SecurityContext中永远是存在着一个权限主体的，启用了匿名登录功能之后，我们所需要做的工作就是从SecurityContext中取出权限主体，然后对其拥有的权限进行校验，不需要每次去检验这个权限主体是否为空了。这样做的好处是我们永远认为请求的主体是拥有权限的，即便他没有登录，系统也会自动为他赋予未登录系统角色的权限，这样后面所有的安全组件都只需要在当前权限主体上进行处理，不用一次一次的判断当前权限主体是否存在。这就更容易保证系统中操作的一致性。“

这种一致性，统一了业务层代码的实现，将匿名用户看做是用户的一种类型，只是访问权限不一样而已。关于它们的优缺点http://www.mossle.com/docs/auth/html/ch107-anonymous.html的最后一部分有介绍。

设置登录后的跳转页面，form-login还提供了两个属性default-target-url和always-use-default-target。如果用户进入登录页面是因为要访问某个受限制的资源，当用户登录后，就会回到该访问资源，但是如果不是，default-target-url，就起到作用，指定了登录后的跳转页面，它的默认值是”/“。always-use-default-target的含义就很简单了，是否永远跳转，不管用户之前访问的资源。

##处理登出

因为在xml配置中添加了security:logout，所以Spring Security也默认提供了一个登出URL："/j_spring_security_logout"。

##获得用户名

让Spring注入Principal即可

{% codeblock lang:java %}
@Controller
public class HomeController {
	@RequestMapping(value = "/home", method = GET)
	public String home(Principal principal, Model model) {
		model.addAttribute("userName", principal.getName());
		return "home";
	}
}
{% endcodeblock %}


参考资料：   
1.Spring security reference
