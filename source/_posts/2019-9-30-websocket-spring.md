---
title: spring实现websocket
date: 2019-09-30 5:00:00
category: learn
toc: true
tags:
    - websocket
    - spring
---



### 三种方式配置websocket

- 使用Java提供的@ServerEndpoint注解实现
- 使用Spring提供的低层级WebSocket API实现
- 使用STOMP消息实现

------

> 要注意所需的包要兼容，不然可能无法初始化 `bean`

#### 一、使用Java提供的@ServerEndpoint注解实现

> 使用@ServerEndpoint注解监听一个WebSocket请求路径：

这里监听的是 `/websocket` ，后面的`{account}`是websocket建立连接时传递的参数，在方法里通过 `@PathParam("account") String account` 来获取

```java
package com.gonghui.intelligentization.service;

import javax.websocket.*;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;

/**
 * @author tangliangdong
 * @data 2019/9/30
 * @time 16:56
 */
@ServerEndpoint("/websocket/{account}")
public class WebSocketEndpoint {

   @OnOpen
    public void open(Session session, @PathParam("account")String account) {
        System.out.println("已连接");
        System.out.println("用户"+account+" 登录");
    }

    @OnMessage
    public void handleMessage(Session session, String message) throws IOException {
        session.getBasicRemote().sendText("Receive message: " + message);
    }

    @OnError
    public void error(Session session, Throwable t){
        t.printStackTrace();
    }

    @OnClose
    public void close() {
        System.out.println("连接关闭");
    }
}
```



`WebSocket` 配置类

```java
package com.gonghui.intelligentization.config;

import com.gonghui.intelligentization.service.WebSocketEndpoint;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

/**
 * websocket相关配置
 * @author Administrator
 * @data 2019/9/30
 * @time 16:56
 */
@Configuration
@EnableWebSocket
public class WebSocketConfig{

    @Bean
    public WebSocketEndpoint WebSocketEndpoint() {
        return new WebSocketEndpoint();
    }

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

---

#### 二、使用Spring提供的低层级WebSocket API实现

##### 1. 添加一个WebSocketHandler：

定义一个继承了AbstractWebSocketHandler类的消息处理类，然后自定义对”建立连接“、”接收/发送消息“、”异常情况“等情况进行处理

```java
/**
 * @author tangliangdong
 * @data 2019/9/27
 * @time 14:10
 */
@Component
public class AiWebSocketHandler extends TextWebSocketHandler {

	private final static Logger LOGGER = LoggerFactory.getLogger(WebSocketHandler.class);

    //已建立连接的用户
    private static final ArrayList<WebSocketSession> users = new ArrayList<WebSocketSession>();
 
