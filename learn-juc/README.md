# JUC介绍
java.util.concurrent包下的工具类

# 回顾
>**进程和线程的区别**
- 进程：一个程序，QQ.exe等程序的集合。
- 一个进程往往可以包含多个线程，至少包含一个。
- java进程默认有两个线程，一个是main线程，另一个是gc线程。
- java可以开启线程的三个方式：Thread，Runnable，Callable
- java创建线程的本质是通过native方法进行创建，而非java本身创建
  
>**并发和并行的区别**
- 并发：多个线程操作同一个资源，CPU单核，多线程进行切换，模拟出多线程的作用。
- 并发编程的本质：充分利用CPU的资源。
- 并行：多核CPU，多个线程同时执行。
  - ```Runtime.getRuntime().availableProcessors()```获得CPU核数。

>**线程的几个状态**

线程从状态由```Thread.State```这个枚举变量所决定，如下：
```java
public enum State {
    NEW,                //新生
    RUNNABLE,           //运行
    BLOCKED,            //阻塞
    WAITING,            //等待（死等）
    TIMED_WAITING,      //超时等待
    TERMINATED;         //终止
}
```

>**wait和sleep的区别**
- 来自不同的类
  - wait来自于Object类
  - sleep来自于Thread类
- 关于锁的释放
  - wait是释放锁
  - sleep不会释放锁（抱着锁睡觉）
- 使用的范围不同
  - wait必须在同步代码块内中。
  - sleep可以在任何地方进行

>**Lock锁**

测试代码(不加锁)：
```java
/**
 * 线程就是一个单独的资源类，没有任何的附属操作
 */
public class Ticket {
    private int num = 20;

    public void sale() {
        if (num > 0) {
            System.out.println(Thread.currentThread().getName() + "卖出了第" + (num--) + "票，剩余" + num);
        }
    }
}

public class SaleTicket {
    public static void main(String[] args) {
        //并发：多线程操作同一个资源类，把资源类丢入线程
        Ticket ticket = new Ticket();

        new Thread(() -> {
            for (int i = 0; i < 20; i++) {
                ticket.sale();
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 20; i++) {
                ticket.sale();
            }
        }).start();
        new Thread(() -> {
            for (int i = 0; i < 20; i++) {
                ticket.sale();
            }
        }).start();
    }
}
```
运行结果如下（有问题，出现并发编程的问题）：

