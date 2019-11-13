
<!-- TOC -->

- [1. **Base**](#1-base)
    - [1.1 为什么要用到并发？（优点）](#11-为什么要用到并发优点)
    - [1.2 并发编程的缺点及解决？](#12-并发编程的缺点及解决)
    - [1.3 异步VS同步？](#13-异步vs同步)
    - [1.4 并发VS并行](#14-并发vs并行)
    - [1.5 临界区](#15-临界区)
- [2. **Base-Thread**](#2-base-thread)
    - [1.1 创建Thread的几种方式？](#11-创建thread的几种方式)
    - [1.2 Thread的状态及转换？](#12-thread的状态及转换)
    - [1.3 Thread几个主要方法？](#13-thread几个主要方法)
    - [1.4 Daemon守护线程及使用注意点？](#14-daemon守护线程及使用注意点)
- [3. **Base-JMM及重排序**](#3-base-jmm及重排序)
    - [1.1 JMM内存结构？](#11-jmm内存结构)
    - [1.2 重排序？](#12-重排序)
    - [1.3 happen-before规则了解？](#13-happen-before规则了解)
- [4. **Base-Synchronize**](#4-base-synchronize)
    - [1.1 Synchronize使用？](#11-synchronize使用)
    - [1.2 monitor机制？](#12-monitor机制)
    - [1.3 Synchronize内存语义？](#13-synchronize内存语义)
    - [1.4 锁优化？](#14-锁优化)
    - [1.5 锁升级逻辑？](#15-锁升级逻辑)
- [5. **Base-volatile**](#5-base-volatile)
    - [1.1 使用原理？](#11-使用原理)
    - [1.2 内存语义？](#12-内存语义)
    - [1.3 内存语义的实现？（内存屏障）](#13-内存语义的实现内存屏障)
- [6. **Base-三大性质**](#6-base-三大性质)
    - [1.1 三大性质](#11-三大性质)
    - [1.2 Synchronize&volatile的三大性质](#12-synchronizevolatile的三大性质)
- [7. **Concurrent包的结构层次**](#7-concurrent包的结构层次)
        - [结构层次](#结构层次)
- [7.1 **Lock**](#71-lock)
    - [1.1 API](#11-api)
    - [1.2 使用及对比Object锁](#12-使用及对比object锁)
    - [1.3 锁特性](#13-锁特性)
- [7.2 **AQS**](#72-aqs)
    - [1.1 理解 & 实现](#11-理解--实现)
    - [1.2 重写方法](#12-重写方法)
    - [1.3 模板方法（分类）](#13-模板方法分类)
    - [1.4 内部同步队列及与AQS联系](#14-内部同步队列及与aqs联系)
    - [1.5 同步状态的管理（重写时）](#15-同步状态的管理重写时)
    - [1.6 独占锁获取方式（逻辑图）](#16-独占锁获取方式逻辑图)
    - [1.7 独占式释放方式](#17-独占式释放方式)
    - [1.8 共享锁](#18-共享锁)
- [7.3 **ReentrantLock**](#73-reentrantlock)
    - [1.1 重入锁的实现原理](#11-重入锁的实现原理)
    - [1.2 公平锁的实现原理](#12-公平锁的实现原理)
    - [1.4 非公平锁的实现原理](#14-非公平锁的实现原理)
    - [1.5 公平锁VS非公平锁](#15-公平锁vs非公平锁)
- [7.4 ReentrantWriteReadLock](#74-reentrantwritereadlock)
    - [1.1 读写锁是怎样实现分别记录读写状态的？](#11-读写锁是怎样实现分别记录读写状态的)
    - [1.2 锁降级](#12-锁降级)
- [7.5 **Condition**](#75-condition)
    - [1.1 Object的wait VS notify/notify](#11-object的wait-vs-notifynotify)
    - [1.2 等待队列](#12-等待队列)
    - [1.3 结合使用分析Condition实现（await/notify）](#13-结合使用分析condition实现awaitnotify)
- [7.6 **LockSupport工具**](#76-locksupport工具)
    - [1.1 主要方法](#11-主要方法)
- [7.7 **concurrentHashMap**](#77-concurrenthashmap)
    - [1.1 initTable()](#11-inittable)
    - [1.1 put()](#11-put)
    - [1.1 get()](#11-get)
- [7.8 **CopyOnWriteArrayList**](#78-copyonwritearraylist)
- [7.11 **CountDownLatch**](#711-countdownlatch)
- [7.12 **CyclicBarrier**](#712-cyclicbarrier)
- [7.13 **Semaphore**](#713-semaphore)
- [7.14 **Exchanger**](#714-exchanger)
- [8. **线程池（ThreadPoolExecutor）**](#8-线程池threadpoolexecutor)
    - [8.1 线程池的好处？](#81-线程池的好处)
    - [8.2 线程池的工作原理](#82-线程池的工作原理)
    - [8.3 创建参数](#83-创建参数)
    - [8.4 如何合理配置线程池参数？](#84-如何合理配置线程池参数)
    - [8.5 ScheduledThreadPoolExecutor](#85-scheduledthreadpoolexecutor)
    - [8.6 FutureTask](#86-futuretask)
- [9. Atomic](#9-atomic)

<!-- /TOC -->
## 1. **Base**
### 1.1 为什么要用到并发？（优点）
* 充分发挥多线程的运算能力
* 方便复杂业务拆分

### 1.2 并发编程的缺点及解决？
* 频繁的上下文切换
	+ 无锁编程
	+ 乐观锁CAS
	+ 携程
* 线程安全
	+ 避免死锁的方法？

### 1.3 异步VS同步？
调用方**用不用**管被调用方，继续执行代码

### 1.4 并发VS并行
并发指的是多个任务交替进行，而并行则是指真正意义上的“同时进行”

### 1.5 临界区
一种公共资源或者说是共享数据，可以被多个线程使用。

## 2. **Base-Thread**
### 1.1 创建Thread的几种方式？
### 1.2 Thread的状态及转换？
### 1.3 Thread几个主要方法？
### 1.4 Daemon守护线程及使用注意点？

## 3. **Base-JMM及重排序**
### 1.1 JMM内存结构？
### 1.2 重排序？
### 1.3 happen-before规则了解？

## 4. **Base-Synchronize**
### 1.1 Synchronize使用？
### 1.2 monitor机制？
### 1.3 Synchronize内存语义？
### 1.4 锁优化？
### 1.5 锁升级逻辑？

## 5. **Base-volatile**
### 1.1 使用原理？
### 1.2 内存语义？
### 1.3 内存语义的实现？（内存屏障）

## 6. **Base-三大性质**
### 1.1 三大性质
### 1.2 Synchronize&volatile的三大性质

## 7. **Concurrent包的结构层次**

#### 结构层次
* 底层：CAS volatile
* 中间层：AQS 非阻塞容器 原子变量类
* 上层：Lock 同步组件 阻塞队列 Executor 并发容器

## 7.1 **Lock**
### 1.1 API
* void lock(); //获取锁
* void lockInterruptibly() throws InterruptedException；//获取锁的过程能够响应中断
* boolean tryLock();//非阻塞式响应中断能立即返回，获取锁放回true反之返回fasle
* boolean tryLock(long time, TimeUnit unit) throws InterruptedException;//超时获取锁，在超时内或者未中断的情况下能够获取锁
* Condition newCondition();//获取与lock绑定的等待通知组件

### 1.2 使用及对比Object锁
* synchronized同步块执行完成或者遇到异常是锁会自动释放
* lock必须调用unlock()方法释放锁，因此在finally块中释放锁

### 1.3 锁特性
* 公平性选择：支持非公平性（默认）和公平的锁获取方式，吞吐量还是非公平优于公平；
* 重入性：支持重入，读锁获取后能再次获取，写锁获取之后能够再次获取写锁，同时也能够获取读锁；
* 锁降级：遵循获取写锁，获取读锁再释放写锁的次序，写锁能够降级成为读锁（读写锁）

## 7.2 **AQS**
### 1.1 理解 & 实现
* 同步器是面向锁的实现者，它简化了锁的实现方式，屏蔽了同步状态的管理，线程的排队，等待和唤醒等底层操作。
* 锁是面向使用者，它定义了使用者与锁交互的接口，隐藏了实现细节；
* 推荐定义为自定义同步组件的静态内部类

### 1.2 重写方法
* protected boolean tryAcquire(int arg);
* protected boolean tryRelease(int arg);
* protected int tryAcquireShared(int arg);
* protected boolean tryReleaseShared(int arg);
* protected boolean isHeldExclusively();

### 1.3 模板方法（分类）
* 独占式获取与释放同步状态；
	+ void acquire(int arg);
	+ void acquireInterruptibly(int arg);
    + boolean tryAcquireNanos(int arg, long nanos);
	+ boolean release(int arg);
* 共享式获取与释放同步状态；
	+ void acquireShared(int arg);
	+ void acquireSharedInterruptibly(int arg);
	+ boolean tryAcquireSharedNanos(int arg, long nanos);
	+ boolean releaseShared(int arg);
* 查询同步队列中等待线程情况；
	+ Collection\<Thread> getQueuedThreads();

### 1.4 内部同步队列及与AQS联系
同步队列是一个双向队列，AQS通过持有头尾指针管理同步队列；
* Node
	+ volatile int waitStatus //节点状态
	+ volatile Node prev //当前节点/线程的前驱节点
	+ volatile Node next; //当前节点/线程的后继节点
	+ volatile Thread thread;//加入同步队列的线程引用
	+ Node nextWaiter;//等待队列中的下一个节点
* waitStatus
	+ int INITIAL = 0;//初始状态
	+ int CANCELLED =  1//节点从同步队列中取消
	+ int SIGNAL    = -1//后继节点的线程处于等待状态，如果当前节点释放同步状态会通知后继节点，使得后继节点的线程能够运行；
	+ int CONDITION = -2//当前节点进入等待队列中
	+ int PROPAGATE = -3//表示下一次共享式同步状态获取将会无条件传播下去
* AQS
	+ private transient volatile Node head;
	+ private transient volatile Node tail;
	
### 1.5 同步状态的管理（重写时）
* private volatile int state;
* getState()
* setState()
* compareAndSetState()

### 1.6 独占锁获取方式（逻辑图）
* 1.先看同步状态是否获取成功，
	+ 1.1如果成功则方法结束返回
	+ 1.2失败，入队操作,继续
* 2.将当前线程构建成Node类型
* 3.当前尾节点是否为null？
	+ 3.1不为null，将当前节点尾插入的方式插入同步队列中，返回
	+ 3.2为空
		- 1.构造头结点*
		- 2.尾插入，CAS操作失败自旋尝试
* 4. 获得当前节点的先驱节点
* 5. 当前节点能否获取独占式锁					
* 5.1 如果当前节点的先驱节点是头结点并且成功获取同步状态，即可以获得独占
* 5.2 获取锁失败，线程进入等待状态等待获取独占式锁

### 1.7 独占式释放方式
### 1.8 共享锁

## 7.3 **ReentrantLock**
### 1.1 重入锁的实现原理
* 检查占有线程是否是当前线程，是的话计数加一
* 释放的时候计数减一，当同步状态为0时，锁成功被释放

### 1.2 公平锁的实现原理
保证请求资源时间上的绝对顺序
* 如果一个锁是公平的，那么锁的获取顺序就应该符合请求上的绝对时间顺序，满足FIFO

### 1.4 非公平锁的实现原理
保证了系统更大的吞吐量。
* 可能刚释放锁的线程能再次获取到锁。

### 1.5 公平锁VS非公平锁

## 7.4 ReentrantWriteReadLock 
### 1.1 读写锁是怎样实现分别记录读写状态的？
通过同步状态（AQS-state），高16位读，低16位写

### 1.2 锁降级

## 7.5 **Condition**
### 1.1 Object的wait VS notify/notify
* Condition能够支持不响应中断，而通过使用Object方式不支持；
* Condition能够支持多个等待队列（new 多个Condition对象），而Object方式只能支持一个；
* Condition能够支持超时时间的设置，而Object不支持

### 1.2 等待队列
等待队列是一个不带头节点的单向队列

### 1.3 结合使用分析Condition实现（await/notify）
* 锁lock
* 同步队列
* 等待队列

## 7.6 **LockSupport工具**
### 1.1 主要方法
synchronzed致使线程阻塞，线程会进入到BLOCKED状态，而调用LockSupprt方法阻塞线程会致使线程进入到WAITING状态。
* void park()：阻塞当前线程，如果调用unpark方法或者当前线程被中断，从能从park()方法中返回
* void park(Object blocker)：功能同方法1，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；
* void parkNanos(long nanos)：阻塞当前线程，最长不超过nanos纳秒，增加了超时返回的特性；
* void parkNanos(Object blocker, long nanos)：功能同方法3，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；
* void parkUntil(long deadline)：阻塞当前线程，知道deadline；
* void parkUntil(Object blocker, long deadline)：功能同方法5，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；
* void unpark(Thread thread):唤醒处于阻塞状态的指定线程

## 7.7 **concurrentHashMap**
### 1.1 initTable()

### 1.1 put()
* 首先对于每一个放入的值，首先利用spread方法对key的hashcode进行一次hash计算，由此来确定这个值在table中的位置；
* 如果当前table数组还未初始化，先将table数组进行初始化操作；
* 如果这个位置是null的，那么使用CAS操作直接放入；
* 如果这个位置存在结点，说明发生了hash碰撞，首先判断这个节点的类型。如果该节点fh==MOVED(代表forwardingNode,数组正在进行扩容)的话，说明正在进行扩容；
* 如果是链表节点（fh>0）,则得到的结点就是hash值相同的节点组成的链表的头节点。需要依次向后遍历确定这个新加入的值所在位置。如果遇到key相同的节点，则只需要覆盖该结点的value值即可。否则依次向后遍历，直到链表尾插入这个结点；
* 如果这个节点的类型是TreeBin的话，直接调用红黑树的插入方法进行插入新的节点；
* 插入完节点之后再次检查链表长度，如果长度大于8，就把这个链表转换成红黑树；
* 对当前容量大小进行检查，如果超过了临界值（实际大小*加载因子）就需要扩容。

### 1.1 get()

## 7.8 **CopyOnWriteArrayList**

## 7.11 **CountDownLatch**
* await() throws InterruptedException：调用该方法的线程等到构造方法传入的N减到0的时候，才能继续往下执行；
* await(long timeout, TimeUnit unit)：与上面的await方法功能一致，只不过这里有了时间限制，调用该方法的线程等到指定的timeout时间后，不管N是否减至为0，都会继续往下执行；
* countDown()：使CountDownLatch初始值N减1；
* long getCount()：获取当前CountDownLatch维护的值；

## 7.12 **CyclicBarrier**
凑齐了一波，然后才能继续执行，否则每个线程都得阻塞等待，直至凑齐一波即可
```
//等到所有的线程都到达指定的临界点
await() throws InterruptedException, BrokenBarrierException 

//与上面的await方法功能基本一致，只不过这里有超时限制，阻塞等待直至到达超时时间为止
await(long timeout, TimeUnit unit) throws InterruptedException, 
BrokenBarrierException, TimeoutException 

//获取当前有多少个线程阻塞等待在临界点上
int getNumberWaiting()

//用于查询阻塞等待的线程是否被中断
boolean isBroken()

	
//将屏障重置为初始状态。如果当前有线程正在临界点等待的话，将抛出BrokenBarrierException。
void reset()
```

## 7.13 **Semaphore**
可以理解为信号量，用于控制资源能够被并发访问的线程数量，以保证多个线程能够合理的使用特定资源。


```
//获取许可，如果无法获取到，则阻塞等待直至能够获取为止
void acquire() throws InterruptedException 

//同acquire方法功能基本一样，只不过该方法可以一次获取多个许可
void acquire(int permits) throws InterruptedException

//释放许可
void release()

//释放指定个数的许可
void release(int permits)

//尝试获取许可，如果能够获取成功则立即返回true，否则，则返回false
boolean tryAcquire()

//与tryAcquire方法一致，只不过这里可以指定获取多个许可
boolean tryAcquire(int permits)

//尝试获取许可，如果能够立即获取到或者在指定时间内能够获取到，则返回true，否则返回false
boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException

//与上一个方法一致，只不过这里能够获取多个许可
boolean tryAcquire(int permits, long timeout, TimeUnit unit)

//返回当前可用的许可证个数
int availablePermits()

//返回正在等待获取许可证的线程数
int getQueueLength()

//是否有线程正在等待获取许可证
boolean hasQueuedThreads()

//获取所有正在等待许可的线程集合
Collection<Thread> getQueuedThreads()
```

## 7.14 **Exchanger**

```
//当一个线程执行该方法的时候，会等待另一个线程也执行该方法，因此两个线程就都达到了同步点
//将数据交换给另一个线程，同时返回获取的数据
V exchange(V x) throws InterruptedException

//同上一个方法功能基本一样，只不过这个方法同步等待的时候，增加了超时时间
V exchange(V x, long timeout, TimeUnit unit)
    throws InterruptedException, TimeoutException 
```

## 8. **线程池（ThreadPoolExecutor）**

### 8.1 线程池的好处？
* 降低资源消耗。通过复用已存在的线程和降低线程关闭的次数来尽可能降低系统性能损耗；
* 提升系统响应速度。通过复用线程，省去创建线程的过程，因此整体上提升了系统的响应速度；
* 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，因此，需要使用线程池来管理线程。

### 8.2 线程池的工作原理
* 1 核心线程数corePoolSize是否已满
	+ 是：2
	+ 否：创建线程执行任务
* 2 阻塞队列是否已满
	+ 是：3
	+ 否：将任务存储到阻塞队列
* 3 线程池中线程maximumPoolSize是否已满
	+ 是：饱和策略
	+ 否：创建线程执行任务

### 8.3 创建参数
* corePoolSize：表示核心线程池的大小。如果调用了prestartCoreThread()或者 prestartAllCoreThreads()，线程池创建的时候所有的核心线程都会被创建并且启动。
* maximumPoolSize：表示线程池能创建线程的最大个数。
* keepAliveTime：空闲线程存活时间。大于核心线程的线程。
* unit：时间单位。为keepAliveTime指定时间单位。
* workQueue：阻塞队列。
* threadFactory：创建线程的工程类。
* handler：饱和策略。采用的策略有这几种：
	+ AbortPolicy： 直接拒绝所提交的任务，并抛出RejectedExecutionException异常；
	+ CallerRunsPolicy：只用调用者所在的线程来执行任务；
	+ DiscardPolicy：不处理直接丢弃掉任务；
	+ DiscardOldestPolicy：丢弃掉阻塞队列中存放时间最久的任务，执行当前任务

### 8.4 如何合理配置线程池参数？
* 任务的性质：CPU密集型任务(Ncpu + 1)，IO密集型任务(2 x Ncpu)和混合型任务(拆分)。
* 任务的优先级：高，中和低（PriorityBlockingQueue）。
* 任务的执行时间：长，中和短（不同的线程池或者按照时间长短设置优先级）。
* 任务的依赖性：是否依赖其他系统资源，如数据库连接（线程数应该设置越大）。


### 8.5 ScheduledThreadPoolExecutor
用来在给定延时后执行异步任务或者周期性执行任务
```
//达到给定的延时时间后，执行任务。这里传入的是实现Runnable接口的任务，
//因此通过ScheduledFuture.get()获取结果为null
public ScheduledFuture<?> schedule(Runnable command,
                                       long delay, TimeUnit unit);
//达到给定的延时时间后，执行任务。这里传入的是实现Callable接口的任务，
//因此，返回的是任务的最终计算结果
 public <V> ScheduledFuture<V> schedule(Callable<V> callable,
                                           long delay, TimeUnit unit);

//是以上一个任务开始的时间计时，period时间过去后，
//检测上一个任务是否执行完毕，如果上一个任务执行完毕，
//则当前任务立即执行，如果上一个任务没有执行完毕，则需要等上一个任务执行完毕后立即执行
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);
//当达到延时时间initialDelay后，任务开始执行。上一个任务执行结束后到下一次
//任务执行，中间延时时间间隔为delay。以这种方式，周期性执行任务。
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);
```

### 8.6 FutureTask

## 9. Atomic
CAS、ABA问题(AtomicStampedReference)