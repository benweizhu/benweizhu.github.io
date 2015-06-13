---
layout: post
title: "Spring Web Security 实战"
date: 2015-06-12 21:12:38 +0800
comments: true
categories: 
---


As you probably know two major areas of application security are "authentication" and "authorization" (or "access-control"). These are the two main areas that Spring Security targets. "Authentication" is the process of establishing a principal is who they claim to be (a "principal" generally means a user, device or some other system which can perform an action in your application)."Authorization" refers to the process of deciding whether a principal is allowed to perform an action within your application. To arrive at the point where an authorization decision is needed, the identity of the principal has already been established by the authentication process. These concepts are common, and not at all specific to Spring Security.

HTTPBASICauthenticationheaders(anIETFRFC-basedstandard)
HTTPDigestauthenticationheaders(anIETFRFC-basedstandard)
HTTPX.509clientcertificateexchange(anIETFRFC-basedstandard)
Form-basedauthentication(forsimpleuserinterfaceneeds)

In many of the examples you will see (and in the sample) applications, we will often use "security" as the default namespace rather than "beans", which means we can omit the prefix on all the security namespace elements, making the content easier to read. 

The namespace is designed to capture the most common uses of the framework and provide a simplified and concise syntax for enabling them within an application. The design is based around the large-scale dependencies within the framework, and can be divided up into the following areas:
• Web/HTTP Security - the most complex part. Sets up the filters and related service beans used to apply the framework authentication mechanisms, to secure URLs, render login and error pages and much more.
• BusinessObject(Method)Security-optionsforsecuringtheservicelayer.
• AuthenticationManager-handlesauthenticationrequestsfromotherpartsoftheframework.
• AccessDecisionManager-providesaccessdecisionsforwebandmethodsecurity.Adefaultonewill be registered, but you can also choose to use a custom one, declared using normal Spring bean syntax.
• AuthenticationProviders - mechanisms against which the authentication manager authenticates users. The namespace provides supports for several standard options and also a means of adding custom beans declared using a traditional syntax.
• UserDetailsService - closely related to authentication providers, but often also required by other beans.


This provides a hook into the Spring Security web infrastructure. DelegatingFilterProxy is a Spring Framework class which delegates to a filter implementation which is defined as a Spring bean in your application context. In this case, the bean is named "springSecurityFilterChain", which is an internal infrastructure bean created by the namespace to handle web security. Note that you should not use this bean name yourself. Once you’ve added this to your web.xml, you’re ready to start editing your application context file. 