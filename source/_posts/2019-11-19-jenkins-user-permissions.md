---
title: jenkins执行脚本权限不足
date: 2019-11-19 14:31:20
description: "jenkins项目构建时执行的shell总会遇到权限不足的问题"
category: Learn
tags:
    - jenkins
---

>  jenkins默认用户为jenkins , 普通用户执行shell脚本,会缺失某些权限 

<!-- more -->

### 打开Jenkins的配置文件

将Jenkins的执行用户修改为 `root`

```bash
vim /etc/sysconfig/jenkins
```



### 修改jenkins用户为`root`，

```bash
JENKINS_USER="root"
```



### 修改Jenkins相关文件夹用户权限

```bash
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
```



### 重启Jenkins

```bash
service jenkins restart
# 或者
systemctl restart jenkins
```



### 查看Jenkins进程所属用户

```bash
ps -ef | grep jenkins
```



若是显示进程的用户是root，则表示修改成功。

![ps -ef | grep jenkins](1.png)



转载自 [Jenkins执行脚本报权限不足错误](https://www.blog.lijinghua.club/article/jenkins_权限不足)