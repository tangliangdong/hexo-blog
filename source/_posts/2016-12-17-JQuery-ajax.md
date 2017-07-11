---
layout:     post
title:      "JQuery ajax提交form表单"
subtitle:   ".serialize()"
date:       2016-12-17 22:00:00
author:     "Tang"
header-img: "img/jquery-ajax.jpg"
catalog:    true
tags:
    - spring
    - jQuery
    - ajax
---

需要form的`id`、`action`、不需要`submit`按钮，input必须要`name`属性。

```html
<form action="email/send?outputid=${outputid}" id="form" method="post">
  <ul>
    <li>收件人：
        <input type="email" name="toEmail">
        <input type="hidden" name="toId">
    </li>
    <li>抄送：<input type="email" name="tlEmail"></li>
    <li>主题：<input type="text" name="subject"></li>
    <li>
        <span>正文内容：</span>
        <textarea cols="41" rows="6" name="content"></textarea>
    </li>
    <li>
        <!-- 用ajax提交form表单不用submit按钮 -->
        <input value="发送" type="button" id="btn"/>
    </li>
  </ul>
</form>
```

只需要在`ajax`的data属性里加上`$(formid).serialize()`，

```javascript
$('#btn').click(function(event) {
    $.ajax({
        type: "POST",
        url: 'email/send?outputid=${outputid}',
        data: $('#form').serialize(),
        async: false,
        error: function(request) {
            
        },
        success: function(data) {

        }
    });
});
```

后台java只需要在方法里加入对象就行了，ajax传过来的属性会自动打包的成这个对象。

```java
//Email email
@RequestMapping("email/send")
    public String send(Email email,Model model,HttpServletRequest req,
        HttpServletResponse res) throws Exception{

        //......
    }
```