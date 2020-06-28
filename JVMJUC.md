### 进程和线程

**进程与线程最主要的区别是它们是操作系统管理资源的不同方式的体现。** 准确来说进程与线程属于衍生关系。 进程是操作系统执行程序的一次过程,在这个过程中可能会产生多个线程。

> 比如在使用QQ时，有窗口线程， 文字发送的线程，语音输入的线程，可能不是很恰当，但是就是这个意思。

**由于系统在线程之间的切换比在进程之间的切换更高效率，所以线程也被成为轻量级进程。**

#### 并发和并行

并发和并行的区别就很明显了。**它们虽然都说是"多个进程同时运行"，但是它们的"同时"不是一个概念。并行的"同时"是同一时刻可以多个进程在运行(处于running)，并发的"同时"是经过上下文快速切换，使得看上去多个进程同时都在运行的现象，是一种OS欺骗用户的现象**。

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

# 多线程

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

#### 多线程的获得方式(四种)

##### 1.前三种

new Thread()//自定义

![image-20200611163612170](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200611163612170.png)

##### 2.使用**Callable接口**

现在我们观察里面的run方法，返回的都是void，也就是说这两种方式都不能返回处理后的结构。但是Callable接口的出现可以有效地解决这一问题

我们先来看Thread的构造方法：

![image-20200611163735002](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200611163735002.png)

没有一个构造方法可以传入Callable接口，这也就意味着不能根据之前那简单的方式来创建线程。

不知道是否可以联想到一个函数式接口Future他实现了RunableFuture 而他又实现了Runnable接口(面向接口的编程思想)

![image-20200611163852642](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200611163852642.png)

### Callable基本使用（demo）(银行对账经常使用对账要有结果)

```java
class MyThread implements Callable<Integer>{

    @Override
    public Integer call() throws Exception {
        System.out.println(Thread.currentThread().getName()+"\t调用call方法");
        TimeUnit.SECONDS.sleep(1);
        return 1024;
    }
}

/**
 * Callable的基本使用
 * 打印：
 * main    调用
 * A   调用call方法
 * call返回：1024
 */
public class CallableDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //FutureTask(Callable<V> callable)
        //FutureTask<V> implements RunnableFuture<V>
        FutureTask<Integer> futureTask = new FutureTask<>(new MyThread());

        new Thread(futureTask,"A").start();
        //一个futureTask被多个线程执行只会被执行一次 而不会执行多次
        //new Thread(futureTask,"B").start();
        System.out.println(Thread.currentThread().getName()+"\t调用1");

        // 如果没有计算完成，就不能执行下面代码
        while (!futureTask.isDone()){

        }

        // get方法建议放在最后调用：因为
        // get方法会要求返回Callable计算结果，如果没有完成，会导致其他线程阻塞，直到计算出结果返回。
        Integer result = futureTask.get();
        System.out.println("call返回："+result);

        System.out.println(Thread.currentThread().getName()+"\t调用2");

    }
}
```

**1.FutureTask任务多线程并发访问时为啥只会被执行一次？**

```java
 public void run() {
    //如果state==new 说明任务没有被执行或者正在被执行还没有执行到set(result)方法。
    //此时通过CAS操作将runner设置为当前线程，这样如果线程正在执行(此时state仍然为   	new)其他线程进来后CAS设置失败（因为期望值已经不再是null），直接return。这就是为啥FutureTask只会执行一次。
        if (state != NEW ||
            !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                         null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
```

**2.FutureTask 的get方法如何实现阻塞的？**

```java
 public V get() throws InterruptedException, ExecutionException {
        int s = state;
        //首先判断state状态，没有完成就会进入awaitDone方法。
        if (s <= COMPLETING)
            s = awaitDone(false, 0L);
        return report(s);
    }
```



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

### Callable和Runnable的区别

相同点：

1、两者都是接口

2、两者都需要调用Thread.start启动线程

不同点：

1. Callabla有返回值
2. Callable会抛异常
3. Callable是call()方法
4. callable和runnable都可以应用于executors。而thread类只支持runnable

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

# java线程池(底层就是 ThreadPoolExecutor)

##### 1、线程池的优势

1. 降低系统资源消耗，通过重用已存在的线程，降低线程创建和销毁造成的消耗；
2. 提高系统响应速度，当有任务到达时，通过复用已存在的线程，无需等待新线程的创建便能立即执行；
3. 方便线程并发数的管控。因为线程若是无限制的创建，可能会导致内存占用过多而产生OOM，并且会造成cpu过度切换（cpu切换线程是有时间成本的（需要保持当前执行线程的现场，并恢复要执行线程的现场)
4.  提高线程客观理性。统一分配、监控、调优。

##### 2、线程池的主要参数

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}
```

1. corePoolSize: 线程池中的常驻核心线程数。

   任务来了->>>

   - 线程小于corePoolSize  即便存在空闲线程，也会创建一个新线程执行该任务

   - 线程大于corePoolSize 放入任务队列

     注意:也可通过 prestartCoreThread() 或 prestartAllCoreThreads() 方法提前启动线程池中的基本线程

2. maximumPoolSize（线程池最大大小）

   - **当队列满了**，且已创建的线程数小于maximumPoolSize，则线程池会创建新的线程来执行任务

     注意对于无界队列 可忽略该参数

3. keepAliveTime（线程存活保持时间）当线程池中线程数大于核心线程数时，线程的空闲时间如果超过线程存活时间，那么这个线程就会被销毁，直到线程池中的线程数小于等于核心线程数。

4. unit：keepAliveTime单位 (时分秒)

5. workQueue：任务队列，被提交但未被执行的任务。

6. threadFactory：线程工厂，用于创建线程，一般用 **默认的即可** threadFactory创建的线程也是采用new Thread()方式，threadFactory创建的线程名都具有统一的风格：pool-m-thread-n（m为线程池的编号，n为线程池内的线程编号

7. handler（线程饱和策略）：当线程池和队列都满了，再加入线程会执行此策略。

##### 3、线程池流程

![image-20200612102304471](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200612102304471.png)

### 线程池拒绝策略时什么？有哪些？分别是什么意思？

**是什么？**

等待队列满了，池中max线程也到了，这时就调用拒绝策略处理这个问题。
**jdk内置4中拒绝策略：**

RejectedExecutionHandler

AbortPolicy：**默认**。直接抛出RejectedExecutionException异常阻止系统正常运行。
CallerRunsPolicy：“调用者运行”。不抛弃任务也不抛异常，而是将任务回退给调用者，从而降低新任务流量。
DiscardOldestPolicy：丢弃等待最久的任务。然后将当前任务加入队列，尝试再次提交当前任务。
DiscardPolicy：直接丢弃任务。不做处理，不抛异常。如果允许任务丢弃，这是最好的策略。

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

分情况讨论：

1.cpu密集型。
	1.即大量计算，没有阻塞。
	2.一般公式：CPU核数+1 个线程的线程池
2.io密集型
	1.并不是一直在跑计算，很可能有很多阻塞等待。
	2.尽可能多开，公式：cpu核数*2。
	3.公式2：cpu核数/（1-阻塞系数），阻塞系数在0.8~0.9之间。8核就是8/(1-0.9) = 80个线程。

##### 7、java中提供的线程池

! 线程池创建方式有多少种？
Executors.newScheduledThreadPool() // 和调度相关
Executors.newWorkStealingPool() // java8新增，可用处理器作为它的并行级别
**三个常用实现方式：**
Executors.newFixedThreadPool() // 固定个数线程。应用场景：执行长期任务，性能好。
Executors.newSingleThreadExecutor(); // 线程池就一个线程。应用场景：任务需要一个个执行。
Executors.newCachedThreadPool(); // 可扩容的，怎么扩由内部机制决定。应用场景：执行很多短期异步小程序 或 负载较轻的服务器。

Executors类提供了4种不同的线程池：newCachedThreadPool, newFixedThreadPool, newScheduledThreadPool, newSingleThreadExecutor

![image-20200612104412728](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200612104412728.png)

1、newCachedThreadPool：用来创建一个可以无限扩大的线程池，适用于负载较轻的场景，执行短期异步任务。（可以使得任务快速得到执行，因为任务时间执行短，可以很快结束，也不会造成cpu过度切换）

```java
newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

2、newFixedThreadPool：创建一个固定大小的线程池，因为采用无界的阻塞队列，所以实际线程数量永远不会变化，适用于负载较重的场景，对当前线程数量进行限制。（保证线程数可控，不会造成线程过多，导致系统负载更为严重）

```java
newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

3、newSingleThreadExecutor：创建一个单线程的线程池，适用于需要保证顺序执行各个任务。

```java
newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

4、newScheduledThreadPool：适用于执行延时或者周期性任务。

#### 线程池3个常用的创建方式（demo)

```java
 public static void main(String[] args) {

//        ExecutorService threadPool = Executors.newFixedThreadPool(5);
//        ExecutorService threadPool = Executors.newSingleThreadExecutor();
//        ExecutorService threadPool = Executors.newCachedThreadPool();
//        Executors.newScheduledThreadPool()

        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
                2,
                5,
                1,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardPolicy()   // 如果是CallerRunsPolicy，打印 main    办理业务，说明将任务回退给了调用者main线程
        );

        try{
            for (int i = 0; i < 11; i++) {
                // 执行
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"\t办理业务");
                });
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            // 关闭
            threadPool.shutdown();
        }
    }
}
```

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

```java

        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
                2,
                5,
                1,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardPolicy()   // 如果是CallerRunsPolicy，打印 main    办理业务，说明将任务回退给了调用者main线程
        );
```

#### 合理配置线程池参数，如何考虑？

分情况讨论：

1.cpu密集型。
		1.即大量计算，没有阻塞。
		2.一般公式：CPU核数+1 个线程的线程池
2.io密集型
		1.并不是一直在跑计算，很可能有很多阻塞等待。
		2.尽可能多开，公式：cpu核数*2。
		3.公式2：cpu核数/（1-阻塞系数），阻塞系数在0.8~0.9之间。8核就是8/(1-0.9) = 80个线程。

# CAS

提问线路：CAS—> Unsafe—> CAS底层原理 —> 原子引用更新 —> 如何规避ABA问题

## compareAndSet怎么用？

比较并交换（compareAndSet）

```java
/**
 * boolean compareAndSet(int expect, int update)
 * - 如果主内存的值=期待值expect，就将主内存值改为update
 * - 该方法可以检测线程a的操作变量X没有被其他线程修改过
 * - 保证了线程安全
 */
public static void main(String[] args) {
    AtomicInteger atomicInteger = new AtomicInteger(5);
    System.out.println(atomicInteger.compareAndSet(5, 10)+ "\t" + atomicInteger);
    System.out.println(atomicInteger.compareAndSet(5, 20)+ "\t" + atomicInteger);
    //true	10
    //false	10
}
12345678910111213
```

## CAS底层原理简述？

1. Compare-And-Swap。是一条CPU并发原语。（原语：操作系统范畴，依赖硬件，不被中断。）
2. 功能是判断内存某个位置的值是否为预期值（Compare），是就更新（Swap），这个过程是原子的。
3. 功能描述：
   1. 判断内存某个位置的值是否为预期值（Compare），是就更新（Swap），这个过程是原子的。
   2. cas有三个操作数，内存值V，旧预期值A，要更新的值B。仅当预期值A=内存值V时，才将内存值V修改为B，否则什么都不做。
4. 自旋：比较并交换，直到比较成功
5. 底层靠Unsafe类保证原子性。

## getAndIncrement() 源码解析（用了cas保证线程安全）

