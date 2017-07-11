---
layout:     post
title:      "Mybatis Generator逆向工具"
subtitle:   "Generator配置Mysql使用方法"
date:       2016-12-09 15:00:00
author:     "Tang"
header-img: "img/mybatis-generator.jpg"
catalog:    true
tags:
    - generator
    - 配置
---

> 由于Mybatis框架实现数据表的映射时，配置的代码比较复杂。Mybatis官方提供了generator插件，根据数据库来逆向生成映射文件，加快开发效率。

### Mybatis Generator 插件下载

[Mybatis的Eclipse插件](https://marketplace.eclipse.org/content/mybatis-generator)

![img](../../../../img/mybatis-generator/generator.jpg)

### 新建 generatorConfig.xml

在项目下的`src`文件夹下右键`new`一个`generatorConfig.xml`

![img](../../../../img/mybatis-generator/generator2.jpg)

### generatorConfig.xml 配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration PUBLIC 
    "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" 
    "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
  <context id="context">
    <!-- 数据库连接,database是连接数据库名称 -->
    <jdbcConnection connectionURL="jdbc:mysql://localhost:3306/database" 
        driverClass="com.mysql.jdbc.Driver" password="root" userId="root" />
    <!-- targetPackage 生成的类要放得包 -->
    <!-- targetProject 目标项目，生成的内容会放在指定目录下 -->
    <javaModelGenerator targetPackage="edu.hziee.models" 
        targetProject="ActivityDemo/src" />
    <sqlMapGenerator targetPackage="edu.hziee.mappers" 
        targetProject="ActivityDemo/src" />
    <javaClientGenerator targetPackage="edu.hziee.mappers" 
        targetProject="ActivityDemo/src" type="XMLMAPPER" />
    <!-- 指定需要逆向的表 -->
    <table schema="activity" tableName="user">
      <generatedKey column="id" sqlStatement="mysql" identity="true" />
    </table>
  </context>
</generatorConfiguration>
```

### 执行generatorConfig.xml

直接在generatorConfig.xml上右键，如图所示的选项，点击运行，即可自动生成映射。

![img](../../../../img/mybatis-generator/generator3.jpg)





