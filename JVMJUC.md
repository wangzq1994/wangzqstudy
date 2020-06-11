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