# 常见面试题
- 谈谈对JVM的理解，java8虚拟机和之间的变化（更新）
- 什么是OOM，什么是栈溢出（StackOverFlowError），如何进行分析
- JVM常用的调优参数
- 内存快照如何进行抓取，怎么分析dump文件
- 谈谈对JVM中类加载器的认识。

需要了解的知识
1. JVM的位置
2. JVM的体系结构
3. 类加载器
4. 双亲委派机制
5. 沙箱安全机制
6. Native
7. PC寄存器
8. 方法区
9. 栈
10. 三种JVM
11. 堆
12. 新生区，老年去
13. 永久区
14. 堆内存调优
15. GC垃圾回收算法
16. JMM
17. 总结

# JVM所处的位置：

![img.png](https://i.niupic.com/images/2020/10/10/8Qxy.png)

# JVM的体系结构：

![img.png](https://i.niupic.com/images/2020/10/10/8QzQ.png)

- Java栈区，本地方法栈区以及程序计数器这三个位置不会有垃圾回收。
- JVM调优：大部分在堆区域（方法区是特殊的堆区）进行调优

# 类加载器
作用：加载Class文件

![img.png](https://i.niupic.com/images/2020/10/10/8QA6.png)

类加载器的种类：
- 虚拟机自带的加载器
- 启动类（根）加载器
- 拓展类加载器
- 应用程序类加载器

# 双亲委派机制
应用程序类加载器->找拓展类加载器是否能够加载->找根类加载器能否加载

如果不行则倒着走：根类加载器能否加载->拓展类加载器是否能够加载->应用程序类加载器

步骤如下：
1. 类加载器收到类加载的请求
2. 将这个请求向上委托给父类加载器去完成，一直向上委托，直到启动类加载器（根类加载器）
3. 启动加载器检测能否加载当前这个类，如果能够加载就结束并使用当前的类加载器，否则抛出异常，通知子类加载进行加载
4. 重复步骤3


如果自定义一个java.lang.String并尝试在应用程序内进行加载，类加载器会加载rt.jar包内的java.lang.String，而不会加载用户自定义的类加载器（为了安全性考虑）。

# 沙箱安全机制
https://blog.csdn.net/qq_30336433/article/details/83268945

# native方法

Thread.start()的底层方法：

![img.png](https://i.niupic.com/images/2020/10/10/8QAj.png)

凡是带了native关键字的，说明java的作用范围达不到了，会去调用底层的C语言的库，会进入本地方法栈。

进入本地方法栈后会进行调用本地方法接口（JNI）。

JNI的解释：
- 作用：拓展java的使用，融合不同的编程语言为java所用（最初为C和C++）
- 在内存区域中专门开辟了一块标记区域：Native Method Stack用来登记native方法
- 在最终执行的时候，加载本地方法库中的方法通过JNI

现代调用其他语言的接口：
- Socket
- WebService
- HTTP

# PC计数器
程序计数器：Program Counter

每一个线程都有一个程序计数器，是线程私有的，就是一个指针，执行方法区中的方法字节码，（用来存储指向下一条指令的地址）。

# 方法区
Method Area方法区，是被所有线程进行共享的。

**静态变量，常量，类的信息（构造方法，接口定义），运行时的常量池存在方法区中，但是实例变量存在堆内存中，和方法区无关。**

方法区：static,final,Class,常量池。

# 栈
栈：先进后出，后进先出

为什么main方法最先执行，最后结束？

- 程序运行最开始：将main方法压入栈
- main方法调用其他方法，将其他方法压入栈
- 其他方法运行完毕时弹出栈
- main方法运行完毕main方法弹出栈，程序结束

栈内寸主管程序的运行，生命周期和线程同步。线程结束栈内存就释放，**对于栈来说，不存在垃圾回收。**一旦线程结束，栈就结束了。

栈内存在的数据：8种基本类型，对象引用，实例的方法。

如果栈满了就会报StackOverFlowError

# 三种JVM
- Hotspot（Sun公司）
- JRockit（BEA公司）
- J9 VM（IBM公司）

# 堆
堆，heap，一个JVM只有一个堆内存，堆内存的大小是可以进行调节的。

类加载器读取了类文件之后，一般会将以下东西放到堆中：
- 类
- 方法
- 常量
- 变量
- 保存所有引用对象的真实对象

堆内存中还需要进行细分为三个区域：
- 新生区
- 老年区
- 永久区

![img.png](https://i.niupic.com/images/2020/10/10/8QGP.png)

幸存区0和幸存区1会不断的进行交换，在多次GC后还存活的对象就会进入老年区。

GC垃圾回收主要是在伊甸园区（新生区）和老年区。

如果内存满了就会报错：OOM（out of memory： heap space），即堆内存不足。

在JDK8之后，永久存储区改名为元空间。

# 新生区，老年区以及永久区
**新生区**
- 类的诞生，成长或者是死亡的区域。
- 伊甸园区，所有的对象都是在伊甸园区进行new出来的。
- 幸存区（0，1）

**GC过程：**
- 如果伊甸园区满了就会触发一次轻GC，活下来的对象会进入幸存区0和1
- 在多次轻GC后导致幸存区0和1内都满了之后对象会进入老年区。
- 如果老年区内也满了之后会触发一次重GC（Full GC）
- 如果新生区和老年区都满了之后会触发OOM错误。

经过调查，99%的对象都是临时对象，即进入老年区的对象很少。

**永久区：**

这个区域常驻内存，用来存放JDK自身携带的Class对象，Interface元对象，存储的是Java运行的一些环境或者是类型学，这个区域不存在垃圾回收。只有在JVM关闭的时候才会释放这个区域的内存。

一个启动类如果加载了大量的第三方jar包，或者是Tomcat部署了太多的应用，大量动态生成反射类，不断的被加载，直到内存满，抛出OOM异常。

- JDK1.6之前：永久代，常量池在方法区中
- JDK1.7：永久代也存在，但是慢慢的进行退化（去永久代），常量池在堆中。
- JDK1.8：无永久代，常量池在元空间。

![img.png](https://i.niupic.com/images/2020/10/10/8QKY.png)

默认情况下JVM分配的总内存是电脑内存的1/4，而初始化的内存是电脑内存的1/64。
```java
public class MemoryDemo {
    public static void main(String[] args) {
        long max = Runtime.getRuntime().maxMemory();
        long total = Runtime.getRuntime().totalMemory();

        System.out.println(max + "byte\t" + max / (1024 * 1024) + "MB");
        System.out.println(total + "byte\t" + total / (1024 * 1024) + "MB");
    }
}
```
运行结果如下：

![img.png](https://i.niupic.com/images/2020/10/10/8QLr.png)

（本人电脑最大内存16GB）。

可以通过JVM启动参数```-Xms1024m -Xmx4096m -XX:+PrintGCDetails```设置我们的最大的可运行内存以及初始化的内存和打印GC信息，如下图所示：

![img.png](https://i.niupic.com/images/2020/10/10/8QLv.png)

-Xms为初始化内存，-Xmx为最大内存。

OOM的解决办法：
1. 尝试扩大堆内存看结果
2. 如果还是OOM，那么分析内存看一下哪个地方出现了问题。

逻辑上存在元空间（永久代），但是物理上是不存在的。

# JProfile分析OOM
- 能够看到代码第几行出错，内存快照分析工具：JProfile。
- Debug。

JProfile可以分析dump内存文件，快速定位内存泄漏；可以获得堆中的数据；

一个OOM的例子如下：
```java
public class MemoryDemo {
    public static void main(String[] args) {
        List<byte[]> byteLists = new LinkedList<>();
        int count = 0;
        try {
            while (true) {
                byteLists.add(new byte[1024]);
                count++;
            }
        }catch (Error e){
            System.out.println(count);
        }

    }
}
```

在运行时加上JVM启动参数： ```XX:+DumpOnOutOfMemoryError```表示出现OOM时生成dump文件，如下图所示：

![img.png](https://i.niupic.com/images/2020/10/10/8QPL.png)

![img.png](https://i.niupic.com/images/2020/10/10/8QPV.png)

可以通过ThreadDump来查看出错的那行代码

![img.png](https://i.niupic.com/images/2020/10/10/8QQ4.png)

面试题：
- JVM的内存模型和分区
- 堆里面的分区有哪些，说说他们的特点
- GC的算法。
- 轻GC和重GC在什么时候发生。

# 常用的GC算法
JVM在进行GC时，并不是对三个区域（新生区，幸存区，老年区）进行统一回收，大部分回收都在新生代进行。

GC的两种种类：
- 轻GC：大部分新生区和偶尔的幸存区
- 重GC：老年区

**引用计数**

一个对象如果存在一个引用，那么其计数器就加1，如果失去一个对象的引用，那么计数器减一，如果计数器为0，标志着这个对象可以被回收。


**复制算法（主要是年轻代使用）**

将幸存区0和1命名为from和to（谁是空的谁是to）。
- 每一次垃圾回收都会将伊甸园区存活的对象移到幸存区中（一旦伊甸园区被GC后，他就会变空）。
- 假设两个幸存区内的对象还活着，就将这一个幸存区的活着的对象复制到另一个幸存区内，将原幸存区清空。
- 当一个对象经历了15次GC后还存活时就将这个对象移到老年区（参数可调）。

经历一次轻GC的过程：

![img.png](https://i.niupic.com/images/2020/10/10/8QRF.png)

好处：
- 没有内存碎片的存在

坏处：
- 浪费的一块内存空间
- 假设对象100%存活，所有的对象都需要进行复制，复制所需要的时间就会更长。

**复制算法最佳使用场景：对象存活度角度的时候，即新生区的时候。**

**标记清除算法**

扫描这些对象，然后对存活的对象进行标记，如果没有标记的对象将执行清除操作。

![img.png](https://i.niupic.com/images/2020/10/10/8QTp.png)

缺点：
- 两次扫描，浪费时间，会产生内存碎片。

优点：
- 不需要额外的内存空间。

**标记压缩**

去除标记清除的“内存碎片的问题”，每次清除之后将存活的对象重新进行排序，去除了内存碎片的问题。

![img.png](https://i.niupic.com/images/2020/10/10/8QUk.png)

缺点：
- 多了一次移动的成本

再进行优化的版本：**标记清除压缩**

多次进行标记清除之后，产生大量内存碎片后进行一次标记压缩算法。

**总结**

内存效率：复制算法>标记清除算法>标记压缩算法

内存整齐度：复制算法=标记压缩算法>标记清除算法

内存利用率：标记压缩算法=标记清除算法>复制算法

**没有最好的GC算法，只有最合适的GC算法。--->GC分代收集算法**

年轻代：
- 存活率低，使用复制算法。

老年代：
- 存活率高，区域大，标记清除+标记压缩混合实现。