```java
/**
 * 参数说明：
 * this: atomicInteger对象
 * valueOffset：对象的内存地址
 * unsafe：sun.misc.Unsafe类
 * AtomicInteger中变量value使用volatile修饰，保证内存可见。
 * 结论：底层依赖CAS操作/Unsafe类
 */
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
/**
 * compareAndSwapInt：即CAS
 * while: 如果修改失败，会一直尝试修改，直到成功。
 */
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
1234567891011121314151617181920212223
```

简述：

1. 调用了Unsafe类的getAndAddInt
2. getAndAddInt使用cas一直循环尝试修改主内存

## 对Unsafe的理解?

Unsave类

1. 该类所有方法都是native修饰，直接调用底层资源。sun.misc包中。
2. 可以像C的指针一样直接操作内存。java的CAS操作依赖Unsafe类的方法。

## ! CAS有哪些缺点？

1. 循环时间长，开销大
   1. 如果cas失败，就一直do while尝试。如果长时间不成功，可能给CPU带来很大开销。
2. 只能保证一个共享变量的原子操作
   1. 如果时多个共享变量，cas无法保证原子性，只能加锁，锁住代码段。
3. 引出来ABA问题。

# ABA问题

## 简述ABA问题和解决方案？

简述版：

ABA问题描述: 线程1做CAS操作将A改为B再改为A，而线程2再做CAS时修改成功了，这不符合设计思想。

怎么解决：AtomicStampReference时间戳原子引用

## ABA问题描述？问题出在哪？

ABA问题描述：

- 比如线程1从内存位置V中取出A，此时线程2也取出A。且线程2做了一次cas将值改为了B，然后又做了一次cas将值改回了A。此时线程1做cas发现内存中还是A，则线程1操作成功。这个时候实际上A值已经被其他线程改变过，这与设计思想是不符合的。

这个过程问题出在哪？

- 如果只在乎结果，ABA不介意B的存在，没什么问题
- 如果B的存在会造成影响，需要通过AtomicStampReference，加时间戳解决。

## 原子更新引用是啥？

AtomicStampReference，使用时间戳，解决cas中出现的ABA问题。

## AtomicReference使用代码演示

demo

```java
/**
 * 如果希望原子操作的变量是User,Book,此时需要使用AtomicReference类
 */
public static void main(String[] args) {
    User z3 = new User("z3",18);
    User l4 = new User("l4",19);
    AtomicReference<User> atomicReference = new AtomicReference<>(z3);
    System.out.println(atomicReference.compareAndSet(z3, l4) + "\t" + atomicReference.get().toString());
    System.out.println(atomicReference.compareAndSet(z3, l4) + "\t" + atomicReference.get().toString());
    //truecom.mxx.juc.User@4554617c
    //false  com.mxx.juc.User@4554617c

}
12345678910111213
```

## AtomicReference存在ABA问题代码验证

demo

```java
AtomicReference atomicReference = new AtomicReference<Integer>(100);
/**
 * ABA问题验证：
 * 1--ABA
 * 2--A,C
 * @param args
 */
public static void main(String[] args) {
    ABADemo abaDemo = new ABADemo();

    new Thread(()->{
        abaDemo.atomicReference.compareAndSet(100,101);
        abaDemo.atomicReference.compareAndSet(101,100);
    },"1").start();

    new Thread(()->{
        // 睡1s等线程1执行完ABA
        try {TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) { e.printStackTrace();}
        System.out.println(abaDemo.atomicReference.compareAndSet(100, 2020)+"\t"+abaDemo.atomicReference.get());
        //true  2020

    },"2").start();
}
1234567891011121314151617181920212223
```

## AtomicStampReference解决ABA问题代码验证

解决思路：每次变量更新的时候，把变量的版本号加一，这样只要变量被某一个线程修改过，该变量版本号就会发生递增操作，从而解决了ABA变化

```java
AtomicStampedReference atomicStampedReference = new AtomicStampedReference<Integer>(100,1);

public static void main(String[] args) {
    // ABAProblem();
    ABADemo abaDemo = new ABADemo();

    new Thread(()->{
        // 等线程2读到初始版本号的值
        try {TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) { e.printStackTrace();}
        System.out.println("线程1在ABA前的版本号："+abaDemo.atomicStampedReference.getStamp());
        abaDemo.atomicStampedReference.compareAndSet(100,101,abaDemo.atomicStampedReference.getStamp(),abaDemo.atomicStampedReference.getStamp()+1);
        abaDemo.atomicStampedReference.compareAndSet(101,100,abaDemo.atomicStampedReference.getStamp(),abaDemo.atomicStampedReference.getStamp()+1);
        System.out.println("线程1在ABA后的版本号："+abaDemo.atomicStampedReference.getStamp());
    },"1").start();

    new Thread(()->{
        // 存一下修改前的版本号
        int stamp = abaDemo.atomicStampedReference.getStamp();
        System.out.println("线程2在修改操作前的版本号："+stamp);
        // 睡1s等线程1执行完ABA
        try {TimeUnit.SECONDS.sleep(2);} catch (InterruptedException e) { e.printStackTrace();}
        System.out.println(abaDemo.atomicStampedReference.compareAndSet(100,2020,stamp,abaDemo.atomicStampedReference.getStamp()+1)+ "\t" + abaDemo.atomicStampedReference.getReference());
        //线程2在修改操作前的版本号：1
        //线程1在ABA前的版本号：1
        //线程1在ABA后的版本号：3
        //false 100

        },"2").start();

}
123456789101112131415161718192021222324252627282930
```

# 集合类不安全

## ArrayList线程不安全演示-什么故障？什么原因？怎么解决？

ArrayList底层是一个数组，默认大小10，超过就扩容，扩原值的一半10+5=15

线程不安全，因为add方法没有加锁。

不安全案例：

```java
public class ContainerNotSafeDemo {

    /**
     * 1. 故障
     *      java.util.ConcurrentModificationException   并发修改异常
     *
     * 2. 原因
     *      线程并发修改导致，线程1正在写入，线程2抢占资源，导致数据不一致。
     *
     * 3. 解决
     *      1 new Vector(); add方法加锁，线程安全，并发性差。
     *      2 Collections.synchronizedList(new ArrayList<>()); 包装成安全的，还是加锁，并发性差。
     *      3 new CopyOnWriteArrayList<>();  juc的类，写时复制
     * 4. 优化
     */
    public static void main(String[] args) {
//        List<String> list = new ArrayList<>();
//        List<String> list = new Vector<>();
        List<String> list = Collections.synchronizedList(new ArrayList<>());
//        List<String> list = new CopyOnWriteArrayList<>();

        for (int i = 1; i <= 30; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,4));
                System.out.println(Thread.currentThread().getName()+"\t"+list);
            },String.valueOf(i)).start();
        }
    }
}
1234567891011121314151617181920212223242526272829
```

## CopyOnWriteArrayList原理？它有什么好？

参考：https://blog.csdn.net/linsongbin1/article/details/54581787

add源码：

```java
public boolean add(E e) {
    // 1. 加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        // 2. 拷贝数组
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 3. 新增元素到新数组
        newElements[len] = e;
        // 4. 将array引用指向新数组
        setArray(newElements);
        return true;
    } finally {
        // 解锁
        lock.unlock();
    }
}
12345678910111213141516171819
```

解释写时复制：

1. 写操作时，先从原有的数组中拷贝一份出来，然后在新的数组做写操作，写完之后，再将原来的数组引用指向到新数组。整个add操作都是在**锁的保护下进行的**。
2. 读操作时，如果写完成且引用指向新数组，则读到的是最新数据；否则读到的是原始数组数据。可见**读操作是不加锁的**。

## CopyOnWriteArrayList 缺点&使用场合

1. **消耗内存**。写操作，拷贝数组，消耗内存，数组大的话可能导致gc
2. **不能实时读**。拷贝新增需要时间，读到的可能是旧数据，能保证最终一致性，但不满足实时要求。

因此，适合**读多写少**的场景。

## CopyOnWriteArrayList透露的思想

1. 读写分离，提高并发
2. 最终一致性
3. 通过另辟空间，来解决并发冲突

## 集合类不安全之Set-演示/故障/解决

Set同理

HashSet > Collections.synchronizedSet() > CopyOnWriteArraySet

且CopyOnWriteArraySet底层还是用的CopyOnWriteArrayList

HashSet底层是HashMap, add(key,一个常量)

## 集合类不安全之Map-演示/故障/解决

Map类似

HashMap > Collections.synchronizedMap() > ConcurrentHashMap

## HashMap底层实现原理-jdk7

可参考：

- 2018/10/30/20181030111752438/#10-4-HashMap与ConcurrentHashMap解析
- HashMap源码分析：https://blog.csdn.net/weixin_36910300/article/details/79985197

**map的存储结构是什么？**

1. Hash表（数组+链表）
2. key-value构成一个entry对象
3. 数组容量：默认16，始终保持 2^n（为了存取高效，减少碰撞，数据分配均匀）

**new HashMap<>()底层发生了什么？**

创建了长度=16的 Entry table。

**源码分析**

- capacity : 当前数组容量
- loadFactor：负载因子，默认为 0.75。
- threshold：扩容的阈值，等于 capacity * loadFactor。当元素个数超过这个值就触发扩容。
- MAXIMUM_CAPACITY = 1 << 30：最大的容量为 2 ^ 30

```java
//调用无参构造器
public HashMap() {
    //默认初始容量大小为16,默认的加载因子为0.75f
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}

public HashMap(int initialCapacity, float loadFactor) {
    //初始容量不能小于0
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    //初始容量不能超过MAXIMUM_CAPACITY
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    //加载因子不能小于等于0,或者加载因子不能是非数字   
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    //设置加载因子
    this.loadFactor = loadFactor;
    //设置临界值
    threshold = initialCapacity;
    //伪构造,里面没有代码
    init();
}
12345678910111213141516171819202122232425
```

**map.put(key1,value1)底层发生了什么？**

1. 求key1的hash值，调用hashCode()，该hash值用来计算Entry数组下标，得到存放位置。
2. 如果该位置为空，则添加成功。
3. 如果不为空，意味着该位置存在链表：
   1. 如果hash值与其他key不同，则添加成功。
   2. 如果key1的hash值有key与之相同，则调用equal()，继续比较：
      1. 如果equal返回false，则添加成功。
      2. 如果equal返回true，则使用value1替换旧值（修改操作）

**map.get(key1)底层发生了什么？**

1. 根据key1计算hash，找到对应数组下标。
2. 遍历该位置处的链表，找到key1.equal(key2)为true的entry，返回其value。

**扩容的原理？**

扩容后，数组大小为原来的 2 倍。

## ConcurrentHashMap底层原理-jdk7

关键词：Segment数组/基于分段锁/提高并发

1. 引入一个Segment数组。每个Segment单元都包含一个与HashMap结构差不多hash表
2. 读取过程：
   1. 先取key的hash值，取高sshift位决定属于哪个Segment单元。
   2. 接着就是HashMap那一套
3. Segment继承jucReetrantLock，上锁方便，即分段锁。因此segment[1]锁了，不影响其他Segment单元并发。

## HashMap底层实现原理-jdk8

与jdk7的不同的地方：

1. new HashMap()时，不创建长度为16的数组。

2. 底层使用Node[], 而不是Entry[]

3. 数组结构采用

   数组+链表+红黑树

   1. 触发时机：当某索引位置链表长度>8 且 数组长度>64时，次索引位置的链表改为红黑树
   2. 红黑树的关键性质: 从根到叶子最长的可能路径不多于最短的可能路径的两倍长。结果是这棵树大致上是平衡的。
   3. 目的：加快检索速度

## ! ConcurrentHashMap底层原理-jdk8

参考：https://www.jianshu.com/p/5dbaa6707017

底层结构

- 和 1.8 HashMap 结构类似，也是数组+链表+红黑树的。**取消了Segment 分段锁**

