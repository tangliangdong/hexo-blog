---
title: nginx反向代理(Reverse Proxy)
toc: true
date: 2020-01-08 13:38:06
categories: 后端
tags:
    - nginx
---



反向代理（Reverse Proxy）方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

<!-- more -->

## 概念

**正向代理**：一个位于客户端和原始服务器之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。

**反向代理**：以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。



![正向代理](1.png)



![反向代理](2.png)



简单来说，

- 正向代理就是**代理服务器**代理的是多个客户端

- 反向代理就是代理服务器代理的是多个服务端

## 反向代理的好处

- 可以起到保护网站安全的作用，因为任何来自Internet的请求都必须先经过代理服务器。
- 通过缓存静态资源，加速Web请求。
- 实现负载均衡。顺便说下，目前市面上，主流的负载均衡方案，硬件设备有F5，软件方案有四层负载均衡的LVS，七层负载均衡的Nginx、Haproxy等。



## nginx反向代理实战

在 http 节点下，添加 `upstream` 节点，添加 tomcat 集群

```
http {
	
	upstream tomcats {
        server 127.0.0.1:9001;
        server 127.0.0.1:9002;
    }
}
```

upstream 还可以为每个设备设置状态值，这些状态值的含义分别如下：

- down：表示单前的server暂时不参与负载.
- weight：默认为1.weight越大，负载的权重就越大。
- max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误.
- fail_timeout: max_fails次失败后，暂停的时间。
- backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

**例子：**

	upstream tomcats{
		server 127.0.0.1:9001 down;
		server 127.0.0.1:9002 backup;
		server 127.0.0.1:9003 weight=2;
		server 127.0.0.1:9004 max_fails=2 fail_timeout=60s;   
	}


负载均衡分配策略：

- nano（轮询）：upstream按照轮询（默认）方式进行负载，每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。虽然这种方式简便、成本低廉。但缺点是：可靠性低和负载分配不均衡。

- weight（权重）：指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。总和为10

- ip_hash（访问ip）：每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

  配置只需要在upstream中加入 ip_hash; 即可。

  ```
  upstream tomcats {
        ip_hash;
        server 127.0.0.1:9001;
        server 127.0.0.1:9002;
  }
  ```

  

- fair（第三方）：按后端服务器的响应时间来分配请求，响应时间短的优先分配。与weight分配策略类似。

  ```
  upstream tomcats {
        server 127.0.0.1:9001;
        server 127.0.0.1:9002;
        fair;
  }
  ```

  

---



在http server 节点下配置location为Tomcat集群

```
http {
	server {
		location / {
            proxy_pass_header Server;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Scheme $scheme;
            proxy_pass http://tomcats;
        }
	}
}
```



## 启动nginx并访问

```bash
./sbin/nginx
```

{% tabs 选项卡 2 %}
<!-- tab server-01 -->
![server-01](3.png)
<!-- endtab -->
<!-- tab server-02 -->
![server-02](4.png)
<!-- endtab -->
<!-- tab server-03 -->
![server-03](5.png)
<!-- endtab -->
{% endtabs %}





## 参考链接

- [Nginx 反向代理详解](https://juejin.im/entry/57fb07b0816dfa0056c0ada8)

