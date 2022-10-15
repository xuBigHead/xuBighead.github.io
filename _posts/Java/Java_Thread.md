# 概念

使用多线程就是因为： **在正确的场景下，设置恰当数目的线程，可以用来程提高序的运行速率。更专业点讲，就是充分地利用CPU和I/O的利用率，提升程序运行速率。**



## 线程、进程和程序

### 线程

**线程**与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享同一块内存空间和一组系统资源，每个线程有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**。所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

线程是操作系统能够进行运算调度的最小单位，它被包含在进程之中，是进程中的实际运作单位。

程序员可以通过它进行多处理器编程，你可以使用多线程对运算密集型任务提速。比如，如果一个线程完成一个任务要100毫秒，那么用十个线程完成该任务只需10毫秒。



```java
    @Test
    public void multiThread(){
        // 获取 Java 线程管理 MXBean
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        // 不需要获取同步的 monitor 和 synchronizer 信息，仅获取线程和线程堆栈信息
        ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
        // 遍历线程信息，仅打印线程 ID 和线程名称信息
        for (ThreadInfo threadInfo : threadInfos) {
            System.out.println("[" + threadInfo.getThreadId() + "] " + threadInfo.getThreadName());
        }
    }
```



输出结果：

```
[6] Monitor Ctrl-Break 
[5] Attach Listener    //添加事件
[4] Signal Dispatcher  // 分发处理给 JVM 信号的线程
[3] Finalizer          //调用对象 finalize 方法的线程
[2] Reference Handler  //清除 reference 线程
[1] main               //main 线程,程序入口
```



**一个 Java 程序的运行是 main 线程和多个其他线程同时运行**。在Java里面没有办法强制启动一个线程，它被线程调度器控制且Java没有公布相关的API。





### 进程

**进程**是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。简单来说，一个进程就是一个执行中的程序，它在计算机中一个指令接着一个指令地执行着，同时，每个进程还占有某些系统资源如 CPU 时间，内存空间，文件，输入输出设备的使用权等等。换句话说，当程序在执行时，将会被操作系统载入内存中。 线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。从另一角度来说，进程属于操作系统的范畴，主要是同一段时间内，可以同时执行一个以上的程序，而线程则是在同一程序内几乎同时执行一个以上的程序段。

一个进程是一个独立(self contained)的运行环境，它可以被看作一个程序或者一个应用。而线程是在进程中执行的一个任务。

线程是进程的子集，一个进程可以有很多线程，每条线程并行执行不同的任务。不同的进程使用不同的内存空间，而所有的线程共享一片相同的内存空间（堆）。每个线程都拥有单独的栈内存（虚拟机栈）用来存储本地数据。



- 进程是运行中的应用程序，线程是进程的内部的一个执行序列
- 进程是资源分配的最小单位，线程是CPU调度的最小单位。
- 一个进程可以有多个线程。线程又叫做轻量级进程，多个线程共享进程的资源
- 进程间切换代价大，线程间切换代价小
- 进程拥有资源多，线程拥有资源少地址
- 进程是存在地址空间的，而线程本身无地址空间，线程的地址空间是包含在进程中的



### 程序

**程序**是含有指令和数据的文件，被存储在磁盘或其他的数据存储设备中，也就是说程序是静态的代码。



## 并发和并行

- **并发：** 同一时间段，多个任务都在执行 (单位时间内不一定同时执行)；
- **并行：** 单位时间内，多个任务同时执行。



## 线程状态

Java 线程在运行的生命周期中的指定时刻只可能处于下面 6 种不同状态的其中一个状态：

| 状态名称                    | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| **初始(NEW)**               | 新创建了一个线程对象，但还没有调用start()方法。              |
| **运行(RUNNABLE)**          | Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。<br/>线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。 |
| **阻塞(BLOCKED)**           | 表示线程阻塞于锁。                                           |
| **等待(WAITING)**           | 进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。 |
| **超时等待(TIMED_WAITING)** | 该状态不同于WAITING，它可以在指定的时间后自行返回。          |
| **终止(TERMINATED)**        | 表示该线程已经执行完毕。                                     |



线程在生命周期中并不是固定处于某一个状态而是随着代码的执行在不同状态之间切换。Java 线程状态变迁如下图所示

![Java线程状态变迁](../../Image/2022/07/220715-2.png)

线程创建之后它将处于 **NEW（新建）** 状态，调用 `start()` 方法后开始运行，线程这时候处于 **READY（可运行）** 状态。可运行状态的线程获得了 cpu 时间片（timeslice）后就处于 **RUNNING（运行）** 状态。



