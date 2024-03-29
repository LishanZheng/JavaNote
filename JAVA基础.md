## 乐观锁、CAS机制（Compare And Swap）

属于乐观锁

Compare：先比较预期值和实际值，若相同才会进行下一步

Swap：替换，把实际值替换为新的值

缺点：

1. CPU压力较大，在高并发的情况下，多个线程尝试更新一个变量，又一直失败，会导致CPU带来很大压力。
2. 只能对一个变量进行原子性更新，对一个代码块只能用synchronize了



## IO模型

### 同步IO模型-IO多路复用

1. #### select

   1. 把已经连接的socket放到一个《文件描述符的集合》中。

   2. 调用select函数把《文件描述符的集合》，从用户态拷贝到内核态。

   3. 内核遍历《文件描述符的集合》检查是否有事件产生，并修改标记为可读/写
   4. 内核把《文件描述符的集合》再次拷贝回用户态。
   5. 用户态遍历《文件描述符的集合》，找到需要读写的socket进行操作。

   总结：2次拷贝、2次遍历、效率比较低、文件描述符集合大小限制为1024

2. #### poll

   1. 把已经连接的socket放到一个《文件描述符的集合》
   2. 把《文件描述符的集合》从用户态拷贝到内核态
   3. 内核态遍历《文件描述符的集合》，查询是否有可读、可写操作并修改标记
   4. 内核态把修改后的《文件描述符的集合》再拷贝回用户态。
   5. 用户态遍历《文件描述符的集合》，对于需要读写的socket进行对应操作。

   总结：总体和select一致，不同的是poll使用双链表代替数组去掉了1024的文件描述符集合数限制。

3. #### epoll

   ##### 主要包括3个API函数，

   1. epoll-create

      创建epoll实例，关联当前进程已经打开的文件。

   2. epoll-ctl

      主要用来管理红黑树上的epitem节点，增删改操作为主

      通过epoll-ctl注册一个socket的流程如下：

       1. 分配一个红黑树节点对象epitem

       2. 添加等待事件到socket的等待队列中，并且注册回调函数ep-poll-callback

          ​	当socket收到数据时，会调用socket等待队列中注册的回调函数。

       3. 将epitem添加到红黑树中。

   3. epoll-wait

      检查创建的epoll实例中是否有就绪的文件描述符。

      本质就是检查rdllist上是否有节点，有就返回，没有就创建等待队列项放入等epoll的等待队列中。

   ##### epoll结构包括

   	1. wq等待队列链表，通过wq找到阻塞的用户进程。
   	2. rbr红黑树，管理所有的socket连接。
   	3. rdllist准备就绪的描述符链表，当连接准备就绪时，放入该链表。减少遍历

   ##### 数据来时的流程

   1. 数据到来时候，先把数据放到socket的接收队列上
   2. socket上的数据准备就绪后，唤醒socket上等待的用户进程，内核会找到epoll-ctl时放入socket等待队列的回调函数ep-poll-callback。
   3. 调用回调函数后，可以找到epoll和epitem对象。
   4. 把epitem对象放入epoll的rdllist就绪队列中。
   5. 查看epoll的等待队列中是否有等待队列项（epoll-wait创建的），没有就结束
   6. 如果有的话就找到等待队列项中的回调函数default-wake-function
   7. 找到回调函数default-wake-function中的进程描述符，唤醒进程。



## 基本数据类型

int ：4个字节  byte 1个字节 short 2个字节

long 8个字节 float 4个字节 double 8个字节

char 2个字节 



## 重写和重载

#### 重写

是子类继承父类后，重写父类的方法，参数列表一致、返回值一致、访问修饰符不能高于父类。

#### 重载

是同一个类中，同名方法的重载，方法名字一样，参数列表不一样、返回值可以不一样、



## HashCode 和 equals

Hashcode() 是散列码，确定对象在哈希表中的位置。

​	当需要把对象加入哈希表中时，会先判断当前对象的哈希码是否存在

​		存在：进行equals来进一步确认，相同就不插入，不相同就重新散列到其他位置

​		不存在：直接放入即可。

​	前提是：当equals相同的时候，hashcode一定要相同，不然就会矛盾。

- hashcode和equals的作用

​	     当hashcode相同时，对象可能不一样需要进一步equals。

​	     当hashcode不同时，对象一定不同。



##  String、StringBuffer、Stringbuilder的区别

- 可变性

​	String的字符数组是用private和final修饰，且没有提供修改字符数组的方法

​	Stringbuilder和Stringbuffer字符数组有提供方法进行修改。都继承于AbstractStringBuilder

- 线程安全性

String和StringBuffer都是线程安全

	1. String是不可修改的，可以理解为常量，线程安全
	1. StringBuffer的修改方法和调用方法都加了同步锁，也是线程安全

StringBuilder不是线程安全的

- 使用总结

