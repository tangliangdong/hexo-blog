---
layout:     post
title:      "window配置maven和spring环境"
date:       2016-07-16 13:00:00
author:     "Tang"
header-img: "img/spring.jpg"
catalog:    true
tags:
    - maven
    - spring
    - 配置
---

# maven 本地环境配置和spring配置

## 1. [maven官网](https://maven.apache.org/download.cgi)
下载maven压缩包

![image](../../../../img/maven-spring-img/maven1.jpg)

## 2. 解压到你要放置的地方，并配置环境变量

先将解压后的根目录路径配置到变量名为 `MAVEN_HOME` 
![image](../../../../img/maven-spring-img/maven2.jpg)
再定位到该路径下的bin文件夹，并将其配置到`Path`变量，`%MAVEN_HOME%\bin`
![image](../../../../img/maven-spring-img/maven3.jpg)

## 3. 新建两个文件夹
一个`repo`当做本地仓库，一个`proj`用来放置项目

## 4. 连接maven和本地仓库`repo`
进入之前解压的maven包下的conf文件夹
![image](../../../../img/maven-spring-img/maven4.jpg)

打开`settings.xml`文件

![image](../../../../img/maven-spring-img/maven5.jpg)

我们需要maven标记我们本地仓库`repo`的位置，但这一行被注释了，我们需要把他放出来，并换上repo的绝对路径

![image](../../../../img/maven-spring-img/maven6.jpg)

## 5. 从maven远程仓库下载资源
打开win命令行，`cd`到`proj`然后创建项目文件夹，此处以`blog`为例，然后在命令行输入以下代码

     mvn archetype:generate -DgroupId=com.itbegin.blog -DartifactId=blog -DarchetypeArtifactId=maven-archetype-webapp

`DgroupId`是java文件的包名，代表组织和整个项目的唯一标志。
`DartifactId`是具体项目的名称，它于groupId共同确定一个项目在maven repo中的位置。

按下`enter`后，如果是第一次，本地仓库会自己去远程仓库下载所需的东西到`repo`里面，以后就无需下载了。

#### 1. 此时，`blog`目录下会有
 - src 
 - pom.xml
其他都是没用的，打开pom.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.itbegin</groupId>
  <artifactId>blog</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding> 
  <version.spring>4.2.4.RELEASE</version.spring>
  </properties>
   
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.8.2</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.3.3</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${version.spring}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>${version.spring}</version>
  </dependency>
  <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-jdbc</artifactId>  
            <version>${version.spring}</version>  
      </dependency> 
  <dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.20</version>
  </dependency>
  <dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.186</version>
  </dependency>
  </dependencies>
  
  <build>
    <finalName>test</finalName>
    <directory>${my.target}</directory>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <source>1.7</source>
                <target>1.7</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.6</version>
            <configuration>  
                <encoding>UTF-8</encoding>  
            </configuration> 
        </plugin>
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
                <path>/</path>
                <contextReloadable>true</contextReloadable>
                <uriEncoding>UTF-8</uriEncoding>
            </configuration>
        </plugin>
    </plugins>
  </build>
</project>
```

直接用以上全部代码覆盖之前pom.xml的代码，当然有需要的可以改。

#### 2. 之后打开`proj\blog\src\main\webapp\WEB-INF` 下的`web.xml`,同样用下方的代码进行覆盖

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name></display-name>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext*.xml</param-value>
    </context-param>
</web-app>
```

#### 3. 创建资源文件
进入`proj\blog\src\main\resources`下，创建`applicationContext-jdbc.xml`，并粘贴以下内容。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 xmlns:jdbc="http://www.springframework.org/schema/jdbc" 
 xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans.xsd 
      http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-3.2.xsd 
">
   <bean id="dataSource" class="org.h2.jdbcx.JdbcConnectionPool"
        destroy-method="dispose">
        <constructor-arg>
            <bean class="org.h2.jdbcx.JdbcDataSource">
                <!-- 内存模式 -->
               <!--  <property name="URL" value="jdbc:h2:" />-->
                <!-- 文件模式 -->
                <property name="URL" value="jdbc:h2:c:\tmp\blog" />               
                <property name="user" value="sa" />
                <property name="password" value="sa" />
            </bean>
        </constructor-arg>
    </bean>
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">  
        <property name = "dataSource" ref="dataSource"/>  
    </bean>  
    <!-- 初始化数据表结构 -->
</beans>
```

#### 4. 创建所需的目录结构
在`WEB-INF`下创建`template`文件夹，里面放`.ftl`文件，用来显示view。

到`proj\blog\src\main`下,创建java的文件夹之后就会有如下所示的大致结构，

![image](../../../../img/maven-spring-img/maven7.jpg)

之后构建`proj\blog\src\main\java\com\itbegin`这样的路径，在这个目录下可以放置如下的文件夹
![image](../../../../img/maven-spring-img/maven8.jpg)

## 6. 启动内置的Tomcat
打开命令行，进入blog项目的根目录，输入`clean tomcat7:run`,便会自动下载以及启动Tomcat,在网址中输入`localhost:8080`即可访问

