## **线程**[**参数**](https://mp.weixin.qq.com/s?__biz=MzIxNTQ4MzE1NA==&mid=2247485631&idx=1&sn=b0d7cd3f337246c79cd08431d9a6d8ec&chksm=9796dec2a0e157d4b8a05b5bc1adcd53bc6ef81112cac5c7dc93370fbbc3baaab717aa5db628&scene=21#wechat_redirect)

1. **corePoolSize**：核心线程数大小

   不管它们创建以后是不是空闲的。线程池需要保持 corePoolSize 数量的线程，除非设置了 allowCoreThreadTimeOut。

2. **maximumPoolSize**：最大线程数
   
      线程池中最多允许创建 maximumPoolSize 个线程
      
3. **keepAliveTime**：存活时间

      如果经过 keepAliveTime 时间后，超过核心线程数的线程还没有接受到新的任务，那就回收。

4. **unit**：时间单位

      keepAliveTime 的时间单位

5. **workQueue**：存放待执行任务的队列

      当提交的任务数超过核心线程数大小后，再提交的任务就存放在这里。它仅仅用来存放被 execute 方法提交的 Runnable 任务。

6. **threadFactory**：线程工厂

      用于指定如何创建一个线程。比如这里面可以自定义线程名称，当进行虚拟机栈分析时，看着名字就知道这个线程是哪里来的，不会懵逼。

7. **handler** ：拒绝策略

      当队列里面放满了任务、最大线程数的线程都在工作时，这时继续提交的任务线程池就处理不了，应该执行怎么样的拒绝策略。





## **线程池的使用**

阿里 Java 开发手册 对线程池的使用进行了限制，可作参考：

 【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。

说明：使用线程池的好处是减少在创建和销毁线程上所花的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。

 【强制】线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

> 说明：Executors返回的线程池对象的弊端如下：
>
> 1）FixedThreadPool和SingleThreadPool:允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。
>
> 2）CachedThreadPool和ScheduledThreadPool:允许的创建线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。

 



 

## ThreadPoolTaskExecutor和ThreadPoolExecutor区别

**ThreadPoolExecutor**

这个类是JDK中的线程池类，继承自Executor， Executor 顾名思义是专门用来处理多线程相关的一个接口，所有县城相关的类都实现了这个接口，里面有一个execute()方法，用来执行线程，线程池主要提供一个线程队列，队列中保存着所有等待状态的线程。避免了创建与销毁的额外开销，提高了响应的速度。

**ThreadPoolTaskExecutor**

这个类则是spring包下的，是sring为我们提供的线程池类，配合@EnableAsync和@Async实现方法或者类的异步调用。



分析下继承关系：

```java
public class ThreadPoolTaskExecutor extends ExecutorConfigurationSupport
		implements AsyncListenableTaskExecutor, SchedulingTaskExecutor{}
```

```java
ExecutorConfigurationSupport extends CustomizableThreadFactory
		implements BeanNameAware, InitializingBean, DisposableBean

AsyncListenableTaskExecutor extends AsyncTaskExecutor

SchedulingTaskExecutor extends AsyncTaskExecutor
```

从上继承关系可知：

ThreadPoolExecutor是一个java类不提供spring生命周期和参数装配。

ThreadPoolTaskExecutor实现了InitializingBean, DisposableBean ，xxaware等，具有spring特性

AsyncListenableTaskExecutor提供了监听任务方法(相当于添加一个任务监听，提交任务完成都会回调该方法)

**简单理解**：

1、ThreadPoolTaskExecutor使用ThreadPoolExecutor并增强，扩展了更多特性

2、ThreadPoolTaskExecutor只关注自己增强的部分，任务执行还是ThreadPoolExecutor处理。

3、前者spring自己用着爽，后者离开spring我们用ThreadPoolExecutor爽。

注意：ThreadPoolTaskExecutor 不会自动创建ThreadPoolExecutor需要手动调initialize才会创建。如果@Bean 就不需手动，会自动InitializingBean的afterPropertiesSet来调initialize



### 问答

**线程池创建之后，会立即创建核心线程么**

不会。从上面的源码可以看出，在刚刚创建ThreadPoolExecutor的时候，线程并不会立即启动，而是要等到有任务提交时才会启动，除非调用了prestartCoreThread/prestartAllCoreThreads事先启动核心线程。



**核心线程永远不会销毁么**

在JDK1.6之后，如果allowsCoreThreadTimeOut=true，核心线程也可以被终止。



**keepAliveTime=0会怎么样**

在JDK1.8中，keepAliveTime=0表示非核心线程执行完立刻终止。



> [10问10答：你真的了解线程池吗？](https://mp.weixin.qq.com/s/axWymUaYaARtvsYqvfyTtw)





## 执行过程

 ![image](../_images/BV1HQ4y1P7hE.png)

步骤：

1. 先使用核心线程执行任务
2. 满了之后放入阻塞队列
3. 阻塞队列满了就创建非核心线程执行任务
4. 总的线程数达到最大线程数之后按照拒绝策略执行处理



**美团点评技术团队：**


 ![image](../_images/74ce096a-1dd4-4f29-925f-1836a0c6c8ff.png)





## ThreadPoolExecutor 的[内部结构](https://segmentfault.com/a/1190000040009000)

 ![image](../_images/2200655553-61f0197c1c387bd5_fix732.png)



