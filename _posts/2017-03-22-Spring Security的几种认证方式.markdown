---
layout: post
title:  "Spring Security的几种认证方式"
date:   2017-03-22 19:50:29 +0800
categories: jekyll update
---
在阅读本文前，如果读者还没有对`Spring Security` 有个大致的了解，建议先阅读这两篇文章
+   [基于JavaConfig集成Spring Security][javaConfig-SpringSecurity]
+   [集成Spring Security自定义登录页][customize-login]

这篇文章我们先介绍两种，一种是基于内存的认证，另一种是基于数据库的认证，通俗来讲就是用户名放在内存或者数据库中。

1.基于内存的认证
只需要在项目中的`SecurityConfig` 中添加如下内容：
```java
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("user").password("password").roles("USER");
    }
```
这段代码表达的意思是：
+   当前基于内存的认证
+   往内存中添加用户名为`user`,密码为`password`,角色为`USER` 的用户

2.基于数据库的认证
此处我们用 `HSQLDB` 数据库储存用户信息
首先把`SecurityConfig` 中的configureGlobal方法改为如下：
```java
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.jdbcAuthentication()
                .dataSource(dataSource)
                .passwordEncoder(new Md5PasswordEncoder());
    }
```
这段代码表达的意思是：
+   当前基于jdbc 数据库的的认证
+   密码Md5 加密存储
 
项目启动时利用`Spring-jdbc`加载如下sql脚本：

```sql
create table users(
 username varchar_ignorecase(50) not null primary key,
 password varchar_ignorecase(50) not null,
 enabled boolean not null
);
create table authorities (
 username varchar_ignorecase(50) not null,
 authority varchar_ignorecase(50) not null,
 constraint fk_authorities_users foreign key(username) references users(username)
);
create unique index ix_auth_username on authorities (username,authority);

INSERT INTO users (username,password,enabled) VALUES ('123456','e10adc3949ba59abbe56e057f20f883e',1);
INSERT INTO authorities (username,authority) VALUES ('123456','admin');
```

`mysql` 版脚本如下：

```sql
CREATE TABLE users (
  username VARCHAR(50) NOT NULL PRIMARY KEY,
  password VARCHAR(50) NOT NULL,
  enabled  BOOLEAN     NOT NULL
);
CREATE TABLE authorities (
  username  VARCHAR(50) NOT NULL,
  authority VARCHAR(50) NOT NULL,
  CONSTRAINT fk_authorities_users FOREIGN KEY (username) REFERENCES users (username)
);
CREATE UNIQUE INDEX ix_auth_username
  ON authorities (username, authority);

INSERT INTO users (username,password,enabled) VALUES ('123456','e10adc3949ba59abbe56e057f20f883e',1);
INSERT INTO authorities (username,authority) VALUES ('123456','admin');
```

[项目完整源码下载][github-source]


[javaConfig-SpringSecurity]: http://www.lijingrong.com/jekyll/update/2017/03/17/%E5%9F%BA%E4%BA%8EJavaConfig%E9%9B%86%E6%88%90Spring-Security.html
[customize-login]:  http://www.lijingrong.com/jekyll/update/2017/03/20/%E9%9B%86%E6%88%90Spring-Security%E8%87%AA%E5%AE%9A%E4%B9%89%E7%99%BB%E5%BD%95%E9%A1%B5.html
[github-source]:https://github.com/lijingrong/spring-security-demos/tree/master/authentication
