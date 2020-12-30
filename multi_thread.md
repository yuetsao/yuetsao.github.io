# MULTI_THREAD
## 基本概念
* 进程
<br/>异常是允许操作系统内核提供进程（process）概念的基本构造块，进程是计算机科学中最深刻、最成功的概念之一。
<br/>在现代系统上运行一个程序时，我们会得到一个假象，就好像我们的程序是系统中当前运行的唯一的程序一样。我们的程序
好像是独占地使用处理器和内存。处理器就好像是无间断地一条接一条地指向我们程序中的指令。最后，我们程序中的代码和数据
好像是系统内存中的唯一对象。这些假象都是通过继承的概念提供给我们的。
<br/>进程的经典定义就是一个***执行中程序的实例***。系统中的每个程序都运行在某个进程的***上下文(context***)中。上下文
是由程序正确运行所需要的状态组成的。这个状态包括存放在内存中的程序的代码和数据，它的栈、通用目的寄存器的内容、程序
计数器、环境变量以及打开文件描述符的集合。
<br/>每次用户通过向shell输入一个可执行目标文件的名字，运行程序时，shell就会创建一个新的进程，然后在这个新进程的上下文中运行这个可
执行目标文件。应用程序也能够创建新进程，并且在这个新进程的上下文运行它们自己的代码或其他应用程序。
<br/>关键抽象
<br/> * 一个独立的逻辑控制流，它提供一个假象，好像我们的程序独占地使用处理器。
<br/> * 一个私有的地址空间，它提供一个假象，好像我们的程序独占地使用内存系统。
* 线程
<br/>作为进程里面最小的执行单元叫做线程，用简单的话来说一个程序例不同的执行路径。
每个线程都运行在进程的上下文中，并共享同样的代码和全局数据。由于网络服务器中对并行处理的要求，线程称为越来越重要的编程
模型，因为多线程之间比多进程之间更容易共享数据，也因为线程一般来说都比进程要更高效。当有多处理器可用的时候，多线程也是一种使
得程序可以运行得更快的方法。
* 纤程
<br/>线程的英文Fibers
<br/>A Fiber is a lightweight thread that uses cooperative multitasking instead of preemptive multitasking. A running fiber must explicitly "yield" to allow another fiber to run, which makes their implementation much easier than kernel or user threads.
<br/>A Coroutine is a component that generalizes a subroutine to allow multiple entry points for suspending and resuming execution at certain locations. Unlike subroutines, coroutines can exit by calling other coroutines, which may later return to the point where they were invoked in the original coroutine.
<br/>A Green Thread is a thread that is scheduled by a virtual machine (VM) instead of natively by the underlying operating system. Green threads emulate multithreaded environments without relying on any native OS capabilities, and they are managed in user space instead of kernel space, enabling them to work in environments that do not have native thread support.
<br/>

### 示例 什么叫线程

``` java
public class CreateThread {
    static class Mythread extends Thread {
        @Override
        public void run() {
            System.out.println("Thread running!");
        }
    }
    
    static class MyRun implements Runnable {

        @Override
        public void run() {
            System.out.println("Run running!");
        }
    }

    public static void main(String[] args) {
        new Mythread().start();
        new Thread(new MyRun()).start();
        new Thread(()-> {
            System.out.println("Hello lambda!"); 
        }).start();
    }
    
}

```

### JMH & Disruptor

