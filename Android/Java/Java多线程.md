

[TOC]



## 基本概念    

​    每个正在系统上运行的程序都是一个进程。每个进程包含一到多个线程。线程是一组指令的集合，或者是程序的特殊段，它可以在程序里独立执行。也可以把它理解为代码运行的上下文 通常由操作系统负责多个线程的调度和执行。

​        使用线程可以把占据时间长的程序中的任务放到后台去处理，程序的运行速度可能加快，在一些等待的任务实现上如用户输入、文件读写和网络收发数据等，线程就比较有用了。在这种情况下可以释放一些珍贵的资源如内存占用等等。

​        如果有大量的线程,会影响性能，因为操作系统需要在它们之间切换，更多的线程需要更多的内存空间，线程的中止需要考虑其对程序运行的影响。通常块模型数据是在多个线程间共享的，需要防止线程死锁情况的发生。

**总结:进程是所有线程的集合，每一个线程是进程中的一条执行路径。**

个人觉得线程是主要是2个目的：

1，防止主线程阻塞

2，充分利用系统资源。在一些“等待”的时候让系统资源处理其他任务。这个等待可以是内存处理完了等待网络真正发送出去，或者等待数据写入到外部存储等。



### 启动线程3中方式：

1，继承thread

2，实现runable

```
CreateRunnable createThread = new CreateRunnable(); 
 Thread thread = new Thread(createThread);
 thread.start();
```

3，匿名内部类 直接



```
Thread thread = new Thread(new Runnable() {
    public void run() { 
    }
});
thread.start();

```

**使用实现实现Runnable接口好，原因实现了接口还可以继续继承，继承了类不能再继承。**

**开始执行线程 注意 开启线程不是调用run方法，而是start方法**

**调用run只是使用实例调用方法。**



### 守护线程

Java中有两种线程，一种是用户线程，另一种是守护线程。 用户线程是指用户自定义创建的线程，主线程停止，用户线程不会停止守护线程当进程不存在或主线程停止，守护线程也会被停止。 使用setDaemon(true)方法设置为守护线程。

```
public class DaemonThread {
    public static void main(String[] args) {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                 System.out.println("我是子线程...1");
                    try {
                        Thread.sleep(100);
                    } catch (Exception e) {
                        System.out.println("Exception");
                    }
                    System.out.println("我是子线程...");
                }
            }
        });
        thread.setDaemon(true);
        thread.start();
        for (int i = 0; i < 10; i++) {
            try {
                Thread.sleep(100);
            } catch (Exception e) {

            }
            System.out.println("我是主线程");
        }
        System.out.println("主线程执行完毕!");
    } 
}
```

主线程执行完毕后，子线程也不再运行

 ![image-20210309145421427](D:\07_个人资料\Java\Java多线程.assets\image-20210309145421427.png)

![image-20210309145509073](D:\07_个人资料\Java\Java多线程.assets\image-20210309145509073.png)



不同的结果

### 线程状态

# ![image-20210309145813023](D:\07_个人资料\Java\Java多线程.assets\image-20210309145813023.png)

​                               

 线程从创建、运行到结束总是处于下面五个状态之一：新建状态、就绪状态、运行状态、阻塞状态及死亡状态。

## **新建状态**

  当用new操作符创建一个线程时， 例如new Thread(r)，线程还没有开始运行，此时线程处在新建状态。 当一个线程处于新生状态时，程序还没有开始运行线程中的代码

## 就绪状态

一个新创建的线程并不自动开始运行，要执行线程，必须调用线程的start()方法。当线程对象调用start()方法即启动了线程，**start()方法创建线程运行的系统资源，并调度线程运行run()方法**。当start()方法返回后，线程就处于就绪状态。

   **处于就绪状态的线程并不一定立即运行run()方法**，线程还必须同其他线程竞争CPU时间，只有获得CPU时间才可以运行线程。**因为在单CPU的计算机系统中，不可能同时运行多个线程，一个时刻仅有一个线程处于运行状态。**因此此时可能有多个线程处于就绪状态。对多个处于就绪状态的线程是由[Java](http://lib.csdn.net/base/java)运行时系统的线程调度程序(*thread scheduler*)来调度的。

多核应该同时可以有多个程序运行

## 运行状态

当线程获得CPU时间后，它才进入运行状态，真正开始执行run()方法.

## 阻塞状态

  线程运行过程中，可能由于各种原因进入阻塞状态:
     1>线程通过调用sleep方法进入睡眠状态；
     2>线程调用一个在I/O上被阻塞的操作，即该操作在输入输出操作完成之前不会返回到它的调用者；
     3>线程试图得到一个锁，而该锁正被其他线程持有；
     4>线程在等待某个触发条件；
 

## 死亡状态

有两个原因会导致线程死亡：
  1) run方法正常退出而自然死亡，
  2) 一个未捕获的异常终止了run方法而使线程猝死。
  为了确定线程在当前是否存活着（就是要么是可运行的，要么是被阻塞了），需要使用isAlive方法。如果是可运行或被阻塞，这个方法返回true； 如果线程仍旧是new状态且不是可运行的， 或者线程死亡了，则返回false.



### join()方法作用



join作用是让其他线程变为等待,   t1.join();// 让其他线程变为等待，直到当前t1线程执行完毕，才释放。

thread.Join把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。比如在线程B中调用了线程A的Join()方法，直到线程A执行完毕后，才会继续执行线程B。

### 优先级

现代操作系统基本采用时分的形式调度运行的线程，线程分配得到的时间片的多少决定了线程使用处理器资源的多少，也对应了线程优先级这个概念。在JAVA线程中，通过一个int priority来控制优先级，范围为1-10，其中10最高，默认值为5。下面是源码（基于1.8）中关于priority的一些量和方法。



