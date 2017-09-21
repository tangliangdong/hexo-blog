---
title: 解决antd Tabs高度占满整个屏幕
date: 2017-09-21 08:21:10
category: Learn
tags:
    - react
    - antd
    - Tabs
---

![Tabs高度占满整个屏幕](antd.png)

官方演示事例：

![官方演示Tabs事例](antd2.png)

![css属性](antd3.png)

发现里面有个 `height: 100%` 的属性，想要 height的百分比生效有几种办法，

* 父元素设置固定高度
* 给所有父元素都设置`height: 100%`，直到html 和 body也设置`height: 100%`;

但在这里都失效了，查了下原来react的入口文件`index.html`需要有 `<!DOCTYPE html>`

![](antd4.png)


