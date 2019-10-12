---
title: 解决springboot的websocket无法注入service的问题
date: 2019-10-12 09:36:01
category: learn
toc: true
tags:
    - springboot
    - websocket
---



### 使用main方法启动的内置tomcat方法

#### springboot启动类

在启动类给websocket拦截器注入应用上下文，以供拦截中获取 *service* 使用

```java
package com.gonghui.intelligentization;

@SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})
@ServletComponentScan
@ImportResource({"classpath:dubbo.xml"})
public class CorpApplication{

    public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(CorpApplication.class, args);
        WebSocketHandshakeInterceptor.setApplicationContext(applicationContext);
    }

}
```



#### websocket拦截器

```java
package com.gonghui.intelligentization.global.interceptors;

import com.gonghui.intelligentization.api.AccountApi;
import com.gonghui.intelligentization.dto.AccountDto;
import com.gonghui.intelligentization.service.AiWebSocketHandler;
import com.gonghui.pay.common.emun.LogTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.http.server.ServletServerHttpRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.server.HandshakeInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.Map;

/**
 * @author tangliangdong
 * @data 2019/9/27
 * @time 14:11
 */
public class WebSocketHandshakeInterceptor implements HandshakeInterceptor {
    
    private static ApplicationContext applicationContext;

    // 在启动类中调用
    public static void setApplicationContext(ApplicationContext context){
        applicationContext = context;
    }

    @Override
    public void afterHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Exception e) {
        System.out.println("After Handshake");
    }

    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Map<String, Object> attributes) throws Exception {
        LogTemplate.LogForInfo("before Handshake");
        if (request instanceof ServletServerHttpRequest) {
            ServletServerHttpRequest servletRequest = (ServletServerHttpRequest) request;
            HttpSession session = servletRequest.getServletRequest().getSession();
            if (session != null) {
                HttpServletRequest req = ((ServletServerHttpRequest) request).getServletRequest();
                String account = req.getParameter("account");

                //使用account区分WebSocketHandler，以便定向发送消息
                AccountDto accountDto = (AccountDto) session.getAttribute(AiWebSocketHandler.WEBSOCKET_USERINFO);
                if (accountDto == null) {
                    // 获取accountApi
                    AccountApi accountApi = applicationContext.getBean(AccountApi.class);
                    accountDto = accountApi.getAccount(account);
                }
                attributes.put(AiWebSocketHandler.WEBSOCKET_USERINFO, accountDto);
            }
        }
        return true;
    }
}
```



#### 遇到的问题

部署到测试环境，

```java
AccountApi accountApi = applicationContext.getBean(AccountApi.class);
```

直接报错，因为测试环境不是用的内置tomcat，不会调用Springboot的启动类，所以 `applicationContext`自然为空，自然会报错。



----

### 部署到Tomcat启动应用，获取service的方法

### 将 WebSocketHandshakeInterceptor 添加到spring的应用上下文中

> 使用 WebSocketConfig 中注入 WebSocketHandshakeInterceptor 
>
> WebSocketConfig 相当于是 xml配置文件



#### 1、方法一 （通过@Bean）

##### websocket配置文件

> 必须加上`@Configuration`注解，`Spring`才能统一管理当前的拦截器实例。

```java
package com.gonghui.intelligentization.config;

import com.gonghui.intelligentization.global.interceptors.WebSocketHandshakeInterceptor;
import com.gonghui.intelligentization.service.AiWebSocketHandler;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;
import org.springframework.web.socket.handler.TextWebSocketHandler;

/**
 * @author Administrator
 * @data 2019/9/27
 * @time 14:13
 */
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {


    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        //1.注册WebSocket
        // webSocketHandshakeInterceptor()调用下面@Bean里的方法
        registry.addHandler(webSocketHandler(), "/websocket")
			.addInterceptors(webSocketHandshakeInterceptor())
			.setAllowedOrigins("*");

        //2.注册SockJS，提供SockJS支持(主要是兼容ie8)
        registry.addHandler(webSocketHandler(),"/websocket")
            .addInterceptors(webSocketHandshakeInterceptor())
            .setAllowedOrigins("*")
            .withSockJS();
    }

    @Bean
    public TextWebSocketHandler webSocketHandler() {
        return new AiWebSocketHandler();
    }

    // 注入
    @Bean
    public WebSocketHandshakeInterceptor webSocketHandshakeInterceptor(){
        return new WebSocketHandshakeInterceptor();
    }

}
```



