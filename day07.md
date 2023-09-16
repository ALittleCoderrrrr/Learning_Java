# Learning_Java_07

## Java 基础
### 1. Java 成员变量初始化顺序
**无父类：**

    静态变量初始零值->静态变量显式赋值->静态代码块->实例变量零值->构造代码块赋值->构造函数
**有父类：**

    父类静态变量初始零值 -> 父类静态变量显式赋值 -> 父类静态代码块赋值 -> 子类静态变量初始零值 -> 子类静态变量显式赋值 -> 子类静态代码块赋值 -> （子类和父类）实例变量默认零值 -> 父类构造代码块赋值 -> 父类构造函数赋值 -> 子类构造代码块赋值 -> 子类构造函数赋值

### 2.深拷贝、浅拷贝
**浅拷贝：**
    类实现Cloneable 接口，重写父类clone函数：return super.clone()；
**深拷贝：**
    方法1:每个引用类属性都重写clone()函数;然后递归调用clone();
    方法2:利用序列化实现深拷贝
```java
import java.io.*;

class MyObject implements Serializable {
    public int value;

    public MyObject(int value) {
        this.value = value;
    }
}

public class DeepCopyExample {
    public static void main(String[] args) throws Exception {
        // 创建一个包含自定义对象的对象
        MyObject original = new MyObject(42);

        // 使用序列化进行深拷贝
        MyObject deepCopy = deepCopy(original);

        // 修改原始对象的值
        original.value = 10;

        // 打印深拷贝对象的值，它应该保持不变
        System.out.println("Deep Copy Value: " + deepCopy.value);
    }

    public static <T> T deepCopy(T object) throws Exception {
        // 创建字节数组输出流和输入流
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream out = new ObjectOutputStream(bos);

        // 将对象写入输出流
        out.writeObject(object);
        out.flush();
        out.close();

        // 从字节数组输入流中读取对象
        ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream in = new ObjectInputStream(bis);

        // 返回深拷贝的对象
        return (T) in.readObject();
    }
}
```

### 3. ArrayList扩容
以无参构造创建的ArrayList容量为0，以带参数构造时容量为参数大小。当向容量为零的List添加元素时，才会分配大小为10的Object数组。当添加元素超过容量时，容量会自动扩充为newCapacity = oldCapacity+oldCapacity>>2;当元素数量超过Integer.MAX_VALUE时，会报错OOM。扩容时会调用Arrays.copyOf()把原数据复制到新数组当中。

### 4. Arrays.sort()和Collections.sort()
**使用Compareable来实现排序：**

两种排序函数会使用反射来查找待排序类是否实现了Comparable接口，如果实现了就寻找待排序类中的CompareTo()函数，根据该函数定义的排序规则来实现排序。如果对象不实现 Comparable 接口，或者没有提供用于比较的 compareTo 方法，那么在尝试使用 Arrays.sort() 进行排序时将抛出 ClassCastException 异常。

```java
import java.util.Arrays;

class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public int compareTo(Person o) {
        // 比较两个 Person 对象的年龄
        return this.age - o.age;
    }

    @Override
    public String toString() {
        return name + " (" + age + " years old)";
    }
}

public class Main {
    public static void main(String[] args) {
        Person[] people = {
            new Person("Alice", 25),
            new Person("Bob", 30),
            new Person("Eve", 22)
        };

        // 使用 Arrays.sort() 进行排序
        Arrays.sort(people);

        // 打印排序后的结果
        for (Person person : people) {
            System.out.println(person);
        }
    }
}
```

**使用Comparator来实现排序：**

通过在调用sort函数时实现comparator接口，来实现自定义排序规则，避免了在类中的改动。

``` java
ArrayList<Integer> arrayList = new ArrayList<Integer>();
arrayList.add(-1);
arrayList.add(3);
arrayList.add(3);
arrayList.add(-5);

// void sort(List list),按自然排序的升序排序
Collections.sort(arrayList);
System.out.println("Collections.sort(arrayList):");
System.out.println(arrayList);
// 定制排序的用法
Collections.sort(arrayList, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o2.compareTo(o1);
    }
});
System.out.println("定制排序后：");
System.out.println(arrayList);
```

### 5. toString()重写
作用：重写toString()可以在调用System.out.println(objeect)时，输出
toString()重写后的结果。

### 6. Deque 双端队列的实现，PriorityQueue
**ArrayDeque：**

    底层使用变长数组+双指针实现，存在扩容过程，但是均摊以后时间复杂度依旧是O(1)。

**LinkedList：**

    底层使用链表实现，每次插入时都需要申请堆空间，运行较慢。

**PriorityQueue:**

    利用二叉堆实现的，底层使用变长数组来存储元素，插入和删除元素的时间复杂度都是O(logN)。不支持存储NULL和non-comparable的对象（没有实现comparable接口），不过在声明时可以实现Comparator接口来指定对比方案。

**BlockingQueue（线程安全Queue，常用于生产者消费者模型）：**

    ArrayBlockingQueue: 底层基于Object数组实现，创建时必须指定容量（有界）。

