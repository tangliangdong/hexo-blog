---
title: hexo next主题添加note
toc: true
date: 2020-02-26 12:38:09
categories: 功能
tags:
    - hexo
---

### hexo next主题不一样的提示块

<!-- more -->

{% note default %}
default 提示块标签
{% endnote %}

{% note primary %}
primary 提示块标签
{% endnote %}

{% note success %}
success 提示块标签
{% endnote %}

{% note info %}
info 提示块标签
{% endnote %}

{% note warning %}
warning 提示块标签
{% endnote %}

{% note danger %}
danger 提示块标签
{% endnote %}



使用方法如下：

```bash
{% note default %}
default 提示块标签
{% endnote %}

{% note primary %}
primary 提示块标签
{% endnote %}

{% note success %}
success 提示块标签
{% endnote %}

{% note info %}
info 提示块标签
{% endnote %}

{% note warning %}
warning 提示块标签
{% endnote %}

{% note danger %}
danger 提示块标签
{% endnote %}
```



### 选项卡功能

---

{% tabs tab,2 %} 
<!-- tab tab1 -->
**选项卡 1** 
<!-- endtab -->
<!-- tab tab2 -->
**选项卡 2**
<!-- endtab -->
<!-- tab A -->
**选项卡 3** 
<!-- endtab -->
{% endtabs %}

---

```
{% tabs tab,2 %} 名字为tab，默认在第1个选项卡，如果是-1则隐藏
<!-- tab tab1 -->
**选项卡 1** 
<!-- endtab -->
<!-- tab tab2 -->
**选项卡 2**
<!-- endtab -->
<!-- tab A -->
**选项卡 3** 名字为A
<!-- endtab -->
{% endtabs %}
```



### 文本居中的引用

{% centerquote %}人的一切痛苦，本质上都是对自己无能的愤怒{% endcenterquote %}

{% cq %} 王小波 {% endcq %}

```markdown  标签调用方法
<!-- HTML方式: 直接在 Markdown 文件中编写 HTML 来调用 -->
<!-- 其中 class="blockquote-center" 是必须的 -->
<blockquote class="blockquote-center">blah blah blah</blockquote>

<!-- 标签 方式，要求版本在0.4.5或以上 -->
{% centerquote %}人的一切痛苦，本质上都是对自己无能的愤怒{% endcenterquote %}

<!-- 标签别名 -->
{% cq %} 王小波 {% endcq %}
```



>   参考自
>
>   -   [在hexo-NexT中插入note提示块](https://jinnsjj.github.io/uncategorized/hexo-next-note/)
>   -   [内置标签](https://theme-next.iissnan.com/tag-plugins.html)