少量用String，大量单线程用Stringbuilder，大量多线程用StringBuffer	

- String.intern方法

intern是一个本地方法，将对象的字符串对象的引用保存到字符串的常量池中。若字符串中有该对象的引用，则返回，没有就先创建后返回。

- String不可变的原因？

1. String的字符数组由private和final修饰，且没有提供修改数组的方法。
2. String类本身也被final修饰，不能被继承后破坏不可变。

- 为什么改用Byte数组？

1. JAVA9之后，String、StringBuilder、StringBuffer都是用 byte数组来存储字符，本质上因为byte数组一个字符只需要8个bit，而char需要16个bit，若字符没有超出Latin-1的范围用byte就可以节省一半的空间，若有超出的字符，那么就和原来一样。

- 字符串常量池，JDK6 -JDK7

JDK6之前String的字符串常量池位于perm，是类静态区域，多次调用intern会导致这个区域内存溢出。JDK7把这个区域移动到 JAVA Heap中了



## 异常

Exception和Error：Exception是程序可以捕获和处理的异常，Error是无法处理的错误，如虚拟机内存不足。



## 反射

- 为什么反射比较慢？

1. 编译器无法优化代码，它无法了解你代码在做什么
2. 反射需要进行安全检查、如检查是否有无参构造、是否有权限访问无参构造、调用者是否有权限使用反射等。
3. 反射在发现调用的对象上比较慢，如通过名称匹配类，匹配方法。

- 反射的优点和缺点

优点：使用起来比较方便，各种框架都可以开箱即用。

缺点：反射比较慢，但对于很多框架来说影响不大。 反射会带来一些安全问题，比如它会无视范型的类型检查。



## 注解

是一种特殊的注释，本质上是继承了特殊的annotation的特殊接口。

解析方式：

	1. 编译期解析，如@Override会在编译阶段检查是否重写父类方法。
	1. 运行时解析，比如Spring框架中的注解，在运行时通过反射进行处理



## 代理

### 静态代理

实现步骤：

1. 定义接口和实现类
2. 定义代理类同样实现接口
3. 目标类注入到代理类，在代理类的实现中进行额外操作并调用原来的方法。



### 动态代理

#### JDK动态代理

实现步骤

1. 定义接口类和实现
2. 定义代理类实现InvocationHandler，实现invoke方法。在invoke方法中通过反射调用原生方法并进行额外操作。
3. 通过Proxy.newProxyInstance(classLoader, class<>Interfaces, InvocationHandler) 方法创建代理对象，返回的是Object，需要转化为接口类。

JDK的致命缺点是只能对实现接口的类进行代理，这个缺点可以通过CGLIB代理消除

#### CGLIB动态代理

实现步骤

1. 实现一个类
2. 实现一个拦截类实现MethodInterceptor。实现intercept方法，方法里通过调用method.invokeSuper来调用原生方法，并加入额外操作
3. 调用时，新建enhancer类，注入（setClassLoader)类加载器、(setSuperClass)被代理类、(setCallback)拦截方法。enhancer.create()创建对象。对象调用方法。

#### 比较两种动态代理

1. JDK动态代理只能代理实现了接口的类或直接代理接口，而CGLIB可以直接代理未实现任何接口的类。
2. CGLIB是生成一个被代理类的子类拦截代理类的方法调用，所以对于final对象没有办法代理。
3. 效率相比，JDK动态代理高于CGLIB动态代理

## JAVA的引用

强引用

在项目中用new对象生成的，都是强引用，除非是被显示的置null。否则不会被垃圾回收自动回收



## 并发编程

### 线程与进程

进程：进程是一个程序的运行过程，系统运行程序的基本单位。

线程 ：线程是比进程更小的执行单位，一个进程可以有多个线程。

不同点：

1. 同一个进程的线程共享进程的方法区和堆。
2. 每个线程有自己的虚拟机栈、本地方法栈、程序计数器。
3. 线程之间切换的代价比进程小。

- 为什么线程的虚拟机栈、程序计数器、本地方法栈是私有？

​		因为要避免线程执行时的局部变量不被其他线程访问。

- 线程、进程之间的同步机制
  1. 临界区
  1. 互斥区
  1. 信号量
  1. 事件


- 进程间的通信方式IPC

  1. 管道

     1. 有名管道，用于非亲属之间的通信
     2. 无名管道，亲属之间的通信

     特点：速度慢、容量有限、只能父子进程之间通信


  2. 消息队列，同一个机器上的通信，与管道类似

  ​	需要考虑上一次没有读取完的数据问题

  3. 信号量，是一个计数器，记录资源的存取状态

  ​	不能传递复杂消息，只能用来同步

  4. 共享存储，共享内存通常由一个进程创建，其他进程对这块内存进行读写。

  ​	速度快、容量可控、但要考虑读写同步问题

  5. Socket

#### 线程的生命周期和状态

线程有6个状态：

