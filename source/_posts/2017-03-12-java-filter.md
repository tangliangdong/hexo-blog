---
layout:     post
title:      "spring Filter过滤器"
subtitle:   "拦截未登录用户"
date:       2017-3-11 01:00:00
author:     "Tang"
header-img: "img/java-filter.jpg"
catalog:    true
tags:
    - spring
    - Filter
---

# spring Filter过滤器

### web.xml

```xml
<!-- web.xml -->
<filter>
    <filter-name>SecurityServlet</filter-name>    
    <filter-class>filter.SecurityServlet</filter-class>    
</filter>
<!-- 过滤所有访问的配置方式 -->
<filter-mapping>
    <filter-name>SecurityServlet</filter-name>    
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

 - 如果在web.xml中采用的是过滤所有url的形式，则需要给静态资源（.css,.js,.jpg）配置，以获取到这些静态资源。详情见链接  [Spring获取静态资源](/2016/12/13/spring-static/)<br>
 - 或者可以采用另一种过滤方式

```xml
<filter-mapping>
    <filter-name>SecurityServlet</filter-name>
    <url-pattern>*.jsp</url-pattern>
</filter-mapping>
<filter-mapping>
    <filter-name>SecurityServlet</filter-name>
    <url-pattern>*.do</url-pattern>
</filter-mapping>
```

### SecurityServlet.java

> 新建一个 Servlet类，还要实现 Filter的抽象方法

```java
public class SecurityServlet extends HttpServlet implements Filter {
    
    @Override
    public void doFilter(ServletRequest arg0, ServletResponse arg1,
            FilterChain arg2) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest)arg0;
        HttpServletResponse response = (HttpServletResponse)arg1;
        HttpSession session = request.getSession(true);
        // String usercode = request.getRemoteUser();
        String user_role = (String) session.getAttribute("username");
        String url = request.getRequestURI();

        // 判断用户是否已登录
        if( user_role ==null ||"".equals(user_role)){
            // 若未登录用户访问的不是登录和注册页面，则自动重定向到登录页面
            if(url!=null && !url.equals("") && url.indexOf("login")<0 
                                && url.indexOf("registed")<0){
                // 跳转到需要的链接，如登录页面、错误页面等
                response.sendRedirect(request.getContextPath() + "/login");
                return;
            }
        }
        // 不需要跳转时，arg2.doFilter(arg0, arg1);即可继续执行用户的请求
        arg2.doFilter(arg0, arg1);
        return;
    }
}
```