那如何保证线程安全的？

- **CAS + synchronized + volatile 来保证并发安全性**，具体的如下

put方法逻辑

源代码解析：

```java
/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    //1. 计算key的hash值
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        //2. 判断Node[]数组是否初始化，没有则进行初始化操作
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        //3. 通过hash定位数组的索引坐标，是否有Node节点，如果没有则使用CAS进行添加（链表的头节点），添加失败则进入下次循环。
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //4. 检查到内部正在扩容，就帮助它一块扩容。
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        // 如果该坐标Node不为null且没有正在扩容，就加锁，进行链表/红黑树 节点添加操作
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    //5. 当前为链表，在链表中插入新的键值对
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    // 6.当前为红黑树，将新的键值对插入到红黑树中
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            // 7.判断链表长度已经达到临界值8（默认值），当节点超过这个值就需要把链表转换为树结构
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    //8.对当前容量大小进行检查，如果超过了临界值（实际大小*加载因子）就需要扩容 
    addCount(1L, binCount);
    return null;
}
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```

解析：对当前的table进行**无条件自循环直到put成功（cas自旋）**。

1. 如果数组下标没有Node节点，就用**CAS+自旋**添加链表头节点。
2. 如果有Node节点，就加synchronized，添加链表或红黑树节点。

get操作，由于数组被volatile修饰了，因此不用担心数组的可见性问题。

```java
volatile Node<K,V>[] table;
1
```

## Map类的各种对比

- HashMap和ConcurrentHashMap对比
- HashMap和HashTable的对比
- HashTable和ConcurrentHashMap对比

# JVM

![image-20200613173015963](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200613173015963.png)

灰色：线程私有，内存很小（kb），不存在垃圾回收（因为生命周期随线程生死）。

橘色：线程共享，存在垃圾回收，大部分垃圾回收都是收的堆。

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

98%的对象都是临时对象

**方法区和永久代**

​       1.方法区：是JVM的一种规范，存放类信息、常量、静态变量、即时编译器编译后的代码等；
​		2.永久代：是HotSpot的一种具体实现，实际指的就是方法区
**关于永久代：**

​	1.方法区（Method Area）和堆一样，是各个线程共享的内存区域
​	2.虽然JVM规范将方法区描述为堆的一个逻辑部分，但它却还有一个别名叫做Non-Heap(非堆)，目的就是要和堆		分开。
​	3.对于HotSpot虚拟机，很多开发者习惯将方法区称之为“永久代(Parmanent Gen)” 。但严格来说，永久代是方	法区的一个实现
​	4.永久存储区（jdk7有）是一个常驻内存区域，用于存放JDK自身所携带的 Class,Interface 的元数据，被装载进	此区域的数据是不会被垃圾回收器回收掉的，关闭 JVM 才会释放此区域所占用的内存。
​	5.Java8将永久代取消了，变成元空间。

##### **字符串常量池在jdk7已经从永久代转移到了堆内存之中。**

**无论是永久代还是元空间，都有可能发生OOM。**

**String对象与常量池**

```java
/*String s1 = new String(“abc”)做了如下事情：
	1、在常量池中创建了“abc”
	2、在java堆中创建了new String(“abc”)对象，该对象指向常量池的“abc”
	3、在java栈中创建了s1引用，指向java堆中的对象
	如下图
*/
```

![image-20200625163107233](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200625163107233.png)

String#intern()方法

intern方法会先去查询常量池中是否有字符串已经存在，如果存在，则返回常量池中的引用，这一点jdk1.6、jdk1.7一样

区别在于，如果在常量池找不到对应的字符串：

jdk1.6 会再将字符串拷贝到常量池，返回指向常量池的引用。

jdk1.7 则不会再将字符串拷贝到常量池，而只是在常量池中生成一个对原字符串（java堆中）的引用。

简单的说，就是往常量池放的东西变了：原来在常量池中找不到时，复制一个副本放到常量池，1.7后则是将在堆上的地址引用复制到常量池。

```java

public class StrTest {

    public static void main(String[] args) {
       /*
        String s1 = "abc";
        String s2 = "abc";
        System.out.println(s1 == s2);   //true，均指向常量池中对象
        */

       /*
        String s1 = new String("abc");

        String s2 = new String("abc");
        System.out.println(s1 == s2);   //false，两个引用指向堆中的不同对象
        */

       /*
        String s1 = "abc";
        String s2 = "a";
        String s3 = "bc";
        String s4 = s2 + s3;
        String s5 = s2 + s3;
        System.out.println(s1 == s4);   //false，因为s2+s3实际上是使用StringBuilder.append来完成，会生成不同的对象
        System.out.println(s4 == s5);   //false，StringBuilder.append在堆中生成不同的对象
        */

       /*
        String s1 = "abc";
        final String s2 = "a";
        final String s3 = "bc";
        String s4 = s2 + s3;
        System.out.println(s1 == s4);   //true，因为final变量在编译后会直接替换成对应的值，所以实际上等于s4=”a”+”bc”，而这种情况下，编译器会直接合并为s4=”abc”，所以最终s1==s4
        */

       /*
        String s = new String("abc");
        String s1 = "abc";
        String s2 = new String("abc");
        System.out.println(s == s1.intern());   //false，"abc"在常量池中已存在，s1.intern()指向常量池的字符串。而s指向堆中的字符串对象
        System.out.println(s == s2.intern());   //false，同理
        System.out.println(s1 == s2.intern());  //true，
        */    
		
        /*
        String str1 = new StringBuilder("a").append("bc").toString();   //会在常量池生成"a"、"bc"，在堆区生成"abc"对象，返回给str1
        System.out.println(str1.intern() == str1);//true,str1.intern()在常量池中没找到"abc",jdk1.7之后会在常量池生成一个引用指向堆区的"abc"对象

        String str2 = new StringBuilder("ja").append("va").toString();//在堆区生成"java"对象
        String str3 = new StringBuilder("ja").append("va").toString();
        System.out.println(str2 == str3);//false
        System.out.println(str2.intern() == str2);//false。str2.intern()在常量池中找到了"java"，则str2.intern()指向的是常量池的"java"
        //常量池中为什么会存在"java"这样的字符串?
        //类加载等操作使"java"字符串加入到了常量池中
        */
    }
}
```



##### Java栈

栈管运行,堆管存储

![image-20200614213534929](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200614213534929.png)

##### StackOverflowError

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

##### MinorGC的过程（复制->清空->互换）

![image-20200614220911672](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200614220911672.png)

##### 堆内存调优

- Xms: 设置初始分配大小，默认为物理内存的1/64
- Xmx: 最大分配内存，默认为物理内存的1/4
- 生产环境下Xms和Xmx的设置需要相同，理由：避免内存忽高忽低，不稳定。

IDEA下的配置

```java
-Xms1024m -Xmx1024m -XX:+PrintGCDetails
```

![image-20200615213622227](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200615213622227.png)

```java
public static void main(String[] args) {

        // 查看cpu核数
        System.out.println(Runtime.getRuntime().availableProcessors());

        // JVM试图使用的最大内存量
        long maxMemory = Runtime.getRuntime().maxMemory();

        // JVM中的内存总量
        long totalMemory = Runtime.getRuntime().totalMemory();

        System.out.println("Xmx: MAX_MEMORY = " + maxMemory + "（字节）、" + (maxMemory / (double)1024 / 1024) + "MB");
        System.out.println("Xms: TOTAL_MEMORY = " + totalMemory + "（字节）、" + (totalMemory / (double)1024 / 1024) + "MB");

    }
}
```



# GC垃圾回收

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

# GC ROOTs

**什么是垃圾**？**如何判断对象是否需要被回收**？

**什么是垃圾？**

- 不再被引用的对象

如何判断对象是否需要被回收？

- 引用计数法：缺点是很难解决循环互相引用
- 可达性分析

**可达性分析，哪些可作为GC ROOTs对象？**

可达性分析

​	1.如果一个对象到GC ROOTs没有引用链，说明次对象不可用
​	2.给定一个集合的引用作为根，遍历对象图，能被遍历到的判为存活，否则死亡。
**哪些可作为GC ROOTs对象？**

​	1.虚拟机栈中的引用的对象（栈帧中的局部变量区）
​	2.方法区中类的静态属性引用的对象
​	3.方法区中常量的引用对象
​	4.本地方法栈(Native方法)中的引用对象

## JVM调优和参数配置

### JVM有哪些参数类型？XX参数？

JVM参数类型

- 标配参数
  - -version
  - -help
  - java -showversion
- x参数
  - -Xint  
  - -Xcomp
  - -Xmixed
- xx参数（重要）

xx参数

- Boolean类型：
  - 公式：`XX:+某个属性` 或者`XX:-某个属性` ，+是开启，-是关闭
  - 是否打印GC收集细节：`-XX:-PrintGCDetails`(默认未开启)
- kv设置类型：
  - 公式:`-XX:属性=属性值`
  - `XX:MetaspaceSize=128m`

- jinfo：查看当前运行程序的配置
  - 公式：`jinfo -flag 配置项 进程编号`
  - `jinfo -flag PrintGCDetails 5988`
  - 查看当前程序**所有的配置参数**：`jinfo -flags 5988`(**注意 这个flags 加了S** )

- -Xms等价于`-XX:InitialHeapSize`，-Xmx等价于`-XX:MaxHeapSize`

### 查看JVM默认值

查看初试默认值

- `-XX:PrintFlagsInitial`

查看修改更新后的值

- `-XX:PrintFlagsFinal`
- 冒号代表被修改过

查看常用参数

- `-XX:PrintCommandLineFlags`
- 能看到当前使用的垃圾回收器(重要)

运行时更改参数

- java -XX:+PrintFlagsFinal  -XX:MaxHeapSize=215m  T (这个是java 文件)

### 项目中常用的JVM配置参数

- `-Xms`
  - 堆初始大小，默认物理内存1/64
- `-Xmx`
  - 堆最大分配，默认物理内存1/4

- `-Xss`
  - 设置单个线程栈的大小，默认512k~1024，等价`XX:ThreadStackSize`，默认=0表示使用初始默认值，不是空间为0
- `-Xmn`
  - 设置新生区大小，一般不调，默认堆空间1/3

- `-XX:Metaspace`
  - 元空间使用本地内存，不在虚拟机中，但默认大小只有20多M
- `-XX:+PrintGCDetails`
  - 日志分析  (GC收集日志信息那节详细有讲)
- `-XX:SurvivorRatio`
  - 幸存区占比，eden：s0：s1默认8：1：1
- -XX:SurvivorRatio=4  eden：s0：s1 4：1：1(主要设置是eden的比列)
  
- `-XX:NewRatio`
  - 新生区占比，默认值为`-XX:NewRatio=2`，表示新生代：老年代为1：2，默认占整个堆 1/3
- `-XX:MaxTenuringTreshold`
  - 设置垃圾最大年龄，默认15。在新生区幸存15次就被转移到养老区。这个值调大，就让对象留在新生区更久，则让对象最大可能在新生代就被回收。

![image-20200626090156486](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200626090156486.png)

**面试问常用参数  这些就足矣了**

### 强/软/弱/虚 四大引用

### 整体架构？四大引用的区别？

java.lang.ref.*

![image-20200626184616909](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200626184616909.png)

- 强引用（默认支持模式）：就算OOM也不会回收。只要对象有一个引用，都不收。
- 软引用：内存足够时不收，内存不足时回收。

- 弱引用：不管内存够不够用，只要GC了都要收。
- 虚引用：就跟没有引用一样，GC来了就被回收，多了一个finalize()通知
  - 不能单独使用，**也无法使用get()访问对象**(返回NUll)，必须和ReferenceQueue联合使用
  - 其意义在说明对象进入了finalization阶段，可以被GC。GC时改对象会收到一个通知，覆写finalize()方法在GC前进行一些清理工作。