1. New 初始状态，线程被创建，但是还没调用start方法。
2. Runnable 运行状态，线程正在被执行，（包括线程的运行状态和就绪状态）
3. Blocked 阻塞状态，线程被锁给阻塞。
4. Waiting 等待状态，线程等待其他线程的操作比如通知或者中断。
5. Time_Waiting 超时等待状态，线程过一段时间后会自动恢复。
6. Terminated 中止状态，线程已经执行完成。

#### 线程死锁条件

* 死锁产生条件
  * 互斥条件：资源任一时刻只能有一个线程占用。
  * 请求与保持条件：占有部分资源的线程请求新资源被阻塞后，之前占有的资源仍然占有。
  * 不剥夺条件：还没运行完毕的线程，已经持有的资源在没有运行完毕之前不会被剥夺。
  * 循环等待条件：若干个线程之间形成一种头尾相连的循环等待资源情况。

* 死锁的预防，破坏死锁产生条件
  * 破坏请求与保持条件：一次性请求所有资源。
  * 破坏不剥夺条件：当占有其他资源的线程请求新的资源没有被满足时，就释放之前占有的资源。
  * 破坏循环等待条件：按照某种顺序请求资源，释放资源的时候按照反序。

#### 线程sleep和wait的异同点

不同点：

1. sleep不会释放锁，wait会释放锁
2. sleep会在一定时间后自动苏醒，wait需要等待其他线程的notify才会苏醒（也可以通过wait（时间）来苏醒）
3. sleep用于暂停执行，wait用来线程之间的通信/交互。

相同点：

1. 都可以用来暂停线程。

#### 执行start() 会调用run方法，和直接调用run方法有什么不同？

start执行后会进行线程的初始化，完成后在调用run方法执行。

直接调用run方法会直接把run方法作为普通方法执行，不是多线程。

#### 线程提交任务的不同方式

Runnable和Callable的区别

1. Runnable不会返回结果和异常，Callable会返回结果和异常。

execute和submit的区别

1. Execute方法用于提交不需要返回值的方法，所以无法判断线程是否执行成功。
2. submit方法可以提交需要有返回值的方法，会返回一个Future对象，通过这个对象可以获取线程的执行信息。



### 线程池

#### 基本参数

1. corePoolSize，核心线程数，最小的可以运行的线程数
2. maximumPoolSize，可以运行的最大线程数量。
3. workQueue，线程队列，当核心线程运行满了，新来的任务就会被放到线程队列。
4. keepAliveTime，当线程池中的线程数量大于核心线程数量时，如果线程空闲时间超过来keepAliveTime则会被回收销毁。
5. timeUnit，KeepAliveTime的时间单位。
6. threadFactory，executor创建新线程时用到的工厂方法。
7. Handler，拒绝策略，当线程池的线程数量达到最大运行线程数，且等待队列也满了，执行的拒绝策略。
   1. AbortPolicy，抛出RejectedExecutionException拒绝执行异常（默认策略）

#### 线程池的创建流程

新任务提交

1. 首先判断当前线程数是否达到corePoolSize核心线程数，没有就创建线程执行任务。
2. 若达到corePoolSize核心线程数，则判断是否等待队列已满，没有满就放入workQueue等待队列。
3. 若workQueue等待队列已满，则判断是否当前线程数到达maximumPoolSize最大线程数，没有到达则创建线程执行任务。
4. 若到达maximumPoolSize最大线程数。则执行拒绝Handler策略。

#### 好处

1. 降低资源消耗：重复利用已经创建的线程，可以节省线程创建和销毁的开销。
2. 提高响应速度：任务来的时候，不需要创建新线程，可以节约创建线程的时间。
3. 提高线程的可管理性：由统一的线程池来管理线程，可以方便统一分配、调优和监控。

### JAVA并发指令

#### Synchronize

修饰对象

1. 修饰实例方法，*需要获得当前实例对象* 的锁
2. 修饰静态方法，需要获得当前class对象的锁，与实例方法不冲突。实例方法是获取实例对象的锁，静态方法是获取当前类的锁，不是同一个锁。
3. 修饰代码块，获取指定对象的锁，也可以是对象.class 获取指定对象类的锁。

底层原理

1. 信息存储位置

   Java对象的对象头中的Mark Word（对象头包括1.自身运行的数据 mark word、2.类型指针，指向类）

   1. 无锁状态：hashcode、分代年龄、是否偏向锁
   2. 偏向锁：偏向线程id、偏向时间戳、分代年龄、是否偏向锁
   3. 轻量锁：指向栈中锁记录的指针
   4. 重量级锁：指向互斥量的指针

2. 不同对象的原理

   1. 方法：通过方法常量池的方法表结构中的ACC_SYNCHRONIZED标志，来告知这是一个同步方法。
   2. 代码块：通过编译时插入monitorenter 和 monitorexit两条指令

