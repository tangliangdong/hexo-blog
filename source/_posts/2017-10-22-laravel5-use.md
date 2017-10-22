---
title: laravel5.5 使用注意点
date: 2017-10-22 13:14:10
category: Learn
tags: 
    - laravel
---

<!-- more -->

### 1. blade模板使用 @section

laravel自带有blade模板引擎，需在后缀前面加上 `.blade`

先创建一个布局视图 `app.blade.php`,

```php
<!-- 文件保存于 resources/views/layouts/app.blade.php -->
<html>
    <head>
        <title>应用程序名称 - @yield('title')</title>
    </head>
    <body>
        @section('header')
            <div class="footer">这是头部</div>
        @show
        
        <div class="container">
            <!--  @yield功能 类似占位符 -->
            @yield('content')
        </div>
        
        @section('footer')
            <div class="footer">这是尾部</div>
        @show
    </body>
</html>
```
        
再新建一个模板 继承该布局
        
```php
<!-- 文件保存于 resources/views/layouts/index.blade.php -->
<html>
    <head>
        @section('title')
            你好 laravel5
        @endsection
    </head>
    <body>
        <!-- 为子视图指定应该 「继承」 的布局 -->
        @extends('layouts.app')
        
        <!-- 继承 Blade 布局的视图可使用 @section 命令将内容注入于布局的 @section 中 -->
        @section('header')
        
        <!-- 主」布局中使用 @yield 的地方会显示这些子视图中的 @section 间的内容 -->
        @section('content')
            Hello world
        @endsection
        
        @section('footer')
    </body>
</html>
```

> 这里有个注意点，比如header 只是想用模板的，不需要在进行定制，就直接写一个 `@section('header')`这个就行了，不需要额外加别 *@* 的进行闭合。如果需要在里面进行个性化定制，则需要加上 `@endsection` 进行闭合

```php
@section('header')
    <p>这是头部</p>
@endsection
```

> 这样写会把之前的全部替换掉，如果是想在原来的基础上添加内容，则加上 `@parent`

```php
@section('header')
    @parent
    <p>这是头部</p>
@endsection
```