### 软引用/弱引用适用场景

一个场景需要读取大量本地图片：

- 每次从硬盘读，性能下降
- 全部加载到内存，OOM

设计思路：HashMap<图片路径，图片对象软引用>，内存不足时，会自动回收图片对象，避免OOM。

### WeakHashMap

- 如果key的引用被置空，GC时entry会被回收
- 减少OOM的概率

### 引用队列ReferenceQueue

- 回收前，弱/软/虚引用 先放ReferenceQueue保存一下

```java
WeakReference<Object> wrf = new WeakReference<Object>(o1,referenceQueue)
```

应用场景？

- 回收前对象被放入引用队列里，可做一些销毁前的操作。

### 引用和GC ROOTs总结

![image-20200626191737356](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200626191737356.png)

# OOM

### StackOverflowError

举例：深度递归，没有出口。栈空间默认大小 512k --1024K

是错误不是异常

![image-20200627071211596](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627071211596.png)

### OutOfMemoryError：Java heap space

举例：while死循环里面new对象

### OutOfMemoryError：GC overhead limit exceeded

![image-20200627072136500](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627072136500.png)

发生场合：大量资源用于GC，却得不到好的回收效果

定义:  超过98%的时间都用来做GC并且回收了不到2%的堆内存 连续多吃 就会抛出

### (重要)OutOfMemoryError：Direct buffer memory

导致原因：直接把本地内存撑爆了。NIO编程，直接分配本地内存不会触发GC。

![image-20200627072544710](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627072544710.png)

vm这个类可以查看MaxDirectMemory 的大小

### (重要)OutOfMemoryError：unable to create new native thread

高并发场景下出现，因为一个系统创建的的最大线程数是有上限的。

导致原因

![image-20200627074429969](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627074429969.png)

服务器级别调参：

上限调整

![image-20200627080008347](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627080008347.png)

修改

![image-20200627080106548](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627080106548.png)



### OutOfMemoryError：Metaspace

封装类信息，常量池。

cglib，反射，动态生成字节码，撑爆元空间

![image-20200627080509239](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627080509239.png)

# 垃圾回收器

### 垃圾回收算法和垃圾回收器的关系

算法是方法论，回收器是落地实现

没有万能的收集器，只有最适合的收集器，进行分代收集

### 4种主要垃圾收集器

Serial(串行)、Parallel(多垃圾并行)、CMS(用户垃圾并行)、G1

- 串行垃圾回收器Serial：为单线程设计，只使用一个线程进行垃圾回收，会暂停所有用户线程。所以**不适合服务器环境**
- 并行垃圾回收器Parallel：多个垃圾收集器线程并行工作，搜集时用户线程也是暂停的（因为是并行搜集，所以暂停时间会比串行短），适用于科学计算/大数据 等**弱交互场景**。
- 并发垃圾回收器CMS：并发标记清除，用户线程和垃圾收集线程同时执行（也可以交替），不需要停顿用户线程，互联网公司多用，适合**对响应时间有要求的场景**

对比

![image-20200627113313487](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627113313487.png)

Java8主要用这三种

- G1：Java9的默认垃圾回收器，将堆内存分割为不同区域然后并发进行回收。

### 查看默认垃圾收集器

```
-XX:PrintCommandLineFlags
```

**Java8默认ParallelGC**

jdk有哪些默认垃圾收集器？

![image-20200627114258681](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627114258681.png)

![image-20200627114324671](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627114324671.png)

SerialOldGC(已经被废弃不用了)

一共7种

如何配置？

```java
-XX:+UseSerialGC
```

# 7大垃圾收集器概述

![image-20200627114936294](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627114936294.png)

下图表示新生代和老年代垃圾回收器的配合使用。

**例如 新生代 使用serial  老年的的 serial Old 会自动被激活**

![image-20200627115503276](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627115503276.png)

## 约定参数说明

- DefNew : Default New Generation，默认新生代使用的垃圾回收器
- Tenured : Old

- ParNew : Parallel New Generation
- PSYoungGen : Parallel Scavenge
- ParOldGen : Parallel Old Generation

JVM的Server/Client模式是什么？

- 基本都是用Server模式的

## Serial (Serial Copying)(串行)

**一句话: 一个单线程的收集器 ,在进行垃圾收集时候,必须暂停其他的工作线程直到他收集结束**

- 最古老，简单高效，单核下垃圾搜集效率高。
- client模式下默认的收集器
- -XX:+UseSerialGC，开启后Serial(Young区)+Serial Old(Old区)组合使用。

- 新生代使用复制算法，老年代使用标记整理算法
- 1:1

## ParNew(Parallel New Generation)(并行)收集器

**Serial收集器的多线程版的版本**

- 也会暂停、是Serial的并行版
- server模式下**新生代的默认垃圾回收**

- -XX:+UseParNewGC，启用ParNew，**老年代不变**，还是Serial Old，**这种组合法已经不推荐**。常见场景是配合老年代 CMS GC使用
- 新生代使用复制算法，老年代使用标记整理算法
- N:1

**修改垃圾线程的线程数**

-XX:ParallelGCThreads    限制线程数量,默认开启和CPU数目相同的线程数

## Parallel (Parallel Scavenge)

使用复制算法,也是一个并行的多线程的垃圾收集器,俗称 吞吐量优先的收集器

- **Java8默认配置**
- 使用复制算法
- 新生代和老年代都用并行的
- N:N

**与ParNew相比，为什么引入Parallel？**

![image-20200627223236214](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627223236214.png)

![image-20200627223328679](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627223328679.png)

- 吞吐量、自适应调节

## Parallel Old

- 目的是为了在老年代同样提供吞吐量优先

配置

![image-20200627223555331](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627223555331.png)

## CMS

- 并发标记清除  一种以获得最短回收停顿时间为目标的收集器
- 目的是获得最短回收时间，适合b/s的网站，重视响应速度。是G1出现之前大型应用首选。
- 并发搜集停顿低，并发指与用户线程一起执行

开启组合

![image-20200627224305203](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627224305203.png)

四步过程：

- 初始标记：标记GC ROOTs的关联对象，速度很快，会暂停工作线程
- 并发标记：进行GC ROOTs的跟踪过程，和用户线程一起工作，不需要暂停工作线程
- 重新标记：修正标记记录（修正期间因用户继续运行导致标记变动的小部分对象的标记记录），会暂停。

- 并发清除：和用户线程一起。清除GC不可达对象


将工作细分，将耗时最长的**并发标记**和**并发清除**两个过程与用户线程一起工作，减少停顿。

优缺点：

- 优：并发搜集停顿低
- 缺：
  - 并发对CPU压力大(GC和工作一起干)；（必须在老年代堆耗尽前GC，否则失败会启动串行GC，造成大停顿）
  - 导致碎片（因为没有整理）

## Serial Old(了解)

- 串行的老年代，单线程，使用标记-整理算法
- client模式默认，CMS后备方案
- java8实际中已经弃用了

# 如何选择垃圾收集器？

![image-20200627230037420](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627230037420.png)

![image-20200627230241525](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200627230241525.png)

## G1

之前收集器的特点

- 年轻代和老年代各自独立，且连续内存
- 年轻代使用复制算法
- 老年代收集必须扫描整个老年代
- 设计原则：尽可能少而快速的执行G

**G1收集器**

- `-XX:UseG1GC`
- 面向服务器的，用于多处理器和大内存环境
- 暂停更小，吞吐更高

- 与CMS一样，与用户线程并发。目的是取代CMS
  - 有整理，无碎片
  - Stop The World可控，在停顿上加了预测机制，用户可指定期望停顿时间
- 主要改变是
  - 新生区（eden、survivor）、老年区不再连续，而是变成一个个resgion(区块)。每个region 1-32M不等

G1特点

![image-20200628091404367](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628091404367.png)

底层原理

- Region区域化垃圾搜集器，最大好处是化整为0，避免全内存扫描，只需按区域扫描
- 不要求对象存储在物理上连续，只需要逻辑上连续
- 每个区域不会固定为某个代服务，可以切换
- 可参数指定区域大小，默认将整个堆划分为2048个分区
- 最大支持32M*2048=64G

**区域化垃圾收集器**

![image-20200628092415896](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628092415896.png)

![image-20200628092442792](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628092442792.png)

![image-20200628092501719](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628092501719.png)

**回收步骤**

YoungGC: 小区域搜集+形成连续内存块

![image-20200628094701880](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628094701880.png)

![image-20200628094712403](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628094712403.png)

![image-20200628094733665](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628094733665.png)

![image-20200628095330868](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628095330868.png)

常用参数了解

![image-20200628095937023](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628095937023.png)



## G1和CMS相比的优势

1、无内存碎片

2、精确控制停顿（用于设期望值、作为软目标，JVM尽可能停顿时间小于这个）

# JMM(java内存模型)

![image-20200617161306167](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617161306167.png)

![image-20200618094946631](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200618094946631.png)

![image-20200617161249827](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200617161249827.png)

1. 可见性 2.原子性 3.有序性

# Volatile

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

#### 如何解决？

```java
private static volatile SingletonDemo instance = null;
// 告诉编译器禁止指令重排
```

# CAS

CAS是什么:  比较并交换（compareAndSet）

![image-20200619102525572](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200619102525572.png)

提问线路：CAS—> Unsafe—> CAS底层原理 —> 原子引用更新 —> 如何规避ABA问题

```java
/**
 * boolean compareAndSet(int expect, int update)
 * - 如果主内存的值=期待值expect，就将主内存值改为update
 * - 该方法可以检测线程a的操作变量X没有被其他线程修改过
 * - 保证了线程安全
 */
public static void main(String[] args) {
    AtomicInteger atomicInteger = new AtomicInteger(5);
    System.out.println(atomicInteger.compareAndSet(5, 10)+ "\t" + atomicInteger);
    System.out.println(atomicInteger.compareAndSet(5, 20)+ "\t" + atomicInteger);
    //true	10
    //false	10
}
```

#### CAS底层原理简述？

一  首先是 Unsafe 类

1. 该类所有方法都是native修饰，直接调用底层资源。sun.misc包中。
2. 可以像C的指针一样直接操作内存。java的CAS操作依赖Unsafe类的方法。

二 自旋锁

1. 自旋：比较并交换，直到比较成功

CAS靠的是底层的Unsafe 类保证原子性

![image-20200619102220908](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200619102220908.png)

#### atomicInteger.getAndIncrement() 详解

![image-20200619102943756](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200619102943756.png)

简述：

1. **调用了Unsafe类的getAndAddInt**
2. getAndAddInt使用cas一直循环尝试修改主内存

#### 总结

![image-20200619103156133](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200619103156133.png)

#### CAS有哪些缺点？

1. 循环时间长，开销大
   1. 如果cas失败，就一直do while尝试。如果长时间不成功，可能给CPU带来很大开销。
2. 只能保证一个共享变量的原子操作
   1. 如果时多个共享变量，cas无法保证原子性，只能加锁，锁住代码段。
3. 引出来ABA问题。

#### 原子类AtomicInteger的ABA问题谈谈?原子更新引用知道吗?

#### ABA问题

ABA问题描述: 比如从内存位置V中取出A，此时线程2也取出A。且线程2做了一次cas将值改为了B，然后又做了一次cas将值改回了A。此时线程1做cas发现内存中还是A，则线程1操作成功。这个时候实际上A值已经被其他线程改变过，这与设计思想是不符合的

#### 这个过程问题出在哪？

- 如果只在乎结果，ABA不介意B的存在，没什么问题
- 如果B的存在会造成影响，需要通过AtomicStampReference，加时间戳解决。

