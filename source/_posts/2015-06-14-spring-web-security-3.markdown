---
layout: post
title: "Spring Web Security 实战 (三) - 改变用户认证方式和安全性"
date: 2015-06-14 17:19:38 +0800
comments: true
categories: 
---

之前实现的表单提交方式来验证用户，实际上是不安全，http请求没有做任何的处理，表单内容以明文方式发送，在浏览器查看请求，得到结果就如下：

j_username:ben  
j_password:ben   
submit:Login  

而且，用户名和密码保存在xml或者properties方式保存在服务器端，也是不合理的。

##Http Basic 

参考：http://blog.itpub.net/23071790/viewspace-709367/

“在HTTP协议进行通信的过程中，HTTP协议定义了基本认证过程以允许HTTP服务器对WEB浏览器进行用户身份证的方法，当一个客户端向HTTP服务 器进行数据请求时，如果客户端未被认证，则HTTP服务器将通过基本认证过程对客户端的用户名及密码进行验证，以决定用户是否合法。客户端在接收到HTTP服务器的身份认证要求后，会提示用户输入用户名及密码，然后将用户名及密码以BASE64加密，加密后的密文将附加于请求信息中， 如当用户名为anjuta，密码为：123456时，客户端将用户名和密码用“：”合并，并将合并后的字符串用BASE64加密为密文，并于每次请求数据 时，将密文附加于请求头（Request Header）中。HTTP服务器在每次收到请求包后，根据协议取得客户端附加的用户信息（BASE64加密的用户名和密码），解开请求包，对用户名及密码进行验证，如果用 户名及密码正确，则根据客户端请求，返回客户端所需要的数据;否则，返回错误代码或重新要求客户端提供用户名及密码。”

{% codeblock lang:xml %}
<security:http>
    <security:intercept-url pattern="/**" access="ROLE_USER"/>
    <security:http-basic/>
</security:http>
{% endcodeblock %}

这种方式直接的表单要好点，但其实也不安全，用户名和密码可以通过BASE64反编码得到。

##使用数据库

如果你想使用数据库作为身份认证的数据来源，非常简单，只需要配置一个DataSource即可。

{% codeblock lang:xml %}
<authentication-manager>
	<authentication-provider>
		<jdbc-user-service data-source-ref="securityDataSource"/> 
	</authentication-provider>
</authentication-manager>
{% endcodeblock %}

这里的securityDataSource是Spring上下文种一个DataSource Bean，它指向一个含有Spring Security“标准用户数据库表”的数据库。

除了直接配置DataSource，你还可以配置一个Spring Security的JdbcDaoImpl的Bean，如下：

{% codeblock lang:xml %}
<authentication-manager>
	<authentication-provider user-service-ref='myUserDetailsService'/> 
</authentication-manager>

<beans:bean id="myUserDetailsService" class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
	<beans:property name="dataSource" ref="dataSource"/> 
</beans:bean>
{% endcodeblock %}
你肯定会问，这个“标准用户数据库表”的结构是什么样的。

默认的结构是有两张表："users"和"authorities"。Users表有三个字段username，password，enabled。Authorities表有两个字段username，authority

如果你不满意标准的表结构，你还可以实现AuthenticationProvider接口自定义实现。

{% codeblock lang:xml %}
<authentication-manager>
	<authentication-provider ref='myAuthenticationProvider'/>
</authentication-manager>
{% endcodeblock %}

##添加Password编码器

将密码以明文的方式存放在代码中或者是数据库中，始终是不安全的，AuthenticationProvider允许设置密码的加密编码。并且应该使用为此目的而设计的标准的安全hash算法，而不是一般的SHA或者MD5哈希算法。

{% codeblock lang:xml %}
<bean name="bcryptEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
<authentication-manager>
    <authentication-provider>
        <password-encoder ref="bcryptEncoder"/>
        <user-service>
            <user name="jimi" password="d7e6351eaa13189a5a3641bab846c8e8c69ba39f"
                  authorities="ROLE_USER, ROLE_ADMIN"/>
            <user name="bob" password="4e7421b1b8765d8f9406d87e7cc6aa784c4ab97f" authorities="ROLE_USER"/>
        </user-service>
    </authentication-provider>
</authentication-manager>
{% endcodeblock %}

Bcrypt是Spring文档里推荐的hash算法，它是一个跨平台的文件加密工具。由它加密的文件可在所有支持的操作系统和处理器上进行转移。它的口令必须是8至56个字符，并将在内部被转化为448位的密钥。（来自-百度百科）

##HTTPS

很多情况下我们依旧需要采用以表单的方式来实现身份认真，就必须要避免，表单数据，即用户名和密码在HTTP协议下以明文传输。

Spring Security可以非常方面的支持要求某些URL必须是在https协议下才可以访问。

{% codeblock lang:xml %}
<http>
<intercept-url pattern="/secure/**" access="ROLE_USER" requires-channel="https"/> 
<intercept-url pattern="/**" access="ROLE_USER" requires-channel="any"/>
...
</http>
{% endcodeblock %}

上面这段代码，当用户以http协议访问/secure/**，就会先跳转到https上。


参考资料：Spring Security Reference





