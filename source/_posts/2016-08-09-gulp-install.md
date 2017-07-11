---
layout:     post
title:      "配置gulp"
subtitle:   "基于流的自动化构建工具"
date:       2016-08-09 20:00:00
author:     "Tang"
header-img: "img/gulp.jpg"
catalog:    true
tags:
    - gulp
    - 配置
    - npm
---

# 介绍 [gulp](http://www.gulpjs.com.cn/)

这里通过`gulp`实现的功能比较简单，就是将项目打包发布，因为通过`http`请求获得很多小文件的效率不如请求一个大文件，所以我们可以将所有的`css`和`js`文件进行合并。

`gulp`可以给每次合并的`js`和`css`文件名后面跟上一个`Hashcode`，防止浏览器以为是一样的文件而不进行更新。

## 1. 给项目书写package.json

`cd`到项目根目录下，输入`npm init`。

然后会要求输入一些初始化信息，除了名字需要小写外，其他暂时不需要写，直接`enter`就行，最后会产生一个`package.json`的文件 

## 2. 安装gulp

我们需要在全局范围和项目下配置`gulp`环境，输入`npm install -g gulp`，然后输入`gulp -v`查看安装的gulp版本号以确认是否成功安装。

`cd`到项目根目录，输入`npm install gulp --save-dev`，给项目配置`gulp`。

## 3. 下载gulp的插件

继续输入

```
npm install gulp-rev gulp-rev-replace gulp-useref gulp-filter gulp-uglify gulp-csso --save-dev
```
下载所需的插件。

![package.json](../../../../img/gulp/gulp1.jpg)

安装的插件都会添加到`package.json`的生产环境中。

## 4. 书写插件的行为

在项目根目录下创建`gulpfile.js`,在里面输入

```javascript
var gulp = require('gulp');
var rev = require('gulp-rev');
var revReplace = require('gulp-rev-replace');
var useref = require('gulp-useref');
var filter = require('gulp-filter');
var uglify = require('gulp-uglify');
var csso = require('gulp-csso');

gulp.task('default',function () {
    var jsFilter = filter('**/*.js',{restore:true});
    var cssFilter = filter('**/*.css',{restore:true});
    var indexHtmlFilter = filter(['**/*','!**/index.html'],{restore:true});

    return gulp.src('src/index.html')
        .pipe(useref())
        .pipe(jsFilter)
        .pipe(uglify())
        .pipe(jsFilter.restore)
        .pipe(cssFilter)
        .pipe(csso())
        .pipe(cssFilter.restore)
        .pipe(indexHtmlFilter)
        .pipe(rev())
        .pipe(indexHtmlFilter.restore)
        .pipe(revReplace())
        .pipe(gulp.dest('dist'));
});
```

根据[官方](https://www.npmjs.com/package/gulp-useref)的要求，我们还需要在`index.html`页面引入`css`和`js`的地方添加如下的注释代码

```html
<!-- build:<type>(alternate search path) <path> <parameters> -->
... HTML Markup, list of script / link tags.
<!-- endbuild -->
```

例如

```html
<html>
<head>
    <!-- build:css css/combined.css -->
    <link href="css/one.css" rel="stylesheet">
    <link href="css/two.css" rel="stylesheet">
    <!-- endbuild -->
</head>
<body>
    <!-- build:js scripts/combined.js -->
    <script type="text/javascript" src="scripts/one.js"></script> 
    <script type="text/javascript" src="scripts/two.js"></script> 
    <!-- endbuild -->
</body>
</html>
```

## 5. 执行gulp插件

之后在命令行输入`gulp`，就可以执行这些插件，会在根目录下生成`dist`的文件夹，里面有我们可以打包发布的文件。