#### 原子更新引用是啥？

**AtomicStampReference**，使用带时间戳的原子类，解决cas中出现的ABA问题。

#### AtomicReference使用代码演示(原子类包装)

我们有atomicInteger atomicDouble  atomicboolearn  可不可有我们字的atomic***

是可以有的  AtomicReference 这类有泛型丢进去什么 就是atomic***

```java
public static void main(String[] args) {
    User z3 = new User("z3",18);
    User l4 = new User("l4",19);
    AtomicReference<User> atomicReference = new AtomicReference<>(z3);
    System.out.println(atomicReference.compareAndSet(z3, l4) + "\t" + atomicReference.get().toString());//ture 交换成功
    System.out.println(atomicReference.compareAndSet(z3, l4) + "\t" + atomicReference.get().toString());//false 交换失败
}
```

#### AtomicReference存在ABA问题代码验证

```java
public static void main(String[] args) {
    ABADemo abaDemo = new ABADemo();

    new Thread(()->{
        abaDemo.atomicReference.compareAndSet(100,101);
        abaDemo.atomicReference.compareAndSet(101,100);
    },"1").start();

    new Thread(()->{
        // 睡1s等线程1执行完ABA
        try {TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) { e.printStackTrace();}
        System.out.println(abaDemo.atomicReference.compareAndSet(100, 2020)+"\t"+abaDemo.atomicReference.get());
        //true  2020

    },"2").start();
}
```

#### AtomicStampReference解决ABA问题代码验证

解决思路：每次变量更新的时候，把变量的版本号加一，这样只要变量被某一个线程修改过，该变量版本号就会发生递增操作，从而解决了ABA变化

```java
AtomicStampedReference atomicStampedReference = new AtomicStampedReference<Integer>(100,1);

public static void main(String[] args) {
    // ABAProblem();
    ABADemo abaDemo = new ABADemo();

    new Thread(()->{
        // 等线程2读到初始版本号的值
        try {TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) { e.printStackTrace();}
        System.out.println("线程1在ABA前的版本号："+abaDemo.atomicStampedReference.getStamp());
        abaDemo.atomicStampedReference.compareAndSet(100,101,abaDemo.atomicStampedReference.getStamp(),abaDemo.atomicStampedReference.getStamp()+1);
        abaDemo.atomicStampedReference.compareAndSet(101,100,abaDemo.atomicStampedReference.getStamp(),abaDemo.atomicStampedReference.getStamp()+1);
        System.out.println("线程1在ABA后的版本号："+abaDemo.atomicStampedReference.getStamp());
    },"1").start();

    new Thread(()->{
        // 存一下修改前的版本号
        int stamp = abaDemo.atomicStampedReference.getStamp();
        System.out.println("线程2在修改操作前的版本号："+stamp);
        // 睡1s等线程1执行完ABA
        try {TimeUnit.SECONDS.sleep(2);} catch (InterruptedException e) { e.printStackTrace();}
        System.out.println(abaDemo.atomicStampedReference.compareAndSet(100,2020,stamp,abaDemo.atomicStampedReference.getStamp()+1)+ "\t" + abaDemo.atomicStampedReference.getReference());
        //线程2在修改操作前的版本号：1
        //线程1在ABA前的版本号：1
        //线程1在ABA后的版本号：3
        //false 100

        },"2").start();
}
```

# 我们知道AyyayList是线程不安全的 编码写出一个不安全的案例

### ArrayList的特点

- ArrayList底层使用Object数组实现。
- ArrayList的容量默认为0,只有在第一次执行add操作时才会初始化容量为10，正常的扩容是为原来的1/2倍。
- 由于ArrayList采用数组实现,它的容量是固定的,所以当添加新元素的时候,如果超出了数组的容量, 那么此时add操作的时间复杂度将会是O(n-1)。
- ArrayList实现了RandomAccess接口，该接口没有具体的规范，只是一个标记， 这代表ArrayList支持快速的随机访问。
- ArrayList在内存空间利用率上肯定是不如LinkedList的， 因为数组是一片固定的连续的内存空间，一旦分配就无法改变， 所以难免会有空间不足或空间使用率很低的情况

### ConcurrentModificationException可以从名字看出是并发修改的异常。

但我要说的是**这个异并不是在修改的时候会抛出的，而是在调用迭代器遍历集合的时候才会抛出。**

而集合类的大部分toString方法，都是使用迭代器遍历的。**所以如果多线程修改集合后， 接着就遍历集合，那么很有可能会抛出ConcurrentModificationException。**

**在ArrayList，HashMap等非线程安全的集合内部都有一个modCount变量， 这个变量是在集合被修改时(删除，新增)，都会被修改。**

如果是多线程对同一个集合做出修改操作，就可能会造成modCount与实际的操作次数不符， 那么最终在调用集合的迭代方法时，modCount与预期expectedModeCount比较， expectedModCount是在迭代器初始化时使用modCount赋值的， **如果发现modCount与expectedModeCount不一致，就说明在使用迭代器遍历集合期间， 有其他线程对集合进行了修改,所以就会抛出ConcurrentModificationException异常。**

### 线程安全的 List

1. 使用集合工具类Collections的 synchronizedList把普通的List转为线程安全的List.(不推荐)
2. 使用Vector.(不推荐)
3. 使用CopyOnWriteArrayList,推荐使用此种方法，**因为以上2种全部都是单纯的Synchronized加锁**.

### CopyOnWriteArrayList(读写分离的思想)

![image-20200620115311767](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200620115311767.png)

```java
public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

**但是CopyOnWriteArrayList不容忽视的缺点就是修改操作比较消耗内存空间，所以它适用于读多写少的环境。**

解释写时复制：

写操作时，先从原有的数组中拷贝一份出来，然后在新的数组做写操作，写完之后，再将原来的数组引用指向到新数组。整个add操作都是在锁的保护下进行的。
读操作时，如果写完成且引用指向新数组，则读到的是最新数据；否则读到的是原始数组数据。可见读操作是不加锁的。

### CopyOnWriteArrayList 缺点&使用场合

消耗内存。写操作，拷贝数组，消耗内存，数组大的话可能导致gc
不能实时读。拷贝新增需要时间，读到的可能是旧数据，能保证最终一致性，但不满足实时要求。
因此，适合读多写少的场景。

### CopyOnWriteArrayList透露的思想

读写分离，提高并发
最终一致性
通过另辟空间，来解决并发冲突

### 集合类不安全之Set-演示/故障/解决

Set同理

HashSet > Collections.synchronizedSet() > CopyOnWriteArraySet

且CopyOnWriteArraySet底层还是用的CopyOnWriteArrayList

```java
 public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
```

HashSet底层是HashMap, add(key,一个常量)

```java
ublic HashSet() {
        map = new HashMap<>();
    }

 // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
 public boolean add(E e) {
        return map.put(e, PRESENT)==null;//put时value 时一个常量
    }
```



### 集合类不安全之Map-演示/故障/解决

Map类似

HashMap > Collections.synchronizedMap() > **ConcurrentHashMap**(这个名字不一样)

### HashMap底层实现原理-jdk7

### HashMap的特点

https://www.cnblogs.com/ylspace/p/12726388.html

- HashMap在Jdk8之前使用拉链法实现,jdk8之后使用拉链法+红黑树实现。
- HashMap是线程不安全的,并允许null key 和 null value。**
- HashMap在我当前的jdk版本(11)的默认容量为0,在第一次添加元素的时候才初始化容量为 16, 之后才扩容为原来的2倍。
- HashMap的扩容是根据 threshold决定的 : threshold = loadFactor * capacity。 当 size 大于 threshold 时,扩容。
- **当每个桶的元素数量达到默认的阈值TREEIFY_THRESHOLD(8)时，HashMap会判断当前数组的 长度是否大于MIN_TREEIFY_CAPACITY(64),如果大于，那么这个桶的链表将会转为红黑树，否则HashMap将会扩容。 当红黑树节点的数量小于等于默认的阈值UNTREEIFY_THRESHOLD(6)时，那么在扩容的时候，这个桶的红黑树将转为链表。**

### **map的存储结构是什么？**

1. Hash表（数组+链表）
2. key-value构成一个entry对象
3. 数组容量：默认16，始终保持 2^n（为了存取高效，减少碰撞，数据分配均匀）

**new HashMap<>()底层发生了什么？**

创建了长度=16的 Entry table。

### 源码分析

capacity : 当前数组容量
loadFactor：负载因子，默认为 0.75。
threshold：扩容的阈值，等于 capacity * loadFactor。当元素个数超过这个值就触发扩容。
MAXIMUM_CAPACITY = 1 << 30：最大的容量为 2 ^ 30

```java
//调用无参构造器
public HashMap() {
    //默认初始容量大小为16,默认的加载因子为0.75f
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}

