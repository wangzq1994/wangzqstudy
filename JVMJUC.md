# JVMJUC学习

### 进程和线程

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

synchronized是jdk提供的jvm层面的同步机制。 **它解决的是多线程之间访问共享资源的同步问题,它保证了 在被它修饰的方法或代码块同一时间只有一个线程执行。**

java6之前的synchronized属于重量锁,性能较差, 它是基于操作系统的Mutex Lock互斥量实现的。

> 

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

##### 8、execute()和submit()方法

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

**newFixedThreadPool(固定大小)和newSingleThreadPoolExecutor(单线程化的线程池)都是创建固定线程的线程池, 尽管它们的线程数是固定的，但是它们的阻塞队列的长度却是Integer.MAX_VALUE的,所以， 队列的任务很可能过多，导致OOM。**

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

##### ConcurrentLinkedQueue

ConcurrentLinkedQueue : 是一个适用于高并发场景下的队列，通过无锁的方式，实现了高并发状态下的高性能，通常ConcurrentLinkedQueue性能好于BlockingQueue.它是一个基于链接节点的无界线程安全队列。该队列的元素遵循先进先出的原则。头是最先加入的，尾是最近加入的，该队列不允许null元素。

ConcurrentLinkedQueue重要方法:
add 和offer() 都是加入元素的方法(在ConcurrentLinkedQueue中这俩个方法没有任何区别)
poll() 和peek() 都是取头元素节点，区别在于前者会删除元素，后者不会。

##### BlockingQueue

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

 

##### **ArrayBlockingQueue**

ArrayBlockingQueue是一个有边界的阻塞队列，它的内部实现是一个数组。有边界的意思是它的容量是有限的，我们必须在其初始化的时候指定它的容量大小，容量大小一旦指定就不可改变。

ArrayBlockingQueue是以先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部。

 

##### LinkedBlockingQueue

LinkedBlockingQueue阻塞队列大小的配置是可选的，如果我们初始化时指定一个大小，它就是有边界的，如果不指定，它就是无边界的。说是无边界，其实是采用了默认大小为Integer.MAX_VALUE的容量 。它的内部实现是一个链表。和ArrayBlockingQueue一样，LinkedBlockingQueue 也是以先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部。

##### PriorityBlockingQueue

PriorityBlockingQueue是一个没有边界的队列，它的排序规则和 java.util.PriorityQueue一样。需要注 意，PriorityBlockingQueue中允许插入null对象。

所有插入PriorityBlockingQueue的对象必须实现 java.lang.Comparable接口，队列优先级的排序规则就 是按照我们对这个接口的实现来定义的。另外，我们可以从PriorityBlockingQueue获得一个迭代器Iterator，但这个迭代器并不保证按照优先级顺 序进行迭代。

##### SynchronousQueue

SynchronousQueue队列内部仅允许容纳一个元素。当一个线程插入一个元素后会被阻塞，除非这个元素被另一个线程消费。

### JVM

![image-20200613173015963](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200613173015963.png)

JDK8之前:（图中灰色的表示是私有且没有GC垃圾回收）

- 线程私有的部分有:程序计数器(PC寄存器),JAVA虚拟机栈,本地方法栈(native)。
- 线程共享部分有: GC堆,永久代(是方法区的一种实现)。

JDK8之后:

- 线程私有的部分不变, 线程共享部分的永久代改为了元空间(MetaSpace) (永久代和元空间都是方法区的实现),字符串常量池也移动到了heap空间

##### 类加载器

![image-20200613204707861](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200613204707861.png)

![image-20200613204803829](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200613204803829.png)

##### 双亲委派机制

![image-20200613205014068](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200613205014068.png)

**双亲委派机制是为了防止类被重复加载，避免核心API遭到恶意破坏。** 如Object类，它由BootstrapClassLoader在JVM启动时加载。 如果没有双亲委派机制，那么Object类就可以被重写，其带来的后果将无法想象。

**在加载一个类时，是由下自上判断类是否被加载的。如果类没有被加载， 就由上自下尝试加载类。**

##### Native Interface

