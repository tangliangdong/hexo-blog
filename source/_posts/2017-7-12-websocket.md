---
title: websocket 使用
date: 2017-07-12 10:11:09
category:   Learn
tags:
    - websocket
---

websocket 是服务器和客户端能够双向通信的协议。而且是支持跨域的。

<!-- more  -->

在用jQuery Mobile做app的时候，老师要求在订单页用websocket连接后台。然后就去网上找websocket文档，发现用sockjs来实现的很多，因为websocket对浏览器兼容性不好，IE就只兼容10以上，这是无法接受的。所有就用sockjs来提交websocket的兼容性。

```
// 生成websocket连接的方法
function connect() {
   //链接SockJS 的endpoint 名称为"/endpointWisely"
   var socket = new SockJS('/endpointWisely');

   // var socket = new WebSocket("ws://localhost:9090/www");
   //使用stomp子协议的WebSocket 客户端
   stompClient = Stomp.over(socket);
   //链接Web Socket的服务端。
   stompClient.connect({}, function(frame) {
       setConnected(true);
       console.log('Connected: ' + frame);
       //订阅/topic/getResponse 目标发送的消息。这个是在控制器的@SendTo中定义的。
       stompClient.subscribe('/topic/getResponse', function(respnose) {
           showResponse(JSON.parse(respnose.body).responseMessage);
       });
   });
}
```

因为 `var socket = new SockJS('/endpointWisely');` 这句话采用的是ajax请求，但是ajax请求会有跨域的问题。

因此还是选择用最原始的方式来实现websocket。

***

### 服务端代码 `MyWebSocket.java`

```java
// MyWebSocket.java
package flybear.hziee.app.controller;

@ServerEndpoint("/websocket")
@Component  
public class MyWebSocket  {
    
    private static CopyOnWriteArraySet<MyWebSocket> webSocketSet = new CopyOnWriteArraySet<>();  
  
    private Session session;
    
    private static MyWebSocket instance;
    
    // 单例模式，获取唯一的websocket。
    public static MyWebSocket getSingleton() {
        if (instance == null) {                         //Single Checked
            synchronized (MyWebSocket.class) {
                if (instance == null) {                 //Double Checked
                    instance = new MyWebSocket();
                }
            }
        }
        return instance ;
    }
    
    // 与客户端产生连接的时候调用
    @OnOpen  
    public void onOpen (Session session){  
        this.session = session;  
        webSocketSet.add(this);  
        addOnlineCount();  
        System.out.println("有新链接加入!当前在线人数为" + getOnlineCount());  
    }  
    
    // 与客户端关闭连接的时候调用
    @OnClose  
    public void onClose (){  
        webSocketSet.remove(this);  
        subOnlineCount();  
        System.out.println("有一链接关闭!当前在线人数为" + getOnlineCount());  
    }  
  
    // 接收到客户端的消息时调用
    @OnMessage  
    public void onMessage (String message, Session session) throws IOException {  
        System.out.println("来自客户端的消息:" + message);  
        // 给客户端群发消息  
        for ( MyWebSocket item : webSocketSet ){
            item.sendMessage(message);
        }
    }
    
    // 服务器向客户端发送消息
    public void sendMessage (String message) throws IOException {  
        this.session.getBasicRemote().sendText(message);  
    }
    
    // 返回所有的websocket连接
    public CopyOnWriteArraySet<MyWebSocket> getWebSocketSet(){
        return webSocketSet;
    }
}
```

注解`@ServerEndpoint("/websocket")`可以将一个普通Java对象（POJO）使用`＠ServerEndpoint`作为WebSocket服务器的端点 。

客户端就能通过`var ws = new WebSocket("ws://localhost:9090/websocket");`发起对服务器的websocket连接。

***

### 客户端代码

```javascript
$(function() {
    // 与服务器产生websocket连接
    var ws = new WebSocket("ws://localhost:9090/websocket");
    
    // 通过websocket向服务器发送数据
    ws.onopen = function() {
        ws.send("我要发个消息到服务器");
        console.log('已经发送');
    };
    
    // 通过websocket接收服务器发送的数据
    ws.onmessage = function(evt) {
        var received_msg = evt.data;
        //处理你的数据
        console.log(evt);
    };
    
    // 与服务器关闭连接时调用
    ws.onclose = function() {
        alert("连接已关闭...");
    };
    
    // 发生错误时调用
    ws.onerror = function() {
        alert("好像发生错误了...");
    };
});
```

***

如果要在别的类，给客户端群发消息，就不能简单把`MyWebSocket.java`对象`new`出来，因为每个对象里都有存session。我们只能通过获取 在客户端连接服务器后产生的实例。

因此可以想到用单例模式。

> 单例对象的类必须保证只有一个实例存在

这是我们想要的。

```java
public void sendNews(){
    MyWebSocket myWebSocket = MyWebSocket.getSingleton();
    try {
        myWebSocket.sendMessage("你好世界");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
这样就能实现在别的类，给客户端发送消息。


