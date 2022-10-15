# JUC
## 基础概念

### 并发和并行

#### 并发

并发（Concurrent）是指在操作系统中，是指一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理器上运行。

并发不是真正意义上的“同时进行”，只是CPU把一个时间段划分成几个时间片段(时间区间)，然后在这几个时间区间之间来回切换，由于CPU处理的速度非常快，只要时间间隔处理得当，即可让用户感觉是多个应用程序同时在进行。



#### 并行

并行（Parallel）是指当系统有一个以上CPU时，当一个CPU执行一个进程时，另一个CPU可以执行另一个进程，两个进程互不抢占CPU资源。

其实决定并行的因素不是CPU的数量，而是CPU的核心数量，比如一个CPU多个核也可以并行。适合科学计算，后台处理等弱交互场景。



##### 并发和并行对比

- 并发，指的是多个事情，在同一时间段内同时发生了；并行，指的是多个事情，在同一时间点上同时发生了。
- 并发的多个任务之间是互相抢占资源的；并行的多个任务之间是不互相抢占资源的。
- 只有在多CPU或者一个CPU多核的情况中，才会发生并行；否则，看似同时发生的事情，其实都是并发执行的。



并发和并行最开始都是**操作系统**中的概念，表示的是CPU执行多个任务的方式。

- 顺序：上一个开始执行的任务完成后，当前任务才能开始执行
- 并发：无论上一个开始执行的任务是否完成，当前任务都可以开始执行

（即 A B 顺序执行的话，A 一定会比 B 先完成，而并发执行则不一定。）

- 串行：有一个任务执行单元，从物理上就只能一个任务、一个任务地执行
- 并行：有多个任务执行单元，从物理上就可以多个任务一起执行

（即在任意时间点上，串行执行时必然只有一个任务在执行，而并行则不一定。）



并发的关键是你有处理多个任务的能力，不一定要同时。并行的关键是你有同时处理多个任务的能力。所以它们最关键的点就是：是否是**同时**。



### 核心问题
#### 分工问题

分工就是将一个比较大的任务，拆分成多个大小合适的任务，交给合适的线程去完成，强调的是性能。

Java中提供的Executor、Fork/Join和Future都是实现分工的一种方式。



#### 同步问题

指的是一个线程执行完任务后，如何通知其他的线程继续执行，强调的是性能。当线程执行的条件不满足时，线程需要继续等待，一旦条件满足，就需要唤醒等待的线程继续执行。

Java中提供了一些实现线程之间同步的工具类，比如说：CountDownLatch、 CyclicBarrier 等。



#### 互斥问题

同一时刻只允许一个线程访问共享变量，强调的是线程执行任务的正确性。

如果多个线程同时访问同一个共享变量，则可能会发生意想不到的后果，而这种意想不到的后果主要是由线程的可见性、原子性和有序性问题产生的。而解决可见性、原子性和有序性问题的核心，就是互斥。

Java中提供的synchronized、Lock、ThreadLocal、final关键字等都可以解决互斥的问题。



### 问题原因
缓存导致的可见性问题、线程切换导致的原子性问题、编译优化导致的有序性问题。



#### 可见性
可见性问题，可以这样理解为一个线程修改了共享变量，另一个线程不能立刻看到，这是由于CPU添加了缓存导致的问题。

单核CPU不存在可见性问题，因为在单核CPU上，无论创建了多少个线程，同一时刻只会有一个线程能够获取到CPU的资源来执行任务，即使这个单核的CPU已经添加了缓存。

多核CPU上，每个CPU的内核都有自己的缓存。当多个不同的线程运行在不同的CPU内核上时，这些线程操作的是不同的CPU缓存。一个线程对其绑定的CPU的缓存的写操作，对于另外一个线程来说，不一定是可见的，这就造成了线程的可见性问题。

##### Java中的可见性问题
使用Java语言编写并发程序时，如果线程使用变量时，会把主内存中的数据复制到线程的私有内存，也就是工作内存中，每个线程读写数据时，都是操作自己的工作内存中的数据。

此时，Java中线程读写共享变量的模型与多核CPU类似，原因是Java并发程序运行在多核CPU上时，线程的私有内存，也就是工作内存就相当于多核CPU中每个CPU内核的缓存了。

```java
@Slf4j
public class ConcurrentTest {
    private int viewableCount;
    private void plusViewableCount() {
        viewableCount++;
    }

    @Test
    @SneakyThrows
    public void viewablePrincipleTest(){
        Thread threadA = new Thread(() -> {
            for(int i = 0; i < 1000; i++){
                plusViewableCount();
            }
        });

        Thread threadB = new Thread(() -> {
            for(int i = 0; i < 1000; i++){
                plusViewableCount();
            }
        });

        threadA.start();
        threadB.start();

        threadA.join();
        threadB.join();
        log.info("viewableCount result : " + viewableCount);
    }
}
```

> 执行结果
```
viewableCunt result : 1483
```

而在整个计算的过程中，线程A和线程B都是基于各自工作内存中的viewableCount值进行计算。这就导致了最终的count值小于2000。



#### 原子性
原子性是指一个或者多个操作在CPU中执行的过程不被中断的特性。原子性操作一旦开始运行，就会一直到运行结束为止，中间不会有中断的情况发生。



##### 线程切换
在并发编程中，往往设置的线程数目会大于CPU数目，而每个CPU在同一时刻只能被一个线程使用。而CPU资源的分配采用了时间片轮转策略，也就是给每个线程分配一个时间片，线程在这个时间片内占用CPU的资源来执行任务。当占用CPU资源的线程执行完任务后，会让出CPU的资源供其他线程运行，这就是任务切换，也叫做线程切换或者线程的上下文切换。

线程在执行某项操作时，此时由于CPU发生了线程切换，CPU转而去执行其他的任务，中断了当前线程执行的操作，这就会造成原子性问题。



#### 有序性



### 代码重排序

在执行程序时，为了提供性能，处理器和编译器常常会对指令进行重排序，但是不能随意重排序，不是你想怎么排序就怎么排序，它需要满足以下两个条件：

- 在单线程环境下不能改变程序运行的结果；
- 存在数据依赖关系的不允许重排序

需要注意的是：重排序不会影响单线程环境的执行结果，但是会破坏多线程的执行语义。



#### as-if-serial

#### happens-before

- as-if-serial语义保证单线程内程序的执行结果不被改变，happens-before关系保证正确同步的多线程程序的执行结果不被改变。
- as-if-serial语义给编写单线程程序的程序员创造了一个幻境：单线程程序是按程序的顺序来执行的。happens-before关系给编写正确同步的多线程程序的程序员创造了一个幻境：正确同步的多线程程序是按happens-before指定的顺序来执行的。
- as-if-serial语义和happens-before这么做的目的，都是为了在不改变程序执行结果的前提下，尽可能地提高程序执行的并行度。



### 乐观锁和悲观锁

悲观锁：总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。再比如 Java 里面的同步原语 synchronized 关键字的实现也是悲观锁。

乐观锁：顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库提供的类似于 write_condition 机制，其实都是提供的乐观锁。在 Java中 java.util.concurrent.atomic 包下面的原子变量类就是使用了乐观锁的一种实现方式 CAS 实现的。



**乐观锁的实现方式：**

1、使用版本标识来确定读到的数据与提交时的数据是否一致。提交后修改版本标识，不一致时可以采取丢弃和再次尝试的策略。

2、java 中的 Compare and Swap 即 CAS ，当多个线程尝试使用 CAS 同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。 CAS 操作中包含三个操作数 —— 需要读写的内存位置（V）、进行比较的预期原值（A）和拟写入的新值(B)。如果内存位置 V 的值与预期原值 A 相匹配，那么处理器会自动将该位置值更新为新值 B。否则处理器不做任何操作。



### Java内存模型

Java内存模型规定和指引Java程序在不同的内存架构、CPU和操作系统间有确定性地行为。它在多线程的情况下尤其重要。Java内存模型对一个线程所做的变动能被其它线程可见提供了保证，它们之间是先行发生关系。这个关系定义了一些规则让程序员在并发编程时思路更清晰。比如，先行发生关系确保了：

- 线程内的代码能够按先后顺序执行，这被称为程序次序规则。
- 对于同一个锁，一个解锁操作一定要发生在时间上后发生的另一个锁定操作之前，也叫做管程锁定规则。
- 前一个对volatile的写操作在后一个volatile的读操作之前，也叫volatile变量规则。
- 一个线程内的任何操作必需在这个线程的start()调用之后，也叫作线程启动规则。
- 一个线程的所有操作都会在线程终止之前，线程终止规则。
- 一个对象的终结操作必需在这个对象构造完成之后，也叫对象终结规则。
- 可传递性



## 关键字

### volatile

#### 功能描述

volatile关键字是Java虚拟机提供的的最轻量级的同步机制。它作为一个修饰符，用来修饰变量。**它保证变量对所有线程可见性，禁止指令重排（保证有序性），但是不保证原子性**。



volatile是一个特殊的修饰符，只有成员变量才能使用它。在Java并发程序缺少同步类的情况下，多线程对成员变量的操作对其它线程是透明的。volatile变量可以保证下一个读取操作会在前一个写操作之后发生。线程都会直接从内存中读取该变量并且不缓存它。这就确保了线程读取到的变量是同内存中是一致的。

对于可见性，Java 提供了 volatile 关键字来**保证可见性**和**禁止指令重排**。 volatile 提供 happens-before 的保证，确保一个线程的修改能对其他线程是可见的。当一个共享变量被 volatile 修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。

从实践角度而言，volatile 的一个重要作用就是和 CAS 结合，保证了原子性。volatile 常用于多线程环境下的单次操作(单次读或者单次写)。



在 JDK1.2 之前，Java 的内存模型实现总是从**主存**（即共享内存）读取变量，是不需要进行特别的注意的。而在当前的 Java 内存模型下，线程可以把变量保存**本地内存**（比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成**数据的不一致**。



![JMM(Java内存模型)](../../Image/2022/07/220716-4.png)

要解决这个问题，就需要把变量声明为**`volatile`**，这就指示 JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。

所以，**`volatile` 关键字 除了防止 JVM 的指令重排 ，还有一个重要的作用就是保证变量的可见性。**



![volatile关键字的可见性](../../Image/2022/07/220716-5.png)



#### JMM内存模型

Java虚拟机规范试图定义一种Java内存模型,来屏蔽掉各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台上都能达到一致的内存访问效果。

Java内存模型规定所有的变量都是存在主内存当中，每个线程都有自己的工作内存。这里的变量包括实例变量和静态变量，但是不包括局部变量，因为局部变量是线程私有的。

线程的工作内存保存了被该线程使用的变量的主内存副本，线程对变量的所有操作都必须在工作内存中进行，而不能直接操作主内存。并且每个线程不能访问其他线程的工作内存。



##### 内存屏障

volatile变量，保证新值能立即同步回主内存，以及每次使用前立即从主内存刷新，所以我们说volatile保证了多线程操作变量的可见性。volatile保证可见性和禁止指令重排，都跟内存屏障有关。

```java
public class Singleton {  
   private volatile static Singleton instance;  
   private Singleton (){}  
   public static Singleton getInstance() {  
   if (instance == null) {  
       synchronized (Singleton.class) {  
       if (instance == null) {  
           instance = new Singleton();  
       }  
       }  
   }  
   return instance;  
   }  
}  
```



编译后，对比有`volatile`关键字和没有`volatile`关键字时所生成的汇编代码，发现有`volatile`关键字修饰时，会多出一个`lock addl $0x0,(%esp)`，即多出一个lock前缀指令，lock指令相当于一个内存屏障，它保证以下这几点：

1. 重排序时不能把后面的指令重排序到内存屏障之前的位置
2. 将本处理器的缓存写入内存
3. 如果是写入动作，会导致其他处理器中对应的缓存无效。

第2点和第3点就是保证`volatile`保证可见性的体现，第1点就是**禁止指令重排的体现**。

内存屏障四大分类：（Load 代表读取指令，Store代表写入指令）

![图片](../../Image/2022/09/220922-2.jpg)

- 在每个volatile写操作的前面插入一个StoreStore屏障。
- 在每个volatile写操作的后面插入一个StoreLoad屏障。
- 在每个volatile读操作的后面插入一个LoadLoad屏障。
- 在每个volatile读操作的后面插入一个LoadStore屏障。



![图片](../../Image/2022/09/220922-3.jpg)

内存屏障保证前面的指令先执行，所以这就保证了禁止了指令重排啦，同时内存屏障保证缓存写入内存和其他处理器缓存失效，这也就保证了可见性。



#### 扩展

##### 修饰数组

Java 中可以创建 volatile 类型数组，不过只是一个指向数组的引用，而不是整个数组。意思是，如果改变引用指向的数组，将会受到 volatile 的保护，但是如果多个线程同时改变数组的元素，volatile 标示符就不能起到之前的保护作用了。



##### volatile和synchronized的区别

`synchronized` 关键字和 `volatile` 关键字是两个互补的存在，而不是对立的存在！

- **`volatile` 关键字**是线程同步的**轻量级实现**，所以**`volatile`性能肯定比`synchronized`关键字要好**。但是**`volatile` 关键字只能用于变量而 `synchronized` 关键字可以修饰方法以及代码块**。
- **`volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。**
- **`volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized` 关键字解决的是多个线程之间访问资源的同步性。**



synchronized 表示只有一个线程可以获取作用对象的锁，执行代码，阻塞其他线程。

volatile 表示变量在 CPU 的寄存器中是不确定的，必须从主存中读取。保证多线程环境下变量的可见性；禁止指令重排序。

**区别**

- volatile 是变量修饰符；synchronized 可以修饰类、方法、变量。
- volatile 仅能实现变量的修改可见性，不能保证原子性；而 synchronized 则可以保证变量的修改可见性和原子性。
- volatile 不会造成线程的阻塞；synchronized 可能会造成线程的阻塞。
- volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化。
- **volatile关键字**是线程同步的**轻量级实现**，所以**volatile性能肯定比synchronized关键字要好**。但是**volatile关键字只能用于变量而synchronized关键字可以修饰方法以及代码块**。synchronized关键字在JavaSE1.6之后进行了主要包括为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁以及其它各种优化之后执行效率有了显著提升，**实际开发中使用 synchronized 关键字的场景还是更多一些**。



##### volatile 变量和 atomic 变量有什么不同

volatile 变量可以确保先行关系，即写操作会发生在后续的读操作之前, 但它并不能保证原子性。例如用 volatile 修饰 count 变量，那么 count++ 操作就不是原子性的。

而 AtomicInteger 类提供的 atomic 方法可以让这种操作具有原子性如getAndIncrement()方法会原子性的进行增量操作把当前值加一，其它数据类型和引用变量也可以进行相似操作。



##### volatile 能使得一个非原子操作变成原子操作吗

关键字volatile的主要作用是使变量在多个线程间可见，但无法保证原子性，对于多个线程访问同一个实例变量需要加锁进行同步。

虽然volatile只能保证可见性不能保证原子性，但用volatile修饰long和double可以保证其操作原子性。



##### volatile 修饰符的有过什么实践

###### 单例模式

对于Double-Check这种可能出现的问题（当然这种概率已经非常小了，但毕竟还是有的嘛~），解决方案是：只需要给instance的声明加上volatile关键字即可volatile关键字的一个作用是禁止指令重排，把instance声明为volatile之后，对它的写操作就会有一个内存屏障（什么是内存屏障？），这样，在它的赋值完成之前，就不用会调用读操作。注意：volatile阻止的不是singleton = newSingleton()这句话内部[1-2-3]的指令重排，而是保证了在一个写操作（[1-2-3]）完成之前，不会调用读操作（if (instance == null)）。

```java
public class Singleton7 {

    private static volatile Singleton7 instance = null;

    private Singleton7() {}