3. Minotor对象

   1. 数据结构

      count ：计数，当前对象没有被获取或者当前线程已经获取到锁了，就加一，释放就减一。

      recursions：重入次数。

      owner：持有锁的线程。

      waitSet：线程调用wait方法后进入waitset，等待其他线程唤醒。

      entryList：队列用来获取锁的缓冲区。把cxq和waitset中的数据移动到entrylist进行排队

      cxq: 多个线程争抢锁，会先存入这个单向链表

   2. enter方法竞争锁的流程

      1. 首先通过CAS尝试把monitor中的owner改为当前线程。修改成功表示获取到锁了，并且可以recursions来记录重入次数
      2. 如果CAS失败，则进行自旋多次尝试。
      3. 如果还失败则让自己加入cxq等待链表。还会再次尝试CAS拿到锁，失败就阻塞。

   3. exit方法释放锁的流程

      1. 先判断recursions是否大于0，如果大于0 表示重入，recursion--即可
      2. 根据QMode的模式执行不同策略
         1. 默认模式，从entryList中唤醒，若为空则把cxq反转，然后从cxq中获取。
         2. 优先从cxq唤醒
         3. 把cxq加入entryList尾部然后从entryList中唤醒 
         4. 把cxq放到entryList头部，然后从entryList唤醒

4. 锁升级过程

   1. 偏向锁：当第一次执行到synchronized代码块时，把锁对象变为偏向锁（通过CAS修改对象头的锁标志位）。后面第二次到达synchronized代码块，检查锁的偏向线程是否是当前线程，如果是就不需要操作。如果不是就说明在竞争锁，进行锁升级
   2. 轻量级锁：没有抢到锁的线程，不断进行自旋获取锁，当自旋次数超过某个值时（默认10）就会进行锁升级（CAS修改锁标志位）。
   3. 重量级锁：当一个线程获取到锁，其他线程尝试获取锁会进入阻塞状态。

#### Volatile

1. 防止指令重排
2. 变量可见性

#### Syn和Vol的区别

1. Syn可以用于代码块、方法。Vol只能用于变量。Vol可以用于轻量级的线程同步
2. Vol可以保证数据的可见性，但不能保证数据的原子性。Syn两者都可以保证。
3. Vol用于多个线程之间变量的共享可见性，Syn用于多个线程之间资源的同步性



### AQS 

AQS是用来构建锁和同步器的框架。（AbstractQueuedSynchronizer）

#### AQS的原理

当被请求的资源是空闲，就将当前的线程变为有效工作线程，并将资源的状态改为锁定状态。

若被请求的资源是被锁定的，就需要一套线程阻塞唤醒的锁分配机制。（AQS使用的是CLH队列锁实现的，将请求不到资源的线程放入队列中）

#### AQS的资源共享方式

1. 独占式：资源只能由一个线程获取，锁包括两种
   1. 公平锁，按照队列顺序，先到的先获取
   2. 非公平锁，不按照队列顺序，谁先抢到锁资源归谁。
2. 共享式：多个线程可以公用这个资源。如CountDownLatch





### ThreadLocal

- ThreadLocal.set详解

通过hash计算出要存的下标。

1. 数据为空，直接放在这个位置。

2. 数据不为空，但是key值一样。直接更新。

3. 数据不为空，且key值不一样，向后遍历

   1. 遇到key值过期的entry（调replaceStaleEntry方法)

      staleSlot=当前index 

      1. 从过期位置向前寻找，直到entry为空的位置停止。（更新slotToExpunge，表示需要清理的过期数据的起始下标）
      2. 寻找完毕后从staleSlot位置开始向后找key相同的元素，
         1. 找到key值一样的就更新，并且交换staleSlot和它的位置。
         2. 没找到值一样的，找到entry为null就停止。替换staleSlot位置上的entry为当前新的。
      3. 从staleToExpunge开始向后清理。
      4. 返回

   2. 没有遇到key值过期的entry

      1. key值一样就更新
      
      2. 遇到了空值，表示是替换操作。
         1. 先进行一次启发式清理工作。
         
         2. 若没有清理掉数据，且size大于threshold阈值，就会进行rehash操作
         
            

- ThreadLocal.replaceStaleEntry详解

当前位置为staleSlot；清理起始位置slotToExpunge；

要插入的值为<key，value>

1. 先从staleSlot位置往前寻找key == null的位置，直到entry == null 停止。把位置存到slotToExpunge中。
2. 向前遍历结束
   1. slotToExpunge==staleSlot 说明前面没有需要清理的。
   2. slotToExpunge！=staleSlot 说明前面有需要清理的。
3. 从staleSlot位置开始向后遍历，
   1. 遇到key相同,则把staleSlot位置的过期数据存入当前位置，然后把staleSlot位置插入<key，value>，
      1. 如果前面没有要清理的，就从当前位置开始清理（即slotToExpunge = i ) 
      2. 如果前面有需要清理的，
      3. 调用cleanSomeSlots（slotToExpunge），然后返回
   2. 遇到key == null 且 前面没有要清理的，则把slotToExpunge改为当前位置。
