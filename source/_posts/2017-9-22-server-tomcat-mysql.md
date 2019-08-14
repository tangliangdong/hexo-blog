---
title: centos7服务器 配置Tomcat和mysql
date: 2017-09-22 21:36:18
category: Learn
toc: true
tags:
    - 服务器
    - 配置
---

## 前言： 

> * 搜索有关配置服务器的信息时
>   * 一定要指明服务器系统的类别（Window,Centos,Ubuntu），
>   * 还要指明系统的版本号（Centos6,Centos7），不同的版本的系统之间的差别也很大。
> * 最好搜索最近一年的，太久远的博文可能就滞后了。

<!-- more -->

还要稍早写的一篇 👉👉👉 [centos7配置apache，添加域名](http://tangliangdong.github.io/2017/09/22/2017-9-22-centos-configuration/)

## centos7 安装、配置mysql数据库

```
yum install mysql
yum install mysql-server
yum install mysql-devel
```

安装mysql和mysql-devel都成功，但是安装mysql-server失败

CentOS 7 版本将MySQL数据库软件从默认的程序列表中移除，用mariadb代替了

MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可。开发这个分支的原因之一是：甲骨文公司收购了MySQL后，有将MySQL闭源的潜在风险，因此社区采用分支的方式来避开这个风险。MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品。

```
yum install mariadb-server mariadb
```

mariadb数据库的相关命令是：

```
systemctl start mariadb  // 启动MariaDB

systemctl stop mariadb  // 停止MariaDB

systemctl restart mariadb  // 重启MariaDB

systemctl enable mariadb  // 设置开机启动
```

启动数据库后，就可以正常使用了

```
mysql -u root -p
```

为了方便使用，需要在本地使用 Navicat 连接服务器上的数据库。

1. 可能会碰到没有权限的问题，可能需要firewall 防火墙开放3306端口

```
systemctl start firewalld  // 开启防火墙

firewall-cmd --zone=public --add-port=3306/tcp --permanent // 将3306端口添加到防火墙

firewall-cmd --reload  // 防火墙重新配置
```

2. 还可能碰到这个问题 

```
Host 'xxx.xx.xxx.xxx' is not allowed to connect to this MySQL server
```

![Nacicat 连接数据库失败](server-mysql-tomcat.png)

> 可能是一种安全预防措施，可以尝试添加新的管理员帐户：admin 123456

```
mysql> CREATE USER 'admin'@'localhost' IDENTIFIED BY '123456';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;
mysql> CREATE USER 'admin'@'%' IDENTIFIED BY '123456';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' WITH GRANT OPTION;
```

这样就可以用新的管理员账号连接mysql了。

![Navicat 配置数据库连接信息](server-mysql-tomcat2.png)

## centos7 安装、配置 Tomcat

### 安装JDK

安装Tomcat之前需要确保已经安装了 JDK

输入 `java -version` 来判断是否已经安装了java

如果没有安装，则本地先下载JDK，再上传到服务器的 `/usr/local/` 目录

[下载jdk-8u144-linux-x64.rpm](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

```
// 执行 rpm包安装
rpm -ivh jdk-8u111-linux-x64.rpm
```

检查java版本

```
java -version
```

### 安装Tomcat

* 下载 [apache-tomcat-8.0.46](http://tomcat.apache.org/download-80.cgi)，解压之后，再上传到 `/usr/local/` 目录

* 创建用户组，并添加用户到用户组

```
groupadd tomcat
usermod -a -G tomcat root
```

* 修改权限

```
chown -R tomcat:root apache-tomcat-8.5.9
```

* 启动Tomcat

```
/usr/local/apache-tomcat-8.0.46/bin/startup.sh
```

> 当然每次启动Tomcat都要进入这个文件夹，很麻烦，我们可以将Tomcat配置为系统服务，直接通过 `systemctl start/stop/restart tomcat` 操作Tomcat。

详细请参考👉👉👉 [Centos7下添加Tomcat为系统服务](http://blog.csdn.net/zuoshoucuoai/article/details/53610558)

* 配置 Tomcat端口号

在 `/usr/local/apache-tomcat-8.0.46/conf/server.xml`

```xml
<!-- 配置Tomcat的端口号的，默认是8080，因为服务器的默认端口是80，这里我们就改成80 -->
<Connector port="80" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

> 我前一篇写 [centos7配置apache，添加域名](http://tangliangdong.github.io/2017/09/22/2017-9-22-centos-configuration/) 里面的Apache也是80端口，两个现在一起无法一起启动，因为端口冲突了，所有先把Apache先关闭。

就算配置Tomcat是8080端口，两个一起启动，Apache的配置文件里 也不能加监听8080端口的代码 `Listen 8080` 。

* 配置 Tomcat绑定多个域名

编辑 `server.xml`，在 `<Service></Service>` 标签中添加：

```
<Host name="zzz.tangliangdong.me"  appBase="webapps" unpackWARs="true" autoDeploy="true">
　　  <Context path="" docBase="/usr/local/apache-tomcat-8.0.46/webapps"/>
　　  <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" prefix="localhost_access_log." suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>

<Host name="www.tangliangdong.me"  appBase="webapps" unpackWARs="true" autoDeploy="true">
　　  <Context path="" docBase="/usr/local/apache-tomcat-8.0.46/webapps/myhomepage"/>
　　  <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" prefix="localhost_access_log." suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
```

只需要改两个地方：

* name  ——  绑定的域名
* docBase  ——  域名映射的绝对路径

这样我们就可以通过 域名来访问服务器对应的路径了

> 当然这个域名需要经过 域名解析的，不然配置了也没用。

## 参考

> 本文部分摘录自以下博客，特别写出来，以表感谢。

* [centos7 mysql数据库安装和配置](https://www.cnblogs.com/starof/p/4680083.html)
* [主机'xxx.xx.xxx.xxx'不允许连接到这个MySQL服务器](https://stackoverflow.com/questions/1559955/host-xxx-xx-xxx-xxx-is-not-allowed-to-connect-to-this-mysql-server)
* [Centos7 下添加Tomcat为系统服务](http://blog.csdn.net/zuoshoucuoai/article/details/53610558)
* [tomcat实现多端口、多域名访问](https://www.bbsmax.com/A/Vx5M10yg5N/)