    public static Singleton7 getInstance() {
        if (instance == null) {
            synchronized (Singleton7.class) {
                if (instance == null) {
                    instance = new Singleton7();
                }
            }
        }

        return instance;
    }

}
```



### final

不可变对象(Immutable Objects)即对象一旦被创建它的状态（对象的数据，也即对象属性值）就不能改变，反之即为可变对象(Mutable Objects)。

不可变对象的类即为不可变类(Immutable Class)。Java 平台类库中包含许多不可变类，如 String、基本类型的包装类、BigInteger 和 BigDecimal 等。

只有满足如下状态，一个对象才是不可变的；

- 它的状态不能在创建后再被修改；
- 所有域都是 final 类型；并且，它被正确创建（创建期间没有发生 this 引用的逸出）。

不可变对象保证了对象的内存可见性，对不可变对象的读取不需要进行额外的同步手段，提升了代码执行效率。



## 线程池

### 简介

服务器在创建和销毁线程上花费的时间和消耗的系统资源都相当大，甚至可能要比在处理实际的用户请求的时间和资源要多的多。

如果并发的请求数量非常多，但每个线程执行的时间很短，这样就会频繁的创建和销毁线程，如此一来会大大降低系统的效率。可能出现服务器在为每个请求创建新线程和销毁线程上花费的时间和消耗的系统资源要比处理实际的用户请求的时间和资源更多。

通过对多个任务重复使用线程，线程创建的开销就被分摊到了多个任务上了，而且由于在请求到达时线程已经存在，所以消除了线程创建所带来的延迟。通过适当的调整线程中的线程数目可以防止出现资源不足的情况。



创建线程要花费昂贵的资源和时间，如果任务来了才创建线程那么响应时间会变长，而且一个进程能创建的线程数有限。

多线程技术主要解决处理器单元内多个线程执行的问题，它可以显著减少处理器单元的闲置时间，增加处理器单元的吞吐能力。
假设一个服务器完成一项任务所需时间为：T1 创建线程时间，T2 在线程中执行任务的时间，T3 销毁线程时间。

如果：T1 + T3 远大于 T2，则可以采用线程池，以提高服务器性能。

线程池技术正是关注如何缩短或调整T1,T3时间的技术，从而提高服务器程序性能的。它把T1，T3分别安排在服务器程序的启动和结束的时间段或者一些空闲的时间段，这样在服务器程序处理客户请求时，不会有T1，T3的开销了。

线程池不仅调整T1,T3产生的时间段，而且它还显著减少了创建线程的数目，看一个例子：

假设一个服务器一天要处理50000个请求，并且每个请求需要一个单独的线程完成。在线程池中，线程数一般是固定的，所以产生线程总数不会超过线程池中线程的数目，而如果服务器不利用线程池来处理这些请求则线程总数为50000。一般线程池大小是远小于50000。所以利用线程池的服务器程序不会为了创建50000而在处理请求时浪费时间，从而提高效率。

为了避免这些问题，在程序启动的时候就创建若干线程来响应处理，它们被称为线程池，里面的线程叫工作线程。从JDK1.5开始，Java API提供了Executor框架让你可以创建不同的线程池。比如单线程池，每次处理一个任务；数目固定的线程池或者是缓存线程池（一个适合很多生存期短的任务的程序的可扩展线程池）。



### 线程池优点

1.重用存在的线程，减少线程创建，消亡的开销，提高性能；

2.提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。

3.提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。



在java程序中，其实经常需要用到多线程来处理一些业务，但是不建议单纯使用继承Thread或者实现Runnable接口的方式来创建线程，那样就会导致频繁创建及销毁线程，同时创建过多的线程也可能引发资源耗尽的风险。所以在这种情况下，使用线程池是一种更合理的选择，方便管理任务，实现了线程的重复利用。所以线程池一般适合那种需要异步或者多线程处理任务的场景。



> **池化技术的思想主要是为了减少每次获取资源的消耗，提高对资源的利用率。线程池、数据库连接池、Http 连接池等等都是对这个思想的应用。**



**线程池**提供了一种限制和管理资源（包括执行一个任务）。 每个**线程池**还维护一些基本统计信息，例如已完成任务的数量。**使用线程池的好处**：

- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度**。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。



### 线程池状态

线程池内部有5个常量来代表线程池的五种状态：

> java.util.concurrent.ThreadPoolExecutor

```java
// 高3位为111 运行状态
private static final int RUNNING    = -1 << COUNT_BITS;
// 高3位为000 关闭状态
private static final int SHUTDOWN   =  0 << COUNT_BITS;
// 高3位为001 停止状态
private static final int STOP       =  1 << COUNT_BITS;
// 高3位为010 整理状态
private static final int TIDYING    =  2 << COUNT_BITS;
// 高3位为011 销毁状态
private static final int TERMINATED =  3 << COUNT_BITS;

// 存储线程池状态和线程池中线程数
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
```



ctl 是对线程池的运行状态和线程池中有效线程的数量进行控制的一个字段， 它包含两部分的信息: 线程池的运行状态 (runState) 和线程池内有效线程的数量 (workerCount)，

可以看到，使用了Integer类型来保存，高3位保存runState，低29位保存workerCount。COUNT_BITS 就是29，CAPACITY就是1左移29位减1（29个1），这个常量表示workerCount的上限值，大约是5亿。



#### 运行状态

**运行（RUNNING）**：线程池创建时就是这个状态，能够接收新任务，以及对已添加的任务进行处理。



#### 关闭状态

**关闭（SHUTDOWN）**：调用shutdown方法线程池就会转换成SHUTDOWN状态，此时线程池不再接收新任务，但能继续处理已添加的任务到队列中任务。



#### 停止状态

**停止（STOP）**：调用shutdownNow方法线程池就会转换成STOP状态，不接收新任务，也不能继续处理已添加的任务到队列中任务，并且会尝试中断正在处理的任务的线程。



#### 整理状态

**整理（TIDYING）**：SHUTDOWN 状态下，ctl记录的任务数为 0， 其他所有任务已终止，线程池会变为 TIDYING 状态；线程池在 SHUTDOWN 状态，任务队列为空且执行中任务为空，线程池会变为 TIDYING 状态；线程池在 STOP 状态，线程池中执行中任务为空时，线程池会变为 TIDYING 状态。

当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理； 可以通过重载terminated()函数来实现。



#### 销毁状态

**销毁（TERMINATED）**：线程池彻底终止。线程池在 TIDYING 状态执行完 terminated() 方法就会转变为 TERMINATED 状态。



#### 总结

5种状态的流转如下：

![图片](../../Image/2022/07/220722-3.png)

在线程池运行过程中，绝大多数操作执行前都得判断当前线程池处于哪种状态，再来决定是否继续执行该操作。



### 线程池组成部分

一个线程池包括以下四个基本组成部分：

1、线程池管理器（ThreadPool）：用于创建并管理线程池，包括创建线程池，销毁线程池，添加新任务；

2、工作线程（PoolWorker）：线程池中线程，在没有任务时处于等待状态，可以循环的执行任务；

3、任务接口（Task）：每个任务必须实现的接口，以供工作线程调度任务的执行，它主要规定了任务的入口，任务执行完后的收尾工作，任务的执行状态等；

4、任务队列（taskQueue）：用于存放没有处理的任务。提供一种缓冲机制。



### 对比Thread创建线程
Thread直接创建线程的弊端

- 每次new Thread新建对象，性能差。

- 线程缺乏统一管理，可能无限制的新建线程，相互竞争，有可能占用过多系统资源导致死机或OOM。

- 缺少更多的功能，如更多执行、定期执行、线程中断。





### 线程池原理

#### 创建线程池

创建线程池的方法有两种，通过ThreadPoolExecutor构造方法实现和通过Executors工具类方法来实现。



##### ThreadPoolExecutor方式

> java.util.concurrent.ThreadPoolExecutor

```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```



`ThreadPoolExecutor` 类中提供四个构造方法，其余三个都是在这个构造方法的基础上产生。



**`ThreadPoolExecutor` 3 个最重要的参数：**

- **`corePoolSize`(int)**： 核心线程数线程数定义了最小可以同时运行的线程数量。会一直存在，除非allowCoreThreadTimeOut设置为true。
- **`maximumPoolSize`(int) **：最大线程数，线程池允许创建的最大线程数。当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
- **`workQueue`**：当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。



**ThreadPoolExecutor 其他常见参数：**

1. **`keepAliveTime`(long)**：超出 corePoolSize 后创建的线程存活时间或者是所有线程最大存活时间，取决于配置。线程池中**非核心线程空闲的存活时间大小**
2. **`unit`** ：`keepAliveTime` 参数的时间单位。
3. **`threadFactory`** ：线程池内部创建线程所用的工厂。它是ThreadFactory类型的变量，用来创建新线程。默认使用Executors.defaultThreadFactory() 来创建线程。使用默认的ThreadFactory来创建线程时，会使新创建的线程具有相同的NORM_PRIORITY优先级并且是非守护线程，同时也设置了线程的名称。
4. **`handler`** ：拒绝策略。当队列已满并且线程数量达到最大线程数量时，会调用该方法处理该任务。



##### Executors方式

《阿里巴巴 Java 开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

Executor框架同java.util.concurrent.Executor 接口在Java 5中被引入。Executor框架是一个根据一组执行策略调用，调度，执行和控制的异步任务的框架。

无限制的创建线程会引起应用程序内存溢出。所以创建一个线程池是个更好的的解决方案，因为可以限制线程的数量并且可以回收再利用这些线程。利用Executor框架可以非常方便的创建一个线程池。



###### FixedThreadPool

> java.util.concurrent.Executors

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory);
}
```



- 核心线程数和最大线程数大小一样
- 没有所谓的非空闲时间，即keepAliveTime为0
- 阻塞队列为无界队列LinkedBlockingQueue



**FixedThreadPool** ： 该方法返回一个固定线程数量的线程池，核心线程数与最大线程数相等。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。

固定数量的线程池，每提交一个任务就是一个线程，直到达到线程池的最大数量，然后后面进入等待队列直到前面的任务完成才继续执行。

创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。



> java.util.concurrent.LinkedBlockingQueue

```java
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}
```

阻塞队列的大小是Integer.MAX_VALUE，相当于无界的阻塞队列的了。



###### SingleThreadExecutor

> java.util.concurrent.Executors

```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
```



- 核心线程数为1
- 最大线程数也为1
- 阻塞队列是LinkedBlockingQueue
- keepAliveTime为0



适用于串行执行任务的场景，一个任务一个任务地执行。



**SingleThreadExecutor：** 方法返回一个只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。

单个线程的线程池，即线程池中每次只有一个线程工作，单线程串行执行任务。

创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。



###### CachedThreadPool

> java.util.concurrent.Executors

```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

    public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
    }
```



- 核心线程数为0
- 最大线程数为Integer.MAX_VALUE
- 阻塞队列是SynchronousQueue
- 非核心线程空闲存活时间为60秒



当提交任务的速度大于处理任务的速度时，每次提交一个任务，就必然会创建一个线程。极端情况下会创建过多的线程，耗尽 CPU 和内存资源。由于空闲 60 秒的线程会被终止，长时间保持空闲的 CachedThreadPool 不会占用任何资源。



**CachedThreadPool：** 接近无限大线程数量的线程池，该方法返回一个可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用。

可缓存线程池，当线程池大小超过了处理任务所需的线程，那么就会回收部分空闲（一般是60秒无执行）的线程，当有任务来时，又智能的添加新线程来执行。

创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，

那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。



###### ScheduledThreadPool

> java.util.concurrent.Executors

```java
    /**
     * 创建一个线程池，可以安排命令在给定延迟后运行，或定期执行。
     */
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }

    public static ScheduledExecutorService newScheduledThreadPool(
            int corePoolSize, ThreadFactory threadFactory) {
        return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
    }
```



> java.util.concurrent.ScheduledThreadPoolExecutor

```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```



- 最大线程数为Integer.MAX_VALUE
- 阻塞队列是DelayedWorkQueue
- keepAliveTime为0
- scheduleAtFixedRate() ：按某种速率周期执行
- scheduleWithFixedDelay()：在某个延迟后执行



大小无限制的线程池，支持定时和周期性的执行线程。

创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。



##### 创建线程池总结

因为从上面构造线程池可以看出，newFixedThreadPool线程池，由于使用了LinkedBlockingQueue，队列的容量默认是无限大，实际使用中出现任务过多时会导致内存溢出；newCachedThreadPool线程池由于核心线程数无限大，当任务过多的时候，会导致创建大量的线程，可能机器负载过高，可能会导致服务宕机。



Executors 返回线程池对象的弊端如下：

- **FixedThreadPool 和 SingleThreadExecutor** ： 允许请求的队列长度为 Integer.MAX_VALUE ，可能堆积大量的请求，从而导致 OOM。
- **CachedThreadPool 和 ScheduledThreadPool** ： 允许创建的线程数量为 Integer.MAX_VALUE ，可能会创建大量线程，从而导致 OOM。



避免使用Executors创建线程池，主要是避免使用其中的默认实现，那么我们可以自己直接调用ThreadPoolExecutor的构造函数来自己创建线程池。在创建的同时，给BlockQueue指定容量就可以了。

```java
private static ExecutorService executor = new ThreadPoolExecutor(10, 10,
        60L, TimeUnit.SECONDS,
        new ArrayBlockingQueue(10));
```



#### 提交任务

##### execute

> java.util.concurrent.ThreadPoolExecutor

```java
   // 存放线程池的运行状态 (runState) 和线程池内有效线程的数量 (workerCount)
   private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

    private static int workerCountOf(int c) {
        return c & CAPACITY;
    }

    private final BlockingQueue<Runnable> workQueue;

    public void execute(Runnable command) {
        // 如果任务为null，则抛出异常。
        if (command == null)
            throw new NullPointerException();
        // ctl 中保存的线程池当前的一些状态信息
        int c = ctl.get();

        // 1.首先判断当前线程池中之行的任务数量是否小于 corePoolSize
        // 如果小于的话，通过addWorker(command, true)新建一个线程，并将任务(command)添加到该线程中；
        // 然后，启动该线程从而执行任务。
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        // 2.如果当前之行的任务数量大于等于 corePoolSize 的时候就会走到这里
        // 通过 isRunning 方法判断线程池状态，线程池处于 RUNNING 状态才会被并且队列可以加入任务，该任务才会被加入进去
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            // 再次获取线程池状态，如果线程池状态不是 RUNNING 状态就需要从任务队列中移除任务，
            // 并尝试判断线程是否全部执行完毕。同时执行拒绝策略。
            if (!isRunning(recheck) && remove(command))
                reject(command);
            // 这一步其实没有很大意义，除非出现线程池所有线程完蛋了，但是队列还有任务的情况。
            // （一般是进入时时运行态，然后遇到状态变更的情况）
            else if (workerCountOf(recheck) == 0)
                // 如果当前线程池为空就新创建一个线程并执行。
                addWorker(null, false);
        }
        //3. 通过addWorker(command, false)新建一个线程，并将任务(command)添加到该线程中；然后，启动该线程从而执行任务。
        //如果addWorker(command, false)执行失败，则通过reject()执行相应的拒绝策略的内容。
        else if (!addWorker(command, false))
            reject(command);
    }

```



![图片](../../Image/2022/09/220922-13.jpg)

在代码中模拟了 10 个任务，我们配置的核心线程数为 5 、等待队列容量为 100 ，所以每次只可能存在 5 个任务同时执行，剩下的 5 个任务会被放到等待队列中去。当前的 5 个任务之行完成后，才会之行剩下的 5 个任务。

![img](../../Image/2022/07/220722-1.png)



###### 判断核心线程数

提交任务后判断当前线程池的线程数是否小于核心线程数，如果小于，那么就直接通过ThreadFactory创建一个线程来执行这个任务。

当任务执行完之后，线程不会退出，而是会去从阻塞队列中获取任务。

提交任务的时候，就算有线程池里的线程从阻塞队列中获取不到任务，如果线程池里的线程数还是小于核心线程数，那么依然会继续创建线程，而不是复用已有的线程。



###### 入等待队列

线程池里的线程数不小于核心线程数时就会尝试将任务放入阻塞队列中，入队成功之后，阻塞的线程就可以获取到任务。



###### 判断最大线程数

等待队列已经满了，任务放入失败了时，判断当前线程池里的线程数是否小于最大线程数，如果小于最大线程数，那么也会创建非核心线程来执行提交的任务。

就算队列中有任务，新创建的线程还是优先处理这个提交的任务，而不是从队列中获取已有的任务执行，从这可以看出，先提交的任务不一定先执行。

线程数已经达到了最大线程数量时，就会执行拒绝策略。



##### submit

> java.util.concurrent.AbstractExecutorService

```java
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}

protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
    return new FutureTask<T>(callable);
}

public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}

protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
    return new FutureTask<T>(runnable, value);
}

public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}
```



######  execute()方法和 submit()方法的区别

1. **`execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；**

2. **`submit()`方法用于提交需要返回值的任务。线程池会返回一个 `Future` 类型的对象，通过这个 `Future` 对象可以判断任务是否执行成功**，并且可以通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用 `get（long timeout，TimeUnit unit）`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。




- execute和submit都属于线程池的方法，execute只能提交Runnable类型的任务，而submit既能提交Runnable类型任务也能提交Callable类型任务。
- execute会直接抛出任务执行时的异常，submit会吃掉异常，可通过Future的get方法将任务执行时的异常重新抛出。
- execute所属顶层接口是Executor,submit所属顶层接口是ExecutorService，实现类ThreadPoolExecutor重写了execute方法,抽象类AbstractExecutorService重写了submit方法。



#### 线程复用

线程池的核心功能就是实现了线程的重复利用，线程在线程池内部被封装成一个Worker对象，Worker继承了AQS，也就是有一定锁的特性。

> java.util.concurrent.ThreadPoolExecutor.Worker

```java
    private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
    {
        /**
         * This class will never be serialized, but we provide a
         * serialVersionUID to suppress a javac warning.
         */
        private static final long serialVersionUID = 6138294804551838833L;

        /** Thread this worker is running in.  Null if factory fails. */
        final Thread thread;
        /** Initial task to run.  Possibly null. */
        Runnable firstTask;
        /** Per-thread task counter */
        volatile long completedTasks;

        /**
         * Creates with given first task and thread from ThreadFactory.
         * @param firstTask the first task (null if none)
         */
        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);
        }

        /** Delegates main run loop to outer runWorker  */
        public void run() {
            runWorker(this);
        }

        // Lock methods
        //
        // The value 0 represents the unlocked state.
        // The value 1 represents the locked state.

        protected boolean isHeldExclusively() {
            return getState() != 0;
        }

        protected boolean tryAcquire(int unused) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        protected boolean tryRelease(int unused) {
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        public void lock()        { acquire(1); }
        public boolean tryLock()  { return tryAcquire(1); }
        public void unlock()      { release(1); }
        public boolean isLocked() { return isHeldExclusively(); }

        void interruptIfStarted() {
            Thread t;
            if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                }
            }
        }
    }
