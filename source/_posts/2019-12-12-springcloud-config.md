---
title: springboot 2.x springcloud Eureka 注册
date: 2019-12-12 11:15:55
toc: true
category: Learn
tags:
    - springcloud
---

基于SpringBoot 2.x的Spring Cloud服务注册与发现

<!-- more -->

# 创建服务注册中心

## 创建springboot工程

1、在**[https://start.spring.io](https://start.spring.io/)**中创建

2、选择Maven Project、Java、2.2.2，添加 *Eureka server* 依赖

![](1.png)

3、点击Generate Project，解压下载的zip压缩包，再用Ideal打开。

## 添加注解

启动类添加 `@EnableEurekaServer` 注解

```java
package com.example.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class SpringcloudApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudApplication.class, args);
    }
}
```



## 添加配置

1、修改pom.xml如下，基本上不需要修改什么，主要是引入`spring-cloud-starter-netflix-eureka-server`包。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>springboot-integration</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.RELEASE</spring-cloud.version>
    </properties>

    <groupId>com.example</groupId>
    <artifactId>springboot-cloud</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>springcloud</name>
    <description>Demo project for Spring Cloud</description>

    <dependencies>
        <!-- Eureka Server包 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

2、在 `application.yml` 中添加以下配置 

```yaml
server:
  # 服务端口
  port: 8001
  
spring:
  application:
    #服务名
    name: service-01

eureka:
  instance:
    # eureka主机名，会在控制页面中显示
    hostname: localhost
  client:
    # 是否将自己注册到Eureka Server，默认为true
    # 由于当前这个应用就是Eureka Server，故而设为false
    registerWithEureka: false
    # 表示是否从Eureka Server获取注册信息，默认为true。因为这是一个单点的Eureka Server，
    # 不需要同步其他的Eureka Server节点的数据，故而设为false
    fetchRegistry: false
    serviceUrl:
      # eureka注册中心服务器地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```



## 启动服务

启动Spring Boot项目，在浏览器中输入http://localhost:8001 即可进入Eureka主页面。

![spring Eureka](3.png)



# 创建服务提供者

## 修改配置

提供者项目创建方式与注册中心服务器基本相同，只需做以下修改：

1、修改启动类注解@EnableEurekaServer为@EnableEurekaClient

2、将pom.xml文件中的  `spring-cloud-starter-netflix-eureka-server`换成 `spring-cloud-starter-netflix-eureka-client`，并加入如下的 *web* 包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>springboot-integration</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>springclient</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>springclient</name>
    <description>Demo project for Spring cloud client</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <!-- 不加入这个web包，该客户端启动会直接关闭 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- Eureka Client 服务提供方的包 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

3、修改application.yml

```yaml
server:
  #服务端口
  port: 8002

spring:
  application:
    name: server-01

eureka:
  instance:
    #eureka主机名，会在控制页面中显示
    hostname: localhost
    #eureka服务器页面中status的请求路径
    status-page-url: http://${eureka.instance.hostname}:${server.port}/index
  client:
    serviceUrl:
      #在注册中心中进行注册
      defaultZone: http://localhost:8001/eureka/
```

## 启动服务

启动项目成功后，即可在服务中心 **DS Replicas -> Instances currently registered with Eureka** 下发现此服务提供者了。

![启动服务后](4.png)