> 操作系统隐藏 Java 虚拟机（JVM）中的 READY 和 RUNNING 状态，它只能看到 RUNNABLE 状态（图源：[HowToDoInJava](https://howtodoinjava.com/)：[Java Thread Life Cycle and Thread States](https://howtodoinjava.com/java/multi-threading/java-thread-life-cycle-and-thread-states/)），所以 Java 系统一般将这两个状态统称为 **RUNNABLE（运行中）** 状态 。



当线程执行 `wait()`方法之后，线程进入 **WAITING（等待）**状态。进入等待状态的线程需要依靠其他线程的通知才能够返回到运行状态，而 **TIME_WAITING(超时等待)** 状态相当于在等待状态的基础上增加了超时限制，比如通过 `sleep（long millis）`方法或 `wait（long millis）`方法可以将 Java 线程置于 TIMED WAITING 状态。当超时时间到达后 Java 线程将会返回到 RUNNABLE 状态。当线程调用同步方法时，在没有获取到锁的情况下，线程将会进入到 **BLOCKED（阻塞）** 状态。线程在执行 Runnable 的`run()`方法之后将会进入到 **TERMINATED（终止）** 状态。



### 初始状态New

实现Runnable接口和继承Thread可以得到一个线程类，new一个实例出来，线程就进入了初始状态。线程对象创建之后、但还没有调用`start()`方法，就是这个状态。

在Java中使用new关键字创建一个线程，新创建的线程将处于新建状态。在创建线程时主要是为线程分配内存并初始化其成员变量的值。



### 可运行状态Runnable

**执行Thread的start方法之后，线程进行RUNNABLE可运行状态。**

它包括就绪（`ready`）和运行中（`running`）两种状态。如果调用`start`方法，线程就会进入`Runnable`状态。它表示我这个线程可以被执行啦（此时相当于`ready`状态），如果这个线程被调度器分配了CPU时间，那么就可以被执行（此时处于`running`状态）。

新建的线程对象在调用start方法之后将转为可运行状态。运行状态中又分：就绪（Ready）和运行中（Running）两种状态。就绪状态指的是JVM完成了方法调用栈和程序计数器的创建，等待该线程的调度和运行。

就绪状态的线程在竞争到CPU的使用权并开始执行run方法的线程执行体时，会转为运行中状态，处于运行中状态的线程的主要任务就是执行run方法中的逻辑代码。



#### 就绪状态Ready

1. 就绪状态只是说你资格运行，调度程序没有挑选到你，你就永远是就绪状态。
2. 调用线程的start()方法，此线程进入就绪状态。
3. 当前线程sleep()方法结束，其他线程join()结束，等待用户输入完毕，某个线程拿到对象锁，这些线程也将进入就绪状态。
4. 当前线程时间片用完了，调用当前线程的yield()方法，当前线程进入就绪状态。
5. 锁池里的线程拿到对象锁后，进入就绪状态。



#### 运行中状态Running

线程调度程序从可运行池中选择一个线程作为当前线程时线程所处的状态。这也是线程进入运行状态的唯一的一种方式。



### 阻塞状态Blocked

阻塞状态是线程阻塞在进入synchronized关键字修饰的方法或代码块(获取锁)时的状态。

阻塞的（被同步锁或者IO锁阻塞）。表示线程阻塞于锁，线程阻塞在进入`synchronized`关键字修饰的方法或代码块(**等待获取锁**)时的状态。比如前面有一个临界区的代码需要执行，那么线程就需要等待，它就会进入这个状态。它一般是从`RUNNABLE`状态转化过来的。如果线程获取到锁，它将变成`RUNNABLE`状态



运行中的线程会主动或被动地放弃 CPU 的使用权并暂停运行，此时该线程将转为阻塞状态，直到再次进入可运行状态，才有机会再次竞争到CPU使用权并转为运行状态。阻塞状态分为如下三种。

**（1）等待阻塞：**在运行状态的线程调用o.wait方法时，JVM会把该线程放入等待队列（Waitting Queue）中，线程转为阻塞状态。

**（2）同步阻塞：**在运行状态的线程尝试获取正在被其他线程占用的对象同步锁时，JVM会把该线程放入锁池（Lock Pool）中，此时线程转为阻塞状态。

**（3）其他阻塞：**运行状态的线程在执行Thread.sleep(long ms)、Thread.join()或者发出I/O请求时，JVM会把该线程转为阻塞状态。直到sleep()状态超时、Thread.join()等待线程终止或超时，或者I/O处理完毕，线程才重新转为可运行状态。



### 等待状态Waiting

处于这种状态的线程不会被分配CPU执行时间，它们要等待被显式地唤醒，否则会处于无限期等待的状态。

永久等待状态，进入该状态的线程需要等待其他线程做出一些特定动作（比如通知）。处于该状态的线程不会被分配CPU执行时间，它们要等待被显式地唤醒，否则会处于无限期等待的状态。一般`Object.wait`。

当线程调用了Object.wait()、Thread.join()、LockSupport.park()会进入等待状态。处于等待状态的线程正在等待另一个线程执行指定的操作。例如，调用Object.wait()的一个线程对象正在等待另一个线程调用该对象的Object.notify()或Object.notifyAll()。调用thread .join()的线程正在等待指定的线程退出。



### 超时等待状态Timed_Waiting

处于这种状态的线程不会被分配CPU执行时间，不过无须无限期等待被其他线程显示地唤醒，在达到一定时间后它们会自动唤醒。

等待指定的时间重新被唤醒的状态。有一个计时器在里面计算的，最常见就是使用`Thread.sleep`方法触发，触发后，线程就进入了`Timed_waiting`状态，随后会由计时器触发，再进入`Runnable`状态。



超时等待和等待状态的不同是，超时等待状态的线程经过超时时间后会自动唤醒。当线程调用了Thread.sleep (long)、Object.wait(long)、Thread.join(long)、LockSupport.parkNanos()、LockSupport.parkUntil()后线程会进入超时等待状态。



### 终止状态Terminated

当线程的run()方法完成时，或者主线程的main()方法完成时，我们就认为它终止了。这个线程对象也许是活的，但是它已经不是一个单独执行的线程。线程一旦终止了，就不能复生。

在一个终止的线程上调用start()方法，会抛出java.lang.IllegalThreadStateException异常。



线程在以如下三种方式结束后转为终止状态。

◎ **线程正常结束：**run方法或call方法执行完成。

◎ **线程异常退出：**运行中的线程抛出一个Error或未捕获的Exception，线程异常退出。

◎ **手动结束：**调用线程对象的stop方法手动结束运行中的线程（该方式会瞬间释放线程占用的同步对象锁，导致锁混乱和死锁，不推荐使用）。



```java
public class ThreadTest {
    private static Object object = new Object();
    public static void main(String[] args) throws Exception {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    for(int i = 0; i< 1000; i++){
                        System.out.print("");
                    }
                    Thread.sleep(500);
                    synchronized (object){
                        object.wait();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    synchronized (object){
                        Thread.sleep(1000);
                    }
                    Thread.sleep(1000);
                    synchronized (object){
                        object.notify();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        
        System.out.println("1"+thread.getState());
        thread.start();
        thread1.start();
        System.out.println("2"+thread.getState());
        while (thread.isAlive()){
            System.out.println("---"+thread.getState());
            Thread.sleep(100);
        }
        System.out.println("3"+thread.getState());
    }
}
```



## 线程状态转换

**（1）**调用new方法新建一个线程，这时线程处于新建状态。

**（2）**调用start方法启动一个线程，这时线程处于可运行状态。可运行状态中又分：就绪（Ready）和运行中（Running）两种状态。处于就绪状态的线程等待线程获取CPU资源，在等待其获取CPU资源后线程会执行run方法进入运行中状态；正在运行的线程在调用了yield方法或失去处理器资源时，会再次进入就绪状态。

**（3）**正在运行中的线程在执行了sleep方法、I/O阻塞、等待同步锁、等待通知、调用suspend方法等操作后，会挂起并进入阻塞状态。阻塞状态的线程由于出现sleep时间已到、I/O方法返回、获得同步锁、收到通知、调用resume方法等情况，会再次进入可运行状态中的就绪状态，等待CPU时间片的轮询。该线程在获取CPU资源后，会再次进入运行状态。

**（4）**当线程调用了Object.wait ()、Object.join()、LockSupport.park()后线程进入等待状态。等待状态的线程调用Object.notify ()、Object. notifyAll()、LockSupport.unpark(Thread)方法后会再次进入可运行状态。

**（5）**当可运行状态的线程调用Thread.sleep(long)、Object.wait(long)、Thread.join(long)、LockSupport.parkNanos()、LockSupport.parkUntil()时线程会进入超时等待状态。当超时等待的线程出现超时时间到、等待进入synchronized方法、等待进入synchronized块或者调用Object.notify ()、Object. notifyAll()、LockSupport.unpark(Thread)时会再次进入可运行状态。

**（6）**处于可运行状态的线程，在调用run方法或call方法正常执行完成、调用stop方法停止线程或者程序执行错误导致异常退出时，会进入终止状态。



## 线程优先级

每一个线程都是有优先级的，一般来说，高优先级的线程在运行时会具有优先权，但这依赖于线程调度的实现，这个实现是和操作系统相关的(OS dependent)。我们可以定义线程的优先级，但是这并不能保证高优先级的线程会在低优先级的线程前执行。线程优先级是一个int变量(从1-10)，1代表最低优先级，10代表最高优先级。



## Java内存模型



## 多线程

先从总体上来说：

- **从计算机底层来说：** 线程可以比作是轻量级的进程，是程序执行的最小单位,线程间的切换和调度的成本远远小于进程。另外，多核 CPU 时代意味着多个线程可以同时运行，这**减少了线程上下文切换的开销**。
- **从当代互联网发展趋势来说：** 现在的系统动不动就要求百万级甚至千万级的并发量，而多线程并发编程正是开发高并发系统的基础，利用好多线程机制可以大大**提高系统整体的并发能力以及性能**。

再深入到计算机底层来探讨：

- **单核时代：** 在单核时代多线程主要是为了提高 CPU 和 IO 设备的综合利用率。举个例子：当只有一个线程的时候会导致 CPU 计算时，IO 设备空闲；进行 IO 操作时，CPU 空闲。我们可以简单地说这两者的利用率目前都是 50%左右。但是当有两个线程的时候就不一样了，当一个线程执行 CPU 计算时，另外一个线程可以进行 IO 操作，这样两个的利用率就可以在理想情况下达到 100%了。
- **多核时代:** 多核时代多线程主要是为了提高 CPU 利用率。举个例子：假如我们要计算一个复杂的任务，我们只用一个线程的话，CPU 只会一个 CPU 核心被利用到，而创建多个线程就可以让多个 CPU 核心被利用到，这样就**提高了 CPU 的利用率**。



并发编程的目的就是为了能提高程序的执行效率提高程序运行速度，但是并发编程并不总是能提高程序运行速度的，而且并发编程可能会遇到很多问题，比如：**内存泄漏**、**上下文切换**、**死锁** 。



### 线程过多

使用多线程可以提升程序性能。但是如果使用过多的线程，则适得其反。

过多的线程会影响程序的系统。一方面，线程的启动和销毁，都是需要开销的。其次，过多的并发线程也会导致共享有限资源的开销增大。过多的线程，还会导致内存泄漏。

因此尽量使用线程池来管理线程，同时还需要设置恰当的线程数。



### 线程共享数据

1. 可以通过类变量直接将数据放到主存中
2. 通过并发的数据结构来存储数据
3. 使用volatile变量或者锁
4. 调用atomic类（如AtomicInteger）



## 死锁

线程死锁描述的是这样一种情况：多个线程同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于线程被无限期地阻塞，因此程序不可能正常终止。



### 死锁达成条件

死锁必须具备以下四个条件：

1. 互斥条件：该资源任意一个时刻只由一个线程占用。
2. 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件:线程已获得的资源在末使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
4. 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。



### 避免死锁

1. **破坏互斥条件** ：这个条件我们没有办法破坏，因为我们用锁本来就是想让他们互斥的（临界资源需要互斥访问）。
2. **破坏请求与保持条件** ：一次性申请所有的资源。
3. **破坏不剥夺条件** ：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源。
4. **破坏循环等待条件** ：靠按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。



### 活锁和死锁的区别

活锁和死锁类似，不同之处在于处于活锁的线程或进程的状态是不断改变的，活锁可以认为是一种特殊的饥饿。简单的说就是，活锁和死锁的主要区别是前者进程的状态可以改变但是却不能继续执行。



## 线程安全

如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。一个线程安全的计数器类的同一个实例对象在被多个线程使用的情况下也不会出现计算失误。很显然你可以将集合类分成两组，线程安全和非线程安全的。Vector 是用同步方法来实现线程安全的, 而和它相似的ArrayList不是线程安全的。



### 实现线程安全方式

- 使用synchronzied同步；
- 使用原子类(atomic concurrent classes)；
- 使用显示锁；
- 使用volatile关键字；
- 使用不变类；



## 扩展概念

### 竞态条件

在大多数实际的多线程应用中，两个或两个以上的线程需要共享对同一数据的存取。如果i线程存取相同的对象，并且每一个线程都调用了一个修改该对象状态的方法，将会发生什么呢？可以想象，线程彼此踩了对方的脚。根据线程访问数据的次序，可能会产生讹误的对象。这样的情况通常称为竞争条件。



### 阻塞式方法

阻塞式方法是指程序会一直等待该方法完成期间不做其他事情，ServerSocket的accept()方法就是一直等待客户端连接。这里的阻塞是指调用结果返回之前，当前线程会被挂起，直到得到结果之后才会返回。此外，还有异步和非阻塞式方法在任务完成前就返回。



### 线程调度器和时间分片

线程调度器（Thread Scheduler）是一个操作系统服务，它负责为Runnable状态的线程分配CPU时间。一旦我们创建一个线程并启动它，它的执行便依赖于线程调度器的实现。时间分片（Time Slicing）是指将可用的CPU时间分配给可用的Runnable线程的过程。分配CPU时间可以基于线程优先级或者线程等待的时间。线程调度并不受到Java虚拟机控制，所以由应用程序来控制它是更好的选择（也就是说不要让你的程序依赖于线程的优先级）。



### 上下文切换

#### CPU上下文

CPU 寄存器，是CPU内置的容量小、但速度极快的内存。而程序计数器，则是用来存储 CPU 正在执行的指令位置、或者即将执行的下一条指令位置。它们都是 CPU 在运行任何任务前，必须的依赖环境，因此叫做CPU上下文。



#### CPU上下文切换

它是指，先把前一个任务的CPU上下文（也就是CPU寄存器和程序计数器）保存起来，然后加载新任务的上下文到这些寄存器和程序计数器，最后再跳转到程序计数器所指的新位置，运行新任务。

一般我们说的**上下文切换**，就是指内核（操作系统的核心）**在CPU上对进程或者线程进行切换**。进程从用户态到内核态的转变，需要通过系统调用来完成。系统调用的过程，会发生CPU上下文的切换。

所以大家有时候会听到这种说法，**线程的上下文切换**。 它指，CPU资源的分配采用了**时间片轮转**，即给每个线程分配一个时间片，线程在时间片内占用 CPU 执行任务。当线程使用完时间片后，就会处于就绪状态并让出 CPU 让其他线程占用，这就是线程的上下文切换。看个图，可能会更容易理解一点

![图片](../../Image/2022/09/220922-15.jpg)

上下文切换是存储和恢复CPU状态的过程，它使得线程执行能够从中断点恢复执行。上下文切换是多任务操作系统和多线程环境的基本特征。

多线程编程中一般线程的个数都大于 CPU 核心的个数，而一个 CPU 核心在任意时刻只能被一个线程使用，为了让这些线程都能得到有效执行，CPU 采取的策略是为每个线程分配时间片并轮转的形式。当一个线程的时间片用完的时候就会重新处于就绪状态让给其他线程使用，这个过程就属于一次上下文切换。

概括来说就是：当前任务在执行完 CPU 时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换回这个任务时，可以再加载这个任务的状态。**任务从保存到再加载的过程就是一次上下文切换**。

上下文切换通常是计算密集型的。也就是说，它需要相当可观的处理器时间，在每秒几十上百次的切换中，每次切换都需要纳秒量级的时间。所以，上下文切换对系统来说意味着消耗大量的 CPU 时间，事实上，可能是操作系统中时间消耗最大的操作。

Linux 相比与其他操作系统（包括其他类 Unix 系统）有很多的优点，其中有一项就是，其上下文切换和模式切换的时间消耗非常少。



### 不可变对象

Immutable对象可以在没有同步的情况下共享，降低了对该对象进行并发访问时的同步化开销。要创建不可变类，要实现下面几个步骤：通过构造方法初始化所有成员、对变量不要提供setter方法、将所有的成员声明为私有的，这样就不允许直接访问这些成员、在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝。



### 忙循环

忙循环就是程序员用循环让一个线程等待，不像传统方法wait(), sleep() 或 yield() 它们都放弃了CPU控制，而忙循环不会放弃CPU，它就是在运行一个空循环。这么做的目的是为了保留CPU缓存，在多核系统中，一个等待线程醒来的时候可能会在另一个内核运行，这样会重建缓存。为了避免重建缓存和减少等待重建的时间就可以使用它了。



### 线程转储

线程转储是一个JVM活动线程的列表，它对于分析系统瓶颈和死锁非常有用。有很多方法可以获取线程转储——使用Profiler，Kill -3命令，jstack工具等等。我们更喜欢jstack工具，因为它容易使用并且是JDK自带的。由于它是一个基于终端的工具，所以我们可以编写一些脚本去定时的产生线程转储以待分析。



### 守护线程

当我们在Java程序中创建一个线程，它就被称为用户线程。一个守护线程是在后台执行并且不会阻止JVM终止的线程。当没有用户线程在运行的时候，JVM关闭程序并且退出。一个守护线程创建的子线程依然是守护线程。



### 伪共享

CPU的缓存是以缓存行(cache line)为单位进行缓存的，当多个线程修改相互独立的变量，而这些变量又处于同一个缓存行时就会影响彼此的性能，这就是伪共享。

![图片](../../Image/2022/09/220922-8.jpg)



CPU执行速度比内存速度快好几个数量级，为了提高执行效率，现代计算机模型演变出CPU、缓存（L1，L2，L3），内存的模型。

CPU执行运算时，如先从L1缓存查询数据，找不到再去L2缓存找，依次类推，直到在内存获取到数据。

为了避免频繁从内存获取数据，聪明的科学家设计出缓存行，缓存行大小为64字节。也正是因为**缓存行的存在**，就导致了伪共享问题，如图所示：

![图片](../../Image/2022/09/220922-9.jpg)

假设数据`a、b`被加载到同一个缓存行。

- 当线程1修改了a的值，这时候CPU1就会通知其他CPU核，当前缓存行（Cache line）已经失效。
- 这时候，如果线程2发起修改b，因为缓存行已经失效了，所以「core2 这时会重新从主内存中读取该 Cache line 数据」。读完后，因为它要修改b的值，那么CPU2就通知其他CPU核，当前缓存行（Cache line）又已经失效。
- 酱紫，如果同一个Cache line的内容被多个线程读写，就很容易产生相互竞争，频繁回写主内存，会大大降低性能。



#### 解决方式

既然伪共享是因为相互独立的变量存储到相同的Cache line导致的，一个缓存行大小是64字节。那么，我们就可以使用**空间换时间**的方法，即**数据填充的方式**，把独立的变量分散到不同的Cache line~



### **happens-before原则**

在Java语言中，有一个先行发生原则（`happens-before`）。它包括八大规则,如下：

- **程序次序规则**：在一个线程内，按照控制流顺序，书写在前面的操作先行发生于书写在后面的操作。
- **管程锁定规则**：一个unLock操作先行发生于后面对同一个锁额lock操作
- **volatile变量规则**：对一个变量的写操作先行发生于后面对这个变量的读操作
- **线程启动规则**：Thread对象的start()方法先行发生于此线程的每个一个动作
- **线程终止规则**：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
- **线程中断规则**：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- **对象终结规则**：一个对象的初始化完成先行发生于他的finalize()方法的开始
- **传递性**：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C



**happens-before其实本质是一种能确保线程及时刷新数据到内存，另一线程能实时从内存读取最新数据以保证数据在线程之间保持一致性的一种机制**。



### 线程调度策略

Java默认的线程调度算法是抢占式。即线程用完CPU之后，操作系统会根据线程优先级、线程饥饿情况等数据算出一个总的优先级并分配下一个时间片给某个线程执行。



#### 抢占式调度策略

Java运行时系统的线程调度算法是抢占式的 (preemptive)。Java运行时系统支持一种简单的固定优先级的调度算法。如果一个优先级比其他任何处于可运行状态的线程都高的线程进入就绪状态，那么运行时系统就会选择该线程运行。新的优先级较高的线程抢占(preempt)了其他线程。但是Java运行时系统并不抢占同优先级的线程。换句话说，Java运行时系统不是分时的(time-slice)。然而，基于Java Thread类的实现系统可能是支持分时的，因此编写代码时不要依赖分时。当系统中的处于就绪状态的线程都具有相同优先级时，线程调度程序采用一种简单的、非抢占式的轮转的调度顺序。

抢占式调度：优先让可运行池中优先级高的线程占用CPU，如果可运行池中的线程优先级相同，那么就随机选择一个线程，使其占用CPU。处于运行状态的线程会一直运行，直至它不得不放弃 CPU。



#### 时间片轮转调度策略

有些系统的线程调度采用时间片轮转(round-robin)调度策略。这种调度策略是从所有处于就绪状态的线程中选择优先级最高的线程分配一定的CPU时间运行。该时间过后再选择其他线程运行。只有当线程运行结束、放弃(yield)CPU或由于某种原因进入阻塞状态，低优先级的线程才有机会执行。如果有两个优先级相同的线程都在等待CPU，则调度程序以轮转的方式选择运行的线程。

分时调度模型：让所有的线程轮流获得cpu的使用权，并且平均分配每个线程占用的 CPU 的时间片。



# 关键字

## synchronized

### 概述

> **synchronized**要理解为**加锁**，而不是锁，这个思维有助于更好的理解线程同步。



**`synchronized` 关键字解决的是多个线程之间访问资源的同步性，保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。**

synchronized是Java中的关键字，是一种同步锁。synchronized关键字可以作用于方法或者代码块。



在 Java 早期版本中，`synchronized` 属于 **重量级锁**，效率低下。因为监视器锁（monitor）是依赖于底层的操作系统的 `Mutex Lock` 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。



 Java 6 之后 Java 官方对从 JVM 层面对 `synchronized` 较大优化，所以现在的 `synchronized` 锁效率也优化得很不错了。JDK1.6 对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。

所以目前不论是各种开源框架还是 JDK 源码都大量使用了 `synchronized` 关键字。



Synchronized是Java中解决并发问题的一种最常用的方法，也是最简单的一种方法。Synchronized的作用主要有三个：

1. 原子性：确保线程互斥的访问同步代码；
2. 可见性：保证共享变量的修改能够及时可见，其实是通过Java内存模型中的 “**对一个变量unlock操作之前，必须要同步到主内存中；如果对一个变量进行lock操作，则将会清空工作内存中此变量的值，在执行引擎使用此变量前，需要重新从主内存中load操作或assign操作初始化变量值**” 来保证的；
3. 有序性：有效解决重排序问题，即 “一个unlock操作先行发生(happen-before)于后面对同一个锁的lock操作”；





### 使用方法

从语法上讲，Synchronized可以把任何一个非null对象作为"锁"，在HotSpot JVM实现中，**锁有个专门的名字：对象监视器（Object Monitor）**。

Synchronized总共有三种用法：

1. 当synchronized作用在实例方法时，监视器锁（monitor）便是对象实例（this）；
2. 当synchronized作用在静态方法时，监视器锁（monitor）便是对象的Class实例，因为Class数据存在于永久代，因此静态方法锁相当于该类的一个全局锁；
3. 当synchronized作用在某一个对象实例时，监视器锁（monitor）便是括号括起来的对象实例；



注意，synchronized 内置锁 是一种 对象锁（锁的是对象而非引用变量），**作用粒度是对象 ，可以用来实现对 临界资源的同步互斥访问 ，是 可重入 的。其可重入最大的作用是避免死锁**，如**子类同步方法调用了父类同步方法，如没有可重入的特性，则会发生死锁。**



#### 修饰实例方法（方法锁）

作用于当前对象实例加锁，进入同步代码前要获得**当前对象实例的锁**。本质上还是对象锁。

```java
synchronized void method() {
  //业务代码
}
```



#### 修饰静态方法（类锁）

也就是给当前类加锁，会作用于类的所有对象实例 ，进入同步代码前要获得 **当前 class 的锁**。因为静态成员不属于任何一个实例对象，是类成员（static 表明这是该类的一个静态资源，不管 new 了多少个对象，只有一份）。所以，如果一个线程 A 调用一个实例对象的非静态 `synchronized` 方法，而线程 B 需要调用这个实例对象所属类的静态 `synchronized` 方法，是允许的，不会发生互斥现象，**因为访问静态 `synchronized` 方法占用的锁是当前类的锁，而访问非静态 `synchronized` 方法占用的锁是当前实例对象锁**。

```java
synchronized staic void method() {
  //业务代码
}
```



#### 修饰代码块（对象锁）

指定加锁对象，对给定对象/类加锁。`synchronized(this|object)` 表示进入同步代码库前要获得**给定对象的锁**。`synchronized(类.class)` 表示进入同步代码前要获得 **当前 class 的锁**。

```java
synchronized(this) {
  //业务代码
}
```



#### 使用方法总结

- `synchronized` 关键字加到 `static` 静态方法和 `synchronized(class)` 代码块上都是是给 Class 类上锁。
- `synchronized` 关键字加到实例方法上是给对象实例上锁。
- 尽量不要使用 `synchronized(String a)` 因为 JVM 中，字符串常量池具有缓存功能！



注意synchronized不能用在**类级别的(静态)代码块**，类级别的代码块在加载顺序上是要优先于任何方法的，其执行顺序只跟代码位置先后有关。不会发生竞争，自然不需要同步。

**构造方法不能使用 synchronized 关键字修饰。**构造方法本身就属于线程安全的，不存在同步的构造方法一说。



- 同时访问synchronized的静态和非静态方法，不能保证线程安全，两者的锁对象不一样。前者是类锁(XXX.class)，后者是this。

- 同时访问synchronized方法和非同步方法，不能保证线程安全，因为synchronized只会对被修饰的方法起作用。

- 个线程同时访问两个对象的非静态同步方法，不能保证线程安全，每个对象都拥有一把锁。两个对象相当于有两把锁，导致锁对象不一致。(PS：如果是类锁，则所有对象共用一把锁)



### 继承性

子类重写父类的synchronized的方法，主要分为两种情况：

- 子类的方法没有被synchronized修饰：synchronized的不具备继承性。所以子类方法是线程不安全的。

- 子类的方法被synchronized修饰：两个锁对象其实是一把锁，而且是**子类对象作为锁**。这也证明了: synchronized的锁是可重入锁。否则将出现死锁问题。



### 线程同步

两个线程间共享数据可以通过共享对象来实现这个目的，或者是使用像阻塞队列这样并发的数据结构。



#### 同步队列

- 当前线程想调用对象A的同步方法时，发现对象A的锁被别的线程占有，此时当前线程进入对象锁的同步队列。简言之，同步队列里面放的都是想争夺对象锁的线程。
- 当一个线程1被另外一个线程2唤醒时，1线程进入同步队列，去争夺对象锁。
- 同步队列是在同步的环境下才有的概念，一个对象对应一个同步队列。
- 线程等待时间到了或被notify/notifyAll唤醒后，会进入同步队列竞争锁，如果获得锁，进入RUNNABLE状态，否则进入BLOCKED状态等待获取锁。



调用obj的wait(), notify()方法前，必须获得obj锁，也就是必须写在synchronized(obj) 代码段内。与等待队列相关的步骤和图：

![img](../../Image/2022/08/220803-1.png)



1. 线程1获取对象A的锁，正在使用对象A。
2. 线程1调用对象A的wait()方法。
3. 线程1释放对象A的锁，并马上进入等待队列。
4. 锁池里面的对象争抢对象A的锁。
5. 线程5获得对象A的锁，进入synchronized块，使用对象A。
6. 线程5调用对象A的notifyAll()方法，唤醒所有线程，所有线程进入同步队列。若线程5调用对象A的notify()方法，则唤醒一个线程，不知道会唤醒谁，被唤醒的那个线程进入同步队列。
7. notifyAll()方法所在synchronized结束，线程5释放对象A的锁。
8. 同步队列的线程争抢对象锁，但线程1什么时候能抢到就不知道了。



### 同步概念

#### Java对象头

在JVM中**，对象在内存中的布局分为三块区域：对象头、实例数据和对齐填充。**如下图所示：

![img](../../Image/2022/08/220803-2.png)

1. 实例数据：存放类的属性数据信息，包括父类的属性信息；
2. 对齐填充：由于虚拟机要求 对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐；
3. **对象头：Java对象头一般占有2个机器码（在32位虚拟机中，1个机器码等于4字节，也就是32bit，在64位虚拟机中，1个机器码是8个字节，也就是64bit），但是 如果对象是数组类型，则需要3个机器码，因为JVM虚拟机可以通过Java对象的元数据信息确定Java对象的大小，但是无法从数组的元数据来确认数组的大小，所以用一块来记录数组长度。**



Synchronized用的锁就是存在Java对象头里的，那么什么是Java对象头呢？Hotspot虚拟机的对象头主要包括两部分数据：**Mark Word（标记字段）、**Class Pointer（类型指针）。其中 Class Pointer是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，Mark Word用于存储对象自身的运行时数据，它是实现轻量级锁和偏向锁的关键。 Java对象头具体结构描述如下：

![img](../../Image/2022/08/220803-3.png)



Java对象头结构组成

Mark Word用于存储对象自身的运行时数据，如：哈希码（HashCode）、GC分代年龄、**锁状态标志**、线程持有的锁、偏向线程 ID、偏向时间戳等。比如锁膨胀就是借助Mark Word的偏向的线程ID 参考：[JAVA锁的膨胀过程和优化(阿里)](https://www.cnblogs.com/aspirant/p/11705068.html) 阿里也经常问的问题

下图是Java对象头 无锁状态下Mark Word部分的存储结构（32位虚拟机）：

![img](../../Image/2022/08/220803-4.png)



Mark Word存储结构

对象头信息是与对象自身定义的数据无关的额外存储成本，但是考虑到虚拟机的空间效率，Mark Word被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据，它会根据对象的状态复用自己的存储空间，也就是说，Mark Word会随着程序的运行发生变化，可能变化为存储以下4种数据：

![img](../../Image/2022/08/220803-5.png)



Mark Word可能存储4种数据

在64位虚拟机下，Mark Word是64bit大小的，其存储结构如下：

![img](../../Image/2022/08/220803-6.png)



64位Mark Word存储结构

对象头的最后两位存储了锁的标志位，01是初始状态，未加锁，其对象头里存储的是对象本身的哈希码，随着锁级别的不同，对象头里会存储不同的内容。偏向锁存储的是当前占用此对象的线程ID；而轻量级则存储指向线程栈中锁记录的指针。从这里我们可以看到，“锁”这个东西，可能是个锁记录+对象头里的引用指针（判断线程是否拥有锁时将线程的锁记录地址和对象头里的指针地址比较)，也可能是对象头里的线程ID（判断线程是否拥有锁时将线程的ID和对象头里存储的线程ID比较）。

![img](../../Image/2022/08/220803-7.png)



#### 对象头中Mark Word与线程中Lock Record

在线程进入同步代码块的时候，如果此同步对象没有被锁定，即它的锁标志位是01，则虚拟机首先在当前线程的栈中创建我们称之为“锁记录（Lock Record）”的空间，用于存储锁对象的Mark Word的拷贝，官方把这个拷贝称为Displaced Mark Word。整个Mark Word及其拷贝至关重要。

**Lock Record是线程私有的数据结构**，每一个线程都有一个可用Lock Record列表，同时还有一个全局的可用列表。每一个被锁住的对象Mark Word都会和一个Lock Record关联（对象头的MarkWord中的Lock Word指向Lock Record的起始地址），同时Lock Record中有一个Owner字段存放拥有该锁的线程的唯一标识（或者`object mark word`），表示该锁被这个线程占用。如下图所示为Lock Record的内部结构：

| Lock Record | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| Owner       | 初始时为NULL表示当前没有任何线程拥有该monitor record，当线程成功拥有该锁后保存线程唯一标识，当锁被释放时又设置为NULL； |
| EntryQ      | 关联一个系统互斥锁（semaphore），阻塞所有试图锁住monitor record失败的线程； |
| RcThis      | 表示blocked或waiting在该monitor record上的所有线程的个数；   |
| Nest        | 用来实现 重入锁的计数；                                      |
| HashCode    | 保存从对象头拷贝过来的HashCode值（可能还包含GC age）。       |
| Candidate   | 用来避免不必要的阻塞或等待线程唤醒，因为每一次只有一个线程能够成功拥有锁，如果每次前一个释放锁的线程唤醒所有正在阻塞或等待的线程，会引起不必要的上下文切换（从阻塞到就绪然后因为竞争锁失败又被阻塞）从而导致性能严重下降。Candidate只有两种可能的值0表示没有需要唤醒的线程1表示要唤醒一个继任线程来竞争锁。 |



#### 监视器（Monitor）

任何一个对象都有一个Monitor与之关联，当且一个Monitor被持有后，它将处于锁定状态。Synchronized在JVM里的实现都是 基于进入和退出Monitor对象来实现方法同步和代码块同步，虽然具体实现细节不一样，但是都可以通过成对的MonitorEnter和MonitorExit指令来实现。

1. **MonitorEnter指令：插入在同步代码块的开始位置，当代码执行到该指令时，将会尝试获取该对象Monitor的所有权，即尝试获得该对象的锁；**
2. **MonitorExit指令：插入在方法结束处和异常处，JVM保证每个MonitorEnter必须有对应的MonitorExit；**

那什么是Monitor？可以把它理解为 一个同步工具，也可以描述为 一种同步机制，它通常被 描述为一个对象。

与一切皆对象一样，所有的Java对象是天生的Monitor，每一个Java对象都有成为Monitor的潜质，因为在Java的设计中 ，**每一个Java对象自打娘胎里出来就带了一把看不见的锁，它叫做内部锁或者Monitor锁**。

也就是通常说Synchronized的对象锁，MarkWord锁标识位为10，其中指针指向的是Monitor对象的起始地址。在Java虚拟机（HotSpot）中，Monitor是由ObjectMonitor实现的，其主要数据结构如下（位于HotSpot虚拟机源码ObjectMonitor.hpp文件，C++实现的）：



#### notify和notifyAll

obj.notify()唤醒在此对象监视器上等待的单个线程，选择是任意性的。notifyAll()唤醒在此对象监视器上等待的所有线程。

notify()方法不能唤醒某个具体的线程，所以只有一个线程在等待的时候它才有用武之地。而notifyAll()唤醒所有线程并允许他们争夺锁确保了至少有一个线程能继续运行。

当线程间是可以共享资源时，线程间通信是协调它们的重要的手段。Object类中wait()\notify()\notifyAll()方法可以用于线程间通信关于资源的锁的状态。

```c++
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; // 记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL; // 处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; // 处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```



ObjectMonitor中有两个队列，_WaitSet 和 _EntryList，用来保存ObjectWaiter对象列表（ 每个等待锁的线程都会被封装成ObjectWaiter对象 ），_owner指向持有ObjectMonitor对象的线程，当多个线程同时访问一段同步代码时：

1. 首先会进入 _EntryList 集合，当线程获取到对象的monitor后，进入 _Owner区域并把monitor中的owner变量设置为当前线程，同时monitor中的计数器count加1；
2. 若线程调用 wait() 方法，将释放当前持有的monitor，owner变量恢复为null，count自减1，同时该线程进入 WaitSet集合中等待被唤醒；
3. 若当前线程执行完毕，也将释放monitor（锁）并复位count的值，以便其他线程进入获取monitor(锁)；



同时**，Monitor对象存在于每个Java对象的对象头Mark Word中（存储的指针的指向），Synchronized锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因，同时notify/notifyAll/wait等方法会使用到Monitor锁对象，所以必须在同步代码块中使用。**

监视器Monitor有两种同步方式：互斥与协作。多线程环境下线程之间如果需要共享数据，需要解决互斥访问数据的问题，监视器可以确保监视器上的数据在同一时刻只会有一个线程在访问。



什么时候需要协作？ 比如一个线程向缓冲区写数据，另一个线程从缓冲区读数据，如果读线程发现缓冲区为空就会等待，当写线程向缓冲区写入数据，就会唤醒读线程，这里读线程和写线程就是一个合作关系。JVM通过Object类的wait方法来使自己等待，在调用wait方法后，该线程会释放它持有的监视器，直到其他线程通知它才有执行的机会。一个线程调用notify方法通知在等待的线程，这个等待的线程并不会马上执行，而是要通知线程释放监视器后，它重新获取监视器才有执行的机会。如果刚好唤醒的这个线程需要的监视器被其他线程抢占，那么这个线程会继续等待。Object类中的notifyAll方法可以解决这个问题，它可以唤醒所有等待的线程，总有一个线程执行。

![img](../../Image/2022/08/220803-8.png)



如上图所示，一个线程通过1号门进入Entry Set(入口区)，如果在入口区没有线程等待，那么这个线程就会获取监视器成为监视器的Owner，然后执行监视区域的代码。如果在入口区中有其它线程在等待，那么新来的线程也会和这些线程一起等待。线程在持有监视器的过程中，有两个选择，一个是正常执行监视器区域的代码，释放监视器，通过5号门退出监视器；还有可能等待某个条件的出现，于是它会通过3号门到Wait Set（等待区）休息，直到相应的条件满足后再通过4号门进入重新获取监视器再执行。

注意当一个线程释放监视器时，在入口区和等待区的等待线程都会去竞争监视器，如果入口区的线程赢了，会从2号门进入；如果等待区的线程赢了会从4号门进入。只有通过3号门才能进入等待区，在等待区中的线程只有通过4号门才能退出等待区，也就是说一个线程只有在持有监视器时才能执行wait操作，处于等待的线程只有再次获得监视器才能退出等待状态。



#####  为什么wait, notify 和 notifyAll这些方法不在thread类里面？

一个很明显的原因是JAVA提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。如果线程需要等待某些锁那么调用对象中的wait()方法就有意义了。如果wait()方法定义在Thread类中，线程正在等待的是哪个锁就不明显了。简单的说，由于wait，notify和notifyAll都是锁级别的操作，所以把他们定义在Object类中因为锁属于对象。



- 面向对象的角度：我们可以把wait和notify直接理解为get和set方法。wait和notify方法都是对对象的锁进行操作，那么自然这些方法应该属于对象。举例来说，门对象上有锁属性，开锁和关锁的方法应该属于门对象，而不应该属于人对象。
- 从观察者模式的角度：对象是被观察者，线程是观察者。被观察者的状态如果发生变化，理应有被观察者去轮询通知观察者，否则的话，观察者怎么知道notify方法应该在哪个时刻调用？n个观察者的notify又如何做到同时调用？



##### 为什么wait和notify方法要在同步块中调用？

当一个线程需要调用对象的wait()方法的时候，这个线程必须拥有该对象的锁，接着它就会释放这个对象锁并进入等待状态直到其他线程调用这个对象上的notify()方法。同样的，当一个线程需要调用对象的notify()方法时，它会释放这个对象的锁，以便其他在等待的线程就可以得到这个对象锁。由于所有的这些方法都需要线程持有对象的锁，这样就只能通过同步来实现，所以他们只能在同步方法或者同步块中被调用。如果你不这么做，代码会抛出IllegalMonitorStateException异常。



##### 为什么应该在循环中检查等待条件?

处于等待状态的线程可能会收到错误警报和伪唤醒，如果不在循环中检查等待条件，程序就会在没有满足结束条件的情况下退出。因此，当一个等待线程醒来时，不能认为它原来的等待状态仍然是有效的，在notify()方法调用之后和等待线程醒来之前这段时间它可能会改变。



### 底层原理

**synchronized的同步是在软件层面依赖JVM，而j.u.c.Lock的同步是在硬件层面依赖特殊的CPU指令。**



#### synchronized 同步语句块

```java
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("synchronized 代码块");
        }
    }
}
```



>通过 JDK 自带的 `javap` 命令查看 `SynchronizedDemo` 类的相关字节码信息：首先切换到类的对应目录执行 `javac SynchronizedDemo.java` 命令生成编译后的 .class 文件，然后执行`javap -c -s -v -l SynchronizedDemo.class`。



**`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。**

当执行 `monitorenter` 指令时，线程试图获取锁也就是获取 **对象监视器 `monitor`** 的持有权。

在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。

在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。



**monitorenter**：每个对象都是一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：

1. 如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者；
2. 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1；
3. 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权；



**monitorexit**：执行monitorexit的线程必须是objectref所对应的monitor的所有者。指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。

monitorexit指令出现了两次，第1次为同步正常退出释放锁；第2次为发生异步退出释放锁；



**Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。**



#### synchronized 修饰方法

```java
public class SynchronizedDemo2 {
    public synchronized void method() {
        System.out.println("synchronized 方法");
    }
}
```



`synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令，取得代之的确实是 `ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。JVM 通过该 `ACC_SYNCHRONIZED` 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。



当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。**在方法执行期间，其他任何线程都无法再获得同一个monitor对象。**



两种同步方式本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。两个指令的执行是JVM通过调用操作系统的互斥原语mutex来实现，被阻塞的线程会被挂起、等待重新调度，会导致“用户态和内核态”两个态之间来回切换，对性能有较大影响。



#### 底层原理总结

`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。

`synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令，取得代之的确实是 `ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。

**两者的本质都是对对象监视器 monitor 的获取。**



如果**synchronized**作用于**代码块**，反编译可以看到两个指令：`monitorenter、monitorexit`，JVM使用`monitorenter和monitorexit`两个指令实现同步；如果作用synchronized作用于**方法**,反编译可以看到`ACCSYNCHRONIZED标记`，JVM通过在方法访问标识符(flags)中加入`ACCSYNCHRONIZED`来实现同步功能。

- 同步代码块是通过`monitorenter和monitorexit`来实现，当线程执行到monitorenter的时候要先获得monitor锁，才能执行后面的方法。当线程执行到monitorexit的时候则要释放锁。
- 同步方法是通过中设置ACCSYNCHRONIZED标志来实现，当线程执行有ACCSYNCHRONI标志的方法，需要获得monitor锁。每个对象都与一个monitor相关联，线程可以占有或者释放monitor。



### Monitor

![图片](../../Image/2022/09/220922-4.jpg)

- 想要获取monitor的线程,首先会进入_EntryList队列。
- 当某个线程获取到对象的monitor后,进入Owner区域，设置为当前线程,同时计数器count加1。
- 如果线程调用了wait()方法，则会进入WaitSet队列。它会释放monitor锁，即将owner赋值为null,count自减1,进入WaitSet队列阻塞等待。
- 如果其他线程调用 notify() / notifyAll() ，会唤醒WaitSet中的某个线程，该线程再次尝试获取monitor锁，成功即进入Owner区域。
- 同步方法执行完毕了，线程退出临界区，会将monitor的owner设为null，并释放监视锁。



#### 对象与Monitor关联

![图片](../../Image/2022/09/220922-5.jpg)

- 在HotSpot虚拟机中,对象在内存中存储的布局可以分为3块区域：**对象头（Header），实例数据（Instance Data）和对象填充（Padding）**。
- 对象头主要包括两部分数据：**Mark Word（标记字段）、Class Pointer（类型指针）**。



Mark Word 是用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等。

![图片](../../Image/2022/09/220922-6.jpg)



**重量级锁，指向互斥量的指针。其实synchronized是重量级锁，也就是说Synchronized的对象锁，Mark Word锁标识位为10，其中指针指向的是Monitor对象的起始地址。**



### synchronized优化

在JDK1.6之前，synchronized的实现直接调用ObjectMonitor的enter和exit，这种锁被称之为重量级锁。从JDK6开始，HotSpot虚拟机开发团队对Java中的锁进行优化，如增加了**适应性自旋、锁消除、锁粗化、轻量级锁和偏向锁等优化策略**，提升了synchronized的性能。

- 偏向锁：在无竞争的情况下，只是在Mark Word里存储当前线程指针，CAS操作都不做。
- 轻量级锁：在没有多线程竞争时，相对重量级锁，减少操作系统互斥量带来的性能消耗。但是，如果存在锁竞争，除了互斥量本身开销，还额外有CAS操作的开销。
- 自旋锁：减少不必要的CPU上下文切换。在轻量级锁升级为重量级锁时，就使用了自旋加锁的方式
- 锁粗化：将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁。
- 锁消除：虚拟机即时编译器在运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除。



### 总结

1. 若是对象锁，则每个对象都持有一把自己的独一无二的锁，且对象之间的锁互不影响 。若是类锁，所有该类的对象共用这把锁。
2. 一个线程获取一把锁，没有得到锁的线程只能排队等待；
3. synchronized 是可重入锁，避免很多情况下的死锁发生。
4. synchronized 方法若发生异常，则JVM会自动释放锁，不会导致死锁。
5. 锁对象不能为空，否则抛出NPE(NullPointerException)
6. 同步本身是不具备继承性的：即父类的synchronized 方法，子类重写该方法,分情况讨论：没有synchonized修饰，则该子类方法不是线程同步的。(PS ：涉及同步继承性的问题要分情况)
7. synchronized本身修饰的范围越小越好。毕竟是同步阻塞。



# 最佳实践

## 实现线程

- 定义`Thread`类的子类，并重写该类的`run`方法
- 定义`Runnable`接口的实现类，并重写该接口的`run()`方法
- 定义`Callable`接口的实现类，并重写该接口的`call()`方法，一般配合`Future`使用
- 线程池的方式



### 继承Thread类

继承Thread类，然后重写run方法。由于Java单继承的特性，这种方式用的比较少。

```java
public class MyThread extends Thread {
	public MyThread() {
		
	}
	public void run() {
		for(int i=0;i<10;i++) {
			System.out.println(Thread.currentThread()+":"+i);
		}
	}
	public static void main(String[] args) {
		MyThread mThread1=new MyThread();
		MyThread mThread2=new MyThread();
		MyThread myThread3=new MyThread();
		mThread1.start();
		mThread2.start();
		myThread3.start();
	}
}

```



### 实现Runnable接口

实现Runable接口并实现run方法。

- 覆写Runnable接口实现多线程可以避免单继承局限；

- 实现Runnable()可以更好的体现共享的概念。

	

```java
public class MyTarget implements Runnable{
	public static int count=20;
	public void run() {
		while(count>0) {
			try {
				Thread.sleep(200);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(Thread.currentThread().getName()+"-当前剩余票数:"+count--);
		}
	}
	public static void main(String[] args) {
		MyThread target=new MyTarget();
		Thread mThread1=new Thread(target,"线程1");
		Thread mThread2=new Thread(target,"线程2");
		Thread mThread3=new Thread(target,"线程3");
		mThread1.start();
		mThread2.start();
		myThread3.start();
	}
}

```



### 实现Callable接口

实现Callable接口并实现call方法。

```java
public class MyTarget implements Callable<String> {
	private int count = 20;

	@Override
	public String call() throws Exception {
		for (int i = count; i > 0; i--) {
			System.out.println(Thread.currentThread().getName()+"当前票数：" + i);
		}
		return "sale out";
	} 

	public static void main(String[] args) throws InterruptedException, ExecutionException {
		Callable<String> callable  =new MyTarget();
		FutureTask <String>futureTask=new FutureTask<>(callable);
		Thread mThread=new Thread(futureTask);
		Thread mThread2=new Thread(futureTask);
		Thread mThread3=new Thread(futureTask);
		mThread.start();
		mThread2.start();
		mThread3.start();
		System.out.println(futureTask.get());
		
	}
}
```



#### 与Runnable对比

Runnable和Callable都代表那些要在不同的线程中执行的任务目标target。

Runnable从JDK1.0开始就有了，Callable是在JDK1.5增加的。它们的主要区别是Callable的 call() 方法可以返回值和抛出异常，而Runnable的run()方法没有这些功能。

- `Runnable`接口中的`run()`方法没有返回值，是`void`类型，它做的事情只是纯粹地去执行`run()`方法中的代码而已；
- `Callable`接口中的`call()`方法是有返回值的，是一个泛型。它一般配合`Future、FutureTask`一起使用，用来获取异步执行的结果。
- `Callable`接口`call()`方法允许抛出异常；而`Runnable`接口`run()`方法不能继续上抛异常；



### 通过线程池



## 线程顺序执行

在多线程中有多种方法让线程按特定顺序执行，你可以用线程类的join()方法在一个线程中启动另一个线程，另外一个线程完成该线程继续执行。为了确保三个线程的顺序你应该先启动最后一个(T3调用T2，T2调用T1)，这样T1就会先完成而T3最后完成。

```java
public class ThreadTest {
    public static void main(String[] args) {
        Thread spring = new Thread(new SeasonThreadTask("春天"));
        Thread summer = new Thread(new SeasonThreadTask("夏天"));
        Thread autumn = new Thread(new SeasonThreadTask("秋天"));

        try
        {
            //春天线程先启动
            spring.start();
            //主线程等待线程spring执行完，再往下执行
            spring.join();
            //夏天线程再启动
            summer.start();
            //主线程等待线程summer执行完，再往下执行
            summer.join();
            //秋天线程最后启动
            autumn.start();
            //主线程等待线程autumn执行完，再往下执行
            autumn.join();
        } catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }
}

class SeasonThreadTask implements Runnable{
    private String name;
    public SeasonThreadTask(String name){
        this.name = name;
    }
    @Override
    public void run() {
        for (int i = 1; i <4; i++) {
            System.out.println(this.name + "来了: " + i + "次");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```



## 异常处理

### 同步代码块

无论你的同步块是正常还是异常退出的，里面的线程都会释放锁，所以对比锁接口我们更喜欢同步块，因为它不用花费精力去释放锁，该功能可以在finally块里释放锁实现。



### Thread线程

Thread.UncaughtExceptionHandler是java SE5中的新接口，它允许我们在每一个Thread对象上添加一个异常处理器。



## 使用线程

**给线程起个有意义的名字**，这样可以方便找bug或追踪；

**避免锁定和缩小同步的范围**，锁花费的代价高昂且上下文切换更耗费时间空间，试试最低限度的使用同步和锁，缩小临界区。因此相对于同步方法我更喜欢同步块，它给我拥有对锁的绝对控制权。

**多用同步类少用wait 和 notify**，首先，CountDownLatch, Semaphore, CyclicBarrier 和 Exchanger 这些同步类简化了编码操作，而用wait和notify很难实现对复杂控制流的控制。其次，这些类是由最好的企业编写和维护在后续的JDK中它们还会不断优化和完善。

**多用并发集合少用同步集合**，这是另外一个容易遵循且受益巨大的最佳实践，并发集合比同步集合的可扩展性更好，所以在并发编程时使用并发集合效果更好。



# Thread

## 工作原理

### 常用方法

#### sleep

Thread.sleep(long millis)，一定是当前线程调用此方法，当前线程进入TIMED_WAITING状态，但不释放对象锁，millis后线程自动苏醒进入就绪状态。作用：给其它线程执行机会的最佳方式。



#### yield

Thread.yield()，一定是当前线程调用此方法，当前线程放弃获取的CPU时间片，但不释放锁资源，由运行状态变为就绪状态，让OS再次选择线程。作用：让相同优先级的线程轮流执行，但并不保证一定会轮流执行。实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。Thread.yield()不会导致阻塞。该方法与sleep()类似，只是不能由用户指定暂停多长时间。

yield方法可以暂停当前正在执行的线程对象，让其它有相同优先级的线程执行。它是一个静态方法而且只保证当前线程放弃CPU占用而不能保证使其它线程一定能占用CPU，执行yield()的线程有可能在进入到暂停状态后马上又被执行。



Thread类的sleep()和yield()方法将在当前正在执行的线程上运行。所以在其他处于等待状态的线程上调用这些方法是没有意义的。这就是为什么这些方法是静态的。它们可以在当前正在执行的线程中工作，并避免程序员错误的认为可以在其他非运行线程调用这些方法。



#### join

thread.join()/thread.join(long millis)，当前线程里调用其它线程t的join方法，当前线程进入WAITING/TIMED_WAITING状态，当前线程不会释放已经持有的对象锁。线程t执行完毕或者millis时间到，当前线程一般情况下进入RUNNABLE状态，也有可能进入BLOCKED状态（因为join是基于wait实现的）。



#### wait

obj.wait()，当前线程调用对象的wait()方法，当前线程释放对象锁，进入等待队列。依靠notify()/notifyAll()唤醒或者wait(long timeout) timeout时间到自动唤醒。



#### 其它

LockSupport.park()/LockSupport.parkNanos(long nanos),LockSupport.parkUntil(long deadlines), 当前线程进入WAITING/TIMED_WAITING状态。对比wait方法,不需要获得锁就可以让线程进入WAITING/TIMED_WAITING状态，需要通过LockSupport.unpark(Thread thread)唤醒。



### start和run的区别

start()方法被用来启动新创建的线程，使该被创建的线程状态变为可运行状态。

直接调用run()方法时，只会是在原来的线程中调用，没有新的线程启动。只有调用start()方法才会启动新线程。



- `start`方法可以启动一个新线程，`run`方法只是类的一个普通方法而已，如果直接调用`run`方法，程序中依然只有主线程这一个线程。
- `start`方法实现了多线程，而`run`方法没有实现多线程。
- `start`不能被重复调用，而`run`方法可以。
- `start`方法中的`run`代码可以不执行完，就继续执行下面的代码，也就是说进行了**线程切换**。然而，如果直接调用`run`方法，就必须等待其代码全部执行完才能继续执行下面的代码。



### sleep和wait的区别

- 两者最主要的区别在于：**`sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁** 。
- 两者都可以暂停线程的执行。
- `wait()` 通常被用于线程间交互/通信，`sleep()`通常被用于暂停执行。
- `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify()`或者 `notifyAll()` 方法。`sleep()`方法执行完成后，线程会自动苏醒。或者可以使用 `wait(long timeout)` 超时后线程会自动苏醒。



Java程序中wait 和 sleep都会造成某种形式的暂停，它们可以满足不同的需要。wait()方法用于线程间通信，如果等待条件为真且其它线程被唤醒时它会释放锁，而sleep()方法仅仅释放CPU资源或者让当前线程停止执行一段时间，但不会释放锁。需要注意的是，sleep（）并不会让线程终止，一旦从休眠中唤醒线程，线程的状态将会被改变为Runnable，并且根据线程调度，它将得到执行。



### interrupted 和 isInterruptedd的区别

interrupted() 和 isInterrupted()的主要区别是前者会将中断状态清除而后者不会。Java多线程的中断机制是用内部标识来实现的，调用Thread.interrupt()来中断一个线程就会设置中断标识为true。当中断线程调用静态方法Thread.interrupted()来检查中断状态时，中断状态会被清零。而非静态方法isInterrupted()用来查询其它线程的中断状态且不会改变中断状态标识。简单的说就是任何抛出InterruptedException异常的方法都会将中断状态清零。无论如何，一个线程的中断状态有有可能被其它线程调用中断来改变。



### wait(),notify()和suspend(),resume()之间的区别

- `wait()`方法使得线程进入阻塞等待状态，并且释放锁
- `notify()`唤醒一个处于等待状态的线程，它一般跟`wait（）`方法配套使用。
- `suspend()`使得线程进入阻塞状态，并且不会自动恢复，必须对应的`resume()`被调用，才能使得线程重新进入可执行状态。`suspend()`方法很容易引起死锁问题。
- `resume()`方法跟`suspend()`方法配套使用。



### 停止线程

Java提供了很丰富的API但没有为停止线程提供API。JDK 1.0本来有一些像stop(), suspend() 和 resume()的控制方法，但是由于潜在的死锁威胁。因此在后续的JDK版本中他们被弃用了，之后Java API的设计者就没有提供一个兼容且线程安全的方法来停止一个线程。当run() 或者 call() 方法执行完的时候线程会自动结束，如果要手动结束一个线程，可以用volatile 布尔变量来退出run()方法的循环或者是取消任务来中断线程。



## 调度方法

| 描述     | 方法                                         |
| -------- | -------------------------------------------- |
| 休眠     | Thread.sleep(long)                           |
| 中断     | thread.interrupt()<br />thread.isInterrupt() |
| 等待     | object.wait()<br />object.wait(long)         |
| 线程让步 | Thread.yield()                               |
| 通知     | object.notify()<br />object.notifyAll()      |



### 休眠

`Thread.sleep(long)`方法，使线程转到**超时等待阻塞（TIMED_WAITING）** 状态。`long`参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，线程自动转为`就绪（Runnable）`状态。



### 中断

`interrupt()`表示中断线程。需要注意的是，`InterruptedException`是线程自己从内部抛出的，并不是`interrupt()`方法抛出的。对某一线程调用`interrupt()`时，如果该线程正在执行普通的代码，那么该线程根本就不会抛出`InterruptedException`。但是，一旦该线程进入到`wait()/sleep()/join()`后，就会立刻抛出InterruptedException。可以用`isInterrupted()`来获取状态。



### 等待

`Object`类中的`wait()`方法，会导致当前的线程等待，直到其他线程调用此对象的`notify()`方法或`notifyAll()`唤醒方法。



### 线程让步

`Thread.yield()`方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。



### 通知

Object的`notify()`方法，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。

`notifyAll()`，则是唤醒在此对象监视器上等待的所有线程。



## 线程通讯方式

### volatile和synchronized关键字

volatile关键字用来修饰共享变量，保证了共享变量的可见性，任何线程需要读取时都要到内存中读取（确保获得最新值）。

synchronized关键字确保只能同时有一个线程访问方法或者变量，保证了线程访问的可见性和排他性。



### 等待/通知机制

等待/通知机制，是指一个线程A调用了对象O的wait()方法进入等待状态，而另一个线程B 调用了对象O的notify()或者notifyAll()方法，线程A收到通知后从对象O的wait()方法返回，进而 执行后续操作。



### 管道输入/输出流

管道输入/输出流和普通的文件输入/输出流或者网络输入/输出流不同之处在于，它主要 用于线程之间的数据传输，而传输的媒介为内存。

管道输入/输出流主要包括了如下4种具体实现：PipedOutputStream、PipedInputStream、 PipedReader和PipedWriter，前两种面向字节，而后两种面向字符。



### join()方法

如果一个线程A执行了thread.join()语句，其含义是：当前线程A等待thread线程终止之后才 从thread.join()返回。线程Thread除了提供join()方法之外，还提供了join(long millis)和join(long millis,int nanos)两个具备超时特性的方法。这两个超时方法表示，如果线程thread在给定的超时 时间里没有终止，那么将会从该超时方法中返回。



### ThreadLocal

ThreadLocal，即线程本地变量（每个线程都有自己唯一的一个哦），是一个以ThreadLocal对象为键、任意对象为值的存储结构。底层是一个ThreadLocalMap来存储信息，key是弱引用，value是强引用，所以使用完毕后要及时清理(尤其使用线程池时)。



## 源码解析

### 属性

> java.lang.Thread

```java
    /* 
     * 与此线程有关的 ThreadLocal 值。此映射由 ThreadLocal 类维护。 
     */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * 与此线程有关的 InheritableThreadLocal 值。
     * 该映射由 InheritableThreadLocal 类维护。
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```



- threadLocals

该变量默认为null，只有调用 ThreadLoacl 的 get 和 set 方法时才会创建。



### run

> java.lang.Thread

```java
    /**
     * 如果该线程是使用单独的 Runnable 运行对象构造的，则调用该 Runnable对象的 run 方法；
     * 否则，此方法不执行任何操作并返回。
     */
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```



### start

```java
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }

    private native void start0();

    /**
     * Notifies the group that the thread {@code t} has failed
     * an attempt to start.
     *
     * <p> The state of this thread group is rolled back as if the
     * attempt to start the thread has never occurred. The thread is again
     * considered an unstarted member of the thread group, and a subsequent
     * attempt to start the thread is permitted.
     *
     * @param  t
     *         the Thread whose start method was invoked
     */
    void threadStartFailed(Thread t) {
        synchronized(this) {
            remove(t);
            nUnstartedThreads++;
        }
    }

    /**
     * 从此组中删除指定的线程。在已销毁的线程组上调用此方法无效。
     */
    private void remove(Thread t) {
        synchronized (this) {
            if (destroyed) {
                return;
            }
            for (int i = 0 ; i < nthreads ; i++) {
                if (threads[i] == t) {
                    System.arraycopy(threads, i + 1, threads, i, --nthreads - i);
                    // 对死线程的空引用，以便垃圾收集器收集它。
                    threads[nthreads] = null;
                    break;
                }
            }
        }
    }