![img.png](https://i.niupic.com/images/2020/10/06/8LQn.png)

**如果对sale函数加上Synchronized关键字进行同步就没有问题：**

![img.png](https://i.niupic.com/images/2020/10/06/8LQo.png)

**如果使用Lock进行操作如下：**
```java
//1.new一个锁
private Lock lock = new ReentrantLock();

public void sale() {
    try {
        //2.业务前加锁
        lock.lock();
        if (num > 0) {
            System.out.println(Thread.currentThread().getName() + "卖出了第" + (num--) + "票，剩余" + num);
        }
    } catch (Exception ignored) {
        
    } finally {
        //3.业务结束时解锁
        lock.unlock();
    }
}
```
运行结果如下（也是正常的）：

![img.png](https://i.niupic.com/images/2020/10/06/8LQt.png)

>**Lock与Synchronized的区别**

1. Synchronized是java内置的关键字，而Lock是一个java类
2. Synchronized无法判断获取锁的状态，Lock可以判断是否获取到了锁
3. Synchronized会自动释放锁，Lock必须手动释放锁，如果不释放锁会造成死锁
4. Synchronized线程1（获得锁，阻塞），线程2是一直等待的，而Lock锁有tryLock()方法
5. Synchronized是可重入锁，不可以中断的，非公平；而Lock是可重入锁，可以判断锁，默认非公平锁（可设置）。
6. Synchronized适合少量代码块同步问题，Lock锁适合大量代码块的同步。

>锁是什么，如何判断锁的是谁？

# 生产者消费者问题
## Synchronized版（wait和notify配合）
代码如下：
```java
public class SynchronizedPC {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(()->{
            try {
                for(int i=0;i<5;i++)
                    data.get();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(()->{
            try {
                for(int i=0;i<5;i++)
                    data.put();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

    }
}

class Data {
    private int num = 0;

    public synchronized void get() throws InterruptedException {
        if (num == 0) {
            //等待
            this.wait();
        }
        num--;
        System.out.println(Thread.currentThread().getName() + "\tget\t" + num);
        //通知其他线程
        this.notifyAll();
    }

    public synchronized void put() throws InterruptedException {
        if (num != 0) {
            //等待
            this.wait();
        }
        num++;
        System.out.println(Thread.currentThread().getName() + "\tput\t" + num);
        //通知其他线程
        this.notifyAll();
    }
}
```
运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/06/8LQx.png)

**问题存在：如果有多于2个线程的话就会出问题**

如下为4个线程的运行结果：

![img.png](https://i.niupic.com/images/2020/10/06/8LQB.png)

官方文档内解释到原因可能是虚假唤醒的操作，改进时改为while来判断即可。

![img.png](https://i.niupic.com/images/2020/10/06/8LQA.png)

改进代码如下：
```java
public synchronized void get() throws InterruptedException {
    while (num == 0) {
        //等待
        this.wait();
    }
    num--;
    System.out.println(Thread.currentThread().getName() + "\tget\t" + num);
    //通知其他线程
    this.notifyAll();
}

public synchronized void put() throws InterruptedException {
    while (num != 0) {
        //等待
        this.wait();
    }
    num++;
    System.out.println(Thread.currentThread().getName() + "\tput\t" + num);
    //通知其他线程
    this.notifyAll();
}
```
## JUC版本
配套使用的是```Condition.await()```和```Condition.signal()```方法来进行。

代码如下：
```java
class Data2 {
    private int num = 0;

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void get() throws InterruptedException {
        lock.lock();
        try {
            while (num == 0) {
                //等待
                condition.await();
            }
            num--;
            System.out.println(Thread.currentThread().getName() + "\tget\t" + num);
            //通知其他线程
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void put() throws InterruptedException {
        lock.lock();
        try {
            while (num != 0) {
                //等待
                condition.await();
            }
            num++;
            System.out.println(Thread.currentThread().getName() + "\tput\t" + num);
            //通知其他线程
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }
}
```
运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/06/8LQD.png)

可能存在的问题：运行状态是随机的，希望有顺序进行执行。

**Condition的优势：精准的进行通知唤醒**

代码如下：
```java
//A->B->C->A...进行操作
public class ConditionTest {
    public static void main(String[] args) {
        Data3 data3 = new Data3();
        new Thread(() -> {
            for (int i = 0; i < 10; i++)
                data3.printA();
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++)
                data3.printB();
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++)
                data3.printC();
        }, "C").start();
    }
}

class Data3 {
    private Lock lock = new ReentrantLock();
    //3个条件
    private Condition conditionA = lock.newCondition();
    private Condition conditionB = lock.newCondition();
    private Condition conditionC = lock.newCondition();

    private int num = 1;

    public void printA() {
        lock.lock();
        try {
            //等待
            while (num != 1) {
                conditionA.await();
            }
            System.out.println(Thread.currentThread().getName() + "执行");
            //通知B
            num = 2;
            conditionB.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() {
        lock.lock();
        try {
            //等待
            while (num != 2) {
                conditionB.await();
            }
            System.out.println(Thread.currentThread().getName() + "执行");
            //通知C
            num = 3;
            conditionC.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC() {
        lock.lock();
        try {
            //等待
            while (num != 3) {
                conditionC.await();
            }
            System.out.println(Thread.currentThread().getName() + "执行");
            //通知A
            num = 1;
            conditionA.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```
运行结果如下图：

![img.png](https://i.niupic.com/images/2020/10/06/8LQQ.png)

# 8锁问题
**8锁，关于锁的8个问题**

1. 标准情况下是先发短信还是先打电话
```java
public class Lock1 {
    public static void main(String[] args) {
        Phone1 phone = new Phone1();
        new Thread(phone::sendSms, "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(phone::call, "A").start();
    }
}

class Phone1 {
    public synchronized void sendSms() {
        System.out.println("sendSms");
    }

    public synchronized void call() {
        System.out.println("call");
    }
}
```
执行的先后顺序是先发短信后打电话，如下图所示：

![img.png](https://i.niupic.com/images/2020/10/06/8LS6.png)

2. 该情况下是先发短信还是先打电话（发短信线程先延迟4s）
```java
public class Lock2 {
    public static void main(String[] args) {
        Phone2 phone = new Phone2();
        new Thread(phone::sendSms, "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(phone::call, "A").start();
    }
}

class Phone2 {
    public synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("sendSms");
    }

    public synchronized void call() {
        System.out.println("call");
    }
}
```
运行结果：还是先发短信后打电话，如下图所示：

![img.png](https://i.niupic.com/images/2020/10/06/8LS8.png)


**上面两个问题说明：synchronized锁的对象是方法的调用者，两个方法用的是同一个锁，谁先拿到，谁先执行。**
  
3. 一个方法有synchronized修饰，另一个为普通方法
```java
public class Lock3 {
    public static void main(String[] args) {
        Phone3 phone = new Phone3();
        new Thread(phone::sendSms, "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(phone::hello, "A").start();
    }
}

class Phone3 {
    public synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("sendSms");
    }

    public void hello() {
        System.out.println("hello");
    }
}
```
输出结果是先输出hello，再输出sendSms，如下图所示：

![img.png](https://i.niupic.com/images/2020/10/06/8LSc.png)

因为普通方法并没有涉及到锁的问题所在，由于延迟的原因就先执行hello方法。

4. 两个对象两个同步方法是先执行发短信还是先打电话
```java
public class Lock4 {
    public static void main(String[] args) {
        Phone4 phone1 = new Phone4();
        Phone4 phone2 = new Phone4();
        
        new Thread(phone1::sendSms, "A").start();//一个先发短信
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(phone2::call, "A").start();//另一个打电话
    }
}

class Phone4 {
    public synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("sendSms");
    }

    public synchronized void call() {
        System.out.println("call");
    }
}
```
运行结果是先打电话，后发短信，运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/06/8LSg.png)

因为synchronized锁的是两个对象，先来先到。

5. 两个静态的同步方法，一个对象
```java
public class Lock5 {
    public static void main(String[] args) {
        Phone5 phone = new Phone5();
        new Thread(() -> phone.sendSms(), "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> phone.call(), "A").start();
    }
}

class Phone5 {
    public static synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("sendSms");
    }

    public static synchronized void call() {
        System.out.println("call");
    }
}
```
运行结果是先发短信，后打电话，运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/06/8LSi.png)

synchronized锁住静态方法时是锁住整个class，即锁住所有该class的对象，即如果是两个不同的对象，即类似4的情况下运行结果也是相同的，如下6。

6. 锁住静态方法，两个对象。
```java
public class Lock6 {
    public static void main(String[] args) {
        Phone6 phone1 = new Phone6();
        Phone6 phone2 = new Phone6();

        new Thread(() -> phone1.sendSms(), "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> phone2.call(), "A").start();
    }
}

class Phone6 {
    public static synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("sendSms");
    }

    public static synchronized void call() {
        System.out.println("call");
    }
}
```
运行结果同5，先发短信，后打电话，运行结果如下图所示：

![img.png](https://i.niupic.com/images/2020/10/06/8LSk.png)

原因同5，synchronized锁住静态方法时是锁住整个class，即锁住所有该class的对象。

7. 一个锁住静态方法，另一个锁住普通方法，一个对象。
```java
public class Lock7 {
    public static void main(String[] args) {
        Phone7 phone = new Phone7();
        new Thread(() -> phone.sendSms(), "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> phone.call(), "A").start();
    }
}

class Phone7 {
    public static synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("sendSms");
    }

    public synchronized void call() {
        System.out.println("call");
    }
}
```
运行结果是先打电话，后发短信，如下图所示：

![img.png](https://i.niupic.com/images/2020/10/06/8LSm.png)

因为一个是锁住Class类模板，另一个是锁住类对象，是两个不同的锁，所以打电话无需等待发短信进行。

8. 一个锁住静态方法，另一个锁住普通方法，两个对象。
```java
public class Lock8 {
    public static void main(String[] args) {
        Phone8 phone1 = new Phone8();
        Phone8 phone2 = new Phone8();

        new Thread(() -> phone1.sendSms(), "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> phone2.call(), "A").start();
    }
}

class Phone8 {
    public static synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("sendSms");
    }

    public synchronized void call() {
        System.out.println("call");
    }
}
```
运行结果依然是先打电话后发短信，如下图所示：

![img.png](https://i.niupic.com/images/2020/10/06/8LSo.png)

原理同7，两个锁是不同的。

# 线程安全集合类
## List
1. Collections.synchroniedList(new ArrayList<>());
2. CopyOnWriteArrayList;

**CopyOnWriteArrayList**
- 写入时复制
- 多个线程调用的时候，读取时是固定的，写入的时候复制回去。
- 读写分离
- CopyOnWriteArrayList效率比Vector效率更高，（synchronized效率更低）

## Set
1. Collections.synchroniedSet(new HashSet<>());
2. CopyOnWriteArraySet;

HashSet的底层就是HashMap，用的就是HashMap的key，其value就是一个不变的值。

## Map
HashMap，装载因子0.75，默认大小16。

1. Collections.synchroniedMap(new HashMap<>());
2. ConcurrentHashMap;


# Callable
1. 可以有返回值
2. 可以抛出异常
3. 方法不同->call方法

线程启动只能通过new Thread(new Runnable())进行启动，不能接受new Callable\<V\>()的参数，而Java内部有一个适配类FutureTask\<V\>来创建，该适配类既能接受Runnable，又能接收Callable参数。
```java
public class CallableTest {
    public static void main(String[] args) {
        FutureTask<String> futureTask = new FutureTask<>(new MyCallable());//适配类
        new Thread(futureTask).start();
        
        try {
            System.out.println(futureTask.get());
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}

class MyCallable implements Callable<String> {

    @Override
    public String call() throws Exception {
        System.out.println("call方法被调用！");
        return "1231234";
    }
}
```

# 常用的辅助类
## CountDownLatch

![img.png](https://i.niupic.com/images/2020/10/06/8LSR.png)

减法计数器，主要操作：
1. 初始化计数器
2. countdown方法，计数器减一
3. await()方法，等待计数器归零后向下执行
   
示例代码如下：
```java
//减法计数器
public class CountDownLatchDemo {
    public static void main(String[] args) {
        CountDownLatch downLatch = new CountDownLatch(6);//总数是6

        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "走了！");
                downLatch.countDown();//计数器减1
            }).start();
        }

        try {
            downLatch.await();//等待计数器归0
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("走完了");
    }
}
```
运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/06/8LSV.png)

## CyclicBarrier

![img.png](https://i.niupic.com/images/2020/10/06/8LSW.png)

加法计数器，主要操作如下：
1. 初始化最大值
2. await()时调用加法计数器
3. 完成时可自定义执行操作

示例代码如下：
```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> System.out.println("召唤神龙成功！"));

        for (int i = 0; i < 7; i++) {
            final int tmp = i;
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "收集" + tmp + "颗龙珠!");
                try {
                    cyclicBarrier.await();//等待
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

## Semaphore

![img.png](https://i.niupic.com/images/2020/10/06/8LT2.png)

信号量，主要操作有以下：
1. 初始化值
2. acquire()获得
3. release()释放

作用：多个共享资源互斥使用，并发限流，控制最大的线程数等。

```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);//3个车位
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire();

                    System.out.println(Thread.currentThread().getName() + "获得车位！");
                    TimeUnit.SECONDS.sleep(2);

                    System.out.println(Thread.currentThread().getName() + "释放车位！");
                    semaphore.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```
运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/06/8LT7.png)


# ReadWriteLock
读写锁，可以多个线程读，只允许一个线程进行写，官方介绍如下：

![img.png](https://i.niupic.com/images/2020/10/07/8Mri.png)

自定义缓存，希望可以多个线程读，只允许一个线程进行写
```java
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache cache = new MyCache();
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            new Thread(() -> cache.put(String.valueOf(finalI), String.valueOf(finalI))).start();
        }

        for (int i = 0; i < 5; i++) {
            int finalI = i;
            new Thread(() -> cache.get(String.valueOf(finalI))).start();
        }
    }
}

/**
 * 自定义缓存
 */
class MyCache {
    private volatile Map<String, Object> map = new HashMap<>();

    //存入（写）
    public void put(String key, Object value) {
        System.out.println(Thread.currentThread().getName() + "写入" + key);
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() + "写入" + key + "OK!");
    }

    //取（读）
    public void get(String key) {
        System.out.println(Thread.currentThread().getName() + "读取" + key);
        Object o = map.get(key);
        System.out.println(Thread.currentThread().getName() + "读取" + key + "OK!");
    }
}
```
但是运行结果如下图所示：

![img.png](https://i.niupic.com/images/2020/10/07/8MrD.png)

其中4写入到4写入OK中间被插队了，显然没有满足要求。

使用ReadWriteLock加读写锁如下：
```java
/**
 * 自定义缓存
 */
class MyCacheLock {
    private volatile Map<String, Object> map = new HashMap<>();
    //读写锁，更加细粒度的控制
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    //存入（写），写入时只希望同时只有一个线程进行写
    public void put(String key, Object value) {
        //加写锁
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "写入" + key);
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "写入" + key + "OK!");
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }

    //取（读），读取时多个线程可以同时读
    public void get(String key) {
        //加读锁
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "读取" + key);
            Object o = map.get(key);
            System.out.println(Thread.currentThread().getName() + "读取" + key + "OK!");
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}
```
运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/07/8Ms5.png)

写入时不能被插队，但是读取时可以任意读取，满足要求。


# BlockingQueue
阻塞队列：
- （存）如果队列满了就必须等待阻塞，等待有空位。
- （取）如果队列是空的，就必须等待生产。
  
官方文档解释如下：

![img.png](https://i.niupic.com/images/2020/10/07/8Msu.png)

使用BlockingQueue的情形：
- 多线程并发处理
- 线程池

队列的四组API
| 方式         | 抛出异常  | 有返回值 | 阻塞等待 | 超时等待          |
| :----------- | :-------- | :------- | :------- | :---------------- |
| 添加         | add()     | offer()  | put()    | offer()的重载方法 |
| 删除         | remove()  | poll()   | take()   | poll()的重载方法  |
| 查看队首元素 | element() | peek()   | -        | -                 |

示例代码如下：
```java
//抛出异常
public static void test1() {
    BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
    System.out.println(blockingQueue.add("a"));
    System.out.println(blockingQueue.add("a"));
    System.out.println(blockingQueue.add("a"));
    //此时会抛出异常
    //System.out.println(blockingQueue.add("a"));

    //队首元素
    System.out.println(blockingQueue.element());

    System.out.println(blockingQueue.remove());
    System.out.println(blockingQueue.remove());
    System.out.println(blockingQueue.remove());

    //此时会抛出异常
    //System.out.println(blockingQueue.remove());
}

//返回false
public static void test2() {
    BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
    System.out.println(blockingQueue.offer("a"));
    System.out.println(blockingQueue.offer("b"));
    System.out.println(blockingQueue.offer("c"));
    //此时会返回false
    System.out.println(blockingQueue.offer("d"));

    //队首元素
    System.out.println(blockingQueue.peek());

    System.out.println(blockingQueue.poll());
    System.out.println(blockingQueue.poll());
    System.out.println(blockingQueue.poll());
    //此时会返回null
    System.out.println(blockingQueue.poll());
}

//阻塞等待
public static void test3() throws InterruptedException {
    BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);

    blockingQueue.put("a");
    blockingQueue.put("b");
    blockingQueue.put("c");

    //此时队列会一直等待
    //blockingQueue.put("d");


    System.out.println(blockingQueue.take());
    System.out.println(blockingQueue.take());
    System.out.println(blockingQueue.take());
    //此时队列会一直等待
    //System.out.println(blockingQueue.take());
}


//超时等待
public static void test4() throws InterruptedException {
    BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
    System.out.println(blockingQueue.offer("a", 2, TimeUnit.SECONDS));
    System.out.println(blockingQueue.offer("b", 2, TimeUnit.SECONDS));
    System.out.println(blockingQueue.offer("c", 2, TimeUnit.SECONDS));
    //此时会等2s后返回false
    System.out.println(blockingQueue.offer("d", 2, TimeUnit.SECONDS));

    //队首元素
    System.out.println(blockingQueue.peek());

    System.out.println(blockingQueue.poll(2, TimeUnit.SECONDS));
    System.out.println(blockingQueue.poll(2, TimeUnit.SECONDS));
    System.out.println(blockingQueue.poll(2, TimeUnit.SECONDS));
    //此时会等2s后返回null
    System.out.println(blockingQueue.poll(2, TimeUnit.SECONDS));
}
```

# SynchronousQueue 
同步队列，容量只有一个，放入后必须等待取出后再进行放新的。

```java
BlockingQueue<String> blockingQueue = new SynchronousQueue<>();

new Thread(() -> {
    try {
        for (int i = 0; i < 3; i++) {
            blockingQueue.put(String.valueOf(i));
            System.out.println(Thread.currentThread().getName() + " put " + i);
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}).start();

new Thread(() -> {
    try {
        for (int i = 0; i < 3; i++) {
            System.out.println(Thread.currentThread().getName() + " take " + blockingQueue.take());
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}).start();
```

# 线程池
**三大方法，七大参数**

程序运行的本质是占用系统的资源，而池化技术可以优化组员的使用。如线程池，连接池，内存池，对象池...

池化技术的本质是事先准备好一些资源，有人来用的时候就来这里拿，用完之后回收。

线程池的好处：
- 降低资源消耗
- 提高响应的速度
- 方便管理

即**线程复用，控制最大并发数，管理线程**

## 三大方法
```java
Executors.newSingleThreadExecutor();//单个线程
Executors.newFixedThreadPool(5);//固定大小的线程池
Executors.newCachedThreadPool();//大小可伸缩的线程池

//使用execute()方法进行运行线程
executorService.execute(new Runnable());
```


## 七大参数
源码分析：
```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }


public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,//最大线程21亿，可能会OOM
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
}


public ThreadPoolExecutor(int corePoolSize,             //核心线程池大小
                            int maximumPoolSize,        //最大核心线程池大小
                            long keepAliveTime,         //超时释放
                            TimeUnit unit,              //超时的单位
                            BlockingQueue<Runnable> workQueue,//阻塞队列
                            ThreadFactory threadFactory,       //线程工厂，创建线程的，一般不会动
                            RejectedExecutionHandler handler) {//拒绝策略
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```
线程池的理解：

![img.png](https://i.niupic.com/images/2020/10/07/8MtH.png)


## 四种拒绝策略

![img.png](https://i.niupic.com/images/2020/10/07/8MtN.png)

```java
private static final RejectedExecutionHandler defaultHandler =
    new AbortPolicy(); //默认的拒绝策略
```

| 名称                | 意义                                           |
| :------------------ | :--------------------------------------------- |
| AbortPolicy         | 如果满了就直接拒绝新的请求，并抛出异常         |
| CallerRunsPolicy    | 由原来的线程进行调用执行                       |
| DiscardPolicy       | 队列满了不会抛出异常，但是会丢掉任务           |
| DiscardOldestPolicy | 队列满了会和最开始的进行竞争替换，不会抛出异常 |

## 手动创建线程池：
```java
        ExecutorService executorService = new ThreadPoolExecutor(2   //核心营业窗口
                , 5                                           //最大营业窗口
                , 3                                           //3s不用超时释放
                , TimeUnit.SECONDS
                , new LinkedBlockingQueue<>(3)             //候客区最多3个位置
                , Executors.defaultThreadFactory()                //默认
                , new ThreadPoolExecutor.AbortPolicy());               //默认拒绝策略

//模拟银行营业
try {
    //最大承载：队列的大小+最大的size，如果同时请求的线程数大于这个数会抛出异常
    for (int i = 0; i < 9; i++) {
        executorService.execute(() -> {
            System.out.println(Thread.currentThread().getName() + " OK!");
        });
    }
} finally {
    executorService.shutdown();
}

```

**创建线程池时第二个参数maximumPoolSize，即线程池的最大线程数如何进行定义：**
1. CPU密集型，CPU是几核就设置为几（使用```Runtime.getRuntime().availableProcessors()```来设置）。
2. IO密集型，判断程序中十分耗IO的线程数，然后设置为比齐更大的数即可。

# 函数式接口
新技术
1. lambda表达式
2. 链式编程
3. 函数式接口
4. Stream流式计算

**函数式接口：只有一个方法的接口**

最标准的函数式接口：Runnable，Callable，有@FunctionalInterface注解。

四大函数式接口：
1. Function
2. Consumer



## Function<T, R>

![img.png](https://i.niupic.com/images/2020/10/07/8Mwi.png)

apply函数的说明（输入R，返回T）：

![img.png](https://i.niupic.com/images/2020/10/07/8Mwk.png)

## Predict\<T>

![img.png](https://i.niupic.com/images/2020/10/07/8Mwn.png)

test函数的说明（输入T，返回boolean）：

![img.png](https://i.niupic.com/images/2020/10/07/8Mwo.png)

## Consumer\<T>

![img.png](https://i.niupic.com/images/2020/10/07/8Mwp.png)

accept函数说明（传入T进行消费）：

![img.png](https://i.niupic.com/images/2020/10/07/8Mwq.png)

## Suplier\<T>

![img.png](https://i.niupic.com/images/2020/10/07/8Mwr.png)

get方法说明（生产一个T）：

![img.png](https://i.niupic.com/images/2020/10/07/8Mws.png)

# Stream流式计算
- 集合，MySQL本质就是用来存储
- 计算交给流

示例代码：
```java
/**
 * 要求：
 * 1.ID是偶数
 * 2.年龄大于23岁
 * 3.用户名转化为大写字母
 * 4.用户名字母倒着排序
 * 5.只输出一个用户
 */
public class StreamDemo {
    public static void main(String[] args) {
        User user1 = new User(1, "a", 21);
        User user2 = new User(2, "b", 22);
        User user3 = new User(3, "c", 23);
        User user4 = new User(4, "d", 24);
        User user5 = new User(5, "e", 25);
        List<User> list = Arrays.asList(user1, user2, user3, user4, user5);

        List<User> result = list.stream()
                .filter(user -> user.getAge() > 23 && user.getId() % 2 == 0)
                .map(user -> new User(user.getId(), user.getName().toUpperCase(), user.getAge()))
                .sorted((u1, u2) -> -1 * u1.getName().compareTo(u2.getName()))
                .limit(1)
                .collect(Collectors.toList());
        System.out.println(result);
    }
}
```
# ForkJoin
>什么是ForkJoin，主要是为了并行执行任务，提高效率所生成的。类似于分治再合并，如图所示：

![img.png](https://i.niupic.com/images/2020/10/07/8MwL.png)

>ForkJoin的一大特点：工作窃取

![img.png](https://i.niupic.com/images/2020/10/07/8MwN.png)

因为ForkJoin内维护的都是双端队列，可以从前后进行取数据。

示例代码：
```java
public class ForkJoinDemo {
    final static long start = 0L;
    final static long end = 10_0000_0000L;

    //普通加法
    public static void test1() {
        long startTime = System.currentTimeMillis();
        long sum = 0L;
        for (long i = start; i < end; ++i) {
            sum += i;
        }
        long endTime = System.currentTimeMillis();
        System.out.println("计算结果：" + sum + "\t花费时间：" + (endTime - startTime));
    }

    //ForkJoin使用
    public static void test2() throws ExecutionException, InterruptedException {
        long startTime = System.currentTimeMillis();

        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> forkJoinTask = new CalculateTask(start, end);

        forkJoinPool.execute(forkJoinTask);

        long sum = forkJoinTask.get();
        long endTime = System.currentTimeMillis();
        System.out.println("计算结果：" + sum + "\t花费时间：" + (endTime - startTime));
    }

    //并行流的计算
    public static void test3() {
        long startTime = System.currentTimeMillis();
        long sum = LongStream.range(start, end).parallel().sum();
        long endTime = System.currentTimeMillis();
        System.out.println("计算结果：" + sum + "\t花费时间：" + (endTime - startTime));
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        test1();
        test2();
        test3();
    }
}
/**
 * 使用ForkJoin的方法
 * 1.ForkJoinPool
 * 2.计算任务ForkJoinTask
 */
class CalculateTask extends RecursiveTask<Long> {
    final long start;
    final long end;
    final static long threshold = 10000L;

    public CalculateTask(long start, long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if ((end - start) < threshold) {//小任务直接计算
            long res = 0L;
            for (long i = start; i < end; ++i) {
                res += i;
            }
            return res;
        } else {
            long middle = (start + end) / 2;//拆分为两个任务

            CalculateTask task1 = new CalculateTask(start, middle);
            task1.fork();   //将任务压入线程队列
            CalculateTask task2 = new CalculateTask(middle + 1, end);
            task2.fork();   //将任务压入线程队列

            //返回结果
            return task1.join() + task2.join();
        }
    }
}
```

# 异步回调
- 类似于Ajax
- 异步执行
- 成功回调
- 失败回调

CompletableFuture官方文档说明如下：

![img.png](https://i.niupic.com/images/2020/10/07/8Mxf.png)

```java
//没有返回值的异步回调
CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println(Thread.currentThread().getName() + " run async!");
});

System.out.println("main thread");

completableFuture.get();//阻塞时执行


 //有返回值的异步回调
CompletableFuture<Integer> completableFuture1 = CompletableFuture.supplyAsync(() -> {
    System.out.println(Thread.currentThread().getName() + " supply async!");
    int i = 10 / 0;//人为产生错误
    return 1024;
});
System.out.println(completableFuture1.whenComplete((unused, throwable) -> { //正常执行
    System.out.println(unused);
    System.out.println(throwable);
}).exceptionally(throwable -> {     //如果出现问题的执行
    throwable.printStackTrace();
    return 233;
}).get());
```

# 理解JMM

JMM：Java Memory Model，java内存模型，不存在的东西，是一种概念和约定。

**关于JMM的一些同步约定：**
1. 线程解锁前，必须将共享变量立刻刷回主存
2. 线程加锁前，必须读取主存中的最新值到线程的工作内存中。
3. 加锁和解锁是同一把锁。


存在主内存和线程的工作内存的区别，8种操作如下：

![img.png](https://i.niupic.com/images/2020/10/08/8OgF.png)

但是可能存在的问题是：线程B修改了变量的值，但是线程A不能及时可见。

**8种操作具体说明如下：**

- lock（锁定）：作用于主内存的变量，把一个变量标识为一条线程独占状态。
- unlock（解锁）：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- read（读取）：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
- load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
- use（使用）：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
- assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- store（存储）：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。
- write（写入）：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。

Java内存模型还规定了在执行上述八种基本操作时，必须满足如下规则：

- 不允许read和load、store和write操作之一单独出现
- 不允许一个线程丢弃它的最近assign的操作，即变量在工作内存中改变了之后必须同步到主内存中。
- 不允许一个线程无原因地（没有发生过任何assign操作）把数据从工作内存同步回主内存中。
- 一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量。即就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。
- 一个变量在同一时刻只允许一条线程对其进行lock操作，lock和unlock必须成对出现
- 如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值
- 如果一个变量事先没有被lock操作锁定，则不允许对它执行unlock操作；也不允许去unlock一个被其他线程锁定的变量。
- 对一个变量执行unlock操作之前，必须先把此变量同步到主内存中（执行store和write操作）。

示例代码如下：
```java
public class JMMDemo {
    private static int num = 0;

    public static void main(String[] args) {
        new Thread(() -> {
            while (num == 0) {
            }
            System.out.println(Thread.currentThread().getName() + "运行结束!");
        }, "A").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        num = 1;

        System.out.println("in main thread num=" + num);
    }
}
```
猜想的是执行num=1之后A线程能够立刻退出，但是事实运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/08/8OgG.png)

可见一个线程修改了公共变量的值，另一个线程不一定能够立即看见，示例代码中可以得知“A线程对于主内存中的变化是不知道的”。



# volatile
volatile是Java虚拟机提供轻量级的同步机制，有如下特点：
1. 保证可见性
2. 不保证原子性
3. 禁止指令重拍

>可见性

如果对上述的测试代码中的num变量加上volatile关键字之后可以保证可见性，运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/08/8OgH.png)

>原子性

原子性：不可分割，线程A在执行任务的时候，不能被打扰的，也不能被分割，要么同时成功，要么同时失败。

示例代码如下：
```java
public class VolatileDemo {
    private static volatile int num = 0;

    public static void add() {
        num++;
    }

    public static void main(String[] args) {
        //理论上 num结果应该为2万
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    add();
                }
            }).start();
        }

        while (Thread.activeCount() > 2) {// 默认main和gc线程
            Thread.yield();
        }
        System.out.println(num);
    }
}
```
运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/08/8OgJ.png)

可见volatile并不会保证原子性。如果需要保证原子性可以加锁（Lock和synchronized）。

本质原因是因为```num++```这行代码并不是原子性操作，对字节码文件反编译结果如下：

![img.png](https://i.niupic.com/images/2020/10/08/8OgM.png)

对于```num++```有如下三步：
1. 获得这个值
2. +1操作
3. 写回这个数据

但是不使用Lock和synchronized如何保证原子性：使用原子类（Atomic）

![img.png](https://i.niupic.com/images/2020/10/08/8OgN.png)

代码修改如下：
```java
public class VolatileDemo {

