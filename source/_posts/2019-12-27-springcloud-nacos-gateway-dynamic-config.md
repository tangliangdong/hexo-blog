---
title: Spring Cloud Gateway和Nacos实现动态路由
date: 2019-12-26 17:08:39
category: Learn
tags:
    - Gateway
    - Spring Cloud
    - Nacos
---



网关中有两个重要的概念，那就是路由配置和路由规则，路由配置是指配置某请求路径路由到指定的目的地址。而路由规则是指匹配到路由配置之后，再根据路由规则进行转发处理。
Spring Cloud Gateway作为所有请求流量的入口，在实际生产环境中为了保证高可靠和高可用，尽量避免重启,需要实现Spring Cloud Gateway动态路由配置。

<!-- more -->

Spring Cloud Gateway的官方文档并没有讲如何动态配置，查看 Spring Cloud Gateway的源码，发现`在org.springframework.cloud.gateway.actuate.GatewayControllerEndpoint`类中提供了动态配置的Rest接口，但是`需要开启Gateway的端点`，而且提供的功能不是很强大。通过参考和GatewayControllerEndpoint相关的代码，可以自己编码实际动态路由配置。

## 用IDE新建Maven工程

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.xiaotang</groupId>
    <artifactId>springcloud-gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springcloud-gateway</name>
    <description>Demo project for Spring Boot Gateway</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
        <spring-cloud-alibaba-dependencies.version>0.2.2.RELEASE</spring-cloud-alibaba-dependencies.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
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

            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba-dependencies.version}</version>
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