4. 若遍历结束（遍历到enrty == null），则说明中间没有过期数据，要新增了。
   1. 在staleSlot位置插入<key，value>
   2. 若前面有要清理的调用（cleanSomeSlots（slotToExpunge）

- ThreadLocal的过期key清理流程

1. 探测式清理

   expungeStaleEntry方法，主要是从当前entry往后清理，遇到null就停止

   1. 按顺序遍历散列数组
      1. 遇到过期的key把entry设为null
      2. 遇到没有过期的key，重新hash，尝试更新位置。

2. 启发式清理

   cleanSomeSlots() 方法，数组长度为n

   1. 从传入的位置开始向后遍历 log2（n)次
      1. 若遍历过程中没有过期key，次数--，直到次数为0
      2. 若遇到过期key则进行探测式清理，且次数重置为log2（n）次

- ThreadLocal的扩容机制

1. 先进行探测式清理
2. 判断大小是否达到3/4的阈值
   1. size >= 3/4 * threshold 到达就resize扩容
3. resize扩容
   1. 大小更新为原来的两倍
   2. 遍历原来的散列表，重新计算元素的hash位置
   3. 更新阈值为原来的两倍

- ThreadLocal.get详解

1. 通过key值找到散列表中的位置slot
2. 是否key一样
   1. key一致，直接返回
   2. key不一致，调用expungeStaleEntry方法，进行探测式清理，再往后查找，直到key一致。

### HashTable

- 线程安全，每个方法都是用synchronize修饰
- null值：无法放入null值，不管key或者value
- 容量：默认大小11、每次扩容为2n+1
- 底层数据结构：数组+链表

### HashMap

- 线程不安全：如果要线程安全使用concurrentHashmap

- null值：map可以存null的key但只能一个。value可以多个为null。

- 容量：默认为16，如果手动输入大小，会变为最近的2的幂次方大小

- 底层数据结构，当链表长度大于阈值（默认8）会变为红黑树。在扩容前会判断如果当前数组的大小小于64会优先进行数组扩容。

  

源码分析 

1. put方法

   - Jdk1.7
     1. 获取key，计算hash下标 
     2. 如果下标的数组没有元素就直接放入、冲突就遍历链表，找到key一样的就覆盖｜不一样的就使用头插法放入元素

   - jdk1.8
     1. 获取key，计算hash下标
     2. 如果下标的数组没有元素就直接放入、冲突就遍历链表或者查询红黑树找key一样的覆盖｜如果没有一样的链表使用尾插法放入元素。红黑树调用红黑树的方法放入元素。
     3. 如果链表长度大于阈值且数组长度大于64才会转化为红黑树。

2. get方法

   - 获取key，计算下标去查找元素。

     

### concurrentHashMap

- jdk1.7版本前

  - 底层使用分段数组+链表，每一段有一个锁来控制并发访问。（segment长度默认16）
  - 分段数组，不可扩容，内部是HashEntry数组可以扩容

- jdk1.8之后

  - 底层使用Node数组+链表/红黑树，并发通过synchronize和CAS来操作。

    - synchronize只用来锁定当前链表或红黑树的首节点，这样只要hash不冲突就不会有并发。

  - Node数组

    - 链表时使用Node、红黑树使用TreeNode（TreeNode背TreeBin包装）

      - TreeBin包括
        1. root（维护红黑树的根节点）
        2. waiter，用来维护当前使用这颗红黑树的线程

    - 因为红黑树在旋转时，根结点会被替换，所以在这个时间点内如果其他线程要写会发生线程不安全的情况，所以使用waiter来记录当前使用红黑树的线程。

      

源码分析

##### JDK1.7版本前 

底层使用segment + HashEntry + 链表

- 初始化方法（若无参则默认容量16、默认负载因子0.75、默认并发等级16）

  1. 检查并发等级是否超过最大值、超过就变为最大值
  2. 初始化segment[0]，默认大小为2、负载因子0.75，所以扩容阈值为1.5.

- PUT 方法（segment默认容量为16（2的N次）、HashEntry默认容量为2、先计算hash值）

  1. 计算segment的位置、通过获取hash值的最高N位作为index（若容量为16则为最高4位）

  2. 检查index位置的segment是否被初始化过。没有就初始化

     Segment的初始化：

     1. 检查该位置的segment的是否为null、为null就用segment[0]的容量、负载因子参数来创建一个HashEntry。
     2. 再次检查segment位置是否为null、为null就用刚刚创建的HashEntry初始化这个Segment
     3. 自旋判断segment位置是否为null、为null 使用CAS来把这个位置赋为刚刚初始化的Segment

  3. 使用tryLock获取segment锁、没有获取到就使用scanAndLockForPut方法继续获取

     scanAndLockForPut方法就是不断自旋获取锁、然后次数达到一定就变为阻塞获取锁。

  4. 计算要put在segment中HashEntry的位置，然后获取这个HashEntry。（计算方法为hash&HashEntry数组的大小-1）

  5. 遍历HashEntry尝试放入元素

     1. HashEntry不存在、先计算容量是否达到阈值、到达就扩容。然后使用头插法放入元素
     2. HashEntry存在、遍历链表，如果找到key一样的就更新。如果都没有一样的，先计算容量是否达到阈值、到达就扩容。没有就使用头插法放入元素。

  6. 如果之前有了就返回旧值、不然就返回null。

