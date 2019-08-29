## Java并发编程的艺术

### 第1章 并发编程的挑战
上下文切换：CPU通过时间片分配算法来循环执行任务。在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这个任务的状态。所以任务从保存到再加载的过程就是一次上下文切换。
线程有创建和上下文切换的开销

一个死锁的例子：demo

避免死锁的几个常见方法： 
* 避免一个线程同时获取多个锁 
* 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源 
* 尝试使用定时锁，使用lock.tryLock(timeout)来代替使用内部锁机制 
* 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况。

### 第2章 Java并发机制的底层实现原理
volatile是轻量级的synchronized，他在多处理器开发中保证了共享变量的“可见性”

synchronized实现同步的基础：Java中每一个对象都可以作为锁 
* 对于普通同步方法，锁是当前实例对象 
* 对于静态同步方法，锁是当前的Class 
* 对于同步方法块， 锁是Synchronized括号里配置的对象
### 第3章 Java内存模型

### 第4章 Java并发编程基础
线程简介
现代操作系统调度的最小单元是线程，也叫轻量级进程，在一个进程里可以创建多个线程。

为什么要使用多线程
更多的处理器核心
更快的响应时间
更好的编程模型
线程优先级
在Java线程中，通过一个整形成员变量priority来控制优先级，优先级的范围从1~10，可以通过setPriority(10)方法修改优先级，默认是5.优先级高的线程分配的时间片的数量要多于优先级低的线程。

线程的优先级不能作为程序正确性的依赖，因为操作系统可以完全不用理会Java对于优先级的设定。

Daemon线程
Daemon线程被用作完成支持性工作，但是在Java虚拟机退出时Daemon线程中的finally块并不一定会执行。 
在构建Daemon线程时，不能依靠finally块中的内容来确保执行关闭或清理资源的逻辑

启动和终止线程
构造线程
一个新构造的线程对象是由其parent线程来进行空间分配的，而child线程继承了parent是否为Daemon、优先级和加载资源的contextClassLoader以及可继承的ThreadLocal，同时还会分配一个唯一的ID来标识这个child线程。

启动线程
调用start()方法可以启动线程。

理解中断
中断表示一个运行中的线程是否被其他线程进行了中断操作。可以通过isInterrupted()来判断是否被中断。

过期的suspend()、resume()和stop();
线程的暂停、恢复、终止。suspeng()方法在调用后线程不会释放已经占有的资源，容易引发死锁问题。

暂停和恢复操作可以用等待/通知机制来代替。

安全地终止线程

线程间通信
volatile和synchronized关键字
关键字volatile能够保证所有线程对变量访问的可见性。
关键字synchronized主要确保多个线程在同一个时刻，只能有一个线程处于方法或者同步快中，它保证了线程对变量访问的可见性和排他性。
等待/通知机制

Thread.join()的使用
如果一个线程A执行了thread.join()语句，含义是：当前线程A等待thread线程终止之后才从thread.join()返回。

当线程终止时，会调用线程自身的notifyAlL()方法，会通知所有等待在该线程对象上的线程。

ThreadLocal的使用
ThreadLocal即线程变量，一个线程可以根据一个ThreadLocal对象查询到绑定在这个线程上的一个值。

线程应用实例
线程池技术
操作系统频繁的进行线程的上下文切换，会无故增加系统的负载，线程的创建和消亡都是需要耗费系统资源的。

线程池技术能够很好解决这个问题。

### 第5章 Java中的锁
Lock接口
Lock相比Synchronized缺少了隐式获取释放锁的便捷性，但是却用了锁获取与释放的可操作性、可中断的获取锁以及超时获取锁等同步特性。 

在finally中释放锁，目的是保证在获取到锁之后，最终能够被释放。

队列同步器
队列同步器AbstractQueuedSynchronizer，是用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作。

独占锁：是在同一时刻只能有一个线程获取到锁，而其他获取锁的线程只能处于同步队列中等待，只有获取锁的线程释放了锁，后续的线程才能获取锁。

在获取同步状态时，同步器维护一个同步队列，获取状态失败的线程都会被加入到队列中并在队列中进行自旋；移除队列(或停止自旋)的条件是前驱结点为头结点且成功获取了同步状态。在释放同步状态时，同步器调用tryRelease(int arg)方法同步状态，然后唤醒头结点的后继节点。

共享式获取与独占式获取最主要的区别在于同一时刻能够有很多个线程同时获取到同步状态。 
通过调用同步器的acquireShare(int arg)方法可以共享式地获取同步状态。

