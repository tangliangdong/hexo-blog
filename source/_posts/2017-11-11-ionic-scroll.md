---
title: ionic scroll水平滚动
date: 2017-11-11 11:38:54
category: Learn
toc: true
tags: 
    - ionic
---

官方的写法：

```html
<ion-scroll scrollX="true"></ion-scroll>

<ion-scroll scrollY="true"></ion-scroll>

<ion-scroll scrollX="true" scrollY="true"></ion-scroll>
```

<!-- more -->

如果要使用横向滚动，必须在 `<ion-scroll>`标签上添加如下属性方可有效

```css
ion-scroll {white-space：nowrap;}
```

不需要给 `ion-scroll` 添加 `width` 属性。

```html
<ion-scroll scrollX="true">
    <img src="assets/imgs/1.jpg" alt="">
</ion-scroll>
```

```css
ion-scroll{
    height: 150px;
    white-space: nowrap;

    img{
        height: 300px;
        width: 300px;
    }
}
```




