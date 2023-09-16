# Learning_Java_02
## Java基础

### 1. 集合
#### 1.1 Java集合底层框架
<div style="text-align:center;">
Java 集合框架
<br>
<img src="./day02_picture/0.png"  width="300" height="200">
</div>

<br>

***List 接口实现类***：

ArrayList: 内部基于动态扩容数组实现，线程不安全；

LinkedList: 内部使用双向链表实现，线程不安全；

Vector：主要是为了多线程设计使用的，线程安全，现在已经被淘汰了；

<br>

***Set 接口实现类***：

HashSet：基于HashMap实现，内部元素无序，线程不安全；

LinkedHashSet：基于LinkedHashSet实现，链表+HashMap，仅支持按插入顺序排序，不支持按查询顺序排序（没有提供对应的接口）。

TreeSet：基于红黑树实现，插入元素按key值排序。修改和查询时间复杂度均为O(logn)。


<br>

***Queue实现类***

PriorityQueue:使用数组来实现堆，可重写Comparator接口实现自定义的排序规则.

ArrayQueue:底层使用数组+双指针实现FIFO，只能从一端插入元素从另一端删除元素.

<br>

***Map实现类***

HashMap：***1***.使用拉链法解决冲突。当拉链长度大于8时，先判断桶长度是否小于64，如果是则优先扩充桶的大小，否则将拉链转为红黑树；***2***. 支持NULL类型的key和value; HashMap默认容量是16，每次扩容时，容量都会变为原来的两倍,如果指定初始容量则会将其扩大为2的次幂。使用2的次幂可以仅利用按位与操作快速确定元素所在的桶序号；***3***. 线程不安全：同时往一个桶插入元素会导致冲突;为了线程安全的实现HashMap，提供了Collections.synchornizedMap(map)的包装方法，该方法使用synchornized关键字对原始方法进行包装。

HashTable：线程安全，内部方法都适用synchronized进行修饰。效率较低，不支持NULL类型的key和value，因为在多线程的情况下，存在二义性问题例如get()方法，key不存在时会返回NULL，但又能是是value本来就是NULL；HashTable默认容量为11，每次扩容为原来的2倍+1。如果指定容量则会直接使用给的容量。

LinkedHashMap：支持按插入顺序和按查询顺序访问，在创建时制定排序标志即可实现。

TreeMap：元素按序排列。


## Java 并发
### 1. 并发容器 java.util.concurrent
#### 1.1 ConcurrentHashMap

线程安全，其底层使用CAS技术结合synchronized技术来实现写入数据：当要放入的桶没有元素时，尝试利用CAS技术写入；如果已经有元素了,则使用synchronized写入并在满足条件下转为红黑树。

#### 1.2 CopyOnWriteArrayList

抛弃了之前的Vector，因为它使用了synchronized技术耗时较长。现在使用的唯一线程安全的List就是CopyOnWriteArrayList。其设计思路在于：每次涉及到修改操作如增、删、改，都会先加ReentranLock锁，将原数据复制一份副本，等在副本上修改完后再将其复制回原数组，然后释放锁。而读操作时则不会加锁，不过可能导致读到旧值。

#### 1.3 ConcurrentLinkedQueue

线程安全的非阻塞Queue，使用CAS非阻塞算法来实现线程安全，性能较高。

#### 1.4 BlockingQueue

提供了可阻塞的插入和移除方法，被广泛应用于“生产者-消费者”场景。其实现类有ArrayBlockingQueue:是基于数组实现的，因此创建时必须指定容量，一旦创建成功就不能再改变容量，默认情况下是非公平访问(内部使用ReentrantLock实现)；LinkedBlockingQueue：基于链表实现，若创建时未指定容量，则按Integer.MAX_VALUE处理、PriorityBlockingQueue：PriorityQueue的线程安全版本；

### 2. 线程
#### 2.1 线程创建
***实现Runnable接口，重写run方法：*** 
```java
Thread thread = new Thread(()->{
        // 线程的执行代码
});
thread.start(); // 启动线程
```
***继承Thread类,重写run方法:***
```java
class MyThread extends Thread {
    public void run() {
        // 线程的执行代码
    }
}

public static void main(String[] args) {
    MyThread thread = new MyThread();
    thread.start(); // 启动线程
}

```

#### 2.2 线程同步

***2.2.1 synchronized 关键字***

