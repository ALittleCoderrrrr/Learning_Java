# Learning_Java_03

## Java 基础

### 1. 匿名类

匿名类（Anonymous Class）是Java中的一种特殊类，它没有显式的类名，而是在需要的地方直接定义并实例化。匿名类通常用于创建一次性的、不需要单独命名的类对象，通常在代码中的局部位置进行定义和使用。

```java
父类或接口 变量名 = new 父类或接口() {
    // 匿名类的成员和方法
}

接口 变量名 = ()->{

};  //使用Lambda表达式创建匿名类：要求必须是函数式接口
```



## Java 并发

### 1. 线程池

#### 1.1 使用线程池的好处：

降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗；

提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行；

提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。


#### 1.2 线程池的创建

**通过ThreadPoolExecutor类创建：**

ThreadPoolExecutor是ExecutorService接口的一个实现，它提供了更多的配置选项，允许你更精细地控制线程池的行为。通过ThreadPoolExecutor构造函数可以指定核心线程数、最大线程数、任务队列等参数。

```java
int corePoolSize = 5;
int maxPoolSize = 10;
long keepAliveTime = 60L;
TimeUnit unit = TimeUnit.SECONDS;
BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>();

ExecutorService executor = new ThreadPoolExecutor(corePoolSize, maxPoolSize, keepAliveTime, unit, workQueue);
```

**通过Executors工厂类创建：**

Executors工厂类提供了一些静态方法来创建不同类型的线程池。以下是一些常用的Executors方法：
```java
Executors.newFixedThreadPool(int n)
\\创建一个固定大小的线程池，该线程池包含指定数量的线程。
Executors.newCachedThreadPool()
\\创建一个根据需要创建新线程的线程池，线程池的大小可自动调整。
Executors.newSingleThreadExecutor()
\\创建一个单线程的线程池，任务按顺序执行。
Executors.newScheduledThreadPool(int n)
\\创建一个定时执行任务的线程池。
```

一般不推荐使用Executors来创建线程池，因为其内部也是调用ThreadPoolExecutor来创建线程池，但是创建FixedThreadPool()和SinleThreadExecutor()时，内部使用无界队列LinkedBlockingQueue可能会导致任务大量堆积内存溢出，其他情况也类似。

#### 1.3 ThreadPoolExecutor的参数
**corePoolSize: 核心线程数**

含义：线程池中保持活跃的最小线程数。即使线程处于空闲状态，也不会被销毁，除非线程池关闭。

**maximumPoolSize: 最大线程数**

含义：线程池中允许创建的最大线程数。当队列满了并且核心线程都在执行任务时，线程池可以创建额外的线程来处理任务，但不会超过这个最大值。

**keepAliveTime: 空闲线程存活时间**

含义：当线程池中的线程数量超过核心线程数，并且这些线程处于空闲状态时，多余的线程会被销毁，直到线程池中的线程数量不超过核心线程数，这个时间就是空闲线程的最大存活时间。

**unit: 存活时间的时间单位**
含义：用于定义keepAliveTime的时间单位，通常为TimeUnit.SECONDS等。

**workQueue: 任务队列**
含义：用于存储等待执行的任务的队列，可以是LinkedBlockingQueue、ArrayBlockingQueue等不同类型的队列。线程池会根据情况将任务放入队列或创建
新线程执行任务。

**threadFactory: 线程工厂**
含义：用于创建新线程的工厂，通常使用默认的线程工厂即可。

**handler: 拒绝策略**
含义：当任务无法被执行时（通常是因为线程池已满且任务队列也满了），用于处理这些被拒绝的任务。可以使用内置的拒绝策略，如
**AbortPolicy**:这是默认的拒绝策略，当线程池无法接受新任务时，它会抛出RejectedExecutionException异常，表示拒绝执行新任务。这种策略会导致任务丢失，因为新任务不会被执行；

**CallerRunsPolicy**:在这个策略下，当线程池无法接受新任务时，会使用提交任务的线程来执行被拒绝的任务。这意味着任务不会被丢弃，但可能会由调用线程来执行，从而降低了并行性；