```



new 一个 Thread，线程进入了新建状态。调用 `start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。 但是，直接执行 `run()` 方法，会把 `run()` 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

**总结： 调用 `start()` 方法方可启动线程并使线程进入就绪状态，直接执行 `run()` 方法的话不会以多线程的方式执行。**



### sleep

> java.lang.Thread

```java
    /**
     * 使当前执行的线程休眠（暂时停止执行）指定的毫秒数，
     * 取决于系统计时器和调度程序的精度和准确性。该线程不会失去任何监视器的所有权。
     */
    public static native void sleep(long millis) throws InterruptedException;

    /**
     * 使当前执行的线程休眠（暂时停止执行）指定的毫秒数加上指定的纳秒数，
     * 具体取决于系统计时器和调度程序的精度和准确性。该线程不会失去任何监视器的所有权。
     */
    public static void sleep(long millis, int nanos)
    throws InterruptedException {
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }

        sleep(millis);
    }
```



### join

> java.lang.Thread

```java
    public final void join() throws InterruptedException {
        join(0);
    }

    public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }

    public final synchronized void join(long millis, int nanos)
    throws InterruptedException {

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }

        join(millis);
    }
```



### yield

> java.lang.Thread

```java
    public static native void yield();
```



### interrupt

`interrupt` 它是真正触发中断的方法。



