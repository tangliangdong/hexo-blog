---
title: 'fetch spring前后端解决跨域问题'
date: 2017-08-05 20:49:55
category: Learn
toc: true
tags:
    - fetch
    - spring
---

初次学习的react的表单提交，选择用新的技术 fetch 来实现异步表单提交。

fetch在旧版本的浏览器上兼容性不佳，特别是IE系的浏览器。但是现在有第三方插件可以很好的提升浏览器的兼容性。

因为浏览器的同源政策，这里也同样遇到了跨域的问题，之前是通过jsonp来解决的，这次通过设置服务器的CORS（跨域资源共享）来解决跨域问题。

<!--more-->

### 前端实现
```js
// React表单异步提交
handleSubmit(e) {
    // 
    let myHeaders = new Headers({
        'Access-Control-Allow-Origin': '*', 
        'Content-Type': 'text/plain'
    });
    fetch('http://localhost:9090/app/login?account=admin&passwd=123456', {
        method: 'POST',
        headers: myHeaders,
        mode: 'cors'
    }).then(response => response.json()).then(json => {
        console.log(json);
    });
}
```

### 服务端实现

spring MVC 4 解决方案

#### 全局实现

```java
package flybear.hziee.app.conf;

@Configuration // 该注解不一定需要，看环境是如何配置的
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    
    // 定义一个类继承 WebMvcConfigurerAdapter
    @Override public void addCorsMappings(CorsRegistry registry) { 
    	registry.addMapping("/**")
        .allowedHeaders("*")
        .allowedMethods("*")
        .allowedOrigins("*");
	 }
}
```

环境配置的不同，这里可能还需要将该类加入到bean容器中。

```
<bean class="flybear.hziee.app.conf.WebMvcConfig"></bean>
```

#### 使用注解，单个方法

```
@CrossOrigin(origins = "*") // 在方法名前使用该注解
```

```java
// 给单个方法设置CORS
@CrossOrigin(origins = "*")
@RequestMapping(value = "login")
public String login(HttpServletRequest req, HttpServletResponse res) {

    // ...
    return jsonString; // 返回json字符串
}
```

#### 一般做法

一般的解决方式就是配置一个Filter过滤器，将需要跨域的请求拦截下来，给请求头加上必要信息。

```
@Component 
public class myCORSFilter implements Filter {
    @Override public void init(FilterConfig filterConfig) throws ServletException {
    
    } 
     
    @Override 
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain filterChain) throws IOException, ServletException { 
        HttpServletResponse httpResponse = (HttpServletResponse) response; 
        httpResponse.setHeader("Access-Control-Allow-Origin", "*"); 
        httpResponse.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE"); 
        httpResponse.setHeader("Access-Control-Max-Age", "3600"); 
        httpResponse.setHeader("Access-Control-Allow-Headers", "x-requested-with,Authorization"); 
        httpResponse.setHeader("Access-Control-Allow-Credentials","true"); 
        filterChain.doFilter(request, response); 
    } 
    
    @Override public void destroy() {
     
    } 
}
```

再在web.xml中进行设置

```
<filter> 
    <filter-name>cors</filter-name> 
    <filter-class>·CLASS_PATH·.myeCORSFilter</filter-class> 
</filter> 
<filter-mapping> 
    <filter-name>cors</filter-name> 
    <url-pattern>/api/*</url-pattern> 
</filter-mapping>
```

这样设置之后ajax不需要修改，直接请求就行了，fetch也一样，甚至不需要额外设置header，就能正常请求到服务器的数据。

设置服务器CORS的方式对前后端的代码改动较少，且相对jsonp来说支持POST提交的方式，是一种很好的解决跨域问题的解决方式。