![image-20200613205308520](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200613205308520.png)

native 标注的方法 是调用第三方的函数库 或者 底层操作系统的 了解就可以

##### PC寄存器(程序计数器)

![image-20200613205519842](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200613205519842.png)

程序计数器的特点

1. 如果当前线程执行的是java方法，那么程序计数器记录的是字节码指令的地址。
2. 如果当前线程执行的native方法，那么程序计数器记录的值为空(undefined)。
3. 程序计数器这部分内存区域是JVM中唯一不会出现OOM错误的区域
4. 程序计数器的生命周期与线程相同,即程序计数器随着线程创建而创建， 随着线程的销毁而销毁。

##### Method Area 方法区

![image-20200613210018492](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200613210018492.png)

##### 永久带(元空间)

![image-20200615211446650](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200615211446650.png)

![image-20200615211813266](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200615211813266.png)

![image-20200615211917053](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200615211917053.png)

**字符串常量池在jdk7已经从永久代转移到了堆内存之中。**

**无论是永久代还是元空间，都有可能发生OOM。**









##### Java栈

栈管运行,堆管存储

![image-20200614213534929](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200614213534929.png)

###### StackOverflowError

当前线程执行或请求的栈的大小超过了Java 虚拟机栈的最大空间(比如递归嵌套调用太深),就可能出现StackOverflowError错误

**Java方法 ====栈帧**    

![image-20200614214014332](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200614214014332.png)

![image-20200614214245928](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200614214245928.png)

![image-20200614214945751](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200614214945751.png)

Java虚拟机栈与程序计数器一样，都是线程私有的部分，生命周期也跟线程一样。

**Java虚拟机栈描述的是Java方法运行时的内存模型，它由一个一个的栈帧组成。**

##### 栈+堆+方法区的交互关系

![image-20200614215328451](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200614215328451.png)

##### Heap 堆

![image-20200614220003405](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200614220003405.png)

**堆是JVM中内存占用最大的一块区域，它是所有线程共享的一块区域。 堆的作用是为对象分配内存并存储和回收它们。 堆是垃圾回收的主要区域，所以堆区也被成为GC堆。**

堆区可以划分为 **新生代(Young Generation),老年代(Old Generation)** 和 永久代(Permanent Generation),但永久代已被元空间代替, **元空间存储的是类的元信息，几乎不可能发生GC。** 

新生代再细分可以分为: **Eden空间，From Survivor空间(from区)和To Survivor空间(TO区)。**

![image-20200614220540962](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200614220540962.png)

![image-20200614220911672](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200614220911672.png)

##### 堆参数调节

![image-20200615212854971](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200615212854971.png)

![image-20200615213025889](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200615213025889.png)

- -Xms

> 设置堆内存初始化大小。

- -Xmx / -XX:MaxHeapSize=?

> 设置堆内存最大值。

![image-20200615213511260](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200615213511260.png)

![image-20200615213622227](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200615213622227.png)

##### OutOfMemoryError

发生OOM的情况:

- java heap space

> 当需要为对象分配内存时，堆空间占用已经达到最大值， 无法继续为对象分配内存，可能会出现OOM: java heap space错误。

- Requested array size exceeds VM limit

> 当为数组分配内存时，数组需要的容量超过了虚拟机的限制范围， 就会抛出OOM: Requested array size exceeds VM limit。 根据我的测试，Integer.MAX_VALUE - 2 是虚拟机能为数组分配的最大容量

- GC overhead limit exceed

> 垃圾回收器花费了很长时间GC,但是GC回收的内存非常少, 就可能抛出OOM:GC overhead limit exceed 错误。
>
> 但是这点在我的机器上测试不出来,可能与jdk版本或gc收集器或Xmx分配内存的大小有关, 一直抛出的是java heap space

- Direct buffer memory

> 当程序分配了超额的本地物理内存(native memory/ direct buffer)， minor gc(young gc)并不会回收这部分内存， 只有 full gc才会回收直接内存，如果不发生full gc， 但直接内存却被使用完了，那么可能会发生 OOM: Direct buffer memory。

