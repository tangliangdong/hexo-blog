---
layout:     post
title:      "Spring mybatis一对多表查询"
subtitle:   "mybatis高级映射"
date:       2016-12-19 22:10:00
author:     "Tang"
header-img: "img/spring-static.jpg"
catalog:    true
tags:
    - spring
    - mybatis
    - 经验
---

比如用户和活动的关系，一个用户可以参加多个活动，一个活动允许多个用户参加，虽然这是多对多的关系，但是实际应用中，基本用到的是一对多，如果用户需要查看自己参与的活动，那用户就只有一个，但活动有多个。

### 用户活动表

**用户表**

![img](../../../../img/spring-mybatis/spring-mybatis1.jpg)

**活动表**

![img](../../../../img/spring-mybatis/spring-mybatis2.jpg)

**用户参与活动表**

![img](../../../../img/spring-mybatis/spring-mybatis3.jpg)

### 用户活动数据类型

**用户模型类**

```java
package edu.hziee.models;

public class User {
    private Integer id;
    private String username;
    private String password;
    private String showname;
    private Integer power;

    //getter和setter方法
    //......
}
```

**活动模型类**

```java
package edu.hziee.models;

public class Activitys {
    private Integer activityId;
    private String name;
    private Integer userId;
    private long startTime;
    private long endTime;
    private String place;
    private Integer peopleCount;

    //参与活动的用户集合
    private List<User> uList;

    //getter和setter方法
    //......
}
```

**参与活动模型类**

```java
package edu.hziee.models;

public class Attendactivity {
    private Integer activityId;
    private Integer userId;

    //getter和setter方法
    //......
}
```

### mapper映射文件

一对多映射的关键是使用了 collection映射。

在**ActivityMapper.xml**加上resultMap，

```xml
<resultMap id="AttendResultMap" type="edu.hziee.models.Activitys">
    <id column="activity_id" jdbcType="INTEGER" property="activityId" />
    <result column="name" jdbcType="VARCHAR" property="name" />
    <result column="user_id" jdbcType="INTEGER" property="userId" />
    <result column="start_time" jdbcType="INTEGER" property="startTime" />
    <result column="end_time" jdbcType="INTEGER" property="endTime" />
    <result column="place" jdbcType="VARCHAR" property="place" />
    <result column="people_count" jdbcType="INTEGER" property="peopleCount" />
    <result column="pass" jdbcType="INTEGER" property="pass" />

<!-- 关联活动明细信息 
    一个订单关联查询出了多条订单明细,要使用collection映射
    collection:对关联查询到的多条记录映射到集合中
    property:将关联查询到的多条记录映射到Activitys类的那个属性
    ofType:指定映射的集合属性中uList的类型
--> 
    <collection property="uList" ofType="edu.hziee.models.User">
        <id column="id" jdbcType="INTEGER" property="id" />
        <result column="username" jdbcType="VARCHAR" property="username" />
        <result column="password" jdbcType="VARCHAR" property="password" />
        <result column="showname" jdbcType="VARCHAR" property="showname" />
        <result column="power" jdbcType="INTEGER" property="power" />
    </collection>
</resultMap>
```

再加入查询语句。

如果连接的几张表里有同名的字段，需要取别名，而且resultMap里的column值也要改成别名，不然无法将User映射到collection里。

```xml
<!-- 因为activitys表的id字段与user表的id字段重名了，因此需要给activity表的字段id取别名activity_id，不然在colleaction映射会出问题 -->
<!-- activitys.id as activity_id -->
<select id="selectAttendUserListByActivityId" resultMap="AttendResultMap" parameterType="java.lang.Integer">
    select user.showname,user.id,
    activitys.id as activity_id,activitys.name,activitys.start_time,activitys.end_time,activitys.place,activitys.people_count,activitys.pass 
    from activitys left join attendactivity on activitys.id = attendactivity.activity_id
    left join user on attendactivity.user_id = user.id 
    where activitys.id = #{activityId}
</select>
```

在 **ActivityMapper.java** 加上

```java
List<Activitys> selectAttendUserListByActivityId(
                    @Param("activityId") int activityId);
```

### 业务类

在 **ActivityService.java** 加上

```java
public List<Activitys> selectAttendUserListByActivityId(int activityId){
    return activitysMapper.selectAttendUserListByActivityId(activityId);
}
```

### 控制器类

在通过spring的控制器将数据以json形式返回到前台显示。

需要相应的jar包

- json-lib-2.4-jdk15.jar

```java
//UserController.java

@RequestMapping("/public")
public String publiced(Model model,HttpServletRequest req,HttpServletResponse res) 
throws IOException{
    //...
    if(req.getMethod().equals("POST")){
        int activityId = Integer.parseInt(
                req.getParameter("activityId"));
        PrintWriter out = res.getWriter();
        List<Activitys> list = attendService.selectAttendUserListByActivityId(activityId);
        //将对象转换成json对象
        JSONArray array = JSONArray.fromObject(list);
        String jsonstr = array.toString();
        //返回给ajax
        out.write(jsonstr);
        return null;
    }

    //...
}
```

### 前台

通过`JSON.parse(data)`将字符串解析为json对象

`eval('(' + str + ')')`也可以解析字符串，但如果str是一串恶意js代码，同样会执行，但`parse()`则会报错，所以一般不用`eval()`

```javascript
$.post('<c:url value="/userinfo/public" />?type=2',{activityId: $(this).attr('data')}, function(data, textStatus, xhr) {
    var obj = JSON.parse(data);
});
```






