## 进程和线程的区别

进程是系统进行资源分配和调度的独立单位，每一个进程都有它自己的内存空间和系统资源。它的分派，切换都需要花费比较大的时间和开销。

为了提高执行效率，减少空转时间和调度切换的时间，所以有了线程。线程取代了进程的调度的基本功能。

简单来说，进程作为**资源分配**的基本单位，线程作为**资源调度**的基本单位。

## Java 线程

> 参考：[Java 线程和操作系统的线程有啥区别？](https://www.cnblogs.com/cswiki/p/14676264.html)

### 线程库

在进入 Java 线程主题之前，有必要讲解一下**线程库** Thread library 的概念。

在上面的模型介绍中，我们提到了通过线程库来创建、管理线程，那么什么是线程库呢？

**线程库就是为开发人员提供创建和管理线程的一套 API**。

当然，线程库不仅可以在用户空间中实现，还可以在内核空间中实现。前者涉及仅在用户空间内实现的 API 函数，没有内核支持。后者涉及系统调用，也就是说调用库中的一个 API 函数将会导致对内核的系统调用，并且需要具有线程库支持的内核。

下面简单介绍下三个主要的线程库：

1. POSIX Pthreads：可以作为用户或内核库提供，作为 POSIX 标准的扩展
2. Win32 线程：用于 Window 操作系统的内核级线程库
3. Java 线程：**Java 线程 API 通常采用宿主系统的线程库来实现**，也就是说在 Win 系统上，Java 线程 API 通常采用 Win API 来实现，在 UNIX 类系统上，采用 Pthread 来实现。

### Java 线程

下面我们来详细讲解 Java 线程：

事实上，**在 JDK 1.2 之前**，Java 线程是基于称为 "绿色线程"（Green Threads）的用户级线程实现的，也就是说程序员大佬们为 JVM 开发了自己的一套线程库或者说线程管理机制。

而**在 JDK 1.2 及以后**，JVM 选择了更加稳定且方便使用的操作系统原生的内核级线程，通过系统调用，将线程的调度交给了操作系统内核。而对于不同的操作系统来说，它们本身的设计思路基本上是完全不一样的，因此它们各自对于线程的设计也存在种种差异，所以 JVM 中明确声明了：**虚拟机中的线程状态，不反应任何操作系统中的线程状态**。

也就是说，在 JDK 1.2 及之后的版本中，Java 的线程很大程度上依赖于操作系统采用什么样的线程模型，这点在不同的平台上没有办法达成一致，JVM 规范中也并未限定 Java 线程需要使用哪种线程模型来实现，可能是一对一，也可能是多对多或多对一。

总结来说，回答下文题，**现今 Java 中线程的本质，其实就是操作系统中的线程，其线程库和线程模型很大程度上依赖于操作系统（宿主系统）的具体实现，比如在 Windows 中 Java 就是基于 Wind32 线程库来管理线程，且 Windows 采用的是一对一的线程模型**。

## 如何解决线程安全的问题

解决线程安全问题的思路有以下：

1. 能不能保证操作的原子性，考虑atomic包下的类够不够我们使用。

2. 能不能保证操作的可见性，考虑volatile关键字够不够我们使用

3. 如果涉及到对线程的控制（比如一次能使用多少个线程，当前线程触发的条件是否依赖其他线程的结果），考虑CountDownLatch/Semaphore等等。

4. 如果是集合，考虑java.util.concurrent包下的集合类。

5. 如果synchronized无法满足，考虑lock包下的类

### Semaphore

**概念**

- `Semaphore`是 Java 中的一个并发工具类，位于`java.util.concurrent`包下。它用于控制同时访问特定资源的线程数量，通过维护一个计数来实现。这个计数表示可用的许可证数量，线程在访问共享资源之前必须获取许可证，访问结束后归还许可证。

**构造函数**

- **`Semaphore(int permits)`**：创建一个具有指定初始许可证数量的`Semaphore`。例如，`Semaphore semaphore = new Semaphore(3);`创建了一个初始有 3 个许可证的`Semaphore`，这意味着最多允许 3 个线程同时访问共享资源。

**应用场景**

- **资源限流**：用于限制对某个资源（如数据库连接、文件读取等）的并发访问数量。例如，数据库连接池可能只允许有限数量的线程同时获取数据库连接，通过`Semaphore`可以很方便地实现这种限制。
- **流量控制**：在网络编程中，可以用于控制同时发送请求的数量，防止过多的请求导致网络拥塞或者服务器过载。例如，一个 HTTP 客户端可能使用`Semaphore`来限制同时发送的 HTTP 请求数量。

**示例**

```java
public class LimitedParallelRunner {
    public static Executor executor =new ThreadPoolTaskExecutor();
    private int parallelQty;
    private Semaphore semaphore;
    private AtomicInteger failQty = new AtomicInteger(0);

    public LimitedParallelRunner(int parallelQty) {
        this.parallelQty = parallelQty;
        this.semaphore = new Semaphore(parallelQty);
    }


    /**
     * 执行任务
     */
    public void execute(Runnable command) {
        try {
            semaphore.acquire();
            executor.execute(() -> {
                try {
                    command.run();
                } catch (Throwable e) {
                    log.error("LimitedParallelRunner execute error",e);
                    failQty.incrementAndGet();
                    throw e;
                } finally {
                    semaphore.release();
                }
            });
        } catch (InterruptedException e) {
            log.error("LimitedParallelRunner execute failed", e);
            Thread.currentThread().interrupt();
            throw new RuntimeException(e);
        }
    }

    /**
     * 等待完成
     */
    public boolean waitCompleted() {
        try {
            semaphore.acquire(parallelQty);
        } catch (InterruptedException e) {
            log.error("LimitedParallelRunner waitCompleted failed", e);
            Thread.currentThread().interrupt();
            throw new RuntimeException(e);
        }
        return ! (failQty.get() > 0);
    }
```

#### 和 CountDownLatch 的区别

**计数器操作**

- **CountDownLatch**
  - 计数器只能递减，不能递增。一旦计数器的值达到 0，就不能再被重置。它主要用于一次性的事件等待场景。
- **Semaphore**
  - 许可证计数可以通过`acquire`和`release`方法动态地增减。这种灵活性使得`Semaphore`可以适应资源数量动态变化的场景。

**应用场景侧重**

- **CountDownLatch**
  - 适用于等待多个任务完成后再进行下一步操作的场景，如多线程任务完成后的汇总、多个模块加载完成后启动主程序等。
- **Semaphore**
  - 更侧重于对资源访问的控制，确保在任何时刻只有特定数量的线程能够访问共享资源，如限制数据库连接的并发使用数量、控制对文件系统的并发读写等。

## 死锁

当前线程拥有其它线程需要的资源，当前线程等待其他线程已拥有的资源，都不放弃自己拥有的资源。

**如何避免死锁**

- 固定加锁顺序：比如利用Hash值大小确定加锁先后。
- 缩小加锁范围：等操作共享变量的时候才加锁。
- 用可释放的定时锁：一段时间申请不到锁权限就释放掉。



### 线程的实现方式

1. **继承Thread类 重写run方法**
   
   ```java
   class MyJob extends Thread{
       @Override
       public void run() {
           System.out.println("do something.");
       }
   }
   ```

2. **实现 Runnable 接口 重写run方法**
   
   ```java
   class MyJob implements Runnable{
       @Override
       public void run() {
           System.out.println("do something.");
       }
   }
   ```

3. **实现Callable 重写call方法，配合FutureTask**
   
   ```java
   import java.util.concurrent.Callable;
   import java.util.concurrent.ExecutionException;
   import java.util.concurrent.FutureTask;
   
   public class CallableExample {
       public static void main(String[] args) {
           // 创建 Callable 对象
           Callable<String> callable = new MyCallable();
   
           // 使用 FutureTask 来包装 Callable 对象
           FutureTask<String> futureTask = new FutureTask<>(callable);
   
           // 创建线程并启动
           Thread thread = new Thread(futureTask);
           thread.start();
   
           try {
               // 获取 Callable 任务的返回结果
               String result = futureTask.get();
               System.out.println("返回结果: " + result);
           } catch (InterruptedException | ExecutionException e) {
               e.printStackTrace();
           }
       }
   }
   
   // 实现 Callable 接口
   class MyCallable implements Callable<String> {
       @Override
       public String call() {
           // 模拟耗时操作
           try {
               Thread.sleep(2000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           return "任务执行完成";
       }
   }
   ```

4. **基于线程池，构建线程**
   
   

总结，底层其实只有一种，实现Runnable；比如Thread类实际实现了Runnable接口；FutureTask类实际也是实现了Runnable接口。



## 线程池

线程池能够对线程进行统一分配，调优和监控:

- 降低资源消耗(线程无限制地创建，然后使用完毕后销毁)
- 提高响应速度(无须创建线程)
- 提高线程的可管理性

### 线程池实现

Java是如何实现和管理线程池的?

从JDK 5开始，把工作单元与执行机制分离开来，工作单元包括Runnable和Callable，而执行机制由Executor框架提供。

- WorkerThread
  
  ```java
  public class WorkerThread implements Runnable {
  
      private String command;
  
      public WorkerThread(String s){
          this.command=s;
      }
  
      @Override
      public void run() {
          System.out.println(Thread.currentThread().getName()+" Start. Command = "+command);
          processCommand();
          System.out.println(Thread.currentThread().getName()+" End.");
      }
  
      private void processCommand() {
          try {
              Thread.sleep(5000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  
      @Override
      public String toString(){
          return this.command;
      }
  }
  ```

- SimpleThreadPool
  
  ```java
  import java.util.concurrent.ExecutorService;
  import java.util.concurrent.Executors;
  
  public class SimpleThreadPool {
  
      public static void main(String[] args) {
          ExecutorService executor = Executors.newFixedThreadPool(5);
          for (int i = 0; i < 10; i++) {
              Runnable worker = new WorkerThread("" + i);
              executor.execute(worker);
            }
          executor.shutdown(); // This will make the executor accept no new threads and finish all existing threads in the queue
          while (!executor.isTerminated()) { // Wait until all threads are finish,and also you can use "executor.awaitTermination();" to wait
          }
          System.out.println("Finished all threads");
      }
  
  }
  ```

程序中我们创建了固定大小为五个工作线程的线程池。然后分配给线程池十个工作，因为线程池大小为五，它将启动五个工作线程先处理五个工作，其他的工作则处于等待状态，一旦有工作完成，空闲下来工作线程就会捡取等待队列里的其他工作进行执行。

Executors 类提供了使用了 ThreadPoolExecutor 的简单的 ExecutorService 实现，但是 ThreadPoolExecutor 提供的功能远不止于此。我们可以在创建 ThreadPoolExecutor 实例时指定活动线程的数量，我们也可以限制线程池的大小并且创建我们自己的 RejectedExecutionHandler 实现来处理不能适应工作队列的工作。

### ThreadPoolExecutor详解

其实java线程池的实现原理很简单，说白了就是一个线程集合 workerSet 和一个阻塞队列 workQueue。当用户向线程池提交一个任务(也就是线程)时，线程池会先将任务放入 workQueue 中。workerSet 中的线程会不断的从 workQueue 中获取线程然后执行。当 workQueue 中没有任务的时候，worker 就会阻塞，直到队列中有任务了就取出来继续执行。

![img](../_images/pdai-java-thread-x-executors-1.png)

## 线程[参数](https://mp.weixin.qq.com/s?__biz=MzIxNTQ4MzE1NA==&mid=2247485631&idx=1&sn=b0d7cd3f337246c79cd08431d9a6d8ec&chksm=9796dec2a0e157d4b8a05b5bc1adcd53bc6ef81112cac5c7dc93370fbbc3baaab717aa5db628&scene=21#wechat_redirect)

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

### 线程池的使用

阿里 Java 开发手册 对线程池的使用进行了限制，可作参考：

 【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。

说明：使用线程池的好处是减少在创建和销毁线程上所花的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。

 【强制】线程池不允许使用**Executors**去创建，而是通过**ThreadPoolExecutor**的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

- Executors.newCachedThreadPool()：无界线程池，可以进行自动线程回收
- Executors.newFixedThreadPool(int)：固定大小的线程池
- Executors.newSingleThreadExecutor()：单个后台线程的线程池
- Executors.newScheduledThreadPool()：执行定时任务的线程池
- Executors.newWorkStealingPool(int)：支持并行执行的线程池

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

## 问答

> 参考：[10问10答：你真的了解线程池吗？](https://mp.weixin.qq.com/s/axWymUaYaARtvsYqvfyTtw)

**线程池创建之后，会立即创建核心线程么**

不会。从上面的源码可以看出，在刚刚创建ThreadPoolExecutor的时候，线程并不会立即启动，而是要等到有任务提交时才会启动，除非调用了prestartCoreThread/prestartAllCoreThreads事先启动核心线程。

**核心线程永远不会销毁么**

在JDK1.6之后，如果allowsCoreThreadTimeOut=true，核心线程也可以被终止。

**keepAliveTime=0会怎么样**

在JDK1.8中，keepAliveTime=0表示非核心线程执行完立刻终止。

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

## ThreadLocal

**提供线程局部变量，实现线程数据隔离**

ThreadLocal是一个将在多线程中为每一个线程创建单独的变量副本的类；当使用ThreadLocal来维护变量时，ThreadLocal会为每个线程创建单独的变量副本，避免因多线程操作共享变量而导致的数据不一致的情况。

**使用**

提到ThreadLocal被提到应用最多的是session管理和数据库链接管理。在Spring中提供了事务相关的操作，事务保证一组操作同时成功或者失败，这意味着我们一次事务的所有操作都需要再同一个数据库连接上。Spring使用ThreadLocal来实现连接共享；ThreadLocal的存储类型是一个Map，Map中的key是DataSource，value是Connection，用ThreadLocal保证了同一个线程获取同一个Connection对象，保证一次事务的所有操作在同一个数据库连接上。

**示例**

Session的管理

```java
private static final ThreadLocal threadSession = new ThreadLocal();  

public static Session getSession() throws InfrastructureException {  
    Session s = (Session) threadSession.get();  
    try {  
        if (s == null) {  
            s = getSessionFactory().openSession();  
            threadSession.set(s);  
        }  
    } catch (HibernateException ex) {  
        throw new InfrastructureException(ex);  
    }  
    return s;  
}  
```

### 容易用错的地方

> 内容来源：[细数ThreadLocal三大坑，内存泄露仅是小儿科](https://mp.weixin.qq.com/s/eWgTmP283kD_M2VxSxvYag)

- 内存泄露
- 线程池中线程上下文丢失
- 并行流中线程上下文丢失

#### 内存泄露

由于`ThreadLocal`的`key`是弱引用，因此如果使用后不调用`remove`清理的话会导致对应的`value`内存泄露。

```java
@Test
public void testThreadLocalMemoryLeaks() {
    ThreadLocal<List<Integer>> localCache = new ThreadLocal<>();
   List<Integer> cacheInstance = new ArrayList<>(10000);
    localCache.set(cacheInstance);
    localCache = new ThreadLocal<>();
}
```

当`localCache`的值被重置之后`cacheInstance`被`ThreadLocalMap`中的`value`引用，无法被GC，但是其`key`对`ThreadLocal`实例的引用是一个弱引用，本来`ThreadLocal`的实例被`localCache`和`ThreadLocalMap`的`key`同时引用，但是当`localCache`的引用被重置之后，则`ThreadLocal`的实例只有`ThreadLocalMap`的`key`这样一个弱引用了，此时这个实例在GC的时候能够被清理。

#### 线程池中线程上下文丢失

`ThreadLocal`不能在父子线程中传递，因此最常见的做法是把父线程中的`ThreadLocal`值拷贝到子线程中，因此大家会经常看到类似下面的这段代码：

```java
for(value in valueList){
     //提交任务，并设置拷贝Context到子线程
     Future<?> taskResult = threadPool.submit(new BizTask(ContextHolder.get()));
     results.add(taskResult);
}
for(result in results){
    result.get();//阻塞等待任务执行完成
}
```

提交的任务定义长这样：

```java
class BizTask<T> implements Callable<T>  {
    private String session = null;

    public BizTask(String session) {
        this.session = session;
    }

    @Override
    public T call(){
        try {
            ContextHolder.set(this.session);
            // 执行业务逻辑
        } catch(Exception e){
            //log error
        } finally {
            ContextHolder.remove(); // 清理 ThreadLocal 的上下文，避免线程复用时context互串
        }
        return null;
    }
}
```

对应的线程上下文管理类为：

```java
class ContextHolder {
    private static ThreadLocal<String> localThreadCache = new ThreadLocal<>();

    public static void set(String cacheValue) {
        localThreadCache.set(cacheValue);
    }

    public static String get() {
        return localThreadCache.get();
    }

    public static void remove() {
        localThreadCache.remove();
    }

}
```

这么写倒也没有问题，我们再看看线程池的设置：

```java
ThreadPoolExecutor executorPool = new ThreadPoolExecutor(20, 40, 30, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>(40), new XXXThreadFactory(), ThreadPoolExecutor.CallerRunsPolicy);
```

其中最后一个参数控制着当线程池满时，该如何处理提交的任务，内置有4种策略：

- ThreadPoolExecutor.AbortPolicy *//直接抛出异常*
- ThreadPoolExecutor.DiscardPolicy *//丢弃当前任务*
- ThreadPoolExecutor.DiscardOldestPolicy *//丢弃工作队列头部的任务*
- ThreadPoolExecutor.CallerRunsPolicy *//转串行执行*

可以看到，我们初始化线程池的时候指定如果线程池满，则新提交的任务转为串行执行，那我们之前的写法就会有问题了，串行执行的时候调用`ContextHolder.remove();`会将主线程的上下文也清理，即使后面线程池继续并行工作，传给子线程的上下文也已经是`null`了，而且这样的问题很难在预发测试的时候发现。

#### 并行流中线程上下文丢失

如果`ThreadLocal`碰到并行流，也会有很多有意思的事情发生，比如有下面的代码：

```java
class ParallelProcessor<T> {
    public void process(List<T> dataList) {
        // 先校验参数，篇幅限制先省略不写
        dataList.parallelStream().forEach(entry -> {
            doIt();
        });
    }
    private void doIt() {
        String session = ContextHolder.get();
        // do something
    }
}
```

这段代码很容易在线下测试的过程中发现不能按照预期工作，因为并行流底层的实现也是一个`ForkJoin`线程池，既然是线程池，那`ContextHolder.get()`可能取出来的就是一个`null`。我们顺着这个思路把代码再改一下：

```java
class ParallelProcessor<T> {

    private String session;

    public ParallelProcessor(String session) {
        this.session = session;
    }

    public void process(List<T> dataList) {
        // 先校验参数，篇幅限制先省略不写
        dataList.parallelStream().forEach(entry -> {
            try {
                ContextHolder.set(session);
                // 业务处理
                doIt();
            } catch (Exception e) {
                // log it
            } finally {
                ContextHolder.remove();
            }
        });
    }

    private void doIt() {
        String session = ContextHolder.get();
        // do something
    }
}
```

修改完后的这段代码可以工作吗？如果运气好，你会发现这样改又有问题，运气不好，这段代码在线下运行良好，这段代码就顺利上线了。不久你就会发现系统中会有一些其他很诡异的bug。原因在于并行流的设计比较特殊，父线程也有可能参与到并行流线程池的调度，那如果上面的`process`方法被父线程执行，那么父线程的上下文会被清理。导致后续拷贝到子线程的上下文都为`null`，同样产生丢失上下文的问题。
