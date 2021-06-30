---
title: springboot transaction事务
toc: true
date: 2020-03-09 13:46:57
categories: 后端
tags:
    - springboot
---



> propagation 事务的传播行为，默认值为 Propagation.REQUIRED。



```java
// 事务
@Transactional(propagation = Propagation.REQUIRED)
public Integer save(){
    //...
    return 1;
}
```

<!-- more -->

1. **REQUIRED**

使用当前事务，如果当前没事务，则自己新建一个事务，子方法是必须运行在一个事务中的；

如果当前存在事务，则加入这个事务，成为一个整体。



2. **SUPPORTS**

如果当前有事务，则使用事务；如果当前没事务，则不使用事务。



3. **MANDATORY**

该传播属性强制必须存在一个事务，如果不存在，则抛出异常。



4. **REQUIRES_NEW**

如果当前有事务，则挂起该事务，并且自己创建一个新的事务给自己使用（别的事务报错，不会影响到该事务）；

如果当前没有事务，则相当于 *REQUIRED*。



5. **NOT_SUPPORTED**

如果当前有事务，则把事务挂起，自己不适用事务去运行数据库操作。



6. **NEVER**

如果当前有事务存在，就直接抛出异常。



7. **NESTED**

- 如果当前有事务，则开启子事务（嵌套事务），嵌套事务是独立提交或者回滚；

- 如果当前没有事务，则同 REQUIRED。

- 但是如果主事务提交，则会携带子事务一起提交。

- 如果主事务回滚，则子事务会一起回滚。相反，子事务异常，则父事务可以回滚或者不回滚。

```java
@Transactional(propagation = Propagation.REQUIRED)
public void testPropagations(){
    saveParent();
    try{
        // save point
        // 这里子事务报错不会影响到父事务
        saveChildren();
    } catch(Exception e){
        e.printStackTrace();
    }
}

public void saveParent(){
    Stu stu = new Stu();
    stu.setName("parent");
    stu.setAge(44);
    stuMapper.insert(stu);
}

@Transactional(propagation = Propagation.NESTED)
public void saveChildren(){
    saveChild1();
    int a= 1 / 0;
    saveChild2();
}
```