```



##### 创建线程

创建线程来执行任务的方法上面提到是通过addWorker方法创建的。在创建Worker对象的时候，会把线程和任务一起封装到Worker内部，然后调用runWorker方法来让线程执行任务。

> java.util.concurrent.ThreadPoolExecutor

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        // 获取线程池的状态
        int c = ctl.get();
        int rs = runStateOf(c);

        // 如果是非运行状态（因为只有运行状态是负数）
        // 判断是不是关闭状态，不接收新任务，但能处理已添加的任务
        // 任务是不是空任务，队列是不是空（这一步说明了关闭状态不接受任务）
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            // 获取活动线程数
            int wc = workerCountOf(c);
            // 检验线程数是否大于容量值【这是避免设置的非核心线程数没有限制大小】
            // 根据传入参数判断核心线程数与非核心线程数是否达到了最大值
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            // 尝试增加workerCount数量【也就是活跃线程数+1】，如果成功，则跳出第一个for循环
            if (compareAndIncrementWorkerCount(c))
                break retry;
            // 如果增加workerCount失败，则重新获取ctl的值
            c = ctl.get();
            // 如果当前的运行状态不等于rs，说明状态已被改变，返回第一个for循环继续执行
            if (runStateOf(c) != rs)
                continue retry;
        }
    }
    // 线程启动标志
    boolean workerStarted = false;
    // 线程添加标志
    boolean workerAdded = false;
    Worker w = null;
    try {
        // 根据firstTask来创建Worker对象，每一个Worker对象都会创建一个线程
        w = new Worker(firstTask);
        final Thread t = w.thread;
        // 如果过线程不为空，则试着将线程加入工作队列中
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            // 加重入锁
            mainLock.lock();
            try {
                // 重新获取线程的状态
                int rs = runStateOf(ctl.get());
                // 是否线程池正处于运行状态
                // 线程池是否处于关闭状态 且 传入的任务为空
                //（说明关闭状态还是能添加工作者，但是不允许添加任务）
                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    // 判断线程是否存活
                    if (t.isAlive()) 
                        throw new IllegalThreadStateException();
                    // workers是一个HashSet,将该worker对象添加其中
                    workers.add(w);
                    // 记录线程工作者的值
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    // 修改添加标记
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            // 如果添加成功，则启动线程
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            // 添加工作者失败方法
            addWorkerFailed(w);
    }
    return workerStarted;
}
```



> java.util.concurrent.ThreadPoolExecutor.Worker

```java
Worker(Runnable firstTask) {
    setState(-1); // inhibit interrupts until runWorker
    this.firstTask = firstTask;
    this.thread = getThreadFactory().newThread(this);
}
```



> java.util.concurrent.ThreadPoolExecutor

```java
private void addWorkerFailed(Worker w) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 移除工作者
        if (w != null)
            workers.remove(w);
        // 任务数量减一
        decrementWorkerCount();
        // 进入整理状态
        tryTerminate();
    } finally {
        mainLock.unlock();
    }
}
```



##### 执行线程

> java.util.concurrent.ThreadPoolExecutor

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```



可以看出线程执行完任务不会退出的原因，runWorker内部使用了while死循环，当第一个任务执行完之后，会不断地通过getTask方法获取任务，只要能获取到任务，就会调用run方法，继续执行任务，这就是线程能够复用的主要原因。

如果从getTask获取不到方法的时候，最后就会调用finally中的processWorkerExit方法，来将线程退出。

因为Worker继承了AQS，每次在执行任务之前都会调用Worker的lock方法，执行完任务之后，会调用unlock方法，这样做的目的就可以通过Woker的加锁状态就能判断出当前线程是否正在运行任务。如果想知道线程是否正在运行任务，只需要调用Woker的tryLock方法，根据是否加锁成功就能判断，加锁成功说明当前线程没有加锁，也就没有执行任务了，在调用shutdown方法关闭线程池的时候，就用这种方式来判断线程有没有在执行任务，如果没有的话，来尝试打断没有执行任务的线程。



#### 再次获取任务

线程在执行完任务之后，会继续从getTask方法中获取任务，获取不到就会退出。

> java.util.concurrent.ThreadPoolExecutor

```java
    private Runnable getTask() {
        boolean timedOut = false; // Did the last poll() time out?

        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
                decrementWorkerCount();
                return null;
            }

            int wc = workerCountOf(c);

            // 判断当前过来获取任务的线程是否可以超时退出
            boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

            if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
                if (compareAndDecrementWorkerCount(c))
                    return null;
                continue;
            }

            try {
                // 超时退出利用队列的 poll 方法实现，超时则会返回null，然后退出线程
                // 没有设置超时则会调用 take 方法阻塞获取任务
                Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
                if (r != null)
                    return r;
                timedOut = true;
            } catch (InterruptedException retry) {
                timedOut = false;
            }
        }
    }
```



根据是否允许超时来选择调用阻塞队列workQueue的poll方法或者take方法。如果允许超时，则会调用poll方法，传入keepAliveTime，也就是构造线程池时传入的空闲时间，这个方法的意思就是从队列中阻塞keepAliveTime时间来获取任务，获取不到就会返回null；如果不允许超时，就会调用take方法，这个方法会一直阻塞获取任务，直到从队列中获取到任务位置。从这里可以看到keepAliveTime是如何使用的了。

所以到这里应该就知道线程池中的线程为什么可以做到空闲一定时间就退出了吧。其实最主要的是利用了阻塞队列的poll方法的实现，这个方法可以指定超时时间，一旦线程达到了keepAliveTime还没有获取到任务，那么就会返回null，上一小节提到，getTask方法返回null，线程就会退出。

这里也有一个细节，就是判断当前获取任务的线程是否可以超时退出的时候，如果将allowCoreThreadTimeOut设置为true，那么所有线程走到这个timed都是true，那么所有的线程，包括核心线程都可以做到超时退出。如果你的线程池需要将核心线程超时退出，那么可以通过allowCoreThreadTimeOut方法将allowCoreThreadTimeOut变量设置为true。



![img](../../Image/2022/07/220722-2.png)



#### 关闭线程池

线程池使用完如果没有关闭，可能会导致内存泄漏问题。线程池提供了shutdown和shutdownNow两个方法来关闭线程池。

shutdownNow()能立即停止线程池，正在跑的和正在等待的任务都停下了。这样做立即生效，但是风险也比较大。

shutdown()只是关闭了提交通道，用submit()是无效的；而内部的任务该怎么跑还是怎么跑，跑完再彻底停止线程池。



##### shutdown

> java.util.concurrent.ThreadPoolExecutor

```java
    public void shutdown() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            advanceRunState(SHUTDOWN);
            interruptIdleWorkers();
            onShutdown(); // hook for ScheduledThreadPoolExecutor
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
    }
```



就是将线程池的状态修改为SHUTDOWN，然后尝试打断空闲的线程（如何判断空闲，上面在说Worker继承AQS的时候说过），也就是在阻塞等待任务的线程。

当我们调用shutdown后，线程池将不再接受新的任务，但也不会去强制终止已经提交或者正在执行中的任务。



##### shutdownNow

> java.util.concurrent.ThreadPoolExecutor

```java
    public List<Runnable> shutdownNow() {
        List<Runnable> tasks;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            advanceRunState(STOP);
            interruptWorkers();
            tasks = drainQueue();
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
        return tasks;
    }
```

就是将线程池的状态修改为STOP，然后尝试打断所有的线程，从阻塞队列中移除剩余的任务，这也是为什么shutdownNow不能执行剩余任务的原因。

shutdown方法和shutdownNow方法的主要区别就是，shutdown之后还能处理在队列中的任务，shutdownNow直接就将任务从队列中移除，线程池里的线程就不再处理了。

对正在执行的任务全部发出interrupt()，停止执行，对还未开始执行的任务全部取消，并且返回还没开始的任务列表。



#### 监控方法

在项目中使用线程池的时候，一般需要对线程池进行监控，方便出问题的时候进行查看。线程池本身提供了一些方法来获取线程池的运行状态。



| 方法                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| getCompletedTaskCount | 已经执行完成的任务数量                                       |
| getActiveCount        | 获取正在执行任务的线程数据                                   |
| getTaskCount          | 获取线程池已执行和未执行的任务总数                           |
| getLargestPoolSize    | 线程池里曾经创建过的最大的线程数量。这个主要是用来判断线程是否满过。 |
| getCorePoolSize       | 获取线程池核心线程数                                         |
| getPoolSize           | 获取当前线程池中线程数量的大小                               |



除了线程池提供的上述已经实现的方法，同时线程池也预留了很对扩展方法。比如在runWorker方法里面，在执行任务之前会回调beforeExecute方法，执行任务之后会回调afterExecute方法，而这些方法默认都是空实现，你可以自己继承ThreadPoolExecutor来扩展重写这些方法，来实现自己想要的功能。

##### getCompletedTaskCount

**获取已完成的任务数量**

> java.util.concurrent.ThreadPoolExecutor

```java
    /**
     * Returns the approximate total number of tasks that have
     * completed execution. Because the states of tasks and threads
     * may change dynamically during computation, the returned value
     * is only an approximation, but one that does not ever decrease
     * across successive calls.
     *
     * @return the number of tasks
     */
    public long getCompletedTaskCount() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            long n = completedTaskCount;
            for (Worker w : workers)
                n += w.completedTasks;
            return n;
        } finally {
            mainLock.unlock();
        }
    }
```



##### getActiveCount

**获取当前线程池中正在执行任务的线程数量**

> java.util.concurrent.ThreadPoolExecutor

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    /**
     * Returns the approximate number of threads that are actively
     * executing tasks.
     *
     * @return the number of threads
     */
    public int getActiveCount() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            int n = 0;
            for (Worker w : workers)
                if (w.isLocked())
                    ++n;
            return n;
        } finally {
            mainLock.unlock();
        }
    }    
}
```



##### getTaskCount

**获取线程池已执行和未执行的任务总数**

> java.util.concurrent.ThreadPoolExecutor

```java
    /**
     * Returns the approximate total number of tasks that have ever been
     * scheduled for execution. Because the states of tasks and
     * threads may change dynamically during computation, the returned
     * value is only an approximation.
     *
     * @return the number of tasks
     */
    public long getTaskCount() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            long n = completedTaskCount;
            for (Worker w : workers) {
                n += w.completedTasks;
                if (w.isLocked())
                    ++n;
            }
            return n + workQueue.size();
        } finally {
            mainLock.unlock();
        }
    }
```



##### getLargestPoolSize

> java.util.concurrent.ThreadPoolExecutor

```java
/**
 * 返回池中同时存在的最大线程数。
 */
public int getLargestPoolSize() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        return largestPoolSize;
    } finally {
        mainLock.unlock();
    }
}
```



##### getCorePoolSize

**获取线程池核心线程数**

> java.util.concurrent.ThreadPoolExecutor

```java
    /**
     * Returns the core number of threads.
     *
     * @return the core number of threads
     * @see #setCorePoolSize
     */
    public int getCorePoolSize() {
        return corePoolSize;
    }    
```



##### getPoolSize

**获取线程池当前的线程数量**

> java.util.concurrent.ThreadPoolExecutor

```java
    /**
     * Returns the current number of threads in the pool.
     *
     * @return the number of threads
     */
    public int getPoolSize() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            // Remove rare and surprising possibility of
            // isTerminated() && getPoolSize() > 0
            return runStateAtLeast(ctl.get(), TIDYING) ? 0
                : workers.size();
        } finally {
            mainLock.unlock();
        }
    }    
```



### 拒绝策略

数据源连接池一般请求的连接数超过连接池的最大值的时候就会触发拒绝策略，策略一般是阻塞等待设置的时间或者直接抛异常。

当提交的任务数大于corePoolSize时，会优先放到队列缓冲区，只有填满了缓冲区后，才会判断当前运行的任务是否大于maxPoolSize，小于时会新建线程处理。大于时就触发了拒绝策略，总结就是：当前提交任务数大于（maxPoolSize + queueCapacity）时就会触发线程池的拒绝策略了。



| 名称                          | 描述           | 提供方   |
| ----------------------------- | -------------- | -------- |
| CallerRunsPolicy              | 调用者运行策略 | JDK      |
| AbortPolicy                   | 中止策略       | JDK      |
| DiscardPolicy                 | 丢弃策略       | JDK      |
| DiscardOldestPolicy           | 丢弃最老策略   | JDK      |
| AbortPolicyWithReport         |                | Dubbo    |
| NewThreadRunsPolicy           |                | Netty    |
| ActiveMq拒绝策略              |                | ActiveMq |
| RejectedExecutionHandlerChain |                | Pinpoint |



#### JDK默认拒绝策略

当触发拒绝策略时，线程池会调用设置的具体的策略，将当前提交的任务以及线程池实例本身传递过来进行处理。所有的拒绝策略都要实现类 RejectedExecutionHandler。

> java.util.concurrent.RejectedExecutionHandler

```java
public interface RejectedExecutionHandler {
    /**
     * 
     */
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```



##### CallerRunsPolicy
用调用者所在的线程来执行任务。
> java.util.concurrent.ThreadPoolExecutor.CallerRunsPolicy
```java
/**
 * A handler for rejected tasks that runs the rejected task
 * directly in the calling thread of the {@code execute} method,
 * unless the executor has been shut down, in which case the task
 * is discarded.
 */
public static class CallerRunsPolicy implements RejectedExecutionHandler {
    /**
     * Creates a {@code CallerRunsPolicy}.
     */
    public CallerRunsPolicy() { }

    /**
     * Executes task r in the caller's thread, unless the executor
     * has been shut down, in which case the task is discarded.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            r.run();
        }
    }
}
```



功能：当触发拒绝策略时，只要线程池没有关闭，就由提交任务的当前线程处理。

使用场景：一般在不允许失败的、对性能要求不高、并发量较小的场景下使用，因为线程池一般情况下不会关闭，也就是提交的任务一定会被运行，但是由于是调用者线程自己执行的，当多次提交任务时，就会阻塞后续任务执行，性能和效率自然就慢了。



##### AbortPolicy

直接抛出异常，这也是默认的策略。
> java.util.concurrent.ThreadPoolExecutor.AbortPolicy
```java
/**
 * A handler for rejected tasks that throws a
 * {@code RejectedExecutionException}.
 */
public static class AbortPolicy implements RejectedExecutionHandler {
    /**
     * Creates an {@code AbortPolicy}.
     */
    public AbortPolicy() { }

    /**
     * Always throws RejectedExecutionException.
     *
     * @param r 请求执行的任务
     * @param e 执行指定任务的执行器
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                             " rejected from " +
                                             e.toString());
    }
}
```

功能：当触发拒绝策略时，直接抛出拒绝执行的异常，中止策略的意思也就是打断当前执行流程

使用场景：这个就没有特殊的场景了，但是一点要正确处理抛出的异常。ThreadPoolExecutor中默认的策略就是AbortPolicy，ExecutorService接口的系列ThreadPoolExecutor因为都没有显示的设置拒绝策略，所以默认的都是这个。但是请注意，ExecutorService中的线程池实例队列都是无界的，也就是说把内存撑爆了都不会触发拒绝策略。当自己自定义线程池实例时，使用这个策略一定要处理好触发策略时抛的异常，因为他会打断当前的执行流程。



##### DiscardPolicy

直接丢弃当前任务。
> java.util.concurrent.ThreadPoolExecutor.DiscardPolicy
```java
/**
 * A handler for rejected tasks that silently discards the
 * rejected task.
 */
public static class DiscardPolicy implements RejectedExecutionHandler {
    /**
     * Creates a {@code DiscardPolicy}.
     */
    public DiscardPolicy() { }

    /**
     * Does nothing, which has the effect of discarding task r.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}
```



##### DiscardOldestPolicy

丢弃队列中最靠前的任务并执行当前任务。
> java.util.concurrent.ThreadPoolExecutor.DiscardOldestPolicy
```java
/**
 * A handler for rejected tasks that discards the oldest unhandled
 * request and then retries {@code execute}, unless the executor
 * is shut down, in which case the task is discarded.
 */
public static class DiscardOldestPolicy implements RejectedExecutionHandler {
    /**
     * Creates a {@code DiscardOldestPolicy} for the given executor.
     */
    public DiscardOldestPolicy() { }

    /**
     * Obtains and ignores the next task that the executor
     * would otherwise execute, if one is immediately available,
     * and then retries execution of task r, unless the executor
     * is shut down, in which case task r is instead discarded.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            e.getQueue().poll();
            e.execute(r);
        }
    }
}
```



