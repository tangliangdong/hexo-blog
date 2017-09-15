---
title: springboot aop 面向切面编程
date: 2017-09-14 21:25:14
category: Learn
toc: true
tags:
    - springboot
    - aop
---

 - 在pom.xml文件中引入AOP依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

<!--more-->

---

 - 使用@Aspect注解将一个java类定义为切面类
 - 使用@Pointcut定义一个切入点，可以是一个规则表达式，比如下例中某个package下的所有函数，也可以是一个注解等。
 - 根据需要在切入点不同位置的切入内容
    - 使用**@Before**在切入点开始处切入内容
    - 使用**@After**在切入点结尾处切入内容
    - 使用**@AfterReturning**在切入点return内容之后切入内容（可以用来对处理返回值做一些加工处理）
    - 使用**@Around**在切入点前后切入内容，并自己控制何时执行切入点自身的内容
    - 使用**@AfterThrowing**用来处理当切入内容部分抛出异常之后的处理逻辑

```
@Aspect
@Configuration
public class TaskAop {
    @After("execution(* flybear.hziee.app.service.TaskService.save*(..)) &&"+"args(task,..)" )
    public void addlog(Task task){
		HttpServletRequest request = ((ServletRequestAttributes) 
	        RequestContextHolder.getRequestAttributes()).getRequest();    
	    HttpSession session = request.getSession();
	    Row user=(Row) session.getAttribute("userinfo");
	    String content= user.getString("username")+"创建了"+task.getName();
	}
	
	// 在切入点return内容之后切入内容（可以用来对处理返回值做一些加工处理）
	@AfterReturning(pointcut="execution(* flybear.hziee.app.service.UserService.Userinfo*(..)) ", returning="retVal")
    public void loginlog(Object retVal){
		
	}
}
```

---

 - Spring AOP无法拦截内部方法调用解决办法

```
@Service
public class TaskService {
    // 获取代理 截取内部方法
    private TaskService getService(){  
        return AopContext.currentProxy() != null ? 
            (TaskService)AopContext.currentProxy() : this;  
    }
    
    public int submit(int taskId, int status, String info) {
        Task task = findById(taskId);
        task.setStatus(status);
        // 用内部方法时，先调用该方法
        int i = getService().update(task);
        return i;
    }
}
```

 - springboot在application.java 入口加入注解

`@EnableAspectJAutoProxy(proxyTargetClass = true,exposeProxy = true)`

```
// Application.java
@EnableAspectJAutoProxy(proxyTargetClass = true,exposeProxy = true)
public class Application{

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
	
}

```


