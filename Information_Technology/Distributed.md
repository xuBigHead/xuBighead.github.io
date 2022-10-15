# 分布式理论
## CAP

当我们的单个数据库的性能产生瓶颈的时候，我们可能会对数据库进行分区，这里所说的分区指的是物理分区，分区之后可能不同的库就处于不同的服务器上了，这个时候单个数据库的ACID已经不能适应这种情况了，而在这种ACID的集群环境下，再想保证集群的ACID几乎是很难达到，或者即使能达到那么效率和性能会大幅下降，最为关键的是再很难扩展新的分区了，这个时候如果再追求集群的ACID会导致我们的系统变得很差，这时我们就需要引入一个新的理论原则来适应这种集群的情况，就是CAP原则或者叫CAP定理？

CAP原理是现代分布式系统的理论基石，所有的分布式系统都是基于CAP理论来考虑和设计的。**CAP理论的核心重点就是描述了任何一个分布式系统最多只能满足以下三个特性中的两个。**



### 基础概念

#### 一致性

> all nodes see the same data at the same time



一致性，强一致性（Consistency），这里一致性的意思是在分布式系统中，对多副本数据的读操作总是能读到之前写操作完成的结果, 说白了就要满足强一致性; 比如一个数据有多个存储节点，我在A节点对数据做出了更新，而A节点需要将数据同步到节点B和节点C甚至更多，同步虽然快，但也需要一定的时间，如果这个时间段内有并发请求过来读取这个数据，请求被负载均衡，就有可能出现多个请求读取到的数据不一致的问题，这是因为有的请求可能会落到还未完成数据同步的节点上；一致性就是为了避免出现这样的情况。

数据一致性指“all nodes see the same data at the same time”，即更新操作成功并返回客户端完成后，所有节点在同一时间的数据完全一致，不能存在中间状态。

分布式环境中，一致性是指多个副本之间能否保持一致的特性。在一致性的需求下，当一个系统在数据一致的状态下执行更新操作后，应该保证系统的数据仍然处理一致的状态。



##### 强一致性

如果时刻保证客户端看到的数据都是一致的，那么称之为强一致性。

强一致性就是在任何时刻都从集群节点中获取的数据都是一致性

- 原子一致性
- 线性一致性



##### 弱一致性

与强一致性相对的就是弱一致性，即数据更新之后，如果立即访问的话可能访问不到或者只能访问部分的数据。

如果允许存在部分数据不一致，那么就称之为弱一致性。

系统中的某个数据被更新后，后续对该数据的读取操作可能得到更新后的值，也可能是更改前的值。可以有多种实现方式。

- 最终一致性
	最终一致性是弱一致性的一种形式，就是不保证在任意时刻任意节点上的同一份数据都是相同的，但是随着时间的迁移，不同节点上的同一份数据总是在向趋同的方向变化；总之就是一段时间后，集群节点的数据会最终达到一致状态。

- 因果一致性
	如果进程A通知进程B它已更新了一个数据项，那么进程B的后续访问将获得的是进程A更新后的值

- 读己之所写一致性
	当进程A自己更新一个数据项之后，它肯定会访问到自己更新过的值，绝不会看到旧值。这是因果一致性模型的一个特例。

- 会话一致性
	这是上一个模型的实用版本，它把访问存储系统的进程放到会话的上下文中。只要会话还存在，系统就保证“读己之所写”一致性。如果由于某些失败情形令会话终止，就要建立新的会话，而且系统的保证不会延续到新的会话。

- 单调读一致性
	如果进程已经看到过数据对象的某个值，那么任何后续访问都不会返回在那个值之前的值。

- 单调写一致性
	系统保证来自同一个进程的写操作顺序执行。要是系统不能保证这种程度的一致性，就非常难以编程了

	

我们知道强一致性就是C嘛，通常在保证AP的分布式系统中，都是通过选择最终一致性来弥补数据分区期间的造成的错误，也就是忽略因出现故障或数据同步未完成而导致的数据分区情况，在故障解决后，或数据同步后对分区时期造成的影响做数据补偿和合并，这样就可以弥补分区期间发生的任何错误。



##### 最终一致性

如果允许存在中间状态，只要求经过一段时间后，数据最终是一致的，则称之为最终一致性。



#### 可用性

系统提供的服务必须一直处于可用的状态，对于用户的每一个操作请求总是能够在有限的时间内返回结果。

每个操作都必须以可预期的响应结束。

可用性（Availability），即在某个组件的集群环境中，如果该组件的某个或多个节点发生了故障，在这些节点故障处理完之前，这个分布式系统也要能保证该组件能正常提供服务，不会因为部分节点的瘫痪而牵连整个服务；通常集群多副本环境本身就可以保证可用性，说白了就是保证服务的高可用。



##### 有限时间内

对于用户的一个操作请求，系统必须能够在指定的时间（响应时间）内返回对应的处理结果，**如果超过了这个时间范围，那么系统就被认为是不可用的**。即这个响应时间必须在一个合理的值内。



##### 返回正常结果

要求系统在完成对用户请求的处理后，返回一个正常的响应结果。正常的响应结果通常能够明确地反映出对请求的处理结果，即成功或失败。



#### 分区容错性

即分布式系统在遇到任何网络分区故障时，仍然需要能够保证对外**提供满足一致性和可用性的服务**，除非是整个网络环境都发生了故障。

网络分区，是指分布式系统中，不同的节点分布在不同的子网络（机房/异地网络）中，由于一些特殊的原因导致这些子网络之间出现网络不连通的状态，但各个子网络的内部网络是正常的，从而导致整个系统的网络环境被切分成了若干孤立的区域。组成一个分布式系统的每个节点的加入与退出都可以看做是一个特殊的网络分区。



即使出现单个组件无法可用，操作依然可以完成。

分区容错性（Partition Tolerance），现实生活中可能会出现机器故障，机房停电，网络故障等客观现象，而集群的环境更是增加了重现的概率。就是因为这些可能会出现的问题，从而导致某个时间段，集群节点数据出现分区现象，所以分区容错性的意思就是**系统容忍短暂出现数据分区的情况，等故障修复，再进行分区数据整合，补偿分区时造成的错误，但在数据分区时无法保证数据强一致性或可用性。**

什么是数据分区？ 分区就是因为集群节点之前无法通信，比如机房A无法跟机房B通信，从而造成机房A的节点有部分机房B没有的新数据，而机房B的节点也有部分机房A没有的新数据，但碍于无法通信，所以无法数据同步，从而造成数据的分区，每个分区的数据都不完整，只有合并在一起才是一个完整的数据；但情况也不仅限于通信故障，就比如数据同步时间过长，也有可能发生数据分区，所以分区对通信时限有严格的要求，系统只要在指定时间内不能达成数据的一致性，比如通信故障，复制同步时间过长，就意味着可能会发生数据分区。



![1](../../Image/2022/04/220413.png)



### 实践

#### CA

放弃分区容错性的话，则放弃了分布式，放弃了系统的可扩展性

单点部署的数据库，没有集群环境，必然可以保证数据的可用性和一致性。

MySQL一般被归类为CA，但也看设置情况，如果是主从同步复制，基本就符合CA，如果是异步复制，那就不一定满足CA了。

满足C需要所有的服务器的数据要一样，也就是说要实现数据的同步，那么同步要不要时间？肯定是要的，并且机器越多，同步的时间肯定越慢，这里问题就来了，我们同时也满足了A，也就是说，我要同步时间短才行。这样的话，机器就不能太多了，也就是说P是满足不了的



#### CP

放弃可用性的话，则在遇到网络分区或其他故障时，受影响的服务需要等待一定的时间，再此期间无法对外提供政策的服务，即不可用

ZooKeeper就是CP的实践者，即任何时刻对ZooKeeper的访问请求能得到一致的数据结果，同时系统对网络分割具备容错性；但是它不能保证每次服务请求的可用性（就是在极端环境下，ZooKeeper可能会丢弃一些请求，消费者程序需要重新请求才能获得结果）。

Mongodb一般被归类为CP。

满足P需要很多服务器，假设有1000台服务器，同时满足了C，也就是说要保证每台机器的数据都一样，那么同步的时间可就很大，在这种情况下，我们肯定是不能保证用户随时访问每台服务器获取到的数据都是最新的，想要获取最新的，你就得等，等全部同步完了，你就可以获取到了，但是我们的A要求短时间就可以拿到想要的数据啊，这不就是矛盾了，所以说这里A是满足不了了。



#### AP

放弃一致性的话（这里指强一致），则系统无法保证数据保持实时的一致性，在数据达到最终一致性时，有个时间窗口，在时间窗口内，数据是不一致的。

Eureka遵循的就是AP, 因为针对同一个服务，即使注册中心的不同节点保存的服务提供者信息不完全相同，但也并不会造成灾难性的后果，因为对于服务消费者来说，能消费才是最重要的，拿到的服务提供者即使不正错，大不了重试，那也好过因为无法获取服务提供者信息而不能去消费好。




### 扩展

#### CAP与ACID

##### C含义对比

ACID理论和CAP理论都有一个C，也都叫一致性Consistent，这两个C是有区别的：

ACID的C指的是事务中的一致性，在一串对数据进行修改的操作中，保证数据的正确性。即数据在事务期间的多个操作中，数据不会凭空的消失或增加，数据的每一个增删改操作都是有因果关系的；比如用户A想用户B转了200块钱，不会出现用户A扣了款，而用户B没有收到的情况。

CAP的C则指的是分布式环境中，多服务之间的复制是异步，需要一定耗时的，不是即时瞬间完成。所以可能会造成某个节点的数据修改，将修改的数据同步到其他服务需要一定的时间，如果此时有并发请求过来，请求负载均衡到多个节点，可能会出现多个节点获取的数据不一致的问题，因为请求有可能会落在还没完成数据同步的节点上；而C就是为了做到在分布式集群环境读到的数据是一致的；当然这里的C也有分类，如强一致性，弱一致性，最终一致性。

即ACID的C着重强调单数据库事务操作时，要保证数据的完整和正确性；而CAP理论中的C指的是对一个数据多个备份的读写一致性。

- ACID一致性是有关数据库规则，数据库总是从一个一致性的状态转换到另外一个一致性的状态；
- CAP的一致性是分布式多服务器之间复制数据令这些服务器拥有同样的数据，由于网速限制，这种复制在不同的服务器上所消耗的时间是不固定的，集群通过组织客户端查看不同节点上还未同步的数据维持逻辑视图，这是一种分布式领域的一致性概念；



##### A含义对比

- ACID中的A指的是原子性(Atomicity)，是指事务被视为一个不可分割的最小工作单元，事务中的所有操作要么全部提交成功，要么全部失败回滚；
- CAP中的A指的是可用性(Availability)，是指集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求；



ACID里的一致性指的是事务执行前后，数据库完整性，而CAP的一致性，指的是分布式节点的数据的一致性。



#### CAP三选二

总之CAP的理论核心就是C , A ,P不能共存，只可能三选二，以求最大能保证的有利利益

因为长时间无法达成数据一致性，就可能造成数据分区，所以要满足P，就必须在A和C做出选择：

(不去满足一致性C)即我不追求数据强一致性，在集群节点某时刻出现数据不一致的情况下，我可以去保证分区容错性，容忍分区的出现，之后再做补偿 , 即做到AP。

(不去满足可用性A)即在可能发生数据分区的时候，在故障解决之前，我去停止集群节点的对外服务，避免出现对数据的增，删，改操作，让数据不被改动，这样就不会导致数据的分区了,做到CP。

(不去满足分区容错性P)则代表当没有发生数据分区时，系统可以保证A和C，但是如果发生数据分区，因为不满足分区容错，即无法容忍分区的存在，就必定需要A,C二选一，最终只剩A或者C , 即 CA。

所以我们可以知道，CA - 单节点系统满足一致性，可用性，因为单节点，自然没有P的问题，但是没有扩展性，非常容易遇到性能瓶颈，其实本质上单节点系统也不需要考虑CAP问题。

CP - 满足一致性，分区容忍必的系统，通常性能不是特别高，因为为了保证数据强一致性，必须等待所有节点完成数据同步才能对外提供服务。

AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些，性能高，一般追求最终一致性

从上面看来，P几乎是必不可少的，因为AC最终会退化成A或C，除非选择做一个单节点服务，但这样也就不是分布式集群系统了，CAP理论就没有卵子用了， 所以AC几乎就是一个最不好的选择，所以通常情况下的实践都是在CP或AP中做出选择。



### 总结

分区是常态，无法避免，CAP三者不可共存，所以必然是三选二。

可用性和一致性是一对冤家，正是因为要做到高可用，所以才会出现不一致性。就因为存在了多个节点，才会出现数据复制和通信问题)；也正是因为出现不一致性，才可能要停止集群服务，防止出现分区。

很多情况，一些实践者并不完全的去追求CP或是AP, 他们可能是尽量的去保证可用性，也尽量的去保证一致性，同时又满足一定的分区。



对于多数大型互联网应用的场景，主机众多、部署分散，而且现在的集群规模越来越大，所以节点故障、网络故障是常态，而且要保证服务可用性达到 N 个 9，即保证 P 和 A，舍弃C（退而求其次保证最终一致性）。虽然某些地方会影响客户体验，但没达到造成用户流程的严重程度。

对于涉及到钱财这样不能有一丝让步的场景，C 必须保证。网络发生故障宁可停止服务，这是保证 CA，舍弃 P。

对于互联网来说，由于网络环境是不可信的，所以分区容错性（P）必须满足。为了用户体验，先选可用性。




## BASE

BASE理论就是对CAP理论中的一致性和可用性权衡之后的结果，指的就是分布式系统中，如果无法做到强一致性，那么就用最终一致性去代替，让分布式系统满足三个特性：

- 基本可用（Basically Available）
- 软状态（Soft State）
- 最终一致性（Eventual Consistency）

BASE理论是提出通过牺牲一致性来获得可用性，并允许数据在一段时间内是不一致的，但最终达到一致状态。



### 基础概念

#### 基本可用

