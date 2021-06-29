---
title: springboot添加动态定时任务
toc: true
date: 2020-03-05 16:36:00
categories: 后端
tags:
    - springboot
---

- 首先这里我们需要重新认识一个类ThreadPoolTaskScheduler：线程池任务调度类，能够开启线程池进行任务调度。

- ThreadPoolTaskScheduler.schedule()方法会创建一个定时计划ScheduledFuture，在这个方法需要添加两个参数，Runnable（线程接口类） 和CronTrigger（定时任务触发器）

- 在ScheduledFuture中有一个cancel可以停止定时任务。

<!-- more -->

因为项目中需要给每个用户添加一个专属的定时器，而且每个用户的定时器任务执行的时间间隔也不尽相同，因此不能采用静态的定时器，而是需要动态的从数据库中读取每个用户自定义的时间间隔设置。

### 动态添加定时器

```java
/**
 * 利用线程池实现任务调度
 * Task任务调度器可以实现任务的调度和删除
 * 原理:
 * 实现一个类：ThreadPoolTaskScheduler线程池任务调度器，能够开启线程池进行任务调度
 * ThreadPoolTaskScheduler.schedule（）方法会创建一个定时计划ScheduleFuture,
 * 在这个方法中添加两个参数一个是Runable:线程接口类，和CronTrigger(定时任务触发器)
 * 在ScheduleFuture中有一个cancel可以停止定时任务
 * @author Admin
 * 
 * Scheduled Task是一种轻量级的任务定时调度器，相比于Quartz,减少了很多的配置信息，但是Scheduled Task 不适用于服务器集群，引文在服务器集群下会出现任务被多次调度执行的情况，因为集群的节点之间是不会共享任务信息的
 * 每个节点的定时任务都会定时执行
 */
@Component
public class ElevatorDynamicTask {

    private static HashMap<String, ScheduledFuture<?>> elevatorTaskMap = new HashMap<>();
    
    @Autowired
    private ElevatorBlackBoxSingleAlarmGroupService elevatorBlackBoxSingleAlarmGroupService;

    @Resource
    public void setThreadPoolTaskScheduler(ThreadPoolTaskScheduler threadPoolTaskScheduler) {
        this.threadPoolTaskScheduler = threadPoolTaskScheduler;
    }

    /**
     * 创建对应定时器
     * @param hxzId
     */
    public void startCron(String hxzId, String minute) {
        if( elevatorTaskMap.containsKey(hxzId) ){
            System.out.println("该" + hxzId + "已存在");
            return;
        }
        LogTemplate.LogForInfo(hxzId + "已添加定时器，时间间隔为"+minute+"分钟");
        elevatorTaskMap.put(hxzId,  threadPoolTaskScheduler.schedule(()->{
            // 定时检测给用户推送预警信息
            System.out.println("定时器")
        }, new CronTrigger("30 0/"+minute+" * * * *")));
    }

    /**
     * 结束对应定时器
     * @param hxzId
     */
    public void stopCron(String hxzId) {
        if (elevatorTaskMap.get(hxzId) != null) {
            elevatorTaskMap.get(hxzId).cancel(true);
            elevatorTaskMap.remove(hxzId);
        }
        System.out.println("DynamicTask.stopCron()");
    }

    public Map getMap(){
        return elevatorTaskMap;
    }
}
```



### 静态定时器

- @Scheduled(fixedDelay = 5000)    两次任务的间隔是**前次任务的结束与下次任务的开始**。
- @Scheduled(fixedRate = 5000)   两次任务执行时间间隔是 **任务的开始点**
- @Scheduled(cron = "0/5 * * * * *")     当时间达到设置的时间会触发事件。

```java
@Component
public class ScheduledTask {

    /**
     * 每次任务执行完5秒后再执行一次
     * @Title: testFixDelay
     * @Description:表示上一次任务执行完成后多久再次执行，参数类型为long，单位ms;
     */
    @Scheduled(fixedDelay = 5000)
    public void catchPicture(){
        System.out.println("定时抓图");
    }
    
    /**
     * 每5秒执行一次任务
     * @Title: testFixDelay
     * @Description:表示上一次任务执行完成后多久再次执行，参数类型为long，单位ms;
     */
    @Scheduled(fixedRate = 5000)
    public void alarm(){
        System.out.println("定时抓图");
    }

    /**
     * 每5秒执行一次
     */
    @Scheduled(cron = "0/5 * * * * *")
    public void check(){
        System.out.println("每5秒检测一次");
    }
    
}
```



### 参考链接

- [spring-boot 定时任务之Scheduled Task](https://blog.csdn.net/qq_34125349/article/details/77430956)

