---
title: spring restTemplate调用提供者方法
date: 2019-12-17 9:00:00
category: Learn
tags:
    - springcloud
    - restTemplate
---

>  RestTemplate 简化了发起HTTP请求以及处理响应的过程，并且支持REST

<!-- more -->

只需要引入spring-web包即可

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

---

先配置 `RestTemplate` 的 configuration 文件，

还要进行中文乱码的配置。

`@LoadBalanced` 声明 该 restTemplate 开启负载均衡

```java
package com.example.springclient.configuration;

import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.MediaType;
import org.springframework.http.converter.StringHttpMessageConverter;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;
import java.nio.charset.StandardCharsets;
import java.time.Duration;
import java.util.ArrayList;
import java.util.List;

@Configuration
public class RestTemplateConfig {

    @LoadBalanced
    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplateBuilder()
                .setConnectTimeout(Duration.ofSeconds(1))
                .setReadTimeout(Duration.ofSeconds(1))
                .build();
        // 防止RestTemplate获取的中文乱码
        restTemplate.getMessageConverters().add(new MappingJackson2HttpMessageConverter());

        StringHttpMessageConverter stringHttpMessageConverter = new StringHttpMessageConverter(StandardCharsets.UTF_8);
        stringHttpMessageConverter.setWriteAcceptCharset(true);

        List<MediaType> mediaTypeList = new ArrayList<>();
        mediaTypeList.add(MediaType.ALL);

        for (int i = 0; i < restTemplate.getMessageConverters().size(); i++) {
            if (restTemplate.getMessageConverters().get(i) instanceof StringHttpMessageConverter) {
                restTemplate.getMessageConverters().remove(i);
                restTemplate.getMessageConverters().add(i, stringHttpMessageConverter);
            }
            if(restTemplate.getMessageConverters().get(i) instanceof MappingJackson2HttpMessageConverter){
                try{
                    ((MappingJackson2HttpMessageConverter) restTemplate.getMessageConverters().get(i)).setSupportedMediaTypes(mediaTypeList);
                }catch(Exception e){
                    e.printStackTrace();
                }
            }
        }
        return restTemplate;
    }
}

```



`Controller`控制器

```java
package com.example.springclient.controllers;

import com.example.springclient.consumers.OpenClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.Map;

/**
 * @author Administrator
 * @data 2019/12/12
 * @time 15:45
 */
@RestController
@RequestMapping("open")
public class OpenController {

    @Autowired
    private RestTemplate restTemplate;

    @PostMapping("test")
    public String test(String name, String age){
        MultiValueMap<String, Object> map = new LinkedMultiValueMap();
        map.add("name", name);
        map.add("age", age);
        String response = restTemplate.postForObject("http://SERVER-01/open/index", map, String.class);
        return response;
    }
}
```

- RestTemplate的post参数不能使用MultiValueMap而不能使用HashMap
- SERVER-01 是服务提供者的服务名
- 还实现了**简单轮询负载均衡（RoundRobin）**







