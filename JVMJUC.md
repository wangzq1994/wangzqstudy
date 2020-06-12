# JVMJUC学习

### 多线程

多线程套路

**线程  操作   资源类**

1 高内聚低耦合下，线程操作资源类
2 判断、干活 、通知
3 防止虚假唤醒  判断用while

基础

1.经典卖票问题

```java
class Ticket{
    private int number = 30;
    Lock lock=new ReentrantLock();
    public  void sale(){
        lock.lock();
        try {
            if (number>0){
                System.out.println(Thread.currentThread().getName()+"\t 当前卖出的第						=="+number-- +"还剩==="+number);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
           lock.unlock();
        }
    }
}
```

```java
Ticket ticket=new Ticket();
        new Thread(()->{for (int i = 0; i < 40; i++) ticket.sale();},"AAA").start();
        new Thread(()->{for (int i = 0; i < 40; i++) ticket.sale();},"BBB").start();
        new Thread(()->{for (int i = 0; i < 40; i++) ticket.sale();},"CCC").start();
```

![image-20200611151941159](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200611151941159.png)

2.多个生成者消费者

```java
class resource2 {
    private Integer number=0;
    Lock lock=new ReentrantLock();
    Condition condition = lock.newCondition();
    public void reduce(){
        lock.lock();
        try {
            while (number==0){
                //this.wait();
                condition.await();
            }
        number--;
        System.out.println(Thread.currentThread().getName()+"==新来消费了=="+number);
        //this.notifyAll();
        condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }

    }
    public void increase(){
        lock.lock();
        try {
            while (number!=0){
                //this.wait();
                condition.await();
            }
            number++;
            System.out.println(Thread.currentThread().getName()+"==新来生产了=="+number);
            //this.notifyAll();
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }

    }
}
public static void main(String[] args) {
        resource2 resource=new resource2();
        new Thread(()->{
            for (int i = 0; i < 30; i++) {
                resource.reduce();
            }
        },"AAA").start();
        new Thread(()->{
            for (int i = 0; i < 30; i++) {
                resource.increase();
            }
        },"BBB").start();
        new Thread(()->{
            for (int i = 0; i < 30; i++) {
                resource.reduce();
            }
        },"CCC").start();
        new Thread(()->{
            for (int i = 0; i < 30; i++) {
                resource.increase();
            }
        },"DDD").start();
    }
```

![image-20200611152756480](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200611152756480.png)

3.多个线程顺序调度精确唤醒

```java
//使用多个condition实现
class printdemo{
    private  int numb=1; //A : 1 B : 2 C : 3
    Lock lock=new ReentrantLock();
    Condition c1=lock.newCondition();
    Condition c2=lock.newCondition();
    Condition c3=lock.newCondition();
    public void printtest5(){
        lock.lock();
        try {
            while (numb!=1){
                c1.await();
            }
            for (int i = 1; i <= 1; i++) {
                System.out.println(Thread.currentThread().getName()+"\t 打印了 "+i);
            }
            numb=2;//修改标志位
            c2.signal();//精准唤醒B线程
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
    public void printtest10(){
        lock.lock();
        try {
            while (numb!=2){
                c2.await();
            }
            for (int i = 1; i <= 2; i++) {
                System.out.println(Thread.currentThread().getName()+"\t 打印了 "+i);
            }
            numb=3;//修改标志位
            c3.signal();//精准唤醒B线程
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
    public void printtest15(){
        lock.lock();
        try {
            while (numb!=3){
                c3.await();
            }
            for (int i = 1; i <= 3; i++) {
                System.out.println(Thread.currentThread().getName()+"\t 打印了 "+i);
            }
            numb=1;//修改标志位
            c1.signal();//精准唤醒B线程
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
public static void main(String[] args) {
        printdemo printdemo=new printdemo();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                printdemo.printtest5();
            }
        },"AAA").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                printdemo.printtest10();
            }
        },"BBB").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                printdemo.printtest15();
            }
        },"CCC").start();
    }
```

![image-20200611162412315](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200611162412315.png)

#### 多线程的获得方式

##### 1.前两种

![image-20200611163612170](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200611163612170.png)

##### 2.使用**Callable接口**

现在我们观察里面的run方法，返回的都是void，也就是说这两种方式都不能返回处理后的结构。但是Callable接口的出现可以有效地解决这一问题

