---
layout:     post
title:      "springboot框架模板"
date:       2017-3-24 01:50:00
author:     "Tang"
header-img: "img/baidu-map.jpg"
catalog:    true
tags:
    - spring
    - springboot
---

[github模板地址](https://github.com/tangliangdong/springboot-template)

### 模板的目录结构

![](../../../../img/springboot/1.png)

 - 第一个 
    - `Application.java` *springboot* 的主程序，启动的入口
 - conf
    - `BeansConfig.java`
    - `WebMvcConfig.java` 过滤器
 - controller 页面控制器
 - druid
    - `DruidAutoConfiguration.java`
    - `DruidProperties.java`
 - interceptor
    - `CoreInterceptor.java` 拦截器
 - mapper 数据库映射文件，但 xml文件 在 *resources/mapper* 文件夹下，需在 `application.properties` 进行配置
 - model 
 - service 业务类
 - util 工具类


> 项目所有的配置文件都在 `/src/main/resources/application.properties` 中



