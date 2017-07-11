---
layout:     post
title:      "springMVC基础配置"
subtitle:   "springMVC和mybatis整合"
date:       2016-11-25 23:00:00
author:     "Tang"
header-img: "img/springmvc-configuration.jpg"
catalog:    true
tags:
    - spring
    - 配置
---

### 配置`web.xml`

> web.xml文件是用来初始化配置信息：比如Welcome页面、servlet、servlet-mapping、filter、listener、启动加载级别等。

`web.xml`在`WEB-INF`目录下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns="http://xmlns.jcp.org/xml/ns/javaee" 
    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
    http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd">

  <!-- 项目名称 -->
  <display-name>myspring</display-name>

  <!-- 项目默认的首页 -->
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>

  <!-- 配置spring的字符集过滤，解决中文插入数据库的乱码问题 -->
  <filter>  
      <filter-name>characterEncodingFilter</filter-name>  
      <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>  
      <init-param>  
          <param-name>encoding</param-name>  
          <param-value>UTF-8</param-value>  
      </init-param>  
      <init-param>  
          <param-name>forceEncoding</param-name>  
          <param-value>true</param-value>  
      </init-param>  
  </filter>
  <filter-mapping>  
      <filter-name>characterEncodingFilter</filter-name>  
      <url-pattern>/*</url-pattern>  
  </filter-mapping>

  <!-- 配置spring的前端控制器 -->
  <!-- spring默认配置文件是/WEB-INF/[servlet名字]-servlet.xml -->
  <servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
    <!-- 或者通过增加这段配置来指定要初始化的dispatcher。 -->
    <!-- <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/dispatcher-servlet.xml</param-value>
    </init-param> -->
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

### 配置`dispatcher-servlet.xml`

`dispatcher-servlet.xml`在`WEB-INF`目录下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
    
    <!-- 注解驱动 -->
    <mvc:annotation-driven />
    <!-- 第三方的库，如bootstrap -->
    <mvc:resources location="/resources/" mapping="/resources/**"/>
    
    <!-- 在指定包下 扫描组件bean -->
    <context:component-scan base-package="edu.hziee.controllers" />
    <context:component-scan base-package="edu.hziee.dao" />
    
    <!-- 连接数据库 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://127.0.0.1:3306/javaee2016?characterEncoding=utf8"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
    
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!-- 在spring中加入数据映射器接口 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="edu.hziee.mappers"></property>
        <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
    </bean>

    <!-- 视图解析器 -->
    <!-- 将jsp页面放在 WEB-INF/views 目录下 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```

还需要导入jar包，mybatis-3.4.1.jar

- spring依赖包
- mybatis依赖的包
- commons-logging-1.2.jar
- mysql-connector-java-5.1.38-bin.jar
- mybatis-spring-1.3.0.jar
- jstl.jar
- standard.jar

需要`build Path`到项目，还要拷贝到`WEB-INF/lib`目录下。







