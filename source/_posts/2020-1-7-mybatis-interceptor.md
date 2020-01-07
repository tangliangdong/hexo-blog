---
title: springboot mybatis拦截器interceptor配置
date: 2020-01-07 10:43:41
categories: Learn
tags: 
    - springboot
    - interceptor
---

在往数据库中插入数据时，如果 *uuid*、*createTime*、*updateTime* 没有赋值，则程序自动添加一个值，更新数据时则自动更新 *updateTime*

<!-- more -->

## 数据库连接配置


```yaml /resources/application.yml
server:
  port: 8020

spring:
  application:
    name: order-server
  datasource:
    url: jdbc:mysql://localhost:3306/order?useUnicode=true&amp;characterEncoding=utf8
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: 123456
mybatis:
  mapper-locations: classpath:/mapper/*.xml
  type-aliases-package: com.xiaotang.springorder
#  configuration:
#    interceptors:
#      - com.xiaotang.springorder.interceptor.InsertInterceptor

```

## 拦截器

Executor 提供的方法中，update 包含了 新增，修改和删除类型，无法直接区分，需要借助 `MappedStatement` 类的属性 `SqlCommandType` 来进行判断，该类包含了所有的操作类型

```java SqlCommandType.java
public enum SqlCommandType {
    UNKNOWN, INSERT, UPDATE, DELETE, SELECT, FLUSH;
}
```

毕竟新增和修改的场景，有些参数是有区别的，比如**创建时间**和**更新时间**，`update` 时是无需兼顾创建时间字段的。

```java 拦截器中获取sql命令类型和参数
// 1. 获取MappedStatement实例, 并获取当前SQL命令类型
MappedStatement ms = (MappedStatement) invocation.getArgs()[0];
SqlCommandType commandType = ms.getSqlCommandType();

// 2. 获取当前正在被操作的类, 有可能是Java Bean, 也可能是普通的操作对象, 比如普通的参数传递
// 普通参数, 即是 @Param 包装或者原始 Map 对象, 普通参数会被 Mybatis 包装成 Map 对象
// 即是 org.apache.ibatis.binding.MapperMethod$ParamMap
Object parameter = invocation.getArgs()[1];
// 获取拦截器指定的方法类型, 通常需要拦截 update
String methodName = invocation.getMethod().getName();
```



```java /src/main/java/com.xiaotang.springorder.interceptor
@Intercepts({
        @Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class })
})
public class InsertInterceptor implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        Object[] objects = invocation.getArgs();
        MappedStatement mappedStatement = (MappedStatement) invocation.getArgs()[0];
        String commandType =  mappedStatement.getSqlCommandType().name();
        Object object = objects[1];
        BeanWrapper beanWrapper = new BeanWrapperImpl(object);
        PropertyDescriptor[] descriptors = beanWrapper.getPropertyDescriptors();
        if("INSERT".equals(commandType)) {
            for (PropertyDescriptor descriptor : descriptors) {
                String filedName = descriptor.getName();
                if (!"class".equals(filedName) && !"empty".equals(filedName)) {
                    Field field = FieldUtils.getField(object.getClass(), filedName, true);
                    UUID uuid = field.getAnnotation(UUID.class);
                    CreateTime createTime = field.getAnnotation(CreateTime.class);
                    UpdateTime updateTime = field.getAnnotation(UpdateTime.class);
                    if (uuid != null) {
                        if(beanWrapper.getPropertyValue(filedName)==null || "".equals(beanWrapper.getPropertyValue(filedName))) {
                            beanWrapper.setPropertyValue(filedName, java.util.UUID.randomUUID().toString().replaceAll("-", ""));
                        }
                    } else if (createTime != null) {
                        beanWrapper.setPropertyValue(filedName, Calendar.getInstance().getTime());
                    } else if (updateTime != null) {
                        beanWrapper.setPropertyValue(filedName, Calendar.getInstance().getTime());
                    }
                }
            }
        } else if("UPDATE".equals(commandType)){
            for (PropertyDescriptor descriptor : descriptors) {
                String filedName = descriptor.getName();
                if(!"class".equals(filedName)&&!"empty".equals(filedName)){
                    Field field=  FieldUtils.getField(object.getClass(), filedName, true);
                    UpdateTime updateTime = field.getAnnotation(UpdateTime.class);
                    if(updateTime!=null){
                        beanWrapper.setPropertyValue(filedName, Calendar.getInstance().getTime());
                    }
                }
            }
        }

        objects[1] = beanWrapper.getWrappedInstance();
        return invocation.proceed();
    }

    @Override
    public Object plugin(Object target) {
        if (target instanceof Executor) {
            return Plugin.wrap(target, this);
        } else {
            return target;
        }
    }

    @Override
    public void setProperties(Properties properties) {
		
    }
}
```

---

这样配置好拦截器并不会被springboot读取，还需要加一个配置类，将拦截器以bean的形式写入

```java /src/main/java/com.xiaotang.springorder.interceptor
@Configuration
public class MybatisConfiguration {

    @Bean
    InsertInterceptor insertInterceptor(){
        return new InsertInterceptor();
    }
}
```



{% note primary %}
文本内容 (支持行内标签)
{% endnote %}