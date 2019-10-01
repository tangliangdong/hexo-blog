---
title:  java客户端连接websocket
date: 2019-08-8 10:30:20
category: learn
toc: true
tags:
    - java
    - spring
    - websocket
---

### spring websocket 访问

#### websocket类

<!-- more -->

```java
@ClientEndpoint
public class ChatClientEndpoint {

    Session userSession = null;
    private MessageHandler messageHandler;

    public ChatClientEndpoint(URI endpointURI) {
        try {
            WebSocketContainer container = ContainerProvider
                    .getWebSocketContainer();
            container.connectToServer(this, endpointURI);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * Callback hook for Connection open events.
     *
     * @param userSession
     *            the userSession which is opened.
     */
    @OnOpen
    public void onOpen(Session userSession) {
        System.out.println("opening websocket");
        this.userSession = userSession;
    }

    /**
     * Callback hook for Connection close events.
     *
     * @param userSession
     *            the userSession which is getting closed.
     * @param reason
     *            the reason for connection close
     */
    @OnClose
    public void onClose(Session userSession, CloseReason reason) {
        System.out.println("closing websocket");
        this.userSession = null;
    }

    /**
     * Callback hook for Message Events. This method will be invoked when a
     * client send a message.
     *
     * @param message
     *            The text message
     */
    @OnMessage
    public void onMessage(String message) {
        if (this.messageHandler != null)
            this.messageHandler.handleMessage(message);
    }

    /**
     * register message handler
     *
     * @param msgHandler
     */
    public void addMessageHandler(MessageHandler msgHandler) {
        this.messageHandler = msgHandler;
    }

    /**
     * Send a message.
     *
     * @param message
     */
    public void sendMessage(String message) {
        this.userSession.getAsyncRemote().sendText(message);
    }

    /**
     * Message handler.
     *
     * @author Jiji_Sasidharan
     */
    public static interface MessageHandler {
        public void handleMessage(String message);
    }
}

```

#### java客户端访问文件

```java
public class ChatBot {

    static ChatClientEndpoint clientEndPoint;

    private final String url = "ws://192.168.1.222:3098";
    static {
        try {
            clientEndPoint = new ChatClientEndpoint(new URI(url));
        } catch (URISyntaxException e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args) throws Exception {
        ChatBot chatBot = new ChatBot();
        chatBot.getVersion();
    }

    public void getVersion(){
        clientEndPoint.addMessageHandler(new ChatClientEndpoint.MessageHandler() {
            public void handleMessage(String message) { // 接收websocket服务器的返回信息
                System.out.println("getVersion");
                System.out.println(message);
                Map<String, Object> result =  JSON.parseObject(message);
            }
        });
        
        Map<String, Object> messageObj = new HashMap<>();
        messageObj.put("namespace", "VCS/main");
        messageObj.put("request", "query.version");
        messageObj.put("msg_id", 123456);
        System.out.println(JSON.toJSONString(messageObj));

        clientEndPoint.sendMessage(JSON.toJSONString(messageObj)); // 向服务端发送json数据
        try {
            Thread.sleep(1000); // 线程睡1s是为了接收服务器返回的消息
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

---

#### spring service文件

`@Value("${vanwei.server.username}") ` 通过这种方式注入的值，无法在静态代码块执行的时候就获取到，因此如果一旦websocket连接断开，则需要重新连接websocket （`clientEndPoint = new ChatClientEndpoint(new URI(vanwei_url));`）

```java
@Service("VanweiService")
public class VanweiService {

    private static ChatClientEndpoint clientEndPoint;

    // application.properties 配置信息
    @Value("${vanwei.server.username}") 
    private String username;
    @Value("${vanwei.server.password}")
    private String password;
    @Value("${vanwei.server.url}")
    private String vanwei_url;

    public void getVersion(){
        try {
            clientEndPoint = new ChatClientEndpoint(new URI(vanwei_url));
        } catch (URISyntaxException e) {
            e.printStackTrace();
        }
        clientEndPoint.addMessageHandler(new ChatClientEndpoint.MessageHandler() { // 监听服务区返回的websocket消息
            public void handleMessage(String message) {
                System.out.println("getVersion");
                System.out.println(message);
                Map<String, Object> result =  JSON.parseObject(message);
                String reply = result.get("reply").toString();
                if( !"200".equals(reply) ){ // 如果获取不成功，即重新登陆
                    System.out.println("websocket登陆认证过期，重新进行登陆");
                    login();
                }
            }
        });

        Map<String, Object> messageObj = new HashMap<>();
        messageObj.put("namespace", "VCS/main");
        messageObj.put("request", "query.version");
        messageObj.put("msg_id", 123456);
        clientEndPoint.sendMessage(JSON.toJSONString(messageObj));

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

> 若是要使websocket始终保持连接，则需要定时发送心跳包