    /**
     * 处理前端发送的文本信息
     * js调用websocket.send时候，会调用该方法
     *
     * @param session
     * @param message
     * @throws Exception
     */
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String username = (String) session.getAttributes().get("WEBSOCKET_USERNAME");
        // 获取提交过来的消息详情
        LOGGER.debug("收到用户 " + username + "的消息:" + message.toString());
        //回复一条信息，
        session.sendMessage(new TextMessage("reply msg:" + message.getPayload()));
    }
 
 
    /**
     * 当新连接建立的时候，被调用
     * 连接成功时候，会触发页面上onOpen方法
     *
     * @param session
     * @throws Exception
     */
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        users.add(session);
        String username = (String) session.getAttributes().get("WEBSOCKET_USERNAME");
        LOGGER.info("用户 " + username + " Connection Established");
        session.sendMessage(new TextMessage(username + " connect"));
        session.sendMessage(new TextMessage("hello wellcome"));
    }
 
    /**
     * 当连接关闭时被调用
     *
     * @param session
     * @param status
     * @throws Exception
     */
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        String username = (String) session.getAttributes().get("WEBSOCKET_USERNAME");
        LOGGER.info("用户 " + username + " Connection closed. Status: " + status);
        users.remove(session);
    }
 
    /**
     * 传输错误时调用
     *
     * @param session
     * @param exception
     * @throws Exception
     */
    @Override
    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
        String username = (String) session.getAttributes().get("WEBSOCKET_USERNAME");
        if (session.isOpen()) {
            session.close();
        }
        LOGGER.debug("用户: " + username + " websocket connection closed......");
        users.remove(session);
    }
 
    /**
     * 给所有在线用户发送消息
     *
     * @param message
     */
    public void sendMessageToUsers(TextMessage message) {
        for (WebSocketSession user : users) {
            try {
                if (user.isOpen()) {
                    user.sendMessage(message);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
 
    /**
     * 给某个用户发送消息
     *
     * @param userName
     * @param message
     */
    public void sendMessageToUser(String userName, TextMessage message) {
        for (WebSocketSession user : users) {
            if (user.getAttributes().get("WEBSOCKET_USERNAME").equals(userName)) {
                try {
                    if (user.isOpen()) {
                        user.sendMessage(message);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
                break;
            }
        }
    }

}
```



##### 2. 创建一个WebSocket握手拦截器

```java
package com.gonghui.intelligentization.global.interceptor;

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
@Component
public class WebSocketHandshakeInterceptor implements HandshakeInterceptor {

    @Autowired
    private static ApplicationContext applicationContext;

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
                // websocket连接传递的参数 account
                String account = req.getParameter("account");

                //使用userName区分WebSocketHandler，以便定向发送消息
                String userName = (String) session.getAttribute("SESSION_USERNAME");
                if (userName == null) {
                    userName = "system-" + session.getId();
                }
                attributes.put("SESSION_USERNAME", accountDto);
            }
        }
        return true;
    }
}
```



##### 3. Spring WebSocket的配置文件，采用的是注解的方式

```java
package com.gonghui.intelligentization.global.config;

/**
 * @author tangliangdong
 * @data 2019/9/27
 * @time 14:13
 */
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        //1.注册WebSocket

        registry.addHandler(webSocketHandler(), "/websocket").
                addInterceptors(new WebSocketHandshakeInterceptor()).setAllowedOrigins("*");

        //2.注册SockJS，提供SockJS支持(主要是兼容ie8)
		registry.addHandler(webSocketHandler(),"/websocket").
                addInterceptors(new WebSocketHandshakeInterceptor()).setAllowedOrigins("*").
                withSockJS();
    }

    @Bean
    public TextWebSocketHandler webSocketHandler() {
        return new AiWebSocketHandler();
    }
}
```




### 前端页面配置

```java
<!DOCTYPE html>
<html>
<head>
    <title>Java后端WebSocket的Tomcat实现</title>
</head>
<body>
    请输入：<textarea rows="3" cols="100" id="inputMsg" name="inputMsg"></textarea>
	<button οnclick="doSend();" id="button">发送</button>
</body>

<script type="text/javascript" src="http://cdn.bootcss.com/jquery/3.1.0/jquery.min.js"></script>
<script type="text/javascript" src="http://cdn.bootcss.com/sockjs-client/1.1.1/sockjs.js"></script>
<script type="text/javascript">

    var websocket = null;
    //判断当前浏览器是否支持WebSocket

    if ('WebSocket' in window) {
        //Websocket的连接
        websocket = new WebSocket("ws://localhost:9999/websocket?account=13567175138");//WebSocket对应的地址
    }
    else if ('MozWebSocket' in window) {
        //Websocket的连接
        websocket = new MozWebSocket("ws://localhost:9999/websocket");//SockJS对应的地址
    }
    else {
        //SockJS的连接
        websocket = new SockJS("ws://localhost:9999/websocket");    //SockJS对应的地址
    }

    websocket.onopen = onOpen;
    websocket.onmessage = onMessage;
    websocket.onerror = onError;
    websocket.onclose = onClose;
 
    function onOpen(openEvt) {
    	console.log("成功连接")
    	console.log(openEvt)
        //alert(openEvt.Data);
    }
 
    function onMessage(evt) {
        console.log(evt.data);
    }
    function onError() {
    	console.log("websocket出现错误")
    }
    function onClose() {
    	console.log("websocket关闭")
    }

    $(function(){
        $("#button").click(function(e){
            console.log(e)
            if (websocket.readyState == websocket.OPEN) {
                var msg = document.getElementById("inputMsg").value;
                console.log(msg)
                websocket.send(msg);//调用后台handleTextMessage方法
                console.log(("发送成功!"));
            } else {
                console.log(("连接失败!"));
            }
        })
    })