`基本可用`（Basically Available）的意思就是分布式系统如果出现不可预知的故障时，允许损失部分的可用性，但并不是整个系统都不可用，即保证系统核心服务可用即可；比如实际开发中，出现部分服务故障，我们可以让系统的响应时间适当变长，或者限流，降低消费，甚至是服务降级，让某个服务暂时无法提供服务。



##### 响应时间

当出现故障时，响应时间增加。



##### 功能损失

当流量高峰期时，屏蔽一些功能的使用以保证系统稳定性（服务降级）。



#### 软状态

软状态（Soft State）指系统中的数据可以存在中间状态， 并该中间状态不会影响到系统的可用性，即允许部分节点的数据存在延迟问题。



#### 最终一致性

最终一致性（Eventual Consistency）强调的是所有的数据副本，在经过一段时间的同步之后，最终都能够达到数据一致的状态。既最终一致性的本质是需要系统保证数据的最终一致性，而不每时每刻的强一致性。但也要保证非一致窗口时期不会对系统数据造成危害。



> 其实只有两类数据一致性，强一致性与弱一致性。强一致性也叫做线性一致性，除此以外，所有其他的一致性都是弱一致性的特殊情况。强一致性，即复制是同步的，弱一致性，即复制是异步的。
>
> 用户更新网站头像，在某个时间点，用户向主库发送更新请求，不久之后主库就收到了请求。在某个时刻，主库又会将数据变更转发给自己的从库。最后，主库通知用户更新成功。如果主库需要等待从库的确认，确保从库已经收到写入操作，那么复制是同步的，即强一致性。如果主库写入成功后，不等待从库的响应，直接返回，则复制是异步的，即弱一致性。
>
> 强一致性可以保证从库有与主库一致的数据。如果主库突然宕机，仍可以保证数据完整。但如果从库宕机或网络阻塞，主库就无法完成写入操作。



最终一致性可分为如下几种：

##### 因果一致性

因果一致性（Causal consistency）即进程A在更新完数据后通知进程B，那么之后进程B对该项数据的范围都是进程A更新后的最新值。



##### 读己之所写

读己之所写（Read your writes）指进程A更新一项数据后，它自己总是能访问到自己更新过的最新值。



##### 会话一致性

会话一致性（Session consistency）指将数据一致性框定在会话当中，在一个会话当中实现读己之所写的一致性。即执行更新后，客户端在同一个会话中始终能读到该项数据的最新值



##### 单调读一致性

单调读一致性（Monotonic read consistency）指如果一个进程从系统中读取出一个数据项的某个值后，那么系统对于该进程后续的任何数据访问都不应该返回更旧的值。



##### 单调写一致性

单调写一致性（Monotoic write consistency）指一个系统需要保证来自同一个进程的写操作被顺序执行。



### 扩展

#### BASE与ACID

ACID是传统数据库常用的设计理念，追求强一致性模型，例如银行的转账场景，最求数据的绝对可靠。而BASE支持的是大型分布式系统，提出通过牺牲强一致性获得高可用性。

虽然ACID和BASE代表了两种截然相反的设计哲学，在分布式系统设计的场景中，系统组件对一致性要求是不同的，因此ACID和BASE又会结合使用。



BASE理论面向的是大型高可用可扩展的分布式系统，和传统的事物ACID特性是相反的。

它完全不同于ACID的强一致性模型，而是通过牺牲强一致性来获得可用性，并允许数据在一段时间内是不一致的，但最终达到一致状态。

但同时，在实际的分布式场景中，不同业务单元和组件对数据一致性的要求是不同的，因此在具体的分布式系统架构设计过程中，ACID特性和BASE理论往往又会结合在一起。



#### BASE和CAP

BASE理论是对CAP中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的总结， 是基于CAP定理逐步演化而来的。BASE理论的核心思想是：**即使无法做到强一致性，但每个应用都可以根据自身业务特点，采用适当的方式来使系统达到最终一致性**。

BASE理论其实就是对CAP理论的延伸和补充，主要是对AP的补充。牺牲数据的强一致性，来保证数据的可用性，虽然存在中间装填，但数据最终一致。




### 总结

总的来说，BASE理论面向的是大型高可用可扩展的分布式系统，是对CAP理论的一些弱化和妥协，为了保证可用性，对强一致性进行了削弱。

在了解了CAP和BASE理论之后，就可以自己去了解一下Redis, MongoDB, MySQL这些数据库怎么处理集群状态下的数据复制了，相信这也是一个很好的加深基础的问题；同时也可以去了解一下通常系统是怎么去实现最终一致性的，怎么解决非一致性窗口期间出现一些数据问题。



# 分布式锁

在一个分布式系统中，由于涉及到多个实例同时对同一个资源加锁的问题，像传统的synchronized、ReentrantLock等单进程情况加锁的api就不再适用，需要使用分布式锁来保证多服务实例之间加锁的安全性。

分布式锁需要具备下述条件：

- 与单机系统一样的资源互斥功能，这是锁的基础
- 高性能获取、释放锁
- 高可用
- 具备可重入性
- 有锁失效机制，防止死锁
- 非阻塞，不管是否获得锁，要能快速返回



## 概述

### 基础规范

#### 加锁规范

##### 保证加锁的原子性

假如加锁成功，但是设置超时时间失败了，该lockKey就变成永不失效。假如在高并发场景中，有大量的lockKey加锁成功了，但不会失效，有可能直接导致redis内存空间不足。



##### 阻塞式加锁

自旋不断尝试加锁（就是在死循环中，不断尝试加锁），如果成功则直接返回。如果失败，则休眠50毫秒，再发起新一轮的尝试。如果到了超时时间，还未加锁成功，则直接返回失败。



##### 超时等待放弃

等待一段时间后还不能加锁成功，则放弃加锁。



#### 释放锁规范

##### 锁的过期时间

设置锁的过期时间主要原因是为了防止死锁。当某个客户端获取到锁，还没来得及主动释放锁，那么此时假如客户端宕机了，又或者是释放锁失败了，那么如果没有设置过期时间，那么这个锁key会一直在，那么其它线程来加锁的时候会发现key已经被加锁了，那么其它线程一直会加锁失败，就会产生死锁的问题。



##### 手动释放锁

当业务执行完成之后，肯定需要手动释放锁，那么为什么需要主动释放锁呢？

第一，假设你任务执行完，没有手动释放锁，如果没有指定锁的超时时间，那么因为有看门狗机制，势必会导致这个锁无法释放，那么就可能造成**死锁**的问题。

第二，如果你指定了锁超时时间，虽然并不会造成死锁的问题，但是会造成**资源浪费**的问题。假设你设置的过期时间是30s，但是你的任务2s就完成了，那么这个锁还会白白被占有28s的时间，这28s内其它线程都无法成功加锁。



释放锁时只能释放自己加的锁，不允许释放别人加的锁。



##### 超时自动释放

通过设置超时时间，分布式锁在一定时间后会自动释放锁。

指定超时时间达到超时释放锁的功能主要还是通过redis自动过期来实现，因为指定了超时时间，加锁成功之后就不会开启watchdog机制来延长加锁的时间。

在实际项目中，指不指定锁的超时时间是根据具体的业务来的，如果你能够比较准确的预估出代码执行的时间，那么可以指定锁超时释放时间来防止业务执行错误导致无法释放锁的问题，如果不能预估出代码执行的时间，那么可以不指定超时时间。



### 存在风险

对于单Redis实例来说，如果Redis宕机了，那么整个系统就无法工作了。所以为了保证Redis的高可用性，一般会使用主从或者哨兵模式。但是如果使用了主从或者哨兵模式，此时Redis的分布式锁的功能可能就会出现问题。



## 数据库分布式锁

### 悲观锁实现

可以使用 select ... for update 来实现分布式锁。



### 乐观锁实现

乐观锁指每次更新操作，都觉得不会存在并发冲突，只有更新失败后，才重试。

创建 version 字段，每次更新修改，都会自增加一，然后去更新余额时，把查出来的那个版本号，带上条件去更新，如果是上次那个版本号，就更新，如果不是，表示别人并发修改过了，就继续重试。



### 总结

适合并发不高的场景，一般需要设置一下重试的次数。



## Redisson分布式锁

说到redis的分布式锁，可能第一时间就想到了setNx命令，这个命令保证一个key同时只能有一个线程设置成功，这样就能实现加锁的互斥性。但是Redisson并没有通过setNx命令来实现加锁，而是自己实现了一套完成的加锁的逻辑。



### 非公平锁

#### 实现流程



#### 实现原理

##### 看门狗机制

Redisson对于这种**未指定超时时间**的加锁，就实现了一个叫watchdog机制，也就是看门狗机制来自动延长加锁的时间。

在客户端通过tryLockInnerAsync方法加锁成功之后，如果你没有指定锁过期的时间，那么客户端会起一个定时任务，来定时延长加锁时间，默认每10s执行一次。所以watchdog的本质其实就是一个定时任务。

因为有了看门狗机制，所以说如果没有设置过期时间并且没有主动去释放锁，那么这个锁就永远不会被释放，因为定时任务会不断的去延长锁的过期时间，造成死锁的问题。但是如果发生宕机了，是不会造成死锁的，因为宕机了，服务都没了，那么看门狗的这个定时任务就没了，也自然不会去续约，等锁自动过期了也就自动释放锁了，跟上述说的为什么需要设置过期时间是一样的。



##### 可重入锁

可重入加锁的意思就是同一个客户端同一个线程也能多次对同一个锁进行加锁。也就是同时可以执行多次 lock方法，流程都是一样的，最后也会调用到lua脚本，所以可重入加锁的逻辑最后也是通过加锁的lua脚本来实现的。



##### 不同线程加锁互斥

上面我们分析了第一次加锁逻辑和可重入加锁的逻辑，因为lua脚本加锁的逻辑同时只有一个线程能够执行（redis是单线程的原因），所以一旦有线程加锁成功，那么另一个线程来加锁，前面两个if条件都不成立，最后通过调用redis的pttl命令返回锁的剩余的过期时间回去。

这样，客户端就根据返回值来判断是否加锁成功，因为第一次加锁和可重入加锁的返回值都是nil，而加锁失败就返回了锁的剩余过期时间。

所以加锁的lua脚本通过条件判断就实现了加锁的互斥操作，保证其它线程无法加锁成功。总的来说，加锁的lua脚本实现了第一次加锁、可重入加锁和加锁互斥的逻辑。



##### 阻塞等待加锁

通过执行死循环（自旋）地的方式来不停地通过tryAcquire方法来尝试加锁，直到加锁成功之后才会跳出死循环，如果一直没有成功加锁，那么就会一直旋转下去，所谓的阻塞，实际上就是自旋加锁的方式。

但是这种阻塞可能会产生问题，因为如果其它线程释放锁失败，那么这个阻塞加锁的线程会一直阻塞加锁，这肯定会出问题的。



##### 超时放弃加锁

超时放弃加锁指的是在指定时间内不断尝试加锁，直到超出指定时间后放弃加锁。



##### 超时自动释放

通过传入leaseTime参数就可以指定锁超时的时间，达到指定时间后就会释放锁。

无论指不指定超时时间，最终其实都会调用tryAcquireAsync方法，只不过当不指定超时时间时，leaseTime传入的是-1，也就是代表不指定超时时间，但是Redisson默认还是会设置30s的过期时间；当指定超时时间，那么leaseTime就是我们自己指定的时间，最终也是通过同一个加锁的lua脚本逻辑。

指定和不指定超时时间的主要区别是，加锁成功之后的逻辑不一样，不指定超时时间时，会开启watchdog后台线程，不断的续约加锁时间，而指定超时时间，就不会去开启watchdog定时任务，这样就不会续约，加锁key到了过期时间就会自动删除，也就达到了释放锁的目的。



#### 源码解析

```java
    public void updateByNoFairLock() {
        RLock lock = redissonClient.getLock("noFairLockName");
        try {
            lock.lock();
            // business function
        } finally {
            lock.unlock();
        }
    }
```



##### getLock

> org.redisson.Redisson

```java
   /**
     * 根据name返回Lock实例
     * 实现了非公平锁，因此不保证依据线程的顺序
     */
    @Override
    public RLock getLock(String name) {
        return new RedissonLock(connectionManager.getCommandExecutor(), name);
    }
```



##### lock

###### 阻塞式加锁

> org.redisson.RedissonLock

```java
    @Override
    public void lock() {
        try {
            lock(-1, null, false);
        } catch (InterruptedException e) {
            throw new IllegalStateException();
        }
    }

    @Override
    public void lock(long leaseTime, TimeUnit unit) {
        try {
          	// 通过指定 leaseTime 表示超过该时间自动释放锁
            lock(leaseTime, unit, false);
        } catch (InterruptedException e) {
            throw new IllegalStateException();
        }
    }

    private void lock(long leaseTime, TimeUnit unit, boolean interruptibly) throws InterruptedException {
        long threadId = Thread.currentThread().getId();
        Long ttl = tryAcquire(leaseTime, unit, threadId);
        // lock acquired
        if (ttl == null) {
            return;
        }

        RFuture<RedissonLockEntry> future = subscribe(threadId);
        commandExecutor.syncSubscription(future);

        try {
            while (true) {
                // 自旋阻塞方式不停尝试加锁
                ttl = tryAcquire(leaseTime, unit, threadId);
                // lock acquired
                if (ttl == null) {
                    break;
                }

                // waiting for message
                if (ttl >= 0) {
                    try {
                        getEntry(threadId).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                    } catch (InterruptedException e) {
                        if (interruptibly) {
                            throw e;
                        }
                        getEntry(threadId).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                    }
                } else {
                    if (interruptibly) {
                        getEntry(threadId).getLatch().acquire();
                    } else {
                        getEntry(threadId).getLatch().acquireUninterruptibly();
                    }
                }
            }
        } finally {
            unsubscribe(future, threadId);
        }
//        get(lockAsync(leaseTime, unit));
    }

    /*
     * 异步转同步的过程
     */
    private Long tryAcquire(long leaseTime, TimeUnit unit, long threadId) {
        return get(tryAcquireAsync(leaseTime, unit, threadId));
    }

    protected final <V> V get(RFuture<V> future) {
        return commandExecutor.get(future);
    }

    /*
     * 异步加锁
     */
    private <T> RFuture<Long> tryAcquireAsync(long leaseTime, TimeUnit unit, long threadId) {
        if (leaseTime != -1) {
            // 不设置过期时间
            return tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
        }
        RFuture<Long> ttlRemainingFuture = tryLockInnerAsync(commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
        ttlRemainingFuture.onComplete((ttlRemaining, e) -> {
            if (e != null) {
                // 发送异常，直接返回
                return;
            }

            // lock acquired
            if (ttlRemaining == null) { //表示第一次设置锁键
                // 看门狗机制，只有不设置过期时间才会启动看门狗进行自动延期
                scheduleExpirationRenewal(threadId);
            }
        });
        return ttlRemainingFuture;
    }
```