重入锁
重入锁ReentrantLock，支持重进入得锁，表示该锁能够支持一个线程对资源的重复加锁。该锁还支持获取锁时的公平和非公平的选择。通过判断当前线程是否为获取锁的线程来决定获取操作是否成功。非公平锁只需要CAS设置同步状态成功，则表示当前线程获取了锁，而非公平所还需要判断加入了同步队列中的当前节点是否有前驱节点的判断 
* 实现重进入，重进入是指任意线程在获取到锁之后能够再次获取该锁而不会被锁所阻塞。

公平锁保证了锁的获取是按照FIFO原则，而代价是进行大量的县城切换。非公平性锁虽然可能造成线程饥饿，但极少的线程切换，保证了更大的吞吐量。

读写锁
读写锁在同一时刻可以允许多个读线程访问。当写锁被获取到时，后续的读写操作都会被阻塞。

读写锁能够提供比排它锁更好的并发性和吞吐量。

只有等待其他度线程都释放了读锁，写锁才能被当前线程获取。

Condition接口
Condition接口提供了监视器方法与Lock配合可以实现等待、通知模式

### 第6章 Java并发容器和框架 

ConcurrentHashMap的结构
一个ConcurrentHashMap里包含了一个Segment数组。Segment的结构和HashMap类似，是一种数组和链表结构。一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素。

ConcurrentHashMap不会对整个容器进行扩容，而只是对某个Segment进行扩容

ConcurrentLinkedQueue
ConcurrentLinkQueue是一个基于链接节点的无界线程安全队列.

ConcurrentLinkQueue由head节点和tail节点组成。默认情况下head节点存储的元素为空，tail节点等于head节点。

入队列
第一将入队列节点设置成当前队列尾节点的下一个节点；第二是更新tail节点，如果tail节点的next节点不为空，则将入队节点设置成tail节点，如果tail节点的next为空，则将入队节点设置成tail的next节点，所以tail节点不总是尾节点。

出队列
首先获取头节点的元素，然后判断头节点元素是否为空，如果为空，表示另外一个线程已经进行了一次出队操作将该节点取走，如果不为空，则使用CAS的方式将头节点的引用设置成null，如果CAS成功，则直接返回头节点的元素，如果不成功，表示另外一个线程已经进行了一次出队操作更新了head节点，导致元素发生了变化，需要重新获取头节点。

Java中的阻塞队列
什么是阻塞队列
阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。 
* 支持阻塞的插入方法：队列满时，队列会阻塞插入元素的线程，知道队列不满。 
* 支持阻塞的移除方法：队列为空时，获取元素的线程会等待队列变为非空。

ArrayBlockQueue
用数组实现的有界阻塞队列。此队列按照FIFO的原则对元素进行排序。==默认不保证线程公平的访问队列==

LinkedBlockQueue
用链表实现的有界阻塞队列。按照FIFO的原则排序。默认最大长度为Integer.MAX_VALUE

PriorityBlockingQueue
支持优先级的无界阻塞队列。元素采取自然顺序升序排列。也可以自定义类实现compareTo()方法指定排序规则。不能保证同优先级元素的顺序。

DelayQueue
支持延迟获取元素的无界阻塞队列。队列使用PriorityQueue来实现。队列中的元素必须实现Delay接口，在创建元素时指定多久才能从队列中获取当前元素。

SynchronousQueue
SynchronousQueue是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。

SynchronousQueue可以看成一个传球手，负责把生产者线程处理的数据直接创递给消费者线程。

LinkedTransferQueue
是一个由链表结构组成的无界阻塞Transfer队列。相对于其他阻塞队列，LinkedTransferQueue多了tryTransfer和transfer方法。

LinkedBlockingQueue
由链表结构组成的双向阻塞队列。可以从队列的两端插入和移除元素。

阻塞队列的实现原理
使用通知模式实现。当生产者往满的队列里添加元素时会阻塞住生产者，当消费者消费了一个队列中农工的元素后，会通知生产者当前队列可用。

Fork/Join框架
一个把大任务分割成若干小任务，最终汇总每个小任务结果后得到大任务结果的框架。

Fork/Join框架的实现原理
ForkJoinPool由ForkJoinPool数组和ForkJoinWorkerThread数组组成，ForkJoinTask数组负责将存放程序提交给ForkJoinPool的任务，而ForkJoinWorkerThread数组负责执行这些任务。