public HashMap(int initialCapacity, float loadFactor) {
    //初始容量不能小于0
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    //初始容量不能超过MAXIMUM_CAPACITY
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    //加载因子不能小于等于0,或者加载因子不能是非数字   
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    //设置加载因子
    this.loadFactor = loadFactor;
    //设置临界值
    threshold = initialCapacity;
    //伪构造,里面没有代码
    init();
}
```

**map.put(key1,value1)底层发生了什么?**

1.求key1的hash值，调用hashCode()，该hash值用来计算Entry数组下标，得到存放位置。
2.如果该位置为空，则添加成功。
3.如果不为空，意味着该位置存在链表：
		1.如果hash值与其他key不同，则添加成功。
		2.如果key1的hash值有key与之相同，则调用equal()，继续比较：
			  1.如果equal返回false，则添加成功。
			  2.如果equal返回true，则使用value1替换旧值（修改操作）
**map.get(key1)底层发生了什么？**

根据key1计算hash，找到对应数组下标。
遍历该位置处的链表，找到key1.equal(key2)为true的entry，返回其value。

### 扩容的原理？

扩容后，数组大小为原来的 2 倍。

HashMap计算添加元素的位置时，使用的位运算，这是特别高效的运算；另外，HashMap的初始容量是2的n次幂，扩容也是2倍的形式进行扩容，是因为容量是2的n次幂，可以使得添加的元素均匀分布在HashMap中的数组上，减少hash碰撞，避免形成链表的结构，使得查询效率降低！

### ConcurrentHashMap底层原理-jdk7

关键词：Segment数组/基于分段锁/提高并发

  1.引入一个Segment数组。每个Segment单元都包含一个与HashMap结构差不多hash表

   2.读取过程：

​		1.先取key的hash值，取高sshift位决定属于哪个Segment单元。
​		2.接着就是HashMap那一套
3.Segment继承jucReetrantLock，上锁方便，即分段锁。因此segment[1]锁了，不影响其他Segment单元并发。

### HashMap底层实现原理-jdk8

**与jdk7的不同的地方：**

 1.new HashMap()时，不创建长度为16的数组。

 2.底层使用Node[], 而不是Entry[]

 3.数组结构采用

​		数组+链表+红黑树

​			1.触发时机：当某索引位置链表长度>8 且 数组长度>64时，次索引位置的链表改为红黑树
​			2.红黑树的关键性质: 从根到叶子最长的可能路径不多于最短的可能路径的两倍长。结果是这棵树大致上是平衡的。
​			3.目的：加快检索速度

### ! ConcurrentHashMap底层原理-jdk8

https://www.cnblogs.com/ylspace/p/12726672.html

1.8版本的ConcurrentHashMap的实现与1.7版本有很大的差别，放弃了段锁的概念，借鉴了HashMap的数据结构：数组＋链表＋红黑树。

ConcurrentHashMap不接受nullkey和nullvalue。

底层结构

和 1.8 HashMap 结构类似，也是数组+链表+红黑树的。取消了Segment 分段锁
那如何保证线程安全的？

CAS + synchronized + volatile 来保证并发安全性，具体的如下
put方法逻辑

```java
/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    //1. 计算key的hash值
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        //2. 判断Node[]数组是否初始化，没有则进行初始化操作
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        //3. 通过hash定位数组的索引坐标，是否有Node节点，如果没有则使用CAS进行添加（链表的头节点），添加失败则进入下次循环。
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //4. 检查到内部正在扩容，就帮助它一块扩容。
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        // 如果该坐标Node不为null且没有正在扩容，就加锁，进行链表/红黑树 节点添加操作
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    //5. 当前为链表，在链表中插入新的键值对
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    // 6.当前为红黑树，将新的键值对插入到红黑树中
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            // 7.判断链表长度已经达到临界值8（默认值），当节点超过这个值就需要把链表转换为树结构
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    //8.对当前容量大小进行检查，如果超过了临界值（实际大小*加载因子）就需要扩容 
    addCount(1L, binCount);
    return null;
}
```

解析：对当前的table进行无条件自循环直到put成功（cas自旋）。

​		1.如果数组下标没有Node节点，就用CAS+自旋添加链表头节点。
​		2.如果有Node节点，就加synchronized，添加链表或红黑树节点。
get操作，由于数组被volatile修饰了，因此不用担心数组的可见性问题。

```java
volatile Node<K,V>[] table;
```

### Map类的各种对比

- HashMap和ConcurrentHashMap对比
- HashMap和HashTable的对比
- HashTable和ConcurrentHashMap对比

# Java锁

### Java 15锁，列举一些？

1.公平锁 / 非公平锁
2.可重入锁 / 不可重入锁
3.独享锁 / 共享锁
4.互斥锁 / 读写锁
5.乐观锁 / 悲观锁
6.分段锁
7.偏向锁 / 轻量级锁 / 重量级锁
8.自旋锁

### 公平和非公平锁是什么？两者区别（优缺点）？两种锁举例？

**是什么？**

​	公平锁：多个线程按照申请锁的顺序来获取锁。
​	非公平锁：多个线程获取锁的顺序不是按照申请舒顺序来的，有可能后申请的线程比先申请的线程优先获取锁。高并发下，有可能造成优先级反转或者饥饿现象。
**区别：**

​	公平锁：保证顺序（队列，FIFO），性能下降。
​	非公平：先尝试直接占有锁，如果尝试失败，再采用类似公平锁的方式。优点在于吞吐量比公平锁大。
**举例**

ReentrantLock可以指定创建公平锁或非公平锁，无参构造默认创建非公平锁。
synchronized是非公平的。

### 可重入锁是什么？与不可重入的区别？可重入锁举例？作用？实现原理？

是什么？

- 也叫递归锁
- 当一个线程获取某个对象锁后，可以再次获取同一把对象锁。
- 即，**线程可进入任何他所拥有的锁  所同步着的代码块。**

```java
public synchronized  void m1(){
      m1();//进入任何他所拥有的锁  所同步着的代码块
      m2();//进入任何他所拥有的锁  所同步着的代码块
}
public synchronized  void m2(){
     
}

```

​			可重入锁：某线程进入外层m1后，可以再次进入递归m1方法。也叫递归锁。
​			不可重入锁：某线程进入外部m1后，不可再进如内部m1，必须等待锁释放。这里就造成了死锁。
**举例：**

​		ReentrantLock/synchronized就是典型的可重入锁
**作用：**

​		避免死锁。案例：递归

**实现原理：**

​		计数器：进入最外层计数器=1，每递归一次，计数器+1，每退出一层，计数器-1，直到计数器=0，说明退出		了最外层，此时该线程释放锁对象，其他线程才能获取该锁。 

### 可重入锁代码验证 

#### synchronized

```java
public class SynchronizedLockDemo {

    public synchronized void m1(){
        System.out.println(Thread.currentThread().getName() + "----m1----");
        m2();
    }

    public synchronized void m2(){
        System.out.println(Thread.currentThread().getName() + "----m2----");
    }

    /**
     * 可以看出，m1,m2是同一把锁，只有线程释放最外层锁，其他线程才能占用该锁。
     * @param args
     */
    public static void main(String[] args) {
        SynchronizedLockDemo rd = new SynchronizedLockDemo();
        for (int i = 0; i < 5; i++) {
            new Thread(()->{
                rd.m1();
            },String.valueOf(i)).start();
        }
    }
     	//0----m1----
        //0----m2----
        //2----m1----
        //2----m2----
```

#### ReentrantLock

```java
public class ReentrantLockDemo {

    Lock lock = new ReentrantLock();

    public void m1(){
        lock.lock();
//        lock.lock();
        try {

            System.out.println(Thread.currentThread().getName() + "----m1----");
            m2();

        }finally {
            lock.unlock();
//            lock.unlock();
        }
    }

    public void m2(){
        lock.lock();
        try {

            System.out.println(Thread.currentThread().getName() + "----m2----");

        }finally {
            lock.unlock();
        }
    }

    /**
     * 锁要成对出现，否则：
     *  - 多一个lock.lock()会造成锁无法释放,程序卡住
     *  - 多一个lock.unlock()直接报错
     */
    public static void main(String[] args) {
        ReentrantLockDemo rd = new ReentrantLockDemo();
        for (int i = 0; i < 5; i++) {
            new Thread(()->{
                rd.m1();
            },String.valueOf(i)).start();
        }
        //0----m1----
        //0----m2----
        //2----m1----
        //2----m2----

```

### 自旋锁是什么？优点？缺点？

- 自旋锁（spinlock）是指尝试获取锁的对象不会立即阻塞，而是采用循环的方式取尝试获取锁。好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU

### 手写一个自旋锁

```java
public class SpinLockDemo {

    /**
     * 手写自旋锁
     * 自旋锁的核心：while+cas+原子引用线程
     *
     * A线程加锁，一顿操作5秒钟，解锁。B线程一直自旋等待A线程释放锁，然后获取锁。
     * 打印结果：
     * A尝试获取锁
     * A成功获取锁
     * A一顿操作5秒...
     * B尝试获取锁
     * A释放锁
     * B成功获取锁
     * B一顿操作
     * B释放锁
     */
    public static void main(String[] args) {
        SpinLockDemo sDemo = new SpinLockDemo();

        new Thread(()->{
            sDemo.myLock();

            System.out.println(Thread.currentThread().getName()+"一顿操作5秒...");
            try {TimeUnit.SECONDS.sleep(5);} catch (InterruptedException e) {e.printStackTrace();}

            sDemo.myUnLock();

        },"A").start();

        // 保证线程A先上的锁
        try {TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}

        new Thread(()->{
            sDemo.myLock();

            System.out.println(Thread.currentThread().getName()+"一顿操作");

            sDemo.myUnLock();

        },"B").start();

    }

    AtomicReference atomicThreadRef = new AtomicReference<Thread>();
    // 加锁
    public void myLock(){
        // 线程A进来，发现是null值，成功操作cas把自己放进去
        // 线程B进来，发现不是null值，一直自旋等待线程A释放锁
        System.out.println(Thread.currentThread().getName()+"尝试获取锁");
        while (!atomicThreadRef.compareAndSet(null,Thread.currentThread())){

        }
        System.out.println(Thread.currentThread().getName()+"成功获取锁");
    }

    // 释放锁
    public void myUnLock(){
        // 线程A发现原子引用是自己，于是cas成功修改为null值，即释放锁
        atomicThreadRef.compareAndSet(Thread.currentThread(),null);
        System.out.println(Thread.currentThread().getName()+"释放锁");
    }
}
```

### (读写锁)独占锁和共享锁是什么？举例？优缺点比较？

### 是什么？

​		独占锁：写锁，该锁只能被一个线程所持有。
​		共享锁：读锁，该锁可被多个线程所持有。
**举例**

​		ReentrantLock和sychronized都是独占锁。
​		ReentrantReadWriteLock，其读锁是共享锁，写锁是独占锁。
**优缺：**

​		共享锁保证并发读是非常高效的；读写、写读、写写过程是互斥的。

**写操作: 原子加独占,整个过程必须是一个完整的统一体,中间不允许被分割,被打断**

### 验证读写锁ReentrantReadWriteLock

并发读写不安全演示

```java
public class ReadWriteUnsafeDemo {

    Map<Integer,String> map = new HashMap<>();

    public void myPut(Integer key, String value){
        System.out.println(Thread.currentThread().getName() + "\t" + "正在写入："+key);
        map.put(key ,value);
        System.out.println(Thread.currentThread().getName() + "\t" + "写入完成："+key);
    }

    public void myGet(Integer key){
        System.out.println(Thread.currentThread().getName() + "\t" + "正在读取："+key);
        String value = map.get(key);
        System.out.println(Thread.currentThread().getName() + "\t" + "读取完成："+key+","+value);
    }

    /**
     * 并发读写不安全演示
     * 打印：
     * 0	正在写入：0
     * 2	正在写入：2
     * 4	正在写入：4
     * 3	正在写入：3
     * 1	正在写入：1
     * 3	写入完成：3
     * 0	正在读取：0
     * 4	写入完成：4
     * 0	读取完成：0,0
     * 2	写入完成：2
     * 0	写入完成：0
     * ...
     * 结论：写入不是原子操作，线程不安全
     *
     * 只锁put不锁get会发生什么？
     * 2	正在写入
     * 0	正在读取
     * 2	写入完成：2
     * 造成写时读，不安全。没有保证写的原子性。
     */
    public static void main(String[] args) {
        ReadWriteUnsafeDemo demo = new ReadWriteUnsafeDemo();
        for(int i=0; i<5; ++i){
            final int tmp = i;
            new Thread(()->{
                demo.myPut(tmp,String.valueOf(tmp));
            },String.valueOf(i)).start();
        }

        for(int i=0; i<5; ++i){
            final int tmp = i;
            new Thread(()->{
                demo.myGet(tmp);
            },String.valueOf(i)).start();
        }
    }
}
```

验证读写锁ReentrantReadWriteLock

```java
public class ReadWriteLockDemo {

    // 缓存资源都加一下volatile，保证线程间可见
    volatile Map<Integer,String> map = new HashMap<>();
    ReadWriteLock rwLock = new ReentrantReadWriteLock();

    public void myPut(Integer key, String value){

        rwLock.writeLock().lock();
        try{
            System.out.println(Thread.currentThread().getName() + "\t" + "正在写入");
            map.put(key ,value);
            System.out.println(Thread.currentThread().getName() + "\t" + "写入完成："+key);
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            rwLock.writeLock().unlock();
        }
    }

    public void myGet(Integer key){

        rwLock.readLock().lock();
        try{
            System.out.println(Thread.currentThread().getName() + "\t" + "正在读取");
            String value = map.get(key);
            System.out.println(Thread.currentThread().getName() + "\t" + "读取完成："+value);
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            rwLock.readLock().unlock();
        }
    }

    /**
     * 要求：
     * - 读可并发
     * - 写和任何操作互斥
     * 核心：ReentrantReadWriteLock + volatile
     * 打印：
     * 2	正在写入
     * 2	写入完成：2
     * 3	正在写入
     * 3	写入完成：3
     * 4	正在写入
     * 4	写入完成：4
     * 4	正在读取
     * 2	正在读取
     * 1	正在读取
     * 1	读取完成：1
     * 3	正在读取
     * 结论：读写锁保证了写原子性，读并发
     */
    public static void main(String[] args) {
        ReadWriteLockDemo demo = new ReadWriteLockDemo();
        for(int i=0; i<5; ++i){
            final int tmp = i;
            new Thread(()->{
                demo.myPut(tmp,String.valueOf(tmp));
            },String.valueOf(i)).start();
        }

        for(int i=0; i<5; ++i){
            final int tmp = i;
            new Thread(()->{
                demo.myGet(tmp);
            },String.valueOf(i)).start();
        }
    }
}
```

### 什么是乐观锁/悲观锁？举例？

**悲观锁**

总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，Java中synchronized和ReentrantLock等**独占锁就是悲观锁思想的实现。**

**乐观锁**

总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，采用版本号+cas的方式去保证线程安全。乐观锁适用于多读写少的应用类型，这样可以提高吞吐量。atomic包的类就是基于cas实现乐观锁的。

**什么是乐观读/悲观读？**

**悲观读**：在没有任何读写锁的时候才能取得写入的锁，可用于实现悲观读取（读优先，没有读时才能写），读多写少的场景下可能会出现线程饥饿。

**乐观读**：如果读多写少，就乐观的认为读写同时发生的情况少，因此不采用完全锁定的方式，而是采用cas实现乐观锁。

### ReentrantReadWriteLock是乐观读还是悲观读？

悲观读

### ! StempedLock作用？

它控制锁有三种模式（写、悲观读、乐观读）。

核心代码：

```java
private final StampedLock sl = new StampedLock();
long stamp = sl.tryOptimisticRead(); // 获得一个乐观读锁
// stamp=0表示没有写锁入侵，
long stamp = sl.readLock(); // 获取悲观读锁
long stamp = lock.writeLock();// 获得写锁
```

ReentrantReadWriteLock是悲观读，读优先，而StempedLock可以指定。

### synchronized和Lock的区别是什么？

1.原始构成
	synchronized是关键字，属于JVM层面（底层通过monitor对象完成，monitorenter\monitorexit）
	Lock是juc里具体的类，是API层面
2.使用方法
	synchronized不需要手动释放锁，代码块执行完就自动释放了
	ReentrantLock必须手动释放锁，否则可能导致死锁
3.等待是否可中断
	synchronized不可中断，除非抛异常或正常执行完毕
	ReentrantLock可中断
	1 lock.tryLock(long timeout, TimeUnit unit)
	2 lock.lockInterruptibly() 直接中断锁
4.加锁是否公平
	synchronized非公平
	ReentrantLock可以指定，默认非公平。
5.锁能否绑定多个条件（Condition）
	synchronized没有，要么随机唤醒一个，要么唤醒全部
	ReentrantLock实现分组唤醒，可精确唤醒。（详见AQS的Condition小节）
	Condition condition_pro = lock.newCondition();
	Condition condition_con = lock.newCondition();

### 死锁是什么？产生死锁的主要原因？

多线程抢资源造成互相等待

原因：

1. 系统资源不足
2. 进程推荐顺序不合适
3. 资源分配不当

### 死锁案例（demo）

```java
class HoldLockThread implements Runnable{
    private String lockA;
    private String lockB;

    public HoldLockThread(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }


    @Override
    public void run() {
        synchronized (lockA){
            System.out.println(Thread.currentThread().getName()+"\t持有"+lockA+"\t等待"+lockB);

            try {TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace(); }
            synchronized (lockB){
                System.out.println(Thread.currentThread().getName()+"\t持有"+lockB+"\t等待"+lockA);
            }

        }
    }
}

/**
 * 死锁案例
 * 打印：
 * A   持有钥匙1  等待钥匙2
 * B   持有钥匙2  等待钥匙1
 */
public class DeadLockDemo {

    public static void main(String[] args) {
        new Thread(new HoldLockThread("钥匙1","钥匙2"),"A").start();
        new Thread(new HoldLockThread("钥匙2","钥匙1"),"B").start();

    }
}
```

### 怎么排查死锁问题？使用过哪些命令行工具？

解决：
jps -l // 查看java进程 进程号
jstack 进程号 // 查看堆栈信息
打印：Found 1 deadlock.

![image-20200624193534268](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200624193534268.png)



# AQS

**什么是AQS?**

- ​    AbstractQueuedSynchronizer类 - AQS
- ​	是juc实现锁或其他同步工具的基础框架，数据结构是队列

- ​	基于AQS的同步组件：CountDownLatch、Semaphore、CyclicBarrier、ReentrantLock、Condition
  

## CountDownLatch -简述？应用场景？应用案例？常用方法？

(倒计时 )计数器  **阻塞某线程直到前面所有线程执行完才被唤醒。**

```java
public class CountDownLatchDemo {

    CountDownLatch countDownLatch = new CountDownLatch(6);

    /**
     * 一个需求：某线程要在前面线程都执行完任务后才能执行。
     * CountLatch应用场景：阻塞某线程直到前面所有线程执行完才被唤醒。
     * demo.countDownLatch.await();
     *      等待前面线程执行完
     * countDownLatch.await(10, TimeUnit.MILLISECONDS);
     *      只等待指定时间，超过就不继续等待。
     * 打印：
     * 0	执行任务
     * 4	执行任务
     * 3	执行任务
     * 2	执行任务
     * 1	执行任务
     * 5	执行任务
     * main	最后执行执行任务
     */
    public static void main(String[] args) throws InterruptedException {

        CountDownLatchDemo demo = new CountDownLatchDemo();

        for(int i=0; i<6; ++i){
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"\t"+"执行任务");
                // 一个线程执行完任务就-1
                demo.countDownLatch.countDown();
            },String.valueOf(i)).start();
        }

