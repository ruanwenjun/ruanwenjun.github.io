---
layout: post
title: Spring Security
tags: [Spring]
---
目录：
- [Hello World](#hello world)


Spring Security是一款安全框架，能够在web请求级别和方法调用级别提供认证和授权功能
- 使用Filter保护web请求并限制URL级别的访问
- 使用Spring AOP保护方法调用，借助于对象代理和使用通知，能够确保只有具备适当权限的用户才能访问安全保护的方法。


## Hello World

(1) 导包

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>4.0.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>4.0.2.RELEASE</version>
</dependency>
```



(2) 在web.xml中配置过滤器

```
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
注意：这里的filter-name必须是springSecurityFilterChain,不能修改

其中需要加载security.xml的配置文件，由于配置了springmvc,所以就一并加载了

(3) 书写springSecurityConfig.xml配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<b:beans xmlns="http://www.springframework.org/schema/security"
         xmlns:b="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/security
		http://www.springframework.org/schema/security/spring-security.xsd">
    <http/>
    <user-service>
        <user name="user" password="password" authorities="ROLE_USER"></user>
    </user-service>
</b:beans>
```
至此，一个最基本的spring security框架已经配置完成，启动服务器会自动跳转到login页面，这个页面是框架自动生成的

![image](https://ruanwenjun.github.io/images/springsecurity/login.png)

必须使用上面配置文件中的user,password才能登陆。由于配置了springmvc，登陆成功后会跳转到index.jsp

![image](https://ruanwenjun.github.io/images/springsecurity/index.png)

页面代码如下，向其中加入一个logout 按钮可以实现登出

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib uri="http://java.sun.com/jstl/core_rt" prefix="c" %>

<html>
<head>
    <title>home</title>
    <link href="<c:url value="/css/main.css"/>" rel="stylesheet">
</head>
<body>
<h1>
    Hello, <c:out value="${pageContext.request.remoteUser}"></c:out>
</h1>
<c:forEach items="${spittleList}" var="spittle">
    <li id="spittle_${spittle.id}">
        <a href="${pageContext.request.contextPath}/spittle/${spittle.id}">
            <span style="font-size: large"><c:out value="${spittle.message}"/></span><br/>
        </a>
        <span><c:out value="${spittle.date}"/></span>
    </li>
</c:forEach>

<a href="/spittle/register">Register</a>
<form action="/logout" method="post">
    <input type="submit" value="logout">
    <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
</form>
</body>
</html>
```
![image](https://ruanwenjun.github.io/images/springsecurity/logout.png)

---

[参考资料一](https://springcloud.cc/spring-security-zhcn.html#release-numbering)
