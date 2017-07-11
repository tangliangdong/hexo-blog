---
layout:     post
title:      "浅谈响应式"
subtitle:   "制作响应式网站的一些经验"
date:       2016-08-10 11:00:00
author:     "Tang"
header-img: "img/self-adapter.jpg"
catalog:    true
tags:
    - 响应式
    - 经验
---

## 响应式网站概念

 - 1. flexible grid layout 弹性网格布局

 - 2. flexible image 弹性图片

 - 3. media queries 媒体查询

## 首先

各种不同的浏览器之前都会存在默认的行为，也就是默认的css样式并不是一样的。

所以我们需要统一浏览器行为，原来我们会引入reset.css，但reset.css会让很多标签失去原来的样式。

因此我们现在大多采用[Normaliez.css](https://necolas.github.io/normalize.css/),它在统一各浏览器默认css样式的同时，也保持了各个标签原有的一些样式。

## 处理IE低版本浏览器兼容性的问题。

### 1. CSS3媒体查询

我们使用到的媒体查询属于css3的属性，因此可能存在浏览器不兼容的情况。

```css
@media screen and (min-width:600px) {
  nav {
    float: left;
    width: 25%;
  }
  section {
    margin-left: 25%;
  }
}
@media screen and (max-width:599px) {
  nav li {
    display: inline;
  }
}
```

`respond.js`库可以帮我们解决浏览器不兼容`media`媒体查询的问题。

只需在`html`前引入下方 **IE条件注释** ，如果 *IE版本* 小于等于8，则执行注释中的语句，我们就可以放心使用`media`

```html
<!--[if lte IE 8]>
    <script src="js/respond.js"></script>
<![endif]-->
```

### 2. HTML5标签

同样，因为IE 6、7、8 不兼容`HTML5`,因此我们需要引入`HTML5shiv`,

```html
<!--[if lt IE 9]>
    <script src="bower_components/html5shiv/dist/html5shiv.js"></script>
<![endif]-->
```

它会通过js的方式创建HTML5的标签，让我们可以放心使用HTML5的新增标签。

### 3. 自适应图片

既然网站是响应式的，我们的图片当然也需要自适应。

我们可以引入HTML5新增元素`<picture></picture>`。

 - 创建<picture></picture>标签。
 - 在这些标签内创建一个你想用来执行任何一个特性的<source>元素。
 - 添加一个`media`属性，用来包含你想要的特性，如视口的当前高度(viewport height)，宽度(width)，方向(orientation)等。
 - 添加一个`srcset`属性与相应的图像文件名相匹配，进行加载。如果你想提供不同的像素密度，例如Retina显示屏，你可以添加额外的文件名到你的`srcset`属性中，
 - 添加一个回退的`<img>`元素。

```html
<picture>
    <source srcset="default.jpg">
    <source srcset="smaller.jpg" media="(max-width: 768px)">
    <img srcset="default.jpg" alt="My default image">
</picture>
```

但是并不是所有的浏览器都支持`<picture></picture>`，因此我们需要引入`picturefill.js`库,它会给不支持`picture`的浏览器使用替代方案。

```html
<script src="picturefill.js"></script>
```

因为IE 9不能识别`<picture></picture>`中的`<source>`标签，因此我们需要这样写。

```html
<picture>
  <!--[if IE 9]><video style="display: none;"><![endif]-->
  <source srcset="examples/images/extralarge.jpg" media="(min-width: 1000px)">
  <source srcset="examples/images/large.jpg" media="(min-width: 800px)">
  <!--[if IE 9]></video><![endif]-->
  <img srcset="examples/images/medium.jpg" alt="…">
</picture>
```

### 4. 预防浏览器不兼容

Modernizr：专为HTML5和CSS3开发的功能检测类库。

我们可以在官网下载我们需要检测的属性。

![image](../../../../img/self-adapter/self-adapter.jpg)

比如我们需要检测`SVG`，我们就可以下载并保存到本地的`js`文件中，并在`html`中引入，如果浏览器不支持这个属性，就会自动在`<html></html>`标签上加上`class="no-svg"`,

```
<html class="no-svg">
    ...
</html>
```

我们就可以在css样式表中书写

```
.no-svg header ... {
    ...
}

.no-svg .picture ...{
    ...
}
```

给不兼容svg的情况增加替代方案。

## 最后

现在浏览器对HTML5和CSS3的兼容性参差不齐，我们也没必要全部记忆，

我们在使用这些属性时可以先去查看下这个标签或属性的浏览器兼容情况。我们可以访问

[can I use](http://caniuse.com/)

我们只需要将我们需要查询标签或属性输入就行，

![image](../../../../img/self-adapter/caniuse.jpg)

#### 推荐框架

- [Bootstrap](http://v3.bootcss.com/)
- [Foundation](http://www.foundcss.com/)
- [Semantic UI](http://www.semantic-ui-cn.com/)
- [Pure.css](http://www.purecss.org/)

#### 比较好的工具

简单安装的教程，本人博客也有写，附上地址：

 - [http-server](https://tangliangdong.github.io/2016/08/08/http-server/)   基于nodejs的轻量级服务器
 - [Browsersync](https://tangliangdong.github.io/2016/08/07/Browsersync/)   多浏览器多终端同时进行调试的工具
 - [Gulp](https://tangliangdong.github.io/2016/08/08/gulp-install/)   基于流的自动化构建工具



