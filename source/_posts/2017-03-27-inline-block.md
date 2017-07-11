---
layout:     post
title:      "display: inline-block的小问题"
date:       2017-3-26 20:00:00
author:     "Tang"
header-img: "img/inline-block.jpg"
catalog:    true
tags:
    - css

---

###  一、同行inline-block相互影响

当两个inline-block元素同行显示时，可能会出现基准为bottom的情况，所以当两边内容不同时，就会出现问题！

![img](../../../../img/inline-block/1.png)

很容易就发现，几个div的对齐方式：

* 内部有字符的：以内部字符的底部为对齐点
* 内部没有字符的：以整个div的底部为对齐点 

```html
<!-- html -->
<style>
.box{
    width: 100px;
    height: 200px;
    background-color: red;
    display: inline-block;
}
</style>
<div>
    <div class="box">
    Lorem ipsum dolor sit amet, consectetur adipisicing elit,
    </div>
    <div class="box">
    Lorem ipsum dolor sit amet, consectetur adipisicing elit. Alias, tempore.
    </div>
    <div class="box"> totam odio magni harum ipsam itaque maiores iure. Repudiandae </div>
    <div class="box"></div>
    <div class="box">Lorem ipsum dolor sit amet,  odit!</div>
</div>
```

#### 1.使用 `vertical-align`

```css
.box{
    width: 100px;
    height: 200px;
    background-color: red;
    display: inline-block;
    
    vertical-align: top;
}
```

#### 2.使用浮动

```css
.box{
    width: 100px;
    height: 200px;
    background-color: red;
    display: inline-block;
    
    float: left;
}
```

> 当然，浮动是魔鬼，使用浮动很容易是布局产生问题。我们应当尽量避免使用浮动布局。使用`display: inline-block` 同样可以很好的实现水平排列的效果。

### 二、去除inline-block元素间的间距

![](../../../../img/inline-block/2.png)

```html
<div>
    <a href="">你好betahouse</a>
    <a href="">去年的现在我来了</a>
    <a href="">今年的现在新的我们来了</a>
</div>
```

元素默认属性是`display: inline-block`的元素，或者通过css属性设置成`display: inline-block`的元素之间有 **空格** 或者 **换行** 都会产生这样的问题。

默认inline-block的元素：

* 图片
* 按钮
* 单复选框
* 单行/多行文本框等HTML控件

#### 1.元素间留白间距出现是因为标签段之间的空格，所以，去掉HTML中的空格，自然就消除了元素间间距。

```html
<div class="space">
    <a href="">
    你好betahouse</a><a href="">
    去年的现在我来了</a><a href="">
    今年的现在新的我们来了</a>
</div>
```

或者借助html注释

```html
<div>
    <a href="">你好betahouse</a><!--
    --><a href="">去年的现在我来了</a><!--
    --><a href="">今年的现在新的我们来了</a>
</div>
```

#### 2.~~使用margin负值~~

之前看到有博文说用设置 margin 负值的方法来解决空格的问题。但在我测试了chrome、safari、firefox之后，发现有些问题。

```html
<style>
.two{
    margin: 50px 0 0px 100px;
}
.two a{
    text-decoration: none;
    border: 1px solid;
    background-color: rgb(116, 241, 241);
    padding: 5px 10px;
    margin-right: -4px;
}
</style>
<div class="two">
    <a href="">你好betahouse</a>
    <a href="">去年的现在</a>
    <a href="">我们来了</a>
    <a href="">今年的现在</a>
    <a href="">我们迎接新的你们</a>
</div>
<div class="two">
    <a href="">你好世界</a>
    <a href="">你好世界</a>
    <a href="">你好世界</a>
    <a href="">你好世界</a>
    <a href="">你好世界</a>
    <a href="">你好世界</a>
</div>

```

![](../../../../img/inline-block/3.png)

就算一样的几个元素水平排列，也会出现间隙不同的情况，测试还发现一个很奇怪的情况：在chrome的缩放级别到 150% 的时候，竟然水平元素的间隙都一起消失了，而且不多不少刚好。

![](../../../../img/inline-block/5.png)

> firefox和chrome一样，缩放级别不同，元素之间的间隙也不同。

![](../../../../img/inline-block/4.png)

唯独就safari显示正常了，缩放多少也还是正常的。

所以不建议使用这种方式

#### 2.省略闭合标签

```html
<div>
    <a href="">你好betahouse
    <a href="">去年的现在我来了    
    <a href="">今年的现在新的我们来了
    <!--若要兼容IE6/7，则在只需在最后加上一个闭合标签</a>-->
</div>
```

#### 3.使用font-size: 0;

给父类加上 `font-size: 0`，再给 inline-block 的元素设置 font-size

```css
.two {
    font-size: 0;
    -webkit-text-size-adjust: none;
    /* chrome以前版本的最小字体限制，但是现在已经取消 */
}
.two a {
    font-size: 16px;
}
```

### line-height

> 建议通过设置 line-height 代替 height 设置元素的高度（height是被line-height撑开的）。