> java.lang.Thread

```java
    public void interrupt() {
        if (this != Thread.currentThread())
            checkAccess();

        synchronized (blockerLock) {
            Interruptible b = blocker;
            if (b != null) {
                interrupt0();           // Just to set the interrupt flag
                b.interrupt(this);
                return;
            }
        }
        interrupt0();
    }
```



### interrupted

`interrupted`是Thread中的一个类方法,它也调用了isInterrupted(true)方法，不过它传递的参数是true，表示将会清除中断标志位。



> java.lang.Thread

```java
    public static boolean interrupted() {
        return currentThread().isInterrupted(true);
    }

    private native boolean isInterrupted(boolean ClearInterrupted);
```



### isInterruptedd

`isInterrupted`是`Thread`类中的一个实例方法，可以判断实例线程是否被中断。



> java.lang.Thread

```java
    public boolean isInterrupted() {
        return isInterrupted(false);
    }
```



### holdsLock

> java.lang.Thread

```java
    public static native boolean holdsLock(Object obj);
```



返回true如果当且仅当当前线程拥有某个具体对象的锁。可以通过这个方法来判断线程是否拥有锁。



### setPriority

> java.lang.Thread

```java
    public final void setPriority(int newPriority) {
        ThreadGroup g;
        checkAccess();
        if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
            throw new IllegalArgumentException();
        }
        if((g = getThreadGroup()) != null) {
            if (newPriority > g.getMaxPriority()) {
                newPriority = g.getMaxPriority();
            }
            setPriority0(priority = newPriority);
        }
    }
```