- Resize 扩容方法（某个Segment内的HashEntry个数扩容后会变为原来的两倍）原来容量为N

   1. 计算新的容量（2*N）、计算新的阈值、创建新的HashEntry数组。

   2. 遍历旧的数组把元素放入新的HashEntry数组中。

      因为都是2的倍数、扩容到新容量时，原来HashEntry数组的元素位置只可能不变或者增加N

   3. 使用头插法放入新的节点。

- GET 方法

  1. 计算key的存放位置
  2. 遍历查找key的value。

- 计算大小size

  1. 先采取不加锁，计算两次大小，若一样则返回
  2. 若不一样，则把所有的segment锁住，计算大小。

##### JDK1.8版本后

底层使用Node+链表/红黑树，当链表容量大于8且数组大小大于64时进行链表到红黑树的转化。

- 初始化方法

  1. 通过对 sizeCtl 变量的自旋和CAS来控制初始化

     -1 表示正在初始化

     -N 表示N-1个线程正在扩容

     如果没有被初始化过，表示table的初始化大小

     被初始化过，表示table的容量大小

- PUT方法

  1. 根据key计算hashcode值
  2. 检查table是否被初始化过
  3. 通过hashcode值找到需要放入的Node，
     1. 若Node为空，直接写入，通过CAS尝试写入，使用自旋保证成功。
     2. 若hashcode值等于MOVED == -1，则需要进行扩容。
     3. 使用synchronize锁写入数据。
  4. 判断链表数量是否达到树化要求，（链表长度大于8且Node数组 >= 64），没达到优先扩容Node数组。

- GET方法

  1. 根据key计算出hashcode值
  2. 找到对应的Node节点
     1. 若头节点就是，直接返回。
     2. 头节点hash值 < 0表示为树或者正在扩容。
     3. 链表遍历查找

- Resize扩容方法

  - 先创建一个大小为原来两倍的table数组，这个过程一般由一个线程完成，不允许并发。
  - 单线程或多线程并发扩容 helpTransfer
    - 多线程会先计算一个步长，表示一个线程一次可以处理的桶个数。
    - 当扩容时没有传入newTable参数表示单线程扩容，
  - 数据进行迁移 transfer
    - 链表迁移
      1. 首先遍历一次找到链表中最后一个相邻不同的节点p，记录它第n位的hash值。
      2. 根据hash值把p节点后面的整个链表都放入对应链表中。（hash =1表示高位、0表示低位）
      3. 再次遍历直到p节点，计算n位hash值，等于1使用尾插法放入高位链表、等于0放入低位链表
    - 红黑树迁移
      1. 先以链表的方式遍历红黑树
      2. 计算每个节点的值后放到对应的高位或者低位链表上。
      3. 如果高位或低位满足红黑树条件，进行树化后插入对应位置。

  扩容前把头节点的hash设为-1、并且对于链表使用尾插法

  1. 线程尝试扩容，如当前Node正在扩容，线程会帮助一起扩容

     帮助扩容：

     	- 会设置一个步长，表示一个线程需要处理的桶个数。
     	- 每个线程处理完会在桶的头部放一个fwd节点，表示处理完毕，其他线程看见会跳过。

- 计算大小size

  通过一个baseCount变量记录当前节点数

  1. 先尝试通过CAS修改baseCount
  2. 如果过多线程竞争修改baseCount，则将cellsbusy设为1，然后暂存baseCount的修改到counterCells数组中。
  3. 若置为1都失败，就会反复进行CAS的baseCount和CAS的counterCells数组



### HashSet





# JVM虚拟机

### 对象

#### 对象的创建

1. 类加载检查

   当遇到new指令时，先检查指令的参数是否能在常量池中找到它的符号引用。若有引用，则去查找是否这个类被加载、解析和初始化过，如果没有就进行类的加载过程。

