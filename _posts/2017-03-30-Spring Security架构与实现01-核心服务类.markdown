---
layout: post
title:  "Spring Security架构实现01-核心服务类"
date:   2017-03-30 14:25:29 +0800
categories: jekyll update
---

系统一般的登录认证逻辑是这样的：

  1.用户输入自己的用户名和密码，点击登录按钮

  2.程序用接收到的用户名查询存储的密码

  3.进行密码比较，相等则认证成功，否则提示用户重新登录

在Spring Security 如何体现上述的认证步骤的呢？下面我们探索下具体的实现设计。

为了实现上述步骤2，Spring Security 定义了 `UserDetailsService` 接口，此接口只定义了一个方法：
```java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
```
此方法的作用是根据用户名获取用户的详细信息，比如用户的密码。
有两个类实现了这个接口，分别是 InMemoryUserDetailsManager 和 JdbcUserDetailsManager
+ `InMemoryUserDetailsManager` 表示用户信息放在内存中
+ `JdbcUserDetailsManager` 表示用户信息放在数据库中。

步骤3的实现稍微复杂点，首先定义了接口 `AuthenticationProvider`,其中定义了一个用户认证的方法
```java
Authentication authenticate(Authentication authentication)
      throws AuthenticationException;
```
Spring Security 里多种实现类来提供不同认证方式，如果默认提供的不能满足我们需求可以自己进行实现，在这里我们只提及常用的几个：
+ DaoAuthenticationProvider 实现，这种实现在获取用户信息时会委托给 UserDetailsService 的具体实现
+ RememberMeAuthenticationProvider 实现，这种实现具体的应用场景就是登录时`记住我`的功能

有可能会有这样的一种场景，一个系统的登录认证模块比较复杂，一部分用户基于数据库认证，另一部分用户基于Ldap 认证，在 Spring Security 为了满足如上的场景，定义了 `AuthenticationManager` 接口来管理 AuthenticationProvider，此接口也只定义了一个方法：
```java
Authentication authenticate(Authentication authentication)
      throws AuthenticationException;
```
主要实现类为 `ProviderManager`, 此类的主要实现代码（此处不是完整代码，只是为了说明问题）：
```java
public class ProviderManager implements AuthenticationManager{
    private List<AuthenticationProvider> providers = Collections.emptyList();
    public ProviderManager(List<AuthenticationProvider> providers,
        AuthenticationManager parent) {
      this.providers = providers;
    }
    public Authentication authenticate(Authentication authentication){
      for (AuthenticationProvider provider : getProviders()) {
        if (!provider.supports(toTest)) {
          continue;
        }
        result = provider.authenticate(authentication);
        if (result != null) {
            break;
        }
      }
      return result;
    }
}
```