功能：如果线程池未关闭，就弹出队列头部的元素，然后尝试执行

使用场景：这个策略还是会丢弃任务，丢弃时也是毫无声息，但是特点是丢弃的是老的未执行的任务，而且是待执行优先级较高的任务。基于这个特性，我能想到的场景就是，发布消息，和修改消息，当消息发布出去后，还未执行，此时更新的消息又来了，这个时候未执行的消息的版本比现在提交的消息版本要低就可以被丢弃了。因为队列中还有可能存在消息版本更低的消息会排队执行，所以在真正处理消息的时候一定要做好消息的版本比较。



#### 其它拒绝策略

##### Dubbo中的线程拒绝策略

```java
public class AbortPolicyWithReport extends ThreadPoolExecutor.AbortPolicy {

    protected static final Logger logger = LoggerFactory.getLogger(AbortPolicyWithReport.class);

    private final String threadName;

    private final URL url;

    private static volatilelong lastPrintTime = 0;

    private static Semaphore guard = new Semaphore(1);

    public AbortPolicyWithReport(String threadName, URL url) {
        this.threadName = threadName;
        this.url = url;
    }

    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        String msg = String.format("Thread pool is EXHAUSTED!" +
                        " Thread Name: %s, Pool Size: %d (active: %d, core: %d, max: %d, largest: %d), Task: %d (completed: %d)," +
                        " Executor status:(isShutdown:%s, isTerminated:%s, isTerminating:%s), in %s://%s:%d!",
                threadName, e.getPoolSize(), e.getActiveCount(), e.getCorePoolSize(), e.getMaximumPoolSize(), e.getLargestPoolSize(),
                e.getTaskCount(), e.getCompletedTaskCount(), e.isShutdown(), e.isTerminated(), e.isTerminating(),
                url.getProtocol(), url.getIp(), url.getPort());
        logger.warn(msg);
        dumpJStack();
        thrownew RejectedExecutionException(msg);
    }

    private void dumpJStack() {
       //省略实现
    }
}
```

Dubbo的工作线程触发了线程拒绝后，主要做了三个事情，原则就是尽量让使用者清楚触发线程拒绝策略的真实原因。

- 输出了一条警告级别的日志，日志内容为线程池的详细设置参数，以及线程池当前的状态，还有当前拒绝任务的一些详细信息。可以说，这条日志，使用dubbo的有过生产运维经验的或多或少是见过的，这个日志简直就是日志打印的典范，其他的日志打印的典范还有spring。得益于这么详细的日志，可以很容易定位到问题所在
- 输出当前线程堆栈详情，这个太有用了，当你通过上面的日志信息还不能定位问题时，案发现场的dump线程上下文信息就是你发现问题的救命稻草。
- 继续抛出拒绝执行异常，使本次任务失败，这个继承了JDK默认拒绝策略的特性



##### Netty中的线程池拒绝策略

```java
private static final class NewThreadRunsPolicy implements RejectedExecutionHandler {
        NewThreadRunsPolicy() {
            super();
        }

        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            try {
                final Thread t = new Thread(r, "Temporary task executor");
                t.start();
            } catch (Throwable e) {
                thrownew RejectedExecutionException(
                        "Failed to start a new thread", e);
            }
        }
    }
```



Netty中的实现很像JDK中的CallerRunsPolicy，舍不得丢弃任务。不同的是，CallerRunsPolicy是直接在调用者线程执行的任务。而 Netty是新建了一个线程来处理的。所以，Netty的实现相较于调用者执行策略的使用面就可以扩展到支持高效率高性能的场景了。但是也要注意一点，Netty的实现里，在创建线程时未做任何的判断约束，也就是说只要系统还有资源就会创建新的线程来处理，直到new不出新的线程了，才会抛创建线程失败的异常。



##### ActiveMq中的线程池拒绝策略

```java
new RejectedExecutionHandler() {
    @Override
    public void rejectedExecution(final Runnable r, final ThreadPoolExecutor executor) {
        try {
            executor.getQueue().offer(r, 60, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            thrownew RejectedExecutionException("Interrupted waiting for BrokerService.worker");
        }

        thrownew RejectedExecutionException("Timed Out while attempting to enqueue Task.");
    }
});
```



ActiveMq中的策略属于最大努力执行任务型，当触发拒绝策略时，在尝试一分钟的时间重新将任务塞进任务队列，当一分钟超时还没成功时，就抛出异常。



##### pinpoint中的线程池拒绝策略

```java
public class RejectedExecutionHandlerChain implements RejectedExecutionHandler {
    private final RejectedExecutionHandler[] handlerChain;

    public static RejectedExecutionHandler build(List<RejectedExecutionHandler> chain) {
        Objects.requireNonNull(chain, "handlerChain must not be null");
        RejectedExecutionHandler[] handlerChain = chain.toArray(new RejectedExecutionHandler[0]);
        returnnew RejectedExecutionHandlerChain(handlerChain);
    }

    private RejectedExecutionHandlerChain(RejectedExecutionHandler[] handlerChain) {
        this.handlerChain = Objects.requireNonNull(handlerChain, "handlerChain must not be null");
    }

    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        for (RejectedExecutionHandler rejectedExecutionHandler : handlerChain) {
            rejectedExecutionHandler.rejectedExecution(r, executor);
        }
    }
}
```

pinpoint的拒绝策略实现很有特点，和其他的实现都不同。他定义了一个拒绝策略链，包装了一个拒绝策略列表，当触发拒绝策略时，会将策略链中的rejectedExecution依次执行一遍。



#### 扩展

##### Future阻塞

把拒绝策略设置为`DiscardPolicy或DiscardOldestPolicy`并且在被拒绝的任务，`Future`对象调用`get()`方法,那么调用线程会一直被阻塞。

因为`FutureTask`的状态大于`COMPLETING`才会返回，要不然都会一直**阻塞等待**。又因为拒绝策略啥没做，没有修改`FutureTask`的状态，因此`FutureTask`的状态一直是`NEW`，所以它不会返回，会一直等待。

这个问题，可以使用别的拒绝策略，比如`CallerRunsPolicy`，它让主线程去执行拒绝的任务，会更新`FutureTask`状态。如果确实想用`DiscardPolicy`，则需要重写`DiscardPolicy`的拒绝策略。

日常开发中，使用` Future.get()` 时，尽量使用带**超时时间的**，因为它是阻塞的。



### 生产实践

通过上面分析提到，通过Executors这个工具类来创建的线程池其实都无法满足实际的使用场景，那么在实际的项目中，到底该如何构造线程池呢，该如何合理的设置参数？



1）线程数

线程数的设置主要取决于业务是IO密集型还是CPU密集型。

CPU密集型指的是任务主要使用来进行大量的计算，没有什么导致线程阻塞。一般这种场景的线程数设置为CPU核心数+1。

IO密集型：当执行任务需要大量的io，比如磁盘io，网络io，可能会存在大量的阻塞，所以在IO密集型任务中使用多线程可以大大地加速任务的处理。一般线程数设置为 2*CPU核心数

java中用来获取CPU核心数的方法是：Runtime.getRuntime().availableProcessors();




2）线程工厂

一般建议自定义线程工厂，构建线程的时候设置线程的名称，这样就在查日志的时候就方便知道是哪个线程执行的代码。



3）有界队列

一般需要设置有界队列的大小，比如LinkedBlockingQueue在构造的时候就可以传入参数，来限制队列中任务数据的大小，这样就不会因为无限往队列中扔任务导致系统的oom。



#### 线程池隔离

为了避免所有的业务逻辑共享一个线程池而导致次要业务影响到主要业务，因此要做线程池隔离。



#### 线程自定义命名

使用线程池时，如果没有给线程池一个有意义的名称，将不好排查回溯问题。

```java
public class ThreadTest {
    public static void main(String[] args) throws Exception {
        ThreadPoolExecutor executorOne = new ThreadPoolExecutor(5, 5, 1,
                TimeUnit.MINUTES, new ArrayBlockingQueue<Runnable>(20),new CustomizableThreadFactory("Tianluo-Thread-pool"));
        executorOne.execute(()->{
            System.out.println("关注公众号：捡田螺的小男孩");
            throw new NullPointerException();
        });
    }
}
```



#### 线程池参数设置

最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）* CPU数目



服务器CPU核数为8核，一个任务线程cpu耗时为20ms，线程等待（网络IO、磁盘IO）耗时80ms，那最佳线程数目：( 80 + 20 )/20 * 8 = 40。也就是设置 40个线程数最佳。



#### 异常处理

使用`submit`提交任务，不会把异常直接这样抛出来。可以改为`execute`方法执行，最好是`try...catch`捕获。还可以为工作者线程设置`UncaughtExceptionHandler`，在`uncaughtException`方法中处理异常。



#### 单例线程池对象

**线程池最好设计成单例模式，给它一个好的命名，以方便排查问题。**



#### ThreadLocal数据错乱

程序运行在 `Tomcat` 中，执行程序的线程是`Tomcat`的工作线程，而`Tomcat`的工作线程是基于**线程池**的。

**线程池会重用固定的几个线程**，一旦线程重用，那么很可能首次从 ThreadLocal 获取的值是之前其他线程的请求遗留的值。这时，ThreadLocal 中的用户信息就是其他线程的信息。

把tomcat的工作线程设置为1时可以复现出这个问题。

```properties
server.tomcat.max-threads=1
```



因此使用ThreadLocal存取数据结束后，需要调用remove方法显示的清空数据。



### 源码解析

#### Executor

该接口表示线程池，其execute方法用来执行Runnable类型的任务。

> java.util.concurrent.Executor

```java
public interface Executor {
    /**
     * 定义了一个用于执行Runnable的execute方法
     */
    void execute(Runnable command);
}
```



#### ExecutorService

> java.util.concurrent.ExecutorService

```java
public interface ExecutorService extends Executor {
    /**
     * 在完成已提交的任务后封闭办事，不再接管新任务,
     */
    void shutdown();

    /**
     * 停止所有正在履行的任务并封闭办事。
     */
    List<Runnable> shutdownNow();

    /**
     * 测试是否该ExecutorService已被关闭。
     */
    boolean isShutdown();

    /**
     * 测试是否所有任务都履行完毕了。
     */
    boolean isTerminated();

    /**
     * 
     */
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    /**
     * 提交Callable类型任务
     */
    <T> Future<T> submit(Callable<T> task);

    /**
     * 提交Runnable类型任务，预先知道返回值
     */
    <T> Future<T> submit(Runnable task, T result);

    /**
     * 提交Runnable类型任务，对返回值无感知
     */
    Future<?> submit(Runnable task);

    /**
     * 永久阻塞 - 提交和执行一个任务列表的所有任务
     */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    /**
     * 带超时阻塞 - 提交和执行一个任务列表的所有任务
     */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    /**
     * 永久阻塞 - 提交和执行一个任务列表的某一个任务
     */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    /**
     * 带超时阻塞 - 提交和执行一个任务列表的某一个任务
     */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```



该接口声明了管理线程池的一些方法。



#### Executors

> java.util.concurrent.Executors

```java
public class Executors {
}
```



该类包含一些静态方法，负责生成各种类型的线程池ExecutorService实例。



#### ThreadPoolExecutor

##### prestartAllCoreThreads

提前创建并启动所有核心线程。

```java
public int prestartAllCoreThreads() {
    int n = 0;
    while (addWorker(null, true))
        ++n;
    return n;
}
```



#### Worker

ThreadPoolExecutor的内部类Worker继承了AQS，使用AQS来实现独占锁的功能。为什么不使用ReentrantLock来实现呢？

可以看到tryAcquire方法，它是不允许重入的，而ReentrantLock是允许重入的：

1）lock方法一旦获取了独占锁，表示当前线程正在执行任务中；
　　2）如果正在执行任务，则不应该中断线程；
　　3）如果该线程现在不是独占锁的状态，也就是空闲的状态，说明它没有在处理任务，这时可以对该线程进行中断；
　　4）线程池在执行shutdown方法或tryTerminate方法时会调用interruptIdleWorkers方法来中断空闲的线程，interruptIdleWorkers方法会使用tryLock方法来判断线程池中的线程是否是空闲状态；
　　5）之所以设置为不可重入，是因为我们不希望任务在调用像setCorePoolSize这样的线程池控制方法时重新获取锁。如果使用ReentrantLock，它是可重入的，这样如果在任务中调用了如setCorePoolSize这类线程池控制的方法，会中断正在运行的线程。

所以，Worker继承自AQS（AbstractQueuedSynchronizer类），用于判断线程是否空闲以及是否可以被中断。

此外，在构造方法中执行了setState(-1);，把state变量设置为-1，为什么这么做呢？是因为AQS中默认的state是0，如果刚创建了一个Worker对象，还没有执行任务时，这时就不应该被中断。tryAcquire方法是根据state是否是0来判断的，所以，setState(-1);将state设置为-1是为了禁止在执行任务前对线程进行中断。正因为如此，在runWorker方法中会先调用Worker对象的unlock方法将state设置为0.



## CAS

CAS（compare and swap）指比较交换。是一种基于锁的操作，而且是乐观锁。

在 java 中锁分为乐观锁和悲观锁。悲观锁是将资源锁住，等一个之前获得锁的线程释放锁之后，下一个线程才可以访问。而乐观锁采取了一种宽泛的态度，通过某种方式不加锁来处理资源，比如通过给记录加 version 来获取数据，性能较悲观锁有很大的提高。

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存地址里面的值和 A 的值是一样的，那么就将内存里面的值更新成 B。CAS是通过无限循环来获取数据的，若果在第一轮循环中，a 线程获取地址里面的值被b 线程修改了，那么 a 线程需要自旋，到下次循环才有可能机会执行。

java.util.concurrent.atomic 包下的类大多是使用 CAS 操作来实现的(AtomicInteger,AtomicBoolean,AtomicLong)。



### ABA问题

比如说一个线程 one 从内存位置 V 中取出 A，这时候另一个线程 two 也从内存中取出 A，并且 two 进行了一些操作变成了 B，然后 two 又将 V 位置的数据变成 A，这时候线程 one 进行 CAS 操作发现内存中仍然是 A，然后 one 操作成功。尽管线程 one 的 CAS 操作成功，但可能存在潜藏的问题。从 Java1.5 开始 JDK 的 atomic包里提供了一个类 AtomicStampedReference 来解决 ABA 问题。

并发环境下，假设初始条件是A，去修改数据时，发现是A就会执行修改。但是看到的虽然是A，中间可能发生了A变B，B又变回A的情况。此时A已经非彼A，数据即使成功修改，也可能有问题。

可以通过`AtomicStampedReference` **解决ABA问题**，它，一个带有标记的原子引用类，通过控制变量值的版本来保证CAS的正确性。



### 循环时间长开销大

对于资源竞争严重（线程冲突严重）的情况，CAS 自旋的概率会比较大，从而浪费更多的 CPU 资源，效率低于 synchronized。

自旋CAS，如果一直循环执行，一直不成功，会给CPU带来非常大的执行开销。很多时候，CAS思想体现，是有个自旋次数的，就是为了避开这个耗时问题~



### 只能保证一个共享变量的原子操作

当对一个共享变量执行操作时，我们可以使用循环 CAS 的方式来保证原子操作，但是对多个共享变量操作时，循环 CAS 就无法保证操作的原子性，这个时候就可以用锁。

CAS 保证的是对一个变量执行操作的原子性，如果对多个变量操作时，CAS 目前无法直接保证操作的原子性的。可以通过这两个方式解决这个问题：1. 使用互斥锁来保证原子性； 2.将多个变量封装成对象，通过AtomicReference来保证原子性。



## AQS

AQS（AbstractQueuedSynchronizer）是一个用来构建锁和同步器的框架，使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如 `ReentrantLock`，`Semaphore`，其他的诸如 `ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask` 等等皆是基于 AQS 的。



### 实现原理

**AQS 核心思想是如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**



> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。



### 资源共享方式

AQS定义两种资源共享方式



- Exclusive（独占）：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：
	- 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
	- 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- **Share**（共享）：多个线程可同时执行，如Semaphore/CountDownLatch。Semaphore、CountDownLatch、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。



ReentrantReadWriteLock 可以看成是组合式，因为ReentrantReadWriteLock也就是读写锁允许多个线程同时对某一资源进行读。

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。



### AbstractQueuedSynchronizer

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如`ReentrantReadWriteLock`。



#### state 状态的维护

- state，int变量，锁的状态，用volatile修饰，保证多线程中的可见性。
- getState()和setState()方法采用final修饰，限制AQS的子类重写它们两。
- compareAndSetState（）方法采用乐观锁思想的CAS算法操作确保线程安全,保证状态 设置的原子性。



#### ConditionObject通知

synchronized控制同步的时候，可以配合Object的wait()、notify()，notifyAll() 系列方法可以实现等待/通知模式。而Lock呢？它提供了条件Condition接口，配合await(),signal(),signalAll() 等方法也可以实现等待/通知机制。ConditionObject实现了Condition接口，给AQS提供条件变量的支持

![图片](../../Image/2022/09/220922-14.jpg)



ConditionObject队列与CLH队列的关系：