使用对象锁实现同步非静态方法：
```java
public synchronized void synchronizedMethod() {
    // 这是一个同步方法
}
```
一旦有线程进入同步函数，其他线程就不能再进入同一个对象的任何同步函数。因为该锁是对象级别的。

使用类锁实现同步静态方法：
```java
public class MyClass {
    public static synchronized void staticSynchronizedMethod() {
        // 这是一个同步的静态方法
    }
}
```
它会锁定MyClass.class对象，与该类对象的锁互不影响，互不阻塞可以并发执行。

使用自定义对象锁实现同步代码块：
``` java
public void someMethod() {
    // 非同步代码
    synchronized (lockObject) {
        // 需要同步的代码
    }
    // 非同步代码
}
```

synchronized 底层实现的本质就是对象监视器monitor。

***wait()/notify()方法***

使用方法如下：
```java
public class WaitNotifyExample {
    public static void main(String[] args) {
        final Object lock = new Object();
        
        Thread producer = new Thread(() -> {
            synchronized (lock) {
                System.out.println("Producer: Producing data...");
                // 模拟生产过程
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Producer: Data ready.");
                lock.notify(); // 唤醒等待的消费者线程
            }
        });

        Thread consumer = new Thread(() -> {
            synchronized (lock) {
                System.out.println("Consumer: Waiting for data...");
                try {
                    lock.wait(); // 等待生产者的通知
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Consumer: Data received.");
            }
        });

        producer.start();
        consumer.start();
    }
}
```

注意使用wait()或者notify()方法必须先要获得锁。notify()只会随机唤醒一个正在wait()的线程，而notifyAll()则会唤醒所有线程。

***锁升级***

synchronized 原本是重量级锁，在Java6以后对其进行优化，引入了锁升级概念：

在进入同步代码块之前，对象处于**无锁状态**；线程进入同步代码块之后，对象锁变为**偏向锁**，在该锁状态下，同一个线程进出同步代码块无需进行加、解锁操作；当有其他线程进入时，锁状态切换为**轻量级锁**，线程们尝试使用CAS操作来获取锁，如果获取不到就自旋而非阻塞；当自旋次数超过一定阈值，锁就变为**重量级锁**，这时获取不到锁的线程直接被阻塞，而不是自旋消耗CPU资源。注意：锁一旦升级便不会再降级。

***synchronized优缺点***

优点：简单易用，无需导包；

缺点：性能较低，功能简单无法完成复杂的并发控制。

***2.2.2 ReentrantLock***

实现了java.util.concurrent.lock接口，提供更强大的并发控制功能如：轮训、超时、中断、公平锁、非公平锁。

**公平/非公平锁：**
```java
    ReentrantLock fairLock = new ReentrantLock(true); //公平锁

    ReentrantLock noFairLock = new ReentrantLock(true); //非公平锁
```

**等待可中断**

```java
public class InterruptibleWaitDemo {
    public static void main(String[] args) {
        final Object lock = new Object();

        Thread waitingThread = new Thread(() -> {
            synchronized (lock) {
                try {
                    System.out.println("线程开始等待...");
                    lock.wait(); // 线程等待，可中断
                    System.out.println("线程被唤醒！");
                } catch (InterruptedException e) {
                    System.out.println("线程被中断！");
                }
            }
        });

        Thread interruptingThread = new Thread(() -> {
            try {
                Thread.sleep(1000); // 睡眠一秒钟
                waitingThread.interrupt(); // 中断等待线程
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        waitingThread.start();
        interruptingThread.start();
    }
}
```
在等待锁的过程中，可以被中断等待，而synchronized则不可以被中断等待。


**尝试获取锁**
```java
ReentrantLock lock = new ReentrantLock();

        Thread thread1 = new Thread(() -> {
            if (lock.tryLock()) {
                try {
                    System.out.println("线程1获取到锁");
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                    System.out.println("线程1释放锁");
                }
            } else {
                System.out.println("线程1未获取到锁");
            }
        });
```
如果可以获取到锁就会获取，如果不可获取则会立即返回，避免被长期阻塞。