        // 阻塞，等待countDownLatch变为0，即所有任务线程执行完。
        demo.countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"\t"+"最后执行执行任务");
    }
}
```

## CyclicBarrier- 简述？应用场景？应用案例？常用方法？

它与CountDownLatch 正好相反

使一组线程到达一个屏障（Barrier）时被阻塞，直到最后一个线程到达屏障时，所有被屏障拦截的线程才能全部被唤醒继续操作。

```java
组线程到达一个屏障（Barrier）时被阻塞，直到最后一个线程到达屏障时，所有被屏障拦截的线程才能全部被唤醒继续操作。
 *  - 使用方法：线程调用await(),会进入等待状态，且计数器-1，直到计数器=0时，所有线程被唤醒。
 *  - 应用场景：5个线程执行完ready任务后都相互等待，直到5个线程的ready任务都执行完毕再执行continue任务。
 *  - 常用方法：
 *      barrier.await();
 *          等待其他线程
 *      barrier.await(2000, TimeUnit.MILLISECONDS);
 *          等待指定时间后会抛异常。
 *      new CyclicBarrier(5, () -> {})
 *          在ready和continue之间会执行的代码放在此处
 *  - 计数器重复使用（Cyclic）：cyclicBarrier.reset();
 *
 * 打印：
 * 0	执行ready
 * 2	执行ready
 * 3	执行ready
 * 1	执行ready
 * 4	执行ready
 * 集中点，此时所有线程完成ready操作
 * 4	执行continue
 * 3	执行continue
 * 0	执行continue
 * 1	执行continue
 * 2	执行continue
 */
public class CyclicBarrierDemo {

    CyclicBarrier cyclicBarrier = new CyclicBarrier(5, () -> {
        System.out.println("集中点，此时所有线程完成ready操作");
    });

    public static void main(String[] args) {
        CyclicBarrierDemo demo = new CyclicBarrierDemo();

        for(int i=0; i<5; ++i){
            new Thread(()->{

                System.out.println(Thread.currentThread().getName()+"\t"+"执行ready");
                try {
                    demo.cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"\t"+"执行continue");
            },String.valueOf(i)).start();
        }

        // 循环使用计数器
        demo.cyclicBarrier.reset();

        // 下一波操作
    }
}
```

## Semaphore- 简述？应用场景？应用案例？常用方法？

semaphore 是信号灯的意思  线程同步的辅助类，可以维护当前访问自身的线程个数，并提供了同步机制 使用Semaphore可以控制同时访问资源的线程个数，例如，实现一个文件允许的并发访问数。

比如我们某个资源的最大访问资源是10个线程  当达到10个线程时 其他线程会被阻塞  当10个中释放掉一个进来一个 

```java
/**
 * Semaphore
 *  - 使用场景：
 *      demo1-共享资源互斥（吃海底捞，座位有限，但是可以等待别人吃完，再进去，不用离开）
 *      demo2-控制并发数（线程池，超过并发数，就抛弃连接）
 *  demo1-打印：
 * 0   占到座位
 * 1   占到座位
 * 2   占到座位
 * 0   释放座位
 * 2   释放座位
 * 3   占到座位
 * 4   占到座位
 * 1   释放座位
 * 5   占到座位
 * 3   释放座位
 * 5   释放座位
 * 4   释放座位
 *  demo2-打印：
 * 0   尝试获取连接
 * 2   尝试获取连接
 * 3   尝试获取连接
 * 1   尝试获取连接
 * 5   尝试获取连接
 * 4   尝试获取连接
 * 5   连接成功
 * 1   连接成功
 * 4   连接成功
 * 3   连接失败
 * 0   连接失败
 * 2   连接失败
 */
public class SemaphoreDemo {

    public static void main(String[] args) {
        // demo1
        haidilao();
        // demo2
        xianchengchi();
    }