- 调用了await()方法的线程，会被加入到conditionObject等待队列中，并且唤醒CLH队列中head节点的下一个节点。
- 线程在某个ConditionObject对象上调用了singnal()方法后，等待队列中的firstWaiter会被加入到AQS的CLH队列中，等待被唤醒。
- 当线程调用unLock()方法释放锁时，CLH队列中的head节点的下一个节点(在本例中是firtWaiter)，会被唤醒。



#### 模板方法设计模式

AQS的典型设计模式就是模板方法设计模式啦。AQS全家桶（ReentrantLock，Semaphore）的衍生实现，就体现出这个设计模式。如AQS提供tryAcquire，tryAcquireShared等模板方法，给子类实现自定义的同步器。



#### 独占与共享模式

- 独占式: 同一时刻仅有一个线程持有同步状态，如ReentrantLock。又可分为公平锁和非公平锁。
- 共享模式:多个线程可同时执行，如Semaphore/CountDownLatch等都是共享式的产物。



#### 自定义同步器

要实现自定义锁的话，首先需要确定要实现的是独占锁还是共享锁，定义原子变量state的含义，再定义一个内部类去继承AQS，重写对应的模板方法即可啦



#### 源码解析

AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

AQS，即`AbstractQueuedSynchronizer`，是构建锁或者其他同步组件的基础框架，它使用了一个`int`成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作。



> java.util.concurrent.locks.AbstractQueuedSynchronizer

```java
    /**
     * 共享变量，使用volatile修饰保证线程可见性
     */
    private volatile int state;

    /**
     * 返回同步状态的当前值
     */
    protected final int getState() {
        return state;
    }

    /**
     * 设置同步状态的值
     */
    protected final void setState(int newState) {
        state = newState;
    }

    /**
     * 原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
     */
    protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```



**AQS底层使用了模板方法模式**

同步器的设计是基于模板方法模式的，如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

1. 使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）
2. 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

这和我们以往通过实现接口的方式有很大区别，这是模板方法模式很经典的一个运用。

**AQS使用了模板方法模式，自定义同步器时需要重写下面几个AQS提供的模板方法：**

> java.util.concurrent.locks.AbstractQueuedSynchronizer

```java
    /**
     * 该线程是否正在独占资源。只有用到condition才需要去实现它。
     */
    protected boolean isHeldExclusively() {
        throw new UnsupportedOperationException();
    }

    /**
     * 独占方式。尝试获取资源，成功则返回true，失败则返回false。
     */
    protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }

    /**
     * 独占方式。尝试释放资源，成功则返回true，失败则返回false。
     */
    protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }

    /**
     * 共享方式。尝试获取资源。
     * 负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
     */
    protected int tryAcquireShared(int arg) {
        throw new UnsupportedOperationException();
    }

    /**
     * 共享方式。尝试释放资源，成功则返回true，失败则返回false。
     */
    protected boolean tryReleaseShared(int arg) {
        throw new UnsupportedOperationException();
    }
```



##### ReentrantLock

以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。



```java
    /**
     * Base of synchronization control for this lock. Subclassed
     * into fair and nonfair versions below. Uses AQS state to
     * represent the number of holds on the lock.
     */
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        /**
         * Performs {@link Lock#lock}. The main reason for subclassing
         * is to allow fast path for nonfair version.
         */
        abstract void lock();

        /**
         * Performs non-fair tryLock.  tryAcquire is implemented in
         * subclasses, but both need nonfair try for trylock method.
         */
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

        protected final boolean isHeldExclusively() {
            // While we must in general read state before owner,
            // we don't need to do so to check if current thread is owner
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        final ConditionObject newCondition() {
            return new ConditionObject();
        }

        // Methods relayed from outer class

        final Thread getOwner() {
            return getState() == 0 ? null : getExclusiveOwnerThread();
        }

        final int getHoldCount() {
            return isHeldExclusively() ? getState() : 0;
        }

        final boolean isLocked() {
            return getState() != 0;
        }

        /**
         * Reconstitutes the instance from a stream (that is, deserializes it).
         */
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }
```



##### CountDownLatch

以CountDownLatch以例，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS(Compare and Swap)减1。等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。



// TODO

[AQS 原理了解么？](https://snailclimb.gitee.io/javaguide-interview/#/./docs/b-3Java%E5%A4%9A%E7%BA%BF%E7%A8%8B?id=_2325-aqs-%e5%8e%9f%e7%90%86%e4%ba%86%e8%a7%a3%e4%b9%88%ef%bc%9f)

[Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)

[Java并发包基石-AQS详解](https://www.cnblogs.com/chengxiao/p/7141160.html)



## 原子操作类

原子操作是指一个不受其他操作影响的操作任务单元。原子操作是在多线程环境下避免数据不一致必须的手段。int++并不是一个原子操作，所以当一个线程读取它的值并加1时，另外一个线程有可能会读到之前的值，这就会引发错误。

原子类就是具有原子/原子操作特征的类，并发包 `java.util.concurrent` 的原子类都存放在`java.util.concurrent.atomic`下。



### 原子操作

原子操作（atomic operation）意为”不可被中断的一个或一系列操作” 。

处理器使用基于对缓存加锁或总线加锁的方式来实现多处理器之间的原子操作。在 Java 中可以通过锁和循环 CAS 的方式来实现原子操作。 CAS 操作——Compare & Set，或是 Compare & Swap，现在几乎所有的 CPU 指令都支持 CAS 的原子操作。

原子操作是指一个不受其他操作影响的操作任务单元。原子操作是在多线程环境下避免数据不一致必须的手段。

int++并不是一个原子操作，所以当一个线程读取它的值并加 1 时，另外一个线程有可能会读到之前的值，这就会引发错误。

为了解决这个问题，必须保证增加操作是原子的，在 JDK1.5 之前我们可以使用同步技术来做到这一点。到 JDK1.5，java.util.concurrent.atomic 包提供了 int 和long 类型的原子包装类，它们可以自动的保证对于他们的操作是原子的并且不需要使用同步。



### 实现原理

Atomic包中的类基本的特性就是在多线程环境下，当有多个线程同时对单个（包括基本类型及引用类型）变量进行操作时，具有排他性，即当多个线程同时对该变量的值进行更新时，仅有一个线程能成功，而未成功的线程可以向自旋锁一样，继续尝试，一直等到执行成功。



> java.util.concurrent.atomic.AtomicInteger

```java
// setup to use Unsafe.compareAndSwapInt for updates（更新操作时提供“比较并替换”的作用）
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
	try {
		valueOffset = unsafe.objectFieldOffset
		(AtomicInteger.class.getDeclaredField("value"));
	} catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;
```



AtomicInteger 类主要利用 CAS (compare and swap) + volatile 和 native 方法来保证原子操作，从而避免 synchronized 的高开销，执行效率大为提升。

CAS的原理是拿期望的值和原本的一个值作比较，如果相同则更新成新的值。UnSafe 类的 objectFieldOffset() 方法是一个本地方法，这个方法是用来拿到“原来的值”的内存地址，返回值是 valueOffset。另外 value 是一个volatile变量，在内存中可见，因此 JVM 可以保证任何时刻任何线程总能拿到该变量的最新值。



### 原子类分类

java.util.concurrent 这个包里面提供了一组原子类。其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由 JVM 从等待队列中选择另一个线程进入，这只是一种逻辑上的理解。

原子类：AtomicBoolean，AtomicInteger，AtomicLong，AtomicReference

原子数组：AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray

原子属性更新器：AtomicLongFieldUpdater，AtomicIntegerFieldUpdater，AtomicReferenceFieldUpdater

解决 ABA 问题的原子类：AtomicMarkableReference（通过引入一个 boolean来反映中间有没有变过），AtomicStampedReference（通过引入一个 int 来累加来反映中间有没有变过）



**基本类型**

使用原子的方式更新基本类型

- `AtomicInteger`：整形原子类
- `AtomicLong`：长整型原子类
- `AtomicBoolean`：布尔型原子类



**数组类型**

使用原子的方式更新数组里的某个元素

- `AtomicIntegerArray`：整形数组原子类
- `AtomicLongArray`：长整形数组原子类
- `AtomicReferenceArray`：引用类型数组原子类



**引用类型**

- `AtomicReference`：引用类型原子类
- `AtomicStampedReference`：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。
- `AtomicMarkableReference` ：原子更新带有标记位的引用类型



**对象的属性修改类型**

- `AtomicIntegerFieldUpdater`：原子更新整形字段的更新器
- `AtomicLongFieldUpdater`：原子更新长整形字段的更新器
- `AtomicReferenceFieldUpdater`：原子更新引用类型字段的更新器



### AtomicInteger

AtomicInteger的底层，是基于CAS实现的。我们可以看下AtomicInteger的添加方法。如下

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
// 通过Unsafe类的实例来进行添加操作
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
        //使用了CAS算法实现
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```



`compareAndSwapInt`是一个native方法哈，它是基于CAS来操作int类型的变量。并且，其它的原子操作类基本也大同小异。



## 锁相关类

### Lock

Lock 接口比同步方法和同步块提供了更具扩展性的锁操作。他们允许更灵活的结构，可以具有完全不同的性质，并且可以支持多个相关类的条件对象。

它的优势有：

（1）可以使锁更公平

（2）可以使线程在等待锁的时候响应中断

（3）可以让线程尝试获取锁，并在无法获取锁的时候立即返回或者等待一段时间

（4）可以在不同的范围，以不同的顺序获取和释放锁

整体上来说 Lock 是 synchronized 的扩展版，Lock 提供了无条件的、可轮询的(tryLock 方法)、定时的(tryLock 带参方法)、可中断的(lockInterruptibly)、可多条件队列的(newCondition 方法)锁操作。另外 Lock 的实现类基本都支持非公平锁(默认)和公平锁，synchronized 只支持非公平锁，当然，在大部分情况下，非公平锁是高效的选择。



#### 源码解析

> java.util.concurrent.locks.Lock

```java
public interface Lock {

    /**
     * 
     */
    void lock();

    /**
     * 
     */
    void lockInterruptibly() throws InterruptedException;

    /**
     * 
     */
    boolean tryLock();

    /**
     * 
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    /**
     * 
     */
    void unlock();

    /**
     * 
     */
    Condition newCondition();
}
```



### ReentrantLock

ReentrantLock重入锁，是实现Lock接口的一个类，也是在实际编程中使用频率很高的一个锁，支持重入性，表示能够对共享资源能够重复加锁，即当前线程获取该锁再次获取不会被阻塞。

在java关键字synchronized隐式支持重入性，synchronized通过获取自增，释放自减的方式实现重入。与此同时，ReentrantLock还支持公平锁和非公平锁两种方式。那么，要想完完全全的弄懂ReentrantLock的话，主要也就是ReentrantLock同步语义的学习：1. 重入性的实现原理；2. 公平锁和非公平锁。



#### 重入性的实现原理

要想支持重入性，就要解决两个问题：**1. 在线程获取锁的时候，如果已经获取锁的线程是当前线程的话则直接再次获取成功；2. 由于锁会被获取n次，那么只有锁在被释放同样的n次之后，该锁才算是完全释放成功**。

ReentrantLock支持两种锁：**公平锁**和**非公平锁**。**何谓公平性，是针对获取锁而言的，如果一个锁是公平的，那么锁的获取顺序就应该符合请求上的绝对时间顺序，满足FIFO**。

![图片](../../Image/2022/09/220923-1.jpg)



#### synchronized和ReentrantLock的区别

synchronized不能扩展锁之外的方法或者块边界，尝试获取锁时不能中途取消；ReentrantLock拥有与 synchronized 相同的并发性和内存语义且它还具有可扩展性。

- `Synchronized`是依赖于`JVM`实现的，而`ReenTrantLock`是`API`实现的。
- 在`Synchronized`优化以前，`synchronized`的性能是比`ReenTrantLock`差很多的，但是自从`Synchronized`引入了偏向锁，轻量级锁（自旋锁）后，两者性能就差不多了。
- `Synchronized`的使用比较方便简洁，它由编译器去保证锁的加锁和释放。而`ReenTrantLock`需要手工声明来加锁和释放锁，最好在finally中声明释放锁。
- `ReentrantLock`可以指定是公平锁还是⾮公平锁。⽽`synchronized`只能是⾮公平锁。
- `ReentrantLock`可响应中断、可轮回，而`Synchronized`是不可以响应中断的



#### 源码解析

##### lock

如果获取了锁立即返回，如果别的线程持有锁，当前线程则一直处于阻塞状态，直到该线程获取锁。



##### tryLock

如果获取了锁立即返回true，如果别的线程正持有锁，立即返回false。



##### tryLock(long timeout, TimeUnit unit)

如果获取了锁定立即返回true，如果别的线程正持有锁，会等待参数给定的时间，在等待的过程中，如果获取了锁定，就返回true，如果等待超时，返回false；



##### lockInterruptibly

如果获取了锁定立即返回；如果没有获取锁，线程处于阻塞状态，直到获取锁或者线程被别的线程中断。



### ReadWriteLock 

读写锁是用来提升并发程序性能的锁分离技术。Java中的ReadWriteLock是Java 5 中新增的一个接口，一个ReadWriteLock维护一对关联的锁，一个用于只读操作一个用于写。在没有写线程的情况下一个读锁可能会同时被多个读线程持有。写锁是独占的，可以使用JDK中的ReentrantReadWriteLock来实现这个规则，它最多支持65535个写锁和65535个读锁。



#### ReentrantReadWriteLock

首先明确一下，不是说 ReentrantLock 不好，只是 ReentrantLock 某些时候有局限。如果使用 ReentrantLock，可能本身是为了防止线程 A 在写数据、线程 B 在读数据造成的数据不一致，但这样，如果线程 C 在读数据、线程 D 也在读数据，读数据是不会改变数据的，没有必要加锁，但是还是加锁了，降低了程序的性能。因为这个，才诞生了读写锁 ReadWriteLock。

ReadWriteLock 是一个读写锁接口，读写锁是用来提升并发程序性能的锁分离技术，ReentrantReadWriteLock 是 ReadWriteLock 接口的一个具体实现，实现了读写的分离，读锁是共享的，写锁是独占的，读和读之间不会互斥，读和写、写和读、写和写之间才会互斥，提升了读写的性能。

而读写锁有以下三个重要的特性：

（1）公平选择性：支持非公平（默认）和公平的锁获取方式，吞吐量还是非公平优于公平。

（2）重进入：读锁和写锁都支持线程重进入。

（3）锁降级：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级成为读锁。



### LockSupport

LockSupport是一个工具类。它的主要作用是**挂起和唤醒线程**。该工具类是创建锁和其他同步类的基础。

```java
public static void park(Object blocker); // 暂停指定线程
public static void unpark(Thread thread); // 恢复指定的线程
public static void park(); // 无期限暂停当前线程
```



```java
public class LockSupportTest {

    private static Object object = new Object();
    static MyThread thread = new MyThread("线程田螺");

    public static class MyThread extends Thread {

        public MyThread(String name) {
            super(name);
        }

        @Override public void run() {
            synchronized (object) {
                System.out.println("线程名字： " + Thread.currentThread());
                try {
                    Thread.sleep(2000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                LockSupport.park();
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("线程被中断了");
                }
                System.out.println("继续执行");
            }
        }
    }

    public static void main(String[] args) {
        thread.start();
        LockSupport.unpark(thread);
        System.out.println("恢复线程调用");
    }
}
```

因为`thread`线程内部有休眠2秒的操作，所以`unpark`方法的操作肯定先于`park`方法的调用。为什么`thread`线程最终仍然可以结束，是因为`park`和`unpark`会对每个线程维持一个许可证（布尔值）



## 同步工具类

### Semaphore

**允许多个线程同时访问：** synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore(信号量)可以指定多个线程同时访问某个资源。

Semaphore 就是一个信号量，它的作用是限制某段代码块的并发数。Semaphore有一个构造函数，可以传入一个 int 型整数 n，表示某段代码最多只有 n 个线程可以访问，如果超出了 n，那么请等待，等到某个线程执行完毕这段代码块，下一个线程再进入。由此可以看出如果 Semaphore 构造函数中传入的 int 型整数 n=1，相当于变成了一个 synchronized 了。

**Semaphore**，我们也把它叫做**信号量**。可以用来控制同时访问**特定资源的线程数量**，通过协调各个线程，以保证合理的使用资源。



#### 实现原理

##### Semaphore构造函数

Semaphore构造函数会创建一个非公平的锁的同步阻塞队列，并且把初始令牌数量（20）赋值给同步队列的state，这个state就是`AQS`的哈。

```java
Semaphore semaphore=new Semaphore(20);

//构造函数，创建一个非公平的锁的同步阻塞队列
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

NonfairSync(int permits) {
    super(permits);
}

//把令牌数量赋值给同步队列的state
Sync(int permits) {
    setState(permits);
}
```



##### 可用令牌数

这个`availablePermits`，获取的就是`state`值。刚开始为20，所以肯定不会为0嘛。

```java
semaphore.availablePermits();

public int availablePermits() {
  return sync.getPermits();
}

final int getPermits() {
  return getState();
}
```



##### 获取令牌

```java
semaphore.acquire();

public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}

public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    //尝试获取令牌,arg为获取令牌个数
    if (tryAcquireShared(arg) < 0)
        //
        doAcquireSharedInterruptibly(arg);
}
```



尝试获取令牌,使用了CAS算法。

```java
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```



可获取令牌的话，就创建节点，加入阻塞队列；重双向链表的head，tail节点关系，清空无效节点;挂起当前节点线程

```java
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    //创建节点加入阻塞队列
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                //返回锁的state
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            //重组双向链表，清空无效节点，挂起当前线程
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```



##### 释放令牌

```java
semaphore.release();

/**
     * 释放令牌
     */
public void release() {
    sync.releaseShared(1);
}

public final boolean releaseShared(int arg) {
    //释放共享锁
    if (tryReleaseShared(arg)) {
        //唤醒所有共享节点线程
        doReleaseShared();
        return true;
    }
    return false;
}
```



#### 生产实践

```java
public class SemaphoreTest {
    private  static Semaphore semaphore=new Semaphore(20);
    public static void main(String[] args) { 
        ExecutorService executorService= Executors.newFixedThreadPool(200);
        //模拟100辆车要来
        for (int i = 0; i < 100; i++) {
            executorService.execute(()->{
                System.out.println("===="+Thread.currentThread().getName()+"准备进入停车场==");
                //车位判断
                if (semaphore.availablePermits() == 0) {
                    System.out.println("车辆不足，请耐心等待");
                }

                try {
                    //获取令牌尝试进入停车场
                    semaphore.acquire();
                    System.out.println("====" + Thread.currentThread().getName() + "成功进入停车场");
                    //模拟车辆在停车场停留的时间
                    Thread.sleep(new Random().nextInt(20000));
                    System.out.println("====" + Thread.currentThread().getName() + "驶出停车场");
                    //释放令牌，腾出停车场车位
                    semaphore.release();
                 } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            });
            //线程池关闭          
            executorService.shutdown();
        }
    }
}
```



### CountDownLatch

CountDownLatch是一个同步工具类，用来协调多个线程之间的同步。这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。



// TODO 

[CountDownLatch](https://snailclimb.gitee.io/javaguide-interview/#/./docs/b-3Java%E5%A4%9A%E7%BA%BF%E7%A8%8B?id=_2327-%e7%94%a8%e8%bf%87-countdownlatch-%e4%b9%88%ef%bc%9f%e4%bb%80%e4%b9%88%e5%9c%ba%e6%99%af%e4%b8%8b%e7%94%a8%e7%9a%84%ef%bc%9f)



### CyclicBarrier 

CyclicBarrier 和 CountDownLatch 非常类似，它也可以实现线程间的技术等待，但是它的功能比 CountDownLatch 更加复杂和强大。主要应用场景和 CountDownLatch 类似。CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。CyclicBarrier默认的构造方法是 CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await()方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞。



#### CycliBarriar 和 CountdownLatch 有什么区别

CountDownLatch与CyclicBarrier都是用于控制并发的工具类，都可以理解成维护的就是一个计数器，但是这两者还是各有不同侧重点的：

- CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；CountDownLatch强调一个线程等多个线程完成某件事情。CyclicBarrier是多个线程互等，等大家都完成，再携手共进。
- 调用CountDownLatch的countDown方法后，当前线程并不会阻塞，会继续往下执行；而调用CyclicBarrier的await方法，会阻塞当前线程，直到CyclicBarrier指定的线程全部都到达了指定点的时候，才能继续往下执行；
- CountDownLatch方法比较少，操作比较简单，而CyclicBarrier提供的方法更多，比如能够通过getNumberWaiting()，isBroken()这些方法获取当前多个线程的状态，并且CyclicBarrier的构造方法可以传入barrierAction，指定当所有线程都到达时执行的业务功能；
- CountDownLatch是不能复用的，而CyclicLatch是可以复用的。



- CountDownLatch：一个或者多个线程，等待其他多个线程完成某件事情之后才能执行;
- CyclicBarrier：多个线程互相等待，直到到达同一个同步点，再继续一起执行。



![图片](../../Image/2022/09/220922-7.jpg)



### Exchanger

Exchanger是一个用于线程间协作的工具类，用于两个线程间交换数据。它提供了一个交换的同步点，在这个同步点两个线程能够交换数据。交换数据是通过exchange方法来实现的，如果一个线程先执行exchange方法，那么它会同步等待另一个线程也执行exchange方法，这个时候两个线程就都达到了同步点，两个线程就可以交换数据。



## 并发任务类

### Callable

Java 5在concurrency包中引入了java.util.concurrent.Callable 接口，它和Runnable接口很相似，但它可以返回一个对象或者抛出一个异常。

Callable接口使用泛型去定义它的返回类型。Executors类提供了一些有用的方法去在线程池中执行Callable内的任务。由于Callable任务是并行的，我们必须等待它返回的结果。java.util.concurrent.Future对象为我们解决了这个问题。在线程池提交Callable任务后返回了一个Future对象，使用它我们可以知道Callable任务的状态和得到Callable返回的执行结果。Future提供了get()方法让我们可以等待Callable结束并获取它的执行结果。



#### 和Runnable互相转换

工具类 `Executors` 可以实现 `Runnable` 对象和 `Callable` 对象之间的相互转换。



> java.util.concurrent.Executors

```java
/**
     * 
     */
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}

