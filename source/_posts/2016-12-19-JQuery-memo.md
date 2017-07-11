---
layout:     post
title:      "jQuery给服务端生成的DOM绑定事件"
subtitle:   "each()遍历JQuery对象"
date:       2016-12-19 18:10:00
author:     "Tang"
header-img: "img/jquery-memo.jpg"
catalog:    true
tags:
    - jQuery
    - 经验
---

通过服务端传过来的Model生成的DOM元素，因为可能会有多个按钮，如果通过设置id值来获取按钮，则只会有第一个带这个id值的按钮绑上指定的事件，其他按钮则没有。

```html
<table class="table table-hover">
    <tr>
        <th>活动名称</th>
        <th>发起人</th>
        <th>发起时间</th>
        <th>结束时间</th>
        <th>活动地点</th>
        <th>可参与人数</th>
        <th>操作</th>
    </tr>
    <c:forEach items="${activitys}" var="activity">
        <tr>
            <td>${activity.name}</td>
            <td><span class="text-info">${activity.getuList().get(0).getShowname()}</span></td>
            <td>${activity.getDateStartTime()}</td>
            <td>${activity.getDateEndTime()}</td>
            <td>${activity.place}</td>
            <td>${activity.peopleCount}</td>
            <td>
                <!-- 想要给动态生成的按钮绑上事件，不能通过设定id值，而要通过class属性 -->
                <button data="${activity.id}" class="btn btn-danger attend-btn">退出活动</button>
            </td>
        </tr>
    </c:forEach>
</table>
```

### each()

`.each(function(index, el) {});`会遍历JQuery对象，通过`$(this)`就可以获取到每次循环的对象（单个按钮），单独给他绑上事件，这样就可以保证所有的按钮都有指定的事件。

```javascript
(function(){
    $('.attend-btn').each(function(index, el) {
        //取一个变量保存$(this)当前操作的DOM，因为后面可能会出现$(this)对象变更的情况
        var $this = $(this);
        $this.click(function(event) {
            $.ajax({
                url: '<c:url value="/userinfo/cancelAttend" />?activityId='+$(this).attr('data'),
                type: 'post',
                dataType: 'json',
                success:function(data){
                    if(data.id === '1'){
                        $this.parent().parent().remove();
                        alert("成功退出该活动");
                    }else{
                        alert("无法退出活动");
                    }
                }
            });
        }); 
    });
})();
```

