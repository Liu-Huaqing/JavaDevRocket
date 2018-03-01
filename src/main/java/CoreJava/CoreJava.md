## CoreJava

### hashcode 相等两个类一定相等吗？equals 呢？相反呢？
* hashcode 相等不代表两个类相等；equals 相等代表类的实现者实现了 equals 并且设定了 equals 相等的逻辑，所以是相等的。
* 相反，两个类相等，则 equals 肯定相等，hashcode 也一定相等；（这里的相等是指的物理上的含义）

### 介绍一下集合框架
集合框架是`一组用来代表和操作集合的架构`，包括相应的：
* `接口`
* `实现`
* `算法`

主要的好处包括：
* 降低了成本：
  * `学习使用新 API` - 基于统一的接口，容易掌握
  * `设计新 API` - 直接使用现有 collection 接口
  * `编程成本` - 写代码直接使用，方便快捷
* 增加了
  * `速度和质量` - 集合框架都已经高质量的实现，可直接使用
  * `互操作性` - 新写的库与其他库，只要遵循标准接口，就可以无缝互操作
  * `代码重用性`

[optional] 将集合框架拆的更细一点的话，包括的内容有：

* `Collection interfaces`               - Represent different types of collections, such as sets, lists, and maps. These interfaces form the basis of the framework.
* `General-purpose implementations`     - Primary implementations of the collection interfaces.
* `Legacy implementations`              - The collection classes from earlier releases, Vector and Hashtable, were retrofitted to implement the collection interfaces.
* `Special-purpose implementations`     - Implementations designed for use in special situations. These implementations display nonstandard performance * characteristics, usage restrictions, or behavior.
* `Concurrent implementations`          - Implementations designed for highly concurrent use.
* `Wrapper implementations`             - Add functionality, such as synchronization, to other implementations.
* `Convenience implementations`         - High-performance "mini-implementations" of the collection interfaces.
* `Abstract implementations`            - Partial implementations of the collection interfaces to facilitate custom implementations.
* `Algorithms`                          - Static methods that perform useful functions on collections, such as sorting a list.
* `Infrastructure`                      - Interfaces that provide essential support for the collection interfaces.
* `Array Utilities`                     - Utility functions for arrays of primitive types and reference objects. Not, strictly speaking, a part of the collections framework, this feature was added to the Java platform at the same time as the collections framework and relies on some of the same infrastructure.


集合接口包括：

The most basic interface, 

`java.util.Collection`, has the following descendants:
* java.util.Set
* java.util.SortedSet
* java.util.NavigableSet
* java.util.Queue
* java.util.concurrent.BlockingQueue
* java.util.concurrent.TransferQueue
* java.util.Deque
* java.util.concurrent.BlockingDeque

`java.util.Map` has the following offspring:
* java.util.SortedMap
* java.util.NavigableMap
* java.util.concurrent.ConcurrentMap
* java.util.concurrent.ConcurrentNavigableMap

在实现接口时候，仍`考虑以下的维度`：

* `Mutability` - Collection is mutable or immutable
* `Modifiability` - 子类在实现的时候，有些修改操作如果不实现可以直接抛出 UnsupportedOperationException
* `Resizability` - List is fixed-size or varaible-size
* `Access Method` - List is sequential access or random access
* `Element Restriction` - 
  * Be of particular type
  * Be not null
  * Obey some arbitrary predicate

接口的实现

![collection-implmentation](./collection-implementation.jpeg)

并发编程中接口实现为：

* `BlockingQueue`
* `TransferQueue`
* `BlockingDeque`
* `ConcurrentMap`
* `ConcurrentNavigableMap`

相关的实现为：

* `LinkedBlockingQueue`
* `ArrayBlockingQueue`
* `PriorityBlockingQueue`
* `DelayQueue`
* `SynchronousQueue`
* `LinkedBlockingDeque`
* `LinkedTransferQueue`
* `CopyOnWriteArrayList`
* `CopyOnWriteArraySet`
* `ConcurrentSkipListSet`
* `ConcurrentHashMap`
* `ConcurrentSkipListMap`

设计目标：

* 【后续补充】

## Hashmap 和 Hashtable 有什么区别？ Hashtable vs SynchronizedMap vs ConcurrentHashMap ?

### Hashmap 和 Hashtable 有什么区别？
* `Hashtable is synchronized, whereas HashMap is not`
* `Hashtable does not allow null keys or values`.  `HashMap allows one null key and any number of null values.`
* One of HashMap's subclasses is LinkedHashMap, so in the event that you'd want predictable iteration order.

`I'd recommend HashMap`. If synchronization becomes an issue, `you may also look at ConcurrentHashMap`.

### Hashtable vs SynchronizedMap vs ConcurrentHashMap

#### SynchronizedMap
* `Synchronization at Object level`.
* Every `read/write operation needs to acquire lock`.
* This essentially gives access to `only one thread to the entire map` & `blocks all the other threads`.
* SynchronizedHashMap returns Iterator, which fails-fast on concurrent modification.

#### ConcurrentHashMap
* `High concurrency`.
* It is `thread safe without synchronizing the whole map`.
* Reads can happen very fast while `write is done with a lock`.
* There is `no locking at the object level`, but at a `hashmap bucket level`.
* ConcurrentHashMap uses `multitude of locks`.



引用

* [@Java 集合框架](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)
* [@Java 8 Collection Tutorials](https://docs.oracle.com/javase/tutorial/collections/index.html)