```
  public static void main(String[] args) {


        // 注意设置了优先级， 不代表每次都一定会被执行。 只是CPU调度会有限分配
        for(int i=0;i<10;i++) {
            PrioritytThread prioritytThread = new PrioritytThread();
            Thread t1 = new Thread(prioritytThread);
            Thread t2 = new Thread(prioritytThread);
            t1.setPriority(1);
            t1.setName("thread1");
            t1.start();
            t2.setName("thread2");
            t2.setPriority(10);
            t2.start();
        }
    }

    public static class PrioritytThread implements Runnable {

        public void run() {
            long startTime = System.currentTimeMillis();
            long addResult = 0;
            for (int i = 0; i < 10000000; i++) {
                new Random().nextInt();
                addResult += i;
            }
            long endTime = System.currentTimeMillis();
            System.out.println(Thread.currentThread().toString()+" " + (endTime - startTime));
        }
    }
```





```
Thread[thread2,10,main] 5887
Thread[thread2,10,main] 6488
Thread[thread2,10,main] 7245
Thread[thread2,10,main] 7302
Thread[thread2,10,main] 8446
Thread[thread2,10,main] 8626
Thread[thread2,10,main] 8761
Thread[thread2,10,main] 9551
Thread[thread2,10,main] 9965
Thread[thread2,10,main] 10026
Thread[thread1,1,main] 15795
Thread[thread1,1,main] 15925
Thread[thread1,1,main] 16117
Thread[thread1,1,main] 16255
Thread[thread1,1,main] 16367
Thread[thread1,1,main] 16381
Thread[thread1,1,main] 16338
Thread[thread1,1,main] 16292
Thread[thread1,1,main] 16874
Thread[thread1,1,main] 16906
```

也可能会有 某几个thread1 在thread2前面执行结束的情况

如果改成 

```
t1.setPriority(5); 
t2.setPriority(6);
```

```
Thread[thread1,5,main] 15732
Thread[thread2,6,main] 15961
Thread[thread1,5,main] 16001
Thread[thread1,5,main] 16029
Thread[thread2,6,main] 16083
Thread[thread2,6,main] 16198
Thread[thread1,5,main] 16628
Thread[thread2,6,main] 16340
Thread[thread2,6,main] 16394
Thread[thread1,5,main] 16489
Thread[thread1,5,main] 16420
Thread[thread2,6,main] 16689
Thread[thread2,6,main] 16484
Thread[thread1,5,main] 16455
Thread[thread2,6,main] 16754
Thread[thread1,5,main] 16782
Thread[thread2,6,main] 16790
Thread[thread1,5,main] 16814
Thread[thread2,6,main] 16745
Thread[thread1,5,main] 16830
```

相差不大的情况效果不明显

如果我们把优先级设置近点的话，发现优先级较高的线程不一定没一次都执行完，线程的优先级与打印的顺序无关，不要将这两点的关系相关联，他们的关系是不确定性和随机性。

线程的优先级仍然无法保障线程的执行次序。只不过，优先级高的线程获取CPU资源的概率较大，优先级低的并非没机会执行。

线程的优先级具有继承性，比如A线程启动B线程，则A和B的线程优先级是一样的。 







#### 现在有T1、T2、T3三个线程，你怎样保证T2在T1执行完后执行，T3在T2执行完后执行

```
 final Thread t1 = new Thread(new Runnable() {
            public void run() {
                for (int i = 0; i < 20; i++) {
                    System.out.println("t1,i:" + i);
                }
            }
        });
        final Thread t2 = new Thread(new Runnable() {
            public void run() {
                try {
                    t1.join();
                } catch (Exception e) {
                    // TODO: handle exception
                }
                for (int i = 0; i < 20; i++) {
                    System.out.println("t2,i:" + i);
                }
            }
        });
        final Thread t3 = new Thread(new Runnable() {
            public void run() {
                try {
                    t2.join();
                } catch (Exception e) {
                    // TODO: handle exception
                }
                for (int i = 0; i < 20; i++) {
                    System.out.println("t3,i:" + i);
                }
            }
        });
        t1.start();
        t2.start();
        t3.start();
```





### Yield方法

 

Thread.yield()方法的作用：暂停当前正在执行的线程，并执行其他线程。（可能没有效果）

yield()让当前正在运行的线程回到可运行状态，以允许具有相同优先级的其他线程获得运行的机会。因此，使用yield()的目的是让具有相同优先级的线程之间能够适当的轮换执行。但是，实际中无法保证yield()达到让步的目的，因为，让步的线程可能被线程调度程序再次选中。

结论：大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。



# 多线程同步

## 

当多个线程同时共享，同一个**全局变量或静态变量**，做写的操作时，可能会发生数据冲突问题，也就是线程安全问题

## 锁

同步代码块 

synchronized(对象)//这个对象可以为任意对象 

{ 

  需要被同步的代码 

} 



什么是同步函数？

  答：在方法上修饰synchronized 称为同步函数

**同步函数用的是什么锁？**

答：同步函数使用this锁

**什么是静态同步函数？**

方法上加上static关键字，使用synchronized 关键字修饰 或者使用类.class文件。

静态的同步函数使用的锁是 该函数所属字节码文件对象 

可以用 getClass方法获取，也可以用当前 类名.class 表示。

代码样例:

```
synchronized (ThreadTrain.class) {
			System.out.println(Thread.currentThread().getName() + ",出售 第" + (100 - trainCount + 1) + "张票.");
			trainCount--;
			try {
				Thread.sleep(100);
			} catch (Exception e) {
			}
}

```

**synchronized** **修饰方法使用锁是当前this锁**。

**synchronized** **修饰静态方法使用锁是当前类的字节码文件**



### Volatile

Volatile 关键字的作用是变量在多个线程之间可见

```
class ThreadVolatileDemo extends Thread {
    public boolean flag = true;
    @Override
    public void run() {
        System.out.println("开始执行子线程....");
        while (flag) {
        }
        System.out.println("线程停止");
    }
    public void setRuning(boolean flag) {
        this.flag = flag;
    }
}
public class ThreadVolatile {
    public static void main(String[] args) throws InterruptedException {
        ThreadVolatileDemo threadVolatileDemo = new ThreadVolatileDemo();
        threadVolatileDemo.start();
        Thread.sleep(3000);
        threadVolatileDemo.setRuning(false);
        System.out.println("flag 已经设置成false");
        Thread.sleep(1000);
        System.out.println(threadVolatileDemo.flag);
    }
}
```

