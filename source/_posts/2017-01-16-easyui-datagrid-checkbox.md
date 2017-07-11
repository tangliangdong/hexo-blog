---
layout:     post
title:      "easyui datagrid复选框选择记录"
date:       2017-01-16 22:00:00
author:     "Tang"
header-img: "img/easyui-datagrid-checkbox.jpg"
catalog:    true
tags:
    - easyui
    - datagrid
---

### easyui通过复选框选择记录的不同情况

#### 选中记录只能通过点击复选框，而点击记录即可取消复选框

```html
<table fit="true" id="measurement-admin-table" class="jq-datagrid" style="height: 600px;" data-options="{
    url:'${z:u('measurement/list')}?power=2&status=2',
    method:'post',
    striped: true,
    idField:'id',
    singleSelect: false,
    selectOnCheck:true,
    checkOnSelect:false,
    columns: [[
        {checkbox:true},
        {field:'projectName',title:'项目名称'}
    ]],
    onLoadSuccess: function (data) {
        window.IsCheckFlag = false;
        if(!IsCheckFlag){
            window.is = false;
        }
    },
    onClickCell: function (rowIndex, field, value) {
        is = false;
    },
    onSelect: function (rowIndex, rowData) {
        if (!is) {
            IsCheckFlag = false;
            is=true;
            $('.jq-datagrid').datagrid('unselectRow', rowIndex);
            $('.jq-datagrid').datagrid('uncheckRow', rowIndex);
        }
    },
    onCheck: function(rowIndex,rowData){
        IsCheckFlag = false;
    },
    onUnselect: function (rowIndex, rowData) {
        if (!is) {
            IsCheckFlag = false;
            is=true;
            $('.jq-datagrid').datagrid('selectRow', rowIndex);
            $('.jq-datagrid').datagrid('uncheckRow', rowIndex);
        }
    }
    
}">
</table>
```

1.`onLoadSuccess`是datagrid加载数据成功时执行的。
1.`onClickCell`当用户单击一个单元格时触发。
1.`onSelect`当用户选中一行时触发。
1.`onCheck`当用户勾选一行的复选框时触发。
1.`onUnselect`当用户取消选中一行时触发。

#### 选中取消记录均只能通过点击复选框

```html
<table fit="true" id="measurement-admin-table" class="jq-datagrid" style="height: 600px;" data-options="{
    url:'${z:u('measurement/list')}?power=2&status=2',
    method:'post',
    striped: true,
    idField:'id',
    singleSelect: false,
    selectOnCheck:true,
    checkOnSelect:false,
    columns: [[
        {checkbox:true},
        {field:'projectName',title:'项目名称'}
    ]],
    onLoadSuccess: function (data) {
        window.IsCheckFlag = false;
    },
    onClickCell: function (rowIndex, field, value) {
        IsCheckFlag = false;
    },
    onSelect: function (rowIndex, rowData) {
        if (!IsCheckFlag) {
            IsCheckFlag = true;
            $('.jq-datagrid').datagrid('unselectRow', rowIndex);
        }
    },                    
    onUnselect: function (rowIndex, rowData) {
        if (!IsCheckFlag) {
            IsCheckFlag = true;
            $('.jq-datagrid').datagrid('selectRow', rowIndex);
        }
    }
    
}">
</table>
```

> 初次上手被easyui坑过好多次，虽然界面不好看，但是能够很方便的实现一些前台展示后台数据的方法，比较适合做后台管理系统之类的。