### setDaemon

> java.lang.Thread

```java
    public final void setDaemon(boolean on) {
        checkAccess();
        if (isAlive()) {
            throw new IllegalThreadStateException();
        }
        daemon = on;
    }
```



使用Thread类的setDaemon(true)方法可以将线程设置为守护线程，需要注意的是，需要在调用start()方法前调用这个方法，否则会抛出IllegalThreadStateException异常。



### 异常相关

#### 获取未捕获异常

> java.lang.Thread

```java
private volatile UncaughtExceptionHandler uncaughtExceptionHandler;

public UncaughtExceptionHandler getUncaughtExceptionHandler() {
    return uncaughtExceptionHandler != null ?
        uncaughtExceptionHandler : group;
}
```



> java.lang.Thread.UncaughtExceptionHandler

```java
@FunctionalInterface
public interface UncaughtExceptionHandler {
    /**
     * 处理未捕获异常
     */
    void uncaughtException(Thread t, Throwable e);
}
```



如果异常没有被捕获该线程将会停止执行。Thread.UncaughtExceptionHandler是用于处理未捕获异常造成线程突然中断情况的一个内嵌接口。

当一个未捕获异常将造成线程中断的时候，JVM会使用Thread.getUncaughtExceptionHandler()来查询线程的UncaughtExceptionHandler并将线程和异常作为参数传递给handler的uncaughtException()方法进行处理。