![image-20210309184056160](D:\07_个人资料\Java\Java多线程.assets\image-20210309184056160.png)

已经将结果设置为fasle为什么？还一直在运行呢。

**原因:线程之间是不可见的，读取的是副本，没有及时读取到主内存结果。**

**解决办法使用Volatile关键字将解决线程之间可见性, 强制线程每次读取该值的时候都去“主内存”中取值**

如果改成 

```
volatile public boolean flag = true;
```

![image-20210309184254066](D:\07_个人资料\Java\Java多线程.assets\image-20210309184254066.png)





```
public class VolatileNoAtomic extends Thread {
    private static volatile int count;
    private static void addCount() {
        for (int i = 0; i < 1000; i++) {
            count++;
        }
        System.out.println(count);
    }
    public void run() {
        addCount();
    }
    public static void main(String[] args) {
        VolatileNoAtomic[] arr = new VolatileNoAtomic[100];
        for (int i = 0; i < 10; i++) {
            arr[i] = new VolatileNoAtomic();
        }
        for (int i = 0; i < 10; i++) {
            arr[i].start();
        }
    }
}
```

![image-20210309184750598](D:\07_个人资料\Java\Java多线程.assets\image-20210309184750598.png)

**结果发现 数据不同步，因为Volatile不用具备原子性。**

可以改为使用AtomicInteger，是一个提供原子操作的Integer类，通过线程安全的方式操作加减。

```
private static AtomicInteger atomicInteger = new AtomicInteger(0);
atomicInteger.incrementAndGet();
```



### volatile与synchronized区别

仅靠volatile不能保证线程的安全性。（原子性）

①volatile轻量级，只能修饰变量。synchronized重量级，还可修饰方法

②volatile只能保证数据的可见性，不能用来同步，因为多个线程并发访问volatile修饰的变量不会阻塞。

synchronized不仅保证可见性，而且还保证原子性，因为，只有获得了锁的线程才能进入临界区，从而保证临界区中的所有语句都全部执行。多个线程争抢synchronized锁对象时，会出现阻塞。

**线程安全性**

线程安全性包括两个方面，①可见性。②原子性。

从上面自增的例子中可以看出：仅仅使用volatile并不能保证线程安全性。而synchronized则可实现线程的安全性。









缺少各个锁的比较，原理！！

### Java锁的深度化



#### 悲观锁与乐观锁

**悲观锁:**悲观锁悲观的认为每一次操作都会造成更新丢失问题，在每次查询时加上排他锁。

每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。

Select * from xxx for update;

**乐观锁:**乐观锁会乐观的认为每次查询都不会造成更新丢失,利用版本字段控制

#### 重入锁

锁作为并发共享数据，保证一致性的工具，在JAVA平台有多种实现(如 synchronized 和 ReentrantLock等等 ) 。这些已经写好提供的锁为我们开发提供了便利。

重入锁，也叫做**递归锁**，指的是同一线程 外层函数获得锁之后 ，**内层递归函数仍然有获取该锁的代码**，但不受影响。
 在JAVA环境下 ReentrantLock 和synchronized 都是 可重入锁

需要补充百度差别和原理 源码



#### 读写锁