我们先来看Thread的构造方法：

![image-20200611163735002](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200611163735002.png)

这个源码我摘自jdk1.8，一共列举了9个构造函数，但是仔细观察就能发现，没有一个构造方法可以传入Callable接口，这也就意味着不能根据之前那简单的方式来创建线程。这时候怎么办呢？那就得换一种思考方式。

既然线程能有返回值，不知道是否可以联想到一个函数式接口Future，我们以此为基点进行查询

![image-20200611163852642](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200611163852642.png)

![image-20200611163906075](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200611163906075.png)

这就是一个最基本的使用方法。当然Future还提供了很多其他的方法：

（1）cancel方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。

参数mayInterruptIfRunning表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。如果任务已经完成，则无论mayInterruptIfRunning为true还是false，此方法肯定返回false，即如果取消已经完成的任务会返回false；

如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，

若mayInterruptIfRunning设置为false，则返回false；

如果任务还没有执行，则无论mayInterruptIfRunning为true还是false，肯定返回true。

（2）isCancelled方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。

（3）isDone方法表示任务是否已经完成，若任务完成，则返回true；

（4）get()方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；

（5）get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

基本上就是这样。其实经常会配合着ExecutorService来使用，现在我们举个例子来看一下：

![image-20200611164348858](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200611164348858.png)

##### Callable和Runnable的区别

相同点：

1、两者都是接口

2、两者都需要调用Thread.start启动线程

不同点：

1、如上面代码所示，callable的核心是call方法，允许返回值，runnable的核心是run方法，没有返回值

2、call方法可以抛出异常，但是run方法不行

3、因为runnable是java1.1就有了，所以他不存在返回值，后期在java1.5进行了优化，就出现了callable，就有了返回值和抛异常

4、callable和runnable都可以应用于executors。而thread类只支持runnable

测试：使用线程池来运行

```java
public static void main(String[] args) throws Exception{
		//1 创建一个线程池
		//调用Executors类的静态方法
		ExecutorService service = Executors.newFixedThreadPool(10);
		//2提交runnable对象
		service.submit(new Runnable() {
			@Override
			public void run() {
			}
		});
		//3 提交callable对象
		service.submit(new Callable<String>() {
			@Override
			public String call() throws Exception {
				return null;
			}
		});
		//4 关闭线程池
		service.shutdown();
	}
```

#### java线程池

##### 1、线程池的优势

1. 降低系统资源消耗，通过重用已存在的线程，降低线程创建和销毁造成的消耗；
2. 提高系统响应速度，当有任务到达时，通过复用已存在的线程，无需等待新线程的创建便能立即执行；
3. 方便线程并发数的管控。因为线程若是无限制的创建，可能会导致内存占用过多而产生OOM，并且会造成cpu过度切换（cpu切换线程是有时间成本的（需要保持当前执行线程的现场，并恢复要执行线程的现场)
4.  提供更强大的功能，延时定时线程池。

