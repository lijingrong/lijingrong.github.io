---
layout: post
title:  "Maven 远程部署Web项目到Tomcat7"
date:   2017-04-13 14:06:29 +0800
categories: jekyll update
---

1.在tomcat 安装目录下找到 conf/tomcat-users.xml 文件，添加用户和相关角色

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="tomcat" password="tomcat" roles="manager-gui,manager-script"/>
```
关于tomcat的用户和角色说明，详情见官网[Manager App HOW-TO][Manager]

2.在项目的pom.xml 添加 tomcat7-maven-plugin 

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.tomcat.maven</groupId>
      <artifactId>tomcat7-maven-plugin</artifactId>
      <version>2.2</version>
      <configuration>
        <url>http://localhost:8080/manager/text</url>
        <username>tomcat</username>
        <password>tomcat</password>
        <update>true</update>
      </configuration>
    </plugin>
  </plugins>
</build>
```

3.执行命令：
```
mvn tomcat:deploy
```

更多信息，可以到[Apache Tomcat Maven Plugin][tomcat-maven-home]

如果是在Intellij Idea 开发，可直接点击右边Maven 菜单栏进行运行，如下图：

<img src="/images/20170413-001.png" style="width: 300px">

注：次配置是 Apache Tomcat/7.0.77 运行成功，在 8.5.13 版本中没有运行成功

[Manager]:http://tomcat.apache.org/tomcat-7.0-doc/manager-howto.html
[tomcat-maven-home]:http://tomcat.apache.org/maven-plugin-2.2/index.html