# Runnable

Runnable 接口不会返回结果或抛出检查异常，但是Callable 接口可以。所以，如果任务不需要返回结果或抛出异常推荐使用 Runnable 接口。



## 互相转换

## 源码解析

> java.lang.Runnable

```java
@FunctionalInterface
public interface Runnable {
    /**
     * 当使用实现接口Runnable的对象来创建线程时，启动线程会导致在该单独执行的线程中调用对象的run方法。
     */
    public abstract void run();
}
```



### 和Thread的关系

> java.lang.Thread

```java
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}

private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    // ...
    this.target = target;
    // ...
}

@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```



Runnable接口实现对象可以作为Thread的构造器参数，Thread对象调用start方法时实际执行的就是Runnable接口实现对象的run方法。



# ThreadLocal

## 实现原理

- `Thread`线程类有一个类型为`ThreadLocal.ThreadLocalMap`的实例变量`threadLocals`，即每个线程都有一个属于自己的`ThreadLocalMap`。
- `ThreadLocalMap`内部维护着`Entry`数组，每个`Entry`代表一个完整的对象，`key`是`ThreadLocal`本身，`value`是`ThreadLocal`的泛型值。
- 并发多线程场景下，每个线程`Thread`，在往`ThreadLocal`里设置值的时候，都是往自己的`ThreadLocalMap`里存，读也是以某个`ThreadLocal`作为引用，在自己的`map`里找对应的`key`，从而可以实现了**线程隔离**。



### ThreadLocalMap的key

一个使用类，有两个共享变量，也就是说用了两个`ThreadLocal`成员变量的话。如果用线程`id`作为`ThreadLocalMap`的`key`，无法区分哪个`ThreadLocal`成员变量。因此还是需要使用`ThreadLocal`作为`Key`来使用。每个`ThreadLocal`对象，都可以由`threadLocalHashCode`属性**唯一区分**的，每一个ThreadLocal对象都可以由这个对象的名字唯一区分。



### 内存泄漏

![图片](../../Image/2022/09/220922-11.jpg)

`ThreadLocalMap`使用`ThreadLocal`的**弱引用**作为`key`，当`ThreadLocal`变量被手动设置为`null`，即一个`ThreadLocal`没有外部强引用来引用它，当系统GC时，`ThreadLocal`一定会被回收。这样的话，`ThreadLocalMap`中就会出现`key`为`null`的`Entry`，就没有办法访问这些`key`为`null`的`Entry`的`value`，如果当前线程再迟迟不结束的话(比如线程池的核心线程)，这些`key`为`null`的`Entry`的`value`就会一直存在一条强引用链：Thread变量 -> Thread对象 -> ThreaLocalMap -> Entry -> value -> Object 永远无法回收，造成内存泄漏。

