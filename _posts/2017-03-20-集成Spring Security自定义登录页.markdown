---
layout: post
title:  "集成Spring Security自定义登录页"
date:   2017-03-20 19:50:29 +0800
categories: jekyll update
---
在开始之前，要创建一个基于JavaConfig 的Spring MVC 项目。
然后按照下面的步骤把Spring Security 集成到项目中。

1.在pom.xml 加入`Spring Securpty` 依赖

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>4.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>4.2.2.RELEASE</version>
</dependency>
```

2.创建类`SecurityConfig`

```java
package com.lijingrong.config;

import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/login").permitAll();
    }
}
```
此类的配置允许用户匿名访问 /login

3.创建类`SecurityWebApplicationInitializer`

```java
package com.lijingrong.config;

import org.springframework.security.web.context.AbstractSecurityWebApplicationInitializer;

public class SecurityWebApplicationInitializer extends AbstractSecurityWebApplicationInitializer {

    public SecurityWebApplicationInitializer(){
        super(SecurityConfig.class);
    }

}
```

4.在相应的MvcConfig配置类中加入如下代码：
```java
@Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/login").setViewName("customLoginPage");
        registry.setOrder(Ordered.HIGHEST_PRECEDENCE);
    }
```

5.在 WEB-INF/jsp/中加入customLoginPage.jsp,内容如下：

```html
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>自定义的登录页</title>
</head>
<body>
<h3>这是自定义的登录页面,你可以任意美化^_^</h3>
<div style="width: 300px">
    <form action="login" method="post">
        <fieldset>
            <legend>Please Login</legend>
            <c:if test="${param.containsKey('error')}">
                <div>
                    Invalid username and password.
                </div>
            </c:if>
            <c:if test="${param.containsKey('logout')}">
                <div>
                    You have been logged out.
                </div>
            </c:if>
            <label for="username">Username</label>
            <input type="text" id="username" name="username"/>
            <label for="password">Password</label>
            <input type="password" id="password" name="password"/>
            <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
            <div>
                <button type="submit" class="btn">Log in</button>
            </div>
        </fieldset>
    </form>
</div>

</body>
</html>

```

6.打包部署tomcat,访问 [http://localhost:8080/hello][jekyll-hello],浏览器会重定向到`http://localhost:8080/hello/login`,此时会出现我们自定义的登录界面，如下图：

<img src="/images/20170320-001.png" style="width: 300px">

[项目完整源码下载][github-source]


[jekyll-hello]: http://localhost:8080/hello
[servlet-getRemoteUser]: https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#getRemoteUser()
[servlet-getUserPrincipal]:https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#getUserPrincipal()
[servlet-isUserInRole]:https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#isUserInRole(java.lang.String)
[servlet-login]:https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#login(java.lang.String,java.lang.String)
[servlet-logout]:https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#logout()
[github-source]:https://github.com/lijingrong/spring-security-demos
