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
        List<CameraArea> cameraAreaList = cameraAreaService.getCameraArea();

        for(CameraArea item: cameraAreaList){
            
            new Thread(()->{
                // 每个监控区域下的所有监控进行抓图
                cameraService.catchPictureByCameraArea(item);
            }).start();
        }
    }catch (Exception e){
        e.printStackTrace();
    }
}
```

但是在询问技术总监时，他还提到一个线程数的问题，并发的线程数不宜太多，太多了服务器容易卡，从而影响到其他的业务。

因此需要限制线程的创建数量



## ExecutorService

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
        // 获取监控区域列表
        List<CameraArea> cameraAreaList = cameraAreaService.getCameraArea();
        
        for(CameraArea item: cameraAreaList){
            // 多线程执行
            executorService.submit(()->{
                // 每个监控区域下的所有监控进行抓图
                cameraService.catchPictureByCameraArea(item);
            });
        }
    }catch (Exception e){
        e.printStackTrace();
    }
}
```



## 多线程实现事务一起回滚

多线程虽然能提高cpu的利用率，但是每个线程里的事务都不一样，springboot的声明式事务也是在单线程里才有效。因此若是某个线程出问题，或者是在主线程上报错，都无法实现事务的整体回滚。

- 为了实现所有线程的事务整体回滚，我们需要手动进行回滚。

- 如果我们不对所有线程进行拦截，每个线程执行完，就自己提交到数据库了。

因此我们需要对主线程进行拦截，让主线程等待所有的子线程执行完毕之后再执行下去。



### 手动回滚提交

```java
@Autowired
private PlatformTransactionManager transactionManager;

public void insert(String content){
    // 1、
    // 获取事务状态
    DefaultTransactionDefinition def = new DefaultTransactionDefinition();
    def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
    TransactionStatus status = transactionManager.getTransaction(def);
    // 回滚提交
    status.setRollbackOnly();
    
    // 2、
    // 直接回滚提交
    TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
}
```

### CyclicBarrier 同步屏障

> CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。
>
> CyclicBarrier 默认的构造方法是 CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。



```java
public class CyclicBarrierTest {

    // 线程数量
    static Integer THREAD_NUM = 3;

    static CyclicBarrier cyclicBarrier = new CyclicBarrier(THREAD_NUM);

    public static void main(String[] args) {
        // 子线程数 = THREAD_NUM - 1 （主线程数）
        for (int i = 0; i < THREAD_NUM - 1; i++) {
            new Thread(()->{
                try {
                    // 进来的线程先等待，当阻塞的线程数达到 THREAD_NUM 时，就会释放
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }

        try {
            // 主线程先等待，当阻塞的线程数达到 THREAD_NUM 时，就会释放
            cyclicBarrier.await();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



### 实例

需要在每个线程的内部将事务保存下来，若是任何一个线程的执行出现错误，我们就回滚所有的事务。

```java
@Autowired
private PlatformTransactionManager transactionManager;

// 事务集合，用来存储子线程事务
List<TransactionStatus> transactionStatuses = Collections.synchronizedList(new ArrayList<TransactionStatus>());

final int MAX_THREADS = 3; // 定义线程数 

CyclicBarrier c = new CyclicBarrier(MAX_THREADS);

//    @Transactional
public void insert(String name){
    // spring无法处理thread的事务，声明式事务无效
    for(int i = 0 ; i < MAX_THREADS - 1 ; i++){
        new Thread(()->{
            // 获取事务状态
            DefaultTransactionDefinition def = new DefaultTransactionDefinition();
            def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
            TransactionStatus status = transactionManager.getTransaction(def);
            // 将当前子线程的事务添加到集合中
            transactionStatuses.add(status);
            
            try {
                 //执行业务逻辑 插入或更新操作
                User user = new User();
                user.setName(name);
                save(user);
                c.await();
            } catch (Exception e){
                e.printStackTrace();
                // 回滚所有事务
                for (TransactionStatus transactionStatus:transactionStatuses) {
                    transactionStatus.setRollbackOnly();
                }
            }
        }).start();
    }

    try {
        c.await();
    } catch (Exception e){
        e.printStackTrace();
    }
}
```



## 参考链接

- [多线程事务回滚](https://blog.csdn.net/zsj777/article/details/81394323?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)