2. 分配内存

   在类加载完后，对象所需的内存大小已经确定。虚拟机为新生对象分配内存，即把一块确定大小的内存从Java堆中分配出来给对象。

   1. 指针碰撞方式，把用过的内存放到一边，另一边都是空闲内存，中间有一个分界指针。每次移动分界指针来获取指定大小内存。
   2. 空闲列表，虚拟机维护一个列表，记录那些内存块是空闲，在分配时，寻找一块可以放下对象的空闲区域划分给对象，记录在列表中。

   **内存分配的并发问题**（同时有多个对象进行分配内存）

   1. CAS+失败重试，CAS是一种乐观锁，假设没有冲突去执行操作，每次操作都不加锁，如果冲突就重试，直到成功。
   2. TLAB，为每个线程在Eden区中分配一些内存，先使用。如果不够用，就使用1的CAS+重试进行内存分配

3. 初始化零值

   内存分配完成后，需要把内存分配到的区域数据全部重新初始化零值，不包括对象头。

4. 设置对象头

   初始化零值后，需要对对象进行必要的信息设置，如是那个类的实例、元数据信息的地址，对象哈希码等信息。把这些信息存放到对象头中。

5. 执行init方法

   从虚拟机角度来看，对象已经生成完毕，但Java程序来看，对象还没执行init方法。执行完init方法才是按程序员的想法进行对象的初始化。得到真正可用的对象。

#### 对象的内存布局

1. 对象头

   1. Mark word 存储对象自身的运行数据（哈希码、GC分代年龄、锁标志）
   2. 类型指针，对象指向的类元数据指针。

2. 实例数据

   ​	程序中定义的字段

3. 填充数据

   ​	没有意义，只是因为虚拟机中是8字节的整数倍。

#### 对象的访问定位

1. 句柄

   Java堆中会划分出一块内存作为句柄池，reference中存放的是对象的句柄地址。句柄包括了对象实例数据的地址和类型数据的地址

   优点：reference中存放的是稳定的句柄地址，修改时只需要需要改句柄中的实例数据指针

2. 直接指针

   reference中存放的是对象的地址。

   优点：访问块，减少了一次指针定位的时间开销。

### 垃圾回收

#### 分区

1. 新生代 Young Generation

   1. Eden

      * 对象会先在Eden区分配，若没有足够内存会进行一次Minor GC

      * Eden里得对象经过一次Minor GC新生代垃圾回收后，会进入s0 或 s1，初始年龄为1

      * 经过一次Minor GC后，Eden/From 会被清空, From和To 会交换角色，来保证To是空的。

      * *年龄阈值*，阈值也会变化，规则是，虚拟机会按年龄从小到大进行判断，若某年龄的内存总大小大于Survivor大小的一半，就取这个值和Max阈值的小的那个作为新阈值。

   2. From Survivor0

      放入From后，若年龄达到阈值则晋升至老年代

      若From放不下，也会直接晋升到老年代

   3. To Sruvivor1

      每次Minor GC 都会，先尝试放进来，处理完From和Eden后，再把To的内容迁到From。保证To是空的。

2. 老年代 Old Generation

   对象比较大会直接放入老年代，减少分配担保的开销

   1. Old Memory

#### 回收

1. 新生代回收（MinorGC/Young GC)

   * 回收范围：新生代区域，即Eden、FromSurvivor、ToSruvivor
   * 触发条件：
     1. Eden空间满，对象分配内存时，无法放入Eden

2. 老年代回收（MajorGC/Old GC）

   * 回收范围：老年代区域

3. 混合回收（MixedGC）

   * 回收范围：新生代+老年代

4. 整堆回收（FullGC）

   一般是触发了MinorGC后，需要空间分配担保，担保失败就会执行FullGC

   * 回收范围：整个Java堆和方法区
   * 触发条件：
     1. 老年代空间不足
        1. 当新生代所有对象的总空间小于老年代的连续空间。
        2. 老年代的连续空间不大于历次晋升到老年代对象的平均大小。

#### 对象的状态

1. 引用计数法

   每有一个引用，计数器就增加1。

   缺点：互相引用的没法回收。

2. 可达性分析算法

   通过GC Roots对象作为起点，GCRoots 没法到达的对象说明需要被回收

   * GC Roots对象
     1. 虚拟机栈引用的对象
     2. 本地方法栈中引用的对象
     3. 方法区中的静态属性引用的对象
     4. 方法区中的常量引用的对象
     5. 所有被同步锁持有的对象

#### 对象的引用

1. 强引用

   绝不回收

2. 软引用

   内存不够就回收

3. 弱引用

   GC发现就回收，不管内存够不够

4. 虚引用

   形同虚设，主要用来追踪对象被垃圾回收的活动 

#### 类的回收判定

1. 该类的所有实例都已经被回收，即Java堆中没有该类的实例对象
2. 该类的ClassLoader已经被回收
3. 没有用到该类的Class对象，即无法通过反射来访问该类的方法。

### 垃圾回收算法

#### 标记-清除算法

先标记出不需要回收的对象，在统一回收没有被标记的对象。带来了两个问题：效率、空间问题。

#### 标记-复制算法

解决效率问题

将内存分为两半，一次使用一半，当某一个半用完后，就将还存活的对象复制到另一半，清空掉这一半。