###### 超时放弃式加锁

```java
    @Override
    public boolean tryLock(long waitTime, TimeUnit unit) throws InterruptedException {
        return tryLock(waitTime, -1, unit);
    }

    @Override
    public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
        long time = unit.toMillis(waitTime);
        long current = System.currentTimeMillis();
        long threadId = Thread.currentThread().getId();
        Long ttl = tryAcquire(leaseTime, unit, threadId);
        // lock acquired
        if (ttl == null) {
            return true;
        }
        
        // 判断是否超时
        time -= System.currentTimeMillis() - current;
        if (time <= 0) {
            acquireFailed(threadId);
            return false;
        }
        
        current = System.currentTimeMillis();
        RFuture<RedissonLockEntry> subscribeFuture = subscribe(threadId);
        if (!subscribeFuture.await(time, TimeUnit.MILLISECONDS)) {
            if (!subscribeFuture.cancel(false)) {
                subscribeFuture.onComplete((res, e) -> {
                    if (e == null) {
                        unsubscribe(subscribeFuture, threadId);
                    }
                });
            }
            acquireFailed(threadId);
            return false;
        }

        try {
            // 自旋加锁前判断是否超时，超时则不再尝试加锁
            time -= System.currentTimeMillis() - current;
            if (time <= 0) {
                acquireFailed(threadId);
                return false;
            }
        
            while (true) {
                long currentTime = System.currentTimeMillis();
                ttl = tryAcquire(leaseTime, unit, threadId);
                // lock acquired
                if (ttl == null) {
                    return true;
                }

                // 判断是否超时，超时了就放弃加锁，推出循环
                time -= System.currentTimeMillis() - currentTime;
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }

                // waiting for message
                currentTime = System.currentTimeMillis();
                if (ttl >= 0 && ttl < time) {
                    getEntry(threadId).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                } else {
                    getEntry(threadId).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
                }

                // 判断是否超时，超时了就放弃加锁，推出循环
                time -= System.currentTimeMillis() - currentTime;
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }
            }
        } finally {
            unsubscribe(subscribeFuture, threadId);
        }
//        return get(tryLockAsync(waitTime, leaseTime, unit));
    }
```



###### 看门狗机制

> org.redisson.RedissonLock

```java
    private void scheduleExpirationRenewal(long threadId) {
        ExpirationEntry entry = new ExpirationEntry();
        ExpirationEntry oldEntry = EXPIRATION_RENEWAL_MAP.putIfAbsent(getEntryName(), entry);
        if (oldEntry != null) {
            oldEntry.addThreadId(threadId);
        } else {
            // 第一次加锁的时候会调用，内部会启动WatchDog
            entry.addThreadId(threadId);
            renewExpiration();
        }
    }

    private void renewExpiration() {
        ExpirationEntry ee = EXPIRATION_RENEWAL_MAP.get(getEntryName());
        if (ee == null) {
            return;
        }
        // 用到了时间轮机制
        Timeout task = commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
            @Override
            public void run(Timeout timeout) throws Exception {
                ExpirationEntry ent = EXPIRATION_RENEWAL_MAP.get(getEntryName());
                if (ent == null) {
                    return;
                }
                Long threadId = ent.getFirstThreadId();
                if (threadId == null) {
                    return;
                }
                // 定期执行如下方法，使用lua脚本来实现加锁时间的延长
                RFuture<Boolean> future = renewExpirationAsync(threadId);
                future.onComplete((res, e) -> {
                    if (e != null) {
                        log.error("Can't update lock " + getName() + " expiration", e);
                        return;
                    }
                    
                    if (res) {
                        // 递归执行自身
                        renewExpiration();
                    }
                });
            }
        }, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);
        
        ee.setTimeout(task);
    }
```



internalLockLeaseTime 默认值是 30 * 1000，所以看门狗机制是默认每10秒执行一次。



> org.redisson.RedissonLock

```java
    protected RFuture<Boolean> renewExpirationAsync(long threadId) {
        return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
                "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                    "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                    "return 1; " +
                "end; " +
                "return 0;",
            Collections.<Object>singletonList(getName()), 
            internalLockLeaseTime, getLockName(threadId));
    }
```



这段lua脚本中参数的意思其实是跟加锁的参数的意思是一样的：

- KEYS[1]：就是锁的名称
- ARGV[1]：就是锁的过期时间
- ARGV[2]：代表了加锁的唯一标识，b983c153-7421-469a-addb-44fb92259a1b:1。

这段lua脚本的意思就是判断来续约的线程跟加锁的线程是同一个，如果是同一个，那么将锁的过期时间延长到30s，然后返回1，代表续约成功，不是的话就返回0，代表续约失败，下一次定时任务也就不会执行了。



###### LUA脚本加锁

> org.redisson.RedissonLock

```java
    <T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
        internalLockLeaseTime = unit.toMillis(leaseTime);

        return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
                  "if (redis.call('exists', KEYS[1]) == 0) then " +
                      "redis.call('hset', KEYS[1], ARGV[2], 1); " +
                      "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                      "return nil; " +
                  "end; " +
                  "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                      "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                      "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                      "return nil; " +
                  "end; " +
                  "return redis.call('pttl', KEYS[1]);",
                    Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
    }
```



其中这段脚本中的lua脚本中的参数的意思：

- KEYS[1]：就是锁的名称
- ARGV[1]：就是锁的过期时间，不指定的话默认是30s
- ARGV[2]：代表了加锁的唯一标识，由UUID和线程id组成。一个Redisson客户端一个UUID，UUID代表了一个唯一的客户端。所以由UUID和线程id组成了加锁的唯一标识，可以理解为某个客户端的某个线程加锁。



LUA的加锁的逻辑。

1）先调用redis的exists命令判断加锁的key存不存在，如果不存在的话，那么就进入if。不存在的意思就是还没有某个客户端的某个线程来加锁，第一次加锁肯定没有人来加锁，于是第一次if条件成立。

2）然后调用redis的hincrby的命令，设置加锁的key和加锁的某个客户端的某个线程，加锁次数设置为1，加锁次数很关键，是实现可重入锁特性的一个关键数据。用hash数据结构保存。hincrby命令完成后就形成如下的数据结构。

```c
lockname:{
    "b983c153-7421-469a-addb-44fb92259a1b:1":1
}
```



3）最后调用redis的pexpire的命令，将加锁的key过期时间设置为30s。

从这里可以看出，第一次有某个客户端的某个线程来加锁的逻辑还是挺简单的，就是判断有没有人加过锁，没有的话就自己去加锁，设置加锁的key，再存一下加锁的线程和加锁次数，设置一下锁的过期时间为30s。



###### **可重入锁**

> org.redisson.RedissonLock

```java
 <T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
        // ...
                  // 判断当前已经加锁的key对应的加锁线程跟要来加锁的线程是不是同一个，
                  // 如果是的话，就将这个线程对应的加锁次数加1，
                  // 也就实现了可重入加锁，同时返回nil回去。
                  "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                      "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                      "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                      "return nil; " +
                  "end; " +
                  "return redis.call('pttl', KEYS[1]);",
         // ...
    }
```



可重入加锁成功之后，加锁key和对应的值可能是这样。

```c
lockname:{
    "b983c153-7421-469a-addb-44fb92259a1b:1":1
}
```



##### 无参tryLock

> org.redisson.RedissonLock

```java
    @Override
    public boolean tryLock() {
        return get(tryLockAsync());
    }

    @Override
    public RFuture<Boolean> tryLockAsync() {
        return tryLockAsync(Thread.currentThread().getId());
    }

    @Override
    public RFuture<Boolean> tryLockAsync(long threadId) {
        return tryAcquireOnceAsync(-1, null, threadId);
    }

    private RFuture<Boolean> tryAcquireOnceAsync(long leaseTime, TimeUnit unit, long threadId) {
        if (leaseTime != -1) {
            // 指定了过期时间，则不调用看门狗方法
            return tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_NULL_BOOLEAN);
        }
      
        // 设置默认过期时间，默认是30秒
        RFuture<Boolean> ttlRemainingFuture = tryLockInnerAsync(commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_NULL_BOOLEAN);
        ttlRemainingFuture.onComplete((ttlRemaining, e) -> {
            if (e != null) {
                return;
            }

            // 获取锁成功后，通过看门狗延迟过期时间
            if (ttlRemaining) {
                // 看门狗机制方法
                scheduleExpirationRenewal(threadId);
            }
        });
        return ttlRemainingFuture;
    }
```



##### 有参tryLock

> org.redisson.RedissonLock

```java
    @Override
    public boolean tryLock(long waitTime, TimeUnit unit) throws InterruptedException {
        return tryLock(waitTime, -1, unit);
    }

    @Override
    public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
        long time = unit.toMillis(waitTime);
        long current = System.currentTimeMillis();
        long threadId = Thread.currentThread().getId();
        Long ttl = tryAcquire(leaseTime, unit, threadId);
        // lock acquired
        if (ttl == null) {
            return true;
        }
        
        // 判断是否超时
        time -= System.currentTimeMillis() - current;
        if (time <= 0) {
            acquireFailed(threadId);
            return false;
        }
        
        current = System.currentTimeMillis();
        // 订阅锁释放事件
        // 如果当前线程通过 Redis 的 channel 订阅锁的释放事件获取得知已经被释放，
        // 则会发消息通知待等待的线程进行竞争.
        RFuture<RedissonLockEntry> subscribeFuture = subscribe(threadId);
        // // 将订阅阻塞，阻塞时间设置为我们调用tryLock设置的最大等待时间，超过时间则返回false
        if (!subscribeFuture.await(time, TimeUnit.MILLISECONDS)) {
            if (!subscribeFuture.cancel(false)) {
                subscribeFuture.onComplete((res, e) -> {
                    if (e == null) {
                        unsubscribe(subscribeFuture, threadId);
                    }
                });
            }
            acquireFailed(threadId);
            return false;
        }
        // 收到订阅的消息后走的逻辑
        try {
            // 判断是否超时
            time -= System.currentTimeMillis() - current;
            if (time <= 0) {
                acquireFailed(threadId);
                return false;
            }
            // 循环获取锁（自旋锁），但由于上面有最大等待时间限制，基本会在上面返回false
            while (true) {
                long currentTime = System.currentTimeMillis();
                ttl = tryAcquire(leaseTime, unit, threadId);
                // lock acquired
                if (ttl == null) {
                    return true;
                }

                // 判断是否超时
                time -= System.currentTimeMillis() - currentTime;
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }

                // 通过信号量(共享锁)阻塞,等待解锁消息. (减少申请锁调用的频率)
                // 如果剩余时间(ttl)小于wait time，
                // 就在 ttl 时间内，从Entry的信号量获取一个许可(除非被中断或者一直没有可用的许可)。
                // 否则就在wait time 时间范围内等待可以通过信号量
                currentTime = System.currentTimeMillis();
                if (ttl >= 0 && ttl < time) {
                    getEntry(threadId).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                } else {
                    getEntry(threadId).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
                }

                // 判断是否超时
                time -= System.currentTimeMillis() - currentTime;
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }
            }
        } finally {
            unsubscribe(subscribeFuture, threadId);
        }
    }
```



##### unlock

> org.redisson.RedissonLock

```java
    @Override
    public void unlock() {
        try {
            get(unlockAsync(Thread.currentThread().getId()));
        } catch (RedisException e) {
            if (e.getCause() instanceof IllegalMonitorStateException) {
                throw (IllegalMonitorStateException) e.getCause();
            } else {
                throw e;
            }
        }
    }

    @Override
    public RFuture<Void> unlockAsync(long threadId) {
        RPromise<Void> result = new RedissonPromise<Void>();
        // 调用LUA脚本释放锁
        RFuture<Boolean> future = unlockInnerAsync(threadId);

        future.onComplete((opStatus, e) -> {
            if (e != null) {
                cancelExpirationRenewal(threadId);
                result.tryFailure(e);
                return;
            }
						// 如果 lua 脚本返回的是null，证明当前线程之前并没有成功获取锁，执行tryFailure方法
            if (opStatus == null) {
                IllegalMonitorStateException cause = new IllegalMonitorStateException("attempt to unlock lock, not locked by current thread by node id: "
                        + id + " thread-id: " + threadId);
                result.tryFailure(cause);
                return;
            }
            // 停止 watchdog 的定时续过期时间，
            // 其实就是将对应的 ExpirationEntry 从 EXPIRATION_RENEWAL_MAP 中移除，
            // 当 watchdog 执行时发现当前客户端当前线程没有 ExpirationEntry 了，那么就会停止执行了。
            cancelExpirationRenewal(threadId);
            // // 成功释放锁，执行 trySuccess 方法
            result.trySuccess(null);
        });

        return result;
    }
```



###### LUA脚本释放锁

> org.redisson.RedissonLock

```java
    protected RFuture<Boolean> unlockInnerAsync(long threadId) {
        return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
                "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
                    "return nil;" +
                "end; " +
                "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
                "if (counter > 0) then " +
                    "redis.call('pexpire', KEYS[1], ARGV[2]); " +
                    "return 0; " +
                "else " +
                    "redis.call('del', KEYS[1]); " +
                    "redis.call('publish', KEYS[2], ARGV[1]); " +
                    "return 1; "+
                "end; " +
                "return nil;",
                Arrays.<Object>asList(getName(), getChannelName()), LockPubSub.UNLOCK_MESSAGE, internalLockLeaseTime, getLockName(threadId));

    }
```



