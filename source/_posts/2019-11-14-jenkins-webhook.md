---
title: jenkins配置webhook自动拉取github打包
date: 2019-11-14 14:25:24
category: learn
toc: true
tags:
    - jenkins
    - webhook
---

当 *github* 上传代码后，主动通知 **jenkins** 去拉取代码，自动化构建项目。

<!-- more -->

先创建一个项目，用于自动拉取github项目，然后自动打包。

因为项目使用`pom.xml`文件构建的，我们就直接创建一个maven项目，选择 `构建一个maven项目` 

![my_second_github_job](1.png)

如果没有这个选项，则需要去`jenkins插件管理`中去安装一个 `Maven Integration插件`

![ 安装Maven Integration插件 ](2.png)