> 之前 WebSocketConfig 配置了 `WebSocketHandshakeInterceptor`，springboot启动一直报错，`WebSocketHandshakeInterceptor`不能被重写



```
***************************
APPLICATION FAILED TO START
***************************

Description:

The bean 'webSocketHandshakeInterceptor', defined in class path resource [com/gonghui/intelligentization/config/WebSocketConfig.class], could not be registered. A bean with that name has already been defined in file [E:\tangliangdong\gfp-intelligentization-platform\corp\target\classes\com\gonghui\intelligentization\global\interceptors\WebSocketHandshakeInterceptor.class] and overriding is disabled.
```



因为在 `WebSocketHandshakeInterceptor`类上加了 `@Component`，因此不能再次注入了



##### websocket拦截器（游离在spring应用上下文环境之外）

```java
package com.gonghui.intelligentization.global.interceptors;

import com.gonghui.intelligentization.api.AccountApi;
import com.gonghui.intelligentization.dto.AccountDto;
import com.gonghui.intelligentization.service.AiWebSocketHandler;
import com.gonghui.pay.common.emun.LogTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.http.server.ServletServerHttpRequest;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.server.HandshakeInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.Map;

/**
 * @author tangliangdong
 * @data 2019/9/27
 * @time 14:11
 */
public class WebSocketHandshakeInterceptor implements HandshakeInterceptor {

    // 已经将 WebSocketHandshakeInterceptor 接入spring的上下文环境了，所以可以直接注入
    @Autowired
    private AccountApi accountApi;

    @Override
    public void afterHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Exception e) {
        System.out.println("After Handshake");
    }

    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Map<String, Object> attributes) throws Exception {
        LogTemplate.LogForInfo("before Handshake");
        if (request instanceof ServletServerHttpRequest) {
            ServletServerHttpRequest servletRequest = (ServletServerHttpRequest) request;
            HttpSession session = servletRequest.getServletRequest().getSession();
            if (session != null) {
                HttpServletRequest req = ((ServletServerHttpRequest) request).getServletRequest();
                String account = req.getParameter("account");

                //使用userName区分WebSocketHandler，以便定向发送消息
                AccountDto accountDto = (AccountDto) session.getAttribute(AiWebSocketHandler.WEBSOCKET_USERINFO);
                if (accountDto == null) {
                    accountDto = accountApi.getAccount(account);
                }
                attributes.put(AiWebSocketHandler.WEBSOCKET_USERINFO, accountDto);
            }
        }
        return true;
    }
}
```



#### 2、方法二 （通过 @Component 实现注入）



```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

	// 在WebSocketHandshakeInterceptor加上了@Component，已经成功加入到应用上下文，可以直接注入
    @Autowired
    private WebSocketHandshakeInterceptor webSocketHandshakeInterceptor;
	
    @Autowired
    private AiWebSocketHandler aiWebSocketHandler;

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
    
        // 1.注册WebSocket
        // 直接使用 @Autowired注入的类
        registry.addHandler(aiWebSocketHandler, "/websocket")
                .addInterceptors(webSocketHandshakeInterceptor)
                .setAllowedOrigins("*");

        // 2.注册SockJS，提供SockJS支持(主要是兼容ie8)
        registry.addHandler(aiWebSocketHandler, "/websocket")
                .addInterceptors(webSocketHandshakeInterceptor)
                .setAllowedOrigins("*")
                .withSockJS();
    }

}
```



websocket拦截器类上要加上

```java
@Component
```



----

在此感谢公司带我的 CTO 雷老板 ヾ(๑╹◡╹)ﾉ"