lua脚本逻辑如下：

1）先判断来释放锁的线程是不是加锁的线程，如果不是，那么直接返回nil，所以从这里可以看出，主要是通过一个if条件来防止线程释放了其它线程加的锁。

2）如果来释放锁的线程是加锁的线程，那么就将加锁次数减1，然后拿到剩余的加锁次数 counter 变量。

3）如果counter大于0，说明有重入加锁，锁还没有彻底的释放完，那么就设置一下锁的过期时间，然后返回0

4）如果counter没大于0，说明当前这个锁已经彻底释放完了，于是就把锁对应的key给删除，然后发布一个锁已经释放的消息，然后返回1。



###### Redis的发布订阅

Redis提供了一组命令可以让开发者实现“发布/订阅”模式(publish/subscribe) . 该模式同样可以实现进程间的消息传递，它的实现原理是：

发布/订阅模式包含两种角色，分别是发布者和订阅者。订阅者可以订阅一个或多个频道，而发布者可以向指定的频道发送消息，所有订阅此频道的订阅者都会收到该消息

发布者发布消息的命令是PUBLISH， 用法是

```
PUBLISH channel message
```

比如向channel.1发一条消息:hello

```
PUBLISH channel.1 “hello”
```

这样就实现了消息的发送，该命令的返回值表示接收到这条消息的订阅者数量。因为在执行这条命令的时候还没有订阅者订阅该频道，所以返回为0. 另外值得注意的是消息发送出去不会持久化，如果发送之前没有订阅者，那么后续再有订阅者订阅该频道，之前的消息就收不到了

订阅者订阅消息的命令是：

```
SUBSCRIBE channel [channel …]
```

该命令同时可以订阅多个频道，比如订阅channel.1的频道：SUBSCRIBE channel.1，执行SUBSCRIBE命令后客户端会进入订阅状态。



### 公平锁

所谓的公平锁就是指线程成功加锁的顺序跟线程来加锁的顺序是一样，实现了先来先成功加锁的特性，所以叫公平锁。就跟排队一样，不插队才叫公平。



#### 实现原理

通过RedissonClient的getFairLock就可以获取到公平锁。Redisson对于公平锁的实现是RedissonFairLock类，通过RedissonFairLock来加锁，就能实现公平锁的特性。



##### 公平锁和非公平锁的比较

公平锁的优点是按序平均分配锁资源，不会出现线程饿死的情况，它的缺点是按序唤醒线程的开销大，执行性能不高。非公平锁的优点是执行效率高，谁先获取到锁，锁就属于谁，不会“按资排辈”以及顺序唤醒，但缺点是资源分配随机性强，可能会出现线程饿死的情况。



#### 源码解析

```java
    public void updateByFairLock() {
        RLock fairLock = redissonClient.getFairLock("fairLockName");
        try {
            fairLock.lock();
            // business function
        } finally {
            fairLock.unlock();
        }
    }
```



##### getFairLock

> org.redisson.Redisson

```java
    @Override
    public RLock getFairLock(String name) {
        return new RedissonFairLock(connectionManager.getCommandExecutor(), name);
    }
```



##### lock

RedissonFairLock继承了RedissonLock，主要是重写了tryLockInnerAsync方法，也就是加锁逻辑的方法。

> org.redisson.RedissonFairLock

```java
    @Override
    <T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
        internalLockLeaseTime = unit.toMillis(leaseTime);

        long currentTime = System.currentTimeMillis();
        if (command == RedisCommands.EVAL_NULL_BOOLEAN) {
            return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
                    // remove stale threads
                    "while true do " +
                        "local firstThreadId2 = redis.call('lindex', KEYS[2], 0);" +
                        "if firstThreadId2 == false then " +
                            "break;" +
                        "end;" +
                        "local timeout = tonumber(redis.call('zscore', KEYS[3], firstThreadId2));" +
                        "if timeout <= tonumber(ARGV[3]) then " +
                            // remove the item from the queue and timeout set
                            // NOTE we do not alter any other timeout
                            "redis.call('zrem', KEYS[3], firstThreadId2);" +
                            "redis.call('lpop', KEYS[2]);" +
                        "else " +
                            "break;" +
                        "end;" +
                    "end;" +

                    "if (redis.call('exists', KEYS[1]) == 0) " +
                        "and ((redis.call('exists', KEYS[2]) == 0) " +
                            "or (redis.call('lindex', KEYS[2], 0) == ARGV[2])) then " +
                        "redis.call('lpop', KEYS[2]);" +
                        "redis.call('zrem', KEYS[3], ARGV[2]);" +

                        // decrease timeouts for all waiting in the queue
                        "local keys = redis.call('zrange', KEYS[3], 0, -1);" +
                        "for i = 1, #keys, 1 do " +
                            "redis.call('zincrby', KEYS[3], -tonumber(ARGV[4]), keys[i]);" +
                        "end;" +

                        "redis.call('hset', KEYS[1], ARGV[2], 1);" +
                        "redis.call('pexpire', KEYS[1], ARGV[1]);" +
                        "return nil;" +
                    "end;" +
                    "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                        "redis.call('hincrby', KEYS[1], ARGV[2], 1);" +
                        "redis.call('pexpire', KEYS[1], ARGV[1]);" +
                        "return nil;" +
                    "end;" +
                    "return 1;",
                    Arrays.<Object>asList(getName(), threadsQueueName, timeoutSetName),
                    internalLockLeaseTime, getLockName(threadId), currentTime, threadWaitTime);
        }

        if (command == RedisCommands.EVAL_LONG) {
            return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
                    // remove stale threads
                    "while true do " +
                        "local firstThreadId2 = redis.call('lindex', KEYS[2], 0);" +
                        "if firstThreadId2 == false then " +
                            "break;" +
                        "end;" +

                        "local timeout = tonumber(redis.call('zscore', KEYS[3], firstThreadId2));" +
                        "if timeout <= tonumber(ARGV[4]) then " +
                            // remove the item from the queue and timeout set
                            // NOTE we do not alter any other timeout
                            "redis.call('zrem', KEYS[3], firstThreadId2);" +
                            "redis.call('lpop', KEYS[2]);" +
                        "else " +
                            "break;" +
                        "end;" +
                    "end;" +

                    // check if the lock can be acquired now
                    "if (redis.call('exists', KEYS[1]) == 0) " +
                        "and ((redis.call('exists', KEYS[2]) == 0) " +
                            "or (redis.call('lindex', KEYS[2], 0) == ARGV[2])) then " +

                        // remove this thread from the queue and timeout set
                        "redis.call('lpop', KEYS[2]);" +
                        "redis.call('zrem', KEYS[3], ARGV[2]);" +

                        // decrease timeouts for all waiting in the queue
                        "local keys = redis.call('zrange', KEYS[3], 0, -1);" +
                        "for i = 1, #keys, 1 do " +
                            "redis.call('zincrby', KEYS[3], -tonumber(ARGV[3]), keys[i]);" +
                        "end;" +

                        // acquire the lock and set the TTL for the lease
                        "redis.call('hset', KEYS[1], ARGV[2], 1);" +
                        "redis.call('pexpire', KEYS[1], ARGV[1]);" +
                        "return nil;" +
                    "end;" +

                    // check if the lock is already held, and this is a re-entry
                    "if redis.call('hexists', KEYS[1], ARGV[2]) == 1 then " +
                        "redis.call('hincrby', KEYS[1], ARGV[2],1);" +
                        "redis.call('pexpire', KEYS[1], ARGV[1]);" +
                        "return nil;" +
                    "end;" +

                    // the lock cannot be acquired
                    // check if the thread is already in the queue
                    "local timeout = redis.call('zscore', KEYS[3], ARGV[2]);" +
                    "if timeout ~= false then " +
                        // the real timeout is the timeout of the prior thread
                        // in the queue, but this is approximately correct, and
                        // avoids having to traverse the queue
                        "return timeout - tonumber(ARGV[3]) - tonumber(ARGV[4]);" +
                    "end;" +

                    // add the thread to the queue at the end, and set its timeout in the timeout set to the timeout of
                    // the prior thread in the queue (or the timeout of the lock if the queue is empty) plus the
                    // threadWaitTime
                    "local lastThreadId = redis.call('lindex', KEYS[2], -1);" +
                    "local ttl;" +
                    "if lastThreadId ~= false and lastThreadId ~= ARGV[2] then " +
                        "ttl = tonumber(redis.call('zscore', KEYS[3], lastThreadId)) - tonumber(ARGV[4]);" +
                    "else " +
                        "ttl = redis.call('pttl', KEYS[1]);" +
                    "end;" +
                    "local timeout = ttl + tonumber(ARGV[3]) + tonumber(ARGV[4]);" +
                    "if redis.call('zadd', KEYS[3], timeout, ARGV[2]) == 1 then " +
                        "redis.call('rpush', KEYS[2], ARGV[2]);" +
                    "end;" +
                    "return ttl;",
                    Arrays.<Object>asList(getName(), threadsQueueName, timeoutSetName),
                    internalLockLeaseTime, getLockName(threadId), threadWaitTime, currentTime);
        }

        throw new IllegalArgumentException();
    }
```



当线程来加锁的时候，如果加锁失败了，那么会将线程扔到一个set集合中，这样就按照加锁的顺序给线程排队，set集合的头部的线程就代表了接下来能够加锁成功的线程。当有线程释放了锁之后，其它加锁失败的线程就会来继续加锁，加锁之前会先判断一下set集合的头部的线程跟当前要加锁的线程是不是同一个，如果是的话，那就加锁成功，如果不是的话，那么就加锁失败，这样就实现了加锁的顺序性。



### 读写锁

在实际的业务场景中，其实会有很多读多写少的场景，那么对于这种场景来说，使用独占锁来加锁，在高并发场景下会导致大量的线程加锁失败，阻塞，对系统的吞吐量有一定的影响，为了适配这种读多写少的场景，Redisson也实现了读写锁的功能。

读写锁的特点：

- 读与读是共享的，不互斥
- 读与写互斥
- 写与写互斥



#### 实现原理

Redisson通过RedissonReadWriteLock类来实现读写锁的功能，通过这个类可以获取到读锁或者写锁，所以真正的加锁的逻辑是由读锁和写锁实现的。

前面说过，加锁成功之后会在redis中维护一个hash的数据结构，存储加锁线程和加锁次数。在读写锁的实现中，会往hash数据结构中多维护一个mode的字段，来表示当前加锁的模式。

所以能够实现读写锁，最主要是因为维护了一个加锁模式的字段mode，这样有线程来加锁的时候，就能根据当前加锁的模式结合读写的特性来判断要不要让当前来加锁的线程加锁成功。

- 如果没有加锁，那么不论是读锁还是写锁都能加成功，成功之后根据锁的类型维护mode字段。
- 如果模式是读锁，那么加锁线程是来加读锁的，就让它加锁成功。
- 如果模式是读锁，那么加锁线程是来加写锁的，就让它加锁失败。
- 如果模式是写锁，那么加锁线程是来加写锁的，就让它加锁失败（加锁线程自己除外）。
- 如果模式是写锁，那么加锁线程是来加读锁的，就让它加锁失败（加锁线程自己除外）。



#### 源码解析

```java
		public void updateByReadLock() {
        RReadWriteLock readWriteLock = redissonClient.getReadWriteLock("readWriteLock");
        RLock readLock = readWriteLock.readLock();
        try {
            readLock.lock();
            // business function
        } finally {
            readLock.unlock();
        }
    }

    public void updateByWriteLock() {
        RReadWriteLock readWriteLock = redissonClient.getReadWriteLock("readWriteLock");
        RLock writeLock = readWriteLock.writeLock();
        try {
            writeLock.lock();
            // business function
        } finally {
            writeLock.unlock();
        }
    }
```



##### getReadWriteLock

> org.redisson.Redisson

```java
    @Override
    public RReadWriteLock getReadWriteLock(String name) {
        return new RedissonReadWriteLock(connectionManager.getCommandExecutor(), name);
    }
```



> org.redisson.RedissonReadWriteLock

```java
    @Override
    public RLock readLock() {
        return new RedissonReadLock(commandExecutor, getName());
    }

    @Override
    public RLock writeLock() {
        return new RedissonWriteLock(commandExecutor, getName());
    }
```



##### 添加读锁

> org.redisson.RedissonReadLock

```java
    @Override
    <T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
        internalLockLeaseTime = unit.toMillis(leaseTime);

        return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
                                "local mode = redis.call('hget', KEYS[1], 'mode'); " +
                                "if (mode == false) then " +
                                  "redis.call('hset', KEYS[1], 'mode', 'read'); " +
                                  "redis.call('hset', KEYS[1], ARGV[2], 1); " +
                                  "redis.call('set', KEYS[2] .. ':1', 1); " +
                                  "redis.call('pexpire', KEYS[2] .. ':1', ARGV[1]); " +
                                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                                  "return nil; " +
                                "end; " +
                                "if (mode == 'read') or (mode == 'write' and redis.call('hexists', KEYS[1], ARGV[3]) == 1) then " +
                                  "local ind = redis.call('hincrby', KEYS[1], ARGV[2], 1); " + 
                                  "local key = KEYS[2] .. ':' .. ind;" +
                                  "redis.call('set', key, 1); " +
                                  "redis.call('pexpire', key, ARGV[1]); " +
                                  "local remainTime = redis.call('pttl', KEYS[1]); " +
                                  "redis.call('pexpire', KEYS[1], math.max(remainTime, ARGV[1])); " +
                                  "return nil; " +
                                "end;" +
                                "return redis.call('pttl', KEYS[1]);",
                        Arrays.<Object>asList(getName(), getReadWriteTimeoutNamePrefix(threadId)), 
                        internalLockLeaseTime, getLockName(threadId), getWriteLockName(threadId));
    }
```



##### 释放读锁



##### 添加写锁

> org.redisson.RedissonWriteLock