调用ForkJoinTask的fork方法，会调用ForkJoinWorkerThread的push方法将数组中的该任务放到工作队列中，再调用sinalWork()方法唤醒或创建一个工作线程来执行任务。

调用join方法，首先通过查看任务状态，看任务是否已经执行完成，如果执行完成，则直接返回任务状态；没有完成，从任务数组里取出任务并执行。

### 第7章 Java中的13个原子操作类
原子更新基本类型类
AtomicBoolean：原子更新布尔类型
AtomicInteger：原子更新整形
AtomicLong：原子更新长整型
原子更新数组
AtomicIntegerArray：原子更新整形数组里的元素。
AtomicLongArray：原子更新长整型数组里的元素。
AtomicReferenceArray：原子更新引用类型数组里的元素。

数组value通过构造方法传递进去，然后AtomicIntegerArray会将当前数组复制一份，所以当AtomicIntegerArray对内部数组元素进行修改时，不会影响传入的数组。

原子更新引用类型
AtomicReference：原子更新引用类型。
AtomicReferenceFieldUpdater：原子更新引用类型里的字段。
AtomicMarkableReference：原子更新带有标记位的引用类型。

原子更新字段类
AtomicIntegerFieldUpdater：原子更新整形的字段的更新器。
AtomicLongFieldUpdater：原子更细长整型字段的更新器。
AtomicStampedReference：原子更新带有版本号的引用类型。可用于原子的更新数据和数据的版本号，可以解决使用CAS进行原子更新时可能出现ABA问题。
每次使用的时候必须使用静态方法newUpdater()创建一个更新器，并且需要设置想要更新的类和属性。更新类的字段必须使用public volatile修饰符。

### 第8章 Java中的并发工具类
等待多线程完成的CountDownLatch 
当调用CountDownLatch的countDown方法是，N就会减1

同步屏障CyclicBarrier
让一组线程到达一个屏障时被阻塞，知道最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。

每个线程调用await方法告诉CyclicBarrier我已经到达了屏障。

CyclicBarrier还提供一个更高级的构造函数CyclicBarrier(int parties, Runnable barrierAction)用于在线程到达屏障时，优先执行barrierAction

CyclicBarrier和CountDownLatch的区别
CountDownLatch的计数器只能使用一次，而CyclicBarrier的计数器可以使用reset()方法重置。 
CyclicBarrier还提供其他有用的方法，getNumberWaiting方法获取CyclicBarrier阻塞的线程数量。isBroken()方法用来了解阻塞线程是否被中断。

控制并发线程数的Semaphore
Semaphore是用来控制同时访问特定资源的线程数量

使用Semaphore的acquire()方法获取一个许可证，使用完之后调用release()方法归还许可证。还可以用tryAcquire()方法尝试获取许可证。

### 第9章 Java中的线程池
线程池实现原理
如果当前运行的线程少于coolPoolSize，则创建新线程来执行任务
如果运行的线程等于或多于coolPoolSize，则将任务加入BlockingQueue
如果无法将任务加入BlockingQueue，则创建新的线程来处理任务
如果创建新线程将使当前运行的线程超出maximumPoolSize，任务将被拒绝，并调用RejectExecutionHandler.rejectedExecution()方法
工作线程：线程池创建线程时，会将线程封装成工作线程Worker，Worker在执行完任务后，还会循环获取工作队列里的任务来执行。

任务队列： 
1. ArrayBlockingQueue：数组结构，FIFO排序 
2. LinkedBlockingQueue：链表结构 FIFO排序 
3. SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态。 
4. PriorityBlockingQueue：具有优先级的无限阻塞队列 
RejectedExecutionHandler(饱和策略)： 
1. AbortPolicy：直接抛出异常 
2. CallerRunsPolicy：只用调用者所在线程来运行任务 
3. DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务 
4. DiscardPolicy：不处理，丢弃掉

execute()方法用于提交不需要返回值的任务。submit()方法用于提交需要返回值的任务，线程池会返回一个future类型的对象。

### 第10章 Executor框架
ThreadPoolExecutor
FixedThreadPool，创建固定线程数。适用于负载比较重的服务器。
SingleThreadPool 创建单个线程 适用于需要保证顺序地执行各个任务。
CacheThreadPool是大小无界的线程池，适用于执行很多的短期异步任务的小程序
ScheduledThreadPoolExecutor
ScheduledThreadPoolExecutor包含若干线程的ScheduleThreadPoolExecutor
SingleThreadScheduledExecutor只包含一个线程的ScheduledThreadPoolExecutor

### 第11章 Java并发编程实战 