##### 2、线程池的主要参数

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}
```

1、corePoolSize（线程池基本大小）：当向线程池提交一个任务时，若线程池已创建的线程数小于corePoolSize，即便此时存在空闲线程，也会通过创建一个新线程来执行该任务，直到已创建的线程数大于或等于corePoolSize时，（除了利用提交新任务来创建和启动线程（按需构造），也可以通过 prestartCoreThread() 或 prestartAllCoreThreads() 方法来提前启动线程池中的基本线程。）

2、maximumPoolSize（线程池最大大小）：线程池所允许的最大线程个数。当队列满了，且已创建的线程数小于maximumPoolSize，则线程池会创建新的线程来执行任务。另外，对于无界队列，可忽略该参数。

3、keepAliveTime（线程存活保持时间）当线程池中线程数大于核心线程数时，线程的空闲时间如果超过线程存活时间，那么这个线程就会被销毁，直到线程池中的线程数小于等于核心线程数。

4、workQueue（任务队列）：用于传输和保存等待执行任务的阻塞队列。

5、threadFactory（线程工厂）：用于创建新线程。threadFactory创建的线程也是采用new Thread()方式，threadFactory创建的线程名都具有统一的风格：pool-m-thread-n（m为线程池的编号，n为线程池内的线程编号）。

5、handler（线程饱和策略）：当线程池和队列都满了，再加入线程会执行此策略。

##### 3、线程池流程

![image-20200612102304471](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200612102304471.png)

##### 4、线程池为什么需要使用（阻塞）队列？

回到了非线程池缺点中的第3点：
 1、因为线程若是无限制的创建，可能会导致内存占用过多而产生OOM，并且会造成cpu过度切换。

​    另外回到了非线程池缺点中的第1点：
 2、创建线程池的消耗较高。

##### 5、线程池为什么要使用阻塞队列而不使用非阻塞队列？

阻塞队列可以保证任务队列中没有任务时阻塞获取任务的线程，使得线程进入wait状态，释放cpu资源。
 当队列中有任务时才唤醒对应线程从队列中取出消息进行执行。
 使得在线程不至于一直占用cpu资源。

（线程执行完任务后通过循环再次从任务队列中取出任务进行执行，代码片段如下
 while (task != null || (task = getTask()) != null) {}）。

不用阻塞队列也是可以的，不过实现起来比较麻烦而已，有好用的为啥不用呢？

##### 6、如何配置线程池

CPU密集型任务
 尽量使用较小的线程池，一般为CPU核心数+1。 因为CPU密集型任务使得CPU使用率很高，若开过多的线程数，会造成CPU过度切换。

IO密集型任务
 可以使用稍大的线程池，一般为2*CPU核心数。 IO密集型任务CPU使用率并不高，因此可以让CPU在等待IO的时候有其他线程去处理别的任务，充分利用CPU时间。

混合型任务
 可以将任务分成IO密集型和CPU密集型任务，然后分别用不同的线程池去处理。 只要分完之后两个任务的执行时间相差不大，那么就会比串行执行来的高效。
 因为如果划分之后两个任务执行时间有数据级的差距，那么拆分没有意义。
 因为先执行完的任务就要等后执行完的任务，最终的时间仍然取决于后执行完的任务，而且还要加上任务拆分与合并的开销，得不偿失。

##### 7、java中提供的线程池

Executors类提供了4种不同的线程池：newCachedThreadPool, newFixedThreadPool, newScheduledThreadPool, newSingleThreadExecutor

![image-20200612104412728](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200612104412728.png)

1、newCachedThreadPool：用来创建一个可以无限扩大的线程池，适用于负载较轻的场景，执行短期异步任务。（可以使得任务快速得到执行，因为任务时间执行短，可以很快结束，也不会造成cpu过度切换）

2、newFixedThreadPool：创建一个固定大小的线程池，因为采用无界的阻塞队列，所以实际线程数量永远不会变化，适用于负载较重的场景，对当前线程数量进行限制。（保证线程数可控，不会造成线程过多，导致系统负载更为严重）

3、newSingleThreadExecutor：创建一个单线程的线程池，适用于需要保证顺序执行各个任务。

4、newScheduledThreadPool：适用于执行延时或者周期性任务。

#### 8、execute()和submit()方法

1、execute()，执行一个任务，没有返回值。

 2、submit()，提交一个线程任务，有返回值。

 submit(Callable<T> task)能获取到它的返回值，通过future.get()获取（阻塞直到任务执行完）。一般使用FutureTask+Callable配合使用（IntentService中有体现）。

submit(Runnable task, T result)能通过传入的载体result间接获得线程的返回值。
 submit(Runnable task)则是没有返回值的，就算获取它的返回值也是null。

Future.get方法会使取结果的线程进入阻塞状态，知道线程执行完成之后，唤醒取结果的线程，然后返回结果。

#### java中创建线程池的方式一般有两种：

- **通过Executors工厂方法创建**
- **通过自定义new `ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue)自定义创建`**

#### 阿里巴巴开发者手册不建议开发者使用Executors创建线程池

**newFixedThreadPool和newSingleThreadPoolExecutor都是创建固定线程的线程池, 尽管它们的线程数是固定的，但是它们的阻塞队列的长度却是Integer.MAX_VALUE的,所以， 队列的任务很可能过多，导致OOM。**

**newCacheThreadPool(可缓存线程池)和newScheduledThreadPool(定长线程池)创建出来的线程池的线程数量却是Integer.MAX_VALUE的， 如果任务数量过多,也很可能发生OOM。**

推荐使用通过自定义new  ThreadPoolExecutor 这种方式创建

### 并发队列阻塞式与非阻塞式的区别

在并发队列上JDK提供了两套实现，一个是以ConcurrentLinkedQueue为代表的高性能队列非阻塞，一个是以BlockingQueue接口为代表的阻塞队列，无论哪种都继承自Queue。

**队列遵循先进先出，后进后出的原则**。

阻塞式队列比非阻塞式队列性好。

### 阻塞式队列与非阻塞队列的区别：

阻塞式队列：

 入列(存)：阻塞式队列，如果存放的队列超出队列的总数，是时候会进行等待(阻塞)。当队列达到总数的时候，入列(生产者)会进行阻塞。这时候只有当消费者消费了队列中的队列之后，生产者才可以继续往队列中存放队列。

 出列(取)：如果获取队列为空的情况下，这时候也会进行等待(阻塞)。这时候队列中没有队列，消费者无法消费队列，只有生产者往对队列中存放队列之后，消费者才可以进行消费。

队列中的队列如果被消费了就会从队列中删除掉。

白话文描述：

阻塞队列与普通队列的区别在于，当队列是空的时，从队列中获取元素的操作将会被阻塞，或者当队列是满时，往队列里添加元素的操作会被阻塞。试图从空的阻塞队列中获取元素的线程将会被阻塞，直到其他的线程往空的队列插入新的元素。同样，试图往已满的阻塞队列中添加新元素的线程同样也会被阻塞，直到其他的线程使队列重新变得空闲起来，如从队列中移除一个或者多个元素，或者完全清空队列.

   1.ArrayDeque, （数组双端队列） 
   2.PriorityQueue, （优先级队列） 
	3.ConcurrentLinkedQueue, （基于链表的并发队列） 
	4.DelayQueue, （延期阻塞队列）（阻塞队列实现了BlockingQueue接口） 
	5.ArrayBlockingQueue, （基于数组的并发阻塞队列） 
	6.LinkedBlockingQueue, （基于链表的FIFO阻塞队列） 
	7.LinkedBlockingDeque, （基于链表的FIFO双端阻塞队列） 
	8.PriorityBlockingQueue, （带优先级的无界阻塞队列） 
	9.SynchronousQueue （并发同步阻塞队列）

#### ConcurrentLinkedQueue

ConcurrentLinkedQueue : 是一个适用于高并发场景下的队列，通过无锁的方式，实现了高并发状态下的高性能，通常ConcurrentLinkedQueue性能好于BlockingQueue.它是一个基于链接节点的无界线程安全队列。该队列的元素遵循先进先出的原则。头是最先加入的，尾是最近加入的，该队列不允许null元素。

ConcurrentLinkedQueue重要方法:
add 和offer() 都是加入元素的方法(在ConcurrentLinkedQueue中这俩个方法没有任何区别)
poll() 和peek() 都是取头元素节点，区别在于前者会删除元素，后者不会。

### BlockingQueue

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：

在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。

阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

BlockingQueue即阻塞队列，从阻塞这个词可以看出，在某些情况下对阻塞队列的访问可能会造成阻塞。被阻塞的情况主要有如下两种：

1. 当队列满了的时候进行入队列操作

2. 当队列空了的时候进行出队列操作

因此，当一个线程试图对一个已经满了的队列进行入队列操作时，它将会被阻塞，除非有另一个线程做了出队列操作；同样，当一个线程试图对一个空队列进行出队列操作时，它将会被阻塞，除非有另一个线程进行了入队列操作。

在Java中，BlockingQueue的接口位于java.util.concurrent 包中(在Java5版本开始提供)，由上面介绍的阻塞队列的特性可知，阻塞队列是线程安全的。

在新增的Concurrent包中，BlockingQueue很好的解决了多线程中，如何高效安全“传输”数据的问题。通过这些高效并且线程安全的队列类，为我们快速搭建高质量的多线程程序带来极大的便利。本文详细介绍了BlockingQueue家庭中的所有成员，包括他们各自的功能以及常见使用场景。

认识BlockingQueue

阻塞队列，顾名思义，首先它是一个队列，而一个队列在数据结构中所起的作用大致如下图所示：

从上图我们可以很清楚看到，通过一个共享的队列，可以使得数据由队列的一端输入，从另外一端输出；

常用的队列主要有以下两种：（当然通过不同的实现方式，还可以延伸出很多不同类型的队列，DelayQueue就是其中的一种）

　　先进先出（FIFO）：先插入的队列的元素也最先出队列，类似于排队的功能。从某种程度上来说这种队列也体现了一种公平性。

　　后进先出（LIFO）：后插入队列的元素最先出队列，这种队列优先处理最近发生的事件。

   多线程环境中，通过队列可以很容易实现数据共享，比如经典的“生产者”和“消费者”模型中，通过队列可以很便利地实现两者之间的数据共享。假设我们有若干生产者线程，另外又有若干个消费者线程。如果生产者线程需要把准备好的数据共享给消费者线程，利用队列的方式来传递数据，就可以很方便地解决他们之间的数据共享问题。但如果生产者和消费者在某个时间段内，万一发生数据处理速度不匹配的情况呢？理想情况下，如果生产者产出数据的速度大于消费者消费的速度，并且当生产出来的数据累积到一定程度的时候，那么生产者必须暂停等待一下（阻塞生产者线程），以便等待消费者线程把累积的数据处理完毕，反之亦然。然而，在concurrent包发布以前，在多线程环境下，我们每个程序员都必须去自己控制这些细节，尤其还要兼顾效率和线程安全，而这会给我们的程序带来不小的复杂度。好在此时，强大的concurrent包横空出世了，而他也给我们带来了强大的BlockingQueue。（在多线程领域：所谓阻塞，在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会自动被唤醒）

 

#### **ArrayBlockingQueue**

ArrayBlockingQueue是一个有边界的阻塞队列，它的内部实现是一个数组。有边界的意思是它的容量是有限的，我们必须在其初始化的时候指定它的容量大小，容量大小一旦指定就不可改变。

ArrayBlockingQueue是以先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部。

 

### LinkedBlockingQueue

LinkedBlockingQueue阻塞队列大小的配置是可选的，如果我们初始化时指定一个大小，它就是有边界的，如果不指定，它就是无边界的。说是无边界，其实是采用了默认大小为Integer.MAX_VALUE的容量 。它的内部实现是一个链表。和ArrayBlockingQueue一样，LinkedBlockingQueue 也是以先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部。

### PriorityBlockingQueue

PriorityBlockingQueue是一个没有边界的队列，它的排序规则和 java.util.PriorityQueue一样。需要注 意，PriorityBlockingQueue中允许插入null对象。

所有插入PriorityBlockingQueue的对象必须实现 java.lang.Comparable接口，队列优先级的排序规则就 是按照我们对这个接口的实现来定义的。另外，我们可以从PriorityBlockingQueue获得一个迭代器Iterator，但这个迭代器并不保证按照优先级顺 序进行迭代。

### SynchronousQueue

SynchronousQueue队列内部仅允许容纳一个元素。当一个线程插入一个元素后会被阻塞，除非这个元素被另一个线程消费。

#### 进程和线程

**进程与线程最主要的区别是它们是操作系统管理资源的不同方式的体现。** 准确来说进程与线程属于衍生关系。 进程是操作系统执行程序的一次过程,在这个过程中可能会产生多个线程。

> 比如在使用QQ时，有窗口线程， 文字发送的线程，语音输入的线程，可能不是很恰当，但是就是这个意思。

**由于系统在线程之间的切换比在进程之间的切换更高效率，所以线程也被成为轻量级进程。**

#### 并发和并行

1. 并发: 多个线程任务被一个cpu轮流执行。注意，这里并不是只允许一个cpu执行多任务，多个cpu执行也是可以的。 **并发强调的是计算机应用程序有处理多个任务的能力。**
2. 并行:多个线程被多个cpu同时执行。这里也并不是只允许多个cpu处理多任务，一个cpu也是可以的， 只要cpu能在同一时刻处理多任务。**并行强调的是计算机应用程序拥有同时处理多任务的能力。**

#### 多线程的利弊

- 利:
  - 线程可以比作轻量级的进程，cpu在线程之间的切换比在进程之间的切换，耗费的资源要少的多。
  - 现在是多核cpu时代，意味着多个线程可以被多个cpu同时运行(并行)，如果可以利用好多线程，那么可以编写出高并发的程序。
- 弊:
  - 虽然线程带来的好处很多，但是并发编程并不容易，如果控制不好线程，那么就可能造成死锁，资源闲置，内存泄露等问题。

#### 什么是上下文切换?

cpu是采用时间片的轮转制度，在多个线程之间来回切换运行的。 当cpu切换到另一个线程的时候，它会先保存当前线程执行的状态， 以便在下次切换回来执行时，可以重新加载状态，继续运行。 **从保存线程的状态再到重新加载回线程的状态的这个过程就叫做上下文切换。**

#### 线程的优先级

在Java中可以通过Thread类的setPriority方法来设置线程的优先级， 虽然可以通过这样的方式来设置线程的优先级，但是线程执行的先后顺序并不依赖与线程的优先级。 换句话说就是，**线程的优先级不保证线程执行的顺序。**

#### 线程的几种状态

见:jdk Thread类源码中的state枚举类

```
  NEW,RUNNABLE,BLOCKED,WAITING,TIMED_WAITING,TERMINATED