    private static AtomicInteger num = new AtomicInteger(0);

    public static void add() {
        num.getAndIncrement();//用的是底层的CAS方法
    }

    public static void main(String[] args) {
        //理论上 num结果应该为2万
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    add();
                }
            }).start();
        }

        while (Thread.activeCount() > 2) {// 默认main和gc线程
            Thread.yield();
        }
        System.out.println(num);
    }
}
```

运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/08/8OgQ.png)

原子类的底层(UnSafe类)都直接和操作系统进行挂钩，所以效率十分高效，比锁的效率更高。

>指令重排

指令重排：我们写的程序代码并不是按照我们写的那样去执行的。

源代码-->编译器优化重排-->指令并行也可能会重排-->内存系统也会进行重排-->执行

```java
int x = 1;//1
int y = 2;//2
x = x * 5;//3
y = x * x;//4
```
指令排序所希望的执行顺序是1234，但是可能执行的时候变成2134，1324

但是不可能是4123（因为处理器在进行指令重排的时候会考虑数据之间的依赖性）

上述为没造成影响的指令重排，但是也可能会有造成影响的指令重排的情况：
a,b,x,y四个变量值都是0
| 线程A | 线程B |
| :---- | :---- |
| x=a   | y=b   |
| b=1   | a=2   |
正常的运行结果：x=0,y=0，但是可能由于指令重排会产生如下的情况：
| 线程A | 线程B |
| :---- | :---- |
| b=1   | a=2   |
| x=a   | y=b   |
最后的运行结果是：x=2，y=1，而volatile可以避免指令重排。

volatile避免指令重排的实现由CPU的内存屏障实现：
1. 保证特定的操作执行顺序
2. 可以保证某些变量的可见性。

![img.png](https://i.niupic.com/images/2020/10/08/8OgR.png)

volatile的指令重排在单例模式中应用的最广。

# 单例模式
>饿汉式

```java
//饿汉式加载
class Hungry {
    private Hungry() {

    }