```java
    @Override
    <T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
        internalLockLeaseTime = unit.toMillis(leaseTime);

        return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
                            "local mode = redis.call('hget', KEYS[1], 'mode'); " +
                            "if (mode == false) then " +
                                  "redis.call('hset', KEYS[1], 'mode', 'write'); " +
                                  "redis.call('hset', KEYS[1], ARGV[2], 1); " +
                                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                                  "return nil; " +
                              "end; " +
                              "if (mode == 'write') then " +
                                  "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                                      "redis.call('hincrby', KEYS[1], ARGV[2], 1); " + 
                                      "local currentExpire = redis.call('pttl', KEYS[1]); " +
                                      "redis.call('pexpire', KEYS[1], currentExpire + ARGV[1]); " +
                                      "return nil; " +
                                  "end; " +
                                "end;" +
                                "return redis.call('pttl', KEYS[1]);",
                        Arrays.<Object>asList(getName()), 
                        internalLockLeaseTime, getLockName(threadId));
    }
```



##### 释放写锁

```java
```



### 批量加锁

#### 实现原理

批量加锁的意思就是同时加几个锁，只有这些锁都算加成功了，才是真正的加锁成功。



#### 源码解析

```java
public void updateByMultiLock() {
    RLock lock1 = redissonClient.getLock("lock1");
    RLock lock2 = redissonClient.getLock("lock2");
    RLock lock3 = redissonClient.getLock("lock3");
    RLock multiLock = redissonClient.getMultiLock(lock1, lock2, lock3);
    try {
        multiLock.lock();
        // business function
    } finally {
        multiLock.unlock();
    }
}
```



##### getMultiLock

> org.redisson.Redisson

```java
@Override
public RLock getMultiLock(RLock... locks) {
    return new RedissonMultiLock(locks);
}
```



##### lock

> org.redisson.RedissonMultiLock

```java
@Override
public void lock() {
    try {
        lockInterruptibly();
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}

@Override
public void lockInterruptibly() throws InterruptedException {
    lockInterruptibly(-1, null);
}

@Override
public void lockInterruptibly(long leaseTime, TimeUnit unit) throws InterruptedException {
    long baseWaitTime = locks.size() * 1500;
    long waitTime = -1;
    if (leaseTime == -1) {
        waitTime = baseWaitTime;
    } else {
        leaseTime = unit.toMillis(leaseTime);
        waitTime = leaseTime;
        if (waitTime <= 2000) {
            waitTime = 2000;
        } else if (waitTime <= baseWaitTime) {
            waitTime = ThreadLocalRandom.current().nextLong(waitTime/2, waitTime);
        } else {
            waitTime = ThreadLocalRandom.current().nextLong(baseWaitTime, waitTime);
        }
    }
    
    while (true) {
        if (tryLock(waitTime, leaseTime, TimeUnit.MILLISECONDS)) {
            return;
        }
    }
}

@Override
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
    long newLeaseTime = -1;
    if (leaseTime != -1) {
        if (waitTime == -1) {
            newLeaseTime = unit.toMillis(leaseTime);
        } else {
            newLeaseTime = unit.toMillis(waitTime)*2;
        }
    }
    
    long time = System.currentTimeMillis();
    long remainTime = -1;
    if (waitTime != -1) {
        remainTime = unit.toMillis(waitTime);
    }
    long lockWaitTime = calcLockWaitTime(remainTime);
    
    int failedLocksLimit = failedLocksLimit();
    List<RLock> acquiredLocks = new ArrayList<>(locks.size());
    // 根据顺序去依次调用传入myLock1、myLock2、myLock3 加锁方法，然后如果都成功加锁了，那么multiLock就算加锁成功。
    for (ListIterator<RLock> iterator = locks.listIterator(); iterator.hasNext();) {
        RLock lock = iterator.next();
        boolean lockAcquired;
        try {
            if (waitTime == -1 && leaseTime == -1) {
                // 加锁
                lockAcquired = lock.tryLock();
            } else {
                long awaitTime = Math.min(lockWaitTime, remainTime);
                // 加锁
                lockAcquired = lock.tryLock(awaitTime, newLeaseTime, TimeUnit.MILLISECONDS);
            }
        } catch (RedisResponseTimeoutException e) {
            unlockInner(Arrays.asList(lock));
            lockAcquired = false;
        } catch (Exception e) {
            lockAcquired = false;
        }
        
        if (lockAcquired) {
            acquiredLocks.add(lock);
        } else {
            if (locks.size() - acquiredLocks.size() == failedLocksLimit()) {
                break;
            }

            if (failedLocksLimit == 0) {
                unlockInner(acquiredLocks);
                if (waitTime == -1) {
                    return false;
                }
                failedLocksLimit = failedLocksLimit();
                acquiredLocks.clear();
                // reset iterator
                while (iterator.hasPrevious()) {
                    iterator.previous();
                }
            } else {
                failedLocksLimit--;
            }
        }
        
        if (remainTime != -1) {
            remainTime -= System.currentTimeMillis() - time;
            time = System.currentTimeMillis();
            if (remainTime <= 0) {
                unlockInner(acquiredLocks);
                return false;
            }
        }
    }

    if (leaseTime != -1) {
        List<RFuture<Boolean>> futures = new ArrayList<>(acquiredLocks.size());
        for (RLock rLock : acquiredLocks) {
            RFuture<Boolean> future = ((RedissonLock) rLock).expireAsync(unit.toMillis(leaseTime), TimeUnit.MILLISECONDS);
            futures.add(future);
        }
        
        for (RFuture<Boolean> rFuture : futures) {
            rFuture.syncUninterruptibly();
        }
    }
    
    return true;
}
```



### 红锁

对于单Redis实例来说，如果Redis宕机了，那么整个系统就无法工作了。所以为了保证Redis的高可用性，一般会使用主从或者哨兵模式。但是如果使用了主从或者哨兵模式，此时Redis的分布式锁的功能可能就会出现问题。

基于这种模式，Redis客户端会在master节点上加锁，然后异步复制给slave节点。但是因为一些原因，master节点宕机了，那么哨兵节点感知到了master节点宕机了，那么就会从slave节点选择一个节点作为主节点，实现主从切换。

这种情况看似没什么问题，但是不幸的事发生了，那就是客户端对原先的主节点加锁，加成之后还没有来得及同步给从节点，主节点宕机了，从节点变成了主节点，此时从节点是没有加锁信息的，如果有其它的客户端来加锁，是能够加锁成功的，这不是很坑爹么。



#### 实现原理

让客户端与多个独立的 Redis 节点并行请求申请加锁，如果能在半数以上的节点成功地完成加锁操作，那么我们就认为，客户端成功地获得分布式锁，否则加锁失败。

在Redis的分布式环境中，我们假设有N个Redis master。这些节点完全互相独立，不存在主从复制或者其他集群协调机制。之前我们已经描述了在Redis单实例下怎么安全地获取和释放锁。我们确保将在每（N)个实例上使用此方法获取和释放锁。在这个样例中，我们假设有5个Redis master节点，这是一个比较合理的设置，所以我们需要在5台机器上面或者5台虚拟机上面运行这些实例，这样保证他们不会同时都宕掉。

为了取到锁，客户端应该执行以下操作:

1. 获取当前Unix时间，以毫秒为单位。
2. 依次尝试从N个实例，使用相同的key和随机值获取锁。在步骤2，当向Redis设置锁时,客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为10秒，则超时时间应该在5-50毫秒之间。这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试另外一个Redis实例。
3. 客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。当且仅当从大多数（这里是3个节点）的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功。
4. 如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。
5. 如果因为某些原因，获取锁失败（没有在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间），客户端应该在所有的Redis实例上进行解锁（即便某些Redis实例根本就没有加锁成功）。



RedissonRedLock加锁过程如下：

- 获取所有的redisson node节点信息，循环向所有的redisson node节点加锁，假设节点数为N，例子中N等于5。一个redisson node代表一个主从节点。
- 如果在N个节点当中，有N/2 + 1个节点加锁成功了，那么整个RedissonRedLock加锁是成功的。
- 如果在N个节点当中，小于N/2 + 1个节点加锁成功，那么整个RedissonRedLock加锁是失败的。
- 如果中途发现各个节点加锁的总耗时，大于等于设置的最大等待时间，则直接返回失败。

RedissonRedLock底层其实也就基于RedissonMultiLock实现的，RedissonMultiLock要求所有的加锁成功才算成功，RedissonRedLock要求只要有N/2 + 1个成功就算成功。



##### 问题

1. 需要额外搭建多套环境，申请更多的资源，需要评估一下成本和性价比。
2. 如果有N个redisson node节点，需要加锁N次，最少也需要加锁N/2+1次，才知道redlock加锁是否成功。显然，增加了额外的时间成本，有点得不偿失。



#### 源码解析

```java
    public void updateByRedLock() {
        RLock lock1 = redissonClient.getLock("lock1");
        RLock lock2 = redissonClient.getLock("lock2");
        RLock lock3 = redissonClient.getLock("lock3");
        RedissonRedLock redissonRedLock = new RedissonRedLock(lock1, lock2, lock3);
        try {
            redissonRedLock.lock();
            // business function
        } finally {
            redissonRedLock.unlock();
        }
    }
```



### 总结

优点：

- 性能好，适合高并发场景
- 较轻量级
- 有较好的框架支持，如 Redisson



缺点：

- 过期时间不好控制
- 需要考虑锁被别的线程误删场景



### 参考资料

