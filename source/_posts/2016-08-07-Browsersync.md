---
layout:     post
title:      "配置Browsersync"
subtitle:   "多浏览器多终端同时进行调试的工具"
date:       2016-08-08 22:00:00
author:     "Tang"
header-img: "img/browsersync.png"
catalog:    true
tags:
    - browsersync
    - 配置
    - npm
---

# Browsersync 简单配置

Browsersync是用于多浏览器多终端同时进行调试的工具，简单易用。
[Browsersync](http://http://www.browsersync.cn/)官网也有详细的讲解。

配置有很多方式，[官方文档](http://www.browsersync.cn/docs/)讲的也挺清楚的。

我就用最简单的一种方式，

## 1. 下载Browsersync

当然电脑上得有nodejs环境,因为要依赖nodejs，用npm包管理器下载Browsersync。

可以输入`node -v`可以查看安装的nodejs版本。

输入`npm -v`查看npm的版本，`-v`是`--version`的简写。

## 2. 安装Browsersync

打开命令行，我们通过输入`npm install -g browser-sync`，`-g`是全局安装browsersync的参数,在本机的任何位置都可以使用browsersync。

卸载可以通过输入`npm uninstall -g browser-sync`。

等待下载完成之后，我们可以输入`browser-sync --version`检查browsersync版本号。

## 3. 运行Browsersync

如果安装成功，我们可以`cd`到项目根目录下，输入`browser-sync start --server --files "**.css"`，就能看到远程访问和本地访问的ip地址。

###### win的node容易出问题，本人电脑在配置的时候也出现了问题，但别的电脑还是可以一步成功的。

之前启动browsersync时出现错误的问题，在询问老师后，得到了解决。

需要在环境变量的`Path`后加上`C:\Windows\System32\`，之后browsersync就能正常启动了，功能还是非常强大的。