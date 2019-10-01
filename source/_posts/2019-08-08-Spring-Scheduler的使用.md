---
title:  Spring Scheduler的使用
date: 2019-08-08 11:30:00
category: learn
toc: true
tags:
    - java
    - spring
---

#### 简介

> Spring Scheduler里有两个概念：任务（Task）和运行任务的框架（TaskExecutor/TaskScheduler）。TaskExecutor顾名思义，是任务的执行器，允许我们异步执行多个任务。TaskScheduler是任务调度器，来运行未来的定时任务。触发器Trigger可以决定定时任务是否该运行了，最常用的触发器是CronTrigger，具体用法会在下面章节中详细介绍。Spring内置了多种类型的TaskExecutor和TaskScheduler，方便用户根据不同业务场景选择。

参考[这篇短文](https://www.baeldung.com/spring-scheduled-tasks)可以快速入门

---

#### Cron 

Cron表达式由6~7项组成，中间用空格分开。从左到右依次是：**秒、分、时、日、月、周几、年（可省略）**。值可以是数字，也可以是以下符号：
- `*`：所有值都匹配

- `?`：无所谓，不关心，通常放在“周几”里

- `,`：或者

- `/`：增量值

- `-`：区间

  

下面举几个例子，看了就知道了：
`0 * * * * *`：每分钟（当秒为0的时候）
`0 0 * * * *`：每小时（当秒和分都为0的时候）
`*/10 * * * * *`：每10秒
`0 5/15 * * * *`：每小时的5分、20分、35分、50分
`0 0 9,13 * * *`：每天的9点和13点
`0 0 8-10 * * *`：每天的8点、9点、10点
`0 0/30 8-10 * * *`：每天的8点、8点半、9点、9点半、10点
`0 0 9-17 * * MON-FRI`：每周一到周五的9点、10点…直到17点（含）
`0 0 0 25 12 ?`：每年12约25日圣诞节的0点0分0秒（午夜）
`0 30 10 * * ? 2016`：2016年每天的10点半

其中的`?`在用法上其实和`*`是相同的。但是`*`语义上表示全匹配，而`?`并不代表全匹配，而是不关心。比如对于`0 0 0 5 8 ? 2016`来说，2016年8月5日是周五，`?`表示我不关心它是周几。而`0 0 0 5 8 * 2016`中的`*`表示周一也行，周二也行……语义上和2016年8月5日冲突了，你说谁优先生效呢。

不记得也没关系，记住[Cron Maker](http://www.cronmaker.com/)也可以，它可以在线生成cron表达式。

---

如果想在不同的开发环境中使用不同的定时器，可以在`application.properties` 中进行配置

```properties
// application properties
// java的properties文件支持value中带空格
cron_expression = 0 0/2 * * * ?
```



```java
@Scheduled(cron = "${cron_expression}")
public void catchAllPicture(){}
```

---

:point_right::point_right::point_right:摘录自[[Spring Scheduler的使用与坑](https://qinghua.github.io/spring-scheduler/)](https://qinghua.github.io/spring-scheduler/)