- unable to create new native thread

> 操作系统的线程资源是有限的， 如果程序创建的线程资源太多(无需超过平台限制的线程资源上限)， 就可能发生 OOM: unable to create new native thread 错误。

- Metaspace

> 当加载到元空间中的类的信息太多，就有可能导致 OOM : Metaspace。 **使用cglib的库，可以动态生成class，所以可以使用cglib测试此错误。**

##### GC收集日志信息

![image-20200617140507459](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617140507459.png)

![image-20200617141615740](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617141615740.png)

总体概述

![image-20200617142329501](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617142329501.png)

![image-20200617142425572](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617142425572.png)

##### 垃圾回收算法

常见的垃圾回收算法主要有以下4种:

1. 引用计数法(只需要了解 不会用)

   ![image-20200617143130284](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617143130284.png)

   system.gc();执行以后不会立刻执行gc 这是显示的唤醒垃圾回收 但是不会立刻执行

2. 复制算法(Copying)

   ![image-20200617144738779](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617144738779.png)

   ![image-20200617144923836](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617144923836.png)

   ![image-20200617145206601](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617145206601.png)

   ![image-20200617145310410](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617145310410.png)

   ![image-20200617150235565](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617150235565.png)

   ![image-20200617150338473](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617150338473.png)

   ![image-20200617150624254](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617150624254.png)

   

   将堆内存分为2块大小相等的内存空间， 每次只使用其中的一块内存，另一块则空闲。 当其中一块内存使用完后， 就将仍然存活的对象复制到另一块空闲内存空间，再清理已使用的内存。

   **复制算法的优点是不会产生连续的内存碎片，速度也很高效。 但是缺点更明显:每次只使用内存的一半，就代表可使用的内存减少了1/2，代价很高昂。**

   **复制算法一般用于新生代。 因为新生代的GC非常频繁，每次GC的对象较多，存活的对象较少。 所以采用复制算法效率更高，复制时只需要复制少量存活的对象。**

3. 标记清除(Mark-Sweep)

   标记-清除算法分为2个步骤：标记和清除。

   首先标记出所有可达(存活)的对象，在标记完成后， 统一回收所有未被标记(不可达)的对象。

   标记-清除算法的缺点主要有2个:

   1. **标记和清除2个阶段的耗时都比较长，可以总结为效率较低。**
   2. **对象在内存中的分布可能是不连续的，分散的，标记-清除后可能造成不连续的内存碎片。** 当内存碎片过多后，后续想要分配较大的对象时，无法找到足够大的内存碎片， 可能又需要触发GC。

   **标记-清除算法一般用于老年代。** 因为老年代中的对象存活率较高，几乎很少被回收， 所以标记-清除和标记-整理算法GC的时间不会太长， GC的对象相比新生代更少。

   ![image-20200617151622557](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617151622557.png)

   ![image-20200617152456706](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617152456706.png)

   ![image-20200617153937399](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617153937399.png)

4. 标记压缩(Mark-Compact)(标记整理)

   标记-整理算法是对标记-清除算法的一种改进。

   标记-整理算法与标记-清除算法的在标记阶段是相同的， 都是首先标记出所有可达(存活)的对象。 但**标记之后并不直接清理未被标记(不可达)的对象， 而是使被标记(存活)的对象向内存一端移动，然后清理掉这一端外的内存。**

   **标记-整理算法的优点是: 几乎不会如标记-清除算法那样产生不连续的内存碎片。 但，所谓慢工出细活,标记-整理的效率是比标记-清除要低的。**

   **标记-整理算法的缺点是: 效率不高**

   **标记-整理算法和标记-清除算法一样，一般用于老年代。**

   ![image-20200617154653155](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617154653155.png)

   

##### 分代收集算法

垃圾回收用什么样的方式回收(GC的理解)

![image-20200617142129926](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617142129926.png)

**分代收集算法并不是指某一种具体的垃圾收集算法， 而是将复制，标记-清除，标记-整理等算法合理运用到堆区的不同空间。** 比如新生代使用复制算法，老年代使用标记清除或标记整理算法。

