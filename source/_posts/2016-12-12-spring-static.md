---
layout:     post
title:      "Spring获取静态资源"
subtitle:   ".css,.js,.jpg,.png"
date:       2016-12-12 16:00:00
author:     "Tang"
header-img: "img/spring-static.jpg"
catalog:    true
tags:
    - spring
    - 经验
---

当我们使用视图解析器，将所有的url都分配到`dispatcher-servlet`处理，因此连`.css,.js.jpg`等文件都一并分配到这里去了，自然就获取不到这些静态资源。

```xml
<!-- web.xml -->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

### 配置dispatcher-servlet.xml

效率比较高的就是在`dispatcher-servlet.xml`配置资源。

#### `<mvc:default-servlet-handler/>`

对进入DispatcherServlet的URL进行筛查，如果发现是静态资源的请求，就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由DispatcherServlet继续处理。

一般Web应用服务器默认的Servlet名称是"default"，因此`DefaultServletHttpRequestHandler`可以找到它。如果你所有的Web应用服务器的默认Servlet名称不是"default"，则需要通过`default-servlet-name`属性显示指定：

```xml
<!-- dispatcher-servlet.xml -->
<mvc:default-servlet-handler default-servlet-name="所使用的Web服务器默认使用的Servlet名称" />
```

#### <mvc:resources />

> 允许静态资源放在任何地方，如WEB-INF目录下、类路径下等

```xml
<!-- dispatcher-servlet.xml -->
<!-- location是工程路径地址，mapping是映射后的访问地址 -->
<mvc:resources location="/resources/" mapping="/resources/**"/>
```

![img](../../../../img/spring-static/spring_static.jpg)

### jsp页面引入静态资源

因为映射的缘故，直接这样引入`<link rel="stylesheet" type="text/css" href="resources/bootstrap/css/bootstrap.css">`，并不能在所有页面都成功，我们有两种方式。

#### <% %>

```html
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+
            request.getServerName()+":"+request.getServerPort()+path+"/";
%>
<link rel="stylesheet" type="text/css" 
    href="<%=path%>/resources/bootstrap/css/bootstrap.css">
```

- *path* 是带项目名的路径。
- `href="/ActivityDemo/resources/bootstrap/css/bootstrap.css"`
- *pathBase* 是带服务器地址和项目名的路径
- `href="http://localhost:8080/ActivityDemo//resources/bootstrap/css/bootstrap.css"`

> 当然这种在jsp页面中嵌套java代码的方式会使得页面太混乱，不易维护。

#### <c:url value="" />

使用jstl的标准标签库，需导入下面两个jar包:

- jstl.jar
- standard.jar

```html
<!-- 使用jstl需在jsp页面顶部引入这句代码 -->
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<link rel="stylesheet" type="text/css" 
    href="<c:url value='/resources/bootstrap/css/bootstrap.css'/>">
<script type="text/javascript" src="<c:url value='/resources/js/jquery-2.2.4.min.js'/>"></script>
```

`value`属性最前面必须是`/`。

###### 注意引入 css 文件是`<link>` 的 `href`属性，引入 js 文件是`<script></script>` 的 `src`属性

显示的地址是带项目名的地址。`href="/ActivityDemo/resources/bootstrap/css/bootstrap.css"`

**建议在jsp页面不要使用<% %>，学习使用jstl标准标签库。**