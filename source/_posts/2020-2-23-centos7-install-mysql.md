---
title: 更新centos7安装mysql5.7
toc: true
date: 2020-02-23 07:59:07
categories:
tags:
    - linux
---

### 1.添加Mysql5.7仓库

```bash
sudo rpm -ivh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

<!-- more -->

### 2. 确认mysql仓库是否添加成功

```bash
sudo yum repolist all | grep mysql | grep enabled
```

如果展示像下面,则表示成功添加仓库:

```bash
mysql-connectors-community/x86_64  MySQL Connectors Community    enabled:     51
mysql-tools-community/x86_64       MySQL Tools Community         enabled:     63
mysql57-community/x86_64           MySQL 5.7 Community Server    enabled:    267
```



### 3. 开始安装Mysql5.7

```bash
sudo yum -y install mysql-community-server
```



### 4. 启动mysql

-   启动

```bash
sudo systemctl start mysqld
```

-   设置系统启动时自动启动

```bash
sudo systemctl enable mysqld
```

-   查看启动状态

```bash
sudo systemctl status mysqld
```



### 5. Mysql的安全设置

CentOS上的root默认密码可以在文件/var/log/mysqld.log找到，通过下面命令可以打印出来

```bash
cat /var/log/mysqld.log | grep -i 'temporary password'
```

登录后需要修改root密码

```bash
alter user 'root'@'localhost' identified by '123456';
```

>   修改用户密码

```bash
update mysql.user set authentication_string=password('123qwe') where user='root' and Host = 'localhost';
```

---

执行下面命令进行安全设置，这个命令会进行设置root密码设置，移除匿名用户，禁止root用户远程连接等

```bash
mysql_secure_installation
```



### 6. 创建用户、赋予权限

-   创建远程登录用户

```bash
CREATE USER 'root'@'%' IDENTIFIED BY '123456';
```

-   赋予该用户所有的权限

```bash
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION;
```



### 7. 设置数据库编码为utf8

-   打开配置文件

```bash
sudo vim /etc/my.cnf
```

-   在[mysqld]，[client]，[mysql]节点下添加编码设置

```bash
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
```

-   重启Mysql即可

```bash
sudo systemctl restart mysqld
```

