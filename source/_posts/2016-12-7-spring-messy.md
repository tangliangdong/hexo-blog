---
layout:     post
title:      "spring 解决乱码"
date:       2016-12-07 14:30:00
author:     "Tang"
header-img: "img/spring-messy.jpg"
catalog:    true
tags:
    - spring
    - 经验
---

### request.getParameter()乱码问题

#### 通过form表单提交的字段

```java
new String(req.getParameter("参数名").getBytes("ISO-8859-1"),"utf-8");
```

#### 通过url 获取的字段

`http://127.0.0.1:8080/prjdm-main/list?projectId=1&content=团队组织架构"`

> 因为直接http请求方式的话，使用的编码是iso-8859-1格式的，所以应该先以这种方式获取流然后转为utf-8格式，这样的话就直接可以显示正确的汉字了

```java
URLDecoder.decode(request.getParameter("参数名"),"UTF-8");
```