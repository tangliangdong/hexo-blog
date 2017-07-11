---
layout:     post
title:      "spring图片上传"
date:       2017-3-10 20:00:00
author:     "Tang"
header-img: "img/photo-upload.jpg"
catalog:    true
tags:
    - java
    - spring
---

# spring 图片上传

### html

需要在form标签里加上`enctype="multipart/form-data"`，不然程序可能会出现如下的异常

> `org.apache.catalina.connector.RequestFacade cannot be cast to org.springframework.web.multipart.MultipartHttpServletRequest`

`method="post"`上传的文件类型也必须是`post`。

要限制上传的文件类型，可以给`<input type="file">`加上
- `accept="image/*"`表示允许上传所有的图片类型，
- `accept="image/gif, image/jpeg,image/png"`表示允许上传的文件仅限于`jpg,gif,png`。

```html
<form action="<c:url value='/public'/>" id="myForm" enctype="multipart/form-data" method="post">
    标题：<input type="text" name="title">
    地点：<input type="text" name="local">
    时间：<input type="text" name="time">
    人数：<input type="number"  name="total">
    <input type="file" name="file_data" accept="image/gif, image/jpeg,image/png">
    <button type="submit" id="public-btn">发布</button>
</form>
```

### Controller

> 在控制器有两种方式可以获取到提交的文件

#### 一、自动和参数对应

`@RequestParam("file_data") MultipartFile filedata)`

```java
@RequestMapping(value="public",method = RequestMethod.POST)
public String publicActivity(@RequestParam("file_data") MultipartFile filedata,
        String title,Model model,HttpServletRequest req,HttpServletResponse res) 
        throws IllegalStateException, IOException{

    String pathRoot = req.getSession().getServletContext().getRealPath("");
    String path="";
    HttpSession session = req.getSession();
    Person person = (Person)session.getAttribute("person");
    if(!filedata.isEmpty()){  
        String fileName = filedata.getOriginalFilename();
        // 获取图片的扩展名
        String extensionName = fileName
                .substring(fileName.lastIndexOf(".") + 1);

        // SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd-hh-mm");
        
        // 新的图片文件名 = 获取时间戳+"."图片扩展名
        // System.currentTimeMillis()用来获取当前时间的时间戳
        String newFileName = String.valueOf(System.currentTimeMillis())
                + "." + extensionName;

        // 获得文件类型（可以判断如果不是图片，禁止上传）  
        // 可以在前端限制文件类型
        String contentType=filedata.getContentType();

        // 获得文件后缀名称  
        // String imageName=contentType.substring(contentType.indexOf("/")+1);
        path = newFileName;

        // 将图片写入对应的位置
        filedata.transferTo(new File( pathRoot + path ));  
    }

    //...... 再将图片的路径存入数据库。
}
```


#### 二、通过HttpServletRequest获取

获取到`MultipartFile`后，再进行后续上传的操作。

```java
MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
MultipartFile filedata = multipartRequest.getFile("file");
```

### `dispatcher-servlet.xml`配置文件

> 配置multipartResolver来解析带enctype="multipart/form-data"属性的表单

** 如果不配置`multipartResolver`，那带属性`enctype="multipart/form-data"`的所有表单数据就都无法获取到数据 *

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">

    <!-- 文件上传头部编码，默认是iso-8859-1，注意defaultEncoding必须和用户的jsp的pageEncoding属性一致，以便能正常读取文件 -->
    <property name="defaultEncoding" value="utf-8"></property>

    <!-- maxUploadSize 是上传文件的大小，单位为字节 -->
    <property name="maxUploadSize" value="10485760000"></property>

    <!-- 读取文件到内存中最大的字节数，默认是1024 -->
    <property name="maxInMemorySize" value="40960"></property>
</bean>
```

需要引入`commons-fileupload-1.3.2.jar`