</script>
</html>
```

---



#### 三、使用STOMP消息实现

STOMP是一个用于C/S之间进行异步消息传输的简单文本协议, 全称是Simple Text Oriented Messaging Protocol。

> 其实STOMP协议并不是为WS所设计的, 它其实是消息队列的一种协议, 和AMQP,JMS是平级的。 只不过由于它的简单性恰巧可以用于定义WS的消息体格式。 目前很多服务端消息队列都已经支持了STOMP, 比如RabbitMQ, Apache ActiveMQ等。很多语言也都有STOMP协议的客户端解析库，像JAVA的Gozirra，C的libstomp，Python的pyactivemq，JavaScript的stomp.js等等。



#### 1、Spring配置STOMP消息

```java
package com.gonghui.intelligentization.config;

import com.gonghui.intelligentization.global.interceptors.WebSocketInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.converter.MessageConverter;
import org.springframework.messaging.handler.invocation.HandlerMethodArgumentResolver;
import org.springframework.messaging.handler.invocation.HandlerMethodReturnValueHandler;
import org.springframework.messaging.simp.config.ChannelRegistration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketTransportRegistration;

import java.util.List;

/**
 * websocket相关配置
 * @author Administrator
 * @data 2019/9/30
 * @time 16:56
 */
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    //用来注册Endpoint，“/gs-guide-websocket”即为客户端尝试建立连接的地址。
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // 需要 setAllowedOrigins("*")，不然无法访问
        registry.addEndpoint("/stomp-websocket")
            .setAllowedOrigins("*")
            .withSockJS();
    }

    @Override
    //实现WebSocketMessageBrokerConfigurer中的此方法，配置消息代理（broker）
    public void configureMessageBroker(MessageBrokerRegistry config) {
         //将"message"前缀绑定到MessageMapping注解指定的方法上。如"message/hello"被指定用greeting()方法来处理.
        config.setApplicationDestinationPrefixes("/message");
        
        //启用SimpleBroker，使得订阅到此"topic"前缀的客户端可以收到greeting消息.
        config.enableSimpleBroker("/ai", "/user");
        
        //点对点推送时，客户端订阅消息的前缀,/user为默认值，也需把 '/user' 添加到 enableSimpleBroker()中。
        config.setUserDestinationPrefix("/user");
    }

    @Override
    public void configureWebSocketTransport(WebSocketTransportRegistration registry){

    }

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        
    }

    @Override
    public void configureClientOutboundChannel(ChannelRegistration registration) {

    }

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {

    }

    @Override
    public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> returnValueHandlers) {

    }

    @Override
    public boolean configureMessageConverters(List<MessageConverter> messageConverters) {
        return false;
    }

}
```



#### 2、收发消息的控制器

```java
package com.gonghui.intelligentization.controller;

import com.alibaba.fastjson.JSONObject;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.annotation.SendToUser;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Controller;


import java.util.HashMap;
import java.util.Map;

/**
 * @author Administrator
 * @data 2019/10/8
 * @time 9:10
 */
@Controller
public class GreetingController {
    
    @Autowired
    private SimpMessagingTemplate simpMessagingTemplate;

    // @SendTo 群发推送
    @MessageMapping("/hello")
    @SendTo("/ai/greetings")
    public String hello(String account) {
        Map<String, Object> map = new HashMap<>();
        map.put("account", account);
        map.put("content", "hello "+ account);
        return JSONObject.toJSONString(map);
    }
    
