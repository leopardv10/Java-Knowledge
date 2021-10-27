## *Chapter 1 Java*

---



### Java基础

---------

#### 1.构造器是否能被重写

构造器只能被重载不能被重写，因为构造方法名和类名相同，而子类名和父类名不同，所以子类无法重写父类构造器。

> 重载是什么？
>
> 一个方法，输入不同，做出的处理也不同。比如有参构造器和无参构造器。



#### 2.接口和抽象类区别

- 接口中的方法不能被实现，而抽象类可以有非抽象方法
- 接口只能由static，final修饰的变量，而抽象类则不一定
- 一个类可以实现多个接口，但只能实现一个抽象类。接口本身还可以extends接口



#### 3.equals和hashCode方法

object默认的equals方法调用的是==，即比较的是对象地址，所以两个对象equals返回相同，hahsCode也相同。

> - 为什么修改了equals也要修改hashCode方法？
>
>   所以修改equals后必须修改hashCode，因为如果修改了equals不修改hashCode就会导致，两个对象调用equals方法返回相同，但hashCode却不相同。
>
> - 已经有了equals为什么还要hashcode？
>
>   因为重写的equals（）里一般比较的比较全面比较复杂，这样效率就比较低，而利用hashCode()进行对比，则只要生成一个hash值进行比较就可以了，效率很高。
>
> - 既然hashcode效率更高为什么还要equals？
>
>   因为hashCode()并不是完全可靠，有时候不同的对象他们生成的hashcode也会一样。



#### 4.final关键字

- 修饰类：这个类不能被继承，类中方法都隐式被声明为final

- 修饰变量：修饰基本类型变量时，相当于把变量变为常量；修饰引用类型变量时，则其不能指向另一个对象

  ```java
  final StringBuilder sb = new StringBuilder();
  sb = new StringBuilder(); // 这样会报错
  ```

- 修饰方法：子类不能修改这个方法



#### 5.JMM(Java内存模型)