**DiscardPolicy**:这个策略简单地丢弃掉无法执行的新任务，不提供任何错误处理。如果你不关心任务的执行结果或者可以容忍一些任务的丢失，可以选择这个策略；

**DiscardOldestPolicy**：这个策略会丢弃掉等待时间最长的任务，然后尝试将新任务加入队列。通常用于控制队列长度，尽量保留最新的任务。

**如何选取线程数量：**

CPU 密集型任务(N+1)： 这种任务消耗的主要是 CPU 资源，可以将线程数设置为 N（CPU 核心数）+1。比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。

I/O 密集型任务(2N)： 这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 2N。

**线程池常用的阻塞队列**

容量为 Integer.MAX_VALUE 的 LinkedBlockingQueue（无界队列）：FixedThreadPool 和 SingleThreadExector 。由于队列永远不会被放满，因此FixedThreadPool最多只能创建核心线程数的线程。

SynchronousQueue（同步队列）：CachedThreadPool 。SynchronousQueue 没有容量，不存储元素，目的是保证对于提交的任务，如果有空闲线程，则使用空闲线程来处理；否则新建一个线程来处理任务。也就是说，CachedThreadPool 的最大线程数是 Integer.MAX_VALUE ，可以理解为线程数是可以无限扩展的，可能会创建大量线程，从而导致 OOM。

DelayedWorkQueue（延迟阻塞队列）：ScheduledThreadPool 和 SingleThreadScheduledExecutor 。DelayedWorkQueue 的内部元素并不是按照放入的时间排序，而是会按照延迟的时间长短对任务进行排序，内部采用的是“堆”的数据结构，可以保证每次出队的任务都是当前队列中执行时间最靠前的。DelayedWorkQueue 添加元素满了之后会自动扩容原来容量的 1/2，即永远不会阻塞，最大扩容可达 Integer.MAX_VALUE，所以最多只能创建核心线程数的线程。



#### 1.4 线程池处理任务的流程
<div style="text-align:center;">
处理流程
<br>
<img src="./day03_picture/0.png"  width="300" height="200">
</div>

<br>

#### 1.5 线程池的使用

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolExample {
    public static void main(String[] args) {
        // 创建一个固定大小的线程池
        ExecutorService executorService = Executors.newFixedThreadPool(3);

        // 提交任务给线程池
        for (int i = 0; i < 5; i++) {
            final int taskNumber = i;
            executorService.submit(new Runnable() {
                @Override
                public void run() {
                    System.out.println("Task " + taskNumber + " is running.");
                }
            });
        }

        // 关闭线程池
        executorService.shutdown();
    }
}
```

submit()接收一个实现runnable或者callable接口对象。

#### 1.6 runnable和callable

在 Java 中，Runnable 和 Callable 都是用于多线程编程的接口，用于定义可以由线程执行的任务。它们之间有以下区别：

返回值：

Runnable 接口没有返回值，它的 run 方法用于执行任务，但不返回结果。
Callable 接口有返回值，它的 call 方法用于执行任务，并返回一个结果。

异常抛出：

Runnable 接口的 run 方法不能抛出受检查异常，只能抛出未受检查异常（运行时异常）。
Callable 接口的 call 方法可以抛出受检查异常。

使用场景：

Runnable 通常用于无需返回结果的任务，如简单的线程执行、定时任务等。
Callable 通常用于需要返回结果并且可能抛出受检查异常的任务，如线程池中的任务，任务执行后可以通过 Future 对象获取结果。

Future：

Runnable 任务无法直接获取执行结果。如果需要等待任务完成并获取结果，通常需要使用 Runnable 任务包装在 Callable 中，然后由线程池执行。

Callable 任务可以通过 Future 对象获取执行结果，可以轻松实现异步任务的结果获取和异常处理。

```java
import java.util.concurrent.*;

public class ParallelExecutionExample {

    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(2);

        Callable<Integer> task1 = () -> {
            // 一些计算任务
            return 42;
        };

