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