---
title: jenkins配置构建失败邮件通知
date: 2019-11-14 14:42:26
category: learn
toc: true
tags:
    - jenkins
    - email
---

jenkins自动化构建邮件通知开发者功能配置

<!-- more -->

## jenkins系统配置

先进入 `Manage Jenkins --> Configure System`

![Configure System](1.png)



###  Jenkins Location  ->  系统管理员邮件地址

输入你的邮箱地址

![系统管理员邮件地址](2.png)

---

### Extended E-mail Notification ->  Default user E-mail suffix

![](3.png)

---

### 邮件通知

![](4.png)

1. SMTP服务器地址需要去对应的邮箱中查看，并需同时开启邮箱的SMTP功能。见下方汇总
2. 需开启SMTP认证，用户名就是邮箱号，密码需要去邮箱设置。
3. SMTP默认是465，也可以在邮箱设置SMTP服务的地方查看。
4. 最后，可以通过发送测试邮件测试配置，如果出现如图中 `Email was successfully sent` 信息，则说明邮件通知功能配置成功。

1、例如网易邮箱配置： `设置 -> POP3/SMTP/IMAP`

![网易邮箱SMTP服务](5.png)

腾讯企业邮开启**SMTP服务**并查看SMTP服务器地址：`设置 -> 收发信设置`

![腾讯企业邮SMTP服务](6.png)

可以看到邮箱SMTP服务的端口为465



2、网易邮箱设置客户端授权密码

![网易邮箱客户端授权码](7.png)

开启客户端授权码并进行设置就行了，如果之前已经设置过，直接输入到jenkins中就行了。

**腾讯企业邮客户端授权密码**

![腾讯企业邮客户端授权码](11.png)

开启安全登陆 -> 生成客户端专用密码（需保存下来，只显示一次）

---

## 针对项目中进行邮件通知配置

先新建一个项目

![](8.png)

在配置项目信息的地方，其他都不用管，拉到页面的最底下，找到 **构建后操作**，点击 **增加构建后的操作步骤**，选择 `E-mail Notification`

![](9.png)

输入需要接受通知的邮箱账号

![](10.png)



## 构建项目

点击 `Build now`，构建项目

![](12.png)

当项目构建失败时，就会发送邮件到项目配置的邮箱地址。

![](13.png)