/**
     * 
     */
public static Callable<Object> callable(Runnable task) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<Object>(task, null);
}
```



#### 源码解析

> java.util.concurrent.Callable

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * 计算结果，如果不能这样做，则抛出异常。
     */
    V call() throws Exception;
}
```



### Future

Future实现对任务的取消、数据获取、任务状态判断等功能。



#### 源码解析

> java.util.concurrent.Future

```java
public interface Future<V> {
    /**
     * 取消任务，如果任务正在运行的，
     * mayInterruptIfRunning为true时，表明这个任务会被打断的，并返回true，
     * 为false时，会等待这个任务执行完，返回true；
     * 若任务还没执行，
     * 取消任务后返回true，如任务执行完，返回false
     */
    boolean cancel(boolean mayInterruptIfRunning);

    /**
     * 判断任务是否被取消了,正常执行完不算被取消
     */
    boolean isCancelled();

    /**
     * 判断任务是否已经执行完成，任务取消或发生异常也算是完成，返回true
     */
    boolean isDone();

    /**
     * 获取任务返回结果，如果任务没有执行完成则等待完成将结果返回，如果获取的过程中发生异常就抛出异常，
     * 比如中断就会抛出InterruptedException异常等异常
     */
    V get() throws InterruptedException, ExecutionException;

    /**
     * 在规定的时间如果没有返回结果就会抛出TimeoutException异常
     */
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```



#### 使用示例

```java
public ExecutorService executorService = Executors.newCachedThreadPool();

@Test
public void futureTest(){
    Future<String> future = executorService.submit(new MyCallable());
    try {
        System.err.println("start");
        // get 方法会阻塞等待异步线程执行结果
        System.out.println(future.get());
        System.err.println("end");
    } catch (Exception e) {
        // do nothing
    } finally {
        executorService.shutdown();
    }
}
```



### FutureTask

在Java并发程序中FutureTask表示一个可以取消的异步运算。它有启动和取消运算、查询运算是否完成和取回运算结果等方法。只有当运算完成的时候结果才能取回，如果运算尚未完成get方法将会阻塞。一个FutureTask对象可以对调用了Callable和Runnable的对象进行包装，由于FutureTask也是调用了Runnable接口所以它可以提交给Executor来执行。



FutureTask包装器是一种非常便利的机制，可将Callable转换成Future和Runnable，它同时实现两者的接口。

FutureTask类是Future 的一个实现，并实现了Runnable，所以可通过Excutor(线程池) 来执行。也可传递给Thread对象执行。如果在主线程中需要执行比较耗时的操作时，但又不想阻塞主线程时，可以把这些作业交给Future对象在后台完成，当主线程将来需要时，就可以通过Future对象获得后台作业的计算结果或者执行状态。



FutureTask是一种可以取消的异步的计算任务。它的计算是通过`Callable`实现的，可以把它理解为是可以返回结果的`Runnable`。

使用FutureTask的优点：

- 可以获取线程执行后的返回结果；
- 提供了超时控制功能。

它实现了`Runnable`接口和`Future`接口，底层基于生产者消费者模式实现。



FutureTask用于在异步操作场景中，FutureTask作为生产者(执行FutureTask的线程)和消费者(获取FutureTask结果的线程)的桥梁，如果生产者先生产出了数据，那么消费者get时能会直接拿到结果；如果生产者还未产生数据，那么get时会一直阻塞或者超时阻塞，一直到生产者产生数据唤醒阻塞的消费者为止。



#### 源码解析

##### 构造器

> java.util.concurrent.FutureTask

```java
    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }

    /**
     * 
     */
    public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
    }
```



##### 与Callable的关系

> java.util.concurrent.FutureTask

```java
public void run() {
    // ...
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                // ...
            }
            // ...
        }
    } finally {
        // ...
    }
}
```



FutureTask实现了Runnable接口，重写了run方法，在run方法中实际调用了Callable实现类对象的call方法。因此Thread的run方法实际调用的是Callable的call方法。



#### 使用示例

```java
public ExecutorService executorService = Executors.newCachedThreadPool();

@Test
public void futureTaskTest(){
    FutureTask<String> futureTask = new FutureTask<>(new MyCallable());
    executorService.submit(futureTask);
    try {
        System.err.println("start");
        System.out.println("task运行结果为：" + futureTask.get());
        System.err.println("end");
    } catch (Exception e) {
        // do nothing
    } finally {
        executorService.shutdown();
    }
}
```




### CompletableFuture

#### 创建
#### runAsync 方法

#### supplyAsync 方法


#### 获取结果



#### 使用示例

```java
@Test
public void completableFutureTest(){
    CompletableFuture<Void> f1 =
        CompletableFuture.runAsync(() -> System.err.println("first"));
    CompletableFuture<Void> f2 =
        CompletableFuture.runAsync(() -> System.err.println("second"));
    f1.thenCombine(f2, (a, b) -> {
        System.err.println("end");
        return "end";
    });
}
```



## 高并发容器

同步集合与并发集合都为多线程和并发提供了合适的线程安全的集合，不过并发集合的可扩展性更高。在Java1.5之前程序员们只有同步集合来用且在多线程并发的时候会导致争用，阻碍了系统的扩展性。Java5介绍了并发集合像ConcurrentHashMap，不仅提供线程安全还用锁分离和内部分区等现代技术提高了可扩展性。

Java集合类都是快速失败的，这就意味着当集合被改变且一个线程在使用迭代器遍历集合的时候，迭代器的next()方法将抛出ConcurrentModificationException异常。

并发容器是针对多个线程并发访问设计的，在jdk5.0引入了concurrent包，其中提供了很多并发容器，如ConcurrentHashMap，CopyOnWriteArrayList等。并发容器使用了与同步容器完全不同的加锁策略来提供更高的并发性和伸缩性。



### ConcurrentHashMap

> JDK1.7使用分段锁，JDK1.8使用CAS+synchronized来提高并发度。本文默认以JDK1.8为参考。



#### 基础概念

ConcurrentHashMap是Java中的一个**线程安全且高效的HashMap实现**。平时涉及高并发如果要用map结构，那第一时间想到的就是它。相对于hashmap来说，ConcurrentHashMap就是线程安全的map，其中利用了锁分段的思想提高了并发度。



那么它到底是如何实现线程安全的？

JDK 1.6版本关键要素：

- segment继承了ReentrantLock充当锁的角色，为每一个segment提供了线程安全的保障；
- segment维护了哈希散列表的若干个桶，每个桶由HashEntry构成的链表。



JDK1.8后，ConcurrentHashMap抛弃了原有的**Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性**。因为在1.6版本的时候JVM对Synchronized的优化非常大。

在JDK1.8版本，它改成了与HashMap一样的数据结构，数组 + 单链表 或者 红黑树的数据结构，

ConcurrentHashMap 把实际 map 划分成若干部分来实现它的可扩展性和线程安全。这种划分是使用并发度获得的，它是 ConcurrentHashMap 类构造函数的一个可选参数，默认值为 16，这样在多线程情况下就能避免争用。

在 JDK8 后，它摒弃了 Segment（锁段）的概念，而是启用了一种全新的方式实现,利用 CAS 算法。同时加入了更多的辅助变量来提高并发度。



##### 并发度

并发度就是`segment`的个数，通常是2的N次方。

ConcurrentHashMap把实际map划分成若干部分来实现它的可扩展性和线程安全。这种划分是使用并发度获得的，它是ConcurrentHashMap类构造函数的一个可选参数，默认值为16，这样在多线程情况下就能避免争用。



#### 实现原理

JDK8中ConcurrentHashMap参考了JDK8 HashMap的实现，采用了数组+链表+红黑树的实现方式来设计，内部大量采用CAS操作，这里我简要介绍下CAS。
CAS是compare and swap的缩写，即我们所说的比较交换。cas是一种基于锁的操作，而且是乐观锁。

在java中锁分为乐观锁和悲观锁。悲观锁是将资源锁住，等一个之前获得锁的线程释放锁之后，下一个线程才可以访问。

而乐观锁采取了一种宽泛的态度，通过某种方式不加锁来处理资源，比如通过给记录加version来获取数据，性能较悲观锁有很大的提高。
CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存地址里面的值和A的值是一样的，那么就将内存里面的值更新成B。

CAS是通过无限循环来获取数据的，若果在第一轮循环中，a线程获取地址里面的值被b线程修改了，那么a线程需要自旋，到下次循环才有可能机会执行。

JDK8中彻底放弃了Segment转而采用的是Node，其设计思想也不再是JDK1.7中的分段锁思想。
Node：保存key，value及key的hash值的数据结构。其中value和next都用volatile修饰，保证并发的可见性。



```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;
}
```



Java8 ConcurrentHashMap结构基本上和Java8的HashMap一样，不过保证线程安全性。

在JDK8中ConcurrentHashMap的结构，由于引入了红黑树，使得ConcurrentHashMap的实现非常复杂，

我们都知道，红黑树是一种性能非常好的二叉查找树，其查找性能为O（logN），但是其实现过程也非常复杂，而且可读性也非常差，

DougLea的思维能力确实不是一般人能比的，早期完全采用链表结构时Map的查找时间复杂度为O（N），

JDK8中ConcurrentHashMap在链表的长度大于某个阈值的时候会将链表转换成红黑树进一步提高其查找性能。



#### 源码解析

##### 构造方法

> java.util.concurrent.ConcurrentHashMap

```java
    public ConcurrentHashMap() {
    }

    /**
     * Creates a new, empty map with an initial table size
     * accommodating the specified number of elements without the need
     * to dynamically resize.
     *
     * @param initialCapacity The implementation performs internal
     * sizing to accommodate this many elements.
     * @throws IllegalArgumentException if the initial capacity of
     * elements is negative
     */
    public ConcurrentHashMap(int initialCapacity) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException();
        int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
                   MAXIMUM_CAPACITY :
                   tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
        this.sizeCtl = cap;
    }

    /**
     * Creates a new map with the same mappings as the given map.
     *
     * @param m the map
     */
    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        this.sizeCtl = DEFAULT_CAPACITY;
        putAll(m);
    }

    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, 1);
    }

    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }
```



#### 总结

JDK1.8版本的ConcurrentHashMap的数据结构已经接近HashMap，相对而言，ConcurrentHashMap只是增加了同步的操作来控制并发，从JDK1.7版本的ReentrantLock+Segment+HashEntry，到JDK1.8版本中synchronized+CAS+HashEntry+红黑树。

**1.数据结构**：取消了Segment分段锁的数据结构，取而代之的是数组+链表+红黑树的结构。

**2.保证线程安全机制**：JDK1.7采用segment的分段锁机制实现线程安全，其中segment继承自ReentrantLock。

JDK1.8采用CAS+Synchronized保证线程安全。3.锁的粒度：原来是对需要进行数据操作的Segment加锁，现调整为对每个数组元素加锁（Node）。4.链表转化为红黑树:定位结点的hash算法简化会带来弊端,Hash冲突加剧,因此在链表节点数量大于8时，会将链表转化为红黑树进行存储。5.查询时间复杂度：从原来的遍历链表O(n)，变成遍历红黑树O(logN)。



#### 扩展

##### 强一致性

在 Java 8 中，ConcurrentHashMap 使用了 unsafe 类来实现读写操作

- 对于写首先使用 compareAndSwapObject（即我们熟悉的 CAS）来更新**内存中**数组中的元素
- 对于读则使用了 getObjectVolatile 来读取**内存中**数组中的元素（在底层其实是用了 C++ 的 volatile 来实现 java 中的 volatile 效果，有兴趣可以看看）

由于读写都是直接对内存操作的，所以通过这样的方式可以保证 put，get 的强一致性，至此真相大白！Java 8 以后 put，get 是可以保证强一致性的！ConcurrentHashMap 是通过 compareAndSwapObject 来取代对数组元素直接赋值的操作，通过 getObjectVolatile 来补上无法对数组元素进行 volatile 读的坑来实现的