1. what is JMH?
* JMH- java Microbenchmark Harness
* 2013 年首发 由JIT的开发人员开发 归于OpenJDK
* Disruptor是英国外汇交易公司LMAX开发的一个高性能队列，研发的初衷是解决内存队列的延迟问题（在性能测试中发现竟然与I/O操作处于同样的数量级）。基于Disruptor开发的系统单线程能支撑每秒600万订单，2010年在QCon演讲后，获得了业界关注。2011年，企业应用软件专家Martin Fowler专门撰写长文介绍。同年它还获得了Oracle官方的Duke大奖。目前，包括Apache Storm、Camel、Log4j 2在内的很多知名项目都应用了Disruptor以获取高性能。在美团技术团队它也有不少应用，有的项目架构借鉴了它的设计机制。本文从实战角度剖析了Disruptor的实现原理。需要特别指出的是，这里所说的队列是系统内部的内存队列，而不是Kafka这样的分布式队列。另外，本文所描述的Disruptor特性限于3.3.4。 
### java内置队列
![java内置队列](https://tva1.sinaimg.cn/large/0081Kckwly1gln529bo6gj30n706ymy1.jpg)
队列对应的数据结构分为：数组，链表和堆。其中，堆一般情况下是为了实现带有优先级特性的队列，暂且不考虑。
<br/> 基于数组线程安全的队列，比较典型的是ArryBlockingQueue，它主要通过加锁的方式来保证线程安全；基于连标的线程安全队列分成LinkedBlockingQueue和ConcurrentLinkedQueue两大类，前者通过锁的方式来实现线程安全，而后者以及上面表格中的LinedTransferQueue都是通过原子变量compare and swap这种不加锁的方式来实现的。
<br/> 通过不枷锁的方式实现的队列都是无界的（无法保证队列的长度在确定的范围内）；而加锁的方式可以实现有界队列。在稳定性要求特别高的系统，为了防止生产者速度过快，只能选择有界队列；同时为了减少java的垃圾回收性能的影响，会尽量选择array/heap格式的数据结构。这样筛选下来，符合条件的队列只有ArryBlockingQueue。
### ArrayBlockingQueue的问题
ArrayBlockingQueue在实际使用过程中，会因为加锁和伪共享等出现严重的性能问题。
### 加锁
现实编程过程中，加锁通常会严重地影响性能。线程会因为竞争不到锁而被挂起，等锁被释放的时候，线程又会被恢复，这个过程中存在着很大的开销，并且通常会有较长时间的中断，因为当一个线程正在等待锁时，它不能做任何其他事情。如果一个线程在持有锁的情况下被延迟执行，例如发生了缺页错误、调度延迟或者其它类似情况，那么所有需要这个锁的线程都无法执行下去。如果被阻塞线程的优先级较高，而持有锁的线程优先级较低，就会发生优先级反转。
<br/> 单线程情况下：不加锁性能>CAS操作性能>加锁的性能。
<br/> 多线程情况下：CAS>加锁，前者大约是后者的8倍。
<br/> 综上所述，加锁的性能是最差的。
### 关于锁和CAS
``` java
public boolean offer(E e) {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if(count == items.length)
	    return false;
	else {
	    insert (e);
	    return true;
	}
    } finally {
        lock.unlock();
    }
}
```
### 原子变量
原子变量能够保证原子性操作，意思是某个任务在执行的过程中，要么全部成功，要们全部失败回滚，恢复到执行之前的状态，不存在初态和成功之间的中间状态。例如CAS操作，要么别比较并交换成功，要么比较交换失败。由CPU保证原子性。
<br/>通过原子变量可以实现线程安全。执行某个任务的时候，先假设不会由冲突，则直接执行成功；当发生冲突的时候，则执行失败，回滚再重新操作，直到不发生冲突。
代码示例是AtomicInteger的getAndAdd方法。CAS是CPU的一个指令，由CPU保证原子性。
``` java
/**
 * Atomically adds the given value to the current value.
 *
 * @param delta the value to add
 * @return the previous value
 */
public final int getAndAdd(int delta) {
    for (;;) {
        int current = get();
        int next = current + delta;
        if (compareAndSet(current, next))
            return current;
    }
}
  
/**
 * Atomically sets the value to the given updated value
 * if the current value {@code ==} the expected value.
 *
 * @param expect the expected value
 * @param update the new value
 * @return true if successful. False return indicates that
 * the actual value was not equal to the expected value.
 */
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```
在高度竞争情况下，锁的性能将超过原子变量的性能，但是更真是的竞争情况下，原子变量的性能将超过锁的性能。同时原子变量不会由死锁等活跃性问题。
### 伪共享
![计算机CPU与缓存示意图](https://tva1.sinaimg.cn/large/0081Kckwly1glndiv9u8pj30su10igoh.jpg)
当cpu执行运算的时候，它先去L1查找所需要的数据，再去L2，然后是L3，如果最后这些缓存中都没有，所需要的数据就要去主内存中拿。走得越远，运算耗费的时间越长。所以如果你在做一些很繁琐的事，你要尽量确保数据在L1缓存中。另外，线程之间共享一份数据的时候，需要一个线程把数据写回主存，而另一个线程访问主存中相应的数据。
<br/>下面是从CPU访问不同层级数据的时间概念：
![](https://tva1.sinaimg.cn/large/0081Kckwly1glndre69loj313w070wfa.jpg)
可见CPU读取主存中的数据会比从L1中读取慢了将近2个数量级。
### 缓存行
Cache是由很多哥cache line组成的。每个cache line通常是64字节，并且它通常是64字节，并且它有效的引用主存中的一块儿地址。一个java的long类型变量是8字节，因此在一个缓存行中可以存8哥long类型的变量。
<br/>CPU每次从主存中拉去数据时，会把相邻的数据也存入同一个cache line。
<br/>在访问一个long数组的时候，如果数组中的一个值被加载到缓存中，他会自动加载另外7个。因此你能非常快的遍历这个数组。事实上，你可以非常快速的遍历在连续内存块中分配的任意数据结构。
``` java
package com.meituan.FalseSharing;
 
/**
 * @author gongming
 * @description
 * @date 16/6/4
 */
public class CacheLineEffect {
    //考虑一般缓存行大小是64字节，一个 long 类型占8字节
    static  long[][] arr;
 
    public static void main(String[] args) {
        arr = new long[1024 * 1024][];
        for (int i = 0; i < 1024 * 1024; i++) {
            arr[i] = new long[8];
            for (int j = 0; j < 8; j++) {
                arr[i][j] = 0L;
            }
        }
        long sum = 0L;
        long marked = System.currentTimeMillis();
        for (int i = 0; i < 1024 * 1024; i+=1) {
            for(int j =0; j< 8;j++){
                sum = arr[i][j];
            }
        }
        System.out.println("Loop times:" + (System.currentTimeMillis() - marked) + "ms");
 
        marked = System.currentTimeMillis();
        for (int i = 0; i < 8; i+=1) {
            for(int j =0; j< 1024 * 1024;j++){
                sum = arr[j][i];
            }
        }
        System.out.println("Loop times:" + (System.currentTimeMillis() - marked) + "ms");
    }
}
```
### 什么是伪共享
ArrayBlockingQueue有三个成员变量：-takeIndex：需要被取走的下标元素，-putIndex可被元素插入的位置的下标-count：队列重元素的数量 这三个变量很容易放到一个缓存行中，但是之间修改没有太多的关联。所以每次修改，都会使之前缓存的数据失效，从而不能完全达到共享的效果。
![](https://tva1.sinaimg.cn/large/0081Kckwly1glnfojt642j31im0i6mzo.jpg)
如上图所示，当生产者线程put一个元素到ArrayBlockingQueue时，putIndex会修改，从而导致消费者线程的缓存中的缓存行无效，需要从主存中重新读取。
<br/>这种无法充分使用缓存行特性的现象，称为伪共享。
<br/>对于伪共享，一般的解决方案是，增大数组元素的间隔使得由不同线程存取的元素位于不同的缓存行上，以空间换时间。
``` java
package com.meituan.FalseSharing;
 
public class FalseSharing implements Runnable{
        public final static long ITERATIONS = 500L * 1000L * 100L;
        private int arrayIndex = 0;
 
        private static ValuePadding[] longs;
        public FalseSharing(final int arrayIndex) {
            this.arrayIndex = arrayIndex;
        }
 
        public static void main(final String[] args) throws Exception {
            for(int i=1;i<10;i++){
                System.gc();
                final long start = System.currentTimeMillis();
                runTest(i);
                System.out.println("Thread num "+i+" duration = " + (System.currentTimeMillis() - start));
            }
 
        }
 
        private static void runTest(int NUM_THREADS) throws InterruptedException {
            Thread[] threads = new Thread[NUM_THREADS];
            longs = new ValuePadding[NUM_THREADS];
            for (int i = 0; i < longs.length; i++) {
                longs[i] = new ValuePadding();
            }
            for (int i = 0; i < threads.length; i++) {
                threads[i] = new Thread(new FalseSharing(i));
            }
 
            for (Thread t : threads) {
                t.start();
            }
 
            for (Thread t : threads) {
                t.join();
            }
        }
 
        public void run() {
            long i = ITERATIONS + 1;
            while (0 != --i) {
                longs[arrayIndex].value = 0L;
            }
        }
 
        public final static class ValuePadding {
            protected long p1, p2, p3, p4, p5, p6, p7;
            protected volatile long value = 0L;
            protected long p9, p10, p11, p12, p13, p14;
            protected long p15;
        }
        public final static class ValueNoPadding {
            // protected long p1, p2, p3, p4, p5, p6, p7;
            protected volatile long value = 0L;
            // protected long p9, p10, p11, p12, p13, p14, p15;
        }
}
```
<br/>备注：在jdk1.8中，由专门的注解@Contended来避免伪共享，更优雅的解决问题。
### Disruptor通过一下设计来解决队列速度慢的问题：
* 环形数组结构
<br/> 为了避免垃圾回收，采用数组而非链表。同时，数组对处理器的缓存机制更加友好。
* 元素位置定位
<br/> 数组长度2^n,通过位运算，加速定位的速度。下标采取递增的形式。不用担心index溢出的问题。index是lang类型，即使100万QPS的处理速度，也要30万年才能用完。
* 无锁设计
<br/> 每个生产者或者消费者线程，会先申请可以操作的元素在数组中的位置，申请之后，直接在该位置写入或者读取数据。
<br/> 下面忽略数组的环形结构，介绍一下如何实现无锁设计。整个过程通过原子变量CAS，保证操作的线程安全。
### 一个生产者
#### 写数据
<br/>生产者线程写数据的流程比较简单：
<br/> 1.申请写入m个元素

## 参考

<div id="refer-anchor-1"></div>

- [1] [深入理解计算机系统]()

<div id="refer-anchor-2"></div>

- [2] [马士兵多线程与高并发]()

<div id="refer-anchor-2"></div>

- [3] [stackoverflow](https://softwareengineering.stackexchange.com/questions/254140/is-there-a-difference-between-fibers-coroutines-and-green-threads-and-if-that-i)
<div ide="refer=anchor-2"></div>

- [4] [美团技术团队](https://tech.meituan.com/2016/11/18/disruptor.html)