```java
    BlockingQueue bq = new ArrayBlockingQueue<>(1);
    bq.put(1);   //put满了就会阻塞，而add则会抛出异常。此时还不会阻塞
    bp.put(2);   //阻塞线程，知道有其他线程调用take()

    //bp.take()； 其他线程调用take。才能把元素2放入。
```

    ArrayBlockingQueue 内部有ReentrantLock,除此之外还声明了对应的Condition notFull,notEmpty()变量用来实现生产者和消费者功能；
    代码如下：

```java
import java.util.*;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    public static void main(String[] args) throws InterruptedException {
        ReentrantLock lock = new ReentrantLock(true);
        Condition notFull = lock.newCondition();
        Condition notEmpty = lock.newCondition();
        int capacity = 10;
        List<Integer> list = new ArrayList<>();

        Thread producer = new Thread(()->{
            for(int i=0;i<10;i++) {
                lock.lock();
                try {
                    while (list.size() == capacity) {
                        notFull.await();
                    }
                    list.add(i);
                    notEmpty.signal();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        });


        Thread consumer = new Thread(()->{
            for(int i=0;i<10;i++) {
                lock.lock();
                try {
                    while (list.size() == 0) {
                        notEmpty.await();
                    }
                    System.out.println(list.get(0));
                    list.remove(0);
                    notFull.signal();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        });

        producer.start();
        consumer.start();
    }
}
```

    LinkedBlockingQueue: 底层基于链表实现，创建时不指定容量就是无界的，指定容量就是有界的,他和ArrayBlockingQueue的底层采用同样的实现方式。

    SynchronousQueue:同步队列，是一种不存储元素的阻塞队列。每个插入操作都必须等待对应的删除操作，反之删除操作也必须等待插入操作。因此，SynchronousQueue通常用于线程之间的直接传递数据.使用方式如下：
```java
import java.util.concurrent.SynchronousQueue;

public class SynchronousQueueExample {
    public static void main(String[] args) {
        // 创建一个 SynchronousQueue
        SynchronousQueue<Integer> synchronousQueue = new SynchronousQueue<>();

        // 创建一个生产者线程
        Thread producer = new Thread(() -> {
            try {
                int data = 42;
                System.out.println("Producer is producing data: " + data);
                synchronousQueue.put(data); 
                // 插入数据并等待消费者取走
                System.out.println("Producer finished");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        // 创建一个消费者线程
        Thread consumer = new Thread(() -> {
            try {
                System.out.println("Consumer is waiting for data");
                int data = synchronousQueue.take(); 
                // 等待生产者插入数据
                System.out.println("Consumer received data: " + data);
                System.out.println("Consumer finished");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        // 启动生产者和消费者线程
        producer.start();
        consumer.start();
    }
}

```

    DelayQueue: 延迟队列，其中的元素只有到了其指定的延迟时间，才能够从队列中出队。


### MAP
**重写hashcode()和Equals：**
```java
import java.util.Objects;

public class MyClass {
    private int id;
    private String name;

    public MyClass(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // Getters and setters for id and name (not shown for brevity).

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        MyClass myClass = (MyClass) o;
        return id == myClass.id && Objects.equals(name, myClass.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }

```

**HashMap put过程：**
    根据计算key的hash扰动函数，使其碰撞可能性进一步减少。
```java
    static final int hash(Object key) {
      int h;
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

当桶位置没有元素时，直接放入。这里n为桶的容量，大小为2的次幂。
（n-1）& hash 就相当于取了hash的低位以使其不溢出。

```java
  if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
```
确保桶的容量始终是不小于给定值的2次幂，其原理是每次将一些位的值设为1，给一个32位的数，只有最高位是1其他都是0，在执行完该操作后全部也能变成1：
```java
   static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

该方法并没有采取传统的取余数运算，而是直接利用按位与计算出桶的索引，速度更快。这也解释了为什么桶的容量要为2的次幂了。

**HashTable 和ConcurrentHashMap：**
    HashTable线程安全，底层采用synchronized关键字对每个函数加锁，性能较差，不支持null key和 null value因为会出现二义性问题。
    扩容时容量会变为原来的2n+1倍，并没有采用像HashMap求Index的操作。
    同时也没有采用红黑树的机制。

    ConcurrentHashMap底层实现:
    1.8以前是分段加锁，1.8以后采用Node数组+链表+红黑树实现，与HashMap在数据结构上保持一致。
    在具体操作上添加了一些同步机制，例如创建表时：
```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        //　如果 sizeCtl < 0 ,说明另外的线程执行CAS 成功，正在进行初始化。
        if ((sc = sizeCtl) < 0)
            // 让出 CPU 使用权
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```
涉及到了自旋、compareAndSwapInt(Object o, long offset,int expected,int x)操作，该操作是一个unsafe类中的一个原子操作，o为要更新的对象，offset为要更新变量在这个对象内存中的偏移量，expected为期望值，x为更新值。

put键值对时：
    当计算出桶位置为空时，直接使用casTabAT()的操作放入元素；当不为空时，使用synchronized，对当前桶元素加锁，依次遍历链表选择添加元素或者更新元素。当为红黑树时，调用红黑树的处理方式来处理。

get操作：不加任何锁，高并发环境下性能很好。

### CopyOnWriteArrayList（线程安全，Vector）
    读读不互斥、读写不互斥、写写互斥；
    
    底层实现：get操作不加锁，add操作加ReentrantLock，先拷贝一份副本，在副本上进行写操作，然后丢弃原来的Object数组，将引用指向新数组。便可以不会影响读操作。






