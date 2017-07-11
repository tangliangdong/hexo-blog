---
layout:     post
title:      "spring返回json对象"
date:       2017-3-10 21:00:00
author:     "Tang"
header-img: "img/java-returnJson.jpg"
catalog:    true
tags:
    - java
    - spring
    - json
---

# spring 后台返回json数据对象

##### 一般用在ajax访问后台数据，不刷新页面更新部分页面数据。

> 在 `@RequestMapping` 中设置 `produces="application/json"`
> 表示将返回的Object的数据转换成JSON对象再返回给客户端。

> 在需要返回 json 对象的方法前加上 `@ResponseBody`。

#### 第一种方式

```java
// consumes="application/json"是限制前台传过来的数据格式必须是JSON类型的。
@RequestMapping(value="verify",method=RequestMethod.POST,produces="application/json",
        consumes="application/json")
@ResponseBody
public Map<String, Object> verify(String username,Model model,
        HttpServletRequest req,HttpServletResponse res){

    //可以通过Map键值对的形式返回
    //也可以传入一个对象
    Map<String,Object> map = new HashMap<String, Object>();
    if(loginService.verify(username))
        map.put("status", 1);
    else
        map.put("status", 0);
    return map;
}
```

#### 第二种方式

```java
@RequestMapping(value="verify")
@ResponseBody
public String verify(String username,Model model,HttpServletRequest req,
        HttpServletResponse res){

    Map<String,Object> map = new HashMap<String, Object>();
    if(loginService.verify(username))
        map.put("status", 1);
    else
        map.put("status", 0);
    // JSONObject.fromObject(Object)可以将一个对象转换成JSON对象。
    return JSONObject.fromObject(map).toString();
}

```

> 如果加了该方法 `@ResponseBody` ，则不能再返回一个页面。

### js

```javascript
// 表单失去焦点后触发
$username.focusout(function(event) {
    var $this = $(this);
    if($this.val()!==''){
        $.ajax({
            url: "<c:url value='/verify'/>",
            type: 'POST',
            dataType: 'json',
            data: {username: $this.val()},
            success:function(data){

                if(data.status === 1){ //用户名已存在
                    canSubmit = false;
                    $this.css('border-color', 'green');
                    $this.next('span').remove();
                }else{ //用户名不存在
                    canSubmit = true;
                    $this.css('border-color', 'red');
                    $this.after('<span style="color:red">此账户已存在</span>')
                }
            }
        });
    }
  
});
```

### json转换需要的第三方的jar

 - json-lib-2.4-jdk15.jar
 - ezmorph-1.0.6.jar
 - commons-logging.jar
 - commons-lang.jar
 - commons-collections.jar
 - commons-beanutils.jar