- [Redission Github](https://github.com/redisson/redisson/wiki/%E7%9B%AE%E5%BD%95)



## Zookeeper分布式锁

### 总结

缺点：

- 性能不如 Redis 实现的分布式锁
- 比较重的分布式锁。



优点：

- 有较好的性能和可靠性
- 有封装较好的框架，如 Curator



## 分布式锁优化

### 分段加锁

>  在java中`ConcurrentHashMap`，就是将数据分为`16段`，每一段都有单独的锁，并且处于不同锁段的数据互不干扰，以此来提升锁的性能。



将锁资源分段加锁，在多线程环境中针不同分段的锁进行竞争加锁，从而避免竞争一把锁，提升系统吞吐量。

将锁分段虽说可以提升系统的性能，但它也会让系统的复杂度提升不少。因为它需要引入额外的路由算法，跨段统计等功能。



#### 实现示例

将库存分段，比如分为100段，这样每段就有20个商品可以参与秒杀。

在秒杀的过程中，先把用户id获取hash值，然后除以100取模。模为1的用户访问第1段库存，模为2的用户访问第2段库存，模为3的用户访问第3段库存，后面以此类推，到最后模为100的用户访问第100段库存。



## 分布式锁总结

- 从性能角度（从高到低）Redis > Zookeeper >= 数据库
- 从理解的难易程度角度（从低到高）数据库 > Redis > Zookeeper
- 从实现的复杂性角度（从低到高）Zookeeper > Redis > 数据库
- 从可靠性角度（从高到低）Zookeeper > Redis > 数据库



如果实际业务场景更需要的是保证数据一致性。那么请使用CP类型的分布式锁，比如：zookeeper，它是基于磁盘的，性能可能没那么好，但数据一般不会丢。

如果实际业务场景更需要的是保证数据高可用性。那么请使用AP类型的分布式锁，比如：redis，它是基于内存的，性能比较好，但有丢失数据的风险。

绝大多数分布式业务场景中，使用redis分布式锁就够了。因为数据不一致问题，可以通过最终一致性方案解决。系统不可用对用户来说更加不友好。



# 分布式事务

## 概述

分布式事务就是为了保证不同数据库的数据一致性，不同服务中的操作要么全部成功，要么全部失败。典型的分布式事务场景：跨银行转操作就涉及调用两个异地银行服务。

分布式锁解决的是分布式资源抢占的问题；分布式事务和本地事务是解决流程化提交问题。



![在这里插入图片描述](../../Image/2022/07/220729-1.png)



对于分布式系统而言，需要保证分布式系统中的数据一致性，保证数据在子系统中始终保持一致，避免业务出现问题。分布式系统中对数要么一起成功，要么一起失败，必须是一个整体性的事务。

分布式事务指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于不同的分布式系统的不同节点之上。

简单的说，在分布式系统上一次大的操作由不同的小操作组成，这些小的操作分布在不同的服务节点上，且属于不同的应用，分布式事务需要保证这些小操作要么全部成功，要么全部失败。



### 基础概念

- 事务：事务是由一组操作构成的可靠的独立的工作单元，事务具备ACID的特性，即原子性、一致性、隔离性和持久性。
- 本地事务：当事务由资源管理器本地管理时被称作本地事务。本地事务的优点就是支持严格的ACID特性，高效，可靠，状态可以只在资源管理器中维护，而且应用编程模型简单。但是本地事务不具备分布式事务的处理能力，隔离的最小单位受限于资源管理器。
- 全局事务：当事务由全局事务管理器进行全局管理时成为全局事务，事务管理器负责管理全局的事务状态和参与的资源，协同资源的一致提交回滚。
- TX协议：应用或者应用服务器与事务管理器的接口。
- XA协议：全局事务管理器与资源管理器的接口。XA是由X/Open组织提出的分布式事务规范。该规范主要定义了全局事务管理器和局部资源管理器之间的接口。主流的数据库产品都实现了XA接口。XA接口是一个双向的系统接口，在事务管理器以及多个资源管理器之间作为通信桥梁。之所以需要XA是因为在分布式系统中从理论上讲两台机器是无法达到一致性状态的，因此引入一个单点进行协调。由全局事务管理器管理和协调的事务可以跨越多个资源和进程。全局事务管理器一般使用XA二阶段协议与数据库进行交互。
- AP：应用程序，可以理解为使用DTP（Data Tools Platform）的程序。
- RM：资源管理器，这里可以是一个DBMS或者消息服务器管理系统，应用程序通过资源管理器对资源进行控制，资源必须实现XA定义的接口。资源管理器负责控制和管理实际的资源。
- TM：事务管理器，负责协调和管理事务，提供给AP编程接口以及管理资源管理器。事务管理器控制着全局事务，管理事务的生命周期，并且协调资源。
- 两阶段提交协议：XA用于在全局事务中协调多个资源的机制。TM和RM之间采取两阶段提交的方案来解决一致性问题。两节点提交需要一个协调者（TM）来掌控所有参与者（RM）节点的操作结果并且指引这些节点是否需要最终提交。两阶段提交的局限在于协议成本，准备阶段的持久成本，全局事务状态的持久成本，潜在故障点多带来的脆弱性，准备后，提交前的故障引发一系列隔离与恢复难题。
- BASE理论：BA指的是基本业务可用性，支持分区失败，S表示柔性状态，也就是允许短时间内不同步，E表示最终一致性，数据最终是一致的，但是实时是不一致的。原子性和持久性必须从根本上保障，为了可用性、性能和服务降级的需要，只有降低一致性和隔离性的要求。
- CAP定理：对于共享数据系统，最多只能同时拥有CAP其中的两个，任意两个都有其适应的场景，真是的业务系统中通常是ACID与CAP的混合体。分布式系统中最重要的是满足业务需求，而不是追求高度抽象，绝对的系统特性。C表示一致性，也就是所有用户看到的数据是一样的。A表示可用性，是指总能找到一个可用的数据副本。P表示分区容错性，能够容忍网络中断等故障。



### 分布式一致性

分布式一致性的根本原因在于数据的分布式操作，引起的本地事务无法保障数据的原子性引起。

分布式一致性问题的解决思路有两种，一种是分布式事务，一种是尽量通过业务流程避免分布式事务。分布式事务是直接解决问题，而业务规避其实通过解决出问题的地方(解决提问题的人)。其实在真实业务场景中，如果业务规避不是很麻烦的前提，最优雅的解决方案就是业务规避。



### 事务种类

#### 柔性事务

柔性事务指的是，不要求强一致性，而是要求最终一致性，允许有中间状态，也就是Base理论，换句话说，就是AP状态。

与刚性事务相比，柔性事务的特点为：有业务改造，最终一致性，实现补偿接口，实现资源锁定接口，高并发，适合长事务。

柔性事务有两个特性：基本可用和柔性状态。

- 基本可用是指分布式系统出现故障的时候允许损失一部分的可用性。
- 柔性状态是指允许系统存在中间状态，这个中间状态不会影响系统整体的可用性，比如数据库读写分离的主从同步延迟等。柔性事务的一致性指的是最终一致性。



##### 补偿型柔性事务

> 补偿模式使用一个额外的协调服务来协调各个需要保证一致性的业务服务，协调服务按顺序调用各个业务微服务，如果某个业务服务调用异常（包括业务异常和技术异常）就取消之前所有已经调用成功的业务服务。



补偿型事务都是同步的，如TCC/FMT、Saga（状态机模式、Aop模式）



##### 通知型柔性事务

通知型事务的主流实现是通过MQ（消息队列）来通知其他事务参与者自己事务的执行状态，引入MQ组件，有效的将事务参与者进行解耦，各参与者都可以异步执行，所以通知型事务又被称为**异步事务**。

通知型事务主要适用于那些需要异步更新数据，并且对数据的实时性要求较低的场景。主要包含**异步确保型事务**和**最大努力通知事务**两种。



- **异步确保型事务**：主要适用于内部系统的数据最终一致性保障，因为内部相对比较可控，如订单和购物车、收货与清算、支付与结算等等场景；
- **最大努力通知**：主要用于外部系统，因为外部的网络环境更加复杂和不可信，所以只能尽最大努力去通知实现数据最终一致性，比如充值平台与运营商、支付对接等等跨网络系统级别对接；



通知型事务都是异步的，如MQ事务消息、最大努力通知型。



#### 刚性事务

刚性事务指的是，要使分布式事务，达到像本地式事务一样，具备数据强一致性，从CAP来看，就是说，要达到CP状态。通常无业务改造，强一致性，原生支持回滚/隔离性，低并发，适合短事务。



刚性事务：XA 协议（2PC、JTA、JTS）、3PC，但由于同步阻塞，处理效率低，不适合大型网站分布式场景。



##### 刚性事务缺陷

1. 数据锁定：数据在事务未结束前，为了保障一致性，根据数据隔离级别进行锁定。
2. 协议阻塞：本地事务在全局事务 没 commit 或 callback前都是阻塞等待的。
3. 性能损耗高：主要体现在事务协调增加的RT成本，并发事务数据使用锁进行竞争阻塞。



### 产生原因

- **跨库事务**

跨库事务指的是，一个应用某个功能需要操作多个库，不同的库中存储不同的业务数据。



- **数据库分库分表**

当数据库单表数据达到千万级别，就要考虑分库分表，那么就会从原来的一个数据库变成多个数据库。例如如果一个操作即操作了01库，又操作了02库，而且又要保证数据的一致性，那么就要用到分布式事务。



- **应用微服务化**

所谓的SOA化，就是业务的服务化。例如电商平台下单操作就会产生调用库存服务扣减库存和订单服务更新订单数据，那么就会设计到订单数据库和库存数据库，为了保证数据的一致性，就需要用到分布式事务。



## 一致性协议



### Paxos协议



### Raft协议



### ZAB协议



## 刚性事务方案

### 2PC提交

二阶段提交（Two-Phase Commit），是指将事务提交分成两个部分：准备阶段和提交阶段。事务的发起者称之为协调者，事务的执行者称为参与者。



#### 执行流程

##### 准备阶段

- 由协调者（coordinator）发起并传递带有事务信息的请求给各个参与者（participants），询问是否可以提交事务，并等待返回结果；
- 每个参与者执行事务操作，将Undo和Redo放入事务日志中（但是不提交）；
- 各参与者向协调者反馈事务问询的响应，执行成功返回YES（可以提交事务），失败NO(不能提交事务)。



##### 提交阶段

###### 正常提交

所有参与者均反馈YES时，即提交事务；

1. 协调者节点通知所有的参与者Commit事务请求；
2. 参与者收到Commit请求之后，就会正式执行本地事务Commit操作，并在完成提交之后释放整个事务执行期间占用的事务资源；
3. 反馈事务提交结果。参与者在完成事务提交后，写协调者发送Ack消息确认；
4. 完成事务。协调者在收到所有参与者的Ack后，完成事务。



###### 异常回滚

任何一个参与者反馈NO时，即中断事务。

1. 协调者节点通知所有的参与者Rollback请求；
2. 参与者收到Rollback请求之后，利用本机Undo信息，就会正式执行本地事务Rollback操作，并在完成提交之后释放整个事务执行期间占用的事务资源。
3. 反馈回滚结果。参与者在完成回滚操作后，向协调者发送Ack消息；
4. 中断事务。协调者收到所有参与者的回滚Ack消息后，完成事务中断。



#### 优缺点

##### 优点

实现简单。



##### 缺点

###### 性能问题

无论是在第一阶段的过程中,还是在第二阶段,所有的参与者资源和协调者资源都是被锁住的,只有当所有节点准备完毕，事务 协调者 才会通知进行全局提交，参与者进行本地事务提交后才会释放资源。这样的过程会比较漫长，对性能影响比较大。

2PC的提交在执行过程中，所有参与事务操作的逻辑都处于阻塞状态，也就是说，各个参与者都在等待其他参与者响应，无法进行其他操作；



###### 节点故障

由于协调者的重要性，一旦协调者发生故障。参与者会一直阻塞下去。尤其在第二阶段，协调者发生故障，那么所有的参与者还都处于锁定事务资源的状态中，而无法继续完成事务操作。（虽然协调者挂掉，可以重新选举一个协调者，但是无法解决因为协调者宕机导致的参与者处于阻塞状态的问题）。

协调者是个单点，一旦出现问题，其他参与者将无法释放事务资源，也无法完成事务操作；



**协调者正常,参与者宕机**

由于协调者无法收集到所有参与者的反馈，会陷入阻塞情况。解决方案:引入超时机制,如果协调者在超过指定的时间还没有收到参与者的反馈,事务就失败,向所有节点发送终止事务请求。



**协调者宕机,参与者正常**

无论处于哪个阶段，由于协调者宕机，无法发送提交请求，所有处于执行了操作但是未提交状态的参与者都会陷入阻塞情况。解决方案:引入协调者备份,同时协调者需记录操作日志.当检测到协调者宕机一段时间后，协调者备份取代协调者，并读取操作日志，向所有参与者询问状态。



**协调者和参与者都宕机**

**如果发生在第一阶段**： 因为第一阶段，所有参与者都没有真正执行commit，所以只需重新在剩余的参与者中重新选出一个协调者，新的协调者在重新执行第一阶段和第二阶段就可以了。

**发生在第二阶段并且挂了的参与者在挂掉之前没有收到协调者的指令**：也就是上面的第2步挂了，这是可能协调者还没有发送第2步就挂了。这种情形下，新的协调者重新执行第一阶段和第二阶段操作。

**发生在第二阶段并且有部分参与者已经执行完commit操作**：就好比这里订单服务A和支付服务B都收到协调者发送的commit信息，开始真正执行本地事务commit,但突发情况，A commit成功，B挂了。这个时候目前来讲数据是不一致的。虽然这个时候可以再通过手段让他和协调者通信，再想办法把数据搞成一致的，但是，这段时间内他的数据状态已经是不一致的了！ 2PC无法解决这个问题。



###### 数据不一致

当执行事务提交过程中，如果协调者向所有参与者发送Commit请求后，发生局部网络异常或者协调者在尚未发送完Commit请求，即出现崩溃，最终导致只有部分参与者收到、执行请求。于是整个系统将会出现数据不一致的情形；



###### 保守

2PC没有完善的容错机制，当参与者出现故障时，协调者无法快速得知这一失败，只能严格依赖超时设置来决定是否进一步的执行提交还是中断事务。



#### 总结

2PC 方案比较适合单体应用里，跨多个库的分布式事务，而且因为严重依赖于数据库层面来搞定复杂的事务，效率很低，绝对不适合高并发的场景。

2PC 方案实际很少用，一般来说某个系统内部如果出现跨多个库的这么一个操作，是不合规的。我可以给大家介绍一下， 现在微服务，一个大的系统分成几百个服务，几十个服务。一般来说，我们的规定和规范，是要求每个服务只能操作自己对应的一个数据库。

如果你要操作别的服务对应的库，不允许直连别的服务的库，违反微服务架构的规范，你随便交叉胡乱访问，几百个服务的话，全体乱套，这样的一套服务是没法管理的，没法治理的，可能会出现数据被别人改错，自己的库被别人写挂等情况。

如果你要操作别人的服务的库，你必须是通过调用别的服务的接口来实现，绝对不允许交叉访问别人的数据库。



### 3PC提交

三阶段提交（Three-Phase Commit）是二阶段提交（2PC）的改进版本，三阶段提交协议主要是为了解决两阶段提交协议的阻塞问题，2pc存在的问题是当协调者崩溃时，参与者不能做出最后的选择。因此参与者可能在协调者恢复之前保持阻塞。三阶段提交（Three-phase commit），是二阶段提交（2PC）的改进版本。

与两阶段提交不同的是，三阶段提交有两个改动点：

- 引入超时机制。同时在协调者和参与者中都引入超时机制；
- 在第一阶段和第二阶段中插入一个准备阶段。保证了在最后提交阶段之前各参与节点的状态是一致的。



三阶段提交协议将事务的提交过程分为CanCommit、PreCommit、do Commit三个阶段来进行处理。



#### 执行流程

![三阶段提交示意图](../../Image/2022/07/220731-1.png)

##### CanCommit

之前2PC的一阶段是本地事务执行结束后，最后不Commit，等其它服务都执行结束并返回Yes，由协调者发生commit才真正执行commit。而这里的CanCommit指的是 尝试获取数据库锁 如果可以，就返回Yes。

这阶段主要分为2步：

**事务询问**：协调者向所有参与者发出包含事务内容的CanCommit请求，询问是否可以提交事务，并等待所有参与者答复。

**响应反馈**：参与者收到CanCommit请求后，如果认为可以执行事务操作，则反馈YES并进入预备状态，否则反馈NO。



##### PreCommit

###### 事务预提交

所有参与者均受到请求并返回YES时，协调者向所有参与者发出PreCommit请求，进入准备阶段，

- 发送预提交请求。协调者向所有节点发出PreCommit请求，并进入prepared阶段；

- 事务预提交：参与者接收到PreCommit请求后，会执行事务操作，将undo和redo信息记录到事务日志中。
- 响应反馈：如果参与者成功的执行了事务操作，则返回ACK响应，同时开始等待最终指令。



###### 事务回滚

有任何一个参与者返回NO或超时，协调者无法收到反馈，则事务中断，协调者向所有参与者发出abort请求。无论收到协调者发出的abort请求，或者在等待协调者请求过程中出现超时，参与者均会中断事务。

- 发送中断请求：协调者向所有参与者发送abort请求。
- 中断事务：参与者收到来自协调者的abort请求之后或超时，执行事务的中断。



##### DoCommit

该阶段进行真正的事务提交，也可以分为以下两种情况。



###### **执行提交**

- 发送提交请求。假如协调者收到了所有参与者的Ack响应，那么将从预提交转换到提交状态，并向所有参与者，发送doCommit请求；

- 事务提交：参与者接收到doCommit请求之后，执行正式的事务提交。并在完成事务提交之后释放所有事务资源。
- 响应反馈：事务提交完之后，向协调者发送Ack响应。
- 完成事务：协调者接收到所有参与者的ack响应之后，完成事务。



###### **中断事务**

协调者没有接收到参与者发送的ACK响应（可能是接受者发送的不是ACK响应，也可能响应超时），那么就会执行中断事务。

- 发送中断请求：协调者向所有参与者发送abort请求
- 事务回滚：参与者接收到abort请求之后，利用其在阶段二记录的undo信息来执行事务的回滚操作，并在完成回滚之后释放所有的事务资源。
- 反馈结果：参与者完成事务回滚之后，向协调者发送ACK消息
- 中断事务：协调者接收到参与者反馈的ACK消息之后，执行事务的中断。



#### 优缺点

##### 优点

3PC有效降低了2PC带来的参与者阻塞范围，并且能够在出现单点故障后继续达成一致；

相对于2PC，3PC主要解决的单点故障问题，并减少阻塞，因为一旦参与者无法及时收到来自协调者的信息之后，他会默认执行commit。而不会一直持有事务资源并处于阻塞状态。

但是这种机制也会导致数据一致性问题，因为，由于网络原因，协调者发送的abort响应没有及时被参与者接收到，那么参与者在等待超时之后执行了commit操作。这样就和其他接到abort命令并执行回滚的参与者之间存在数据不一致的情况。



参与者如果在不同阶段宕机，我们来看看3PC如何应对：

- 阶段1: 协调者或协调者备份未收到宕机参与者的vote，直接中止事务；宕机的参与者恢复后，读取logging发现未发出赞成vote，自行中止该次事务
- 阶段2: 协调者未收到宕机参与者的precommit ACK，但因为之前已经收到了宕机参与者的赞成反馈(不然也不会进入到阶段2)，协调者进行commit；协调者备份可以通过问询其他参与者获得这些信息，过程同理；宕机的参与者恢复后发现收到precommit或已经发出赞成vote，则自行commit该次事务
- 阶段3: 即便协调者或协调者备份未收到宕机参与者t的commit ACK，也结束该次事务；宕机的参与者恢复后发现收到commit或者precommit，也将自行commit该次事务。



##### 缺点

###### 数据不一致

但3PC带来了新的问题，在参与者收到preCommit消息后，如果网络出现分区，协调者和参与者无法进行后续的通信，这种情况下，参与者在等待超时后，依旧会执行事务提交，这样会导致数据的不一致。

但是这种机制也会导致数据一致性问题，因为，由于网络原因，协调者发送的abort响应没有及时被参与者接收到，那么参与者在等待超时之后执行了commit操作。这样就和其他接到abort命令并执行回滚的参与者之间存在数据不一致的情况。

在2PC中一个参与者的状态只有它自己和协调者知晓，假如协调者提议后自身宕机，在协调者备份启用前一个参与者又宕机，其他参与者就会进入既不能回滚、又不能强制commit的阻塞状态，直到参与者宕机恢复。



#### 扩展

##### 与2PC比较

三阶段提交协议在协调者和参与者中都引入 **超时机制**，并且把两阶段提交协议的第一个阶段拆分成了两步：询问，然后再锁资源，最后真正提交。

相比较2PC而言，3PC对于协调者（Coordinator）和参与者（Partcipant）都设置了超时时间，而2PC只有协调者才拥有超时机制。这解决了一个什么问题呢？

这个优化点，主要是避免了参与者在长时间无法与协调者节点通讯（协调者挂掉了）的情况下，无法释放资源的问题，因为参与者自身拥有超时机制会在超时后，自动进行本地commit从而进行释放资源。而这种机制也侧面降低了整个事务的阻塞时间和范围。

另外，通过CanCommit、PreCommit、DoCommit三个阶段的设计，相较于2PC而言，多设置了一个缓冲阶段保证了在最后提交阶段之前各参与节点的状态是一致的。



## 柔性事务方案

### Saga



### TCC

TCC将事务提交分为Try-Confirm-Cancel 3个操作。

- Try：完成业务检查、预留业务资源；
- Confirm：使用预留的资源执行业务操作（需要保证幂等性）；
- Cancel：取消执行业务操作，释放预留的资源（需要保证幂等性）。

TCC事务处理流程和 2PC 二阶段提交类似，不过2PC通常都是在跨库的DB层面，而TCC本质就是一个应用层面的2PC，如下为TCC原理图。

![TCC概览](../../Image/2022/07/220720-49.png)



优点：XA两阶段提交资源层面的，而TCC实际上把资源层面二阶段提交上提到了业务层面来实现。有效了的避免了XA两阶段提交占用资源锁时间过长导致的性能地下问题。



缺点：主业务服务和从业务服务都需要进行改造，从业务方改造成本更高。以上文中的订单服务为例，2PC中只需要提供一个下单接口即可，而TCC中缺需要提供Try-Confirm-Cancel三个接口，大大增加了开发量。



国内厂商在TCC实战中，提出了三种TCC变种实现：

- 通用型TCC，如果我们上面介绍的TCC模型实例，从业务服务需要提供try、confirm、cancel
- 补偿性TCC，从业务服务只需要提供Do和Compensate两个接口
- 异步确保型TCC，主业务服务的直接从业务服务是可靠消息服务，而真正的从业务服务则通过消息服务解耦，作为消息服务的消费端，异步地执行。



**TCC 分布式事务模型包括三部分：**

1.**主业务服务**：主业务服务为整个业务活动的发起方，服务的编排者，负责发起并完成整个业务活动。

2.**从业务服务**：从业务服务是整个业务活动的参与方，负责提供 TCC 业务操作，实现初步操作(Try)、确认操作(Confirm)、取消操作(Cancel)三个接口，供主业务服务调用。

3.**业务活动管理器**：业务活动管理器管理控制整个业务活动，包括记录维护 TCC 全局事务的事务状态和每个从业务服务的子事务状态，并在业务活动提交时调用所有从业务服务的 Confirm 操作，在业务活动取消时调用所有从业务服务的 Cancel 操作。



TCC 提出了一种新的事务模型，基于业务层面的事务定义，锁粒度完全由业务自己控制，目的是解决复杂业务中，跨表跨库等大颗粒度资源锁定的问题。

TCC 把事务运行过程分成 Try、Confirm / Cancel 两个阶段，每个阶段的逻辑由业务代码控制，避免了长事务，可以获取更高的性能。



#### 工作流程

TCC(Try-Confirm-Cancel)分布式事务模型相对于 XA 等传统模型，其特征在于**它不依赖资源管理器(RM)对分布式事务的支持，而是通过对业务逻辑的分解来实现分布式事务**。

**TCC 模型认为对于业务系统中一个特定的业务逻辑，其对外提供服务时，必须接受一些不确定性，即对业务逻辑初步操作的调用仅是一个临时性操作，调用它的主业务服务保留了后续的取消权。如果主业务服务认为全局事务应该回滚，它会要求取消之前的临时性操作，这就对应从业务服务的取消操作。而当主业务服务认为全局事务应该提交时，它会放弃之前临时性操作的取消权，这对应从业务服务的确认操作。每一个初步操作，最终都会被确认或取消。**

因此，针对一个具体的业务服务，TCC 分布式事务模型需要业务系统提供三段业务逻辑：

**初步操作 Try**：完成所有业务检查，预留必须的业务资源。

**确认操作 Confirm**：真正执行的业务逻辑，不作任何业务检查，只使用 Try 阶段预留的业务资源。因此，只要 Try 操作成功，Confirm 必须能成功。另外，Confirm 操作需满足幂等性，保证一笔分布式事务有且只能成功一次。

**取消操作 Cancel**：释放 Try 阶段预留的业务资源。同样的，Cancel 操作也需要满足幂等性。



![image.png](../../Image/2022/07/220731-5.png)



**Try 阶段**： 调用 Try 接口，尝试执行业务，完成所有业务检查，预留业务资源。

**Confirm 或 Cancel 阶段**： 两者是互斥的，只能进入其中一个，并且都满足幂等性，允许失败重试。

**Confirm 操作**： 对业务系统做确认提交，确认执行业务操作，不做其他业务检查，只使用 Try 阶段预留的业务资源。
**Cancel 操作**： 在业务执行错误，需要回滚的状态下执行业务取消，释放预留资源。

*Try 阶段失败可以 Cancel，如果 Confirm 和 Cancel 阶段失败了怎么办？*

TCC 中会添加事务日志，如果 Confirm 或者 Cancel 阶段出错，则会进行重试，所以这两个阶段需要支持幂等；如果重试失败，则需要人工介入进行恢复和处理等。



#### TCC事务要求

1. 可查询操作：服务操作具有全局唯一的标识，操作唯一的确定的时间。
2. 幂等操作：重复调用多次产生的业务结果与调用一次产生的结果相同。一是通过业务操作实现幂等性，二是系统缓存所有请求与处理的结果，最后是检测到重复请求之后，自动返回之前的处理结果。
3. TCC操作：Try阶段，尝试执行业务，完成所有业务的检查，实现一致性；预留必须的业务资源，实现准隔离性。Confirm阶段：真正的去执行业务，不做任何检查，仅适用Try阶段预留的业务资源，Confirm操作还要满足幂等性。Cancel阶段：取消执行业务，释放Try阶段预留的业务资源，Cancel操作要满足幂等性。TCC与2PC(两阶段提交)协议的区别：TCC位于业务服务层而不是资源层，TCC没有单独准备阶段，Try操作兼备资源操作与准备的能力，TCC中Try操作可以灵活的选择业务资源，锁定粒度。TCC的开发成本比2PC高。实际上TCC也属于两阶段操作，但是TCC不等同于2PC操作。
4. 可补偿操作：Do阶段：真正的执行业务处理，业务处理结果外部可见。Compensate阶段：抵消或者部分撤销正向业务操作的业务结果，补偿操作满足幂等性。约束：补偿操作在业务上可行，由于业务执行结果未隔离或者补偿不完整带来的风险与成本可控。实际上，TCC的Confirm和Cancel操作可以看做是补偿操作。



### MQ事务消息

消息发送一致性：是指产生消息的业务动作与消息发送的一致。也就是说，如果业务操作成功，那么由这个业务操作所产生的消息一定要成功投递出去(一般是发送到kafka、rocketmq、rabbitmq等消息中间件中)，否则就丢消息。



#### 消息不可靠性

既然提到了可靠消息的最终一致性，那么说明现有的消息发送逻辑存在不可靠性，我们用下面的几种情况来演示消息的不可靠性。



**先进行数据库操作，再发送消息**

这种情况下无法保证数据库操作与发送消息的一致性，因为可能数据库操作成功，发送消息失败。



**先发送消息，再操作数据库**

这种情况下无法保证数据库操作与发送消息的一致性，因为可能发送消息成功，数据库操作失败。



**在数据库事务中，先发送消息，后操作数据库**

这里使用spring 的@Transactional注解，方法里面的操作都在一个事务中。同样无法保证一致性，因为发送消息成功了，数据库操作失败的情况下，数据库操作是回滚了，但是MQ消息没法进行回滚。



**在数据库事务中，先操作数据库，后发送消息**

这种情况下，貌似没有问题，如果发送MQ消息失败，抛出异常，事务一定会回滚(加上了@Transactional注解后，spring方法抛出异常后，会自动进行回滚)。

这只是一个假象，因为发送MQ消息可能事实上已经成功，如果是响应超时导致的异常。这个时候，数据库操作依然回滚，但是MQ消息实际上已经发送成功，导致不一致。



#### JTA事务管理器

前面通过spring的@Transactional注解加在方法上，来开启事务。其实有一个条件没有明确的说出来，就是我们配置的事务管理器是DataSourceTransactionManager。

事实上，Spring还提供了另外一个分布式事务管理器JtaTransactionManager。这个是使用XA两阶段提交来保证事务的一致性。当然前提是，你的消息中间件是实现了JMS规范中事务消息相关API（回顾前面我们介绍JTA规范时，提到DB、MQ都只是资源管理器RM，对于事务管理器来说，二者是等价的）。

因此如果你满足了2个条件：1、使用JtaTransactionManager 2、DB、MQ分别实现了JDBC、JMS规范中规定的RM应该实现的两阶段提交的API，就可以保证消息发送的一致性。

DB作为RM，一般都是支持两阶段提交的。不过，一些MQ中间件并不支持，所以你要找到支持两阶段提交的MQ中间件。另外，JtaTransactionManager只是一个代理，你需要提供一个真实的事务管理器(TM)实现。如前面提到了atomikos公司，就有这样的产品。

但是笔者依然不建议，这样做。因为XA两阶段提交性能低，我们使用消息中间件就是为了异步解耦，这种情况，虽然保证了一致性，但是响应时间却大大增加，系统可用性降低。



#### 实现方案

以RocketMQ的事务消息为例，如下图所示，消息的可靠发送由发送端 Producer进行保证(消费端无需考虑)，可靠发送消息的步骤如下：

1. 发送一个事务消息，这个时候，RocketMQ将消息状态标记为Prepared，注意此时这条消息消费者是无法消费到的；
2. 执行业务代码逻辑，可能是一个本地数据库事务操作；
3. 确认发送消息，这个时候，RocketMQ将消息状态标记为可消费，这个时候消费者，才能真正的保证消费到这条数据。

如果确认消息发送失败了怎么办？RocketMQ会定期扫描消息集群中的事务消息，如果发现了Prepared消息，它会向消息发送端(生产者)确认。RocketMQ会根据发送端设置的策略来决定是回滚还是继续发送确认消息。这样就保证了消息发送与本地事务同时成功或同时失败。

如果消费失败怎么办？阿里提供给我们的解决方法是：人工解决。

![在这里插入图片描述](../../Image/2022/07/220731-2.png)

![RocketMQ](../../Image/2022/07/220720-45.png)



1. 事务发起方首先发送半消息到MQ；
2. MQ通知发送方消息发送成功；
3. 在发送半消息成功后执行本地事务；
4. 根据本地事务执行结果返回commit或者是rollback；
5. 如果消息是rollback, MQ将丢弃该消息不投递；如果是commit，MQ将会消息发送给消息订阅方；
6. 订阅方根据消息执行本地事务；
7. 订阅方执行本地事务成功后再从MQ中将该消息标记为已消费；
8. 如果执行本地事务过程中，执行端挂掉，或者超时，MQ服务器端将不停的询问producer来获取事务状态；
9. Consumer端的消费成功机制有MQ保证；



### 本地消息表

本地消息表的核心思想就是将分布式事务**拆分**成本地事务进行处理。



并不是所有的mq都支持事务消息。也就是消息一旦发送到消息队列中，消费者立马就可以消费到。此时可以使用独立消息服务、或者本地事务表。

![本地事务](../../Image/2022/07/220720-46.png)



可以看到，其实就是将消息先发送到一个我们自己编写的一个"独立消息服务"应用中，刚开始处于prepare状态，业务逻辑处理成功后，确认发送消息，这个时候"独立消息服务"才会真正的把消息发送给消息队列。消费者消费成功后，ack时，除了对消息队列进行ack(图中没有画出)，对于独立消息服务也要进行ack，"独立消息服务"一般是把这条消息删除。而定时扫描prepare状态的消息，向消息发送端(生产者)确认的工作也由独立消息服务来完成。

对于"本地事务表"，其实和"独立消息服务"的作用类似，只不过"独立消息服务"是需要独立部署的，而"本地事务表"是将"独立消息服务"的功能内嵌到应用中。



#### 实现方案

发送消息方：

- 需要有一个消息表，记录着消息状态相关信息。
- 业务数据和消息表在同一个数据库，要保证它俩在同一个本地事务。直接利用本地事务，将业务数据和事务消息直接写入数据库。
- 在本地事务中处理完业务数据和写消息表操作后，通过写消息到 MQ 消息队列。使用专门的投递工作线程进行事务消息投递到MQ，根据投递ACK去删除事务消息表记录
- 消息会发到消息消费方，如果发送失败，即进行重试。

消息消费方：

- 处理消息队列中的消息，完成自己的业务逻辑。
- 如果本地事务处理成功，则表明已经处理成功了。
- 如果本地事务处理失败，那么就会重试执行。
- 如果是业务层面的失败，给消息生产方发送一个业务补偿消息，通知进行回滚等操作。

生产方和消费方定时扫描本地消息表，把还没处理完成的消息或者失败的消息再发送一遍。如果有靠谱的自动对账补账逻辑，这种方案还是非常实用的。



#### 优缺点

优点：

- 本地消息表建设成本比较低，实现了可靠消息的传递确保了分布式事务的最终一致性。
- 无需提供回查方法，进一步减少的业务的侵入。
- 在某些场景下，还可以进一步利用注解等形式进行解耦，有可能实现无业务代码侵入式的实现。

缺点：

- 本地消息表与业务耦合在一起，难于做成通用性，不可独立伸缩。
- 本地消息表是基于数据库来做的，而数据库是要读写磁盘IO的，因此在高并发下是有性能瓶颈的



#### 扩展

##### 对比MQ事务消息

**二者的共性**：

1、 事务消息都依赖MQ进行事务通知，所以都是异步的。

2、 事务消息在投递方都是存在重复投递的可能，需要有配套的机制去降低重复投递率，实现更友好的消息投递去重。

3、 事务消息的消费方，因为投递重复的无法避免，因此需要进行消费去重设计或者服务幂等设计。



**二者的区别：**

MQ事务消息：

- 需要MQ支持半消息机制或者类似特性，在重复投递上具有比较好的去重处理；
- 具有比较大的业务侵入性，需要业务方进行改造，提供对应的本地操作成功的回查功能；

DB本地消息表：

- 使用了数据库来存储事务消息，降低了对MQ的要求，但是增加了存储成本；
- 事务消息使用了异步投递，增大了消息重复投递的可能性；



### 最大努力通知

最大努力通知方案的目标，就是发起通知方通过一定的机制，最大努力将业务处理结果通知到接收方。

最大努力通知型( Best-effort delivery)是最简单的一种柔性事务，适用于一些最终一致性时间敏感度低、业务链路较短，且被动方处理结果不影响主动方的处理结果。典型的使用场景：如银行通知、商户通知等。最大努力通知型的实现方案，一般符合以下特点：

1. 不可靠消息：业务活动主动方，在完成业务处理之后，向业务活动的被动方发送消息，直到通知N次后不再通知，允许消息丢失(不可靠消息)。
2. 定期校对：业务活动的被动方，根据定时策略，向业务活动主动方查询(主动方提供查询接口)，恢复丢失的业务消息。



**最大努力通知事务**主要用于**外部系统**，因为外部的网络环境更加复杂和不可信，所以只能尽最大努力去通知实现数据最终一致性，**比如充值平台与运营商、支付对接、商户通知等等跨平台、跨企业的系统间业务交互场景**；

而**异步确保型事务**主要适用于**内部系统**的数据最终一致性保障，因为内部相对比较可控，比如订单和购物车、收货与清算、支付与结算等等场景。



通知型事务的难度在于： **投递消息和参与者本地事务的一致性保障**。

**因为核心要点一致，都是为了保证消息的一致性投递**，所以，最大努力通知事务在投递流程上跟异步确保型是一样的，因此也有**两个分支**：

- **基于MQ自身的事务消息方案**
- **基于DB的本地事务消息表方案**



#### 实现方案

##### MQ事务消息方案

因为异步确保型在于内部的事务处理，所以MQ和系统是直连并且无需严格的权限、安全等方面的思路设计。最大努力通知事务在于第三方系统的对接，所以最大努力通知事务有几个特性：

- 业务主动方在完成业务处理后，向业务被动方(第三方系统)发送通知消息，允许存在消息丢失。
- 业务主动方提供递增多挡位时间间隔(5min、10min、30min、1h、24h)，用于失败重试调用业务被动方的接口；在通知N次之后就不再通知，报警+记日志+人工介入。
- 业务被动方提供幂等的服务接口，防止通知重复消费。
- 业务主动方需要有定期校验机制，对业务数据进行兜底；防止业务被动方无法履行责任时进行业务回滚，确保数据最终一致性。

![img](../../Image/2022/07/220731-3.png)



1. 业务活动的主动方，在完成业务处理之后，向业务活动的被动方发送消息，允许消息丢失。
2. 主动方可以设置时间阶梯型通知规则，在通知失败后按规则重复通知，直到通知N次后不再通知。
3. 主动方提供校对查询接口给被动方按需校对查询，用于恢复丢失的业务消息。
4. 业务活动的被动方如果正常接收了数据，就正常返回响应，并结束事务。
5. 如果被动方没有正常接收，根据定时策略，向业务活动主动方查询，恢复丢失的业务消息。

**特点**

1. 用到的服务模式：可查询操作、幂等操作；
2. 被动方的处理结果不影响主动方的处理结果；
3. 适用于对业务最终一致性的时间敏感度低的系统；
4. 适合跨企业的系统间的操作，或者企业内部比较独立的系统间的操作，比如银行通知、商户通知等；



##### 本地消息表方案

![在这里插入图片描述](../../Image/2022/07/220731-4.png)

发送消息方：

- 需要有一个消息表，记录着消息状态相关信息。
- 业务数据和消息表在同一个数据库，要保证它俩在同一个本地事务。直接利用本地事务，将业务数据和事务消息直接写入数据库。
- 在本地事务中处理完业务数据和写消息表操作后，通过写消息到 MQ 消息队列。使用专门的投递工作线程进行事务消息投递到MQ，根据投递ACK去删除事务消息表记录
- 消息会发到消息消费方，如果发送失败，即进行重试。
- 生产方定时扫描本地消息表，把还没处理完成的消息或者失败的消息再发送一遍。如果有靠谱的自动对账补账逻辑，这种方案还是非常实用的。

最大努力通知事务在于第三方系统的对接，所以最大努力通知事务有几个特性：

- 业务主动方在完成业务处理后，向业务被动方(第三方系统)发送通知消息，允许存在消息丢失。
- 业务主动方提供递增多挡位时间间隔(5min、10min、30min、1h、24h)，用于失败重试调用业务被动方的接口；在通知N次之后就不再通知，报警+记日志+人工介入。
- 业务被动方提供幂等的服务接口，防止通知重复消费。
- 业务主动方需要有定期校验机制，对业务数据进行兜底；防止业务被动方无法履行责任时进行业务回滚，确保数据最终一致性。



#### 扩展

##### 对比异步确保型事务

最大努力通知事务在我认知中，其实是基于异步确保型事务发展而来适用于外部对接的一种业务实现。他们主要有的是业务差别，如下：

• 从参与者来说：最大努力通知事务适用于跨平台、跨企业的系统间业务交互；异步确保型事务更适用于同网络体系的内部服务交付。

• 从消息层面说：最大努力通知事务需要主动推送并提供多档次时间的重试机制来保证数据的通知；而异步确保型事务只需要消息消费者主动去消费。

• 从数据层面说：最大努力通知事务还需额外的定期校验机制对数据进行兜底，保证数据的最终一致性；而异步确保型事务只需保证消息的可靠投递即可，自身无需对数据进行兜底处理。



### 最终一致性

#### 查询模式

服务操作都需要提供一个查询接口，用来向外部输出操作执行的状态。

服务操作的使用方可以通过查询接口，得知服务操作执行的状态，然后根据不同状态来做不同的处理操作。



#### 补偿模式

如果整个操作处于不正常的状态，我们需要修正操作中有问题的子操作，这可能需要重新执行未完成的子操作，后者取消已经完成的子操作，通过修复使整个分布式系统达到一致，为了让系统最终一致而做的努力都叫做补偿

1. 自动恢复：程序根据发生不一致的环境，通过继续未完成的操作，或者回滚已经完成的操作，自动来达到一致
2. 通知运营：如果程序无法自动恢复，并且设计时考虑到了不一致的场景，可以提供运营功能，通过运营手工进行补偿
3. 通知技术：如果很不巧，系统无法自动回复，又没有运营功能，那必须通过技术手段来解决，技术手段包括走数据库变更或者代码变更来解决，这是最糟的一种场景



#### 异步确保模式

异步确保模式是补偿模式的一个典型案例，经常应用到使用方对响应时间要求并不太高，我们通常把这类操作从主流程中摘除，通过异步的方式进行处理，处理后把结果通过通知系统通知给使用方，这个方案最大的好处能够对高并发流量进行削峰。

实践中，将要执行的异步操作封装后持久入库，然后通过定时捞取未完成的任务进行补偿操作来实现异步确保模式，只要定时系统足够健壮，任何一个任务最终会被成功执行。



#### 定期校对模式

在操作的主流程中的系统间执行校对操作，我们可以事后异步的批量校对操作的状态，如果发现不一致的操作，则进行补偿，补偿操作与补偿模式中的补偿操作是一致的

实现定期校对的一个关键就是分布式系统中需要有一个自始至终唯一的ID，常用的唯一id。



#### 可靠消息模式

对于异步的操作可以使用消息队列，通过消息队列将调用方和被调用方进行解耦，提高系统响应速度，同时能够达到消峰目的；

对于消息队列，我们需要建立特殊的设施保证可靠的消息发送以及处理机的幂等。



发送消息之前，把消息持久到数据库，状态标记为待发送，然后发送消息，如果发送成功，将消息改为发送成功。定时任务定时从数据库捞取一定时间内未发送的消息，将消息发送

使用第三方消息管理器，发送消息之前，先发送一个预消息给第三方消息管理器，消息管理器将其持久到数据库，并标记状态为待发送，发送成功后，标记消息为发送成功。定时任务定时从数据库捞取一定时间内未发送的消息，回查业务系统是否要继续发送，根据查询结果来确定消息的状态



保证消息一定要发送出去，那么就需要有重试机制，有了重试机制，消息一定会重复，那么我们需要对重复做处，常用几种方案

- 使用数据库表的唯一索引进行防重，拒绝重复的请求
- 使用分布式中间件Redis进行防重
- 使用状态机防重，单据相关的业务会涉及到状态机，状态在不同情况下会发生变更，如果状态机已经处于下一个状态，这时候来一个上一个状态的变更，理论上是不能够变更的，保证了有限状态机的幂等
- 使用乐观锁防重，数据更新带条件，这也是在系统设计的时候，合理的选择乐观锁，通过version或者其他条件来做乐观锁，这样保证更新及时在并发的情况下，也不会有太大的问题



## 总结

| 方案       | 类型           | 类型     |
| ---------- | -------------- | -------- |
| 2PC        | 两阶段型       | 刚性事务 |
| 3PC        | 两阶段型       | 刚性事务 |
| TCC        | 补偿型         | 柔性事务 |
| SAGA       | 补偿型         | 柔性事务 |
| MQ事务消息 | 异步确保型     | 柔性事务 |
| 本地消息表 | 异步确保型     | 柔性事务 |
| MQ事务消息 | 最大努力通知型 | 柔性事务 |
| 本地消息表 | 最大努力通知型 | 柔性事务 |



### 参考资料

[一致性协议 （史上最全）](https://www.cnblogs.com/crazymakercircle/p/14334422.html)



## Seata

Seata是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。Seata将为用户提供了AT、TCC、SAGA和XA事务模式，为用户打造一站式的分布式解决方案。

Seata的分布式事务解决方案是业务层面的解决方案，只依赖于单台数据库的事务能力。Seata框架中一个分布式事务包含3中角色：



### Seata组成

| 角色                         | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| Transaction Coordinator (TC) | 事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚。 |
| Transaction Manager (TM)     | 控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议。 |
| Resource Manager (RM)        | 控制分支事务，负责分支注册、状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚。 |

<img src="../../Image/2022/07/220720-48.png" alt="Seata框架" style="zoom: 80%;" />



### Seata事务流程

一个典型的seata分布式事务的流程如下：

1. TM向TC发出开启全局事务请求，TC生成全局事务的唯一标识XID，设此处的全局事务为T1；
2. 在全局事务T1的各个流程中，XID会作为事务的标识在微服务之间流转；
3. RM向TC注册本地事务，注册的本地事务的会作为全局事务T1的分支事务；
4. TM可以请求TC控制全局事务T1提交或全局事务T1回滚；
5. TC可以请求全局事务T1下的所有分支事务提交或回滚；



![Seata流程](../../Image/2022/07/220720-47.png)



### Seata使用示例

**maven依赖**

```xml
<seata.version>1.4.2</seata.version>

<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-all</artifactId>
    <version>${seata.version}</version>
</dependency>
```



引入Seata服务之后，只需要在分布式事务最外层的方法上添加分布式事务注解 `@GlobalTransactional`。添加注解之后，在执行业务逻辑之前，Seata会先生成全局事务ID，并且在调用其它服务时，会在请求中携带全局事务ID。如果其它微服务也添加了Seata依赖，这些微服务会获取全局事务ID，并且参与到全局事务中。



## 参考资料
- []()
- []()
- []()
- [TXC分布式事务简介](https://www.cnblogs.com/aspirant/p/12750589.html)
