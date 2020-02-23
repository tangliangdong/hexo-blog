---
title: linux curl模拟 GET/POST 调试接口
toc: true
date: 2020-02-22 22:02:57
categories: 后端
tags:
    - linux
---



## curl GET请求

```bash
curl http://localhost:8088/project/getProjectList?pageNum=1&pageSize=10
```

若是想看更加详细的请求信息，我们可以再加上 -v 参数

```bash
curl http://localhost:8088/project/getProjectList?pageNum=1&pageSize=10 -v
```



## curl POST请求

>   我们可以用 `-X POST` 来申明我们的请求方法，默认是GET请求，用 `-d` 参数，来传送我们的参数。

{% note danger %}
所以，我们可以用 `-X PUT` 和 `-X DELETE` 来指定另外的请求方法。
{% endnote %}

```bash
curl -d "mobile=18257146867&authCode=123456&CID=0&platform=1" http://localhost:8088/login/loginByAuthCode
```



如果要传输json格式参数，则可以用 `-H` 参数来申明请求的 `header`

```bash
curl localhost:9999/api/send -X POST -H "Content-Type:application/json" -d '"username":"xiaotang","content":"hello world"'
```



我们可以用 `-H` 来设置更多的 `header` 比如，用户的 `token` 之类的。



## curl POST 上传文件

可以用 `-F "file=@__FILE_PATH__"` 的格式，传输文件即可。命令如下：

```bash
curl localhost:8088/upload/file -F "file=@/mnt/1463739554391.jpg" -F "fileType=2" -F "location=杭州"
```

每个参数都要单独用 -F 命令单独写



---

参考自 [curl 模拟 GET\POST 请求，以及 curl post 上传文件](https://blog.csdn.net/FungLeo/article/details/80703365) 