    // 线程池：最大连接数为3，超过就抛弃连接。
    private static void xianchengchi() {
        Semaphore semaphore = new Semaphore(3);
        for(int i=0; i<6; ++i){
            new Thread(()->{
                try {
                    System.out.println(Thread.currentThread().getName()+"\t"+"尝试获取连接");
                    // 只等200毫秒，等不到就抛弃，连接失败
                    if(semaphore.tryAcquire(200, TimeUnit.MILLISECONDS)){
                        System.out.println(Thread.currentThread().getName()+"\t"+"连接成功");
                        TimeUnit.SECONDS.sleep(1);
                    }else {
                        System.out.println(Thread.currentThread().getName()+"\t"+"连接失败");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }

    // 海底捞：3个座位6个人
    private static void haidilao() {
        Semaphore semaphore = new Semaphore(3);
        for(int i=0; i<6; ++i){
            new Thread(()->{
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"\t"+"占到座位");
                    // 吃一会儿
                    TimeUnit.SECONDS.sleep(3);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    System.out.println(Thread.currentThread().getName()+"\t"+"释放座位");
                    semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

## Condition是什么？核心方法？使用案例？

Condition：多线程间协调通信的工具类
核心：

```java
Condition condition = reentrantLock.newCondition();
condition.await();
condition.signalAll();
```

## Condition分组唤醒是什么？锁绑定多个Condition使用案例？

核心代码：

```java
// 生产者组
Condition condition_pro = lock.newCondition();
// 消费者组
Condition condition_con = lock.newCondition();
// 阻塞当前生产者线程
condition_pro.await();
// 唤醒所有生产者
condition_pro.signalAll();
// 阻塞当前消费者线程
condition_con.await();
// 唤醒所有消费者
condition_con.signalAll();
```

锁绑定多个Condition使用案例：

```java
/**
 * 锁绑定多个条件（Condition） 案例
 * 要求：A打印3次，通知B打印4次，通知C打印5次；循环5轮；
 *
 * 线程   操作  资源类
 * 判断   干活  通知
 * 防止假唤醒while
 */
public class MultiConditionDemo {

    public static void main(String[] args) {
        Resources rs = new Resources();
        new Thread(()->{
            for (int i = 0; i < 5; i++) {
                rs.pa3();
            }
        },"A").start();

        new Thread(()->{
            for (int i = 0; i < 5; i++) {
                rs.pb4();
            }
        },"B").start();

        new Thread(()->{
            for (int i = 0; i < 5; i++) {
                rs.pc5();
            }
        },"C").start();
    }
}

class Resources{
    Lock lock = new ReentrantLock();
    String thread = "A";    // 当前线程
    Condition cA = lock.newCondition();
    Condition cB = lock.newCondition();
    Condition cC = lock.newCondition();

    public void pa3(){
        lock.lock();
        try{
            while(!thread.equals("A")){
                cA.await();
            }
            for (int i = 0; i < 3; i++) {
                System.out.println("A打印");
            }
            // A做完了通知B
            thread="B";
            cB.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    public void pb4(){
        lock.lock();
        try{
            while(!thread.equals("B")){
                cB.await();
            }
            for (int i = 0; i < 4; i++) {
                System.out.println("B打印");
            }
            // B做完了通知C
            thread="C";
            cC.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    public void pc5(){
        lock.lock();
        try{
            while(!thread.equals("C")){
                cC.await();
            }
            for (int i = 0; i < 5; i++) {
                System.out.println("C打印");
            }
            // C做完了通知A
            thread="A";
            cA.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
```



# 阻塞队列

## 什么是阻塞队列？什么情况会导致阻塞是？主要应用场景？

队列：FILO

以下情况发生阻塞：

- 队满时 入队阻塞
- 队空时 出队阻塞

应用场景：生产者消费者模式

## 为什么要用阻塞队列？有什么好？

阻塞就是线程挂起，当满足条件后，又被唤醒。

为什么需要BlockingQueue?

- 不再需要关心线程阻塞和唤醒的时机，因为BlockingQueue包办了这个细节
- 在Juc出现前，程序员必须手动控制这些细节，兼顾效率和线程安全，增大了开发难度。

## BlockingQueue的架构图？有哪些实现类？

实现类：

1. ***ArrayBlockingQueue: 由数组结构组成的有界阻塞队列**
2. ***LinkedBlockingQueue: 由链表结构组成的有界阻塞队列**（但默认大小为Integer.MAX_VALUE，大小配置可选
3. PriorityBlockingQueue: 支持优先级排序的无界阻塞队列
4. DelayQueue: 使用优先级队列实现的延迟无界阻塞队列（内部采用PriorityQueue（排序）与ReentrantLock（锁）实现）
5. ***SynchronousQueue: 队列只插入一个元素，同步队列 也即 单个元素的队列**
6. LinkedTransferQueue: 由链表结构组成的无界阻塞队列
7. LinkedBlockingDeque：由链表结构组成的双向阻塞队列
   带 * 是重要的，是线程池的底层实现。

## ! BlockingQueue的核心方法？

三套操作：插入、移除、检查。

四套处理：抛异常，返回特殊值，阻塞，超时

一个图表

![image-20200622210505560](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200622210505560.png)

![image-20200622211613494](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200622211613494.png)

## 阻塞队列的应用场景举例？

1、生产者消费者模式

2、线程池底层原理

3、消息中间件

## 生产者消费者模式的传统版实现？（synchronized版和Lock版）

1、sync+wait+notify

```java
public class ProdConsumer_TDemo1 {

    public static void main(String[] args) {
        Number1 number = new Number1();

        new Thread(()->{
            for (int i = 0; i < 5; i++) {
                number.plus();
            }
        },"A").start();

        new Thread(()->{
            for (int i = 0; i < 5; i++) {
                number.reduce();
            }
        },"B").start();
    }
}

class Number1{
    private int num = 0;

    public synchronized void plus(){

        // 使用while不用if,防止线程虚假唤醒。当前线程被唤醒后，会重新走验证程序，不会直接往下执行。
        while( num!=0 ){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num++;
        System.out.println(Thread.currentThread().getName()+"\tplus"+"\tnum="+num);
        this.notify();
    }

    public synchronized void reduce(){

        // 使用while不用if,防止线程虚假唤醒。当前线程被唤醒后，会重新走验证程序，不会直接往下执行。
        while( num==0 ){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num--;
        System.out.println(Thread.currentThread().getName()+"\treduce"+"\tnum="+num);
        this.notify();
    }
}
```

2、lock+await+signalAll(Condition)

```java
class Number2{
    private int num = 0;
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    public void plus(){
        // 线程1调用了lock方法，加入到了AQS的等待队里里面去
        lock.lock();
        try{
            // 使用while不用if,防止线程虚假唤醒。当前线程被唤醒后，会重新走验证程序，不会直接往下执行。
            while( num!=0 ){
                // 调用await方法后，从AQS队列里移除了，进入到了condition队列里面去，等待一个信号
                condition.await();
            }
            num++;
            System.out.println(Thread.currentThread().getName()+"\tplus"+"\tnum="+num);
            // 调用signalAll发送信号的方法,Condition节点的线程1节点元素被取出，放在了AQS等待队列里（注意并没有被唤醒）
            condition.signalAll();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            // 线程1释放锁，这时候AQS队列中只剩下线程2，线程2开始执行
            lock.unlock();
        }

    }

    public void reduce(){
        lock.lock();
        try{
            // 使用while不用if,防止线程虚假唤醒。当前线程被唤醒后，会重新走验证程序，不会直接往下执行。
            while( num==0 ){
                condition.await();
            }
            num--;
            System.out.println(Thread.currentThread().getName()+"\treduce"+"\tnum="+num);
            condition.signalAll();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
public class ProdConsumer_TDemo2 {

    public static void main(String[] args) {
        Number2 number = new Number2();

        new Thread(()->{
            for (int i = 0; i < 5; i++) {
                number.plus();
            }
        },"A").start();

        new Thread(()->{
            for (int i = 0; i < 5; i++) {
                number.reduce();
            }
        },"B").start();
    }


}
```

## 生产者消费者模式的阻塞队列版实现？

```java
/**
 * 使用阻塞队列实现生产者消费者（不使用Lock/Condition相关的东西）。
 * 题目：当flag=true时，启动生产者和消费者线程，一直循环做生产消费，当flag=false时，停止所有线程，程序运行结束。
 * 测试：开启一个生产者和一个消费者，5秒钟修改flag，看执行效果。
 * 核心：
 *  num++操作改成 AtomicInteger;
 *  BlockingQueue不写具体类，留接口给外部；
 *  flag标记对所有线程可见，所以需要volatile。
 *  打印：
 *  启动生产者
 * 启动消费者
 * 生产者	成功入队	蛋糕1
 * 消费者	成功出队	蛋糕1
 * 生产者	成功入队	蛋糕2
 * 消费者	成功出队	蛋糕2
 * 生产者	成功入队	蛋糕3
 * 消费者	成功出队	蛋糕3
 * 生产者	成功入队	蛋糕4
 * 消费者	成功出队	蛋糕4
 * 生产者	成功入队	蛋糕5
 * 消费者	成功出队	蛋糕5
 * 消费者	消费等待超过2秒，消费失败
 */
public class ProdConsumer_BDemo {
    public static void main(String[] args) {
        // 如果生产速度快，就把容量弄大点，免得经常入队失败
        MyResource rs = new MyResource(new ArrayBlockingQueue<String>(3));

        new Thread(()->{
            System.out.println("启动"+Thread.currentThread().getName());
            try {
                rs.myProducer();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"生产者").start();

        new Thread(()->{
            System.out.println("启动"+Thread.currentThread().getName());
            try {
                rs.myConsumer();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"消费者").start();

        // 5s后修改flag,结束程序
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        rs.STOP();

    }
}

class MyResource{

    private AtomicInteger atomicInteger = new AtomicInteger(0);
    private BlockingQueue<String> blockingQueue = null;
    private volatile boolean FLAG=true;


    public MyResource(BlockingQueue<String> blockingQueue) {
        this.blockingQueue = blockingQueue;
    }

    public void STOP(){
        FLAG=false;
    }

    public void myProducer() throws InterruptedException {
        int id;
        String cake;
        boolean result;
        while (FLAG){
            // 创建蛋糕
            id = atomicInteger.incrementAndGet();// 蛋糕id
            cake = "蛋糕"+id;
            // 蛋糕入队
            result = blockingQueue.offer(cake, 2, TimeUnit.SECONDS);
            if(result){
                System.out.println(Thread.currentThread().getName()+"\t成功入队\t"+cake);
            }else {
                // 说明队列满了
                System.out.println(Thread.currentThread().getName()+"\t入队失败\t"+cake);
            }

            // 生产者产的慢，产一个消费者就吃一个。
            // 生产者产的快，就会入队失败，等消费者来吃。
            TimeUnit.SECONDS.sleep(1);
        }
    }

    public void myConsumer() throws InterruptedException {
        while (FLAG){
            // 消费蛋糕
            String cake = blockingQueue.poll(2, TimeUnit.SECONDS);
            if(cake==null || cake.equals("")){
                System.out.println(Thread.currentThread().getName()+"\t消费等待超过2秒，消费失败");
                // 通知生产者也退出，不然生产者一直循环尝试入队。
//                FLAG=false;
//                return;
            }else{
                System.out.println(Thread.currentThread().getName()+"\t成功出队\t"+cake);
            }
            // TimeUnit.SECONDS.sleep(2);
        }
    }
```

# 生存服务器变慢谈谈思路和性能分析

#### 整机  top  

![image-20200628133718550](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628133718550.png)

代表系统一分钟 五分钟 十五分钟 的平均负载值

如果相加除以三再乘100%大于60% 那就是负载压力重

这个命令 按键盘上的1 会出个每个核的情况和负载均衡

uptime (精简)

#### CPU:vmstat 

vmstat -n 2 3 表示每两秒取样一次共计取样三次

查看cpu 包含但不限于

![image-20200628135625372](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628135625372.png)

![image-20200628135649546](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628135649546.png)

![image-20200628135804102](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628135804102.png)

![image-20200628135911575](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628135911575.png)

#### 内存:free

![image-20200628140034624](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628140034624.png)

![image-20200628140257996](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628140257996.png)

#### 硬盘: df

df -h

#### 磁盘IO: iostat

![image-20200628140617612](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628140617612.png)

![image-20200628140514859](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628140514859.png)

![image-20200628140537915](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628140537915.png)

#### 网络IO:ifstat

![image-20200628140735571](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628140735571.png)

# 假如生成环境出现cpu过高 谈谈分析思路和定位

结合linux和JDK的命令一起分析

1. 先用top命令找出cpu占比最高的

2. ps -ef或者jps 进一步定位 找出是那个后台程序

   ![image-20200628141333259](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628141333259.png)

3. 定位到具体的线程或者代码

   ps -mp 进程 -o THREAD,tid.time![image-20200628141600841](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628141600841.png)

   

   ![image-20200628141523228](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628141523228.png)

   

4. 将需要的线程ID转换为16进制格式(英文小写格式)

   ![image-20200628141721653](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628141721653.png)

5. jstack 进程ID | grep tid(16进制线程ID小写英文)-A60

![image-20200628142022237](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628142022237.png)

# 对于JDK自带的JVM监控和性能分析工具用过哪些 一般你是怎么用的

# GITHUB 骚操作

看github优秀的框架和源码提升自己

![image-20200628142957263](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628142957263.png)

![image-20200628143201671](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628143201671.png)

### in关键词限制搜索范围

![image-20200628143526403](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628143526403.png)

seckill in:name,readme

### stars或fork数量关键词去查找

![image-20200628144145956](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628144145956.png)

![image-20200628144502451](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628144502451.png)

![image-20200628144610732](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628144610732.png)

![image-20200628144629523](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628144629523.png)

### awesome 加强搜索  搜索优秀的框架 教程(非常重要)

![image-20200628144953442](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628144953442.png)

![image-20200628145209581](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628145209581.png)

高亮显示一段 #L13-L23

#### 在仓库项目中搜索

![image-20200628145543338](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628145543338.png)

#### 搜索某个地区的大佬

![image-20200628145815254](E:\mianshixuexi\wangzqstudy\JVMJUC.assets\image-20200628145815254.png)