当`ThreadLocal`变量被手动设置为`null`后的引用链图：

![图片](../../Image/2022/09/220922-12.jpg)

`ThreadLocalMap`的设计中已经考虑到这种情况。所以也加上了一些防护措施：即在`ThreadLocal`的`get`,`set`,`remove`方法，都会清除线程`ThreadLocalMap`里所有`key`为`null`的`value`。



## ThreadLocal
### 概述
**`ThreadLocal`类主要解决的就是让每个线程绑定自己的值。**

**访问`ThreadLocal`变量的每个线程都会有这个变量的本地副本，可以使用 `get（）` 和 `set（）` 方法来获取默认值或将其值更改为当前线程所存的副本的值，从而避免了线程安全问题。**

ThreadLocal是Java里一种特殊的变量。每个线程都有一个ThreadLocal就是每个线程都拥有了自己独立的一个变量，竞争条件被彻底消除了。如果为每个线程提供一个自己独有的变量拷贝，将大大提高效率。首先，通过复用减少了代价高昂的对象的创建个数。其次，你在没有使用高代价的同步或者不变性的情况下获得了线程安全。



ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。同步机制采用了“时间换空间”的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。而ThreadLocal采用了“空间换时间”的方式。

ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。



### 使用场景

讨论ThreadLocal用在什么地方前，我们先明确下，如果仅仅就一个线程，那么都不用谈ThreadLocal的，ThreadLocal是用在多线程的场景的！！！

ThreadLocal归纳下来就2类用途：

- **保存线程上下文信息，在任意需要的地方可以获取！！！**
- **线程安全的，避免某些情况需要考虑线程安全必须同步带来的性能损失！！！**



#### 保存线程上下文信息

由于ThreadLocal的特性，同一线程在某地方进行设置，在随后的任意地方都可以获取到。从而可以用来保存线程上下文信息。

常用的比如每个请求怎么把一串后续关联起来，就可以用ThreadLocal进行set，在后续的任意需要记录日志的方法里面进行get获取到请求id，从而把整个请求串起来。

还有比如Spring的事务管理，用ThreadLocal存储Connection，从而各个DAO可以获取同一Connection，可以进行事务回滚，提交等操作。

备注： ThreadLocal的这种用处，很多时候是用在一些优秀的框架里面的，一般我们很少接触，反而下面的场景我们接触的更多一些！



#### 线程安全的

ThreadLocal为解决多线程程序的并发问题提供了一种新的思路。但是ThreadLocal也有局限性。

ThreadLocal无法解决共享对象更新问题，ThreadLocal对象建议用static修饰。这个变量是针对一个线程内所有操作共享的，所以设置为静态变量，所有此类实例共享此静态变量，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象（只要是这个线程内定义的）都可以操控这个变量。

这里把ThreadLocal定义为static还有一个好处就是，由于ThreadLocal有强引用在，那么在ThreadLocalMap里对应的Entry的键会永远存在，那么执行remove的时候就可以正确进行定位到并且删除！！！



每个线程往ThreadLocal中读写数据是线程隔离，互相之间不会影响的，所以**ThreadLocal无法解决共享对象的更新问题！**

由于不需要共享信息，自然就不存在竞争问题了，从而保证了某些情况下线程的安全，以及避免了某些情况需要考虑线程安全必须同步带来的性能损失！！！



### 原理

![img](../../Image/2022/08/220803-17.png)



#### threadLocalHashCode属性
通过属性threadLocalHashCode和容量长度len的进行位运算来获取在table中的下标位置。



#### nextHashCode属性
因为nextHashCode属性是static的原因，在每次new ThreadLocal时因为threadLocalHashCode的初始化，会使threadLocalHashCode值自增一次，增量为0x61c88647。



#### HASH_INCREMENT常量
HASH_INCREMENT常量值为0x61c88647。

0x61c88647是斐波那契散列乘数,它的优点是通过它散列(hash)出来的结果分布会比较均匀，可以很大程度上避免hash冲突，已初始容量16为例，hash并与15位运算计算数组下标结果如下：

ThreadLocalMap使用的是线性探测法，均匀分布的好处在于很快就能探测到下一个临近的可用slot，从而保证效率。。

|hashCode	|数组下标    |
| ----      | ----      |
|0x61c88647	|7          |
|0xc3910c8e	|14         |
|0x255992d5	|5          |
|0x8722191c	|12         |
|0xe8ea9f63	|3          |
|0x4ab325aa	|10         |
|0xac7babf1	|1          |
|0xe443238	|8          |
|0x700cb87f	|15         |

总结如下：
- 对于某一ThreadLocal来讲，他的索引值i是确定的，在不同线程之间访问时访问的是不同的table数组的同一位置即都为table[i]，只不过这个不同线程之间的table是独立的。
- 对于同一线程的不同ThreadLocal来讲，这些ThreadLocal实例共享一个table数组，然后每个ThreadLocal实例在table中的索引i是不同的。



### 总结

#### 和synchronized比较

`ThreadLocal`和`Synchronized`都是为了解决多线程中相同变量的访问冲突问题，不同的点是

- `Synchronized`是通过线程等待，牺牲时间来解决访问冲突。
- `ThreadLocal`是通过每个线程单独一份存储空间，牺牲空间来解决冲突，并且相比于`Synchronized`，`ThreadLocal`具有线程隔离的效果，只有在线程内才能获取到对应的值，线程外则不能访问到想要的值。

正因为`ThreadLocal`的线程隔离特性，使他的应用场景相对来说更为特殊一些。在android中Looper、ActivityThread以及AMS中都用到了`ThreadLocal`。当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用`ThreadLocal`。



#### ThreadLocal与内存泄漏

`ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用,而 value 是强引用。所以，如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来，`ThreadLocalMap` 中就会出现 key 为 null 的 Entry。假如我们不做任何措施的话，value 永远无法被 GC 回收，这个时候就可能会产生内存泄露。

ThreadLocalMap 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。使用ThreadLocal的过程中，显式地进行调用remove方法，将弱引用的key与强引用的value一起删除，避免引起内存泄漏。



### 源码解析
#### 基础结构
> java.lang.ThreadLocal
```java
public class ThreadLocal<T> {
   
    private final int threadLocalHashCode = nextHashCode();

    private static AtomicInteger nextHashCode =
        new AtomicInteger();
    
    private static final int HASH_INCREMENT = 0x61c88647;
    
    private static int nextHashCode() {
        // HASH_INCREMENT可以让生成出来的值或者说ThreadLocal的ID较为均匀地分布在2的幂大小的数组中
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }

    public ThreadLocal() {
    }
```



#### set

> java.lang.ThreadLocal
```java
public class ThreadLocal<T> {
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    
    ThreadLocalMap getMap(Thread t) {
        // 获取当前Thread类维护的ThreadLocalMap类型属性threadLocals
        return t.threadLocals;
    }

    void createMap(Thread t, T firstValue) {
        // 创建ThreadLocalMap对象并复制给当前Thread对象的属性threadLocals
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```



线程本地变量实质上是存放在了 Thread 类的 threadLocals 属性中，ThreadLocal 可以理解为 ThreadLocalMap 的封装。

每个 Thread 中都具备一个ThreadLocalMap，而ThreadLocalMap可以存储以ThreadLocal为 key ，Object 对象为 value 的键值对。

在同一个线程中声明了两个 ThreadLocal 对象的话，会使用 Thread内部都是使用仅有那个ThreadLocalMap 存放数据的，ThreadLocalMap的 key 就是 ThreadLocal对象，value 就是 ThreadLocal 对象调用set方法设置的值。



#### get

> java.lang.ThreadLocal
```java
public class ThreadLocal<T> {
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        // 设置初始化值并返回
        return setInitialValue();
    }

    private T setInitialValue() {
        // 获取初始化值，默认为null
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }    

    protected T initialValue() {
        return null;
    }
}
```



#### remove

> java.lang.ThreadLocal
```java
public class ThreadLocal<T> {
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }        
}
```



Entry的key指向ThreadLocal是弱引用。弱引用也是用来描述非必需对象的，当JVM进行垃圾回收时，无论内存是否充足，**该对象仅仅被弱引用关联**，那么就会被回收。

**当仅仅只有ThreadLocalMap中的Entry的key指向ThreadLocal的时候，ThreadLocal会进行回收的**。

ThreadLocal被垃圾回收后，在ThreadLocalMap里对应的Entry的键值会变成null，但是Entry是强引用，那么Entry里面存储的Object，并没有办法进行回收，所以ThreadLocalMap 做了一些额外的回收工作。



由于线程的生命周期很长，如果我们往ThreadLocal里面set了很大很大的Object对象，虽然set、get等等方法在特定的条件会调用进行额外的清理，但是**ThreadLocal被垃圾回收后，在ThreadLocalMap里对应的Entry的键值会变成null，但是后续在也没有操作set、get等方法了。**

**所以最佳实践，应该在不使用的时候，主动调用remove方法进行清理。**



## SuppliedThreadLocal

### 概述
`SuppliedThreadLocal`继承了`ThreadLocal`类，对其功能提供了扩展，可以通过函数式接口`Supplier`来设置默认的初始化值。



### 源码解析

> java.lang.ThreadLocal.SuppliedThreadLocal
```java
static final class SuppliedThreadLocal<T> extends ThreadLocal<T> {

    private final Supplier<? extends T> supplier;

    SuppliedThreadLocal(Supplier<? extends T> supplier) {
        this.supplier = Objects.requireNonNull(supplier);
    }

    @Override
    protected T initialValue() {
        return supplier.get();
    }
}
```



## ThreadLocalMap

### 概述
ThreadLocalMap是定义于ThreadLocal类中的默认静太类，没有对外暴露任何public方法，因此只能有ThreadLocal内进行调用。

Thread类有属性变量threadLocals （类型是ThreadLocal.ThreadLocalMap），也就是说每个线程有一个自己的ThreadLocalMap ，所以每个线程往这个ThreadLocal中读写隔离的，并且是互相不会影响的。



### 实现原理

#### key

> To help deal with very large and long-lived usages, the hash table entries use WeakReferences **for** keys.

ThreadLocal的key虽然是弱引用，但是有ThreadLocal变量进行引用，因此不会被GC进行回收。除非手动将ThreadLocal变量设置为null。



**key的弱引用**

- 如果`Key`使用强引用：当`ThreadLocal`的对象被回收了，但是`ThreadLocalMap`还持有`ThreadLocal`的强引用的话，如果没有手动删除，ThreadLocal就不会被回收，会出现Entry的内存泄漏问题。
- 如果`Key`使用弱引用：当`ThreadLocal`的对象被回收了，因为`ThreadLocalMap`持有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。`value`则在下一次`ThreadLocalMap`调用`set,get，remove`的时候会被清除。



使用弱引用作为`Entry`的`Key`，可以多一层保障：弱引用`ThreadLocal`不会轻易内存泄漏，对应的`value`在下一次`ThreadLocalMap`调用`set,get,remove`的时候会被清除。

内存泄漏的根本原因是，不再被使用的`Entry`，没有从线程的`ThreadLocalMap`中删除。一般删除不再使用的`Entry`有这两种方式：

- 一种就是，使用完`ThreadLocal`，手动调用`remove()`，把`Entry从ThreadLocalMap`中删除
- 另外一种方式就是：`ThreadLocalMap`的自动清除机制去清除过期`Entry`.（`ThreadLocalMap`的`get(),set()`时都会触发对过期`Entry`的清除）



### 源码解析

#### 基础结构
> java.lang.ThreadLocal.ThreadLocalMap
```java
static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;
        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    /**
     * 初始化容器容量
     */
    private static final int INITIAL_CAPACITY = 16;

    private Entry[] table;

    /**
     * table中元素的数量
     */
    private int size = 0;

    /**
     * 当size >= threshold时，遍历table并删除key为null的元素，如果删除后size >= threshold*3/4时，需要对table进行扩容。
     */
    private int threshold; // Default to 0

