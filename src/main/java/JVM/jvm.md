# JVM
## 请介绍一下 JVM 内存模型？用过什么垃圾回收器？都说说呗?

### Hotspot JVM Architecuture

![jvm_architecture](./jvm_architecture.png)

### 理解内存模型

运行时内存模型，分为线程私有和共享数据区两大类，其中线程私有的数据区包含程序计数器、虚拟机栈、本地方法区，所有线程共享的数据区包含Java堆、方法区，在方法区内有一个常量池。

![jvm_memory_model](./jvm_memory_1.png)

线程私有区：
* `程序计数器`，记录正在执行的虚拟机字节码的地址;
* `虚拟机栈`：方法执行的内存区，每个方法执行时会在虚拟机栈中创建栈帧;
* `本地方法栈`：虚拟机的Native方法执行的内存区;

线程共享区：

* `Java堆`：对象分配内存的区域；
* `方法区`：存放`类信息、常量、静态变量、编译器编译后的代码`等数据；
* `常量池`：存放编译器生成的各种字面量和符号引用，是方法区的一部分

### 详细模型

JVM 运行时内存五大块区域

![jvm_memory_model](./jvm_memory_2.png)

Java 7 JVM 内存再细分
* 堆被分为2个不同的区域：`新生代（Young）和老年代（Tenured）`
* 新生代被分为：`Eden、From Suvivor、To Survivor`
  * Most of the `newly created objects are located in the Eden memory space`.
  * When Eden space is filled with objects, `Minor GC is performed and all the survivor objects are moved to one of the survivor spaces`.
  * `Minor GC` also checks the survivor objects and move them to the other survivor space. 
  * `Objects that are survived after many cycles of GC, are moved to the Old generation memory space`. 
* 方法区又被叫做 `PermGen`

![jvm_memory_model](./jvm_memory_4.png)

![jvm_memory_model](./jvm_memory_3.png)


Java 8 内存模型发生变化

![jvm_memory_model](./jvm_memory_5.png)

### 用过什么垃圾收集器？（更多的垃圾收集器细节，暂未研究）
* Serial GC - STW 单线程`对新生代进行垃圾收集`
* Parallel GC (Throughput Collector) - STW 多线程，`对新生代进行垃圾收集`
* CMS GC - Concurrent Mark Sweep (CMS) The Concurrent Low Pause Collector，`对旧生代垃圾收集`，由于垃圾收集时候不 STW，可以提高应用响应能力
* G1 GC - 内存布局都和上面不一样，是`未来的垃圾收集器`

### 性能调优
`Garbage Collector 的选择和新生代、旧生代大小的配置，会严重影响进程的吞吐量、相应时间`。因此需要进行合理的调优和测试，`使用 JVisual VM 来进行性能监控`。

## 线上频繁报 Full GC 如何处理？CPU 使用率过高怎么办？如何定位问题？如何解决？说一下解决思路和方法。
这里先定义 Minor GC、Major GC、Full GC

Minor GC
* `Collecting garbage from Young space`
* `All Minor GCs do trigger stop-the-world pauses`, but the length is short

什么是 Full GC:
* Full GC == Major GC 指的是对老年代/永久代的 stop the world 的 GC
* `Full GC 的次数 = Major GC STW 的次数`

`Full GC 会降低应用的性能和效率`

什么时候会触发 Full GC:
* `System.gc`
* `老年代空间不足` - 新生代转入、创建大对象、大数组时候，Full GC 后空间仍然不足，则报 OOM: Java Heap Space
* `永生代空间不足` - Full GC 后空间仍然不足，则报 OOM: PermGen space
* `CMS GC 时出现promotion failed和concurrent mode failure`
* `统计得到的Minor GC晋升到旧生代的平均大小大于老年代的剩余空间`
* `堆中分配很大的对象`

Full GC 该如何处理：
* 判断堆大小是否合理，是否有内存泄露或者无谓的内存使用；
* 对不合理的内存泄露，通过`修改程序代码`来解决；
* 如果程序本身需要较大的堆，`加大堆的大小`；

CPU 使用率过高怎么办？
* 找到 CPU 使用率过高的线程 id；
* 使用 jstack 来查看导致 CPU 使用率过高的代码；

## 知道字节码吗？字节码都有哪些？（忽略）

## 讲讲类加载器呗？都有哪些类加载器？这些类加载器都加载哪些文件？

## 知道 osgi 吗？他是如何实现的？

## 请问你做过那些 JVM 优化？使用什么方法？达到什么效果？？

## class.forName("java.lange.String") 和 String.class.getClassLoader().loadClass("java.lang.String")

引用
* [@G1 and CMS Gabage Collector 详解](http://www.oracle.com/technetwork/tutorials/tutorials-1876574.html)
* [@Java 7 GC Basics[重要]](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)
* [@Java 7 垃圾收集器使用](http://www.fasterj.com/articles/oraclecollectors1.shtml)
* [@Java 垃圾收集器](http://www.cnblogs.com/zhanglei93/p/6636831.html)
* [@JVM 内存模型](http://gityuan.com/2016/01/09/java-memory/)
* [@JVM 异常完全指南](https://www.jianshu.com/p/2fdee831ed03)
* [@Java 7 Memory Model](https://www.journaldev.com/2856/java-jvm-memory-model-memory-management-in-java)
* [@触发 JVM 进行 Full GC 的情况及应对策略](http://blog.csdn.net/chenleixing/article/details/46706039)
* [@分析 Java 应用 cpu 占用过高的问题](http://blog.csdn.net/jiangguilong2000/article/details/17971247)

