---
title: docker 修改创建容器时的参数
toc: true
date: 2020-02-19 14:59:52
categories: 后端
tags:
    - docker
---



每次docker 创建容器的时候总是很快就建好了，但是之后会发现总是少写了参数，比如mysql的 `MYSQL_ROOT_PASSWORD`、tomcat和mysql的 `--link` 。但是删了容器重新来过也麻烦。但是直接进入到容器里吧，连 `vim`、`vi` 的编辑器都没有。但是我们还可以通过修改容器配置文件的方法来达到我们的目的。



### 方法一

- 停止容器 `docker stop <container ID>`
- 停止docker服务 `systemctl stop docker`
- 进入到 `/var/lib/docker/containers/<container ID>`，其下有两个文件
  - hostconfig.json
  - config.v2.json

比如修改容器的端口映射：

```bash
cd /var/lib/docker/containers/<container ID>`  #容器id

vim hostconfig.json

# 之前的端口映射
"PortBindings":{"8080/tcp":[{"HostIp":"","HostPort":"9001"}]}
# 前一个数字是容器端口, 后一个是宿主机端口。将宿主机的9001端口映射到容器的8080端口
```

- 启动docker服务(systemctl start docker)
- 启动容器(docker start <container id>)



**hostconfig.json**

![hostconfig.json](1.png)



**config.v2.json**

![config.v2.json](2.png)