现代的几乎所有的JVM都使用分代收集，毕竟每种算法都有优缺点， 结合它们的特点，对不同的环境采用不同的算法是非常明智的选择。

##### 小总结

![image-20200617155458406](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617155458406.png)

![image-20200617155646040](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617155646040.png)

![image-20200617155750217](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617155750217.png)

![image-20200617155837093](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617155837093.png)

#### JMM(java内存模型)

![image-20200617161306167](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617161306167.png)

![image-20200618094946631](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200618094946631.png)

![image-20200617161249827](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617161249827.png)

1. 可见性 2.原子性 3.有序性

#### Volatile

**volatile是java虚拟机提供的一种轻量级同步机制**

1. 保证可见性
2. **不保证原子性**
3. 禁止指令重排（保证了有序性）

高并发下不用synchronized，因为并发性不好；用volatile和juc下面的类；

#### volatile保证内存的可见性

可见性是指一个线程的对于共享数据的修改对其他线程是可见的。 jvm的内存模型是: **线程总是从主内存读取变量到工作内存，然后在工作内存中进行修改， 在修改完变量后，才把变量同步到主内存中。** 如果多个线程同时读取了一个变量到各自的内存中，其中一个线程对变量进行了修改，并同步回了主内存， 但其它线程仍然使用的是原来的旧值，这就造成了数据的不一致。

解决这个问题的办法就是给变量加上volatile关键字修饰。 volatile使得被它修饰的变量在被线程修改后，那么线程就需要把修改后的变量重新同步到主内存， 会通知其他线程重新从主内存读取该变量;

```java
public class VolatileDemo {
 
// int num = 1;
volatile int num = 1;
public void changeNum(){
this.num = 30;
}
 
public static void main(String[] args) {
// 线程 操作 资源类
VolatileDemo vd = new VolatileDemo();
new Thread(()->{
System.out.println("进入线程"+Thread.currentThread().getName()+"，num="+vd.num);
// 休息，等待num被其他线程读取到
try {
TimeUnit.SECONDS.sleep(3);
} catch (InterruptedException e) {
e.printStackTrace();
}
vd.changeNum();
System.out.println("线程"+Thread.currentThread().getName()+"修改num，num="+vd.num);
},"a").start();
 
while (vd.num==1){
// 一直等待循环
}
 
System.out.println(Thread.currentThread().getName()+"线程结束，num="+vd.num);
 
// 加volatile:
//进入线程a，num=1
//线程a修改num，num=30

```

#### volatile不保证原子性

![image-20200618101510344](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200618101510344.png)

```java
public class VolatileDemo {

  // int num = 1;
  volatile int num = 1;
  public void addNum(){
    num++;
  }
  /**
   * 不保证原子性验证：
   * - 20个线程，每个线程1000次，做+1操作
   * - 20个线程全部跑完后，看main函数打印结果
   * 结论：
   * - 每次结果都不一样，小于20001
   *
   * @param args
   */
  public static void main(String[] args) {
    VolatileDemo vd = new VolatileDemo();
    for (int i = 0; i < 20; i++) {
      new Thread(()->{
        for (int j = 0; j < 1000; j++) {
          vd.addNum();
        }
      },"线程"+i).start();
    }
    // 主线程+GC线程
    while (Thread.activeCount()>2){
      // 线程礼让
      Thread.yield();
    }
    System.out.println(Thread.currentThread().getName()+"线程: num="+ vd.num);
  }
}

```

#### volatile不保证原子性理论解释（num++为什么不安全）

num++出现问题底层逻辑（javap -c）：

获得：各线程从主内存读num到工作内存
修改：各线程在各自工作内存做+1操作，工作内存中的操作互相不可见。
写回：线程在写回前被挂起了，写回的时候相互覆盖，造成数值丢失。

![image-20200618101914580](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200618101914580.png)

#### volatile不保证原子性问题怎么解决？

加synchronized，并发性能不好
juc的atomic，比如AtomicInteger的getAndIncrement()

#### AtomicInteger保证原子性代码演示

