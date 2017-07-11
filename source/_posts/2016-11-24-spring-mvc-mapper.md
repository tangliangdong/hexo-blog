---
layout:     post
title:      "spring mvc控制器映射"
date:       2016-11-24 19:30:00
author:     "Tang"
header-img: "img/springmvc.jpg"
catalog:    true
tags:
    - spring
    - 经验
---

### spring映射

#### 简单的页面跳转

```html
<a href="blog">blogs页面</a>
```

```java
@Controller
@RequestMapping("/blog")
public class BlogController {
    @RequestMapping(method=RequestMethod.GET)
    public ModelAndView getBlogs(){
        ModelAndView mav = new ModelAndView("blog/blogs");
        //从数据库取得数据
        List<Blog> blogs = blogDao.selectBlogs();
        //将数据存在model里
        mav.addObject("blogs",blogs);
        return mav;
    }
}
```

- 超链接地址`href`要对应控制器的映射地址`@RequestMapping("/blog")`，这样才会调用映射的方法。
- `ModelAndView mav = new ModelAndView("blog/blogs")`或者`mav.setViewName("blog/blogs")`都能用来设置显示 **blog文件夹** 下的`blogs.jsp`页面。

![image](../../../../img/springmvc/springmvc1.jpg)

因为在`dispatcher-servlet.xml`中配置过视图解析器如下

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/views/"></property>
    <property name="suffix" value=".jsp"></property>
</bean>
```

- `InternalResourceViewResolver`会把返回的视图名称都解析为`InternalResourceView`对象，
- `InternalResourceView`会把`Controller`处理器方法返回的模型属性都存放到对应的request属性中
- `InternalResourceViewResolver`会给它加上定义好的前缀和后缀，组成`/WEB-INF/views/xxx.jsp`的形式。如之前的例子，则是显示`/WEB-INF/views/blog/blogs.jsp`。

#### 重定向`redirect`

> redirect 可以重定向到某个controller映射。

如下图，如果我要点击删除一条记录，然后再跳回这个页面。就需要用到请求重定向。

![image](../../../../img/springmvc/springmvc2.jpg)

```java
@RequestMapping(value = "delete/{id}",method = RequestMethod.GET)
public ModelAndView deleteBlog(@PathVariable int id){
    ModelAndView mav = new ModelAndView();
    //根据id从数据库删除对应数据
    blogDao.deleteBlog(id);
    //从数据库重新取数据
    List<Blog> blogs = blogDao.selectBlogs();
    //设置视图名称
    mav.setViewName("redirect:/blog");
    return mav.addObject("blogs",blogs);
}
```

因为我们需要返回到原来的页面，但是需要重新获取、显示数据，我们就不能只是简单的通过`mav.setViewName("blog/blogs")`显示这个视图，而是要调用映射到这个页面的方法，在通过这个方法获取数据，再显示到`blogs.jsp`页面上，也就是如下的方法。

```java
@Controller
@RequestMapping("/blog")
public class BlogController {
    @RequestMapping(method=RequestMethod.GET)
    public ModelAndView getBlogs(){
        ModelAndView mav = new ModelAndView("blog/blogs");
        //从数据库取得数据
        List<Blog> blogs = blogDao.selectBlogs();
        //将数据存在model里
        mav.addObject("blogs",blogs);
        return mav;
    }
}
```

现在页面的URL是`http://localhost:8080/myspring/blog`，如果不使用`redirect` (即`mav.setViewName("blog/blogs")`) ，点击超链接，则URL变成`http://localhost:8080/myspring/blog/delete/36`,会显示没有映射的警告且不会有页面显示。