```

#### sleep方法和wait方法

1. sleep方法是Thread类的方法；而wait方法是Object类的方法。
2. sleep方法会**使当前线程让出cpu的调度资源**，从而让其他线程有获得被执行的机会， **但是并不会让当前线程释放锁对象。** 而wait方法是**让当前线程释放锁并进入wait状态，** 不参与获取锁的争夺，从而让其他等待资源的线程有机会获取锁， 只有当其他线程调用notify或notifyAll方法是，被wait的线程才能重新与其他线程一起争夺资源。 

#### stop,suspend,resume等方法为什么会被遗弃

- stop: stop方法被弃用很好理解，因为stop方法是强行终止线程的执行， 不管线程的run方法是否执行完，资源是否释放完，它都会终止线程的运行，并释放锁。 显然，这在设计上就不合理。
- suspend和resume: suspend方法用于阻塞一个线程,但并不释放锁， 而resume方法的作用只是为了恢复被suspend的线程。 假设A，B线程都争抢同一把锁，A线程成功的获得了锁， 然后被suspend阻塞了，却并没有释放锁，它需要其他线程来唤醒， 但此时B线程需要获得这把锁才能唤醒A，所以此时就陷入了死锁。

#### interrupt,interrupted,isInterrupted方法区别

- interrupt: 这个方法并不是中断当前线程，而是给当前线程设置一个中断状态。
- isInterrupted: 当线程调用interrupt方法后，线程就有了一个中断状态， 而使用isInterrupted方法就可以检测到线程的中断状态。
- interrupted: 这个方法用于清除interrupt方法设置的中断状态。 如果一个线程之前调用了interrupt方法设置了中断状态， 那么interrupted方法就可以清除这个中断状态。

#### join方法

join方法的作用是让指定线程加入到当前线程中执行。

假如在main方法里面创建一个线程A执行，并调用A的join方法， 那么当前线程就是main，指定的A线程就会在main之前执行， 等A执行完后，才会继续执行main。

```
    public static void main(String[] args) throws Exception
    {

        Thread a = new Thread(()->
        {
            try
            {
                TimeUnit.SECONDS.sleep(1);
                
            }catch (Exception e){}

            System.out.println("thread join");
        });
        a.start();

        //a会在main线程之前执行
        a.join();

        System.out.println("main");
    }
```

**join方法的底层是wait方法，调用A线程(子线程)的join方法实际上是让main线程wait， 等A线程执行完后，才能继续执行后面的代码。**

#### yield方法

yield属于Thread的静态方法， 它的作用是让当前线程让出cpu调度资源。

yield方法其实就和线程的优先级一样，你虽然指定了， 但是最后的结果不由得你说了算， **即使调用了yield方法，最后仍然可能是这个线程先执行， 只不过说别的线程可能先执行的机会稍大一些。**

### 并发

#### synchronized

synchronized是jdk提供的jvm层面的同步机制。 **它解决的是多线程之间访问共享资源的同步问题,它保证了 在被它修饰的方法或代码块同一时间只有一个线程执行。**

java6之前的synchronized属于重量锁,性能较差, 它是基于操作系统的Mutex Lock互斥量实现的。

**因为java线程是映射到操作系统的线程之上的, 所以暂停或唤醒线程都需要Java程序从用户态转换为内核态,这段转换时间消耗较长。**

> java6之后jvm团队对synchronized做出了非常大的优化。