    private final static Hungry HUNGRY = new Hungry();

    public static Hungry getInstance() {
        return HUNGRY;
    }
}
```
可能会产生的问题：大内存的存放，只想要使用时才进行加载，解决办法：**懒汉式**

>懒汉式
```java
//懒汉式
class Lazy {
    private Lazy() {

    }

    private static Lazy lazy;

    private static Lazy getInstance() {
        if (lazy == null) {     //运行时才创建
            lazy = new Lazy();
        }
        return lazy;
    }
}
```
这种方式的懒汉式加载在单线程下是没问题的，但是在多线程下的懒汉加载是有问题的：

![img.png](https://i.niupic.com/images/2020/10/08/8OgZ.png)

改进版本如下：
```java
public static Lazy getInstance() {
    if (lazy == null) {     //运行时才创建
        synchronized (Lazy.class) {//锁住内部再进行判断
            if (lazy == null) {
                lazy = new Lazy();
            }
        }
    }
    return lazy;
}
```

但是极端情况下也会有问题，因为```lazy = new Lazy()```这行代码并不是原子性操作，指令重排时可能会造成问题。

```lazy = new Lazy()```的过程：
1. 分配内存空间
2. 执行构造方法，初始化对象
3. 将这个对象指向这个内存地址

如果指令重排，造成A线程执行的结果是1-->3-->2，但是此时再进入一个B线程，获得内存的地址，但是还没有进行初始化，可能会造成问题，所以需要将单例对象使用volatile关键字进行修饰。

**还可能存在的问题：可能通过反射来获取其构造函数（记得设置setAccessable为true），通过构造函数来反射创建对象。**

# CAS


AtomicInteger的自增操作底层实现：

![img.png](https://i.niupic.com/images/2020/10/08/8Ohg.png)

其中Object为Atomic对象，offset为地址的偏移值，通过这两个值可以通过getIntVolatile函数进行获取int在内存中的值（使用native方法），然后再比较这个值和原先的地址中的值进行交换设置。

其中do-while内为一个自旋锁。

CAS：compareAndSet()，比较当前工作内存中的值和主内存中的值，如果这个值是期望的，那么就设置新的值，如果不是就一直循环。

缺点：
1. 循环会耗时（自旋锁）
2. 一次性只能保证一个共享变量的原子性
3. ABA问题

**ABA问题**

线程1将原来的值A换成了一个新的值B，然后又换回原来的值A，但是线程2对其是不知道原来的值是经过变换之后的。这个情况对于SQL来说时为乐观锁。

解决办法：使用原子引用，即带版本号的原子操作。Java内为AtomicReference。
```java
public class CASDemo {
    public static void main(String[] args) {
        //注意如果泛型是包装类需要注意对象的引用问题。Integer是-128~127使用缓存
        AtomicStampedReference<Integer> integerAtomic = new AtomicStampedReference<>(123, 1);
        new Thread(() -> {

            System.out.println(integerAtomic.compareAndSet(123, 1,
                    integerAtomic.getStamp(), integerAtomic.getStamp() + 1));

            System.out.println(integerAtomic.compareAndSet(1, 123,
                    integerAtomic.getStamp(), integerAtomic.getStamp() + 1));


        }, "A").start();
        new Thread(() -> {
            int version = 1;//获得版本号
            //延迟一下
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(integerAtomic.compareAndSet(123, 1,
                    version, version + 1));//修改失败
        }, "B").start();
    }
}
```
# 各种锁
1. 公平锁，非公平锁
   - 公平锁：非常公平，不能够插队，必须先来后到
   - 非公平锁：不公平，能够插队。
   - ReetrantLock的构造参数就是公平或者是非公平。
2. 可重入锁（递归锁）
   - 拿到了外面对象的锁就相当于获得了对象内部对象的锁。
```java
public class LockDemo {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(() -> phone.sms()).start();

