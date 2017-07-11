---
layout:     post
title:      "mybatis SQL大于号、小于号"
date:       2016-12-17 13:00:00
author:     "Tang"
header-img: "img/mybatis-sql.jpg"
catalog:    true
tags:
    - spring
    - mybatis
    - 经验
---

> 因为这个是xml格式的，所以不允许出现类似“>”这样的字符，但是都可以使用`<![CDATA[ ]]>`符号进行说明，将此类符号不进行解析

```
<![CDATA[ when min(starttime)<='12:00' and max(endtime)<='12:00' ]]>   
```

#### 举个栗子

```xml
<!-- 小于号 -->
<select id="selectUser" resultMap="BaseResultMap">
    select username,showname,id,power 
    from user 
    where <![CDATA[ power < 2 ]]>
</select>

<!-- 不等于号 -->
 <select id="selectUser" resultMap="BaseResultMap">
    select username,showname,id,power
    from user 
    where <![CDATA[ power <> 2 ]]>
  </select>
```