    private void setThreshold(int len) {
        // threshold是table大小的2/3
        threshold = len * 2 / 3;
    }

    private static int nextIndex(int i, int len) {
        return ((i + 1 < len) ? i + 1 : 0);
    }

    private static int prevIndex(int i, int len) {
        return ((i - 1 >= 0) ? i - 1 : len - 1);
    }

    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        // 定义一个长度为16的Entry数组table
        table = new Entry[INITIAL_CAPACITY];
        // 位运算，获取存放下标位置
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }

    private ThreadLocalMap(ThreadLocalMap parentMap) {
        Entry[] parentTable = parentMap.table;
        int len = parentTable.length;
        setThreshold(len);
        table = new Entry[len];

        for (int j = 0; j < len; j++) {
            Entry e = parentTable[j];
            if (e != null) {
                @SuppressWarnings("unchecked")
                ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
                if (key != null) {
                    Object value = key.childValue(e.value);
                    Entry c = new Entry(key, value);
                    int h = key.threadLocalHashCode & (len - 1);
                    while (table[h] != null)
                        h = nextIndex(h, len);
                    table[h] = c;
                    size++;
                }
            }
        }
    }
}
```



#### set

> java.lang.ThreadLocal.ThreadLocalMap
```java
static class ThreadLocalMap {
    private void set(ThreadLocal<?> key, Object value) {

        // We don't use a fast path as with get() because it is at
        // least as common to use set() to create new entries as
        // it is to replace existing ones, in which case, a fast
        // path would fail more often than not.

        Entry[] tab = table;
        int len = tab.length;
        int i = key.threadLocalHashCode & (len-1);

        for (Entry e = tab[i];
             e != null;
             e = tab[i = nextIndex(i, len)]) {
            ThreadLocal<?> k = e.get();

            if (k == key) {
                // 当前线程key存在，设置value值并返回
                e.value = value;
                return;
            }

            if (k == null) {
                // 如果k为null，表示这个位置的value已经是陈旧的元素，将该旧元素替换
                replaceStaleEntry(key, value, i);
                return;
            }
        }
        
        // 当前下标位置没有Entry对象，则创建Entry对象并设置到当前下标位置    
        tab[i] = new Entry(key, value);
        int sz = ++size;
        if (!cleanSomeSlots(i, sz) && sz >= threshold)
            // 满足没有清除元素且元素数量大于设置的threshold条件
            rehash();
    }

    private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                                   int staleSlot) {
        // 删除所有陈旧元素并设置新元素
        Entry[] tab = table;
        int len = tab.length;
        Entry e;

        int slotToExpunge = staleSlot;
        for (int i = prevIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = prevIndex(i, len))
            if (e.get() == null)
                slotToExpunge = i;

        for (int i = nextIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = nextIndex(i, len)) {
            ThreadLocal<?> k = e.get();

            if (k == key) {
                e.value = value;

                tab[i] = tab[staleSlot];
                tab[staleSlot] = e;

                if (slotToExpunge == staleSlot)
                    slotToExpunge = i;
                cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
                return;
            }

            if (k == null && slotToExpunge == staleSlot)
                slotToExpunge = i;
        }

        tab[staleSlot].value = null;
        tab[staleSlot] = new Entry(key, value);

        if (slotToExpunge != staleSlot)
            cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
    }
   
    private boolean cleanSomeSlots(int i, int n) {
        boolean removed = false;
        Entry[] tab = table;
        int len = tab.length;
        do {
            i = nextIndex(i, len);
            Entry e = tab[i];
            if (e != null && e.get() == null) {
                n = len;
                removed = true;
                i = expungeStaleEntry(i);
            }
        } while ( (n >>>= 1) != 0);
        return removed;
    }

    private void expungeStaleEntries() {
        Entry[] tab = table;
        int len = tab.length;
        for (int j = 0; j < len; j++) {
            Entry e = tab[j];
            if (e != null && e.get() == null)
                expungeStaleEntry(j);
        }
    }
}
```



#### getEntry

> java.lang.ThreadLocal.ThreadLocalMap
```java
static class ThreadLocalMap {
    private Entry getEntry(ThreadLocal<?> key) {
        int i = key.threadLocalHashCode & (table.length - 1);
        Entry e = table[i];
        if (e != null && e.get() == key)
            // 指定下标的entry存在且key与当前参数TreadLocal相等，则返回对应的值
            return e;
        else
            return getEntryAfterMiss(key, i, e);
    }

    private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
        Entry[] tab = table;
        int len = tab.length;

        while (e != null) {
            ThreadLocal<?> k = e.get();
            if (k == key)
                return e;
            if (k == null)
                expungeStaleEntry(i);
            else
                i = nextIndex(i, len);
            e = tab[i];
        }
        return null;
    }    

    private int expungeStaleEntry(int staleSlot) {
        Entry[] tab = table;
        int len = tab.length;

        // expunge entry at staleSlot
        tab[staleSlot].value = null;
        tab[staleSlot] = null;
        size--;

        // Rehash until we encounter null
        Entry e;
        int i;
        for (i = nextIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = nextIndex(i, len)) {
            ThreadLocal<?> k = e.get();
            if (k == null) {
                e.value = null;
                tab[i] = null;
                size--;
            } else {
                int h = k.threadLocalHashCode & (len - 1);
                if (h != i) {
                    tab[i] = null;

                    // Unlike Knuth 6.4 Algorithm R, we must scan until
                    // null because multiple entries could have been stale.
                    while (tab[h] != null)
                        h = nextIndex(h, len);
                    tab[h] = e;
                }
            }
        }
        return i;
    }
}
```



#### remove

> java.lang.ThreadLocal.ThreadLocalMap
```java
    private void remove(ThreadLocal<?> key) {
        Entry[] tab = table;
        int len = tab.length;
        int i = key.threadLocalHashCode & (len-1);
        for (Entry e = tab[i];
             e != null;
             e = tab[i = nextIndex(i, len)]) {
            if (e.get() == key) {
                e.clear();
                expungeStaleEntry(i);
                return;
            }
        }
    }    

    private int expungeStaleEntry(int staleSlot) {
        Entry[] tab = table;
        int len = tab.length;

        // expunge entry at staleSlot
        tab[staleSlot].value = null;
        tab[staleSlot] = null;
        size--;

        // Rehash until we encounter null
        Entry e;
        int i;
        for (i = nextIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = nextIndex(i, len)) {
            ThreadLocal<?> k = e.get();
            if (k == null) {
                e.value = null;
                tab[i] = null;
                size--;
            } else {
                int h = k.threadLocalHashCode & (len - 1);
                if (h != i) {
                    tab[i] = null;

                    // Unlike Knuth 6.4 Algorithm R, we must scan until
                    // null because multiple entries could have been stale.
                    while (tab[h] != null)
                        h = nextIndex(h, len);
                    tab[h] = e;
                }
            }
        }
        return i;
    }
```



#### rehash

> java.lang.ThreadLocal.ThreadLocalMap
```java
static class ThreadLocalMap {
    private void rehash() {
        // 删除所有旧的元素
        expungeStaleEntries();

        if (size >= threshold - threshold / 4)
            // 超出设置的threshold*3/4时，进行扩容操作
            resize();
    }

    private void resize() {
        Entry[] oldTab = table;
        int oldLen = oldTab.length;
        // 扩容后的大小为原先的2倍
        int newLen = oldLen * 2;
        Entry[] newTab = new Entry[newLen];
        int count = 0;

        for (int j = 0; j < oldLen; ++j) {
            Entry e = oldTab[j];
            if (e != null) {
                ThreadLocal<?> k = e.get();
                if (k == null) {
                    e.value = null;
                } else {
                    int h = k.threadLocalHashCode & (newLen - 1);
                    while (newTab[h] != null)
                        h = nextIndex(h, nsewLen);
                    newTab[h] = e;
                    count++;
                }
            }
        }

        setThreshold(newLen);
        size = count;
        table = newTab;
    }
}
```



## InheritableThreadLocal

`ThreadLocal`是线程隔离的，如果希望父子线程共享数据，可以使用`InheritableThreadLocal`。

```java
public class InheritableThreadLocalTest {

   public static void main(String[] args) {
       ThreadLocal<String> threadLocal = new ThreadLocal<>();
       InheritableThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();

       threadLocal.set("关注公众号：捡田螺的小男孩");
       inheritableThreadLocal.set("关注公众号：程序员田螺");

       Thread thread = new Thread(()->{
           System.out.println("ThreadLocal value " + threadLocal.get());
           System.out.println("InheritableThreadLocal value " + inheritableThreadLocal.get());
       });
       thread.start();
       
   }
}
```



```bash
//运行结果
ThreadLocal value null
InheritableThreadLocal value 关注公众号：程序员田螺
```



子线程中，是可以获取到父线程的 `InheritableThreadLocal `类型变量的值，但是不能获取到 `ThreadLocal `类型变量的值。

这是因为在`Thread`类中，除了成员变量`threadLocals`之外，还有另一个成员变量：`inheritableThreadLocals`。

```java
public class Thread implements Runnable {
   ThreadLocalMap threadLocals = null;
   ThreadLocalMap inheritableThreadLocals = null;
 }
```



`Thread`类的`init`方法中，有一段初始化设置：

```java
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {

    // ......
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    /* Stash the specified stack size in case the VM cares */
    this.stackSize = stackSize;

    /* Set thread ID */
    tid = nextThreadID();
}
static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
    return new ThreadLocalMap(parentMap);
}
```



可以发现，当`parent的inheritableThreadLocals`不为`null`时，就会将`parent`的`inheritableThreadLocals`，赋值给前线程的`inheritableThreadLocals`。说白了，就是如果当前线程的`inheritableThreadLocals`不为`null`，就从父线程哪里拷贝过来一个过来，类似于另外一个`ThreadLocal`，数据从父线程那里来的。



## 使用场景

`ThreadLocal`的应用场景主要有以下这几种：

- 使用日期工具类，当用到`SimpleDateFormat`，使用ThreadLocal保证线性安全
- 全局存储用户信息（用户信息存入`ThreadLocal`，那么当前线程在任何地方需要时，都可以使用）
- 保证同一个线程，获取的数据库连接`Connection`是同一个，使用`ThreadLocal`来解决线程安全的问题
- 使用`MDC`保存日志信息。



# 其它线程相关类

## ThreadGroup

ThreadGroup是一个类，它的目的是提供关于线程组的信息。

ThreadGroup API比较薄弱，它并没有比Thread提供了更多的功能。它有两个主要的功能：一是获取线程组中处于活跃状态线程的列表；二是设置为线程设置未捕获异常处理器(ncaught exception handler)。但在Java 1.5中Thread类也添加了setUncaughtExceptionHandler(UncaughtExceptionHandler eh) 方法，所以ThreadGroup是已经过时的，不建议继续使用。



## Timer相关

java.util.Timer是一个工具类，可以用于安排一个线程在未来的某个特定时间执行。Timer类可以用安排一次性任务或者周期任务。

java.util.TimerTask是一个实现了Runnable接口的抽象类，我们需要去继承这个类来创建我们自己的定时任务并使用Timer去安排它的执行。



# ForkJoin

fork join框架是JDK7中出现的一款高效的工具，Java开发人员可以通过它充分利用现代服务器上的多处理器。它是专门为了那些可以递归划分成许多子模块设计的，目的是将所有可用的处理能力用来提升程序的性能。fork join框架一个巨大的优势是它使用了工作窃取算法，可以完成更多任务的工作线程可以从其它线程中窃取任务来执行。

Fork/Join框架是Java7提供的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。

Fork/Join框架需要理解两个点，「分而治之」和「工作窃取算法」。



**工作窃取算法**

把大任务拆分成小任务，放到不同队列执行，交由不同的线程分别执行时。有的线程优先把自己负责的任务执行完了，其他线程还在慢慢悠悠处理自己的任务，这时候为了充分提高效率，就需要工作盗窃算法啦~

![图片](../../Image/2022/09/220922-10.jpg)

工作盗窃算法就是，「某个线程从其他队列中窃取任务进行执行的过程」。一般就是指做得快的线程（盗窃线程）抢慢的线程的任务来做，同时为了减少锁竞争，通常使用双端队列，即快线程和慢线程各在一端。
