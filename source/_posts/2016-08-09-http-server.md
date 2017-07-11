---
layout:     post
title:      "配置http-server"
subtitle:   "基于nodejs的轻量级服务器"
date:       2016-08-09 13:00:00
author:     "Tang"
header-img: "img/http-server.jpg"
catalog:    true
tags:
    - http-server
    - 配置
    - npm
---

#### http-server [官网介绍](https://www.npmjs.com/package/http-server)

### 1. 安装 http-server

在window命令行输入`npm install http-server -g`，全局安装`http-server`,输入`http-server -v`查看版本号。

### 2. 启动 http-server

在命令行输入`http-server src`,src指的需要提供服务的路径。

然后会显示两个地址，一个本机地址，还有个是远程访问的地址，可以通过局域网访问。

### 3. 参数

我们可以在启动`http-server`服务时，通过添加`-p 端口号`，来更改服务器端口号，防止端口被占用。
