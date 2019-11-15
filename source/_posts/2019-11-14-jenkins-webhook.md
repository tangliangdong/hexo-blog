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

## Github获取 Personal access tokens

`登陆github -> settings -> Developer settings -> Personal access tokens`

![获取Personal access tokens`](20.png)

需要选上图中的两个复选框，这样Jenkins才能配置`github服务器`

![](21.png)

确认后生成的 token要保存起来，这个只会显示一次。

然后我们就可以去配置Jenkins的`github服务器`。

## jenkins 配置 Configure System

![Github服务器配置](22.png)

添加凭据

![](23.png)

类型需要选择`Secret Text`，这里出现的 **Secret** 就是我们之前去github生成的 `Personal access tokens`，再写个描述，确认就ok了。

点击连接测试后，如果下方显示的是`Credentials verified for user tangliangdong, rate limit: 4994`表明连接成功了。



## 配置Maven项目

因为项目使用`pom.xml`文件构建的，我们就直接创建一个maven项目，选择 `构建一个maven项目` 

![my_second_github_job](1.png)

如果没有这个选项，则需要去`jenkins插件管理`中去安装一个 `Maven Integration插件`，然后重启jenkins就能使用了。

![ 安装Maven Integration插件 ](2.png)



接下来就要带着我们的 **github仓库地址** 到处粘贴了，

```
https://github.com/tangliangdong/user-dev.git
```

![GitHub项目Url](3.png)



![源码管理](4.png)

 Credentials 需要github登陆的凭据。

源码库浏览器必须指定，输入的url还是github仓库地址，不带 `.git`

![添加凭据](5.png)



![构建触发器 webhook](6.png)

当我们向`user-dev` 仓库提交代码后，github会主动通知我们的jenkins去拉取代码，进行一次构建。



在 **构建环境** 中 `Use secret text(s) or file(s)`，选择的就是之前添加过的 `Github access token`

![ Use secret text(s) or file(s) ](7.png)



构建前执行 maven的命令 

```bash
clean package -P test
```

![ Pre Steps ](8.png)



构建失败后邮件通知。

![ E-mail Notification ](9.png)



## 配置 Github Webhook

`进入 Github项目Settings -> Webhook`，添加项目的 Webhook

![](10.png)

添加Jenkins的Webhook地址，因为这个是Github接收到提交的请求后，去通知Jenkins网站来拉取代码，因此这里的配置Jenkins代码肯定是要在公网能访问到的，但我当时是在我的树莓派上配置的，因此需要用诸如**花生壳**这类的内网映射工具来将本地的IP端口映射到公网上，才能让github访问到。webhook地址如下，直接在访问jenkins访问地址后面加上`/github-webhook`就行了。

```text
http://2m7t216256.wicp.vip:12006/github-webhook/
```

![](11.png)

绿色表示可以使用了，

![](12.png)

我们向 `user-dev` 提交代码，就能看到Jenkins在自动拉取代码进行构建了。