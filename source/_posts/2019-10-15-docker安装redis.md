---
title: 2019-10-15-docker安装redis
date: 2019-10-15 11:25:26
category: learn
toc: true
tags:
    - docker
    - redis
---



### 下载拉取Redis镜像

```shell
docker pull redis
```

![](1.png)



### 运行docker

```shell
docker run --name some-redis -d redis
```

其它启动方式：

启动redis实例并指定**端口**：

```shell
docker run --name redis -d -p 6379:6379 redis
```

启动redis实例，指定**端口**和**密码**：
```shell
docker run --name redis -d -p 6379:6379 redis --requirepass "admin"
```