![img](https://pic3.zhimg.com/80/v2-f36f366c07a6188ea3fdefc794ba021a_1440w.jpg)

Java内存模型规定了所有的**全局变量**都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，然后再刷新到主存。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步进行。

而JMM就作用于工作内存和主存之间数据同步过程。他规定了如何做数据同步以及什么时候做数据同步。

> 为什么局部变量不会出现线程安全问题？
>
> 因为局部变量都存放在线程各自的工作内存里，是线程私有的，只有全局变量被线程共享。



#### 6.public，protected，private

- public所有其它类都能访问；
- protected只能在自己所在的包中被访问，或者其它包中的子类访问；
- private只能在当前类中访问（子类无法继承private方法）；



#### 7.String为什么用final修饰？（还是不太懂）

- 因为String不可变，只会指向新的地址，所以可以实现字符串常量池，节约内存空间，提高效率（创建String a = "abc"时，会先检查字符串常量池中有无"abc"，若有则直接指向字符串常量池中的变量，如果没有就在字符串常量池中新建字符串对象）。
- 因为String对象不可变，所以是线程安全的。



#### 8.Java中的异常处理

在Java中所有异常都有一个共同父类Throwable，Throwable有两个重要的子类Exception和Error。

- Exception：程序本身可以处理的异常。
  - 受检查异常：编译过程中如何没有catch/throw的话无法通过编译，如：IOException。
  - 不受检查异常：RuntimeException，包括NullPointException，IndexOutOfBoundException等。
- Error：程序无法处理的错误，无法用catch捕获。



### 集合

-----------

#### 1.要求线程安全时用什么List

- Vector：所有方法都用synchronized修饰
- SynchronizedList：使用同步代码块的方式，效率比vector高
- CopyOnWriteArrayList：写时加锁，读不加锁



#### 2.ConcurrentHashMap如何实现线程安全

1. JDK1.7：segment分段锁（可重入锁，实现了ReentrantLock），每个segment包含一个HashEntry数组，每个HashEntry是一个链表因此在ConcurrentHashMap查询一个元素的过程需要进行两次Hash操作，如下所示：
   - 第一次Hash定位到Segment
   - 第二次Hash定位到元素所在的链表的头部

<img src="https://segmentfault.com/img/remote/1460000024432654" alt="img" style="zoom:80%;" />

2. JDK1.8：采用CAS机制和synchronized关键字
   - CAS：put数据时首先计算出对应位置，如果当前这个位置的Node为null，则通过CAS方式的方法写入。所谓的CAS，即即compareAndSwap，执行CAS操作的时候，将内存位置的值与预期原值比较，如果相匹配，那么处理器会自动将该位置值更新为新值，否则，处理器不做任何操作。
   - synchronized：当头结点不为null时，则使用该头结点加锁，这样就能多线程去put hashCode相同的时候不会出现数据丢失的问题。

>为什么头节点不为空时用synchronized而不是cas？
>
>假设该位置有一条长度为3的链表4->5->6，线程A插入数据后原本预期链表长度变为4。如果采用cas机制，当线程A插入数据时，可能有线程B删除了链表的最后一个数字6，然后线程C将链表第二个数字5改成了6，这时线程A还是可以插入数据，但插入后的链表长度依旧是3和原本预期的4不一致。



#### 3.ArrayList扩容

- 初始化时如果不指定容量则默认为10。
- 若arraylist容量无法满足新元素的要求则按1.5倍扩容，扩容时用Arrays.copyOf方法将旧数组复制到新数组，若扩容后的容量仍小于最低的存储要求，则取值为最低存储要求。



#### 4.HashMap

- 扩容

  元素数量达到阈值时触发扩容机制。默认初始容量为16，之后每次扩容乘2

- 数据结构

  数组+链表，当链表长度>8时变成红黑树；之所以用红黑树而不是二叉查找树主要是因为后者极端情况下可能出现线性结构。

  > 为何使用拉链法？（可补充）
  >
  > 1. 链表删除结点方便
  > 2. 链表的空间可动态申请，不必事先确定表长

  

- put操作的过程

  首先计算key的hashcode，再用map.size()- 1和 hashcode进行二进制按位与运算出其在数组中对应的位置，如果当前位置存在与插入数据相同的元素就覆盖，不存在则尾插。（头插多线程会出现循环链表）

- 为何长度是2的幂次方？

  一句话，HashMap的长度为2的幂次方的原因是为了减少Hash碰撞，尽量使Hash算法的结果均匀分布。任何一个2的倍数n，n - 1的二进制每一位都是1，这样按位与运算的结果就完全取决于hashcode



### 多线程

-----

#### 1.并发编程三要素（三要素的意义？）

- 原子性：一个或多个操作要么全部执行成功要么全部执行失败。
- 有序性：程序执行的顺序按代码先后顺序执行（可能有指令重排序）。
- 可见性：多个线程访问同一变量时如果一个线程对其进行了修改，其它线程能立刻获取最新值。



#### 2. 线程池参数（调用线程是什么？）

- corePoolSize：线程池保留的最小线程数，若线程池中的线程数 < corePoolSize，则在执行时execute()时创建

- maximumPoolSize：线程池中允许拥有的最大线程数

- keepAliveTime：线程闲置时，保持线程存活的时间

- unit：即为时间单位

- workQueue：工作队列，存放提交的等待任务，有大小限制

  - ArrayBlockingQueue：有界队列
  - LinkedBlockingQueue：无界队列
  - SynchronousQueue：不存储元素，一个put操作必须等待take操作，否则不能添加（保证来一个任务，有闲线程就用空闲线程来执行，否则创建新的线程执行）
  - PriorityBlockingQueue：优先级队列，可设置任务优先级

- handler：拒绝策略，有四种取值

  - AbortPolicy：默认，丢弃任务并抛出异常

  - CallerRunsPolicy：由调用线程执行该任务

  - DiscardPolicy：丢弃任务但不抛出异常

  - DiscardOldestPolicy：丢弃队列最早未处理任务，然后尝试重新执行该任务



#### 3.线程池工作原理

- 如果当前运行的线程数 < corePoolSize，创建新的线程执行任务
- 如果运行的线程数等于或大于corePoolSize，将任务加入任务队列
- 如果队列已满，则在非corePool中创建新的线程
- 如果新创建的线程使当前运行的线程数超过maximumPoolSize，执行handler策略



#### 4.volatile&ThreadLocal关键字

1. volatile: 可以保证变量的可见性，以及防止指令重排序；
2. ThreadLocal: 使线程拥有自己的局部变量；



#### 5.线程状态

- New（新建）
- Runnable（可运行）：调用start方法后，线程进入Runnable状态。但进入Runnable状态的线程可能在运行也可能不在运行。
- Blocked（阻塞）：当一个线程试图获取锁，但锁此时被其它线程占有，该线程就处于阻塞状态。
- Waiting（等待）：线程持有锁，进入了同步代码块，但因为某些原因，比如调用了Object.wait()方法，就会进入等待状态。
- Timed waiting（计时等待）：调用有超时参数的方法后进入计时等待，比如Thread.sleep。
- Terminate：run方法正常退出或异常终止。



#### 6.线程池中的阻塞队列

- ArrayBlockingQueue：固定容量，不能扩容；队列满后就执行拒绝策略而不是无限增加等待线程，可以防止资源耗尽。

- LinkedBlockingQueue：默认容量为Integer.MAX_VALUE；如 FixedThreadPool 的线程数是固定的，在任务激增的时候无法增加更多的线程来处理 Task，所以需要 LinkedBlockingQueue 这样没有容量上限的 Queue 来存储那些还没处理的 Task。（FixedThreadPool使用）

- SynchronousQueue：一个不存储元素的BlockingQueue，每一个put操作必须等待一个take操作，否则不能添加元素（CachedThreadPool所使用）。

- PriorityQueue：支持优先级的阻塞队列。

  ![image-20210514225639525](C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20210514225639525.png)

https://blog.csdn.net/vincent_wen0766/article/details/108593587



#### 7.AQS



#### 8.synchronized锁升级

锁状态：无锁 -> 偏向锁 -> 轻量级锁（自旋锁） -> 重量级锁，且只能升级不能降级。

- 偏向锁：初次执行到synchronized代码块时锁变成偏向锁。没有发生锁竞争，及只有一个线程获取锁时为偏向锁。这种情况下，线程执行完同步代码块时不会释放锁，当该线程再次到达同步代码块时，则不需要重新加锁。
- 轻量级锁：当锁是偏向锁时被其它线程访问，锁就升级为轻量级锁，其它线程会通过自旋的方式来尝试获得锁，线程不会阻塞，从而提升性能。
- 如果锁竞争严重，某个线程自旋次数达到最大值，便会将锁升级为重量级锁，此时等待获取锁的线程会进入阻塞状态。

> 为什么锁不能降级？
>
> 锁升降级效率比较低，如果频繁升降级会对JVM性能造成影响。

https://segmentfault.com/a/1190000022904663



#### 9.synchronized原理（待完善）

- synchronized同步语句块使⽤的是 monitorenter 和 monitorexit 指令，其中 monitorenter指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。当执行monitorenter时线程会试图获取monitor（monitor对象存在于每个Java对象的对象头中）。当计数器为零时就可成功获取，然后计数器+1。执行monitorexist指令时，锁计数器-1，当计数器为0时表示锁被释放。如果获取锁失败线程就会阻塞。
- synchronized修饰同步方法时是通过ACC_SYNCHRONIZED标识来指明该方法是一个同步方法。



#### 10.ReentrantLock

原理：



> ReentrantLock和synchronized区别

1. 两者都是可重入锁
2. synchronized依赖于JVM，而ReentrantLock属于Java中一个类
3. synchronized是非公平锁，而ReentrantLock可指定是公平锁还是非公平锁
4. ReentrantLock需要手动释放锁（unlock）

> 如何选用synchronized和lock？

1. 如果需要创建公平锁就需要使用lock；
2. lock使用更加灵活，比如trylock(3, TimeUnit.SECONDS)方法表示在三秒内线程会不断尝试去获取锁，此外使用使用 lockInterruptibly() 方法可以中断当前获得的锁。如果有更多定制化需求时可以选lock；



#### 11.线程池种类

- newFixedThreadPool

  接收参数为所设定线程数量nThread，corePoolSize为nThread maximumPoolSize为nThread；keepAliveTime为0L(不限时)；unit为：TimeUnit.MILLISECONDS；WorkQueue为new LinkedBlockingQueue。

- newCachedThreadPool

  corePoolSize为0；maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为60L；unit为TimeUnit.SECONDS；workQueue为SynchronousQueue(同步队列)；有空线程就用空线程执行，没有空线程就创建新线程，提高了线程的复用率。

- newSingleThreadExecutor

  corePoolSize为1；maximumPoolSize为1；keepAliveTime为0L；unit为TimeUnit.MILLISECONDS；workQueue为LinkedBlockingQueue；适用于一个任务一个任务执行的场景。

- newScheduledThreadPool

  corePoolSize为传递来的参数，maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为0；unit为：TimeUnit.NANOSECONDS；workQueue为：new DelayedWorkQueue() 一个按超时时间升序排序的队列；适合周期性任务。



#### 12.上下文切换

将一个线程执行时CPU寄存器以及程序计数器里的内容保存到内核中，再将另一个线程的内容加载到进来的过程。

https://zhuanlan.zhihu.com/p/52845869



#### 13.死锁

多个线程在执行过程中，因争夺资源形成的相互等待的过程，若无外力作用它们将永远阻塞下去。

死锁形成四个条件：

- 互斥条件：该资源任一时刻只能被一个线程占用
- 保持条件：一个线程阻塞时不会主动放弃自己持有的资源
- 不剥夺条件：线程已获得的资源在使用完之前不能被其它线程强行释放
- 循环等待：若干线程之间形成一种互相等待的关系



#### 14.sleep()和wait()的区别

- sleep()不会释放锁，而wait()则释放了锁；
- sleep到了时间会自己醒过来，wait则需要notify去唤醒；
- wait一般用于线程间通信，sleep则用于暂停线程执行；



#### 15.乐观锁和悲观锁

- 乐观锁：适用于读多写少的情况，因为这样比较当前值和预期值时发生冲突的情况就比较少，省去了加锁的开销。但如果冲突经常发生就会因为自旋导致性能下降，因此不适合写操作多的场景。
  - MVCC：
  - CAS：
- 悲观锁：
  - Reentrantlock
  - synchronized



#### 16. 并发和并行区别

- 并发:一个处理器同时处理多个任务。
- 并行:多个处理器或者是多核的处理器同时处理多个不同的任务。

--------------

### JVM



#### 1.JVM内存区域

<img src="C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20210415134044316.png" alt="image-20210415134044316" style="zoom:80%;" />

- 堆：存放对象实例，垃圾收集器管理的区域
  - OutOfMemoryError
- 方法区：存储已被虚拟机加载的类型信息、常量、静态变量等
- 本地方法栈：为虚拟机使用到的本地方法服务（第三方语言写的）
  - StackOverFlowError：当线程调用一个方法时，jvm压入一个新的栈帧到这个线程的栈中，如果方法的嵌套调用层次太多(如递归调用),就会出现栈溢出。
- 虚拟机栈：存放方法的信息，操作数（中间变量），方法出口信息等
- 程序计数器：选取下一条执行的字节码指令



#### 2.如何判断对象已经死亡

<img src="C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20210415141416684.png" alt="image-20210415141416684" style="zoom:70%;" />

- 可达性分析：从一系列GC Roots作为起点向下搜索，所走过的路径称为引用链，当一个对象没有和任何引用链相连，即为不可用对象。

- 引用计数：给对象中添加一个计数器，每当被引用时计数器+1，引用失效计数器-1，当计数器为0时该对象即不能再被使用。

  
  
  > 哪些对象可作为GC Roots：
  >
  > 
  >
  > 栈和方法区里引用到的对象



#### 3.各种引用

- 强引用

  String str = new String("hello")，只要强引用在对象就不会被回收；

- 软引用

  软引用关联的对象内存不足时会被回收；

- 弱引用

  每次GC都会被回收；

- 虚引用

  不影响对象的生存时间，唯一的作用是对象被回收时发一个系统通知；



#### 4.垃圾回收算法

- 标志清除
- 标记整理
- 复制算法
- 分代收集



#### 5.类加载过程

1. 定义：举个通俗点的例子来说，JVM在执行某段代码时，遇到了class A， 然而此时内存中并没有class A的相关信息，于是JVM就会到相应的class文件中去寻找class A的类信息，并加载进内存中，这就是我们所说的类加载过程。

   编译即把Java文件编译成字节码文件（.class文件），运行则是把编译生成的.class文件交给JVM运行。而类加载指的是JVM把.class文件中类信息加载进内存并解析生成对应的class对象的过程。

   

2. 步骤

- 加载：把.class文件通过类加载器加载到内存中

- 链接：

  - 验证：保证加载进来的字节流符合虚拟机规范，不会造成安全错误
  - 准备：为类变量（static）分配内存，并赋予初值
  - 解析：将常量池中的符号引用（字符串）替换为直接引用（内存地址）

- 初始化：对类变量（static）进行初始化，是执行类构造器的过程。如果一个类的父类尚未初始化，则优先初始化父类。

  [详见](https://cloud.tencent.com/developer/article/1628085#:~:text=%E7%B1%BB%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B%E5%8F%AA%E6%98%AF%E4%B8%80%E4%B8%AA,%E4%B8%8D%E5%86%8D%E5%A4%9A%E5%81%9A%E8%B5%98%E8%BF%B0%E3%80%82)



#### 6.双亲委派

![preview](https://segmentfault.com/img/bVcHO1F/view)

类加载接收到类加载请求时，首先会请求⽗类加载器去处理，因此所有的请求最终都应该传送到顶层的启动类加载器 BootstrapClassLoader 中。当⽗类加载器无法处理时，才由自己来处理。

好处：

- 因为双亲委派是向上委托加载的，所以它可以确保类只被加载一次，**避免重复加载**；
- Java的核心API都是通过启动类加载器进行加载的，如果别人通过定义同样路径的类比如java.lang.Integer，类加载器通过向上委托，两个Integer，那么最终被加载的应该是jdk的Integer类，而并非我们自定义的，这样就**避免了我们恶意篡改核心包的风险**；



#### 7.什么情况会触发Full GC

- System.gc()方法的调用：调用System.gc()方法会建议JVM进行Full GC，但注意这只是建议，JVM执行不执行是另外一回事儿，不过在大多数情况下会增加Full GC的次数，导致系统性能下降，一般建议不要手动进行此方法的调用，可以通过-XX:+ DisableExplicitGC来禁止RMI调用System.gc。
- 老年代空间不足
- Metaspace区（存放类的元数据）内存不足



#### 8.垃圾收集器

JDK1.8默认使用**Parallel Scanvage** + **Parallel Old**

<img src="https://img-blog.csdnimg.cn/20200304160245276.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzMzY2MjI5,size_16,color_FFFFFF,t_70" alt="img" style="zoom:80%;" />

1. 新生代垃圾收集器：

- Serial: 单线程垃圾回收, 复制算法
- ParNew: 多线程垃圾回收，复制算法
- Parallel Scanvenge: 关注吞吐量，用户代码运行时间和cpu运行总时间比值

2. 老年代垃圾收集器：

- Serial Old:  标记整理

- Paraller Old: 标记整理

- CMS: 初始标记->并发标记->重新标记->并发清除

  ​			CMS使用的是标记清除算法

- ![image-20210715234252055](C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20210715234252055.png)

  > CMS的缺点：
  >
  > 
  >
  > 1. 对CPU资源敏感
  > 2. 无法处理浮动垃圾
  > 3. 使用标记清除有空间碎片

3. G1收集器：



#### 9.内存泄漏

在Java中，**内存泄漏**就是存在一些被分配的对象，这些对象有下面两个特点，**首先**，这些对象是可达的，即**在有向图中，存在通路可以与其相连**；**其次**，**这些对象是无用的，即程序以后不会再使用这些对象**。

```java
void method(){
    //1.对vector的操作
    Vector vector = new Vector();
    for (int i = 1; i<100; i++){
        Object object = new Object();
        vector.add(object);
        object = null;
    }
    //2.下面省略与vector无关的其他操作 
}
```

- 这里的内存泄露指的是操作1完成后，vector里的object对象就不需要了，但是此时如果发生了GC却不会回收这些object对象，因为vector还在引用他们，但这些object对象已经没用了，这种情况下就会发生内存溢出。

```java
void method(){
     //1.对vector的操作
    Vector vector = new Vector();
    for (int i = 1; i<100; i++){
        Object object = new Object();
        vector.add(object);
        object = null;
    }
    vector = null;
    //2.下面是与vector无关的其他操作
}
```

- 这种情况下，vector操作完后将其赋值为null，此时不再有对象引用这些object，所以就避免了内存泄漏的情况。



#### 10.Java对象创建过程（待完善）

![image-20210715221914725](C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20210715221914725.png)

- 类加载检查：虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。
- 分配内存
- 初始化零值
- 设置对象头
- 执行init方法



#### 11.对象从新生代到老年代的过程

![image-20211017175838351](C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20211017175838351.png)

1. 创建对象时优先分配在Eden区，对于大对象则直接进入老年代。
2. 当Eden区空间不够时会触发Minor GC，Eden区的存活对象会进入From Survivor区。
3. From Survivor和To Survivor的存在是因为新生代垃圾回收采用的是复制算法。
4. 对象在Survivor区每熬过一次Minor GC年龄会+1，当达到阈值后就会晋升到老年代中。



#### 12.Full GC排查（待完善）

- 检查JVM配置，堆内存大小，新生代老年代大小
- 观察老年代的内存变化，看FGC后内存是否明显变小，排除内存泄漏
- 通过jmap命令查看堆内存中的对象，观察其实例数，所占内存
- 通过JVisualVM查看堆内存文件，找到其所属的业务对象，再通过代码去排查问题。



#### 13.JVM命令（待完善）

- jps：查询虚拟机进程状况
- jstat：虚拟机统计信息监视
- jinfo：查看虚拟机参数配置
- jmap：生成堆转储快照



## *Chapter 2 MySQL*

------------------

### 1. MyISAM和InnoDB区别

- MyISAM只支持表锁，InnoDB支持行锁和表锁；
- InnoDB支持事务；
- InnoDB支持外键；
- InnoDB支持MVCC；
- InnoDB是聚簇索引，MyISAM是非聚簇索引；



### 2.Explain语句

explain语句显示的字段: id, select_type, table, type, possible_keys, keys, key_len, ref, rows, extra 

- id
  - id相同，执行顺序从上到下。
  - id不同，id越大优先级越高，优先执行。

- select_type：查询类型
  - SIMPLE：查询中不包括子查询或者UNION；
  - PRIMARY：查询中若包括复杂的子查询，最外层的标记为PRIMARY；
  - SUBQUERY：子查询；
  - DERIVED：子查询中生成的临时表；
  - UNION：连接两个以上的 SELECT 语句的结果组合到一个结果集合中。
  - UNION RESULT：UNION的结果。

- <font color=red>**type**</font>：查询类型

  - 8种类型：all，index，range，ref，eq_ref，const，system

  - 从好到坏的顺序：system > const > eq_ref > ref > range > index > all

  - system当table只有一行数据时出现，实际场景几乎不会出现；const表示通过索引一次就找到了，用于直接按主键或唯一键读取；eq_ref和ref都是两表连接时出现的查询类型；range即为范围查询(eg: where 3 < id < 8)；index为全索引扫描；all为全表扫描。

  - eq_ref`: 想象你有两张桌子。表A包含列(id，text)，其中id是主键。表B具有相同的列(id，text)，其中id是主键。表A包含以下数据：

    ```java
    1, Hello 
    2, How are you
    ```

    表B有以下数据：

    ```java
    1, world!
    2, you?
    ```

    想象一下`eq_ref`为A和B之间的JOIN：

    ```mysql
    select A.text, B.text where A.ID = B.ID
    ```

    这个JOIN非常快，因为对于表A中扫描的每一行，表B中只能有一行满足JOIN条件。一个，不超过一个。那是因为B.id是独一无二的

  - ref` : 现在想象另一个带有列(id，text)的表C，其中id是索引但非`UNIQUE`。表C具有以下数据：

    ```java
    1, John!
    1, Jack!
    ```

    想象一下`ref`作为A和C之间的JOIN：

    ```mysql
    select A.text, C.text where A.ID = C.ID
    ```

    此JOIN不如前一个快，因为对于表A中扫描的每一行，表C中有几个可能的行，它们可以满足JOIN条件(上面的循环中没有中断)。那是因为C.ID不是独一无二的。

  - range：范围查询

    ```mysql
    SELECT * FROM tbl_name WHERE key_column BETWEEN 10 and 20;
    ```

  - index：索引全表扫描，把整个索引树全部查一遍

  - possible_keys, keys

    理论上用到的索引，实际用到的索引。

- key_len

  索引字段的最大可能长度，并非实际长度。

- ref

  显示用到了索引中的哪些列。

- rows

  估算查询到结果需要用到的行数，理论上越少越好。

  

### 3.索引



#### 3.1索引分类

- 普通索引：无任何限制

- 唯一索引：索引列的值必须唯一，但允许有null值

- 主键索引：索引列的值必须唯一，且不允许null值

- 组合索引：在多个字段上创建索引，遵循最左前缀原则

- 全文索引

- 另一种分类方式：聚簇索引&非聚簇索引

  MyISAM使用非聚簇索引，叶子结点存放数据记录的地址。

  InnoDB使用聚簇索引，叶子结点包含了完整的数据记录。

  对于InnoDB，非主键索引在叶子节点存储的是主键的值，然后用主键到主键索引中获取数据。
  
  

#### 3.2 创建索引的原则

- 最左前缀匹配原则
- 为经常需要排序（索引已经排序），分组，查询以及联合查询（加快表的连接速度）的列做索引
- 选择唯一索引，唯一索引的值是唯一的，可以更快地确定某条记录
- 不要让索引列参与计算，否则索引会失效
- 限制索引的数量，因为更新表时也要更新索引
- 如果索引字段很长最好使用值的前缀作为索引

> B+树总结：
>
> - B+树非叶子结点只存储索引。不存储数据，所以单一节点存储元素更多，树高度更矮，IO次数更少，因此查询效率更高；
>
> - 所有查询都要查找到叶子结点，因此查询性能稳定。而B树每个结点都可能查询到数据，所以性能不稳定；
>
> - 叶子结点通过双向链表连接，所以范围查询很快；
>
>   https://segmentfault.com/a/1190000020416577
>
>   https://www.cxyxiaowu.com/3726.html



#### 3.3 索引失效的情况（待完善）

1. 模糊匹配使用通配符（“%abc”）开头
2. 索引列参与计算
3. 没遵循最左匹配原则



### 4.事务

#### 4.1四大特性

ACID：原子性（Atomicity），一致性（Consistency），隔离性（Isolation），持久性（Durability）

- 原子性：事务是最小的执行单位，不允许分割。事务中的所有命令要么全部执行成功，要么全部执行失败；
- 一致性：？？？？
- 隔离性：并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
- 持久性： 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。



#### 4.2并发事务带来的问题

- 脏读：一个事务读取了另一个事务尚未提交的数据；
- 幻读：A事务的执行期间，事务B对事务A访问过的表中**插入或删除**了某一行，导致事务A第二次访问该表时读到了新插入的行，或者少了一行；
- 不可重复读：A事务的执行期间，事务B对事务A访问过的表中**修改**了某一行，导致事务A第二次访问同一行时读到的数据不一样；



#### 4.3数据库隔离级别

- 读未提交

  - 一个事务还没提交，它所做的修改就能被别的事务看到

  - 会出现脏读、幻读、不可重复读
  - 只在写数据时加行级共享锁

- 读已提交

  - 一个事务提交后，它所做的修改才能被别的事务看到
  - 会出现幻读、不可重复读
  - 写数据时加行级排他锁，读数据时加行级共享锁

- 可重复读

  - 一个事务在执行过程中看到的数据跟事务刚开始时看到的数据保持一致
  - 会出现幻读
  - 写数据时加行级排他锁，读数据时加行级共享锁+MVCC

- 可串行化

  - 加表锁或MVCC&间隙锁
  - 避免了脏读、幻读、不可重复读的发生

> MVCC（多版本并发控制）介绍
>
> https://blog.csdn.net/qq_44836294/article/details/108059551?utm_medium=distribute.pc_relevant.none-task-blog-searchFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-searchFromBaidu-1.control



### 5.锁

#### 5.1按读写划分

- 排它锁（写锁）：事务对数据添加排它锁后，其它事务不能对该数据添加读锁或写锁
- 共享锁（读锁）：事务对数据添加共享锁后，其它事务只能对该数据添加读锁而不能添加写锁

> 加写锁可以避免脏读，那么加读锁的作用是什么？
>
> 避免读到写操作的中间状态。



#### 5.2按粒度划分

- 表锁：开销小，加锁快，不会出现死锁，并发度最低
- 行锁：开销大，加锁慢，会出现死锁，并发度最高
- 页锁：应用于BDB引擎，开销和加锁时间介于行锁和表锁之间，会出现死锁，并发度一般

> 为何表锁开销小，加锁快？
>
> 如果要锁多条记录，对每条记录依次加锁开销比直接锁整张表更大。即使只锁一行还是要先找到对应的那一行，所以说行锁开销大、加锁慢。



#### 5.3行锁分类

- 间隙锁

- 记录锁

- 临键锁：InnoDB行锁默认算法，可以理解为记录锁和间隙锁的结合。不仅锁住查询出来的记录，还会所住该查询范围内的所有间隙，同时会锁住下一个相邻区域。

  <img src="C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20210415225740657.png" alt="image-20210415225740657" style="zoom:80%;" />

  例如select * from table where id > 1 and id < 7；会锁住id = 1, 3, 6a和(1, 3), (3, 6)以及(6, 10)



### 6.主从复制





### 7. SQL语句的执行过程

1. 连接器，负责与客户端通信
2. 查询缓存，检查之前是否执行过该语句，缓存里有无结果
3. 缓存未命中的话执行分析器，分析语法错误
4. 优化器，决定执行顺序，索引如何选择等
5. 执行器



### 8.drop, truncate, delete区别

- drop是直接删除表
- truncate是删除表中所有数据，且无法恢复
- delete可以加where语句



### 9.慢查询排查

- 开启慢查询日志记录慢查询信息；
- mysqldumpslow来分析慢查询日志；
- explain语句显示sql语句的执行情况；

可能原因：

- 没有用到索引
- 查询了不必要的列
- 查询的数据量过大
- 查询语句不好，没有优化
- innodb在刷脏页
- 死锁



### 10.分库分表（待补充）

分表

1. 垂直分表

   表中的字段较多，一般将不常用的、 数据较大、长度较长的拆分到“扩展表“。一般情况加表的字段可能有几百列，此时是按照字段进行数竖直切。注意垂直分是列多的情况。

2. 水平分表

   单表的数据量太大。按照某种规则（RANGE,HASH取模等），切分到多张表里面去。 但是这些表还是在同一个库中，所以库级别的数据库操作还是有IO瓶颈。这种情况是不建议使用的，因为数据量是逐渐增加的，当数据量增加到一定的程度还需要再进行切分。比较麻烦。

分库

1. 垂直分库

   一个数据库的表太多。此时就会按照一定业务逻辑进行垂直切，比如用户相关的表放在一个数据库里，订单相关的表放在一个数据库里。注意此时不同的数据库应该存放在不同的服务器上，此时磁盘空间、内存、TPS等等都会得到解决。

2. 水平分库

   水平分库理论上切分起来是比较麻烦的，它是指将单张表的数据切分到多个服务器上去，每个服务器具有相应的库与表，只是表中数据集合不同。 水平分库分表能够有效的缓解单机和单库的性能瓶颈和压力，突破IO、连接数、硬件资源等的瓶颈。
   
   

### 11. redo log, undo log, bin log



## *Chapter 3 Redis*

---



### 1.Redis 为什么快

- 完全基于内存，绝大部分请求是纯粹的内存操作，非常快速；
- 采用单线程，避免了CPU上下文切换，加锁释放锁的消耗，也不会出现死锁的情况；
- 使用 I/O多路复用模型，非阻塞 IO；



###  2.Redis多线程

- 我们所说的Redis单线程，指的是"其网络IO和键值对读写是由一个线程完成的"，也就是说，**Redis中只有网络请求模块和数据操作模块是单线程的。而其他的如持久化存储模块、集群支撑模块等是多线程的。**

- Redis6.0针对处理网络请求过程采用了多线程，而数据的读写命令，仍然是单线程处理的。

  https://www.cnblogs.com/hollischuang/p/14535826.html



### 3.Redis作用

- 缓存
- 排行榜：很多网站都有排行榜应用的，如京东的月度销量榜单、商品按时间的上新排行榜等。Redis提供的有序集合数据类构能实现各种复杂的排行榜应用。
- 计数器：如电商网站商品的浏览量、视频网站视频的播放数等。为了保证数据实时效，每次浏览都得给+1，并发量高时如果每次都请求数据库操作无疑是种挑战和压力。Redis提供的incr命令来实现计数器功能，内存操作，性能非常好，非常适用于这些计数场景。
- 分布锁：可以利用Redis的setnx功能来编写分布式的锁，如果设置返回1说明获取锁成功，否则获取锁失败，实际应用中要考虑的细节要更多。
- 消息队列：Redis提供了发布/订阅及阻塞队列功能，能实现一个简单的消息队列系统。另外，这个不能和专业的消息中间件相比。



### 4.Redis持久化机制

Redis持久化就是将内存中的数据写入到磁盘，防止服务宕机导致内存丢失。

- RDB：快照持久化（默认）：fork一个子进程将数据写入到临时文件中，再替换之前的rdb文件。通过redis.conf文件中的save参数调节快照的周期。缺点：RDB 是间隔一段时间进行持久化，如果持久化之间 redis 发生故障，会发生数据丢失。所以这种方式更适合数据要求不严谨的时候)

  <img src="C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20210411231845065.png" alt="image-20210411231845065" style="zoom:70%;" />

- AOF（append only file）：将Redis执行的每次写命令记录到单独的日志文件中，当重启Redis会重新将持久化的日志中文件恢复数据。当两种方式同时开启时，数据恢复Redis会优先选择AOF恢复。Redis配置文件中存在三种不同的AOF持久化方式，分别是：

  <img src="C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20210411232734738.png" alt="image-20210411232734738" style="zoom:80%;" />



### 5.Redis处理过期key

通过EXPIRE指令可以设置redis过期时间，比如一般项目中的token或者登录信息，短信验证码等，都有时间限制。Redis如何处理过期的key：

- 定期删除：每隔100ms随机抽取一批设置了expire time的key，检查其是否过期，过期及删除。

- 惰性删除：需要访问一个key的时候才判断其是否过期，过期及删除。



### 6.Redis内存淘汰策略

当**内存不足**时，redis会执行内存淘汰策略；

六种内存淘汰策略：

- volatile-lru：在设置了过期时间的keys中，移除最近最少使用的key。（最长时间未被使用）
- volatile-random：在设置了过期时间的keys中，随机移除某个key。
- volatile-ttl：在设置了过期时间的keys中，有更早过期时间的key优先移除。
- noeviction：新写入操作会报错。
- allkeys-lru：在所有keys中，移除最近最少使用的key。（这个是**最常用**的）
- allkeys-random：在所有keys中，随机移除某个key。

4.0后新增两种淘汰策略：

- volatile-lfu：在设置了过期时间的keys中，移除最不经常使用的key。（一定时间内用的最少）
- allkeys-lfu：在所有keys中，移除最不经常使用的key。



### 7.Redis事务

- 事务开始用MULTI指令，接着命令入队，事务执行EXEC指令。

- Redis事务不保证原子性，没有回滚，一条命令执行失败，其余命令仍然执行；但如果EXEC指令执行前，某一条指令出现语法错误等，则所有命令都不会执行。

- Redis事务功能是通过MULTI、EXEC、DISCARD和WATCH 四个原语实现的。

- WATCH 命令是一个乐观锁，可以为 Redis 事务提供CAS行为。 可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行，监控一直持续到EXEC命令；通过调用DISCARD，客户端可以清空事务队列，并放弃执行事务， 并且客户端会从事务状态中退出。

  （`Watch` 命令是`Exec`命令的执行条件；也就是说，如果Watch的Key没有被修改则Redis执行事务，否则（Watch的key被其他事务修改了）事务不会被执行。）



### 8.Redis集群模式

#### 8.1 主从复制

数据库分为两类，一类是主数据库（master），另一类是从数据库(slave）。主数据库负责写操作，当写操作导致数据变化时会自动将数据同步给从数据库。而从数据库一般是只读的，并接受主数据库同步过来的数据。一个主数据库可以拥有多个从数据库，而一个从数据库只能拥有一个主数据库。

好处：读写分离，分担主数据库压力；方便做容灾恢复。

#### 8.2 哨兵模式

主从复制的模式，当主服务器宕机后，需要手动把一台从服务器切换为主服务器，费事费力，还会造成一段时间内服务不可用。因此更多时候优先考虑哨兵模式。

**哨兵是一个独立运行的进程。原理是哨兵向Redis服务器发送命令，等待其响应，从而监控运行的多个 Redis 实例是否运行正常**。当哨兵监测到 master 宕机，会自动将 slave 切换成 master ，然后通过**发布订阅模式**通知其他的从服务器，修改配置文件，让它们切换主机；

一个哨兵进程对Redis服务器进行监控，也可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式。

> 发布订阅模式：
>
> 发布者只需告诉Broker，我要发的消息，topic是AAA；
>
> 订阅者只需告诉Broker，我要订阅topic是AAA的消息；
>
> 于是，当Broker收到发布者发过来消息，并且topic是AAA时，就会把消息推送给订阅了topic是AAA的订阅者。当然也有可能是订阅者自己过来拉取，看具体实现。
>
> **也就是说，发布订阅模式里，发布者和订阅者，不是松耦合，而是完全解耦的。**

#### 8.3 Cluster模式

Redis集群有16384个哈希槽，每个 key 通过 CRC16 校验后对 16384 取模来决定放置哪个槽。集群的每个节点负责一部分hash槽，举个例子，比如当前集群有3个节点，那么：

- 节点 A 包含 0 到 5460 号哈希槽
- 节点 B 包含 5461 到 10922 号哈希槽
- 节点 C 包含 10923 到 16383 号哈希槽

这种结构很容易添加或者删除节点。如果我想新添加个节点 D ， 我需要从节点 A， B， C 中得部分槽到 D 上。如果我想移除节点 A ，需要将 A 中的槽移到 B 和 C 节点上，然后将没有任何槽的 A 节点从集群中移除即可。

在 Redis 的每一个节点上，都有这么两个东西，一个是插槽（slot），它的的取值范围是：0-16383。还有一个就是 cluster，可以理解为是一个集群管理的插件。我们的存取的 Key随机到达一个Redis节点时，Redis 会根据 CRC16 的算法得出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，通过这个值去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作。



### 9. Redis分布式锁

Redis为单进程单线程模式，采用队列模式将并发访问变成串行访问，且多客户端对Redis的连接并不存在竞争关系Redis中可以使用SETNX（set if not exists）命令实现分布式锁。

eg: setnx key value

<Integer> 1

返回1代表设置成功，返回0则失败；

**解锁**：DEL key

**设置超时时间**：EXPIRE key timeout, 设置 key 的超时时间，以保证即使锁没有被显式释放，锁也可以在一定时间后自动释放，避免资源被永远锁住。



### 10. 并发竞争key

分布式锁，zookeeper和redis都可以实现。



### 11. 缓存雪崩、穿透与击穿

#### 1.缓存雪崩

> 概念

缓存同一时间大面积失效或者服务器宕机，后面的请求全落到数据库上，造成数据库短时间内承受大量请求而崩掉。

> 解决方案

- 搭建redis集群。
- 限流：通过加锁或者队列控制数据库的写缓存的线程数量，比如对于某个key只允许一个线程查询和写缓存；
- 服务降级：停掉一些不重要的服务，缓解系统压力，保证主要服务的运行。

- **数据预热**：将热点数据提前部署到缓存中，并设置不同的过期时间。可以避免活动刚开始时用户的请求先查询数据库的问题。

#### 2.缓存穿透

> 概念

数据在redis和数据库中都不存在时，如果有很多用户同时查询该数据，请求就会全部落到数据库上。

> 解决方案

- 布隆过滤器：判断用户请求的key是否存在，不存在直接返回错误信息给客户端。
- 缓存空对象：将无效的key添加到缓存中去，但是要设置过期时间防止redis中存储大量无效key。

#### 3.缓存击穿

> 概念

对于某个热点数据key，在不停承受着高并发访问时，突然缓存失效，这一瞬间所有的访问都会直接请求到数据库上。

> 解决方案

- 不给热点数据设置过期时间。
- 使用分布式锁，保证对于某个key的请求同时只有一个线程去查询后端服务。



### 12. 缓存与数据库的双写一致性

先更新数据库，再写缓存。若先更新缓存，再更新数据库可能会出现脏读。



### 13. Redis五大数据结构的底层实现



#### 1.zset数据结构

[跳表](https://www.jianshu.com/p/9d8296562806)



#### 2.string数据结构

简单动态字符串（simple dynamic string，SDS）

```c
struct sdshdr {

    // 记录 buf 数组中已使用字节的数量
    // 等于 SDS 所保存字符串的长度
    int len;

    // 记录 buf 数组中未使用字节的数量
    int free;

    // 字节数组，用于保存字符串
    char buf[];
};
```

- SDS 获取字符串长度时间复杂度O(1)：因为 SDS 通过 len 字段来存储长度，使用时直接读取就可以；
- SDS 能杜绝缓冲区的溢出：因为当 SDS API 要对 SDS 进行修改时，会先检查 SDS 的空间是否足够，如果不够的话 SDS 会自动扩容。扩容时如果所需长度小于1M则两倍扩容，大于1M则直接扩容1M。



#### 3.hash实现

- ziplist
  1. 哈希对象保存的所有键值对的键和值的字符串长度都小于 `64` 字节；
  2. 哈希对象保存的键值对数量小于 `512` 个；

- hashtable

http://redisbook.com/preview/object/hash.html



#### 4.set实现

- intset

  实际上，intset是一个由整数组成的**有序集合**，从而便于在上面进行二分查找，用于快速地判断一个元素是否属于这个集合。

  使用intset的前提：

  - 如果能够转成int的对象（isObjectRepresentableAsLongLong），那么就用intset保存。
  - 如果用intset保存的时候，如果长度超过512（REDIS_SET_MAX_INTSET_ENTRIES）就转为hashtable编码。

- hashtable

  

#### 5.list实现

- ziplist

- linkedlist

  使用linkedlist的前提：

  - 试图往列表新添加一个字符串值，且字符串的长度超过 `server.list_max_ziplist_value` （默认值为 `64` ）。
  - `ziplist` 包含的节点超过 `server.list_max_ziplist_entries` （默认值为 `512` ）。



### 14. Redis的IO多路复用

- ##### 网络模块通过单个线程监控多个网络请求状态，直到有数据可读可写时，才去通知操作模块的线程去处理数据。

https://zhuanlan.zhihu.com/p/115912936



## Chapter 4 计算机网络

-----

### 1.TCP协议

#### 1.1三次握手，四次挥手

SYN表示建立连接，FIN表示关闭连接，ACK表示响应，

<img src="C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20210416184339452.png" alt="image-20210416184339452" style="zoom:70%;" />

<img src="C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20210416184514700.png" alt="image-20210416184514700" style="zoom:75%;" />



#### 1.2四次挥手后客户端为什么进入TIME-WAIT状态

- 第四次挥手的ACK报文可能会丢失，这时服务器会重新断开请求，所以客户端此时还不能关闭连接
- 让本次连接存留在网络里的报文全部消失，防止下次连接受到本次连接报文的影响

> 为什么TIME-WAIT是2msl？
>
> msl是报文在网络中最大存活时间，若第四次挥手客户端发送的ACK没送到服务端它最多滞留1msl，随后服务端会重传FIN报文，其在网络中的最大滞留时间也是1msl，一共2msl。



#### 1.3TCP协议如何保证可靠传输

- 应用数据被分割成TCP认为最适合发送的数据块，并对每个数据包编号，让接收方接收到有序数据

- 通过滑动窗口实现流量控制

- 拥塞控制：当网络拥塞时减少数据的发送

- 超时重传：每发完一个分组就停止发送，等待对方确认，收到确认后再发送下一个分组

  

#### 1.4拥塞控制

- 慢开始

  刚开始建立连接时发送窗口大小为1，然后每次乘2增大窗口大小

- 拥塞避免

  当发送窗口大小达到一个阈值后，窗口大小每次+1

- 快重传

  假设接收端没有收到2号报文段，发送方只要一连收到三个2号报文的重复确认就立即重传2号报文，而不必等待重传计时器到期

- 快恢复

  若出现超时重传或快重传，则发送窗口阈值减半，然后以新的阈值为起点，开始拥塞避免算法

  ![TCP流量控制和拥塞控制简述1](https://res-static.hc-cdn.cn/fms/img/f95966316b30037b3b2e37bb2ce641051603799679983.png)



#### 1.5滑动窗口

https://blog.csdn.net/wdscq1234/article/details/52444277

https://www.bilibili.com/video/BV1FE411C7dk?from=search&seid=16626600627621038598

-------

### 2.OSI七层协议

<img src="C:\Users\leopa\AppData\Roaming\Typora\typora-user-images\image-20210416191124914.png" alt="image-20210416191124914" style="zoom:80%;" />

- 应用层：定义应用进程间的通信和交互规则，包括域名系统DNS，HTTP协议等；
- 表示层：数据格式转换服务，
- 会话层
- 传输层：建立端口到端口的通信，包括TCP和UDP协议等；
- 网络层：
- 数据链路层：通过一条链路从一个结点将另一个结点传送数据；
  - 功能：组帧，链路管理，流量控制等。
- 物理层

-----

### 3.DNS解析流程

本地DNS服务器->DNS根服务器->域服务器->域名解析服务器

- 客户端向本地DNS服务器发起请求；
- 本地DNS服务器查询缓存，若有直接返回，若无向DNS根服务器发起请求；
- 根服务器告诉本地DNS服务器相应的域服务器的地址；
- 本地DNS服务器向域服务器发起请求，域服务器会给出相应的域名解析服务器的地址；
- 本地DNS服务器向域名解析服务器发起请求，得到IP地址，返回给客户端并将其保存在缓存中；

-----------------

### 4.HTTP

#### 4.1HTTP请求格式

- 请求方法+协议版本
- 请求头：
  - User-Agent: 生成请求的浏览器类型
  - Accept: 客户端可识别的响应内容的类型
  - Accept-Language: 客户端可接受的自然语言
  - Accept-Encoding: 客户端可接受的编码压缩格式
  - Host: 请求的主机名
  - Connection: 连接方式，keep-alive或close
- 请求体



#### 4.2HTTP请求过程

- DNS解析
- 建立连接
- 发送HTTP请求
- 服务端处理请求返回HTML
- 浏览器解析并显示
- 连接结束



#### 4.3HTTPS

HTTPS是建立在SSL/TLS协议之上的HTTP协议

> 加密过程

- 客户端发起https请求
- 服务端返回证书（公钥）
- 客户端生成随机对称密钥
- 使用服务端的证书对生成的对称密钥加密
- 发送加密后的对称密钥给服务器
- 服务器使用自己的私钥对其解密
- 双方都拥有对称密钥，便可以用其对文件加密传输

https://segmentfault.com/a/1190000014303224

https://zhuanlan.zhihu.com/p/96494976



#### 4.4put和post方法区别

- put请求用来传输文件
- post一般传输内容，等后端服务解析后返回

-----

### Socket

#### 1.什么是socket？

socket本质上就是对 TCP/IP 的运用进行了一层封装，然后应用程序直接调用 socket API 即可进行通信。那么它是如何工作的呢？它分为 2 个部分，服务端需要建立 socket 来监听指定的地址，然后等待客户端来连接。而客户端则需要建立 socket 并与服务端的 socket 地址进行连接。

<img src="https://pic1.zhimg.com/v2-c5c1adef314050e1a524b4612e9d7c98_r.jpg?source=1940ef5c" alt="preview" style="zoom:67%;" />

-----

### 五种IO模型

- 同步IO：
  - 阻塞IO：BIO，最常用；在调用read()/recvfrom（）函数时，导致应用程序阻塞；若数据没有准备好，就什么都不干一直等待，直到数据准备就绪了就搬迁（从内核把读到的数据拿到用户区）在网络上传输时：等待的时间长，因为是远距离传输；在本地上传输时：不需要等
  - 非阻塞IO：NIO，浪费CPU资源，一般不常用；把一个SOCKET接口设置为非阻塞，就是告诉内核，当请求的I/O操作无法完成时，不要将进程睡眠，而是返回一个错误。通过I/O操作函数recvfrom（）不断地测试数据是否准备好，如果没有准备好，继续轮询测试，直到数据准备好，从内核拿到用户空间，I/O函数返回成功。
  - IO多路复用：单个线程通过记录跟踪多个IO流状态，知道有数据可读可写时，才去调用IO操作函数。
  - 信号驱动IO：在信号驱动IO模型中，当用户线程发起一个IO请求操作，会给对应的socket注册一个信号函数，然后用户线程会继续执行，当内核数据就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在信号函数中调用IO读写操作来进行实际的IO请求操作。
- 异步IO：用户进程进行aio_read系统调用之后，就可以去处理其他的逻辑了，无论内核数据是否准备好，都会直接返回给用户进程，不会对进程造成阻塞。等到数据准备好了，内核直接复制数据到进程空间，然后从**内核向进程发送通知**，此时数据已经在用户空间了,可以对数据进行处理了。

详见：https://www.jianshu.com/p/486b0965c296



## Chapter 5 框架

--------

### Spring



#### 1.IOC

1. 概念

   Class A要调用Class B，常规做法要在A中new出B对象，但通过IOC将创建B对象的过程交给了**IOC容器**，然后直接注入A中**（依赖注入/控制反转）**，而不需要在Class A中进行。

2. 优点

- 依赖注入：将**被调用类**作为参数传入**调用类**，实现了上层类对下层类的控制。

  https://www.zhihu.com/question/23277575

- IOC容器：假设B中调用了C，C中调用了D，如果没有IOC容器，我们需要先new D然后将D传入C再将C传入B，再将B传入A，这样需要写大量new。有了IOC容器这一过程直接交给IOC容器而不用自己手写。

3. 底层实现

   反射，工厂模式

------------------------------

#### 2.AOP

1. 概念

   运行时动态地将代码切入到类的指定方法，指定位置上的编程思想。

2. 优点

   增强原有方法，减少重复代码。

3. 底层实现

   JDK动态代理（接口），CGLIB动态代理（继承）

--------------------------------------

#### 3.Spring事务



##### 3.1 Spring事务实现

spring 中的事务实现从原理上说比较简单，通过 AOP在方法执行前后增加数据库事务的操作。

1. 在方法开始时判断是否开启新事务，需要开启事务则设置事务手动提交 set autocommit=0;
2. 在方法执行完成后手动提交事务 commit;
3. 在方法抛出指定异常后调用 rollback 回滚事务;



##### 3.2 Transactional注解的使用

直接在类或方法上添加@Transactional标签。

rollbackfor参数：默认只有RunTimeException时才会回滚，设置为rollbackFor=Exception.class则可以在非运行时异常也能回滚。



##### 3.3 事务的传播机制（待完善）

- 传播机制生效条件：

  因为 spring 是使用 aop 来代理事务控制 ，是针对于接口或类的，所以在同一个类中两个方法的调用，传播机制是不生效的。

- 传播机制类型：

  1. ### PROPAGATION_REQUIRED (默认)

     - 支持当前事务，如果当前没有事务，则新建事务
     - 如果当前存在事务，则加入当前事务，合并成一个事务

  2. ### REQUIRES_NEW

     - 新建事务，如果当前存在事务，则把当前事务挂起
     - 这个方法会独立提交事务，不受调用者的事务影响，父级异常，它也是正常提交

  3. ### NESTED

     - 如果当前存在事务，它将会成为父级事务的一个子事务，方法结束后并没有提交，只有等父事务结束才提交
     - 如果当前没有事务，则新建事务
     - 如果它异常，父级可以捕获它的异常而不进行回滚，正常提交
     - 但如果父级异常，它必然回滚，这就是和 `REQUIRES_NEW` 的区别

  4. ### SUPPORTS

     - 如果当前存在事务，则加入事务
     - 如果当前不存在事务，则以非事务方式运行，这个和不写没区别

  5. ### NOT_SUPPORTED

     - 以非事务方式运行
     - 如果当前存在事务，则把当前事务挂起

  6. ### MANDATORY

     - 如果当前存在事务，则运行在当前事务中
     - 如果当前无事务，则抛出异常，也即父级方法必须有事务

  7. ### NEVER

     - 以非事务方式运行，如果当前存在事务，则抛出异常，即父级方法必须无事务

> Q1:A方法未添加事务，B方法添加了事务，A调用B事务会生效吗？

|          | A有无事务注释 | B有无事务注释 | A异常    | B异常          |
| -------- | ------------- | ------------- | -------- | -------------- |
| AB同类   | 有            | 无            | AB回滚   | AB回滚         |
| AB同类   | 无            | 有            | AB不回滚 | AB不回滚       |
| AB不同类 | 有            | 无            | AB回滚   | AB回滚         |
| AB不同类 | 无            | 有            | AB不回滚 | A不回滚，B回滚 |

> Q2:final, private, static方法添加事务标签不生效。



##### 3.3 Spring事务隔离级别

1. DEFAULT，这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别.
2. 读未提交，（read uncommited） ：脏读，不可重复读，幻读都有可能发生
3. 读已提交，（read commited）：避免脏读。但是不可重复读和幻读有可能发生
4. 可重复读，（repeatable read） ：避免脏读和不可重复读.但是幻读有可能发生
5. 可串行化，（serializable） ：避免以上所有读问题



#### 4. RestController和Controller

单独使用@Controller注解而不加@ResponseBody返回是一个视图，@RestController返回json或xml数据



#### 5. Spring bean的作用域

- singleton：Spring中的bean默认都是单例的
- prototype：每次请求创建一个新的实例
- request：每次HTTP请求创建一个实例，该实例仅在当前HTTP request有效
- session：每次HTTP请求创建一个实例，该实例仅在当前session有效



#### 6. Spring注入bean的方式

1. xml注入
   - set方法注入
   - 构造函数注入
   - 工厂方法注入
2. 注解注入
   - @Autowired
   - @Resource
   - @Required



#### 7. Spring Bean的线程安全问题

Spring的Bean默认是单例模式，因此当存在全局变量时是存在线程安全问题的。

解决方法：

1. 使用prototype作用域。
2. 使用TreadLocal来定义成员变量



#### 8. Spring如何解决循环依赖？





#### 9. Spring Bean的生命周期（待完善）

大致过程：

1. 实例化：创建实例对象，分配内存空间
2. 属性赋值：如通过@Value注解为属性赋值
3. 初始化：注入依赖
4. 销毁



### SpringMVC

------

#### 1. MVC是什么？

MVC的全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，是一种软件设计典范。它是用一种业务逻辑、数据与界面显示分离的方法来组织代码，将众多的业务逻辑聚集到一个部件里面，在需要改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑，达到减少编码的时间。

V即View视图是指用户看到并与之交互的界面。比如由html元素组成的网页界面，或者软件的客户端界面。MVC的好处之一在于它能为应用程序处理很多不同的视图。在视图中其实没有真正的处理发生，它只是作为一种输出数据并允许用户操纵的方式。

M即model模型是指模型表示业务规则。在MVC的三个部件中，模型拥有最多的处理任务。被模型返回的数据是中立的，模型与数据格式无关，这样一个模型能为多个视图提供数据，由于应用于模型的代码只需写一次就可以被多个视图重用，所以减少了代码的重复性。

C即controller控制器是指控制器接受用户的输入并调用模型和视图去完成用户的需求，控制器本身不输出任何东西和做任何处理。它只是接收请求并决定调用哪个模型构件去处理请求，然后再确定用哪个视图来显示返回的数据。



#### 2. SpringMVC的执行流程

![img](https://upload-images.jianshu.io/upload_images/5220087-3c0f59d3c39a12dd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1002/format/webp)



----------------------

### MyBatis



#### 1. MyBatis如何防止sql注入？

- MyBatis启动了sql预编译功能，在sql执行前会先将sql语句发送给数据库进行编译；执行时直接用入参替换掉预编译好的“？”占位符即可，这样即使入参是敏感字段，如“or '1' == '1' ”也只是作为一个参数传入。sql注入只对编译过程起作用。



-----

### SpringCloud





## Chapter 6 Linux

----

### 1.常见命令

#### 1.1 tail

- 常用查看日志文件。

- 常用参数：

  -f 循环读取

> tail -f app.log



#### 1.2 find

- 沿着文件层次结构向下遍历，匹配符合条件的文件，并执行相应的操作。

- 常用参数：

  -name 按照文件名查找文件

> 找到当前目录下所有的java文件
>
> find . -name "*.java"



#### 1.3 tar

- 压缩解压缩文件

- 常用参数：

  -z 解压gzip文件

  -c 建立新的压缩文件

  -x 从压缩文件中提取文件

  -v显示操作过程

  -f指定压缩文件

> tar -zcvf test.tar.gz test.log  # 打包且以gzip压缩
>
> tar -zxvf test.tar.gz  # 解压



#### 1.4 chmod

- 用于改变linux系统文件或目录的访问权限。

- 常用参数：

  -R 对目前目录下的所有文件与子目录进行相同的权限变更

  7 读 + 写 + 执行，每三位为一组，它们分别是文件所有者（User）的读、写、执行，用户组（Group）的读、写、执行以及其它用户（Other）的读、写、执行。

> chmod -R 777 app.jar



#### 1.5 top

- 显示当前系统正在执行的进程的相关信息，包括进程ID、内存占用率、CPU占用率等。

- 常用参数：

  -i<时间> 设置间隔时间

  -u<用户名> 指定用户名

  -p<进程号> 指定进程

  -n<次数> 循环显示的次数



#### 1.6 free

- 显示系统使用和空闲的内存

- 常用参数：

  -b  以Byte为单位显示内存使用情况

  -k  以KB为单位显示内存使用情况

  -m  以MB为单位显示内存使用情况

  -g 以GB为单位显示内存使用情况



#### 1.7 lsof

- 查看进程打开的文件，打开某文件的进程，进程打开的端口

- 常用参数：

  -a 列出打开文件的进程

  -p 进程号 列出指定进程打开的文件



#### 1.8 ifconfig

- 查看和配置网络设备。



#### 1.9 ping

- 检查网络可用性



#### 1.10 netstat



#### 1.11 telnet

- 远程登录



#### 1.12 ps显示进程状态



## Chapter 7 设计模式

----

### 工厂模式

好处：

- 解耦：将对象创建和使用过程分开。
- 降低重复代码，方便维护。

#### 1.简单工厂模式

```java
// 动物工厂
public class AnimalFactory {
    // 创建dog
    public static Dog createDog() {
        return new Dog();
    }
    
    // 创建cat
    public static Cat createCat() {
        return new Cat();
    }
}

Cat cat = AnimalFactory.createCat();
Dog dog = AnimalFactory.createDog();
```

缺点：需要创建别的对象时要修改代码

优点：只需要一个工厂来创建对象，代码量少



#### 2.工厂方法模式

**开闭原则**：对扩展开放，对修改关闭。

```java
// 抽象工厂
public interface AnimalFactory {
    Animal createAnimal();
}

// cat工厂
public class CatFactory implements AnimalFactory {
    @Override
    public Aniaml createAnimal() {
        return new Cat();
    }
}

// dog工厂
public class DogFactory implements AnimalFactory {
    @Override
    public Aniaml createAnimal() {
        return new Dog();
    }
}

public abstract class Animal {
    public abstract void eat();
}

public class Cat extends Animal {
    @Override
    public void eat() {
        System.out.println("猫吃鱼");
    }
}

public class Dog extends Animal {
    @Override
    public void eat() {
        System.out.println("狗吃骨头");
    }
}
```

优点：相对简单工厂模式，增加新的类时职需扩展出一个新的工厂类即可，而不需要修改原来的代码

缺点：每一个对象需要一个单独的工厂



#### 3.抽象工厂模式

简单工厂模式和工厂方法模式都只能创建一类对象->Animal，但如果现在我有多类对象该怎么办？

```java
// PC类
public interface PC {
    void make();
}

// 小米PC
public class MiPC implements PC {
    public MiPC() {
        this.make();
    }
    @Override
    public void make() {
        System.out.println("make xiaomi PC!");
    }
}

// MAC PC
public class MAC implements PC {
    public MAC() {
        this.make();
    }
    @Override
    public void make() {
        System.out.println("make MAC!");
    }
}
```

```java
// 手机类
public interface Phone {
    void make();
}

// 小米手机
public class MiPhone implements Phone {
    public MiPhone() {
        this.make();
    }
    @Override
    public void make() {
        System.out.println("make xiaomi phone!");
    }
}

// 苹果手机
public class IPhone implements Phone {
    public IPhone() {
        this.make();
    }
    @Override
    public void make() {
        System.out.println("make iphone!");
    }
}
```

```java
public interface AbstractFactory {
    Phone makePhone();
    PC makePC();
}

// 小米工厂
public class XiaoMiFactory implements AbstractFactory{
    @Override
    public Phone makePhone() {
        return new MiPhone();
    }
    @Override
    public PC makePC() {
        return new MiPC();
    }
}

// MAC工厂
public class AppleFactory implements AbstractFactory {
    @Override
    public Phone makePhone() {
        return new IPhone();
    }
    @Override
    public PC makePC() {
        return new MAC();
    }
}
```

此处对产品分组，可分为小米组和苹果组，各自包含手机和电脑两类产品。此时只需一个小米工厂生产小米组的产品，苹果工厂生产苹果组的产品。**即将产品进行分组，每组中的不同产品由同一个工厂类的不同方法来创建。**

-----

### 单例模式

好处：

- 有些对象只需要一个，比如线程池，日志等，如果有多个实例可能会出现问题。
- 节省内存空间，避免重复创建和销毁对象。

实现方式：

- 饿汉式

  ```java
  class Single {
      // 1.私有化构造器
      private Single() {}
      
      // 2.内部创建对象，并声明为静态
      private static Single instance = new Single();
      
      // 3.提供公共静态方法，返回类对象
      public static Single getInstance() {return instance;}
  }
  ```

- 懒汉式

  ```java
  class Single {
      private Single() {}
      
      private static volatile Single instance;
      
      public static Single getInstance() {
          if(instance == null) {
              synchronized(Single.class) {
                  if(instance == null) {instance = new Single();}
              }
          }
          return instance;
      }
  }
  ```

  > 为什么用了synchronized还要用volatile？
  >
  > 
  >
  > - 既然synchronized块已经起到及时将instance更新到主内存中，并保证了线程安全，那么为什么还要加volatile关键字呢，其实也是起到一个优化作用，synchronized块只有执行完才会同步到主内存，那么比如说instance刚创建完成，不为空，但还没有跳出synchronized块，此时又有10000个线程调用方法，那么如果没有volatile,此使instance在主内存中仍然为空，这一万个线程仍然要通过第一次判断，进入代码块前进行等待，正是有了volatile，一旦instance改变，那么便会同步到主内存，即使没有出synchronized块，instance仍然同步到了主内存，通过不了第一个判断也就避免了新加的10000个线程进入去争取锁。
  >
  >   
  >
  > - 另一方面也是为了防止指令重排序，从new对象在底层分分配内存，初始化对象，引用指向地址三步的话，如果第二步和第三步颠倒了，那么我们拿到引用地址去用的话可能对象还没初始化完全，就会出现问题

  


-----

### 代理模式

#### 1.静态代理

代理对象和目标对象实现同一个接口

```java
// 接口
public interface IUserDao {
    public void save();
}

// 目标对象
public class UserDao implements IUserDao{
    @Override
    public void save() {
        System.out.println("保存数据");
    }
}

// 代理对象
public class UserDaoProxy implements IUserDao{
    private IUserDao target;
    
    public UserDaoProxy(IUserDao target) {
        this.target = target;
    }
    
    @Override
    public void save() {
        System.out.println("开启事务");//扩展了额外功能
        target.save();
        System.out.println("提交事务");
    }
}
```

#### 2.JDK动态代理



#### 3.cglib动态代理



## Chapter 8 消息队列

----

### 1.消息队列的作用

- 异步：提升系统吞吐量（单位时间内处理请求的数量）。
- 解耦：下游模块出问题不影响上游模块使用。
- 削峰：将系统的超量请求暂存其中，以便系统后期可以慢慢进行处理，避免了请求的丢失或系统被压垮。



### 2.RocketMQ组成部分

- NameServer：
- Producer：
- Consumer：
- Broker：
- Message Queue：消息存储的物理单位



### 3.RocketMQ消费模式

- 集群消费：1.对于单个consumer group来说，消息只需要被处理一次。

  ​				  2.多个group同时消费一个topic，每个group都会有一个consumer消费到数据。

- 广播消费：一条消息会被所有订阅的consumer消费，而不用管consumer group。



### 4.如何解决消息重复消费？

原因：正常情况下consumer消费完消息后会发送ack通知broker消息已正常消费，并从queue中移除。但当ack因为网络问题没有到达broker，broker就会认为该消息没有被消费，会开启重投机制把消息再次投到consumer。

解决方案：

- 数据库表：每条消息都有自己的id，处理完消息后将其插入数据库，每次消费前去查表是否已经消费过该条数据。
- map：单机可以使用map记录已经消费的消息。
- redis：redis分布式锁。



### 5.RocketMQ如何保证顺序消费？（待完善）

RocketMQ采用了局部顺序一致性的机制，实现了单个队列中的消息严格有序。也就是说，如果想要保证顺序消费，必须将一组消息发送到同一个队列中，然后再由消费者进行注意消费。

9

### 6. 为什么RocketMQ采用pull消息的方式而不是push

如果broker主动推送消息的话有可能push速度快，消费速度慢的情况，那么就会造成消息在 Consumer 端堆积过多，同时又不能被其他 Consumer 消费的情况。而 pull 的方式可以根据当前自身情况来 pull，不会造成过多的压力而造成瓶颈。所以采取了 pull 的方式。



### 7. RocketMQ如何保证消息不丢失

1. producer：默认同步发送，发送完后等待broker响应，若超时未收到响应会重发。
2. broker：开启同步刷盘模式会等消息刷盘后再返回producer成功，同时开启主从复制，等slave也将消息刷盘后broker才会返回producer成功。
3. consumer：消费成功后才返回给broker ack，若消息消费失败会重试。

----

## Chapter 9 算法



### 时间复杂度

https://blog.csdn.net/l975764577/article/details/39399077

#### 1.树

- 二叉树查找复杂度

  最好：相当于二分查找，$O(\log n)$；
  
  最坏：$O(n)$

#### 2.二分法

$O(\log n)$

-----

### 排序算法



#### 1.冒泡排序

- 对0~n-1的数组从左到右比较相邻元素，若left > right交换二者。
- 对每一对相邻元素作此操作，最后最右边的元素会是最大元素。
- 对0~n-2的元素重复上述操作，直到数组完全顺序为止。

数组无序时时间复杂度为$O(n^2)$，有序时为$O(n)$；

```java
public void sort(int[] arr) {
    for(int i = 0; i < arr.length - 1; i++) {
        boolean flag = false;
        for(int j = 0; j < arr.length - i - 1; j++) {
            if(arr[j] > arr[j + 1]) {
                int tmp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = tmp;
                flag = true;
            }
        }
        if(!flag) {break;}
    }
}
```



#### 2.快速排序

最好情况下时间复杂度为$o(n\log n)$，最坏情况下时间复杂度为$O(n^2)$；

```java
public void quickSort(int[] arr, int l, int r) {
    if(l >= r) {return;}
    
    int i = l, j = r;
    int tmp;
    int base = arr[l];
    
    while(i < j) {
        while(arr[j] >= base && i < j) {j--;}
        while(arr[i] <= base && i < j) {i++;}
        
        tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
    arr[l] = arr[i];
    arr[i] = base;
    
    quickSort(arr, l, i - 1);
    quickSort(arr, i + 1, r);
}
```



#### 3.归并排序

时间复杂度：$O(n \log n)$



#### 4.插入排序

时间复杂度：$O(n^2)$



## Chapter 10 Zookeeper