#### 标记-整理算法

和标记清除类似

标记出不需要回收的对象，然后把这些对象往一边移动，然后直接清楚边界以外的内存。

#### 分代收集算法

对于每一代的特点，使用对应的收集算法

1. 新生代

   新生代的对象可能需要大量回收，所以要使用标记复制算法提高效率，只需要付出少量的复制成本就可以进行垃圾回收。

2. 老年代

   老年代的对象一般较少回收，所以使用标记清除或标记整理进行垃圾回收

### 垃圾回收器

Serial 收集器-单线程回收

Serial-Old收集器-老年代版本

ParNew收集器-多线程回收

Parallel Scavenge收集器-多线程回收，更关注吞吐量

Parallel Old收集器--老年代版本

CMS收集器-真正的并发收集

	1. 初始标记，暂停所有线程，标记
	2. 并发标记，启动GC和用户线程，维护一个闭包来查询可达对象，
	3. 重新标记，修正并发标记期间产生的新标记。
	4. 并发清除，开启用户线程，并且GC线程进行清理
	5. 缺点：CPU敏感、无法处理浮动垃圾、标记清楚碎片多

G1收集器。总体看是标记整理，局部看是标记复制。它维护了一个优先列表，优先选择回收价值大的区域进行回收。

### 类文件

class文件

1. 魔数（Magic Number）4Byte

   用来确定文件是否是可以被虚拟机接受的class文件

2. 版本号（Minor&Major Version）2+2Byte

   1. 次版本号
   2. 主版本号 （JAVA8，9）

3. 常量池（Constant Pool）

   1. 常量池数量
   2. 常量池具体（包括字面量和符号引用）

4. 访问标志（Access Flags）

   用来识别一些类或接口的访问信息，包括是类还是接口，

5. 当前类、父类、接口的索引

   1. 当前类
   2. 父类
   3. 接口数
   4. 接口列表

6. 字段表集合（Fields）

   描述接口和类中声明的变量，包括类级和实例变量。但是不包括局部变量

   1. 字段个数
   2. 字段列表

7. 方法表（Methods)

   与字段类似

   1. 方法个数
   2. 方法列表

8. 属性表（Attributes)

   1. 属性个数
   2. 属性列表

### 类加载器

#### JVM自带的三个ClassLoader

1. BootstrapClassLoader（启动类加载器）

   负责加载`%JAVA_HOME%/lib`目录

2. ExtensionClassLoader（扩展类加载器）

   负责加载`%JRE_HOME%/lib/ext` 目录下的 jar 包和类

3. AppClassLoader（应用程序类加载器）

   面向用户的加载器，负责加载当前应用classpath下的所有jar包

#### 双亲委派模型

核心：防止重复加载

1. 自底向上，检查是否被加载过
2. 自顶向下，尝试加载类

JVM如何区别不同类？

1. 不同类名
2. 不同的加载器

### 类加载过程

类加载过程分为：加载 - 连接（验证 - 准备 - 解析） - 初始化

- 加载

加载阶段和连接阶段并非完全隔离，加载一半可能就进行连接了。

1. 通过全类名获取定义此类的二进制字节流
2. 把字节流代表的静态数据转为运行时方法区的数据结构
3. 在内存中生成一个class对象，来给方法区数据作为访问入口

- 验证

1. 文件格式验证

   保证文件的格式正确，比如版本号是否在范围内。

2. 元数据验证

   对字节码的信息进行语义分析，验证信息的正确性，比如每个类除了Object都有父类

3. 字节码验证

   对数据流和控制流进行验证，保证运行过程中不会危害虚拟机配合工作。

4. 符号引用验证

   确保解析动作能正确执行。

- 准备

为类变量分配内存并设置初始值

- 解析

将常量池中的符号引用替换为直接引用的过程，也就是得到类、方法在内存中的指针或者偏移量

- 初始化

执行初始化方法，是类加载的最后一步

- 卸载

即卸载该类就是把该类的Class对象进行垃圾回收

需要满足条件：

1. 该类的所有实例对象都被GC
2. 该类没有被其他地方引用
3. 该类的类加载器被GC了



### OSR 栈上替换

OSR的全称是 On Stack Replacement。

- 为什么要OSR？

某些热点代码，在解释执行的比较慢，需要进行编译后减少后面执行时的时间，因为编译执行的速度明显快于解释执行。但在这个编译替换的过程中，编译需要时间，解释器还在不断的解释执行后面的代码。就会导致当编译完成后，需要等下一次调用这个方法时才会把解释执行的代码替换为编译后的代码。但可能是个循环，调用一次就没了，但是循环还在不断进行，这时编译器就做了无用功。所以就需要OSR，它把解释器栈的数据打包到OSR buffer中，然后就执行OSR版本的代码。执行完后又回到低优化等级的代码中继续执行。

