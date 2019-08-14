---
title: css-loader css模块化
date: 2017-09-21 11:40:02
category: Learn
toc: true
tags:
    - css-loader
    - webpack
---

### CSS全局作用域的问题：

* 一旦项目很大了，那很可能出现css重名的情况，因此为了避免这个问题，只能把css类名取的很长，有时候还会乱七八糟的取。
* 而且多人协作的话，这个问题就更加显著了。

> 通过 CSS Modules 可以解决CSS全局作用域的问题，

保证单个组件的所有样式：

1. 集中在同一个地方
2. 只应用于该组件

<!-- more -->

在需要的地方用 import 引入

```js
import styles from "./styles.css"
```
---

### Webpack

选择用Webpack来实现 CSS Modules，在webpack.config.js中加上如下配置，使得webpack将CSS文件作为CSS模块来看待：

```
module.exports = {
  entry: './src/js/root.jsx',
  output: {
    filename: 'bundle-[chunkhash].js',
    path: path.resolve(__dirname, 'dist')
  },
  resolve: {
    extensions: ['.js','.jsx','.scss','css']
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [{
          loader: "style-loader"
        }, {
          loader: "css-loader",
          options: {
            modules: false,
            importLoaders: 1,
            localIdentName: '[path][name]__[local]--[hash:base64:5]',
          }
        }]
      },
    ]
  },
};
```

#### css-loader 参数

##### modules 是否开启CSS模块化

开启css模块话后:

scss

```scss
.mobileHeader{
  flex: 1;
  background: #f6f6f6;
  header{
    border-bottom: 1px solid $headerColor;
    padding-left: 10px;
  }
}
```

打包生成的css

```css
._2sNr12LUYLzeia7ixOVvZ2 {
    flex: 1;
    background: #f6f6f6; 
}
._2sNr12LUYLzeia7ixOVvZ2 header {
    border-bottom: 1px solid #2bb7f5;
    padding-left: 10px; 
}
```

打包生成的html

```html
<div class="_2sNr12LUYLzeia7ixOVvZ2"></div>
```

##### localIdentName 配置生成的标识符

`localIdentName: '[path][name]__[local]--[hash:base64:5]'`

scss

```scss
.mobileHeader{
  flex: 1;
  background: #f6f6f6;
  header{
    border-bottom: 1px solid $headerColor;
    padding-left: 10px;
  }
}
```

打包生成的css

```css
.src-sass-mobile__mobileHeader--2sNr1 {
    flex: 1;
    background: #f6f6f6; 
}
.src-sass-mobile__mobileHeader--2sNr1 header {
    border-bottom: 1px solid #2bb7f5;
    padding-left: 10px; 
}
```

打包生成的html

```html
<div class="src-sass-mobile__mobileHeader--2sNr1"></div>
```

---

### CSS Modules后覆盖antd组件样式

之前在使用 react ➕ andtd 的时候，有些地方需要覆盖和添加antd的样式，但是直接写在scss文件里面，打包出来后，因为css模块化的缘故，导致css类名改变无法覆盖原来的样式。

因此就去找如何才能让 class不被编译为哈希字符串。

用 **:global()** 包裹就行了，这样声明的 `class` 是全局的CSS，语法： `:global(.className)`

```scss
:global(.ant-carousel .slick-slider) {
  text-align: center;
  height: 300px;
  width: 600px;
  line-height: 160px;
  background: #364d79;
  overflow: hidden;
}
```

CSS Modules 提供了两种作用域：

```css
// 局部作用域，
local(.title) {
  color: red;
}

// 全局作用域
:global(.title) {
  color: green;
}
```












