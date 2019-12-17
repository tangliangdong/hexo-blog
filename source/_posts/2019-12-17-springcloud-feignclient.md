---
title: 服务消费者Feign访问springcloud服务提供者
date: 2019-12-17 09:59:13
category: Learn
tags:
    - springcloud
    - feign
---

`Feign`是一个声明式的伪`Http`客户端，它使得写`Http`客户端变得更简单。

使用`Feign`，只需要创建一个接口并注解，它具有可插拔的注解特性，可使用`Feign` 注解和`JAX-RS`注解，`Feign`支持可插拔的编码器和解码器，`Feign`默认集成了`Ribbon`，并和`Eureka`结合，默认实现了负载均衡的效果。

<!-- more-->

### Feign 简介

**`Feign` 具有如下特性：**

- 可插拔的注解支持，包括`Feign`注解和`JAX-RS`注解
- 支持可插拔的`HTTP`编码器和解码器
- 支持`Hystrix`和它的`Fallback`
- 支持`Ribbon`的负载均衡
- 支持`HTTP`请求和响应的压缩`Feign`是一个声明式的`Web Service`客户端，它的目的就是让`Web Service`调用更加简单。它整合了`Ribbon`和`Hystrix`，从而不再需要显式地使用这两个组件。`Feign`还提供了`HTTP`请求的模板，通过编写简单的接口和注解，就可以定义好`HTTP`请求的参数、格式、地址等信息。接下来，`Feign`会完全代理`HTTP`的请求，我们只需要像调用方法一样调用它就可以完成服务请求。

简而言之：`Feign`能干`Ribbon`和`Hystrix`的事情，但是要用`Ribbon`和`Hystrix`自带的注解必须要引入相应的`jar`包才可以。

结构：

![](1.png)

- springcloud  服务注册中心
- springserver  服务提供者 （服务名：SERVER-01）
- springclient  服务消费者



> 前一篇文章写过如何配置使用服务器注册中心，这一次主要是讲如何用 *Feign* 代替 *RestTemplate* 实现消费者访问提供者的接口
>
> **服务提供者不需要修改任何代码，主要是对服务消费者进行改造。**



# 配置Feign消费者

## 添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```



## 开启Feign

在工程的启动类中,通过`@EnableFeignClients` 注解开启Feign的功能：

```java
package com.example.springclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableEurekaClient
@EnableFeignClients
@SpringBootApplication
public class SpringclientApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringclientApplication.class, args);
    }
}
```

## 定义接口

通过`@FeignClient（"服务名"）`，来指定调用哪个服务。
比如在代码中调用了`SERVER-01`服务的 `/` 接口，`/` 就是调用：服务提供者项目：`springserver` 的 `index()` 方法，代码如下：

```java
package com.example.springclient.consumers;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import java.util.Map;

@FeignClient("SERVER-01")
public interface OpenClient {

    @PostMapping("open/index")
    Map<String, Object> getIndex(@RequestParam("name") String name, @RequestParam("age") String age);

}

```



## 消费方法

在Controller中调用提供者提供的方法

```java
package com.example.springclient.controllers;

import com.example.springclient.consumers.OpenClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("open")
public class OpenController {

    @Autowired
    private OpenClient openClient;

    @PostMapping("index")
    public Map<String, Object> getIndex(String name, String age){
        return openClient.getIndex(name, age);
    }
}
```



## 添加配置

```yaml
server:
  #服务端口
  port: 8011

spring:
  application:
    name: client-01

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

### 服务注册中心

![服务注册中心](2.png)

