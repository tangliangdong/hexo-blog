---
layout:     post
title:      "freemarker自动生成java、jsp"
date:       2017-3-16 21:00:00
author:     "Tang"
header-img: "img/freemarker.jpg"
catalog:    true
tags:
    - java
    - freemarker
---

# freemarker自动生成java文件

### 什么是freemarker?

> FreeMarker是一个模板引擎，一个基于模板生成文本输出的通用工具，使用纯Java编写 

### FreeMarker特性

 - 能够生成各种文本：HTML、XML、RTF、Java源代码等等 
 - 易于嵌入到你的产品中：轻量级；不需要Servlet环境 
 - 插件式模板载入器：可以从任何源载入模板，如本地文件、数据库等等 
 - 你可以按你所需生成文本：保存到本地文件；作为Email发送；从Web应用程序发送它返回给Web浏览器

> 需要 freemarker.jar

模板文件均为.ftl后缀的文件

```java
// ServiceGenerator.java
package test;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.io.OutputStreamWriter;
import java.io.Writer;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import vo.User;
import freemarker.template.Configuration;
import freemarker.template.Template;
import freemarker.template.TemplateException;
import freemarker.template.TemplateExceptionHandler;

public class ServiceGenerator {

    public static void main(String[] args) {
        
        List<User> list = new ArrayList<User>();
        // 需要先创建User的类
        User u = new User();
        u.setId(1);
        u.setUsername("张三");
        list.add(u);
        User u1 = new User();
        u1.setId(2);
        u1.setUsername("李四");
        list.add(u1);
        try {
            new ServiceGenerator().gen("User","x_user","UserService.java");
            new ServiceGenerator().createJsp("hello world",list,"hello.jsp");
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TemplateException e) {
            e.printStackTrace();
        }
    }

    /**
     * 用模板创建service代码
     * @param pojo
     * @param tablename
     * @param filename
     * @throws IOException
     * @throws TemplateException
     */
    public void gen(String pojo,String tablename,String filename) 
                                        throws IOException, TemplateException{
        // 通过Freemaker的Configuration读取相应的ftl
        Configuration cfg = new Configuration();
        // 设定去哪里读取相应的ftl模板文件
        cfg.setDirectoryForTemplateLoading(
                new File("/Users/18072/Workspaces/MyEclipse for Spring 2014/sshTest/src/test1"));   
        cfg.setDefaultEncoding("UTF-8");
        cfg.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW_HANDLER);

        // 在模板文件目录中找到名称为 service.ftl 的文件
        Template temp = cfg.getTemplate("service.ftl");  

        // 填入模板需要的数据
        Map<String, Object> root = new HashMap<String, Object>();
        root.put("POJO", pojo);
        root.put("table_name", tablename);

        // 设置文件的生成目录
        File dir = new File("/Users/18072/Workspaces/MyEclipse for Spring 2014/sshTest/src/service");
        if(!dir.exists()){
            dir.mkdirs();
        }
        
        OutputStream fos = new  FileOutputStream( new File(dir, filename)); //java文件的生成目录   
        Writer out = new OutputStreamWriter(fos);
        // 通过Template可以将模板文件输出到相应的流
        temp.process(root, out);

        fos.flush();
        fos.close();
        System.out.println("gen code success!");
    }
    
    /**
     * 用模板创建jsp文件
     * @param title
     * @param users
     * @param filename
     * @throws IOException
     * @throws TemplateException
     */
    public void createJsp(String title,List<User> users,String filename) 
            throws IOException, TemplateException{
        // 通过Freemarker的Configuration读取相应的ftl
        Configuration cfg = new Configuration();
        // 设定去哪里读取相应的ftl模板文件
        cfg.setDirectoryForTemplateLoading(
        new File("/Users/18072/Workspaces/MyEclipse for Spring 2014/sshTest/src/test1"));   
        cfg.setDefaultEncoding("UTF-8");
        cfg.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW_HANDLER);
        
        // 在模板文件目录中找到名称为 service.ftl 的文件
        Template temp = cfg.getTemplate("hello.ftl");  
        
        // 填入模板需要的数据
        Map<String, Object> root = new HashMap<String, Object>();
        root.put("title", title);
        root.put("users", users);
        
        // 设置文件的生成目录
        File dir = new File("/Users/18072/Workspaces/MyEclipse for Spring 2014/sshTest/WebRoot");
        if(!dir.exists()){
            dir.mkdirs();
        }
        
        OutputStream fos = new  FileOutputStream( new File(dir, filename)); 
        Writer out = new OutputStreamWriter(fos);
        // 通过Template可以将模板文件输出到相应的流
        temp.process(root, out);
        
        fos.flush();
        fos.close();
        System.out.println("createJsp code success!");
    }
}
```

***

### 生成Service的模板文件

```java
// service.ftl
package com.hziee.app.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.hziee.app.mapper.${POJO}Mapper;
import com.hziee.app.model.${POJO};
import com.hziee.core.mybatis.SqlRunner;
import com.hziee.core.sql.Row;

import java.util.List;

@Service
public class ${POJO}Service {

    @Autowired
    private ${POJO}Mapper ${table_name}Mapper;
    
    @Autowired
    private SqlRunner sqlRunner;

    public List<File> getAll() {
        return ${table_name}Mapper.selectAll();
    }

    public List<Row> findBySQL(String fields, String where) {
        String sql = "select "+fields+" from ${table_name} "+where;
        return sqlRunner.select(sql);
    }
    
    public ${POJO} getById(Integer id) {
        return ${table_name}Mapper.selectByPrimaryKey(id);
    }

    public void deleteById(Integer id) {
            ${table_name}Mapper.deleteByPrimaryKey(id);
    }

    public int save(${POJO} entity) {
        if (entity.getId() != null) {
            return ${table_name}Mapper.updateByPrimaryKey(entity);
        } else {
            return ${table_name}Mapper.insert(entity);
        }
    }
}
```

### 生成jsp页面的模板文件

> 若jsp页面中使用了el表达式，则可以用 `${r"${}"}` 将el表达式包裹在里面。

```html
<!-- hello.ftl -->
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<html>
    <head>
        <title>Hello !</title>
    </head>
    <body>
        <h1>hello ${title}</h1>
        <!-- 循环生成用户 -->
        <#list users as user>
            <p>${user.id} 姓名：${user.username}, ${r"${username}"}</p>
        </#list>
    </body>
</html>
```

**生成的效果图** 

![img](../../../../img/freemarker/1.png)

### 数字格式化插值

数字格式化插值可采用#{expr;format}形式来格式化数字,其中format可以是:

 - mX:小数部分最小X位
 - MX:小数部分最大X位

如下面的例子:

```html
<!-- 初始化x、y的值 -->
<#assign x=2.582/>
<#assign y=4/>

#{x; M2} <#-- 输出2.58 -->
#{y; M2} <#-- 输出4 -->

#{x; m2} <#-- 输出2.6 -->
#{y; m2} <#-- 输出4.0 -->

#{x; m1M2} <#-- 输出2.58 -->
#{x; m1M2} <#-- 输出4.0 -->
```






