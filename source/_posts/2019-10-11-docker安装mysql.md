---
title: docker安装mysql
date: 2019-10-11 15:44:44
category: learn
toc: true
tags:
    - docker
    - mysql
---



### 1. docker中搜索可用镜像

```shell
docker search mysql
```

![](1.png)

> 如果需要查看mysql可用的版本标签号，则需要去 [*docker register*](https://hub.docker.com/) 查看。

![](2.png)

### 2. 拉取MySQL镜像

```shell
docker pull mysql:5.7
```

![](3.png)

### 3. 查看本地mysql镜像

```shell
docker image ls mysql
```

![](4.png)

### 4. 生成并运行mysql容器

```shell
docker run --name mysql5.7 -e MYSQL_ROOT_PASSWORD=123456 -d -i -p 3306:3306 --restart=always  mysql:5.7 mysqld --lower_case_table_names=1
```

![](5.png)



以上参数的含义：



- --name mysql5.7  将容器命名为mysql5.7，后面可以用这个name进行容器的启动暂停等操作
- -e  MYSQL_ROOT_PASSWORD=123456 设置MySQL密码为123456
- -d  此容器在后台运行,并且返回容器的ID
- -i  以交互模式运行容器
- -p  进行端口映射，格式为`主机(宿主)端口:容器端口`
- --restart=always  当docker重启时，该容器自动重启
- --lower_case_table_names=1  数据库表名不区分大小写，在window下是不区分大小写的， 可能在本地运行是OK的，但是一旦到测试环境就不行了。



{% note danger %}
如果在创建容器的时候有些属性未设置，比如 `lower_case_table_names`，`MYSQL_ROOT_PASSWORD`, 可以去到

`/var/lib/docker/containers/<container id>` 文件目录下，可以修改下面两个文件的参数

- hostconfig.json
- config.v2.json

然后重启 docker，其中的参数才会生效，只是重启容器是无效的。

{% endnote %}



### 进入mysql容器

```shell
docker exec -ti mysql /bin/bash
```

![](6.png)



> 很多参数的修改都不用进入到容器中，而是在创建容器的时候就赋予了的。



---

参考自 [瑜戈的基于docker安装MySQL](https://juejin.im/post/5babba8e5188255c960c3c63)