注意并不是说 ConcurrentHashMap 所有的操作都是强一致性的，比如 Java 8 中计算容量的方法 size() 就是弱一致性（Java 7 中此方法反而是强一致性）。



##### JDK1.7实现原理

在JDK1.7中ConcurrentHashMap采用了数组+Segment+分段锁的方式实现。



**1.Segment(分段锁)**
ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表,同时又是一个ReentrantLock（Segment继承了ReentrantLock）。



**2.内部结构**
ConcurrentHashMap使用分段锁技术，将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问，能够实现真正的并发访问。如下图是ConcurrentHashMap的内部结构图：

![在这里插入图片描述](../../Image/2022/08/220801-10.png)

从上面的结构我们可以了解到，ConcurrentHashMap定位一个元素的过程需要进行两次Hash操作。第一次Hash定位到Segment，第二次Hash定位到元素所在的链表的头部。



**3.该结构的优劣势**

- 坏处

这一种结构的带来的副作用是Hash的过程要比普通的HashMap要长。



- 好处

写操作的时候可以只对元素所在的Segment进行加锁即可，不会影响到其他的Segment，这样，在最理想的情况下，ConcurrentHashMap可以最高同时支持Segment数量大小的写操作（刚好这些写操作都非常平均地分布在所有的Segment上）。

所以，通过这一种结构，ConcurrentHashMap的并发能力可以大大的提高。



##### 并发容器和同步容器

何为同步容器：可以简单地理解为通过 synchronized 来实现同步的容器，如果有多个线程调用同步容器的方法，它们将会串行执行。比如 Vector，Hashtable，以及 Collections.synchronizedSet，synchronizedList 等方法返回的容器。可以通过查看 Vector，Hashtable 等这些同步容器的实现代码，可以看到这些容器实现线程安全的方式就是将它们的状态封装起来，并在需要同步的方法上加上关键字 synchronized。

并发容器使用了与同步容器完全不同的加锁策略来提供更高的并发性和伸缩性，例如在 ConcurrentHashMap 中采用了一种粒度更细的加锁机制，可以称为分段锁，在这种锁机制下，允许任意数量的读线程并发地访问 map，并且执行读操作的线程和写操作的线程也可以并发的访问 map，同时允许一定数量的写操作线程并发地修改 map，所以它可以在并发环境下实现更高的吞吐量。



同步集合与并发集合都为多线程和并发提供了合适的线程安全的集合，不过并发集合的可扩展性更高。在 Java1.5 之前程序员们只有同步集合来用且在多线程并发的时候会导致争用，阻碍了系统的扩展性。Java5 介绍了并发集合像ConcurrentHashMap，不仅提供线程安全还用锁分离和内部分区等现代技术提高了可扩展性。



##### SynchronizedMap 和 ConcurrentHashMap 有什么区别？

SynchronizedMap 一次锁住整张表来保证线程安全，所以每次只能有一个线程来访为 map。

ConcurrentHashMap 使用分段锁来保证在多线程下的性能。

ConcurrentHashMap 中则是一次锁住一个桶。ConcurrentHashMap 默认将hash 表分为 16 个桶，诸如 get，put，remove 等常用操作只锁当前需要用到的桶。

这样，原来只能一个线程进入，现在却能同时有 16 个写线程执行，并发性能的提升是显而易见的。

另外 ConcurrentHashMap 使用了一种不同的迭代方式。在这种迭代方式中，当iterator 被创建后集合再发生改变就不再是抛出ConcurrentModificationException，取而代之的是在改变时 new 新的数据从而不影响原有的数据，iterator 完成后再将头指针替换为新的数据 ，这样 iterator线程可以使用原来老的数据，而写线程也可以并发的完成改变。



##### ConcurrentHashMap与HashMap的区别

HashMap是线程不安全的，在多线程环境下，使用Hashmap进行put操作会引起死循环，导致CPU利用率接近100%，所以在并发情况下不能使用HashMap。



HashTable和HashMap的实现原理几乎一样，差别无非是HashTable不允许key和value为null，HashTable是线程安全的。但是HashTable线程安全的策略实现代价却太大了，简单粗暴，get/put所有相关操作都是synchronized的，这相当于给整个哈希表加了一把大锁。多线程访问时候，只要有一个线程访问或操作该对象，那其他线程只能阻塞，相当于将所有的操作串行化，在竞争激烈的并发场景中性能就会非常差。



ConcurrentHashMap为了应对hashmap在并发环境下不安全而诞生的，ConcurrentHashMap的设计与实现非常精巧，大量的利用了volatile，final，CAS等lock-free技术来减少锁竞争对于性能的影响。

ConcurrentHashMap避免了对全局加锁改成了局部加锁操作，这样就极大地提高了并发环境下的操作速度。



### CopyOnWriteArrayList

CopyOnWriteArrayList 是一个并发容器。有很多人称它是线程安全的，我认为这句话不严谨，缺少一个前提条件，那就是非复合场景下操作它是线程安全的。

CopyOnWriteArrayList(免锁容器)的好处之一是当多个迭代器同时遍历和修改这个列表时，不会抛出 ConcurrentModificationException。在CopyOnWriteArrayList 中，写入将导致创建整个底层数组的副本，而源数组将保留在原地，使得复制的数组在被修改时，读取操作可以安全地执行。

CopyOnWriteArrayList 的使用场景

通过源码分析，我们看出它的优缺点比较明显，所以使用场景也就比较明显。就是合适读多写少的场景。

CopyOnWriteArrayList 的缺点

1. 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致 young gc 或者 full gc。
2. 不能用于实时读的场景，像拷贝数组、新增元素都需要时间，所以调用一个 set 操作后，读取到数据可能还是旧的，虽然CopyOnWriteArrayList 能做到最终一致性,但是还是没法满足实时性要求。
3. 由于实际使用中可能没法保证 CopyOnWriteArrayList 到底要放置多少数据，万一数据稍微有点多，每次 add/set 都要重新复制数组，这个代价实在太高昂了。在高性能的互联网应用中，这种操作分分钟引起故障。

CopyOnWriteArrayList 的设计思想

1. 读写分离，读和写分开
2. 最终一致性
3. 使用另外开辟空间的思路，来解决并发冲突



####  CopyOnWriteArrayList内存占用过多

```java
public boolean add(E e) {
    // 获取独占锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 获取array
        Object[] elements = getArray();
        // 复制array到新数组，添加元素到新数组
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        // 替换数组
        setArray(newElements);
        return true;
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```



CopyOnWriteArrayList 内部维护了一个数组，成员变量 array 就指向这个内部数组，所有的读操作都是基于新的array对象进行的。

因为上了独占锁，所以如果多个线程调用add()方法只有一个线程会获得到该锁，其他线程被阻塞，直到锁被释放， 由于加了锁，所以整个操作的过程是原子性操作

CopyOnWriteArrayList 会将 新的array复制一份，然后在新复制处理的数组上执行增加元素的操作，执行完之后再将复制的结果指向这个新的数组。

由于每次写入的时候都会对数组对象进行复制，复制过程不仅会占用双倍内存，还需要消耗 CPU 等资源，所以当列表中的元素比较少的时候，这对内存和 GC 并没有多大影响，但是当列表保存了大量元素的时候，

对 CopyOnWriteArrayList 每一次修改，都会重新创建一个大对象，并且原来的大对象也需要回收，这都可能会触发 GC，如果超过老年代的大小则容易触发Full GC，引起应用程序长时间停顿。



####  CopyOnWriteArrayList是弱一致性的

```java
public Iterator<E> iterator() {
    return new COWIterator<E>(getArray(), 0);
}

static final class COWIterator<E> implements ListIterator<E> {
    /** Snapshot of the array */
    private final Object[] snapshot;
    /** Index of element to be returned by subsequent call to next.  */
    private int cursor;

    private COWIterator(Object[] elements, int initialCursor) {
        cursor = initialCursor;
        snapshot = elements;
    }

    public boolean hasNext() {
        return cursor < snapshot.length;
    }

    public boolean hasPrevious() {
        return cursor > 0;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        if (! hasNext())
            throw new NoSuchElementException();
        return (E) snapshot[cursor++];
    }
}
```



调用iterator方法获取迭代器返回一个COWIterator对象

COWIterator的构造器里主要是 保存了当前的list对象的内容和遍历list时数据的下标。

snapshot是list的快照信息，因为CopyOnWriteArrayList的读写策略中都会使用getArray()来获取一个快照信息，生成一个新的数组。

所以在使用该迭代器元素时，其他线程对该lsit操作是不可见的，因为操作的是两个不同的数组所以造成弱一致性。



```java
private static void CopyOnWriteArrayListTest(){
    CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList();
    list.add("test1");
    list.add("test2");
    list.add("test3");
    list.add("test4");
    
    Thread thread = new Thread(() -> {
        System.out.println(">>>> start");
        list.add(1, "replaceTest");
        list.remove(2);
    });
    
    // 在启动线程前获取迭代器
    Iterator<String> iterator = list.iterator();

    thread.start();

    try {
        // 等待线程执行完毕
        thread.join();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    while (iterator.hasNext()){
        System.out.println(iterator.next());
    }
}
```



上面的demo中在启动线程前获取到了原来list的迭代器，

在之后启动新建一个线程，在线程里面修改了第一个元素的值，移除了第二个元素

在执行完子线程之后，遍历了迭代器的元素，发现子线程里面操作的一个都没有生效，这里提现了迭代器弱一致性。



####  CopyOnWriteArrayList的迭代器不支持增删改

```java
private static void CopyOnWriteArrayListTest(){
    CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
    list.add("test1");
    list.add("test2");
    list.add("test3");
    list.add("test4");

    Iterator<String> iterator = list.iterator();

    while (iterator.hasNext()){
        if ("test1".equals(iterator.next())){
            iterator.remove();
        }
    }

    System.out.println(list.toString());
}
```



CopyOnWriteArrayList 迭代器是只读的，不支持增删操作

CopyOnWriteArrayList迭代器中的 remove()和 add()方法，没有支持增删而是直接抛出了异常

因为迭代器遍历的仅仅是一个快照，而对快照进行增删改是没有意义的。



#### 源码解析

> java.util.concurrent.CopyOnWriteArrayList

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    
}
```



### BlockingQueue

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。

这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。

阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

JDK7 提供了 7 个阻塞队列。分别是：

ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。

LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。

PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。

DelayQueue：一个使用优先级队列实现的无界阻塞队列。

SynchronousQueue：一个不存储元素的阻塞队列。

LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。

LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

Java 5 之前实现同步存取时，可以使用普通的一个集合，然后在使用线程的协作和线程同步可以实现生产者，消费者模式，主要的技术就是用好，wait,notify,notifyAll,sychronized 这些关键字。而在 java 5 之后，可以使用阻塞队列来实现，此方式大大简少了代码量，使得多线程编程更加容易，安全方面也有保障。

BlockingQueue 接口是 Queue 的子接口，它的主要用途并不是作为容器，而是作为线程同步的的工具，因此他具有一个很明显的特性，当生产者线程试图向 BlockingQueue 放入元素时，如果队列已满，则线程被阻塞，当消费者线程试图从中取出一个元素时，如果队列为空，则该线程会被阻塞，正是因为它所具有这个特性，所以在程序中多个线程交替向 BlockingQueue 中放入元素，取出元素，它可以很好的控制线程之间的通信。

阻塞队列使用最经典的场景就是 socket 客户端数据的读取和解析，读取数据的线程不断将数据放入队列，然后解析线程不断从队列取数据解析。



生产者消费者模型

经典的“生产者”和“消费者”模型中，在concurrent包发布以前，在多线程环境下，我们每个程序员都必须去自己控制这些细节，尤其还要兼顾效率和线程安全，而这会给我们的程序带来不小的复杂度。好在此时，强大的concurrent包横空出世了，而他也给我们带来了强大的BlockingQueue。（在多线程领域：所谓阻塞，在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会自动被唤醒）



#### BlockingQueue成员

因为它隶属于集合家族，自己又是个接口。所以是有很多成员的，下面简单介绍一下

###### 1. ArrayBlockingQueue

基于数组的阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定长数组，以便缓存队列中的数据对象，这是一个常用的阻塞队列，除了一个定长数组外，ArrayBlockingQueue内部还保存着两个整形变量，分别标识着队列的头部和尾部在数组中的位置。 ArrayBlockingQueue在生产者放入数据和消费者获取数据，都是共用同一个锁对象，由此也意味着两者无法真正并行运行，这点尤其不同于LinkedBlockingQueue；按照实现原理来分析，ArrayBlockingQueue完全可以采用分离锁，从而实现生产者和消费者操作的完全并行运行。Doug Lea之所以没这样去做，也许是因为ArrayBlockingQueue的数据写入和获取操作已经足够轻巧，以至于引入独立的锁机制，除了给代码带来额外的复杂性外，其在性能上完全占不到任何便宜。 ArrayBlockingQueue和LinkedBlockingQueue间还有一个明显的不同之处在于，前者在插入或删除元素时不会产生或销毁任何额外的对象实例，而后者则会生成一个额外的Node对象。这在长时间内需要高效并发地处理大批量数据的系统中，其对于GC的影响还是存在一定的区别。而在创建ArrayBlockingQueue时，我们还可以控制对象的内部锁是否采用公平锁，默认采用非公平锁。

###### 2.LinkedBlockingQueue

基于链表的阻塞队列，同ArrayListBlockingQueue类似，其内部也维持着一个数据缓冲队列（该队列由一个链表构成），当生产者往队列中放入一个数据时，队列会从生产者手中获取数据，并缓存在队列内部，而生产者立即返回；只有当队列缓冲区达到最大值缓存容量时（LinkedBlockingQueue可以通过构造函数指定该值），才会阻塞生产者队列，直到消费者从队列中消费掉一份数据，生产者线程会被唤醒，反之对于消费者这端的处理也基于同样的原理。而LinkedBlockingQueue之所以能够高效的处理并发数据，还因为其对于生产者端和消费者端分别采用了独立的锁来控制数据同步，这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。 作为开发者，我们需要注意的是，如果构造一个LinkedBlockingQueue对象，而没有指定其容量大小，LinkedBlockingQueue会默认一个类似无限大小的容量（Integer.MAX_VALUE），这样的话，如果生产者的速度一旦大于消费者的速度，也许还没有等到队列满阻塞产生，系统内存就有可能已被消耗殆尽了。

###### 3. DelayQueue 延迟队列

DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞，所以一定要注意内存的使用。 使用场景： 　　DelayQueue使用场景较少，但都相当巧妙，常见的例子比如使用一个DelayQueue来管理一个超时未响应的连接队列。

###### 4. PriorityBlockingQueue

基于优先级的阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定），但需要注意的是PriorityBlockingQueue并不会阻塞数据生产者，而只会在没有可消费的数据时，阻塞数据的消费者。因此使用的时候要特别注意，**生产者生产数据的速度绝对不能快于消费者消费数据的速度**，否则时间一长，会最终耗尽所有的可用堆内存空间。在实现PriorityBlockingQueue时，内部控制线程同步的锁采用的是公平锁。

###### 5. SynchronousQueue

一种无缓冲的等待队列，类似于无中介的直接交易，有点像原始社会中的生产者和消费者，生产者拿着产品去集市销售给产品的最终消费者，而消费者必须亲自去集市找到所要商品的直接生产者，如果一方没有找到合适的目标，那么对不起，大家都在集市等待。相对于有缓冲的BlockingQueue来说，少了一个中间经销商的环节（缓冲区），如果有经销商，生产者直接把产品批发给经销商，而无需在意经销商最终会将这些产品卖给那些消费者，由于经销商可以库存一部分商品，因此相对于直接交易模式，总体来说采用中间经销商的模式会吞吐量高一些（可以批量买卖）；但另一方面，又因为经销商的引入，使得产品从生产者到消费者中间增加了额外的交易环节，单个产品的及时响应性能可能会降低。

##### 小结

BlockingQueue不光实现了一个完整队列所具有的基本功能，同时在多线程环境下，他还自动管理了多线间的自动等待于唤醒功能，从而使得程序员可以忽略这些细节，关注更高级的功能。



### ConcurrentLinkedQueue

ConcurrentLinkedQueue非阻塞无界链表队列

ConcurrentLinkedQueue是一个线程安全的队列，基于链表结构实现，是一个无界队列，理论上来说队列的长度可以无限扩大。

与其他队列相同，ConcurrentLinkedQueue也采用的是先进先出（FIFO）入队规则，对元素进行排序。 （推荐学习：java面试题目）

当我们向队列中添加元素时，新插入的元素会插入到队列的尾部；而当我们获取一个元素时，它会从队列的头部中取出。

因为ConcurrentLinkedQueue是链表结构，所以当入队时，插入的元素依次向后延伸，形成链表；而出队时，则从链表的第一个元素开始获取，依次递增；

值得注意的是，在使用ConcurrentLinkedQueue时，如果涉及到队列是否为空的判断，切记不可使用size()==0的做法，因为在size()方法中，是通过遍历整个链表来实现的，在队列元素很多的时候，size()方法十分消耗性能和时间，只是单纯的判断队列为空使用isEmpty()即可。



## 生产实践

### **多线程下 i++** 

- 使用循环CAS，实现i++原子操作
- 使用锁机制，实现i++原子操作
- 使用synchronized，实现i++原子操作



```java
public class AtomicIntegerTest {
    private static AtomicInteger atomicInteger = new AtomicInteger(0);
    public static void main(String[] args) throws InterruptedException {
        testIAdd();
    }
    private static void testIAdd() throws InterruptedException {
        //创建线程池
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        for (int i = 0; i < 1000; i++) {
            executorService.execute(() -> {
                for (int j = 0; j < 2; j++) {
                    //自增并返回当前值
                    int andIncrement = atomicInteger.incrementAndGet();
                    System.out.println("线程:" + Thread.currentThread().getName() + " count=" + andIncrement);
                }
            });
        }
        executorService.shutdown();
        Thread.sleep(100);
        System.out.println("最终结果是 ：" + atomicInteger.get());
    }
}
```



# AQS

AbstractQueuedSynchronizer同步器，实现JUC核心基础组件。解决了子类实现同步器时涉及的大量细节问题，例如获取同步状态、FIFO同步队列等。采用模板方法模式，AQS实现大量通用方法，子类通过继承方式实现其抽象方法来管理同步状态。



## CLH同步队列

FIFO双向队列，AQS依赖该队列来解决同步状态的管理问题。首节点唤醒，等待队列加入到CLH同步队列的尾部。

![AQS原理图](../../Image/2022/07/220728-7.png)

CLH 同步队列,全英文`Craig, Landin, and Hagersten locks`。是一个FIFO双向队列，其内部通过节点head和tail记录队首和队尾元素，队列元素的类型为Node。AQS依赖它来完成同步状态state的管理，当前线程如果获取同步状态失败时，AQS则会将当前线程已经等待状态等信息构造成一个节点（Node）并将其加入到CLH同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点唤醒（公平锁），使其再次尝试获取同步状态。



## 同步状态的获取与释放

### 独占式

#### 获取锁

##### 获取同步状态

> java.util.concurrent.locks.AbstractQueuedSynchronizer

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```