    // SendToUser为点对点推送
    // 客户端需订阅 '/user/{account}/ai/message'
    @MessageMapping("/queue")
	@SendToUser("/ai/message")
    public void queue(String account) {
        Map<String, Object> map = new HashMap<>();
        map.put("account", account);
        map.put("content", "hello "+ account);
        simpMessagingTemplate.convertAndSendToUser(account, "/queue/greetings", map);
    }
}
```



#### 服务端主动向客户端进行一对一推送

- simpMessagingTemplate.convertAndSend(String destination, Object payload); // 群发
- simpMessagingTemplate.convertAndSendToUser(String user, String destination, Object payload); // 一对一
  - *user* 是发送的用户标识
  - *destination* 是发送路径
  - *payload* 是发送的消息



```java
package com.gonghui.intelligentization.service.websocket;

import com.gonghui.intelligentization.dto.AccountWebSocketDto;
import com.gonghui.intelligentization.dto.AiWebsocketAlarmDto;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Service;

/**
 * @author Administrator
 * @data 2019/10/8
 * @time 15:54
 */
@Service("AiWebsocketService")
public class AiWebsocketService {

    //提供给用户发送消息处理
    @Autowired
    private SimpMessagingTemplate simpMessagingTemplate;

    public void sendMessageToUser(String account, Map<String, Object> data) {
        System.out.println("给某个用户发送消息");
        // 实际发送的地址是 /user/{account}/ai/websocket
        // 客户端订阅的websocket地址为 /user/{account}/ai/websocket
        simpMessagingTemplate.convertAndSendToUser(account, "/ai/websocket", data);
    }
    
    public void sendMessageToUser(Map<String, Object> data) {
        System.out.println("给某个用户发送消息");
        // 实际发送的地址是 /user/{account}/ai/websocket
        // 客户端订阅的websocket地址为 /user/{account}/ai/websocket
        simpMessagingTemplate.convertAndSend("/ai/message", data);
    }
}
```



#### 4、前端页面 SockJS

```html
//使用SockJS和stomp.js来打开“gs-guide-websocket”地址的连接，这也是我们使用Spring构建的SockJS服务。
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/sockjs-client/1.1.4/sockjs.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/stomp.js/2.3.3/stomp.min.js"></script>
<script type="text/javascript">
    function connect() {
        var socket = new SockJS('http://localhost:9998/stomp-websocket');
        stompClient = Stomp.over(socket);
        stompClient.connect({}, function (frame) {
            //连接成功后的回调方法
            setConnected(true);
            console.log('Connected: ' + frame);
            //订阅/ai/greetings地址，当服务端向此地址发送消息时，客户端即可收到。
            stompClient.subscribe('/ai/greetings', function (greeting) {
                //收到消息时的回调方法，展示欢迎信息。
                console.log(JSON.parse(greeting.body).content);
            });
            
            // 订阅一对一发送消息接口， 'xiaotang' 为用户标识
            stompClient.subscribe('/user/xiaotang/ai/websocket', function (greeting) {
                //收到消息时的回调方法，展示欢迎信息。
                console.log(JSON.parse(greeting.body).content);
            });
        });
    }
    //断开连接的方法
    function disconnect() {
        if (stompClient !== null) {
            stompClient.disconnect();
        }
        setConnected(false);
        console.log("Disconnected");
    }
    //将用户输入的账号信息,使用STOMP客户端发送到"/message/hello"地址。它正是我们在GreetingController中定义的greeting()方法所处理的地址.
    function sendName() {
        stompClient.send("/message/hello", {}, JSON.stringify({'account': $("#account").val()}));
    }
</script>
```



---

要确保 `spring-websocket` 、`spring-messaging`与 `spring-core` 的版本号一致，不然会无法创建 *bean*

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-websocket</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-messaging</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>${spring.version}</version>
</dependency>
```



---

转载自：

- [Spring Boot中使用WebSocket总结（一）：几种实现方式详解](https://www.zifangsky.cn/1355.html)
- [使用spring-websocket包搭建websocket服务](https://blog.csdn.net/zsg88/article/details/76862495)