        Callable<String> task2 = () -> {
            // 另一个计算任务
            return "Hello, Callable!";
        };

        Future<Integer> future1 = executor.submit(task1);
        Future<String> future2 = executor.submit(task2);

        Integer result1 = future1.get();
        String result2 = future2.get();

        System.out.println("Result 1: " + result1);
        System.out.println("Result 2: " + result2);

        executor.shutdown();
    }
}
```

1.7 Semaphore（基于AQS）

synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，而Semaphore(信号量)可以用来控制同时访问特定资源的线程数量。Semaphore 的使用简单，我们这里假设有 N(N>5) 个线程来获取 Semaphore 中的共享资源，下面的代码表示同一时刻 N 个线程中只有 5 个线程能获取到共享资源，其他线程都会阻塞，只有获取到共享资源的线程才能执行。等到有线程释放了共享资源，其他阻塞的线程才能获取到.

```java
// 初始共享资源数量
final Semaphore semaphore = new Semaphore(5);
// 获取1个许可
semaphore.acquire();
// 释放1个许可
semaphore.release();
```

使用Semaphore来实现三个进程依次打印A\B\C
```java
package com.demo0;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.StreamTokenizer;
import java.util.Arrays;
import java.util.Enumeration;
import java.util.HashSet;
import java.util.Set;
import java.util.concurrent.Semaphore;

public class Main {
    public static void main(String[] args) throws IOException {
        Semaphore semaphoreA = new Semaphore(1);
        Semaphore semaphoreB = new Semaphore(0);
        Semaphore semaphoreC = new Semaphore(0);

        Thread threadA = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    semaphoreA.acquire();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("A");
                semaphoreB.release();
            }
        });

        Thread threadB = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    semaphoreB.acquire();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("B");
                semaphoreC.release();
            }
        });

        Thread threadC = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    semaphoreC.acquire();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("C");
                semaphoreA.release();
            }
        });

        threadA.start();
        threadB.start();
        threadC.start();
    }
}
```

#### 1.8 CountDownLatch（基于AQS）

CountDownLatch 是 Java 中并发编程的一个工具类，它提供了一种等待多个线程完成任务的机制。CountDownLatch 的核心思想是：一个线程等待其他多个线程完成某个任务，等待的线程会阻塞直到计数器减为零。

CountDownLatch 的主要方法包括：

CountDownLatch(int count)：构造方法，传入一个正整数 count 作为计数器的初始值。

void await()：调用该方法的线程会阻塞，直到计数器的值减为零。如果计数器的值不为零，那么调用 await() 的线程就会一直等待。

boolean await(long timeout, TimeUnit unit)：带有超时参数的等待方法，它允许在等待一段时间后超时返回，如果超时返回 false，否则返回 true。

void countDown()：计数器减一，表示一个任务已经完成。

```java
import java.util.concurrent.CountDownLatch;

public class Main {
    public static void main(String[] args) throws InterruptedException {
        int numberOfThreads = 5;
        CountDownLatch latch = new CountDownLatch(numberOfThreads);

        for (int i = 0; i < numberOfThreads; i++) {
            Thread thread = new Thread(() -> {
                // 模拟线程执行任务
                System.out.println("Thread " + Thread.currentThread().getId() + " is working.");
                latch.countDown(); // 任务完成，计数器减一
            });
            thread.start();
        }

        latch.await(); // 等待所有线程完成任务，当计数器减为零时，继续执行
        System.out.println("All threads have finished their work.");
    }
}
```

#### submit() 和 execute()
execute()方法用于提交不需要返回值的任务,所以无法判断任务是否被线程池执行成功与否；

submit()方法用于提交需要返回值的任务。线程池会返回一个 Future 类型的对象，通过这个 Future 对象可以判断任务是否执行成功isDone()方法，并且可以通过 Future 的 get()方法来获取返回值，get()方法会阻塞当前线程直到任务完成，而使用 get（long timeout，TimeUnit unit）方法的话，如果在 timeout 时间内任务还没有执行完，就会抛出 java.util.concurrent.TimeoutException。
