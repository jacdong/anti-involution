## java 基础知识

### Java基础知识
#### volatile
* **内存模型**

```properties
JMM 模型，工作内存，主内存
```


* **volatile 的主要作用** <br/>
	<font color=red>当变量申明为volatile 之后， 寄存器在运行时都会注意到这个变量是共享的，因此不会将该变量的操作与其他内存操作进行重排序。
	volatile 的变量不会被缓存在寄存器或者其他处理器不可以见的地方，因此每次读取的volatile 变量值总是最新的。
	首先禁止重排序，保证了每次对于这个共享变量的操作是串行的，然后当需要这个值的时候，其实每次读取的就是最新的值</font>
	Java语言提供了一种稍弱的同步机制，即volatile变量，用来确保将变量的更新操作通知到其他线程。当把变量声明为volatile类型后，
	编译器与运行时都会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存操作一起重排序。volatile变量不会被缓存在寄存器或者对其他处理器不可见的地方，
	因此在读取volatile类型的变量时总会返回最新写入的值。
	在访问volatile变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此volatile变量是一种比sychronized关键字更轻量级的同步机制。

```properties
1. 可见性
2. 有序性 <禁止指令重排> 
	synchronized 同样可以保证有序性，不过synchronized 是通过限制在同一个时刻只允许一个线程对其lock 来保证有序性
```

* **volatile 如何实现将工作内存的数据刷新到主内存**<br/>
   缓存一致协议，cpu 在写数据的时候，如果发现这个变量是共享变量，那么会通知所有使用到这个变量的其他cpu，将这个变量的缓存失效，当其他CPU 需要使用这个变量的时候
   重新从主存里面去获取
   
```java
	1. 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；
	即在执行到内存屏障这句指令时，在它前面的操作已经全部完成
　　	2. 它会强制将对缓存的修改操作立即写入主存
　　	3. 如果是写操作，它会导致其他CPU中对应的缓存行无效.
```


** 在每个volatile写操作前插入StoreStore屏障，在写操作后插入StoreLoad屏障；在每个volatile读操作前插入LoadLoad屏障，在读操作后插入LoadStore屏障； **

#### final
1. **如何解决可见性**
由于final修饰的变量是常量，也就是说不可变的，因为final在类加载或者类初始化的时候已经确定了final修饰的变量的值，所以final可以保证可见性。
2. **如果解决有序性**
final的有序性也是通过重排序规则来保证的，对于final域，编译器和处理器要遵守以下两个重排序规则。
1)、在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
```java
即先赋值final 域，再将包含final 域的对象赋予其他对象进行应用
```
2)、初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。
```java
即先读取包含final 域的对象，再读取final 值
```

*写final域重排序规则* <br/>
写final域的重排序规则禁止把final域的写重排序到构造函数之外。这个规则包含两方面
	> JMM禁止编译器把final域的写重排序到构造函数之外。
	> 编译器会在final域的写之后，构造函数return之前，插入一个StoreStore屏障。这个屏障禁止处理器把final域的写重排序到构造函数之外
写final域的重排序规则可以确保：在**对象引用**为任意线程可见之前，对象的final域已经被正确初始化过了，而普通域就不具有这个保障。

读final域重排序规则
读final域的重排序规则是，在一个线程中，初次读对象引用与初次读该对象包含的final域，JMM禁止处理器重排序这两个操作(注意，这个规则仅仅针对处理器)。编译器会在读final域操作的前面插入一个LoadLoad屏障。
final引用不能从构造函数内“溢出”
读final域的重排序规则可以确保：在读一个对象的final域之前，**一定会先读这个包含这个final域的对象的引用。**

#### static
#### java 中volatile ， final ， synchronized 都是怎样保证可见性，有序性和原子性
1.Synchronized 主要是通过加锁的方式保证有序性
当释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中。
当获取锁时，JMM会把该线程对应的本地内存置为无效，从而使得被监视其保护的临界区代码必须从主内存中读取共享变量。

#### Happens-Before规则
* 程序次序规则：在一个线程内一段代码的执行结果是有序的。就是还会指令重排，但是随便它怎么排，结果是按照我们代码的顺序生成的不会变。
* 管程锁定规则：就是无论是在单线程环境还是多线程环境，对于同一个锁来说，一个线程对这个锁解锁之后，另一个线程获取了这个锁都能看到前一个线程的操作结果！(管程是一种通用的同步原语，synchronized就是管程的实现）
* volatile变量规则：就是如果一个线程先去写一个volatile变量，然后一个线程去读这个变量，那么这个写操作的结果一定对读的这个线程可见。
* 线程启动规则：在主线程A执行过程中，启动子线程B，那么线程A在启动子线程B之前对共享变量的修改结果对线程B可见。
* 线程终止规则：在主线程A执行过程中，子线程B终止，那么线程B在终止之前对共享变量的修改结果在线程A中可见。也称线程join()规则。
* 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程代码检测到中断事件的发生，可以通过Thread.interrupted()检测到是否发生中断。
* 传递性规则：这个简单的，就是happens-before原则具有传递性，即hb(A, B) ， hb(B, C)，那么hb(A, C)。
* 对象终结规则：这个也简单的，就是一个对象的初始化的完成，也就是构造函数执行的结束一定 happens-before它的finalize()方法。

### Java 类的实例化过程
### 集合类的关系与梳理
#### 集合类的继承关系
#### hashmap 与 hashtable 的关系
#### 源码知识hashmap/ConcurrentHashMap
```java
1. hashmap  每次增长都是前一次初始容量的的2倍
2. 之所以是2倍 是因为   在求index 的时候  hash&(length-1)[都是00000011111……] 相当于右位移了2n 位置。相当于取模
3. 扩容之后，jdk 1.7 需要重新计算hash 值， jdk 1.8 利用  length 长度增加了2n 次方，所以对应的hash&(length-1) length-1 也就是高位多了一个2n 次方的1 ， 因此只是相当于原位置加capacity 的下表
4. 第一次初始化会生成大于初始数组最小的2n 次方的长度数组
```

[HashMap 源码讲解](https://www.cnblogs.com/ITtangtang/p/3948406.html "hashmap 笔记")<br/>
[ConcurrentHashMap 源码讲解](https://www.itqiankun.com/article/concurrenthashmap-principle "ConcurrentHashMap 笔记")

#### JUC 知识
> ConcurrentHashMap

```java
```
> CopyOnWriteArrayList

```java
```

### 线程相关知识
#### 线程状态 与转变
#### 线程间通信
#### 锁synchorized/Lock/CountDownLatch/CycleBarire
### ThreadLocal
[ThreadLocal源码解析](https://www.itqiankun.com/article/1564891332 "ThreadLocal 源码解析")
[ThreadLocal 内存泄露](https://link.zhihu.com/?target=https%3A//juejin.im/post/5e184276e51d4557e86e8afd)
### 线程池
#### 基本参数 及原理
#### AQS
### JVM
#### 对象引用的方式
#### 对象计数方式
#### 垃圾回收算法
#### 垃圾回收器