**配合Conditon 实现不同条件下唤醒不同进程**
``` java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ProducerConsumerExample {
    public static void main(String[] args) {
        Buffer buffer = new Buffer(5); // 缓冲区大小为5

        Thread producer = new Thread(new Producer(buffer));
        Thread consumer = new Thread(new Consumer(buffer));

        producer.start();
        consumer.start();
    }
}

class Buffer {
    private Queue<Integer> queue;
    private int capacity;
    private Lock lock = new ReentrantLock();
    private Condition notFull = lock.newCondition();
    private Condition notEmpty = lock.newCondition();

    public Buffer(int capacity) {
        this.capacity = capacity;
        this.queue = new LinkedList<>();
    }

    public void produce(int item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await();
            }
            queue.offer(item);
            System.out.println("Produced: " + item);
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public int consume() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await();
            }
            int item = queue.poll();
            System.out.println("Consumed: " + item);
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}

class Producer implements Runnable {
    private Buffer buffer;

    public Producer(Buffer buffer) {
        this.buffer = buffer;
    }

    @Override
    public void run() {
        try {
            for (int i = 0; i < 5; i++) {
                buffer.produce(i);
                Thread.sleep(100);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Consumer implements Runnable {
    private Buffer buffer;

    public Consumer(Buffer buffer) {
        this.buffer = buffer;
    }

    @Override
    public void run() {
        try {
            for (int i = 0; i < 5; i++) {
                buffer.consume();
                Thread.sleep(200);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

#### 2.2.3  Atomic 原子类

juc.atomic包下提供了原子类型的基本类型、数组类型、引用类型的
类和有关方法，支持使用原子操作更新或设置值，底层利用CAS操作来实现。

### 2.3 锁
#### 3.1 悲观锁
认为并发执行一定会出问题，所以在访问临界资源时必须先加锁阻塞。例如synchronized,ReentrantLock的设计思路便是如此。在高并发的情况下，多线程会造成激烈的锁竞争，频繁的切换上下文影响执行速度。
#### 3.2 乐观锁
认为不会出问题，所以先执行线程，如果发生冲突了再回退。具体可以使用版本机制、或者CAS算法实现。例如juc.atomic包下的原子类就是使用CAS方式实现的。它不会造成锁竞争，但是如果冲突频繁发生也会影响CPU性能。
#### 3.3 CAS
CAS算法是乐观锁的一种实现，它先读入要更新的变量值V，并将之赋给本线程的E变量，将修改后的值记作N。等要写回N时，先拿本线程的E变量与内存的V进行比较，如果相等则写回N并告知成功。否则将被告知失败。Java并没有直接实现CAS，可以使用sun.misc包下的Unsafe类提供的compareAndSwapObject、compareAndSwapInt、compareAndSwapLong方法来实现的对Object、int、long类型的 CAS 操作。

CAS存在三大问题：ABA、循环时间开销大（自旋）、只能对单个共享变量有效。


### 2.4 ThreadLocl
该类型的变量会自动在每个访问的线程中都保留一个副本，线程可以调用threadLocal.get() 或者 threadLocal.set()来获取值或者修改值,使用方法如下：
```java
public class ThreadLocalExample {
    //将变量初始值设置为“Default Value”,每个线程的值都会是这个
    private static ThreadLocal<String> threadLocal = ThreadLocal.withInitial(() -> "Default Value");

    public static void main(String[] args) {
        // 在主线程中设置 ThreadLocal 变量的值
        threadLocal.set("Main Thread Value");
        
        // 创建一个新线程，并在其中访问 ThreadLocal 变量
        Thread thread = new Thread(() -> {
            System.out.println("ThreadLocal Value in Child Thread: " + threadLocal.get());
        });
        thread.start();
        
        // 在主线程中访问 ThreadLocal 变量
        System.out.println("ThreadLocal Value in Main Thread: " + threadLocal.get());
    }
}

```
底层原理：每个线程内部维护一个ThreadLocalMap，其记录了ThreadLocal变量的弱引用和其对应的值。

ThreadLocal为什么会内存泄漏？

ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用来引用它，那么系统 GC 的时候，这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value永远无法回收，造成内存泄漏。

其实，ThreadLocalMap的设计中已经考虑到这种情况，也加上了一些防护措施：在ThreadLocal的get(),set(),remove()的时候都会清除线程ThreadLocalMap里所有key为null的value。

但是这些被动的预防措施并不能保证不会内存泄漏：

使用static的ThreadLocal，延长了ThreadLocal的生命周期，可能导致的内存泄漏（参考ThreadLocal 内存泄露的实例分析）。
分配使用了ThreadLocal又不再调用get(),set(),remove()方法，那么就会导致内存泄漏。