相比[Java中的锁(Locks in Java)](http://ifeve.com/locks/)里Lock实现，读写锁更复杂一些。假设你的程序中涉及到对一些共享资源的读和写操作，且写操作没有读操作那么频繁。在没有写操作的时候**，两个线程同时读一个资源没有任何问题，所以应该允许多个线程能在同时读取共享资源。但是如果有一个线程想去写这些共享资源，就不应该再有其它线程对该资源进行读或写**（译者注：也就是说：读-读能共存，读-写不能共存，写-写不能共存）。这就需要一个读/写锁来解决这个问题。Java5在java.util.concurrent包中已经包含了读写锁。尽管如此，我们还是应该了解其实现背后的原理。

```
static Map<String, Object> map = new HashMap<String, Object>();
static ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
static Lock r = rwl.readLock();
static Lock w = rwl.writeLock();

// 获取一个key对应的value
public static final Object get(String key) {
    r.lock();
    try {
        System.out.println("正在做读的操作,key:" + key + " 开始");
        Thread.sleep(100);
        Object object = map.get(key);
        System.out.println("正在做读的操作,key:" + key + " 结束");
        System.out.println();
        return object;
    } catch (InterruptedException e) { 
    } finally {
        r.unlock();
    }
    return key;
}

// 设置key对应的value，并返回旧有的value
public static final Object put(String key, Object value) {
    w.lock();
    try { 
        System.out.println("正在做写的操作,key:" + key + ",value:" + value + "开始.");
        Thread.sleep(100);
        Object object = map.put(key, value);
        System.out.println("正在做写的操作,key:" + key + ",value:" + value + "结束.");
        System.out.println();
        return object;
    } catch (InterruptedException e) { 
    } finally {
        w.unlock();
    }
    return value;
} 
// 清空所有的内容
public static final void clear() {
    w.lock();
    try {
        map.clear();
    } finally {
        w.unlock();
    }
} 
public static void main(String[] args) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                put(i + "", i + "");
            }
        }
    }).start();
    new Thread(new Runnable() {
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                get(i + "");
            }
        }
    }).start();
}
```

正在做写的操作,key:0,value:0开始.
正在做写的操作,key:0,value:0结束.

正在做写的操作,key:1,value:1开始.
正在做写的操作,key:1,value:1结束.

正在做写的操作,key:2,value:2开始.
正在做写的操作,key:2,value:2结束.

正在做写的操作,key:3,value:3开始.
正在做写的操作,key:3,value:3结束.

正在做写的操作,key:4,value:4开始.
正在做写的操作,key:4,value:4结束.

正在做读的操作,key:0 开始
正在做读的操作,key:0 结束

正在做写的操作,key:5,value:5开始.
正在做写的操作,key:5,value:5结束.

正在做写的操作,key:6,value:6开始.
正在做写的操作,key:6,value:6结束.

正在做读的操作,key:1 开始
正在做读的操作,key:1 结束

正在做写的操作,key:7,value:7开始.
正在做写的操作,key:7,value:7结束.

正在做写的操作,key:8,value:8开始.
正在做写的操作,key:8,value:8结束.

正在做写的操作,key:9,value:9开始.
正在做写的操作,key:9,value:9结束.

正在做读的操作,key:2 开始
正在做读的操作,key:2 结束

正在做读的操作,key:3 开始
正在做读的操作,key:3 结束

正在做读的操作,key:4 开始
正在做读的操作,key:4 结束

正在做读的操作,key:5 开始
正在做读的操作,key:5 结束

正在做读的操作,key:6 开始
正在做读的操作,key:6 结束

正在做读的操作,key:7 开始
正在做读的操作,key:7 结束

正在做读的操作,key:8 开始
正在做读的操作,key:8 结束

正在做读的操作,key:9 开始
正在做读的操作,key:9 结束





#### CAS无锁机制

（1）与锁相比，使用比较交换（下文简称CAS）会使程序看起来更加复杂一些。但由于其非阻塞性，它对死锁问题天生免疫，并且，线程间的相互影响也远远比基于锁的方式要小。更为重要的是，使用无锁的方式完全没有锁竞争带来的系统开销，也没有线程间频繁调度带来的开销，因此，它要比基于锁的方式拥有更优越的性能。

（2）无锁的好处：

第一，在高并发的情况下，它比有锁的程序拥有更好的性能；

第二，它天生就是死锁免疫的。

就凭借这两个优势，就值得我们冒险尝试使用无锁的并发。

（3）CAS算法的过程是这样：**它包含三个参数CAS(V,E,N): V表示要更新的变量，E表示预期值，N表示新值。仅当V值等于E值时，才会将V的值设为N，如果V值和E值不同，则说明已经有其他线程做了更新，则当前线程什么都不做。最后，CAS返回当前V的真实值。**

（4）CAS操作是抱着乐观的态度进行的，它总是认为自己可以成功完成操作。当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余均会失败。失败的线程不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。基于这样的原理，CAS操作即使没有锁，也可以发现其他线程对当前线程的干扰，并进行恰当的处理。

（5）简单地说，CAS需要你额外给出一个期望值，也就是你认为这个变量现在应该是什么样子的。如果变量不是你想象的那样，那说明它已经被别人修改过了。你就重新读取，再次尝试修改就好了。

（6）在硬件层面，大部分的现代处理器都已经支持原子化的CAS指令。在JDK 5.0以后，虚拟机便可以使用这个指令来实现并发操作和并发数据结构，并且，这种操作在虚拟机中可以说是无处不在。





















## 多线程有三大特性

**原子性、可见性、有序性**

### 什么是原子性

即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

一个很经典的例子就是银行账户转账问题： 
 比如从账户A向账户B转1000元，那么必然包括2个操作：从账户A减去1000元，往账户B加上1000元。这2个操作必须要具备原子性才能保证不出现一些意外的问题。

我们操作数据也是如此，比如i = i+1；其中就包括，读取i的值，计算i，写入i。这行代码在[Java](http://lib.csdn.net/base/java)中是不具备原子性的，则多线程运行肯定会出问题，所以也需要我们使用同步和lock这些东西来确保这个特性了。 

原子性其实就是保证数据一致、线程安全一部分，



### 什么是可见性

当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

若两个线程在不同的cpu，那么线程1改变了i的值还没刷新到主存，线程2又使用了i，那么这个i值肯定还是之前的，线程1对变量的修改线程没看到这就是可见性问题。 

### 什么是有序性

程序执行的顺序按照代码的先后顺序执行。

一般来说处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。如下：

int a = 10;  //语句1

int r = 2;  //语句2

a = a + 3;  //语句3

r = a*a;   //语句4

则因为重排序，他还可能执行顺序为 2-1-3-4，1-3-2-4
 但绝不可能 2-1-4-3，因为这打破了依赖关系。
 显然重排序对单线程运行是不会有任何问题，而多线程就不一定了，所以我们在多线程编程时就得考虑这个问题了。



## Java内存模型

共享内存模型指的就是Java内存模型(简称JMM)，**JMM**决定一个线程对共享变量的写入时,能对另一个线程可见**。从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：**线程之间的共享变量存储在主内存（**main memory**）中，每个线程都有一个私有的本地内存（**local memory**），本地内存中存储了该线程以读**/**写共享变量的副本。**本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化。**

![image-20210309163945051](D:\07_个人资料\Java\Java多线程.assets\image-20210309163945051.png)





# 多线程通信



```
public class IntThrad extends Thread {
    private Res res;
    public IntThrad(Res res) {
        this.res = res;
    } 
    @Override
    public void run() {
        int count = 0;
        while (true) {
            if (count == 0) {
                res.userName = "余胜军";
                res.userSex = "男";
            } else {
                res.userName = "小紅";
                res.userSex = "女";
            }
            count = (count + 1) % 2;
        }
    }
}
```

```
public class OutThread extends Thread {
    private Res res;

    public OutThread(Res res) {
        this.res = res;
    }

    @Override
    public void run() {
        while (true) {
            System.out.println(res.userName + "--" + res.userSex);
        }
    }
}
```

```
public class InOutTest {
    public static void main(String[] args) {
        Res res = new Res();
        IntThrad intThrad = new IntThrad(res);
        OutThread outThread = new OutThread(res);
        intThrad.start();
        outThread.start(); 
    }
}
```

小紅--女
余胜军--女
小紅--女
余胜军--男
小紅--女
余胜军--男
余胜军--女
余胜军--男
余胜军--男
小紅--女
小紅--女
余胜军--男
小紅--男

**数据发生错乱，造成线程安全问题**

需要加锁



```
		synchronized (res) {
				if (count == 0) {
					res.userName = "余胜军";
					res.userSex = "男";
				} else {
					res.userName = "小紅";
					res.userSex = "女";
				}
				count = (count + 1) % 2;
			}


synchronized (res) {
				System.out.println(res.userName + "," + res.sex);
			}

```



## wait()、notify、notifyAll()方法

wait()、notify()、notifyAll()是三个定义在Object类里的方法，可以用来控制线程的状态。

这三个方法最终调用的都是jvm级的native方法。随着jvm运行平台的不同可能有些许差异。

 如果对象调用了wait方法就会使持有该对象的线程把该对象的控制权交出去，然后处于等待状态。

如果对象调用了notify方法就会通知某个正在等待这个对象的控制权的线程可以继续运行。

如果对象调用了notifyAll方法就会通知所有等待这个对象控制权的线程继续运行。

**注意:一定要在线程同步中使用,并且是同一个锁的资源**



## wait与sleep区别?

对于sleep()方法，我们首先要知道该方法是属于Thread类中的。而wait()方法，则是属于Object类中的。

sleep()方法导致了程序暂停执行指定的时间，让出cpu该其他线程，但是他的监控状态依然保持者，当指定的时间到了又会自动恢复运行状态。

在调用sleep()方法的过程中，线程不会释放对象锁。

而当调用wait()方法的时候，线程会放弃对象锁，进入等待此对象的等待锁定池，只有针对此对象调用notify()方法后本线程才进入对象锁定池准备

获取对象锁进入运行状态。



在 jdk1.5 之后，并发包中新增了 Lock 接口(以及相关实现类)用来实现锁功能，Lock 接口提供了与 synchronized 关键字类似的同步功能，但需要在使用时手动获取锁和释放锁。



```
Lock lock  = new ReentrantLock();
lock.lock();
try{
//可能会出现线程安全的操作
}finally{
//一定在finally中释放锁
//也不能把获取锁在try中进行，因为有可能在获取锁的时候抛出异常
  lock.ublock();
}

```



## Lock 接口与 synchronized 关键字的区别

Lock 接口可以尝试非阻塞地获取锁 当前线程尝试获取锁。如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁。
 Lock 接口能被中断地获取锁 与 synchronized 不同，获取到锁的线程能够响应中断，当获取到的锁的线程被中断时，中断异常将会被抛出，同时锁会被释放。

Lock 接口在指定的截止时间之前获取锁，如果截止时间到了依旧无法获取锁，则返回。

## Condition用法

 Condition的功能类似于在传统的线程技术中的,Object.wait()和Object.notify()的功能。

```
Condition condition = lock.newCondition();
res. condition.await();  类似wait
res. Condition. Signal() 类似notify

```





## ThreadLoca

##  

 ThreadLocal提高一个线程的局部变量，访问某个线程拥有自己局部变量。

 当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。

ThreadLocal的接口方法

ThreadLocal类接口很简单，只有4个方法，我们先来了解一下：

- void     set(Object value)设置当前线程的线程局部变量的值。

- public     Object get()该方法返回当前线程所对应的线程局部变量。

- public     void remove()将当前线程局部变量的值删除，目的是为了减少内存的占用，该方法是JDK     5.0新增的方法。需要指出的是，当线程结束后，对应该线程的局部变量将自动被垃圾回收，所以显式调用该方法清除线程的局部变量并不是必须的操作，但它可以加快内存回收的速度。

- protected     Object initialValue()返回该线程局部变量的初始值，该方法是一个protected的方法，显然是为了让子类覆盖而设计的。这个方法是一个延迟调用方法，在线程第1次调用get()或set(Object)时才执行，并且仅执行1次。ThreadLocal中的缺省实现直接返回一个null。

## ThreadLoca实现原理

ThreadLoca通过map集合

Map.put(“当前线程”,值)；



```
class ResG {
    public static Integer count = 0;
    public Integer getNum() {
        count = count + 1;
        return count;
    }
}
public class ThreadLocaDemo2 extends Thread {
    private ResG res;
    public ThreadLocaDemo2(ResG res) {
        this.res = res;
    }
    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            System.out.println(Thread.currentThread().getName() + "---" + " num:" + res.getNum());
        }
    }
    public static void main(String[] args) {
        ResG res = new ResG();
        ThreadLocaDemo2 threadLocaDemo1 = new ThreadLocaDemo2(res);
        ThreadLocaDemo2 threadLocaDemo2 = new ThreadLocaDemo2(res);
        ThreadLocaDemo2 threadLocaDemo3 = new ThreadLocaDemo2(res);
        threadLocaDemo1.start();
        threadLocaDemo2.start();
        threadLocaDemo3.start();
    }
}
```

![image-20210309194836410](D:\07_个人资料\Java\Java多线程.assets\image-20210309194836410.png)



使用的是同一个变量，数据不独立

使用ThreadLocal

```
class ResG {
    // 生成序列号共享变量
    public static Integer count = 0;
    public static ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>() {
        protected Integer initialValue() {
            return 0;
        }
    };
    public Integer getNum() {
        count = threadLocal.get() + 1;
        threadLocal.set(count);
        return count;
    }
}
```

![image-20210309195005382](D:\07_个人资料\Java\Java多线程.assets\image-20210309195005382.png)



# java并发包&并发队列



## 并发包

### 同步容器类

#### Vector与ArrayList区别

1.ArrayList是最常用的List实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。数组的缺点是每个元素之间不能有间隔，当数组大小不满足时需要增加存储能力，就要讲已经有数组的数据复制到新的存储空间中。当从ArrayList的中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。

2.Vector与ArrayList一样，**也是通过数组实现的，不同的是它支持线程的同步**，即某一时刻只有一个线程能够写Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此，访问它比访问ArrayList慢

**注意: Vector线程安全、ArrayList ** 

Vector![image-20210310103336464](D:\07_个人资料\Java\Java多线程.assets\image-20210310103336464.png)



ArrayList ![image-20210310103350523](D:\07_个人资料\Java\Java多线程.assets\image-20210310103350523.png)

#### HasTable与HasMap

1.HashMap不是线程安全的 

HastMap是一个接口 是map接口的子接口，是将键映射到值的对象，其中键和值都是对象，并且不能包含重复键，但可以包含重复值。HashMap允许null key和null value，而hashtable不允许。

2.HashTable是线程安全的一个Collection。

3.HashMap是Hashtable的轻量级实现（非线程安全的实现），他们都完成了Map接口，主要区别在于HashMap允许空（null）键值（key）,由于非线程安全，效率上可能高于Hashtable。
 HashMap允许将null作为一个entry的key或者value，而Hashtable不允许。
 HashMap把Hashtable的contains方法去掉了，改成containsvalue和containsKey。

**注意: HashTable线程安全，HashMap线程不安全。**

源码分析

```
HashMap:

public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

```
HashTable

public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }
```



 Collections.synchronized*(m) 将线程不安全额集合变为线程安全集合



#### ConcurrentHashMap

hashtable效率太低了，jdk1.5之后，发明了一种新并发包

ConcurrentMap接口下有俩个重要的实现 :
 ConcurrentHashMap
 ConcurrentskipListMap (支持并发排序功能。弥补ConcurrentHashMap)
 **ConcurrentHashMap**内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个
 小的HashTable,它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并
 发进行。把一个整体分成了16个段(Segment.也就是最高支持16个线程的并发修改操作。
 这也是在重线程场景时**减小锁的粒度从而降低锁竞争**的一种方案。并且代码中大多共享变
 量使用volatile关键字声明，目的是第一时间获取修改的内容，性能非常好。

![image-20210310111219948](D:\07_个人资料\Java\Java多线程.assets\image-20210310111219948.png)

因此并不是不会有锁，而是如果不是同一个组就不是同一个锁，不会竞争。如果多个线程访问的是同一个组那么还是会有锁竞争

#### CountDownLatch  

CountDownLatch类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

```
System.out.println("等待子线程执行完毕...");
final CountDownLatch countDownLatch = new CountDownLatch(2);
new Thread(new Runnable() { 
    @Override
    public void run() {
        System.out.println("子线程," + Thread.currentThread().getName() + "开始执行...");
          try {
                    sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
        countDownLatch.countDown();// 每次减去1
        System.out.println("子线程," + Thread.currentThread().getName() + "结束执行...");
    }
}).start();
new Thread(new Runnable() { 
    @Override
    public void run() {
        System.out.println("子线程," + Thread.currentThread().getName() + "开始执行...");
          try {
                    sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
        countDownLatch.countDown();
        System.out.println("子线程," + Thread.currentThread().getName() + "结束执行...");
    }
}).start();

countDownLatch.await();// 调用当前方法主线程阻塞  countDown结果为0, 阻塞变为运行状态
System.out.println("两个子线程执行完毕....");
System.out.println("继续主线程执行..");
```

等待子线程执行完毕...
子线程,Thread-0开始执行...
子线程,Thread-0结束执行...
子线程,Thread-1开始执行...
子线程,Thread-1结束执行...
两个子线程执行完毕....
继续主线程执行..

Process finished with exit code 0



#### CyclicBarrier

CyclicBarrier初始化时规定一个数目，然后计算调用了CyclicBarrier.await()进入等待的线程数。当线程数达到了这个数目时，所有进入等待状态的线程被唤醒并继续。 

 CyclicBarrier就象它名字的意思一样，可看成是个障碍， 所有的线程必须到齐后才能一起通过这个障碍。 

CyclicBarrier初始时还可带一个Runnable的参数， 此Runnable任务在CyclicBarrier的数目达到后，所有其它线程被唤醒前被执行。

```
  private CyclicBarrier cyclicBarrier;
    public Writer(CyclicBarrier cyclicBarrier) {
        this.cyclicBarrier = cyclicBarrier;
    } 
    @Override
    public void run() {
        System.out.println("线程" + Thread.currentThread().getName() + ",正在写入数据");
        try {
            Thread.sleep(3000);
        } catch (Exception e) {
            // TODO: handle exception
        }
        System.out.println("线程" + Thread.currentThread().getName() + ",写入数据成功....."); 
        try {
            cyclicBarrier.await();
        } catch (Exception e) {
        }
        System.out.println("所有线程执行完毕..........");
    } 
} 
public static void main(String[] args) {
    CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
    for (int i = 0; i < 5; i++) {
        Writer writer = new Writer(cyclicBarrier);
        writer.start();
    } 
}
```

```
线程Thread-2,正在写入数据
线程Thread-3,正在写入数据
线程Thread-0,正在写入数据
线程Thread-1,正在写入数据
线程Thread-4,正在写入数据
线程Thread-2,写入数据成功.....
线程Thread-3,写入数据成功.....
线程Thread-0,写入数据成功.....
线程Thread-1,写入数据成功.....
线程Thread-4,写入数据成功.....
所有线程执行完毕..........
所有线程执行完毕..........
所有线程执行完毕..........
所有线程执行完毕..........
所有线程执行完毕..........

Process finished with exit code 0
```

#### Semaphore

Semaphore是一种基于计数的信号量。它可以设定一个阈值，基于此，多个线程竞争获取许可信号，做自己的申请后归还，超过阈值后，线程申请许可信号将会被阻塞。Semaphore可以用来构建一些对象池，资源池之类的，比如数据库连接池，我们也可以创建计数为1的Semaphore，将其作为一种类似互斥锁的机制，这也叫二元信号量，表示两种互斥状态。它的用法如下：

availablePermits函数用来获取当前可用的资源数量

wc.acquire(); //申请资源

wc.release();// 释放资源

需求: 一个厕所只有3个坑位，但是有10个人来上厕所，那怎么办？假设10的人的编号分别为1-10，并且1号先到厕所，10号最后到厕所。那么1-3号来的时候必然有可用坑位，顺利如厕，4号来的时候需要看看前面3人是否有人出来了，如果有人出来，进去，否则等待。同样的道理，4-10号也需要等待正在上厕所的人出来后才能进去，并且谁先进去这得看等待的人是否有素质，是否能遵守先来先上的规则。

代码:

```
class Parent implements Runnable {
    private String name;
    private Semaphore wc;
    public Parent(String name,Semaphore wc){
        this.name=name;
        this.wc=wc;
    }
    @Override
    public void run() {
        try {
            // 剩下的资源(剩下的茅坑)
            int availablePermits = wc.availablePermits();
            if (availablePermits > 0) {
                System.out.println(name+"天助我也,终于有茅坑了...");
            } else {
                System.out.println(name+"怎么没有茅坑了...");
            }
            //申请茅坑 如果资源达到3次，就等待
            wc.acquire();
            System.out.println(name+"终于轮我上厕所了..爽啊");
            Thread.sleep(new Random().nextInt(1000)); // 模拟上厕所时间。
            System.out.println(name+"厕所上完了...");
            wc.release();
        } catch (Exception e) {
        }
    }
}
public class TestSemaphore {
    public static void main(String[] args) {
         Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <=5; i++) {
            Parent parent = new Parent("第"+i+"个人,",semaphore);
            new Thread(parent).start();
        }
    } 
}
```

```
第1个人,天助我也,终于有茅坑了...
第2个人,天助我也,终于有茅坑了...
第2个人,终于轮我上厕所了..爽啊
第1个人,终于轮我上厕所了..爽啊
第3个人,天助我也,终于有茅坑了...
第3个人,终于轮我上厕所了..爽啊
第4个人,怎么没有茅坑了...
第5个人,怎么没有茅坑了...
第3个人,厕所上完了...
第4个人,终于轮我上厕所了..爽啊
第2个人,厕所上完了...
第5个人,终于轮我上厕所了..爽啊
第1个人,厕所上完了...
第4个人,厕所上完了...
第5个人,厕所上完了...

Process finished with exit code 0
```

### 并发队列

在并发队列上JDK提供了两套实现，一个是以ConcurrentLinkedQueue为代表的高性能队

列，一个是以BlockingQueue接口为代表的阻塞队列，无论哪种都继承自Queue。

#### ConcurrentLinkedDeque

ConcurrentLinkedQueue : 是一个适用于高并发场景下的队列，通过无锁的方式，实现
 了高并发状态下的高性能，通常ConcurrentLinkedQueue性能好于BlockingQueue.它
 是一个基于链接节点的无界线程安全队列。该队列的元素遵循先进先出的原则。头是最先
 加入的，尾是最近加入的，该队列不允许null元素。
 ConcurrentLinkedQueue重要方法:
 add 和offer() 都是加入元素的方法(在ConcurrentLinkedQueue中这俩个方法没有任何区别)
 poll() 和peek() 都是取头元素节点，区别在于前者会删除元素，后者不会。

```
ConcurrentLinkedDeque q = new ConcurrentLinkedDeque();
q.offer("余胜军");
q.offer("码云");
q.offer("蚂蚁课堂");
q.offer("张杰");
q.offer("艾姐");
//从头获取元素,删除该元素
System.out.println(q.poll());
System.out.println(q.size());
//从头获取元素,不刪除该元素
System.out.println(q.peek());
System.out.println(q.peek());
//获取总长度
System.out.println(q.size());
```

余胜军
4
码云
码云
4



#### BlockingQueue

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：

**在队列为空时，获取元素的线程会等待队列变为非空。**
**当队列满时，存储元素的线程会等待队列可用。** 

阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

BlockingQueue即阻塞队列，从阻塞这个词可以看出，在某些情况下对阻塞队列的访问可能会造成阻塞。被阻塞的情况主要有如下两种：

**\1. 当队列满了的时候进行入队列操作**

**\2. 当队列空了的时候进行出队列操作**

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

####  ArrayBlockingQueue

ArrayBlockingQueue是一个**有边界的阻塞队列**，它的内部实现是一个**数组**。有边界的意思是它的容量是有限的，我们必须在其初始化的时候指定它的容量大小，容量大小一旦指定就不可改变。

ArrayBlockingQueue是以先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部。下面

#### LinkedBlockingQueue

LinkedBlockingQueue阻塞队列大小的**配置是可选的**，如果我们初始化时指定一个大小，它就是有边界的，如果不指定，它就是无边界的。说是无边界，其实是采用了默认大小为Integer.MAX_VALUE的容量 。它的内部实现是一个**链表**。

和ArrayBlockingQueue一样，LinkedBlockingQueue 也是以先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部。



#### PriorityBlockingQueue

PriorityBlockingQueue是一个没有边界的队列，它的排序规则和 java.util.PriorityQueue一样。需要注意，PriorityBlockingQueue中允许插入null对象。

所有插入PriorityBlockingQueue的对象必须实现 java.lang.Comparable接口，队列优先级的排序规则就是按照我们对这个接口的实现来定义的。

另外，我们可以从PriorityBlockingQueue获得一个迭代器Iterator，但这个迭代器并不保证按照优先级顺序进行迭代。

#### SynchronousQueue 

**SynchronousQueue**队列内部仅允许容纳一个元素。当一个线程插入一个元素后会被阻塞，除非这个元素被另一个线程消费。





# 线程池

Java中的线程池是运用场景最多的并发框架，几乎所有需要异步或并发执行任务的程序
 都可以使用线程池。在开发过程中，合理地使用线程池能够带来3个好处。
 第一：降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
 第二：提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。执行完上一个流程，立马可以执行到下一个执行流程，不需要进入销毁，创建，等待流程
 第三：提高线程的可管理性。线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，
 还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控。但是，要做到合理利用
 线程池，必须对其实现原理了如指掌。

**执行完之后不会进入销毁流程**，也避免了创建流程，这些都是消耗资源的



线程池是为突然大量爆发的线程设计的，通过有限的几个固定线程为大量的操作服务，减少了创建和销毁线程所需的时间，从而提高效率。

如果一个线程的时间非常长，就没必要用线程池了(不是不能作长时间操作，而是不宜。)，况且我们还不能控制线程池中线程的开始、挂起、和中止。

Executor框架的最顶层实现是ThreadPoolExecutor类，Executors工厂类中提供的newScheduledThreadPool、newFixedThreadPool、newCachedThreadPool方法其实也只是ThreadPoolExecutor的构造函数参数不同而已。通过传入不同的参数，就可以构造出适用于不同应用场景下的线程池，那么它的底层原理是怎样实现的呢，这篇就来介绍下ThreadPoolExecutor线程池的运行过程。 

corePoolSize： 核心池的大小。 当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中
 maximumPoolSize： 线程池最大线程数，它表示在线程池中最多能创建多少个线程；
 keepAliveTime： 表示线程没有任务执行时最多保持多久时间会终止。
 unit： 参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：

## 线程池四种创建方式

Java通过Executors（jdk1.5并发包）提供四种线程池，分别为：
 newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。

案例演示:
 newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
 newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
 newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

```
   // 无限大小线程池 jvm自动回收
        ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();
        for (int i = 0; i < 100; i++) {
            final int temp = i;
            newCachedThreadPool.execute(new Runnable() {
                @Override
                public void run() {
//                    try {
//                        Thread.sleep(100);
//                    } catch (Exception e) {
//                        // TODO: handle exception
//                    }
                    System.out.println(Thread.currentThread().getName() + ",i:" + temp);
                }
            });
        }
```

**注释掉sleep发现只会创建30几个线程（具体数量应该跟系统有关系），加上sleep就创建100个了。因为前面的线程没有执行完，未释放出来，所以只能创建新的线程。**

```
ExecutorService newFixedThreadPool = Executors.newFixedThreadPool(3);
for (int i = 0; i < 10; i++) {
    final int temp = i;
    newFixedThreadPool.execute(new Runnable() {

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + ",i:" + temp);

        }
    });
}
创建3个线程，其余等待
```

```
ScheduledExecutorService newScheduledThreadPool = Executors.newScheduledThreadPool(3);
for (int i = 0; i < 10; i++) {
    final int temp = i;
    newScheduledThreadPool.schedule(new Runnable() {
        public void run() {
            System.out.println(Thread.currentThread().getName()+"  i:" + temp);
            try {
                sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }, 3, TimeUnit.SECONDS);
}
```

可定时，3秒之后开始执行。 

```
ExecutorService newSingleThreadExecutor = Executors.newSingleThreadExecutor();
for (int i = 0; i < 10; i++) {
    final int index = i;
    newSingleThreadExecutor.execute(new Runnable() {

        @Override
        public void run() {
            System.out.println("index:" + index);
            try {
                Thread.sleep(200);
            } catch (Exception e) {
                // TODO: handle exception
            }
        }
    });
}
```







## 线程池原理剖析

提交一个任务到线程池中，线程池的处理流程如下：

1、判断线程池里的核心线程是否都在执行任务，如果不是（核心线程空闲或者还有核心线程没有被创建）则创建一个新的工作线程来执行任务。如果核心线程都在执行任务，则进入下个流程。

2、线程池判断工作队列是否已满，如果工作队列没有满，则将新提交的任务存储在这个工作队列里。如果工作队列满了，则进入下个流程。

3、判断线程池里的线程是否都处于工作状态，如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务。

![img](D:\07_个人资料\Java\Java多线程.assets\clip_image002.jpg)

如果核心线程池没有满就加到核心队列中执行

如果满了就加到核心线程等待队列池中，等待执行。

如果等待队列也满了就加到 最大线程池中执行

如果最大线程池中也满了就会触发 拒绝执行 

![image-20210310185000373](D:\07_个人资料\Java\Java多线程.assets\image-20210310185000373.png)



默认最大队列未int最大值



## 合理配置线程池

要想合理的配置线程池，就必须首先分析任务特性，可以从以下几个角度来进行分析：

任务的性质：CPU密集型任务，IO密集型任务和混合型任务。

任务的优先级：高，中和低。

任务的执行时间：长，中和短。

任务的依赖性：是否依赖其他系统资源，如数据库连接。

任务性质不同的任务可以用不同规模的线程池分开处理。**CPU密集型任务配置尽可能少的线程数量**，如配置Ncpu+1个线程的线程池。**IO密集型任务则由于需要等待IO操作，线程并不是一直在执行任务，则配置尽可能多的线程，如2*Ncpu**。混合型的任务，如果可以拆分，则将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐率要高于串行执行的吞吐率，如果这两个任务执行时间相差太大，则没必要进行分解。我们可以通过Runtime.getRuntime().availableProcessors()方法获得当前设备的CPU个数。

优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先得到执行，需要注意的是如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。

执行时间不同的任务可以交给不同规模的线程池来处理，或者也可以使用优先级队列，让执行时间短的任务先执行。

**依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，如果等待的时间越长CPU空闲时间就越长，那么线程数应该设置越大，这样才能更好的利用CPU。**

一般总结哦，有其他更好的方式，希望各位留言，谢谢。

 

CPU密集型时，任务可以少配置线程数，大概和机器的cpu核数相当，这样可以使得每个线程都在执行任务

IO密集型时，大部分线程都阻塞，故需要多配置线程数，2*cpu核数

操作系统之名称解释：

某些进程花费了绝大多数时间在计算上，而其他则在等待I/O上花费了大多是时间，

前者称为计算密集型（CPU密集型）computer-bound，后者称为I/O密集型，I/O-bound。

