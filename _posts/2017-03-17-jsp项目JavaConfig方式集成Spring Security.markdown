---
layout: post
title:  "jsp项目JavaConfig集成Spring Security"
date:   2017-03-17 14:44:26 +0800
categories: jekyll update
---
先构建一个最简单的jsp Web 项目，目录如下：

<img src="/images/20170317-001.png" style="width: 300px">

index.jsp内容如下：
{% highlight html %}
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Hello!</title>
</head>
<body>
    hello!
</body>
</html>
{% endhighlight %}
把项目打包部署到Tomcat , 启动Tomcat 然后打开浏览器访问 [http://localhost:8080/hello][jekyll-hello], 页面显示`hello!`, 说明项目运行成功。
下面我们开始引入Spring Security 保护这个请求地址，用户必须输入正确的用户名和密码才能访问。
首先在项目的pom.xml 文件中引入：
{% highlight xml %}
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
{% endhighlight %}

创建Spring Security Configuration,在项目中创建一个名为 SecurityConfig的类，内容如下：
```java
package com.lijingrong.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;

@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("user").password("password").roles("USER");
    }
}
```

类SecurityConfig会帮我们做如下事情：
*   应用中的每个URL需要登录才能访问
*   为应用生成一个登录页
*   允许用户用用户名：user,密码为：password 进行登录
*   允许用户退出登录
*   集成了如下Servlet API 方法
    +   [HttpServletRequest#getRemoteUser()][servlet-getRemoteUser]
    +   [HttpServletRequest.html#getUserPrincipal()][servlet-getUserPrincipal]
    +   [HttpServletRequest.html#isUserInRole(java.lang.String)][servlet-isUserInRole]
    +   [HttpServletRequest.html#login(java.lang.String, java.lang.String)][servlet-login]
    +   [HttpServletRequest.html#logout()][servlet-logout]

上面我们把Spring Security 的相关配置配好了，下面要把这些配置加载到我们的项目中去。
创建名为 SecurityWebApplicationInitializer 的类，继承 AbstractSecurityWebApplicationInitializer，具体代码如下：
```java
package com.lijingrong.config;

import org.springframework.security.web.context.AbstractSecurityWebApplicationInitializer;

public class SecurityWebApplicationInitializer extends AbstractSecurityWebApplicationInitializer {

    public SecurityWebApplicationInitializer(){
        super(SecurityConfig.class);
    }

}

```

类 `SecurityWebApplicationInitializer` 会帮我们做如下工作：
*   为应用中的每个URL 自动注册  springSecurityFilterChain Filter
*   创建一个 ContextLoaderListener 加载 `SecuritConfig`
把项目打包部署到Tomcat , 启动Tomcat 然后打开浏览器访问 [http://localhost:8080/hello][jekyll-hello], 页面会跳到登录页，如下图：

<img src="/images/20170317-002.png" style="width: 300px">

你用非正确的用户名:123,密码:456 登录，会得到如下提示：

<img src="/images/20170317-003.png" style="width: 300px">

用正确的用户名：user,密码:password 登录，会跳转到如下界面：

<img src="/images/20170317-004.png" style="width: 300px">

Spring Security 也为我们提供了退出登录功能

在index.jsp 添加如下代码：
```html
<form class="form-inline" action="logout" method="post">
    <input type="submit" value="Log out"/>
    <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
</form>
```

这时刷心之前访问的页面会出现一个Log out 按钮，点击按钮后会出现如下界面：

<img src="/images/20170317-005.png" style="width: 300px">

总结：

现在你应该知道怎么用Spring Security 来保护我们的Web 应用，我们的配置用的是 JavaConfig 而不是XML .

[项目完整源码下载][github-source]


[jekyll-hello]: http://localhost:8080/hello
[servlet-getRemoteUser]: https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#getRemoteUser()
[servlet-getUserPrincipal]:https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#getUserPrincipal()
[servlet-isUserInRole]:https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#isUserInRole(java.lang.String)
[servlet-login]:https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#login(java.lang.String,java.lang.String)
[servlet-logout]:https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#logout()
[github-source]:https://github.com/lijingrong/spring-security-demos