```java
AtomicInteger num2 = new AtomicInteger(1);
public void addNumByAtomic(){
    num2.getAndIncrement();
     // num2.incrementAndGet();
}
/**
 * volatile不保证原子性解决：
 * - AtomicInteger
 */
public static void main(String[] args) {
  // seeOkByVolatile();
  VolatileDemo vd = new VolatileDemo();
  for (int i = 0; i < 20; i++) {
    new Thread(()->{
      for (int j = 0; j < 1000; j++) {
        // 不安全
        vd.addNum();
        // 安全
        vd.addNumByAtomic();
      }
    },"线程"+i).start();
  }

  // 主线程+GC线程
  while (Thread.activeCount()>2){
    // 线程礼让
    Thread.yield();
  }

  System.out.println(Thread.currentThread().getName()+"线程: num="+ vd.num);
  System.out.println(Thread.currentThread().getName()+"线程: num2="+ vd.num2);
  // main线程: num=13808
  // main线程: num2=20001
}
```

#### volatile禁止指令重排序

1. Java内存模型中，为提高性能，允许编译器和处理器对指令进行重排序。
2. 重排时会考虑到指令间的数据依赖性
3. 不会影响单线程环境下程序执行
4. 多线程环境下由于线程交替执行，由于编译器优化重排的存在，两线程使用的变量能否保证一致性是无法确定的。结果无法预测。

![image-20200618181825699](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200618181825699.png)

####  指令重排造成的不安全举例

```java
int a=0;
boolean flag = false;

public void m1(){
  a=1; // 1
  flag=true; // 2
}

public void m2() {
  if(flag){
    a = a+5; // 3
    System.out.println("revalue="+a);
  }
}
```

解释：

1. 线程1执行m1，线程2执行m2

2. 在不重排时，一定是按照123步骤执行，结果为6

3. 如果发生重排，比如1和2交换了顺序，当m1执行完2时，线程切换，执行m1，这时可以进入if函数，a结果为5

   

#### 禁止指令重拍小总结(了解即可)

![image-20200618184728705](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200618184728705.png)

![image-20200618184905783](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200618184905783.png)

#### 如何保证有序性？

保证有序性：volatile、synchronized、Lock

#### (面试题)你在哪些地方用到了volatile?

1. 单例模式在多线程下不安全
2. 读写锁/手写缓存
3. cas底层源码分析

#### 单例模式在多线程下不安全代码演示

```java
public class SingletonDemo {

  private static SingletonDemo instance = null;

  private SingletonDemo() {
    System.out.println(Thread.currentThread().getName()+"\t调用构造方法");
  }

  public static SingletonDemo getInstance() {
    if(instance == null){
      instance = new SingletonDemo();
    }

    return instance;
  }
  public static void main(String[] args) {
    // 单线程下，单例模式正常。
//    System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
//    System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
//    System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
    //main 调用构造方法
    //true
    //true
    //true


    // 多线程下，单例模式不行
    for (int i = 0; i < 10; i++) {
      new Thread(()->{
        SingletonDemo.getInstance();
      },"线程"+i).start();
    }
    //线程0 调用构造方法
    //线程4 调用构造方法
    //线程3 调用构造方法
    //线程2 调用构造方法
    //线程1 调用构造方法
  }
}
```

#### ! 单例模式在多线程下不安全解决方案？

DCL(double check Lock)双端检索机制+volatile

双端检索机制：在加锁前和加锁后都进行一次判断

demo:

```java
public static SingletonDemo getInstance() {
  if(instance == null){
    synchronized (SingletonDemo.class){
      if(instance == null){
        instance = new SingletonDemo();
      }
    }
  }
  return instance;
}

```

![image-20200618191957982](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200618191957982.png)

![image-20200618192126724](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200618192126724.png)

因此多线程下，当线程a访问instance!=null时，instance实例却未必初始化完成（还没做2）；此时切到线程b，线程b直接取intance实例，这个实例是未完成初始化的实例。因此线程不安全。

###### 如何解决？

```java
private static volatile SingletonDemo instance = null;
// 告诉编译器禁止指令重排
```