##### 响应中断

> java.util.concurrent.locks.AbstractQueuedSynchronizer

```java
public final void acquireInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (!tryAcquire(arg))
        doAcquireInterruptibly(arg);
}
```



##### 超时获取

> java.util.concurrent.locks.AbstractQueuedSynchronizer

```java
public final boolean tryAcquireNanos(int arg, long nanosTimeout)
    throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    return tryAcquire(arg) ||
        doAcquireNanos(arg, nanosTimeout);
}
```



#### 释放锁

> java.util.concurrent.locks.AbstractQueuedSynchronizer

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```



### 共享式

#### 获取锁

> java.util.concurrent.locks.AbstractQueuedSynchronizer

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```



#### 释放锁

> java.util.concurrent.locks.AbstractQueuedSynchronizer

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```



# CAS

Compare And Swap，整个JUC体系最核心，最基础理论。

内存值V，旧的预期值A，要更新的值B，当且仅当内存值V的值等于旧的预期值A时才会将内存值V的值修改为B，否则什么都步处理。



> sun.misc.Unsafe

```java
// CAS，如果对象偏移量上的值=期待值，更新为x，返回true，否则false.
public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object x);  
```

与上述方法类似的有compareAndSwapInt，compareAndSwapLong，compareAndSwapBoolean，compareAndSwapChar等等。  



缺陷：

- 循环时间太长；
- 只能保证一个共享变量原子操作；
- ABA问题，该问题可以通过版本号或AtomicStampedReference解决。



# 并发集合

## ConcurrentHashMap

CAS + Synchronized 来保证并发更新的安全，底层采用数据 + 链表/红黑树的存储结构。



### 内部类

#### Node

key-value键值对。



#### TreeNode

红黑树节点。



#### TreeBin

相关与一颗红黑树，其构造方法就是构造红黑树的过程。



#### ForwardingNode

辅助节点，用于ConcurrentHashMap扩容操作。



### 属性

#### sizeCtl

控制标识符，用来控制table的初始化和扩容操作。

负数表示正在进行初始化或扩容操作，-1表示正在初始化，-N表示有N-1个线程正在进行扩容操作。正数或0表示hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小。



### 源码解析

#### initTable

初始化方法，只能有一个线程参与初始化过程，其它线程必须挂起。构造函数不做初始化，只有在put操作时才触发初始化操作。

sizeCtl为负数时，表示正在进行初始化，线程挂起；线程获取初始化资格后进行初始化过程；初始化步骤完成后，设置sizeClt的值，表示下一次扩容的大小。



> java.util.concurrent.ConcurrentHashMap

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            Thread.yield();
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            // CAS操作返回true表示获取初始化资格成功
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                // 设置sizeCtl的值，即0.75n
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```



#### put

根据hash值计算节点插入在table的位置，如果该位置为空，则直接插入，否则插入到链表或红黑树。步骤如下：

- table为null，线程进入初始化，如果有其它线程正在初始化，则该线程挂起；
- 如果插入的当前位置为null，则说明该位置是第一次插入，利用CAS插入节点即可，插入成功则调用addCount判断是否需要扩容，插入失败则通过自旋继续匹配；
- 如果该节点的hash == MOVED，则表示有线程正在扩容，进入扩容进程中；
- 其余情况就是按照链表或红黑树结构插入节点，该过程需要加锁（synchronized）。



> java.util.concurrent.ConcurrentHashMap

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

public V putIfAbsent(K key, V value) {
    return putVal(key, value, true);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;           
        }
        else if ((fh = f.hash) == MOVED)
            // 表示有线程正在扩容，进入扩容进程中
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // 加锁处理链表或红黑树数据结构
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                              value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```



#### get

从链表或红黑树节点中获取数据。

> java.util.concurrent.ConcurrentHashMap

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```



#### 扩容

扩容步骤如下：

- 构建一个nextTable，其大小为原来大小的2倍，该步骤在单线程环境下完成；
- 将原来的table的内容复制到nextTable中，该步骤允许多线程操作。



> java.util.concurrent.ConcurrentHashMap

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                        (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                        (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
```



#### 链表转换为红黑树

如果链表中的元素个数达到了阈值8，则将链表转换为红黑树。



> java.util.concurrent.ConcurrentHashMap

```java
// 链表转红黑树阈值
static final int TREEIFY_THRESHOLD = 8;
// 红黑树转链表阈值
static final int UNTREEIFY_THRESHOLD = 6;
```



> java.util.concurrent.ConcurrentHashMap

```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}
```



## ConcurrentLinkedQueue

ConcurrentLinkedQueue是基于链接节点的无边界的线程安全队列，采用FIFO原则对元素进行排序，内部采用CAS算法实现。该队列允许不一致性和弱一致性。

ConcurrentLinkedQueue的不变性如下：

- 在入队的最后一个元素的next为null；
- 队列中所有未删除节点的item都不能为null且都能从head节点遍历到；
- 对于要删除的节点，不是直接将其设置为null，而是先将其item域设置为null（迭代器会跳过item为null的节点）；
- 允许head和tail更新之后，即head和tail不是总是指向第一个元素和最后一个元素；



## ConcurrentSkipListMap

该队列采用跳表数据结构，跳表是一种平衡二叉树结构，让已排序的数据分布在多层链表中，以0-1随机数决定一个数据的向上攀升与否，是一种以**空间换时间**的算法。

通过在每个节点中增加了向前的指针，在插入、删除、查找时可以忽略一些不可能涉及到的节点，从而提高了效率。

跳表的结构具备如下特性：

- 由多层结构组成，level是通过一定的概率随机生成的；
- 每一层都是一个有序的链表，默认升序。也可以根据创建时提供的Comparator来指定排序的方式；
- 最底层的链表包含所有的元素；
- 元素一定会出现在它第一次出现的层级及之下的所有层级；
- 每个节点包含两个指针，一个指向同一层级链表的下一个元素，另一个指向下面一层的元素。



## ConcurrentSkipListSet

内部采用ConcurrentSkipListMap实现。



# 并发工具类

## CyclicBarrier

CyclicBarrier允许一组线程互相等待，直到到达某个公共屏障点（common barrier point）。即让一组线程到达一个屏障时倍阻塞，直到最后一个线程到达屏障时，屏障才会打开，所有被阻塞的线程继续运行。

**底层实现采用ReentrantLock + Condition。**一般应用于多线程结果合并的操作，用于多线程计算数据，最后合并计算结果的场景。



## CountDownLatch

在完成一组正在其他线程中执行的操作之前，允许一个或多个线程一直等待。底层实现采用**共享锁**。

用给定的计数初始化CountDownLatch，由于调用了countDown方法，当计数到达0之前，await方法会一直阻塞。然后会释放所有等待的线程，await的所有后续调用都会立即方悔。这种现象只会出现一次，计数无法被重置。



### 扩展

#### 与CyclicBarrier的区别

CountDownLatch是允许一个或多个线程等待其他线程完成，CyclicBarrier是允许多个线程互相等待。

CountDownLatch的计数无法被重置，CyclicBarrier的计数可以被重置后使用，因此被称为循环的barrier。



## Semaphore

Semaphore表示信号量，用来控制访问多个共享资源的计数器。底层实现采用**共享锁**。



## Exchanger



# Java JMM

Java内存模型（JMM）



## 线程通信机制

线程通信机制有内存共享和消息传递两种，Java采用的是内存共享的方式。



## 内存模型

### 重排序

重排序指的是为了程序的性能，处理器、编译器都会对程序进行重排序处理。重排序需要具备如下条件：

- 在单线程环境下不能改变程序运行的结果；
- 存在数据依赖关系的情况下不允许重排序。



重排序在多线程环境下可能会导致数据不安全。



### 顺序一致性

顺序一致性是在多线程环境下的理论参考模型，为程序提供了内存可见性保证。顺序一致性具备如下特点：

- 一个线程中的所有操作必须按照程序的顺序来执行；
- 所有线程都只能看到一个单一的操作执行顺序；
- 每个都做都必须原子执行且立刻对多有线程可见。



### happens-before

happens-before是JMM核心理论，保证内存可见性。在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系。

如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，且第一个操作的执行顺序在第二个操作之前。两个操作之间存在happens-before关系，并不一定要按照happens-before原则指定的顺序来执行。如果重排序后的执行结果与按照happens-before关系来执行的结果一致，则这种重排序并不非法。



### as-if-serial

as-if-serial表示所有操作都可以为了优化而被重排序，但是必须保证重排序后执行的结果不能改变。



## synchronized

synchronized可以保证方法或代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时还可以保证共享变量的内存可见性。



### 锁对象

#### 普通同步方法

锁是当前实例对象。



#### 静态同步方法

锁是当前类的class对象。



#### 同步方法块

锁是括号里面指定的对象。



### 实现机制

#### Java对象头

synchronized的锁就是保存在Java对象头中的，包含标记字段（Mark Word）和类型指针（Class Pointer）两部分数据。

标记字段被设计成一个非固定的数据结构以便于在极小的空间内存中存储尽可能多的数据，会根据对象的状态复用存储空间。标记字段包括哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID和偏向时间戳。



#### monitor

初始时为null表示当前没有任何线程拥有该monitor record，当线程成功拥有该锁后保存线程唯一标识，当锁被释放时又设置为null。



### 锁优化

#### 自旋锁

自旋锁表示线程通过**循环方式**等待一段时间，不会立即挂起，看持有锁的线程是否会很快释放锁。

因为线程的频繁挂起和唤醒负担较重，如果认为每个线程占有锁的时间很短，线程挂起唤醒就会得不偿失，此时就需要通过使用自旋锁来避免线程频繁挂起唤醒。



##### 适应性自旋锁

因为自旋锁自旋的次数不固定，是由前一次在同一个锁上的自旋时间以及锁的拥有者的状态来决定的，这就带来了不确定性，因此引入了适应性自旋锁。

适应性自旋锁表示自旋成功时，加入自旋次数。如果获取锁经常失败，则降低自旋次数。



#### 锁消除

锁消除表示通过将变量逃逸分析作为依据，如果不存在数据竞争的情况下，JVM会消除锁机制。



#### 锁粗化

锁粗化表示将多个连续的加锁和解锁操作连接在一起，扩展成一个范围更大的锁。例如for循环内部获取锁。



### 锁类型

#### 轻量级锁

因为绝大部分的锁在生命周期内都是不会存在竞争的，因此引入了轻量级锁，通过CAS来获取锁和释放锁。在没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能损耗。

轻量级的缺点在于在多线程环境下，运行效率会比重量级锁还要低。



#### 偏向锁

偏向锁主要用来在非多线程竞争的情况下减少不必要的轻量级锁执行路径，避免不必要的CAS操作。如果竞争失败则会升级为轻量级锁。



#### 重量级锁



## volatile

通过volatile修饰的变量，总是可以获取到该变量最新的写。



## DCL



# 线程池

## 概述



## Executor

### Executors



### ThreadPoolExecutor





## Future

### Future



### FutureTask



# 阻塞队列

java.util.concurrent.BlockingQueue的特性是：当队列是空的时，从队列中获取或删除元素的操作将会被阻塞，或者当队列是满时，往队列里添加元素的操作会被阻塞。

阻塞队列不接受空值，当你尝试向队列中添加空值的时候，它会抛出NullPointerException。

阻塞队列的实现都是线程安全的，所有的查询方法都是原子的并且使用了内部锁或者其他形式的并发控制。

BlockingQueue 接口是java collections框架的一部分，它主要用于实现生产者-消费者问题。



## ArrayBlockingQueue

是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。在构造函数时确认大小，确认后不支持改变。在多线程环境下不保证公平性。基于ReentrantLock + Condition实现。



## LinkedBlockingQueue

一个基于链表结构的无界阻塞队列，此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。Executors.newFixedThreadPool()使用了这个队列



## DelayQueue

支持延时获取元素的无界阻塞队列，主要应用于缓存超时数据清除和任务超时处理。基于ReentrantLock + Condition和根据Delay时间排序的优先级队列PriorityQueue实现。

Delayed接口用来标记哪些应该在给定延时之后执行的对象，该接口要求实现类必须定义一个compareTo方法，该方法提供与该接口的getDelay方法一致的排序。



## SynchronousQueue

一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，Executors.newCachedThreadPool使用了这个队列。



## PriorityBlockingQueue

一个具有优先级的无界阻塞队列，默认情况下采用自然顺序升序排序，可以通过指定Comparator来对元素进行排序。基于ReentrantLock + Conditon 和 二叉堆实现。

二叉堆可以分为最大堆和最小堆，最大堆的父节点的键值总是大于或等于任何一个子节点的键值，最小堆的父节点的键值总是小于或等于任何一个子节点的键值。其添加操作则是不断“上冒”，而删除操作则是不断”下掉“。



## LinkedTransferQueue

一个由链表结构组成的无界阻塞队列，相当于ConcurrentLinkedQueue、SynchronousQueue（公平模式下）、无界的LinkedBlockingQueues等的超集。

预占模式表示有就直接拿走，没有就占着位置直到拿到或者超时、中断。



## LinkedBlockingDeque

一个由链表结构组成的双向阻塞队列，容量可选，在初始化时指定容量大小防止过度膨胀。如果不设置则默认大小为Integer.MAX_VALUE。通过工作窃取模式实现。



# 锁

## ReentrantLock



## ReentrantReadWriteLock



## Condition

Lock提供条件Condition，对线程的等待、唤醒操作更加详细和灵活。内部维护一个Condition队列。当线程调用await方法，将会以当前线程构造成一个节点，并将节点加入到该队列的尾部。



# Atomic原子类

## 基本类型

通过原子的方式更新基本数据类型。如AtomicBoolean、AtomicInteger、AtomicLong等。

| 类名          | 描述             |
| ------------- | ---------------- |
| AtomicBoolean | 原子更新布尔类型 |
| AtomicInteger | 原子更新整型     |
| AtomicLong    | 原子更新长整型   |



## 数组类型

通过原子的方式更新数组中的某个元素。如AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray等。

| 类名                 | 描述                         |
| -------------------- | ---------------------------- |
| AtomicIntegerArray   | 原子更新整型数组中的元素     |
| AtomicLongArray      | 原子更新长整型数组中的元素   |
| AtomicReferenceArray | 原子更新引用类型数组中的元素 |



## 引用类型

通过原子的方式更新多个变量。如AtomicReference、AtomicReferenceFieldUpdater、AtomicMarkableReference等。

| 类名                        | 描述                         |
| --------------------------- | ---------------------------- |
| AtomicReference             | 原子更新引用类型             |
| AtomicReferenceFieldUpdater | 原子更新引用类型中的字段     |
| AtomicMarkableReference     | 原子更新带有标记位的引用类型 |



## 字段类型

通过原子的方式更新某个类中的某个字段。AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicStampedReference等。

| 类名                      | 描述                         |
| ------------------------- | ---------------------------- |
| AtomicIntegerFieldUpdater | 原子更新整型字段的更新器     |
| AtomicLongFieldUpdater    | 原子更新长整型字段的更新器   |
| AtomicStampedReference    | 原子更新带有版本号的引用类型 |



# ThreadLocal



# Fork/Join

