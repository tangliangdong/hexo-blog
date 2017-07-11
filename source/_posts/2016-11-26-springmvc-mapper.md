---
layout:     post
title:      "springMVC操作数据库"
subtitle:   "DAO访问数据对象"
date:       2016-11-26 22:00:00
author:     "Tang"
header-img: "img/springmvc-mapper.jpg"
catalog:    true
tags:
    - spring
---

### view、controller、service、dao的理解

- view：视图，用户看到的东西，后台怎么处理不关心
- Controller：控制层，也可以成为action（控制视图的跳转），可以调用service层的方法。
- service：业务逻辑层它向上层contrller提供接口，并且使用dao层提供的接口
- dao:数据访问对象，dao层里面要完成的是数据访问逻辑以及对数据的访问，dao为上层service提供接口。

> 你写程序不应该仅仅为实现功能考虑，更多的还是应该为将来的维护考虑，因为大部分的时间还是在维护上的。

### `BlogMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 映射到接口 -->
<mapper namespace="edu.hziee.mappers.BlogMapper">

    <!-- 设置返回结果集的映射 -->
    <resultMap id="blogResultMap" type="edu.hziee.models.Blog">
        <id property="id" column="id" />
        <result property="content" column="content" />
        <result property="author" column="author" />
        <result property="title" column="title" />
    </resultMap>
    
    <select id="selectBlog" resultMap="blogResultMap">
        select id, content,author,title from Blog where id = #{id}
    </select>

    <select id="selectBlogs" resultMap="blogResultMap">
        select id, content,author,title from Blog
    </select>

    <!-- 自动生成 自增的id，设置 useGeneratedKeys="true" keyProperty="id" -->
    <insert id="insertBlog" useGeneratedKeys="true" keyProperty="id">
        insert into Blog(content,author,title) values (#{content}, #{author},#{title})
    </insert>

    <update id="updateBlog" parameterType="edu.hziee.models.Blog">
        update Blog set
        content = #{content},
        title = #{title},
        author = #{author}
        where id=#{id}
    </update>
    
    <delete id="deleteBlog">
        delete from Blog where id=#{id}
    </delete>
</mapper>
```

### `blogMapper.java`接口

`blogMapper.java`中的抽象方法要和xml文件中的sql语句对应。

```java
package edu.hziee.mappers;
//只能是接口，不能是具体的类
public interface BlogMapper {
    Blog selectBlog(int id);
    List<Blog> selectBlogs();
    int insertBlog(Blog blog);
    int updateBlog(Blog blog);
    int deleteBlog(int id);
}
```

### `BlogDao.java` DAO层

> (Data Access Object)是用于访问数据的对象
> DAO层（数据库操作层）：负责对数据向数据库增删改查的操作。

```java
package edu.hziee.dao;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import edu.hziee.mappers.BlogMapper;
import edu.hziee.models.Blog;

@Repository
public class BlogDao {

    //注解驱动，自动装配
    @Autowired
    private BlogMapper blogMapper;
    
    //增加
    public void insertBlog(Blog blog){
        blogMapper.insertBlog(blog);
    }
    //查询
    public Blog selectBlog(int id){
        return blogMapper.selectBlog(id);
    }
    //查询
    public List<Blog> selectBlogs(){
        return blogMapper.selectBlogs();
    }
    //删除
    public void deleteBlog(int id){
        blogMapper.deleteBlog(id);
    }
    //更新
    public void updateBlog(Blog blog){
        blogMapper.updateBlog(blog);
    }
}
```

### `BlogController.java` controller控制层

> 把用户请求的数据经过业务处理层处理之后封装成一个Model ，然后再把该Model 返回给对应的View 进行展示

```java
package edu.hziee.controllers;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;

import edu.hziee.dao.BlogDao;
import edu.hziee.models.Blog;

@Controller
@Transactional
@RequestMapping("/blog")
public class BlogController {
    
    //注解驱动
    @Autowired
    BlogDao blogDao;
    
    @RequestMapping(method=RequestMethod.GET)
    public ModelAndView getBlogs(){
        ModelAndView mav = new ModelAndView("blog/blogs");
        //从数据库获取blog数据
        List<Blog> blogs = blogDao.selectBlogs();
        //显示在view视图上
        mav.addObject("blogs",blogs);
        return mav;
    }
}
```