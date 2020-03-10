---
title: springboot 多线程事务一起回滚
toc: true
date: 2020-03-10 15:55:49
categories: 后端
tags:
    - 多线程
    - springboot
---

今天再看极客时间的时候，看到关于项目中 *ThreadLocal* 的坑。学习的途中，想着之前项目上调用所有的海康摄像头抓图的那个方法执行起来特别慢，每个监控都要发送一次请求到海康ISC平台。如果网络是通的，那还好说，如果监控的网络不通，那接口调用起来就特别慢，都要等好几秒超时了才会返回，因为做的是单线程的，所以就只能一个一个等着，速度就特别慢，所以在这里就比较适合用多线程来实现。

<!-- more -->

实际使用下来，也将时间缩减到了原来的 **1/4** 了，效果显著。

```java
public void catchAllPicture(){
    try{
        // 获取所有的需要同步的监控区域
        List<DockingCameraApiDto> dockingCameraList = dockingCameraService.getDockingCameraAndHikIsc(new DockingCameraApiDto());

        for(DockingCameraApiDto item: dockingCameraList){
            
            new Thread(()->{
                // 每个监控区域下的所有监控进行抓图
                cameraService.catchPictureByDockingCamera(item);
            }).start();
        }
    }catch (Exception e){
        e.printStackTrace();
    }
}
```

但是在询问技术总监时，他还提到一个线程数的问题，并发的线程数不宜太多，太多了服务器容易卡，从而影响到其他的业务。

因此需要限制线程的创建数量



### ExecutorService

使用该类的工厂方法创建实例，

```java
final int MAX_THREADS = 3; //定义线程数最大值

ExecutorService executor = Executors.newFixedThreadPool(MAX_THREADS);
```

使用 `submit()` 方法将 `Callable`或`Runnable`任务交给 `ExecutorService`，并返回 `Future`类型的结果。

```java
Future<String> future = executorService.submit(runnableTask);
```

---

将 ExecutorService 使用到项目中：

```java
final int MAX_THREADS = 3; //定义线程数最大值

// 最多可创建三个线程
ExecutorService executorService = Executors.newFixedThreadPool(MAX_THREADS);

public void catchAllPicture(){
  
    try{
        List<DockingCameraApiDto> dockingCameraList = dockingCameraService.getDockingCameraAndHikIsc(new DockingCameraApiDto());
        for(DockingCameraApiDto item: dockingCameraList){
            // 多线程执行
            executorService.submit(()->{
                // 每个监控区域下的所有监控进行抓图
                cameraService.catchPictureByDockingCamera(item);
            });
        }
    }catch (Exception e){
        e.printStackTrace();
    }
}
```



### 多线程实现事务一起回滚

多线程虽然能提高cpu的利用率，但是每个线程里的事务都不一样，springboot的声明式事务也是在单线程里才有效。因此若是某个线程出问题，或者是在主线程上报错，都无法实现事务的整体回滚。

为了实现所有线程的事务整体回滚，我们需要手动进行回滚







