- [线程池的作用](#%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%9A%84%E4%BD%9C%E7%94%A8)
- [线程池的实现原理](#%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86)
- [ThreadPoolExecutor的使用](#threadpoolexecutor%E7%9A%84%E4%BD%BF%E7%94%A8)
  - [创建线程池](#%E5%88%9B%E5%BB%BA%E7%BA%BF%E7%A8%8B%E6%B1%A0)
- [提交任务](#%E6%8F%90%E4%BA%A4%E4%BB%BB%E5%8A%A1)
- [关闭线程池](#%E5%85%B3%E9%97%AD%E7%BA%BF%E7%A8%8B%E6%B1%A0)
- [ThreadPoolExecutor运行机制](#threadpoolexecutor%E8%BF%90%E8%A1%8C%E6%9C%BA%E5%88%B6)
- [设置合理的线程池大小](#%E8%AE%BE%E7%BD%AE%E5%90%88%E7%90%86%E7%9A%84%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%A4%A7%E5%B0%8F)
- [Executor](#executor)
  - [Executor结构](#executor%E7%BB%93%E6%9E%84)
  - [线程池](#%E7%BA%BF%E7%A8%8B%E6%B1%A0)
## 线程池的作用
1. 减少资源的开销: 

    减少了每次创建线程、销毁线程的开销。
2. 提高响应速度: 

    每次请求到来时，由于线程的创建已经完成，故可以直接执行任务，因此提高了响应速度。

3. 提高线程的可管理性 

    线程是一种稀缺资源，若不加以限制，不仅会占用大量资源，而且会影响系统的稳定性。 
    因此，线程池可以对线程的创建与停止、线程数量等等因素加以控制，使得线程在一种可控的范围内运行，不仅能保证系统稳定运行，而且方便性能调优。

## 线程池的实现原理

线程池一般由两种角色构成：多个工作线程 和 一个阻塞队列。

1. 工作线程 

    工作线程是一组已经处在运行中的线程，它们不断地向阻塞队列中领取任务执行。

2. 阻塞队列 

    阻塞队列用于存储工作线程来不及处理的任务。当工作线程都在执行任务时，到来的新任务就只能暂时在阻塞队列中存储。

## ThreadPoolExecutor的使用

### 创建线程池

通过如下代码即可创建一个线程池：

```java
new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, timeUnit, runnableTaskQueue, handler);
```

- corePoolSize：基本线程数量 

    它表示你希望线程池达到的一个值。线程池会尽量把实际线程数量保持在这个值上下。

- maximumPoolSize：最大线程数量 

    这是线程数量的上界。 

    如果实际线程数量达到这个值：

    1. 阻塞队列未满：任务存入阻塞队列等待执行
    2. 阻塞队列已满：调用饱和策略

- keepAliveTime：空闲线程的存活时间 

    当实际线程数量超过corePoolSize时，若线程空闲的时间超过该值，就会被停止。 

    PS：当任务很多，且任务执行时间很短的情况下，可以将该值调大，提高线程利用率。

- timeUnit：keepAliveTime的单位

- runnableTaskQueue：任务队列 

    这是一个存放任务的阻塞队列，可以有如下几种选择： 
    1. ArrayBlockingQueue: 由数组实现的阻塞队列，FIFO。
    2. LinkedBlockingQueue: 链表实现的阻塞队列，FIFO。 吞吐量通常要高于ArrayBlockingQueue。是个无界队列。
    3. SynchronousQueue： 它是一个没有存储空间的阻塞队列，任务提交给它之后必须要交给一条工作线程处理；如果当前没有空闲的工作线程，则立即创建一条新的工作线程。 cachedThreadPool用的阻塞队列就是它。 
    它是一个无界队列。
    4. PriorityBlockingQueue: 优先权阻塞队列。

- handler：饱和策略 

    当实际线程数达到maximumPoolSize，并且阻塞队列已满时，就会调用饱和策略。 

    1. AbortPolicy: 默认。直接抛异常。
    2. CallerRunsPolicy:只用调用者所在的线程执行任务。
    3. DiscardOldestPolicy: 丢弃任务队列中最久的任务。
    4. DiscardPolicy: 丢弃当前任务。

## 提交任务

可以向ThreadPoolExecutor提交两种任务：Callable和Runnable。

1. Callable 

    该类任务有返回结果，可以抛出异常。通过submit函数提交，返回Future对象。 可通过get获取执行结果。

2. Runnable 

    该类任务只执行，无法获取返回结果，并在执行过程中无法抛异常。 通过execute提交。

## 关闭线程池
关闭线程池有两种方式：shutdown和shutdownNow，关闭时，会遍历所有的线程，调用它们的interrupt函数中断线程。但这两种方式对于正在执行的线程处理方式不同。

1. shutdown() 

    仅停止阻塞队列中等待的线程，那些正在执行的线程就会让他们执行结束。

2. shutdownNow() 

    不仅会停止阻塞队列中的线程，而且会停止正在执行的线程。

## ThreadPoolExecutor运行机制

当有请求到来时：

1. 若当前实际线程数量 少于 corePoolSize，即使有空闲线程，也会创建一个新的工作线程；
2. 若当前实际线程数量处于corePoolSize和maximumPoolSize之间，并且阻塞队列没满，则任务将被放入阻塞队列中等待执行；
3. 若当前实际线程数量 小于 maximumPoolSize，但阻塞队列已满，则直接创建新线程处理任务；
4. 若当前实际线程数量已经达到maximumPoolSize，并且阻塞队列已满，则使用饱和策略。


## 设置合理的线程池大小

任务一般可分为：CPU密集型、IO密集型、混合型，对于不同类型的任务需要分配不同大小的线程池。

1. CPU密集型任务 

    尽量使用较小的线程池，一般为CPU核心数+1。 
    因为CPU密集型任务使得CPU使用率很高，若开过多的线程数，只能增加上下文切换的次数，因此会带来额外的开销。

2. IO密集型任务 

    可以使用稍大的线程池，一般为2*CPU核心数。 
    IO密集型任务CPU使用率并不高，因此可以让CPU在等待IO的时候去处理别的任务，充分利用CPU时间。

3. 混合型任务 

    可以将任务分成IO密集型和CPU密集型任务，然后分别用不同的线程池去处理。 
    只要分完之后两个任务的执行时间相差不大，那么就会比串行执行来的高效。 
    因为如果划分之后两个任务执行时间相差甚远，那么先执行完的任务就要等后执行完的任务，最终的时间仍然取决于后执行完的任务，而且还要加上任务拆分与合并的开销，得不偿失。


## Executor

<div align="center"> <img src="https://leanote.com/api/file/getImage?fileId=58ae9edfab644146d000125b"/> </div><br>
 
在HotSpot虚拟机中，Java中的线程将会被一一映射为操作系统的线程。 
在Java虚拟机层面，用户将多个任务提交给Executor框架，Executor负责分配线程执行它们； 
在操作系统层面，操作系统再将这些线程分配给处理器执行。

### Executor结构
 
Executor框架中的所有类可以分成三类：

<div align="center"> <img src="https://leanote.com/api/file/getImage?fileId=58aea128ab6441490b000fc5"/> </div><br>


1. 任务 

    任务有两种类型：Runnable和Callable。

2. 任务执行器 

    Executor框架最核心的接口是Executor，它表示任务的执行器。 

    Executor的子接口为ExecutorService。 

    ExecutorService有两大实现类：ThreadPoolExecutor和ScheduledThreadPoolExecutor。

3. 执行结果 

    Future接口表示异步的执行结果，它的实现类为FutureTask。

### 线程池
Executors工厂类可以创建四种类型的线程池，通过Executors.newXXX即可创建。

1. FixedThreadPool
```java
public static ExecutorService newFixedThreadPool(int nThreads){
    return new ThreadPoolExecutor(nThreads,nThreads,0L,TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());
}
```

- 它是一种固定大小的线程池；
- corePoolSize和maximunPoolSize都为用户设定的线程数量nThreads；
- keepAliveTime为0，意味着一旦有多余的空闲线程，就会被立即停止掉；但这里keepAliveTime无效；
- 阻塞队列采用了LinkedBlockingQueue，它是一个无界队列；
- 由于阻塞队列是一个无界队列，因此永远不可能拒绝任务；
- 由于采用了无界队列，实际线程数量将永远维持在nThreads，因此maximumPoolSize和keepAliveTime将无效。

2. CachedThreadPool

```java
public static ExecutorService newCachedThreadPool(){
    return new ThreadPoolExecutor(0,Integer.MAX_VALUE,60L,TimeUnit.MILLISECONDS,new SynchronousQueue<Runnable>());
}
```


- 它是一个可以无限扩大的线程池；
- 它比较适合处理执行时间比较小的任务；
- corePoolSize为0，maximumPoolSize为无限大，意味着线程数量可以无限大；
- keepAliveTime为60S，意味着线程空闲时间超过60S就会被杀死；
- 采用SynchronousQueue装等待的任务，这个阻塞队列没有存储空间，这意味着只要有请求到来，就必须要找到一条工作线程处理他，如果当前没有空闲的线程，那么就会再创建一条新的线程。

3. SingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor(){
    return new ThreadPoolExecutor(1,1,0L,TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());
}
```


- 它只会创建一条工作线程处理任务；
- 采用的阻塞队列为LinkedBlockingQueue；

4. ScheduledThreadPool

- 它用来处理延时任务或定时任务。 
- 它接收SchduledFutureTask类型的任务，有两种提交任务的方式：

    1. scheduledAtFixedRate

    2. scheduledWithFixedDelay

    3. SchduledFutureTask

    **接收的参数：**

    - time：任务开始的时间
    - sequenceNumber：任务的序号
    - period：任务执行的时间间隔
    - 它采用DelayQueue存储等待的任务

    - DelayQueue内部封装了一个PriorityQueue，它会根据time的先后时间排序，若time相同则根据sequenceNumber排序；
    - DelayQueue也是一个无界队列；

    **工作线程的执行过程：**

    - 工作线程会从DelayQueue取已经到期的任务去执行；
    - 执行结束后重新设置任务的到期时间，再次放回DelayQueue