        new Thread(() -> phone.sms()).start();
    }
}

class Phone {
    public synchronized void sms() {
        System.out.println(Thread.currentThread().getName() + " sms！");
        call();//获得里面的锁
    }

    public synchronized void call() {
        System.out.println(Thread.currentThread().getName() + " call！");
    }
}
```
运行结果只能是如下图所示：

![img.png](https://i.niupic.com/images/2020/10/08/8Onr.png)

Lock版本代码如下：
```java
public class LockDemo {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(() -> phone.sms()).start();

        new Thread(() -> phone.sms()).start();
    }
}

class Phone {
    Lock lock = new ReentrantLock();

    public void sms() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + " sms！");
            call();//获得里面的锁
        } finally {
            lock.unlock();
        }

    }

    public void call() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + " call！");
        } finally {
            lock.unlock();
        }
    }
}
```
3. 自旋锁
   
CAS内的自旋锁：

![img.png](https://i.niupic.com/images/2020/10/08/8Ohg.png)

使用CAS实现的自旋锁
```java
public class SpinlockDemo {
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    //加锁
    public void myLock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + ">>>myLock OK!");
        //自旋锁
        while (!atomicReference.compareAndSet(null, thread)) {

        }
    }

    //解锁
    public void myUnlock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + ">>>myUnLock OK!");

        //不用自旋
        atomicReference.compareAndSet(thread, null);
    }

    public static void main(String[] args) {

        SpinlockDemo spinlockDemo = new SpinlockDemo();

        new Thread(() -> {
            spinlockDemo.myLock();
            try {
                TimeUnit.SECONDS.sleep(8);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                spinlockDemo.myUnlock();
            }
        }).start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            spinlockDemo.myLock();//这里需等待线程1解锁
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                spinlockDemo.myUnlock();
            }
        }).start();
    }
}
```
4. 死锁
   
死锁的情况：

![img.png](https://i.niupic.com/images/2020/10/08/8Ooq.png)

死锁的例子：
```java
public class DeadLockDemo {
    public static void main(String[] args) {
        final String lockA = "lockA";
        final String lockB = "lockB";
        new Thread(new MyRunnable(lockA,lockB)).start();
        new Thread(new MyRunnable(lockB,lockA)).start();
    }
}

class MyRunnable implements Runnable {
    private final String lockA;
    private final String lockB;

    public MyRunnable(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @Override
    public void run() {
        synchronized (lockA) {
            System.out.println(Thread.currentThread().getName() + "lock:" + lockA + "=>get" + lockB);
            synchronized (lockB) {
                System.out.println(Thread.currentThread().getName() + "lock:" + lockB + "=>get" + lockA);
            }
        }
    }
}
```
运行结果如下图所示：

![img.png](https://i.niupic.com/images/2020/10/08/8Oot.png)

死锁排查：
1. 使用```jps -l```定位进程号。
2. 使用```jstack 进程号```找到死锁问题。


![img.png](https://i.niupic.com/images/2020/10/08/8Oov.png)
