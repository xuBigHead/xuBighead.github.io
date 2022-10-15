# 前言

![u=4174373953,1719844580&fm=15&gp=0](https://raw.githubusercontent.com/xuBigHead/pic/master/img/RocketMQ%E5%9B%BE%E6%A0%87.png)

[Rocket Github地址](https://github.com/apache/rocketmq/tree/master/docs/cn)



# 面试相关

RocketMQ是一个纯Java、分布式、队列模型的开源消息中间件，前身是MetaQ，是阿里参考Kafka特点研发的一个队列模型的消息中间件，后开源给apache基金会成为了apache的顶级开源项目，具有高性能、高可靠、高实时、分布式特点。



RocketMQ优点：

- 单机吞吐量：十万级
- 可用性：非常高，分布式架构
- 消息可靠性：经过参数优化配置，消息可以做到0丢失
- 功能支持：MQ功能较为完善，还是分布式的，扩展性好
- 支持10亿级别的消息堆积，不会因为堆积导致性能下降
- 源码是java，我们可以自己阅读源码，定制自己公司的MQ，可以掌控



RocketMQ缺点：

- 支持的客户端语言不多，目前是java及c++，其中c++不成熟
- 社区活跃度不是特别活跃那种
- 没有在 mq 核心中去实现**JMS**等接口，有些系统要迁移需要修改大量代码



**为什么选择RocketMQ？**

ActiveMQ较老；RabbitMQ是erlang开发的，不方便看源码和定制；Kafka更多用于大数据。

RocketMQ是Java开发的，经过了阿里的超高并发和高吞吐的考验。



**MQ设计思路**

可伸缩；数据持久化；可用性；数据不丢失。



## 消息队列

**优点**：**解耦**、**异步**、**削峰**。

**缺点**：**系统可用性降低**；**系统复杂度提高**；**一致性问题**。



## 组成角色

![图片](../../Image/2022/07/220725-1.png)

| 角色       | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| Nameserver | 无状态，动态列表；这也是和zookeeper的重要区别之一。zookeeper是有状态的。 |
| Producer   | 消息生产者，负责发消息到Broker。                             |
| Broker     | 就是MQ本身，负责收发消息、持久化消息等。                     |
| Consumer   | 消息消费者，负责从Broker上拉取消息进行消费，消费完进行ack。  |



### Nameserver

类似一个注册中心，底层由netty实现，提供了路由管理、服务注册、服务发现的功能，是一个无状态节点。

nameserver是服务发现者，集群中各个角色（producer、broker、consumer等）都需要定时向nameserver上报自己的状态，以便互相发现彼此，超时不上报的话，nameserver会把它从列表中剔除。

nameserver可以部署多个，当多个nameserver存在的时候，其他角色同时向他们上报信息，以保证高可用。NameServer集群间互不通信，没有主备的概念。nameserver内存式存储，nameserver中的broker、topic等信息默认不会持久化，所以他是无状态节点。



**NameServer**是一个功能齐全的服务器，其角色类似Dubbo中的Zookeeper，但NameServer与Zookeeper相比更轻量。主要是因为每个NameServer节点互相之间是独立的，没有任何信息交互。

**NameServer**压力不会太大，平时主要开销是在维持心跳和提供Topic-Broker的关系数据。

但有一点需要注意，Broker向NameServer发心跳时， 会带上当前自己所负责的所有**Topic**信息，如果**Topic**个数太多（万级别），会导致一次心跳中，就Topic的数据就几十M，网络情况差的话， 网络传输失败，心跳失败，导致NameServer误认为Broker心跳失败。

**NameServer** 被设计成几乎无状态的，可以横向扩展，节点之间相互之间无通信，通过部署多台机器来标记自己是一个伪集群。

每个 Broker 在启动的时候会到 NameServer 注册，Producer 在发送消息前会根据 Topic 到 **NameServer** 获取到 Broker 的路由信息，Consumer 也会定时获取 Topic 的路由信息。



### Producer

消息的生产者，Producer随机选择其中一个NameServer节点建立长连接，获得Topic路由信息（包括topic下的queue，这些queue分布在哪些broker上等等）。接下来向提供topic服务的master建立长连接（因为rocketmq只有master才能写消息），且定时向master发送心跳。

**Producer**由用户进行分布式部署，消息由**Producer**通过多种负载均衡模式发送到**Broker**集群，发送低延时，支持快速失败。



1、获得 Topic-Broker 的映射关系。

Producer 启动时，也需要指定 Namesrv 的地址，从 Namesrv 集群中选一台建立长连接。

生产者每 30 秒从 Namesrv 获取 Topic 跟 Broker 的映射关系，更新到本地内存中。然后再跟 Topic 涉及的所有 Broker 建立长连接，每隔 30 秒发一次心跳。

2、生产者端的负载均衡。

生产者发送时，会自动轮询当前所有可发送的broker，一条消息发送成功，下次换另外一个broker发送，以达到消息平均落到所有的broker上。



#### 消息发送方式

**RocketMQ** 提供了三种方式发送消息：同步、异步和单向。



##### 同步发送

同步发送指消息发送方发出数据后会在收到接收方发回响应之后才发下一个数据包。一般用于重要通知消息，例如重要通知邮件、营销短信。



##### 异步发送

异步发送指发送方发出数据后，不等接收方发回响应，接着发送下个数据包，一般用于可能链路耗时较长而对响应时间敏感的业务场景，例如用户视频上传后通知启动转码服务。



##### 单向发送

单向发送是指只负责发送消息而不等待服务器回应且没有回调函数触发，适用于某些耗时非常短但对可靠性要求并不高的场景，例如日志收集。



#### Queue选择算法

分为两种，一种是直接发消息，client内部有选择queue的算法，不允许外界改变。还有一种是可以自定义queue的选择算法，内置了三种算法，不喜欢的话可以自定义算法实现。

有时候我们不希望默认的queue选择算法，而是需要自定义，一般最常用的场景在顺序消息，顺序消息的发送一般都会指定某组特征的消息都发当同一个queue里，这样才能保证顺序，因为单queue是有序的。



内置了三种算法，三种算法都实现了一个共同的接口MessageQueueSelector：

- `SelectMessageQueueByRandom`
- `SelectMessageQueueByHash`
- `SelectMessageQueueByMachineRoom`
- 要想自定义逻辑的话，直接实现接口重写select方法即可。



###### SelectMessageQueueByRandom

> org.apache.rocketmq.client.producer.selector.SelectMessageQueueByRandom

```java
public class SelectMessageQueueByRandom implements MessageQueueSelector {
    private Random random = new Random(System.currentTimeMillis());

    @Override
    public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
        // mqs.size()：队列的个数。假设队列个数是4，那么这个value就是0-3之间随机。
        int value = random.nextInt(mqs.size());
        return mqs.get(value);
    }
}
```



###### SelectMessageQueueByHash

> org.apache.rocketmq.client.producer.selector.SelectMessageQueueByHash

```java
public class SelectMessageQueueByHash implements MessageQueueSelector {

    @Override
    public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
        int value = arg.hashCode();
        if (value < 0) {
            // 防止出现负数，取个绝对值，这也是我们平时开发中需要注意到的点
            value = Math.abs(value);
        }
        // 直接取余队列个数。
        value = value % mqs.size();
        return mqs.get(value);
    }
}
```



###### SelectMessageQueueByMachineRoom

> org.apache.rocketmq.client.producer.selector.SelectMessageQueueByMachineRoom

```java
public class SelectMessageQueueByMachineRoom implements MessageQueueSelector {
    private Set<String> consumeridcs;

    @Override
    public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
        return null;
    }

    public Set<String> getConsumeridcs() {
        return consumeridcs;
    }

    public void setConsumeridcs(Set<String> consumeridcs) {
        this.consumeridcs = consumeridcs;
    }
}
```



#### 扩展

##### Topic和Queue的区别

queue就是来源于数据结构的FIFO队列。而Topic是个抽象的概念，每个Topic底层对应N个queue，而数据也真实存在queue上的。



### Broker

broker主要用于producer和consumer接收和发送消息，是消息中间件的消息存储、转发服务器。每个Broker节点，在启动时，都会遍历NameServer列表，与每个NameServer建立长连接，注册自己的信息，之后定时上报提交自己的信息。

- **Broker**是具体提供业务的服务器，单个Broker节点与所有的NameServer节点保持长连接及心跳，并会定时将**Topic**信息注册到NameServer，顺带一提底层的通信和连接都是**基于Netty实现**的。
- **Broker**负责消息存储，以Topic为纬度支持轻量级的队列，单机可以支撑上万队列规模，支持消息推拉模型。
- 官网上有数据显示：具有**上亿级消息堆积能力**，同时可**严格保证消息的有序性**。



#### 接收消息

##### Queue的分布

一个topic的queue可以分布到多个Broker上。比如一个topic有4个queue，他可能分配到broker-a上三个queue，broker-b上1个queue，这个queue的分配是由broker端决定的。

**每个queue的消息都是不一样的，也就是比如你发N条消息，他可能一部分在broker-a上一部分在broker-b上，不管他在哪，消息都是不一样的，不要理解成M-S那种复制。他只是负载均衡将queue分配到了不同的broker上。**



#### 持久化消息

消息持久化的地方其实是磁盘上，在如下目录里的commitlog文件夹里。文件大小是1.0G，超过1.0G再写入消息的话会自动创建新的commitlog文件。

```java
// 数据存储根目录
private String storePathRootDir = System.getProperty("user.home") + File.separator + "store";
// commitlog目录
private String storePathCommitLog = System.getProperty("user.home") + File.separator + "store" + File.separator + "commitlog";
// 每个commitlog文件大小为1GB，超过1GB则创建新的commitlog文件
private int mappedFileSizeCommitLog = 1024 * 1024 * 1024;
```



先将消息写入到ByteBuffer缓冲区，然后默认是每隔500毫秒刷一次盘。有两种刷盘方式，同步和异步，一般选择异步，同步效率低，但是更可靠。



|         类名          |                 描述                 | 刷盘性能 |
| :-------------------: | :----------------------------------: | :------: |
| CommitRealTimeService |      异步刷盘 &&开启字节缓冲区       |   最高   |
| FlushRealTimeService  |     异步刷盘&&关闭内存字节缓冲区     |   较高   |
|  GroupCommitService   | 同步刷盘，刷完盘才会返回消息写入成功 |   最低   |



Broker的**Buffer**通常指的是Broker中一个队列的内存Buffer大小，这类**Buffer**通常大小有限。

另外，RocketMQ没有内存**Buffer**概念，RocketMQ的队列都是持久化磁盘，数据定期清除。

RocketMQ同其他MQ有非常显著的区别，RocketMQ的内存**Buffer**抽象成一个无限长度的队列，不管有多少数据进来都能装得下，这个无限是有前提的，Broker会定期删除过期的数据。

例如Broker只保存3天的消息，那么这个**Buffer**虽然长度无限，但是3天前的数据会被从队尾删除。



##### 同步刷盘



##### 异步刷盘



#### 扩展

##### 消息被消费后会立即删除吗？

不会，每条消息都会持久化到CommitLog中，每个Consumer连接到Broker后会维持消费进度信息，当有消息消费后只是当前Consumer的消费进度（CommitLog的offset）更新了。



##### 那么消息会堆积吗？什么时候清理过期消息？

4.6版本默认48小时后会删除不再使用的CommitLog文件，避免消息堆积。

- 检查这个文件最后访问时间
- 判断是否大于过期时间
- 指定时间删除，默认凌晨4点



##### 任何一台Broker突然宕机了怎么办？

Broker主从架构以及多副本策略。Master收到消息后会同步给Slave，这样一条消息就不止一份了，Master宕机了还有slave中的消息可用，保证了MQ的可靠性和高可用性。

而且Rocket MQ4.5.0开始就支持了Dlegder模式，基于raft的，做到了真正意义的HA。



##### RocketMQ 是如何保证数据的高容错性的?

- 在不开启容错的情况下，轮询队列进行发送，如果失败了，重试的时候过滤失败的Broker
- 如果开启了容错策略，会通过RocketMQ的预测机制来预测一个Broker是否可用
- 如果上次失败的Broker可用那么还是会选择该Broker的队列
- 如果上述情况失败，则随机选择一个进行发送
- 在发送消息的时候会记录一下调用的时间与是否报错，根据该时间去预测broker的可用时间



### Consumer

消息的消费者，通过NameServer集群获得Topic的路由信息，连接到对应的Broker上消费消息。由于Master和Slave都可以读取消息，因此Consumer会与Master和Slave都建立连接进行消费消息。

- **Consumer**也由用户部署，支持PUSH和PULL两种消费模式，支持**集群消费**和**广播消息**，提供**实时的消息订阅机制**。
- **Pull**：拉取型消费者（Pull Consumer）主动从消息服务器拉取信息，只要批量拉取到消息，用户应用就会启动消费过程，所以 Pull 称为主动消费型。
- **Push**：推送型消费者（Push Consumer）封装了消息的拉取、消费进度和其他的内部维护工作，将消息到达时执行的回调接口留给用户应用程序来实现。所以 Push 称为被动消费类型，但从实现上看还是从消息服务器中拉取消息，不同于 Pull 的是 Push 首先要注册消费监听器，当监听器处触发后才开始消费消息。



1、获得 Topic-Broker 的映射关系。

Consumer 启动时需要指定 Namesrv 地址，与其中一个 Namesrv 建立长连接。消费者每隔 30 秒从 Namesrv 获取所有Topic 的最新队列情况，

Consumer 跟 Broker 是长连接，会每隔 30 秒发心跳信息到Broker .

2、消费者端的负载均衡。根据消费者的消费模式不同，负载均衡方式也不同。



#### 消费原理

- Consumer端发心跳给Broker，Broker收到后存到consumerTable里（就是个Map），key是GroupName，value是ConsumerGroupInfo。
- ConsumerGroupInfo里面是包含topic等信息的，但是问题就出在上一步骤，key是groupName，你同GroupName的话Broker心跳最后收到的Consumer会覆盖前者的。



这样同key，肯定产生了覆盖。所以Consumer1不会收到任何消息，但是Consumer2为什么只收到了一半（不固定）消息呢？

那是因为：你是集群模式消费，它会负载均衡分配到各个节点去消费，所以一半消息（不固定个数）跑到了Consumer1上，结果Consumer1订阅的是tag1，所以不会任何输出。

**如果换成BROADCASTING，那绝逼后者会收到全部消息，而不是一半，因为广播是广播全部Consumer。**



#### 消费模式

##### 集群消费

1.一条消息只会被同Group中的一个Consumer消费一次

2.多个Group同时消费一个Topic时，每个Group都会有一个Consumer消费到数据

3.在消息重投时，不能保证路由到同一台机器上

4.消费状态由broker维护



##### 广播消费

消息将对一个Consumer Group 下的各个 Consumer 实例都消费一遍。即使这些 Consumer 属于同一个Consumer Group ，消息也会被 Consumer Group 中的每个 Consumer 都消费一次。



- 消费进度由consumer维护

- 保证每个消费者都消费一次消息

- 消费失败的消息不会重投

	

#### 消费方式

RocketMQ没有真正意义的push，都是pull，虽然有push类，但实际底层实现采用的是**长轮询机制**，即拉取方式。



> broker端属性 longPollingEnable 标记是否开启长轮询，默认开启。



##### 为什么要主动拉取消息而不使用事件监听方式？

事件驱动方式是建立好长连接，由事件（发送数据）的方式来实时推送。

如果broker主动推送消息的话有可能push速度快，消费速度慢的情况，那么就会造成消息在consumer端堆积过多，同时又不能被其他consumer消费的情况。而pull的方式可以根据当前自身情况来pull，不会造成过多的压力而造成瓶颈。所以采取了pull的方式。



##### 实现原理

Consumer首次请求Broker

- Broker中是否有符合条件的消息
- 有 ->
- - 响应Consumer
	- 等待下次Consumer的请求
- 没有
- - DefaultMessageStore#ReputMessageService#run方法
	- PullRequestHoldService 来Hold连接，每个5s执行一次检查pullRequestTable有没有消息，有的话立即推送
	- 每隔1ms检查commitLog中是否有新消息，有的话写入到pullRequestTable
	- 当有新消息的时候返回请求
	- 挂起consumer的请求，即不断开连接，也不返回数据
	- 使用consumer的offset
	



### 总结

核心流程如下：

- Broker都注册到Nameserver上
- Producer发消息的时候会从Nameserver上获取发消息的topic信息
- Producer向提供服务的所有master建立长连接，且定时向master发送心跳
- Consumer通过NameServer集群获得Topic的路由信息
- Consumer会与所有的Master和所有的Slave都建立连接进行监听新消息



## 工作流程

Producer 与 NameServer集群中的其中一个节点（随机选择）建立长连接，定期从 NameServer 获取 **Topic** 路由信息，并向提供 Topic 服务的 **Broker Master** 建立长连接，且定时向 **Broker** 发送心跳。

**Producer** 只能将消息发送到 Broker master，但是 **Consumer** 则不一样，它同时和提供 Topic 服务的 Master 和 Slave建立长连接，既可以从 Broker Master 订阅消息，也可以从 Broker Slave 订阅消息。



**执行流程**：

1、启动 Namesrv，Namesrv起 来后监听端口，等待 Broker、Producer、Consumer 连上来，相当于一个路由控制中心。

2、Broker 启动，跟所有的 Namesrv 保持长连接，定时发送心跳包。

3、收发消息前，先创建 Topic 。创建 Topic 时，需要指定该 Topic 要存储在 哪些 Broker上。也可以在发送消息时自动创建Topic。

4、Producer 发送消息。

5、Consumer 消费消息。



### NameServer

NameServer启动流程如下：

- 第一步是初始化配置
- 创建**NamesrvController**实例，并开启两个定时任务：每隔10s扫描一次**Broker**，移除处于不激活的**Broker**；每隔10s打印一次KV配置。
- 第三步注册钩子函数，启动服务器并监听Broker。



NameServer 会有每 10s 一次的定时任务检查 Broker 是否下线了，如果 120s 内有没有收到 Broker 心跳，则关闭 channel，把 Broker 信息从本地缓存移除。消费者则默认每隔 30s 向 NameServer 拉取路由信息来刷新本地缓存的 Broker 列表。也就是说可能会有最多 150s 的时间消费者拉取消息失败。



### Producer

![img](../../Image/2022/07/220725-5.png)



### Broker

**Broker**在RocketMQ中是进行处理Producer发送消息请求，Consumer消费消息的请求，并且进行消息的持久化，以及HA策略和服务端过滤，就是集群中很重的工作都是交给了**Broker**进行处理。

**Broker**模块是通过BrokerStartup进行启动的，会实例化BrokerController，并且调用其初始化方法。

Broker初始化，会根据配置创建很多线程，主要用来**发送消息**、**拉取消息**、**查询消息**、**客户端管理**和**消费者管理**，也有很多**定时任务**，同时也注册了很多**请求处理器**，用来发送拉取消息查询消息的。



### Consumer

![img](../../Image/2022/07/220725-6.png)



消费端会通过**RebalanceService**线程，10秒钟做一次基于**Topic**下的所有队列负载。



### 扩展

##### ACK

**ACK机制是发生在Consumer端的，不是在Producer端的**。也就是说Consumer消费完消息后要进行ACK确认，如果未确认则代表是消费失败，这时候Broker会进行重试策略（仅集群模式会重试）。ACK的意思就是：Consumer说：ok，我消费成功了。这条消息给我标记成已消费吧。



## 负载均衡

RocketMQ通过Topic在多Broker中分布式存储实现负载均衡。



### producer端

**Producer**通过轮训某个**Topic**下面的所有队列实现发送方的负载均衡。

发送端指定message queue发送消息到相应的broker，来达到写入时的负载均衡。

- 提升写入吞吐量，当多个producer同时向一个broker写入数据的时候，性能会下降
- 消息分布在多broker中，为负载消费做准备



**默认策略是随机选择：**

- producer维护一个index
- 每次取节点会自增
- index向所有broker个数取余
- 自带容错策略



**其他实现：**

- SelectMessageQueueByHash
- SelectMessageQueueByRandom
- SelectMessageQueueByMachineRoom 没有实现

也可以自定义实现**MessageQueueSelector**接口中的select方法



### consumer端

采用的是平均分配算法来进行负载均衡。



**其他负载均衡算法**

- 平均分配策略(默认)(AllocateMessageQueueAveragely) 

- 环形分配策略(AllocateMessageQueueAveragelyByCircle)
-  手动配置分配策略(AllocateMessageQueueByConfig) 
- 机房分配策略(AllocateMessageQueueByMachineRoom) 
- 一致性哈希分配策略(AllocateMessageQueueConsistentHash) 
- 靠近机房策略(AllocateMachineRoomNearby)



#### 扩展

##### 当消费负载均衡consumer和queue不对等的时候会发生什么？

- queue个数大于Consumer个数，且queue个数能整除Consumer个数的话， 那么Consumer会平均分配queue。
- queue个数大于Consumer个数，且queue个数不能整除Consumer个数的话， 那么会有一个Consumer多消费1个queue，其余Consumer平均分配。
- queue个数小于Consumer个数，那么会有Consumer闲置，就是浪费掉了，其余Consumer平均分配到queue上。



##### Consumer掉线或上线

当一个consumer出现宕机后，默认最多20s，其它机器将重新消费已宕机的机器消费的queue，同样当有新的Consumer连接上后，20s内也会完成rebalance使得新的Consumer有机会消费queue里的msg。

等等，好像有问题：新上线一个Consumer要等20s才能负载均衡？这不是搞笑呢吗？肯定有猫腻。

确实，新启动Consumer的话会立即唤醒沉睡的线程， 让他立马进行this.mqClientFactory.doRebalance()。



## 主从和集群

### 基础概念

**为什么要集群**

- 单点存在单点故障问题
- 集群可以分担压力，提高QPS
- 主从可以保证消息可靠性，比如只有M没S。M磁盘坏了，那未被消费的消息都丢了。而S可以作为备份。



### 集群模式

#### 单M模式

- 只有一个Master节点，所以单点故障是致命缺点。
- 优点：配置简单，方便部署。
- 缺点：单点故障，一旦Broker重启或者直接宕机了，那会导致整个服务不可用。



#### 多M模式

- 一个集群无Slave节点，全是Master节点。
- 优点：配置相对不复杂，单个M宕机或者重启对业务系统无感知，照常提供服务。只是这个broker上如果有消息未被消费的话可能无法继续消费，但是消息不会丢失，持久化到磁盘的。异步刷盘的话会存在少量丢失。
- 缺点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息实时性会受到受到影响。



#### 多M多S模式

一个集群既有Master节点又有Slave节点。



##### 异步复制

- 每个 Master 配置一个 Slave，有多对Master-Slave， HA，采用异步/同步复制方式，主备有短暂消息延迟，毫秒级。
- 优点：即使磁盘损坏，消息丢失的非常少，且消息实时性不会受影响，因为Master 宕机后，消费者仍然可以从 Slave消费，此过程对应用透明。不需要人工干预。性能同多 Master 模式几乎一样。
- 缺点：Master 宕机，磁盘损坏情况，会丢失少量消息。



##### 同步双写

- 每个 Master 配置一个 Slave，有多对Master-Slave， HA采用同步双写方式，主备都写成功，向应用返回成功。
- 优点：数据与服务都无单点， Master宕机情况下，消息无延迟，服务可用性与数据可用性都非常高。
- 缺点：性能比异步复制模式略低，大约低 10%左右，发送单个消息的 RT会略高。目前主宕机后，备机不能自动切换为主机，后续会支持自动切换功能。
- 要想真正意义的保证消息不丢失，这个同步双写是必须的 。



## 实际场景

### 重复消费

影响消息正常发送和消费的**重要原因是网络的不确定性。**



**引起重复消费的原因**

- ACK

正常情况下在consumer真正消费完消息后应该发送ack，通知broker该消息已正常消费，从queue中剔除

当ack因为网络原因无法发送到broker，broker会认为词条消息没有被消费，此后会开启消息重投机制把消息再次投递到consumer。



- 消费模式

在CLUSTERING模式下，消息在broker中会保证相同group的consumer消费一次，但是针对不同group的consumer会推送多次。



**解决方案**

- 数据库表

处理消息前，使用消息主键在表中带有约束的字段中insert

- Map

单机时可以使用map *ConcurrentHashMap* -> putIfAbsent  guava cache

- Redis

分布式锁搞起来。



### 消息去重

使用业务端逻辑保持幂等性，保证每条消息都有唯一编号。



### 顺序消费

首先多个queue只能保证单个queue里的顺序，queue是典型的FIFO，天然顺序。多个queue同时消费是无法绝对保证消息的有序性的。所以总结如下：

同一topic，同一个QUEUE，发消息的时候一个线程去发送消息，消费的时候 一个线程去消费一个queue里的消息。

但问题是1个topic有N个queue，作者这么设计的好处也很明显，天然支持集群和负载均衡的特性，将海量数据均匀分配到各个queue上，你发了10条消息到同一个topic上，这10条消息会自动分散在topic下的所有queue中，所以消费的时候不一定是先消费哪个queue，后消费哪个queue，这就导致了无序消费。

一个Producer发送了m1、m2、m3、m4四条消息到topic上，topic有四个队列，由于自带的负载均衡策略，四个队列上分别存储了一条消息。queue1上存储的m1，queue2上存储的m2，queue3上存储的m3，queue4上存储的m4，Consumer消费的时候是多线程消费，所以他无法保证先消费哪个队列或者哪个消息，比如发送的时候顺序是m1，m2，m3，m4，但是消费的时候由于Consumer内部是多线程消费的，所以可能先消费了queue4队列上的m4，然后才是m1，这就导致了无序。



#### 解决方案

##### 消息发到同一个queue

Rocket MQ提供了MessageQueueSelector接口，可以重写里面的接口，实现自己的算法，举个最简单的例子：判断`i % 2 == 0`，那就都放到queue1里，否则放到queue2里。



##### 单线程消费

比如你新需求：把未支付的订单都放到queue1里，已支付的订单都放到queue2里，支付异常的订单都放到queue3里，然后你消费的时候要保证每个queue是有序的，不能消费queue1一条直接跑到queue2去了，要逐个queue去消费。

这时候思路是发消息的时候利用自定义参数arg，消息体里肯定包含支付状态，判断是未支付的则选择queue1，以此类推。这样就保证了每个queue里只包含同等状态的消息。那么消费者目前是多线程消费的，肯定乱序。三个queue随机消费。解决方案更简单，直接将**消费端的线程数改为1个**，这样队列是FIFO，他就逐个消费了。

这种方式的缺点在于并行度就会成为消息系统的瓶颈。



### 消息丢失

![图片](../../Image/2022/07/220725-3.png)



消息流程分为如下三大部分，每一部分都有可能会丢失数据。

- 生产阶段：Producer通过网络将消息发送给Broker，这个发送可能会发生丢失，比如网络延迟不可达等。
- 存储阶段：Broker肯定是先把消息放到内存的，然后根据刷盘策略持久化到硬盘中，刚收到Producer的消息，再内存中了，但是异常宕机了，导致消息丢失。
- 消费阶段：消费失败了其实也是消息丢失的一种变体吧。



#### Producer

##### 同步发送消息

采取send()同步发消息，发送结果是同步感知的。

> org.apache.rocketmq.client.producer.DefaultMQProducer

```java
// 同步发送
public SendResult send(Message msg) throws MQClientException, RemotingException,      MQBrokerException, InterruptedException {}

// 异步发送，sendCallback作为回调
public void send(Message msg,SendCallback sendCallback) throws MQClientException, RemotingException, InterruptedException {}

// 单向发送，不关心发送结果，最不靠谱
public void sendOneway(Message msg) throws MQClientException, RemotingException, InterruptedException {}
```



`send` 方法是一个同步操作，只要这个方法不抛出任何异常，就代表消息已经**发送成功**。

消息发送成功仅代表消息已经到了 Broker 端，Broker 在不同配置下，可能会返回不同响应状态:

判断返回状态是否是 `SendStatus.SEND_OK`。若是其他状态，就需要考虑补偿重试。



###### 消息发送状态

- `SendStatus.SEND_OK`

消息发送成功。要注意的是消息发送成功也不意味着它是可靠的。要确保不会丢失任何消息，还应启用同步Master服务器或同步刷盘，即SYNC_MASTER或SYNC_FLUSH。



- `SendStatus.FLUSH_DISK_TIMEOUT`

消息发送成功但是服务器刷盘超时。此时消息已经进入服务器队列（内存），只有服务器宕机，消息才会丢失。消息存储配置参数中可以设置刷盘方式和同步刷盘时间长度，如果Broker服务器设置了刷盘方式为同步刷盘，即FlushDiskType=SYNC_FLUSH（默认为异步刷盘方式），当Broker服务器未在同步刷盘时间内（默认为5s）完成刷盘，则将返回该状态——刷盘超时。



- `SendStatus.FLUSH_SLAVE_TIMEOUT`

消息发送成功，但是服务器同步到Slave时超时。此时消息已经进入服务器队列，只有服务器宕机，消息才会丢失。如果Broker服务器的角色是同步Master，即SYNC_MASTER（默认是异步Master即ASYNC_MASTER），并且从Broker服务器未在同步刷盘时间（默认为5秒）内完成与主服务器的同步，则将返回该状态——数据同步到Slave服务器超时。



- `SendStatus.SLAVE_NOT_AVAILABLE`

消息发送成功，但是此时Slave不可用。如果Broker服务器的角色是同步Master，即SYNC_MASTER（默认是异步Master服务器即ASYNC_MASTER），但没有配置slave Broker服务器，则将返回该状态——无Slave服务器可用。



##### 自动重试机制

发送失败后可以重试，设置重试次数，默认3次。可以根据api进行更改，比如改为10次。

```java
producer.setRetryTimesWhenSendFailed(10);
```



> org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl

```java
// 自动重试次数，this.defaultMQProducer.getRetryTimesWhenSendFailed()默认为2，如果是同步发送，默认重试3次，否则重试1次
int timesTotal = communicationMode == CommunicationMode.SYNC ? 1 + this.defaultMQProducer.getRetryTimesWhenSendFailed() : 1;
int times = 0;
for (; times < timesTotal; times++) {
      // 选择发送的消息queue
    MessageQueue mqSelected = this.selectOneMessageQueue(topicPublishInfo, lastBrokerName);
    if (mqSelected != null) {
        try {
            // 真正的发送逻辑，sendKernelImpl。
            sendResult = this.sendKernelImpl(msg, mq, communicationMode, sendCallback, topicPublishInfo, timeout - costTime);
            switch (communicationMode) {
                case ASYNC:
                    return null;
                case ONEWAY:
                    return null;
                case SYNC:
                    // 如果发送失败了，则continue，意味着还会再次进入for，继续重试发送
                    if (sendResult.getSendStatus() != SendStatus.SEND_OK) {
                        if (this.defaultMQProducer.isRetryAnotherBrokerWhenNotStoreOK()) {
                            continue;
                        }
                    }
                    // 发送成功的话，将发送结果返回给调用者
                    return sendResult;
                default:
                    break;
            }
        } catch (RemotingException e) {
            continue;
        } catch (...) {
            continue;
        }
    }
}
```



重试流程如下：

- 重试次数同步是1 + `this.defaultMQProducer.getRetryTimesWhenSendFailed()`，其他方式默认1次。
- this.defaultMQProducer.getRetryTimesWhenSendFailed()默认是2，我们可以手动设置`producer.setRetryTimesWhenSendFailed(10);`
- 调用sendKernelImpl真正的去发送消息
- 如果是sync同步发送，且发送失败了，则continue，意味着还会再次进入for，继续重试发送
- 发送成功的话，将发送结果返回给调用者
- 如果发送异常进入catch了，则continue继续下次重试。



##### 多个Master节点

集群部署，比如发送失败了的原因可能是当前Broker宕机了，重试的时候会发送到其他Broker上。

假设Broker宕机了，但是生产环境一般都是多M多S的，所以还会有其他master节点继续提供服务，这也不会影响到我们发送消息，我们消息依然可达。因为比如恰巧发送到broker的时候，broker宕机了，producer收到broker的响应发送失败了，这时候producer会自动重试，这时候宕机的broker就被踢下线了， 所以producer会换一台broker发送消息。



##### 总结

失败会自动重试，即使重试N次也不行后，那客户端也会知道消息没成功，这也可以自己补偿等，不会盲目影响到主业务逻辑。再比如即使Broker挂了，那还有其他Broker再提供服务了，高可用，不影响。

总结为几个字就是：**同步发送+自动重试机制+多个Master节点。**



#### Broker

Broker是先把消息放到内存的，然后根据刷盘策略持久化到硬盘中，刚收到Producer的消息，再内存中了，但是异常宕机了，导致消息丢失。

默认情况下，消息只要到了 Broker 端，将会优先保存到内存中，然后立刻返回确认响应给生产者。随后 Broker 定期批量的将一组消息从内存异步刷入磁盘。

这种方式减少 I/O 次数，可以取得更好的性能，但是如果发生机器掉电，异常宕机等情况，消息还未及时刷入磁盘，就会出现丢失消息的情况。

若想保证 Broker 端不丢消息，保证消息的可靠性，我们需要将消息保存机制修改为同步刷盘方式，即消息**存储磁盘成功**，才会返回响应。



##### 同步刷盘

MQ持久化消息分为两种：同步刷盘和异步刷盘。默认情况是异步刷盘，Broker收到消息后会先存到cache里然后立马通知Producer说消息我收到且存储成功了，你可以继续你的业务逻辑了，然后Broker起个线程异步的去持久化到磁盘中，但是Broker还没持久化到磁盘就宕机的话，消息就丢失了。同步刷盘的话是收到消息存到cache后并不会通知Producer说消息已经ok了，而是会等到持久化到磁盘中后才会通知Producer说消息完事了。这也保障了消息不会丢失，但是性能不如异步高。看业务场景取舍。

修改刷盘策略为同步刷盘。默认情况下是异步刷盘的。

```bash
# 默认情况为 ASYNC_FLUSH，修改为同步刷盘：SYNC_FLUSH，实际场景看业务，同步刷盘效率肯定不如异步刷盘高。
flushDiskType = SYNC_FLUSH
```

若 Broker 未在同步刷盘时间内（**默认为 5s**）完成刷盘，将会返回 `SendStatus.FLUSH_DISK_TIMEOUT` 状态给生产者。



对应的Java配置类如下：

```java
package org.apache.rocketmq.store.config;

public enum FlushDiskType {
    // 同步刷盘
    SYNC_FLUSH,
    // 异步刷盘（默认）
    ASYNC_FLUSH
}
```



异步刷盘默认10s执行一次，源码如下：

```java
/*
 * {@link org.apache.rocketmq.store.CommitLog#run()}
 */

while (!this.isStopped()) {
    try {
        // 等待10s
        this.waitForRunning(10);
        // 刷盘
        this.doCommit();
    } catch (Exception e) {
        CommitLog.log.warn(this.getServiceName() + " service has exception. ", e);
    }
}
```



##### 集群部署

集群部署，主从模式，高可用。

即使Broker设置了同步刷盘策略，但是Broker刷完盘后磁盘坏了，这会导致盘上的消息全TM丢了。但是如果即使是1主1从了，但是Master刷完盘后还没来得及同步给Slave就磁盘坏了，不也是GG吗？没错！

所以我们还可以配置不仅是等Master刷完盘就通知Producer，而是等Master和Slave都刷完盘后才去通知Producer说消息ok了。

为了保证可用性，Broker 通常采用一主（**master**）多从（**slave**）部署方式。为了保证消息不丢失，消息还需要复制到 slave 节点。

默认方式下，消息写入 **master** 成功，就可以返回确认响应给生产者，接着消息将会异步复制到 **slave** 节点。

```bash
# 默认为 ASYNC_MASTER
brokerRole=SYNC_MASTER
```

此时若 master 突然**宕机且不可恢复**，那么还未复制到 **slave** 的消息将会丢失。

为了进一步提高消息的可靠性，我们可以采用同步的复制方式，**master** 节点将会同步等待 **slave** 节点复制完成，才会返回确认响应。

如果 **slave** 节点未在指定时间内同步返回响应，生产者将会收到 `SendStatus.FLUSH_SLAVE_TIMEOUT` 返回状态。



##### 总结

若想很严格的保证Broker存储消息阶段消息不丢失，则需要如下配置，但是性能肯定远差于默认配置。

```bash
# master 节点配置
flushDiskType = SYNC_FLUSH
brokerRole=SYNC_MASTER

# slave 节点配置
brokerRole=slave
flushDiskType = SYNC_FLUSH
```



上面这个配置含义是：

Producer发消息到Broker后，Broker的Master节点先持久化到磁盘中，然后同步数据给Slave节点，Slave节点同步完且落盘完成后才会返回给Producer说消息ok了。



#### Consumer

消费失败了其实也是消息丢失的一种变体。

消费者从 broker 拉取消息，然后执行相应的业务逻辑。一旦执行成功，将会返回 `ConsumeConcurrentlyStatus.CONSUME_SUCCESS` 状态给 Broker。

如果 Broker 未收到消费确认响应或收到其他状态，消费者下次还会再次拉取到该条消息，进行重试。这样的方式有效避免了消费者消费过程发生异常，或者消息在网络传输中丢失的情况。



##### ACK

消费者会先把消息拉取到本地，然后进行业务逻辑，业务逻辑完成后手动进行ack确认，这时候才会真正的代表消费完成。而不是说pull到本地后消息就算消费完了。

```java
 consumer.registerMessageListener(new MessageListenerConcurrently() {
     @Override
     public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
         for (MessageExt msg : msgs) {
             String str = new String(msg.getBody());
             System.out.println(str);
         }
         // ack，只有等上面一系列逻辑都处理完后，到这步CONSUME_SUCCESS才会通知broker说消息消费完成，
         // 如果上面发生异常没有走到这步ack，则消息还是未消费状态。
         // 而不是像比如redis的blpop，弹出一个数据后数据就从redis里消失了，
         // 并没有等我们业务逻辑执行完才弹出。
         return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
     }
 });
```



以上消费消息过程的，我们需要**注意返回消息状态**。只有当业务逻辑真正执行成功，我们才能返回 `ConsumeConcurrentlyStatus.CONSUME_SUCCESS`。否则我们需要返回 `ConsumeConcurrentlyStatus.RECONSUME_LATER`，稍后再重试。



##### 消息重试

消息消费失败自动重试。如果消费消息失败了，没有进行ack确认，则会自动重试，重试策略和次数（默认15次）如下配置。

```java
/**
 * Broker可以配置的所有选项
 */
public class org.apache.rocketmq.store.config.MessageStoreConfig {
    private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";
}
```



### 消息堆积

首先要找到是什么原因导致的消息堆积，是Producer太多了，Consumer太少了导致的还是说其他情况，总之先定位问题。

然后看下消息消费速度是否正常，正常的话，可以通过上线更多consumer临时解决消息堆积问题



1、如果可以添加消费者解决，就添加消费者的数据量
		2、如果出现了queue，但是消费者多的情况。可以使用准备一个临时的topic，同时创建一些queue，在临时创建一个消费者来把这些消息转移到topic中，让消费者消费。



#### 如果Consumer和Queue不对等，上线了多台也在短时间内无法消费完堆积的消息怎么办？

- 准备一个临时的topic
- queue的数量是堆积的几倍
- queue分布到多Broker中
- 上线一台Consumer做消息的搬运工，把原来Topic中的消息挪到新的Topic里，不做业务逻辑处理，只是挪过去
- 上线N台Consumer同时消费临时Topic中的数据
- 改bug
- 恢复原来的Consumer，继续消费之前的Topic



#### 堆积时间过长消息超时了？

RocketMQ中的消息只会在commitLog被删除的时候才会消失，不会超时。也就是说未被消费的消息不会存在超时删除这情况。



#### 堆积的消息会不会进死信队列？

不会，消息在消费失败后会进入重试队列（%RETRY%+ConsumerGroup），重试16次才会进入死信队（%DLQ%+ConsumerGroup）。



```java
public class MessageStoreConfig {
    // 每隔如下时间会进行重试，到最后一次时间重试失败的话就进入死信队列了，1s和5s被跳过。
 private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";
}
```



### 事务消息

RocketMQ 4.3+提供分布事务功能，通过 RocketMQ 事务消息能达到分布式事务的最终一致。



#### 实现原理

![img](../../Image/2022/07/220725-2.png)

**Half Message：**预处理消息，当broker收到此类消息后，会存储到RMQ_SYS_TRANS_HALF_TOPIC的消息消费队列中

**检查事务状态：**Broker会开启一个定时任务，消费RMQ_SYS_TRANS_HALF_TOPIC队列中的消息，每次执行任务会向消息发送者确认事务执行状态（提交、回滚、未知），如果是未知，Broker会定时去回调在重新检查。

**超时：**如果超过回查次数，默认回滚消息。

也就是他并未真正进入Topic的queue，而是用了临时queue来放所谓的half message，等提交事务后才会真正的将half message转移到topic下的queue。



#### 实现流程

**1.Producer发送半消息（Half Message）到broker。**

- Half Message又叫预发送消息（prepare message），发送成功后开始执行本地事务。
- 如果本地事务执行成功的话则返回commit，如果执行失败则返回rollback。（这个是在事务消息的回调方法里由开发者自己决定commit or rollback）



Producer发送上一步的commit还是rollback到broker，这里有两种情况：

**1.如果broker收到了commit/rollback消息 ：**

- 如果收到了commit，则broker认为整个事务是没问题的，执行成功的。那么会下发消息给Consumer端消费。
- 如果收到了rollback，则broker认为本地事务执行失败了，broker将会删除Half Message，不下发给Consumer端。



**2.如果broker未收到消息（如果执行本地事务突然宕机了，相当本地事务执行结果返回unknow，则和broker未收到确认消息的情况一样处理。）：**

- broker会定时回查本地事务的执行结果：如果回查结果是本地事务已经执行则返回commit，若未执行，则返回rollback。
- Producer端回查的结果发送给Broker。Broker接收到的如果是commit，则broker视为整个事务执行成功，如果是rollback，则broker视为本地事务执行失败，broker删除Half Message，不下发给consumer。如果broker未接收到回查的结果（或者查到的是unknow），则broker会**定时进行重复回查**，以确保查到最终的事务结果。重复回查的时间间隔和次数都可配。



1、生产者向MQ服务器发送half消息。
		2、half消息发送成功后，MQ服务器返回确认消息给生产者。
		3、生产者开始执行本地事务。
		4、根据本地事务执行的结果（`UNKNOW`、`commit`、`rollback`）向MQ Server发送提交或回滚消息。
		5、如果错过了（可能因为网络异常、生产者突然宕机等导致的异常情况）提交/回滚消息，则MQ服务器将向同一组中的每个生产者发送回查消息以获取事务状态。
		6、回查生产者本地事物状态。
		7、生产者根据本地事务状态发送提交/回滚消息。
		8、MQ服务器将丢弃回滚的消息，但已提交（进行过二次确认的half消息）的消息将投递给消费者进行消费。

`Half Message`：预处理消息，当broker收到此类消息后，会存储到`RMQ_SYS_TRANS_HALF_TOPIC`的消息消费队列中

`检查事务状态`：Broker会开启一个定时任务，消费`RMQ_SYS_TRANS_HALF_TOPIC`队列中的消息，每次执行任务会向消息发送者确认事务执行状态（提交、回滚、未知），如果是未知，Broker会定时去回调在重新检查。

超时：如果超过回查次数，默认回滚消息。
也就是他并未真正进入Topic的queue，而是用了临时queue来放所谓的`half message`，等提交事务后才会真正的将half message转移到topic下的queue。



1.producer(本例中指A系统)发送半消息到broker，这个半消息不是说消息内容不完整， 它包含完整的消息内容， 在producer端和普通消息的发送逻辑一致

2.broker存储半消息，半消息存储逻辑与普通消息一致，只是属性有所不同，topic是固定的RMQ_SYS_TRANS_HALF_TOPIC，queueId也是固定为0，这个tiopic中的消息对消费者是不可见的，所以里面的消息永远不会被消费。这就保证了在半消息提交成功之前，消费者是消费不到这个半消息的

3.broker端半消息存储成功并返回后，A系统执行本地事务，并根据本地事务的执行结果来决定半消息的提交状态为提交或者回滚

4.A系统发送结束半消息的请求，并带上提交状态(提交 or 回滚)

5.broker端收到请求后，首先从RMQ_SYS_TRANS_HALF_TOPIC的queue中查出该消息，设置为完成状态。如果消息状态为提交，则把半消息从RMQ_SYS_TRANS_HALF_TOPIC队列中复制到这个消息原始topic的queue中去(之后这条消息就能被正常消费了)；如果消息状态为回滚，则什么也不做。

6.producer发送的半消息结束请求是 oneway 的，也就是发送后就不管了，只靠这个是无法保证半消息一定被提交的，rocketMq提供了一个兜底方案，这个方案叫消息反查机制，Broker启动时，会启动一个TransactionalMessageCheckService 任务，该任务会定时从半消息队列中读出所有超时未完成的半消息，针对每条未完成的消息，Broker会给对应的Producer发送一个消息反查请求，根据反查结果来决定这个半消息是需要提交还是回滚，或者后面继续来反查

7.consumer(本例中指B系统)消费消息，执行本地数据变更(至于B是否能消费成功，消费失败是否重试，这属于正常消息消费需要考虑的问题)



#### 扩展

**Spring事务、常规的分布式事务不行吗？Rocketmq的事务是否多此一举了呢？**

MQ用于解耦，分布式事务直接操作了不同的系统，那么这些系统之间就是强耦合。果中间插了个mq，账号系统操作完发消息到mq，这时候只要保证发送成功就提交，发送失败则回滚，这步怎么保证，就是靠事务了。而且用RocketMQ做分布式事务的也蛮多的。



### 消息过滤

- **Broker**端消息过滤　　
	在**Broker**中，按照**Consumer**的要求做过滤，优点是减少了对于**Consumer**无用消息的网络传输。缺点是增加了Broker的负担，实现相对复杂。
- **Consumer**端消息过滤
	这种过滤方式可由应用完全自定义实现，但是缺点是很多无用的消息要传输到**Consumer**端。



### 回溯消费

回溯消费是指Consumer已经消费成功的消息，由于业务上的需求需要重新消费，要支持此功能，Broker在向Consumer投递成功消息后，消息仍然需要保留。并且重新消费一般是按照时间维度。

例如由于Consumer系统故障，恢复后需要重新消费1小时前的数据，那么Broker要提供一种机制，可以按照时间维度来回退消费进度。

**RocketMQ**支持按照时间回溯消费，时间维度精确到毫秒，可以向前回溯，也可以向后回溯。



### 延迟消息

定时消息是指消息发到**Broker**后，不能立刻被**Consumer**消费，要到特定的时间点或者等待特定的时间后才能被消费。

如果要支持任意的时间精度，在**Broker**层面，必须要做消息排序，如果再涉及到持久化，那么消息排序要不可避免的产生巨大性能开销。

**RocketMQ**支持定时消息，但是不支持任意时间精度，支持特定的level，例如定时5s，10s，1m等。



## 优化

### 开发

- 同一group下，多机部署，并行消费

- 单个Consumer提高消费线程个数

- 批量消费

	- 消息批量拉取
	- 业务逻辑批量处理

	


### 运维

- 网卡调优
- jvm调优
- 多线程与cpu调优
- Page Cache



## 源码解析

### 发送消息

```java
public class ProducerDemo {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("my-producer");
        // 设置namesrv
        producer.setNamesrvAddr("124.57.180.156:9876");
        producer.start();

        Message msg = new Message("myTopic001", "hello world".getBytes());
        SendResult result = producer.send(msg);
        System.out.println("发送消息成功！result is : " + result);
    }
}
```



#### 创建生产者

> org.apache.rocketmq.client.producer.DefaultMQProducer

```java
public DefaultMQProducer(final String producerGroup) {
    this(null, producerGroup, null);
}

public DefaultMQProducer(final String namespace, final String producerGroup, RPCHook rpcHook) {
    this.namespace = namespace;
    this.producerGroup = producerGroup;
    defaultMQProducerImpl = new DefaultMQProducerImpl(this, rpcHook);
}
```



#### 启动生产者

> org.apache.rocketmq.client.producer.DefaultMQProducer

```java
@Override
public void start() throws MQClientException {
    this.setProducerGroup(withNamespace(this.producerGroup));
    this.defaultMQProducerImpl.start();
    if (null != traceDispatcher) {
        try {
            traceDispatcher.start(this.getNamesrvAddr(), this.getAccessChannel());
        } catch (MQClientException e) {
            log.warn("trace dispatcher start failed ", e);
        }
    }
}
```



> org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl

```java
    public void start() throws MQClientException {
        this.start(true);
    }

    public void start(final boolean startFactory) throws MQClientException {
        switch (this.serviceState) {
            // 默认为CREATE_JUST状态
            case CREATE_JUST:
                //  先默认成启动失败，等最后完全启动成功的时候再置为ServiceState.RUNNING
                this.serviceState = ServiceState.START_FAILED;
                // 检查配置，比如group有没有写，是不是默认的那个名字，长度是不是超出限制了，等一系列验证。
                this.checkConfig();

                if (!this.defaultMQProducer.getProducerGroup().equals(MixAll.CLIENT_INNER_PRODUCER_GROUP)) {
                    this.defaultMQProducer.changeInstanceNameToPID();
                }
								// 单例模式，获取MQClientInstance对象，客户端实例。
                // 也就是Producer所部署的机器实例对象，负责操作的主要对象。
                this.mQClientFactory = MQClientManager.getInstance().getOrCreateMQClientInstance(this.defaultMQProducer, rpcHook);
								// 注册producer，其实就是往producerTable map里设置key-value
                boolean registerOK = mQClientFactory.registerProducer(this.defaultMQProducer.getProducerGroup(), this);
                if (!registerOK) {
                    this.serviceState = ServiceState.CREATE_JUST;
                    throw new MQClientException("The producer group[" + this.defaultMQProducer.getProducerGroup()
                        + "] has been created before, specify another name please." + FAQUrl.suggestTodo(FAQUrl.GROUP_NAME_DUPLICATE_URL),
                        null);
                }
                // 将topic信息存到topicPublishInfoTable这个map里
                this.topicPublishInfoTable.put(this.defaultMQProducer.getCreateTopicKey(), new TopicPublishInfo());

                if (startFactory) {
                    // 真正的启动核心类
                    mQClientFactory.start();
                }

                log.info("the producer [{}] start OK. sendMessageWithVIPChannel={}", this.defaultMQProducer.getProducerGroup(),
                    this.defaultMQProducer.isSendMessageWithVIPChannel());
                // 都启动完成，没报错的话，就将状态改为运行中
                this.serviceState = ServiceState.RUNNING;
                break;
            case RUNNING:
            case START_FAILED:
            case SHUTDOWN_ALREADY:
                throw new MQClientException("The producer service state not OK, maybe started once, "
                    + this.serviceState
                    + FAQUrl.suggestTodo(FAQUrl.CLIENT_SERVICE_NOT_OK),
                    null);
            default:
                break;
        }

        this.mQClientFactory.sendHeartbeatToAllBrokerWithLock();

        this.timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                try {
                    // 每隔1s扫描过期的请求
                    RequestFutureTable.scanExpiredRequest();
                } catch (Throwable e) {
                    log.error("scan RequestFutureTable exception", e);
                }
            }
        }, 1000 * 3, 1000);
    }
```



> org.apache.rocketmq.client.impl.factory.MQClientInstance

```java
    public void start() throws MQClientException {

        synchronized (this) {
            // 默认为CREATE_JUST状态
            switch (this.serviceState) {
                case CREATE_JUST:
                    // 先默认成启动失败，等最后完全启动成功的时候再置为ServiceState.RUNNING
                    this.serviceState = ServiceState.START_FAILED;
                    // If not specified,looking address from name server
                    if (null == this.clientConfig.getNamesrvAddr()) {
                        this.mQClientAPIImpl.fetchNameServerAddr();
                    }
                    // 启动请求响应通道，核心netty
                    this.mQClientAPIImpl.start();
                    // 启动各种定时任务
                    this.startScheduledTask();
                    // 启动拉取消息服务
                    this.pullMessageService.start();
                    // 启动Rebalance负载均衡服务
                    this.rebalanceService.start();
                    // Start push service
                    this.defaultMQProducer.getDefaultMQProducerImpl().start(false);
                    log.info("the client factory [{}] start OK", this.clientId);
                    // 都启动完成，没报错的话，就将状态改为运行中
                    this.serviceState = ServiceState.RUNNING;
                    break;
                case START_FAILED:
                    throw new MQClientException("The Factory object[" + this.getClientId() + "] has been created before, and failed.", null);
                default:
                    break;
            }
        }
    }
```



启动的定时任务如下：

- 每隔2分钟去检测namesrv的变化；
- 每隔30s从nameserver获取topic的路由信息有没有发生变化，或者说有没有新的topic路由信息；
- 每隔30s清除下线的broker；
- 每隔5s持久化所有的消费进度；
- 每隔1分钟检测线程池大小是否需要调整。



#### 发送消息

> org.apache.rocketmq.common.message.Message

```java
public Message(String topic, byte[] body) {
    this(topic, "", "", 0, body, true);
}

public Message(String topic, String tags, String keys, int flag, byte[] body, boolean waitStoreMsgOK) {
    this.topic = topic;
    this.flag = flag;
    this.body = body;
    if (tags != null && tags.length() > 0) {
        this.setTags(tags);
    }

    if (keys != null && keys.length() > 0) {
        this.setKeys(keys);
    }

    this.setWaitStoreMsgOK(waitStoreMsgOK);
}
```



> org.apache.rocketmq.client.producer.DefaultMQProducer

```java
@Override
public SendResult send(
    Message msg) throws MQClientException, RemotingException, MQBrokerException, InterruptedException {
    Validators.checkMessage(msg, this);
    msg.setTopic(withNamespace(msg.getTopic()));
    return this.defaultMQProducerImpl.send(msg);
}
```



> org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl

```java
    public SendResult send(
        Message msg) throws MQClientException, RemotingException, MQBrokerException, InterruptedException {
        return send(msg, this.defaultMQProducer.getSendMsgTimeout());
    }
    
    public SendResult send(Message msg,
        long timeout) throws MQClientException, RemotingException, MQBrokerException, InterruptedException {
        return this.sendDefaultImpl(msg, CommunicationMode.SYNC, null, timeout);
    }

    private SendResult sendDefaultImpl(
        Message msg,
        final CommunicationMode communicationMode,
        final SendCallback sendCallback,
        final long timeout
    ) throws MQClientException, RemotingException, MQBrokerException, InterruptedException {
        // 检查Producer上是否是RUNNING状态
        this.makeSureStateOK();
        // 消息格式的校验
        Validators.checkMessage(msg, this.defaultMQProducer);
        final long invokeID = random.nextLong();
        long beginTimestampFirst = System.currentTimeMillis();
        long beginTimestampPrev = beginTimestampFirst;
        long endTimestamp = beginTimestampFirst;
        // 尝试获取topic的路由信息
        TopicPublishInfo topicPublishInfo = this.tryToFindTopicPublishInfo(msg.getTopic());
        if (topicPublishInfo != null && topicPublishInfo.ok()) {
            boolean callTimeout = false;
            // 选择消息要发送的队列
            MessageQueue mq = null;
            Exception exception = null;
             // 发送结果
            SendResult sendResult = null;
            // 自动重试次数，this.defaultMQProducer.getRetryTimesWhenSendFailed()默认为2，
            // 如果是同步发送，默认重试3，否则重试1次
            int timesTotal = communicationMode == CommunicationMode.SYNC ? 1 + this.defaultMQProducer.getRetryTimesWhenSendFailed() : 1;
            int times = 0;
            String[] brokersSent = new String[timesTotal];
            for (; times < timesTotal; times++) {
                String lastBrokerName = null == mq ? null : mq.getBrokerName();
                // 选择topic的一个queue，然后往这个queue里发消息。
                MessageQueue mqSelected = this.selectOneMessageQueue(topicPublishInfo, lastBrokerName);
                if (mqSelected != null) {
                    // 给mq赋值，如果首次失败了，那么下次重试的时候（也就是下次for的时候），mq就有值了。
                    mq = mqSelected;
                    brokersSent[times] = mq.getBrokerName();
                    try {
                        // 发送开始时间
                        beginTimestampPrev = System.currentTimeMillis();
                        if (times > 0) {
                            //Reset topic with namespace during resend.
                            msg.setTopic(this.defaultMQProducer.withNamespace(msg.getTopic()));
                        }
                        long costTime = beginTimestampPrev - beginTimestampFirst;
                        if (timeout < costTime) {
                            callTimeout = true;
                            break;
                        }
                        // 真正的发消息方法
                        sendResult = this.sendKernelImpl(msg, mq, communicationMode, sendCallback, topicPublishInfo, timeout - costTime);
                        // 发送结束时间
                        endTimestamp = System.currentTimeMillis();
                        // 更新broker的延迟情况
                        this.updateFaultItem(mq.getBrokerName(), endTimestamp - beginTimestampPrev, false);
                        switch (communicationMode) {
                            case ASYNC:
                                return null;
                            case ONEWAY:
                                return null;
                            case SYNC:
                                // 同步的，将返回的结果返回，
                                // 如果返回结果状态不是成功的，则continue，进入下一次循环进行重试。 
                                if (sendResult.getSendStatus() != SendStatus.SEND_OK) {
                                    if (this.defaultMQProducer.isRetryAnotherBrokerWhenNotStoreOK()) {
                                        continue;
                                    }
                                }

                                return sendResult;
                            default:
                                break;
                        }
                    } catch (RemotingException e) {
                        endTimestamp = System.currentTimeMillis();
                        this.updateFaultItem(mq.getBrokerName(), endTimestamp - beginTimestampPrev, true);
                        log.warn(String.format("sendKernelImpl exception, resend at once, InvokeID: %s, RT: %sms, Broker: %s", invokeID, endTimestamp - beginTimestampPrev, mq), e);
                        log.warn(msg.toString());
                        exception = e;
                        continue;
                    } catch (MQClientException e) {
                        endTimestamp = System.currentTimeMillis();
                        this.updateFaultItem(mq.getBrokerName(), endTimestamp - beginTimestampPrev, true);
                        log.warn(String.format("sendKernelImpl exception, resend at once, InvokeID: %s, RT: %sms, Broker: %s", invokeID, endTimestamp - beginTimestampPrev, mq), e);
                        log.warn(msg.toString());
                        exception = e;
                        continue;
                    } catch (MQBrokerException e) {
                        endTimestamp = System.currentTimeMillis();
                        this.updateFaultItem(mq.getBrokerName(), endTimestamp - beginTimestampPrev, true);
                        log.warn(String.format("sendKernelImpl exception, resend at once, InvokeID: %s, RT: %sms, Broker: %s", invokeID, endTimestamp - beginTimestampPrev, mq), e);
                        log.warn(msg.toString());
                        exception = e;
                        switch (e.getResponseCode()) {
                            case ResponseCode.TOPIC_NOT_EXIST:
                            case ResponseCode.SERVICE_NOT_AVAILABLE:
                            case ResponseCode.SYSTEM_ERROR:
                            case ResponseCode.NO_PERMISSION:
                            case ResponseCode.NO_BUYER_ID:
                            case ResponseCode.NOT_IN_CURRENT_UNIT:
                                continue;
                            default:
                                if (sendResult != null) {
                                    return sendResult;
                                }

                                throw e;
                        }
                    } catch (InterruptedException e) {
                        endTimestamp = System.currentTimeMillis();
                        this.updateFaultItem(mq.getBrokerName(), endTimestamp - beginTimestampPrev, false);
                        log.warn(String.format("sendKernelImpl exception, throw exception, InvokeID: %s, RT: %sms, Broker: %s", invokeID, endTimestamp - beginTimestampPrev, mq), e);
                        log.warn(msg.toString());

                        log.warn("sendKernelImpl exception", e);
                        log.warn(msg.toString());
                        throw e;
                    }
                } else {
                    break;
                }
            }

            if (sendResult != null) {
                return sendResult;
            }

            String info = String.format("Send [%d] times, still failed, cost [%d]ms, Topic: %s, BrokersSent: %s",
                times,
                System.currentTimeMillis() - beginTimestampFirst,
                msg.getTopic(),
                Arrays.toString(brokersSent));

            info += FAQUrl.suggestTodo(FAQUrl.SEND_MSG_FAILED);

            MQClientException mqClientException = new MQClientException(info, exception);
            if (callTimeout) {
                throw new RemotingTooMuchRequestException("sendDefaultImpl call timeout");
            }

            if (exception instanceof MQBrokerException) {
                mqClientException.setResponseCode(((MQBrokerException) exception).getResponseCode());
            } else if (exception instanceof RemotingConnectException) {
                mqClientException.setResponseCode(ClientErrorCode.CONNECT_BROKER_EXCEPTION);
            } else if (exception instanceof RemotingTimeoutException) {
                mqClientException.setResponseCode(ClientErrorCode.ACCESS_BROKER_TIMEOUT);
            } else if (exception instanceof MQClientException) {
                mqClientException.setResponseCode(ClientErrorCode.BROKER_NOT_EXIST_EXCEPTION);
            }

            throw mqClientException;
        }

        validateNameServerSetting();

        throw new MQClientException("No route info of this topic: " + msg.getTopic() + FAQUrl.suggestTodo(FAQUrl.NO_TOPIC_ROUTE_INFO),
            null).setResponseCode(ClientErrorCode.NOT_FOUND_TOPIC_EXCEPTION);
    }
```



##### 更新Broker延迟情况

> org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl

```java
    public void updateFaultItem(final String brokerName, final long currentLatency, boolean isolation) {
        this.mqFaultStrategy.updateFaultItem(brokerName, currentLatency, isolation);
    }
```



> org.apache.rocketmq.client.latency.MQFaultStrategy

```java
    private long[] latencyMax = {50L, 100L, 550L, 1000L, 2000L, 3000L, 15000L};
    private long[] notAvailableDuration = {0L, 0L, 30000L, 60000L, 120000L, 180000L, 600000L};

    public void updateFaultItem(final String brokerName, final long currentLatency, boolean isolation) {
        if (this.sendLatencyFaultEnable) {
            // 首次isolation传入的是false，currentLatency是发送消息所耗费的时间
            long duration = computeNotAvailableDuration(isolation ? 30000 : currentLatency);
            this.latencyFaultTolerance.updateFaultItem(brokerName, currentLatency, duration);
        }
    }

    private long computeNotAvailableDuration(final long currentLatency) {
        // 根据延迟时间对比延迟级别数组latencyMax 和不可用时长数组notAvailableDuration 
        // 来将该broker加进faultItemTable中。
        for (int i = latencyMax.length - 1; i >= 0; i--) {
            if (currentLatency >= latencyMax[i])
                return this.notAvailableDuration[i];
        }

        return 0;
    }
```



> ​	org.apache.rocketmq.client.latency.LatencyFaultToleranceImpl

```java
    @Override
    public void updateFaultItem(final String name, final long currentLatency, final long notAvailableDuration) {
        FaultItem old = this.faultItemTable.get(name);
        if (null == old) {
            final FaultItem faultItem = new FaultItem(name);
            faultItem.setCurrentLatency(currentLatency);
            faultItem.setStartTimestamp(System.currentTimeMillis() + notAvailableDuration);

            old = this.faultItemTable.putIfAbsent(name, faultItem);
            if (old != null) {
                old.setCurrentLatency(currentLatency);
                old.setStartTimestamp(System.currentTimeMillis() + notAvailableDuration);
            }
        } else {
            old.setCurrentLatency(currentLatency);
            old.setStartTimestamp(System.currentTimeMillis() + notAvailableDuration);
        }
    }
```

给startTimestamp赋值为duration的结果，给isAvailable()所用。也就是说只有notAvailableDuration == 0的时候，isAvailable()才会返回true。



RocketMQ为每个Broker预测了个可用时间(当前时间+notAvailableDuration)，当当前时间大于该时间，才代表Broker可用，而notAvailableDuration有6个级别和latencyMax的区间一一对应，根据传入的currentLatency去预测该Broker在什么时候可用。

根据执行时间来看落入哪个区间，在0~100的时间内notAvailableDuration都是0，都是可用的，大于该值后，可用的时间就会开始变大了，就认为不是最优解，直接舍弃。



##### 普通消息发送失败

> org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl

```java
private SendResult sendDefaultImpl(
    Message msg,
    final CommunicationMode communicationMode,
    final SendCallback sendCallback,
    final long timeout
) throws MQClientException, RemotingException, MQBrokerException, InterruptedException {
    // ...
    if (topicPublishInfo != null && topicPublishInfo.ok()) {
        // ...
        MessageQueue mq = null;
        // ...
        for (; times < timesTotal; times++) {
            String lastBrokerName = null == mq ? null : mq.getBrokerName();
            MessageQueue mqSelected = this.selectOneMessageQueue(topicPublishInfo, lastBrokerName);
            if (mqSelected != null) {
                mq = mqSelected;
                // ...
                try {
                    // ...
                    switch (communicationMode) {
                        case ASYNC:
                            return null;
                        case ONEWAY:
                            return null;
                        case SYNC:
                            if (sendResult.getSendStatus() != SendStatus.SEND_OK) {
                                if (this.defaultMQProducer.isRetryAnotherBrokerWhenNotStoreOK()) {
                                    // 同步消息发送失败进行重试
                                    continue;
                                }
                            }
                            return sendResult;
                        default:
                            break;
                    }
                } // ...
            } else {
                break;
            }
        }
		// ...
    }
	// ...
}
```



> org.apache.rocketmq.client.impl.producer.TopicPublishInfo

```java
public MessageQueue selectOneMessageQueue(final String lastBrokerName) {
    if (lastBrokerName == null) {
        return selectOneMessageQueue();
    } else {
        int index = this.sendWhichQueue.getAndIncrement();
        for (int i = 0; i < this.messageQueueList.size(); i++) {
            int pos = Math.abs(index++) % this.messageQueueList.size();
            if (pos < 0)
                pos = 0;
            MessageQueue mq = this.messageQueueList.get(pos);
            // BrokerName 不等于上次的，才会返回
            if (!mq.getBrokerName().equals(lastBrokerName)) {
                return mq;
            }
        }
        return selectOneMessageQueue();
    }
}
```



发送消息时，如果Broker挂了，同步和异步消息会重试并发送到其它的Broker上。Producer 默认采用 round-robin 的方式，重试前会记录上一次发送消息的 Broker，然后选择下一个 Broker。



##### 延迟隔离策略

RocketMQ 有延迟隔离策略，如果发送某一个 Broker 失败了，会将其隔离，优先选择正常的 Broker 发送消息。**这个策略默认是不开启的。**



**开启延迟隔离策略**

```
DefaultMQProducer producer = new DefaultMQProducer("ProducerGroupName");
producer.setSendLatencyFaultEnable(true);
producer.start();
```



开启之后，发送消息时会记录发送消息花费的时间（下面 latencyMax 变量），超过一定时间，这个 Broker 就会在一段时间内不允许发送（下面 notAvailableDuration 变量）。

> org.apache.rocketmq.client.latency.MQFaultStrategy

```java
private long[] latencyMax = {50L, 100L, 550L, 1000L, 2000L, 3000L, 15000L};
private long[] notAvailableDuration = {0L, 0L, 30000L, 60000L, 120000L, 180000L, 600000L};  
```



> org.apache.rocketmq.client.latency.MQFaultStrategy

```java
public MessageQueue selectOneMessageQueue(final TopicPublishInfo tpInfo, final String lastBrokerName) {
    // sendLatencyFaultEnable为true表示开启了延迟隔离策略
    if (this.sendLatencyFaultEnable) {
        try {
            int index = tpInfo.getSendWhichQueue().getAndIncrement();
            for (int i = 0; i < tpInfo.getMessageQueueList().size(); i++) {
                int pos = Math.abs(index++) % tpInfo.getMessageQueueList().size();
                if (pos < 0)
                    pos = 0;
                MessageQueue mq = tpInfo.getMessageQueueList().get(pos);
                if (latencyFaultTolerance.isAvailable(mq.getBrokerName())) {
                    if (null == lastBrokerName || mq.getBrokerName().equals(lastBrokerName))
                        return mq;
                }
            }

            final String notBestBroker = latencyFaultTolerance.pickOneAtLeast();
            int writeQueueNums = tpInfo.getQueueIdByBroker(notBestBroker);
            if (writeQueueNums > 0) {
                final MessageQueue mq = tpInfo.selectOneMessageQueue();
                if (notBestBroker != null) {
                    mq.setBrokerName(notBestBroker);
                    mq.setQueueId(tpInfo.getSendWhichQueue().getAndIncrement() % writeQueueNums);
                }
                return mq;
            } else {
                latencyFaultTolerance.remove(notBestBroker);
            }
        } catch (Exception e) {
            log.error("Error occurred when selecting message queue", e);
        }

        return tpInfo.selectOneMessageQueue();
    }

    return tpInfo.selectOneMessageQueue(lastBrokerName);
}
```



##### 顺序消息发送失败

对于**全局顺序消息**，如果设置了所有消息要发送到同一个 Broker 的同一个 MessageQueue 中的情况，恰好是这个 Broker 挂了，那就只能等 Broker 重启后再发送了。而对于**局部顺序消息**，比如同一个订单相关的消息要发送到同一个 Broker 的同一个 MessageQueue 中的情况，如果这个 Broker 挂了，那 MessageQueueSelector 会选择其他 Broker 上的 MessageQueue 进行发送，这会影响当前这笔订单消费的顺序性。而其他订单可以被 Producer 发送到其他的队列中，不受影响。

Broker1 挂之前，Order1 的消息发送到了 Broker1，Broker1 挂之后，Order1 的消息被发送到了 Broker2。在 Broker1 恢复前，消费者只能消费 Broker2 上拉取 Order1 的消息，Broker1 恢复后消费者线程再从 Broker1 拉取，因此 Order1 的消息产生乱序。**这里假设没有从节点**。



#### 总结

首先需要配置好生产者组名、namesrv地址和topic以及要发送的消息内容，然后启动Producer的start()方法，启动完成后调用send()方法进行发送。

start()方法内部会进行检查namesrv、生产者组名等参数验证，然后内部会获取一个mQClientFactory对象，此对象内包含了所有与Broker进行通信的api，然后通过mQClientFactory启动请求响应通道，主要是netty，接下来启动一些定时任务，比如与broker的心跳等，还会启动负载均衡服务等，最后都启动成功的话将服务的状态标记为RUNNING。

启动完成后调用send()方法发消息，有三种发送方式，同步、异步、oneWay，都大同小异，唯一的区别的就是异步的多个线程池去异步调用发送请求，而同步则是当前请求线程直接同步调用的，核心流程都是：

先选择一个合适的queue来存储消息，选择完后拼凑一个header参数对象，通过netty的形式发送给broker。

这里值得注意的是：如果发送失败的话他会自动重试，默认同步发送的次数是3次，也就是失败后会自动重试2次。



### 发送消息Queue选择算法

```java
    /*
		 * 只发送消息，queue的选择由默认的算法来实现
		 */
    @Override
    public SendResult send(
        Collection<Message> msgs) throws MQClientException, RemotingException, MQBrokerException, InterruptedException {
        return this.defaultMQProducerImpl.send(batch(msgs));
    }
    
		/*
		 * 自定义选择queue的算法进行发消息
		 */
    @Override
    public SendResult send(Collection<Message> msgs,
        MessageQueue messageQueue) throws MQClientException, RemotingException, MQBrokerException, InterruptedException {
        return this.defaultMQProducerImpl.send(batch(msgs), messageQueue);
    }
```



#### 默认Queue发送

> org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl

```java
public MessageQueue selectOneMessageQueue(final TopicPublishInfo tpInfo, final String lastBrokerName) {
    return this.mqFaultStrategy.selectOneMessageQueue(tpInfo, lastBrokerName);
}
```



> org.apache.rocketmq.client.latency.MQFaultStrategy

```java
public MessageQueue selectOneMessageQueue(final TopicPublishInfo tpInfo, final String lastBrokerName) {
    // 默认为false，代表不启用broker故障延迟
    if (this.sendLatencyFaultEnable) {
        try {
            // 随机数且+1
            int index = tpInfo.getSendWhichQueue().getAndIncrement();
            for (int i = 0; i < tpInfo.getMessageQueueList().size(); i++) {
                // 先（随机数 +1） % queue.size()
                int pos = Math.abs(index++) % tpInfo.getMessageQueueList().size();
                if (pos < 0)
                    pos = 0;
                MessageQueue mq = tpInfo.getMessageQueueList().get(pos);
                // 看找到的这个queue所属的broker是不是可用的
                if (latencyFaultTolerance.isAvailable(mq.getBrokerName())) {
                    // 非失败重试，直接返回到的队列
                    // 失败重试的情况，如果和选择的队列是上次重试是一样的，则返回
                    
                    // 也就是说如果你这个queue所在的broker可用，
                    // 且不是重试进来的或失败重试的情况，如果和选择的队列是上次重试是一样的。
                    if (null == lastBrokerName || mq.getBrokerName().equals(lastBrokerName))
                        return mq;
                }
            }
            // 如果所有队列都不可用，那么选择一个相对好的broker，不考虑可用性的消息队列
            final String notBestBroker = latencyFaultTolerance.pickOneAtLeast();
            int writeQueueNums = tpInfo.getQueueIdByBroker(notBestBroker);
            if (writeQueueNums > 0) {
                final MessageQueue mq = tpInfo.selectOneMessageQueue();
                if (notBestBroker != null) {
                    mq.setBrokerName(notBestBroker);
                    mq.setQueueId(tpInfo.getSendWhichQueue().getAndIncrement() % writeQueueNums);
                }
                return mq;
            } else {
                latencyFaultTolerance.remove(notBestBroker);
            }
        } catch (Exception e) {
            log.error("Error occurred when selecting message queue", e);
        }
        // 随机选择一个queue
        return tpInfo.selectOneMessageQueue();
    }
     // 当sendLatencyFaultEnable=false的时候选择queue的方法，默认就是false。
    return tpInfo.selectOneMessageQueue(lastBrokerName);
}
```



##### 不启用broker故障延迟

即上述if代码块中的逻辑。

> org.apache.rocketmq.client.latency.MQFaultStrategy

```java
if (this.sendLatencyFaultEnable) {
    ....
}
```

先`（随机数 +1） % queue.size()`，然后看你这个queue所属的broker是否可用，可用的话且不是重试进来的或失败重试的情况，如果和选择的队列是上次重试是一样的，那直接return你就完事了。那么怎么看broker是否可用的呢？



> org.apache.rocketmq.client.latency.LatencyFaultToleranceImpl

```java
    @Override
    public boolean isAvailable(final String name) {
        final FaultItem faultItem = this.faultItemTable.get(name);
        if (faultItem != null) {
            return faultItem.isAvailable();
        }
        return true;
    }
```



> org.apache.rocketmq.client.latency.LatencyFaultToleranceImpl.FaultItem

```java
        public boolean isAvailable() {
            return (System.currentTimeMillis() - startTimestamp) >= 0;
        }
```



##### 启用broker故障延迟

> org.apache.rocketmq.client.impl.producer.TopicPublishInfo

```java
public MessageQueue selectOneMessageQueue(final String lastBrokerName) {
    // 第一次就是null，第二次（也就是重试的时候）就不是null了。
    if (lastBrokerName == null) {
        // 第一次选择队列的逻辑
        return selectOneMessageQueue();
    } else {
        // 第一次选择队列发送消息失败了，第二次重试的时候选择队列的逻辑
        int index = this.sendWhichQueue.getAndIncrement();
        for (int i = 0; i < this.messageQueueList.size(); i++) {
            int pos = Math.abs(index++) % this.messageQueueList.size();
            if (pos < 0)
                pos = 0;
            MessageQueue mq = this.messageQueueList.get(pos);
            // 过滤掉上次发送消息失败的队列
            if (!mq.getBrokerName().equals(lastBrokerName)) {
                return mq;
            }
        }
        // 没找到能用的queue的话继续走默认的那个
        return selectOneMessageQueue();
    }
}

    public MessageQueue selectOneMessageQueue() {
        // 当前线程有个ThreadLocal变量，存放了一个随机数
        // 然后取出随机数根据队列长度取模且将随机数+1
        int index = this.sendWhichQueue.getAndIncrement();
        int pos = Math.abs(index) % this.messageQueueList.size();
        if (pos < 0)
            pos = 0;
        return this.messageQueueList.get(pos);
    }
```



好吧，其实也有点随机一个的意思。但是亮点在于取出随机数根据队列长度取模且将随机数+1，这个+1亮了（getAndIncrement cas +1）。

当消息第一次发送失败时，lastBrokerName会存放当前选择失败的broker（mq = mqSelected），通过重试，此时lastBrokerName有值，代表上次选择的boker发送失败，则重新对sendWhichQueue本地线程变量+1，遍历选择消息队列，直到不是上次的broker，也就是为了规避上次发送失败的broker的逻辑所在。

举个例子：你这次随机数是1，队列长度是4，1%4=1，这时候失败了，进入重试，那么重试之前，也就是在上一步1%4之后，他把1进行了++操作，变成了2，那么你这次重试的时候就是2%4=2，直接过滤掉了刚才失败的broker。



so easy，你上次不是失败了，进入我这里重试来了吗？我也很简单，我就还是取出随机数+1然后取模队列长度，我看这个broker是不是上次失败的那个，是他小子的话就过滤掉，继续遍历queue找下一个能用的。



##### 总结

- 在不开启容错的情况下，轮询队列进行发送，如果失败了，重试的时候过滤失败的Broker
- 如果开启了容错策略，会通过RocketMQ的预测机制来预测一个Broker是否可用
- 如果上次失败的Broker可用那么还是会选择该Broker的队列
- 如果上述情况失败，则随机选择一个进行发送
- 在发送消息的时候会记录一下调用的时间与是否报错，根据该时间去预测broker的可用时间



### Broker接收消息

| 类              | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| MappedFile      | 对应的是commitlog文件。                                      |
| MappedFileQueue | 是`MappedFile` 所在的文件夹，对 `MappedFile` 进行封装成文件队列。 |
| CommitLog       | 针对 `MappedFileQueue` 的封装使用。                          |



#### 接收消息

调用链如下：

BrokerStartup.start() -》

 BrokerController.start() -》 

NettyRemotingServer.start() -》 

NettyRemotingServer.prepareSharableHandlers() -》 

new NettyServerHandler() -》 

NettyRemotingAbstract.processMessageReceived() -》 

NettyRemotingAbstract.processRequestCommand() -》 

SendMessageProcessor.processRequest()



> SendMessageProcessor.processRequest()

```java
@Override
public RemotingCommand processRequest(ChannelHandlerContext ctx,
                                      RemotingCommand request) throws RemotingCommandException {
    RemotingCommand response = null;
    try {
        // 调用asyncProcessRequest
        response = asyncProcessRequest(ctx, request).get();
    } catch (InterruptedException | ExecutionException e) {
        log.error("process SendMessage error, request : " + request.toString(), e);
    }
    return response;
}
```



```java
public CompletableFuture<RemotingCommand> asyncProcessRequest(ChannelHandlerContext ctx,
                                                                  RemotingCommand request) throws RemotingCommandException {
    final SendMessageContext mqtraceContext;
    switch (request.getCode()) {
        // 表示消费者发送的消息，发送者消费失败会重新发回队列进行消息重试
        case RequestCode.CONSUMER_SEND_MSG_BACK:
            return this.asyncConsumerSendMsgBack(ctx, request);
        default:
            // 解析header，也就是我们Producer发送过来的消息都在request里，给他解析到SendMessageRequestHeader对象里去。
            SendMessageRequestHeader requestHeader = parseRequestHeader(request);
            if (requestHeader == null) {
                return CompletableFuture.completedFuture(null);
            }
            mqtraceContext = buildMsgContext(ctx, requestHeader);
            // 将解析好的参数放到SendMessageContext对象里
            this.executeSendMessageHookBefore(ctx, request, mqtraceContext);
            if (requestHeader.isBatch()) {
                // 批处理消息用
                return this.asyncSendBatchMessage(ctx, request, mqtraceContext, requestHeader);
            } else {
                // 非批处理，我们这里介绍的核心。
                return this.asyncSendMessage(ctx, request, mqtraceContext, requestHeader);
            }
    }
}
```



```java
private CompletableFuture<RemotingCommand> asyncSendMessage(ChannelHandlerContext ctx, RemotingCommand request,
                                                                SendMessageContext mqtraceContext,
                                                                SendMessageRequestHeader requestHeader) {
    final byte[] body = request.getBody();

    int queueIdInt = requestHeader.getQueueId();
    TopicConfig topicConfig = this.brokerController.getTopicConfigManager().selectTopicConfig(requestHeader.getTopic());

    // 拼凑message对象
    MessageExtBrokerInner msgInner = new MessageExtBrokerInner();
    msgInner.setTopic(requestHeader.getTopic());
    msgInner.setQueueId(queueIdInt);
    msgInner.setBody(body);
    msgInner.setFlag(requestHeader.getFlag());
    MessageAccessor.setProperties(msgInner, MessageDecoder.string2messageProperties(requestHeader.getProperties()));
    msgInner.setPropertiesString(requestHeader.getProperties());
    msgInner.setBornTimestamp(requestHeader.getBornTimestamp());
    msgInner.setBornHost(ctx.channel().remoteAddress());
    msgInner.setStoreHost(this.getStoreHost());
    msgInner.setReconsumeTimes(requestHeader.getReconsumeTimes() == null ? 0 : requestHeader.getReconsumeTimes());
    
    CompletableFuture<PutMessageResult> putMessageResult = null;
    Map<String, String> origProps = MessageDecoder.string2messageProperties(requestHeader.getProperties());
    // 真正接收消息的方法
    putMessageResult = this.brokerController.getMessageStore().asyncPutMessage(msgInner);
    return handlePutMessageResultFuture(putMessageResult, response, request, msgInner, responseHeader, mqtraceContext, ctx, queueIdInt);
}
```



至此我们的消息接收完成了，都封装到了MessageExtBrokerInner对象里。



#### 持久化消息

![img](../../Image/2022/07/220725-4.png)

```java
@Override
public CompletableFuture<PutMessageResult> asyncPutMessage(MessageExtBrokerInner msg) {
    CompletableFuture<PutMessageResult> putResultFuture = this.commitLog.asyncPutMessage(msg);
    putResultFuture.thenAccept((result) -> {
        ......
    });
    return putResultFuture;
}
```



```java

public CompletableFuture<PutMessageResult> asyncPutMessage(final MessageExtBrokerInner msg) {
    // 获取最后一个文件，MappedFile就是commitlog目录下的那个0000000000文件
    MappedFile mappedFile = this.mappedFileQueue.getLastMappedFile();
    try {
        // 追加数据到commitlog
        result = mappedFile.appendMessage(msg, this.appendMessageCallback);
        switch (result.getStatus()) {
            ......
        }
        // 将内存的数据持久化到磁盘
        CompletableFuture<PutMessageStatus> flushResultFuture = submitFlushRequest(result, putMessageResult, msg);
    }
}
```



```java
public AppendMessageResult appendMessagesInner(final MessageExt messageExt, final AppendMessageCallback cb) {
    // 将消息写到内存
    return cb.doAppend(this.getFileFromOffset(), byteBuffer, this.fileSize - currentPos, (MessageExtBrokerInner) messageExt);
}
```



```java
@Override
public AppendMessageResult doAppend(final long fileFromOffset, final ByteBuffer byteBuffer, final int maxBlank,
                                    final MessageExtBrokerInner msgInner) {
    // Initialization of storage space
    this.resetByteBuffer(msgStoreItemMemory, msgLen);
    // 1 TOTALSIZE
    this.msgStoreItemMemory.putInt(msgLen);
    // 2 MAGICCODE
    this.msgStoreItemMemory.putInt(CommitLog.MESSAGE_MAGIC_CODE);
    // 3 BODYCRC
    this.msgStoreItemMemory.putInt(msgInner.getBodyCRC());
    // 4 QUEUEID
    this.msgStoreItemMemory.putInt(msgInner.getQueueId());
    // 5 FLAG
    this.msgStoreItemMemory.putInt(msgInner.getFlag());
    // 6 QUEUEOFFSET
    this.msgStoreItemMemory.putLong(queueOffset);
    // 7 PHYSICALOFFSET
    this.msgStoreItemMemory.putLong(fileFromOffset + byteBuffer.position());
    // 8 SYSFLAG
    this.msgStoreItemMemory.putInt(msgInner.getSysFlag());
    // 9 BORNTIMESTAMP
    this.msgStoreItemMemory.putLong(msgInner.getBornTimestamp());
    // 10 BORNHOST
    this.resetByteBuffer(bornHostHolder, bornHostLength);
    this.msgStoreItemMemory.put(msgInner.getBornHostBytes(bornHostHolder));
    // 11 STORETIMESTAMP
    this.msgStoreItemMemory.putLong(msgInner.getStoreTimestamp());
    // 12 STOREHOSTADDRESS
    this.resetByteBuffer(storeHostHolder, storeHostLength);
    this.msgStoreItemMemory.put(msgInner.getStoreHostBytes(storeHostHolder));
    // 13 RECONSUMETIMES
    this.msgStoreItemMemory.putInt(msgInner.getReconsumeTimes());
    // 14 Prepared Transaction Offset
    this.msgStoreItemMemory.putLong(msgInner.getPreparedTransactionOffset());
    // 15 BODY
    this.msgStoreItemMemory.putInt(bodyLength);
    if (bodyLength > 0)
        this.msgStoreItemMemory.put(msgInner.getBody());
    // 16 TOPIC
    this.msgStoreItemMemory.put((byte) topicLength);
    this.msgStoreItemMemory.put(topicData);
    // 17 PROPERTIES
    this.msgStoreItemMemory.putShort((short) propertiesLength);
    if (propertiesLength > 0)
        this.msgStoreItemMemory.put(propertiesData);

    final long beginTimeMills = CommitLog.this.defaultMessageStore.now();
    // Write messages to the queue buffer
    byteBuffer.put(this.msgStoreItemMemory.array(), 0, msgLen);
    return result;
}
```



这一步已经把消息保存到缓冲区里了，也就是msgStoreItemMemory，这里采取的NIO。下一步就要刷盘了。



```java
public CompletableFuture<PutMessageStatus> submitFlushRequest(AppendMessageResult result, PutMessageResult putMessageResult,
                                                              MessageExt messageExt) {
    // 同步刷盘
    if (FlushDiskType.SYNC_FLUSH == this.defaultMessageStore.getMessageStoreConfig().getFlushDiskType()) {
        // // 同步刷盘service -> GroupCommitService
        final GroupCommitService service = (GroupCommitService) this.flushCommitLogService;
        if (messageExt.isWaitStoreMsgOK()) {
            // 数据准备
            GroupCommitRequest request = new GroupCommitRequest(result.getWroteOffset() + result.getWroteBytes(),
                                                                this.defaultMessageStore.getMessageStoreConfig().getSyncFlushTimeout());
            // 将数据对象放到requestsWrite里
            service.putRequest(request);
            return request.future();
        } else {
            service.wakeup();
            return CompletableFuture.completedFuture(PutMessageStatus.PUT_OK);
        }
    }
    // 异步刷盘
    else {
        if (!this.defaultMessageStore.getMessageStoreConfig().isTransientStorePoolEnable()) {
            
            flushCommitLogService.wakeup();
        } else  {
            commitLogService.wakeup();
        }
        return CompletableFuture.completedFuture(PutMessageStatus.PUT_OK);
    }
}
```



##### 同步刷盘

```java
public synchronized void putRequest(final GroupCommitRequest request) {
    synchronized (this.requestsWrite) {
        this.requestsWrite.add(request);
    }
    // 这里很关键！！！，给他设置成true。然后计数器-1。下面run方法的时候才会进行交换数据且return
    if (hasNotified.compareAndSet(false, true)) {
        waitPoint.countDown(); // notify
    }
}
```



```java
public void run() {
    while (!this.isStopped()) {
        try {
            // 是同步还是异步的关键方法，也就是说组不阻塞全看这里。
            this.waitForRunning(10);
            // 真正的刷盘逻辑
            this.doCommit();
        } catch (Exception e) {
            CommitLog.log.warn(this.getServiceName() + " service has exception. ", e);
        }
    }
}
```



```java
protected volatile AtomicBoolean hasNotified = new AtomicBoolean(false);
// 其实就是CountDownLatch
protected final CountDownLatch2 waitPoint = new CountDownLatch2(1);

protected void waitForRunning(long interval) {
    // 如果是true，且给他改成false成功的话，则onWaitEnd()且return，但是默认是false，也就是默认情况下这个if不会进。
    if (hasNotified.compareAndSet(true, false)) {
        this.onWaitEnd();
        return;
    }

    //entry to wait
    waitPoint.reset();

    try {
        // 等待，默认值是1，也就是waitPoint.countDown()一次后就会激活这里。
        waitPoint.await(interval, TimeUnit.MILLISECONDS);
    } catch (InterruptedException e) {
        log.error("Interrupted", e);
    } finally {
        // 给状态值设置成false
        hasNotified.set(false);
        this.onWaitEnd();
    }
}
```



总结下同步刷盘的主要流程：

> 核心类是GroupCommitService，核心方法 是waitForRunning。

- 先调用putRequest方法将hasNotified变为true，且进行notify，也就是`waitPoint.countDown()`。
- 其次是run方法里的`waitForRunning()`，`waitForRunning()`判断hasNotified是不是true，是true则交换数据然后return掉，也就是不进行await阻塞，直接return。
- 最后上一步return了，没有阻塞，那么顺理成章的调用doCommit进行真正意义的刷盘。



##### 异步刷盘

```java
class FlushRealTimeService extends FlushCommitLogService {
    @Override
    public void run() {
        while (!this.isStopped()) {
            try {
    // 每隔500ms刷一次盘
                if (flushCommitLogTimed) {
                    Thread.sleep(500);
                } else {
                    // 根上面同步刷盘调用的是同一个方法，区别在于这里没有将hasNotified变为true，
                    // 也就是还是默认的false，那么waitForRunning方法内部的第一个判断就不会走，
                    // 就不会return掉，就会进行下面的await方法阻塞，默认阻塞时间是500毫秒。
                    // 也就是默认500ms刷一次盘。
                    this.waitForRunning(500);
                }
                // 调用mappedFileQueue的flush方法
                CommitLog.this.mappedFileQueue.flush(flushPhysicQueueLeastPages);
            } catch (Throwable e) {
            }
        }
    }
}
```



```java
public boolean flush(final int flushLeastPages) {
    MappedFile mappedFile = this.findMappedFileByOffset(this.flushedWhere, this.flushedWhere == 0);
    if (mappedFile != null) {
        // 真正的刷盘操作
        int offset = mappedFile.flush(flushLeastPages);
    }
}
```



```java
public int flush(final int flushLeastPages) {
    if (this.isAbleToFlush(flushLeastPages)) {
        try {
            if (writeBuffer != null || this.fileChannel.position() != 0) {
                // 刷盘   NIO
                this.fileChannel.force(false);
            } else {
    // 刷盘  NIO
                this.mappedByteBuffer.force();
            }
        } catch (Throwable e) {
            log.error("Error occurred when force data to disk.", e);
        }
    }
    return this.getFlushedPosition();
}
```



- 判断`flushCommitLogTimed`是不是true，默认false，是true则直接sleep(500ms)然后进行`mappedFileQueue.flush()`刷盘。
- 若是false，则进入`waitForRunning(500)`，这里是和同步刷盘的区别关键所在，同步刷盘之前将hasNotified变为true了，所以直接一套小连招：`return+doCommit`了 ，异步这里直接调用的`waitForRunning(500)`，在这之前没任何对hasNotified的操作，所以不会return，而是会继续走下面的`waitPoint.await(500, TimeUnit.MILLISECONDS);`进行阻塞500毫秒，500毫秒后自动唤醒然后进行flush刷盘。也就是异步刷盘的话默认500ms刷盘一次。



核心类MappedFile对应的是每个commitlog文件，MappedFileQueue相当于文件夹，管理所有的文件，还有一个管理者CommitLog对象，他负责提供一些操作。具体的是Broker端拿到消息后先将消息、topic、queue等内容存到ByteBuffer里，然后去持久化到commitlog文件中。commitlog文件大小为1G，超出大小会新创建commitlog文件来存储，采取的nio方式。



### 消费者启动

> org.apache.rocketmq.client.consumer.DefaultMQPushConsumer

```java    @Override
    @Override
    public void start() throws MQClientException {
        setConsumerGroup(NamespaceUtil.wrapNamespace(this.getNamespace(), this.consumerGroup));
        this.defaultMQPushConsumerImpl.start();
        if (null != traceDispatcher) {
            try {
                traceDispatcher.start(this.getNamesrvAddr(), this.getAccessChannel());
            } catch (MQClientException e) {
                log.warn("trace dispatcher start failed ", e);
            }
        }
    }
```



> org.apache.rocketmq.client.impl.consumer.DefaultMQPushConsumerImpl

```java
public synchronized void start() throws MQClientException {
        switch (this.serviceState) {
            case CREATE_JUST:
                log.info("the consumer [{}] start beginning. messageModel={}, isUnitMode={}", this.defaultMQPushConsumer.getConsumerGroup(),
                    this.defaultMQPushConsumer.getMessageModel(), this.defaultMQPushConsumer.isUnitMode());
                this.serviceState = ServiceState.START_FAILED;

                this.checkConfig();

                this.copySubscription();

                if (this.defaultMQPushConsumer.getMessageModel() == MessageModel.CLUSTERING) {
                    this.defaultMQPushConsumer.changeInstanceNameToPID();
                }

                this.mQClientFactory = MQClientManager.getInstance().getOrCreateMQClientInstance(this.defaultMQPushConsumer, this.rpcHook);

                this.rebalanceImpl.setConsumerGroup(this.defaultMQPushConsumer.getConsumerGroup());
                this.rebalanceImpl.setMessageModel(this.defaultMQPushConsumer.getMessageModel());
                this.rebalanceImpl.setAllocateMessageQueueStrategy(this.defaultMQPushConsumer.getAllocateMessageQueueStrategy());
                this.rebalanceImpl.setmQClientFactory(this.mQClientFactory);

                this.pullAPIWrapper = new PullAPIWrapper(
                    mQClientFactory,
                    this.defaultMQPushConsumer.getConsumerGroup(), isUnitMode());
                this.pullAPIWrapper.registerFilterMessageHook(filterMessageHookList);

                if (this.defaultMQPushConsumer.getOffsetStore() != null) {
                    this.offsetStore = this.defaultMQPushConsumer.getOffsetStore();
                } else {
                    switch (this.defaultMQPushConsumer.getMessageModel()) {
                        case BROADCASTING:
                            this.offsetStore = new LocalFileOffsetStore(this.mQClientFactory, this.defaultMQPushConsumer.getConsumerGroup());
                            break;
                        case CLUSTERING:
                            this.offsetStore = new RemoteBrokerOffsetStore(this.mQClientFactory, this.defaultMQPushConsumer.getConsumerGroup());
                            break;
                        default:
                            break;
                    }
                    this.defaultMQPushConsumer.setOffsetStore(this.offsetStore);
                }
                this.offsetStore.load();

                if (this.getMessageListenerInner() instanceof MessageListenerOrderly) {
                    this.consumeOrderly = true;
                    this.consumeMessageService =
                        new ConsumeMessageOrderlyService(this, (MessageListenerOrderly) this.getMessageListenerInner());
                } else if (this.getMessageListenerInner() instanceof MessageListenerConcurrently) {
                    this.consumeOrderly = false;
                    this.consumeMessageService =
                        new ConsumeMessageConcurrentlyService(this, (MessageListenerConcurrently) this.getMessageListenerInner());
                }

                this.consumeMessageService.start();

                boolean registerOK = mQClientFactory.registerConsumer(this.defaultMQPushConsumer.getConsumerGroup(), this);
                if (!registerOK) {
                    this.serviceState = ServiceState.CREATE_JUST;
                    this.consumeMessageService.shutdown(defaultMQPushConsumer.getAwaitTerminationMillisWhenShutdown());
                    throw new MQClientException("The consumer group[" + this.defaultMQPushConsumer.getConsumerGroup()
                        + "] has been created before, specify another name please." + FAQUrl.suggestTodo(FAQUrl.GROUP_NAME_DUPLICATE_URL),
                        null);
                }

                mQClientFactory.start();
                log.info("the consumer [{}] start OK.", this.defaultMQPushConsumer.getConsumerGroup());
                this.serviceState = ServiceState.RUNNING;
                break;
            case RUNNING:
            case START_FAILED:
            case SHUTDOWN_ALREADY:
                throw new MQClientException("The PushConsumer service state not OK, maybe started once, "
                    + this.serviceState
                    + FAQUrl.suggestTodo(FAQUrl.CLIENT_SERVICE_NOT_OK),
                    null);
            default:
                break;
        }

        this.updateTopicSubscribeInfoWhenSubscriptionChanged();
        this.mQClientFactory.checkClientInBroker();
        this.mQClientFactory.sendHeartbeatToAllBrokerWithLock();
        // 立刻负载均衡
        this.mQClientFactory.rebalanceImmediately();
    }
```



> org.apache.rocketmq.client.impl.factory.MQClientInstance

```java
    public void start() throws MQClientException {

        synchronized (this) {
            switch (this.serviceState) {
                case CREATE_JUST:
                    this.serviceState = ServiceState.START_FAILED;
                    // If not specified,looking address from name server
                    if (null == this.clientConfig.getNamesrvAddr()) {
                        this.mQClientAPIImpl.fetchNameServerAddr();
                    }
                    // Start request-response channel
                    this.mQClientAPIImpl.start();
                    // Start various schedule tasks
                    this.startScheduledTask();
                    // Start pull service
                    this.pullMessageService.start();
                    // 启动负载均衡服务
                    this.rebalanceService.start();
                    // Start push service
                    this.defaultMQProducer.getDefaultMQProducerImpl().start(false);
                    log.info("the client factory [{}] start OK", this.clientId);
                    this.serviceState = ServiceState.RUNNING;
                    break;
                case START_FAILED:
                    throw new MQClientException("The Factory object[" + this.getClientId() + "] has been created before, and failed.", null);
                default:
                    break;
            }
        }
    }
```



### 消费负载均衡

#### 负载均衡方式

> org.apache.rocketmq.client.impl.consumer.RebalancePushImpl

```java
    public RebalancePushImpl(String consumerGroup, MessageModel messageModel,
        AllocateMessageQueueStrategy allocateMessageQueueStrategy,
        MQClientInstance mQClientFactory, DefaultMQPushConsumerImpl defaultMQPushConsumerImpl) {
        // 可以看到很简单，调用了父类RebalanceImpl的构造器
        super(consumerGroup, messageModel, allocateMessageQueueStrategy, mQClientFactory);
        this.defaultMQPushConsumerImpl = defaultMQPushConsumerImpl;
    }
```



> org.apache.rocketmq.client.impl.consumer.RebalanceImpl

```java
    public void doRebalance(final boolean isOrder) {
        Map<String, SubscriptionData> subTable = this.getSubscriptionInner();
        if (subTable != null) {
            for (final Map.Entry<String, SubscriptionData> entry : subTable.entrySet()) {
                final String topic = entry.getKey();
                try {
                    this.rebalanceByTopic(topic, isOrder);
                } catch (Throwable e) {
                    if (!topic.startsWith(MixAll.RETRY_GROUP_TOPIC_PREFIX)) {
                        log.warn("rebalanceByTopic Exception", e);
                    }
                }
            }
        }
        // 移除未订阅的topic对应的消息队列
        this.truncateMessageQueueNotMyTopic();
    }

    private void rebalanceByTopic(final String topic, final boolean isOrder) {
        switch (messageModel) {
            case BROADCASTING: {
                Set<MessageQueue> mqSet = this.topicSubscribeInfoTable.get(topic);
                if (mqSet != null) {
                    
                    boolean changed = this.updateProcessQueueTableInRebalance(topic, mqSet, isOrder);
                    if (changed) {
                        this.messageQueueChanged(topic, mqSet, mqSet);
                        log.info("messageQueueChanged {} {} {} {}",
                            consumerGroup,
                            topic,
                            mqSet,
                            mqSet);
                    }
                } else {
                    log.warn("doRebalance, {}, but the topic[{}] not exist.", consumerGroup, topic);
                }
                break;
            }
            case CLUSTERING: {
                Set<MessageQueue> mqSet = this.topicSubscribeInfoTable.get(topic);
                // 所有的Consumer客户端cid，比如：172.16.20.246@7832
                List<String> cidAll = this.mQClientFactory.findConsumerIdList(topic, consumerGroup);
                if (null == mqSet) {
                    if (!topic.startsWith(MixAll.RETRY_GROUP_TOPIC_PREFIX)) {
                        log.warn("doRebalance, {}, but the topic[{}] not exist.", consumerGroup, topic);
                    }
                }

                if (null == cidAll) {
                    log.warn("doRebalance, {} {}, get consumer id list failed", consumerGroup, topic);
                }

                if (mqSet != null && cidAll != null) {
                    List<MessageQueue> mqAll = new ArrayList<MessageQueue>();
                    mqAll.addAll(mqSet);
                    // 排序消息队列和消费者数组，因为是在进行分配队列，排序后，各Client的顺序才能保持一致
                    Collections.sort(mqAll);
                    Collections.sort(cidAll);
										// 默认AllocateMessageQueueAveragely
                    AllocateMessageQueueStrategy strategy = this.allocateMessageQueueStrategy;
                    // 根据队列分配策略分配消息队列
                    List<MessageQueue> allocateResult = null;
                    try {
                        // 分配消息
                        allocateResult = strategy.allocate(
                            this.consumerGroup,
                            this.mQClientFactory.getClientId(),
                            mqAll,
                            cidAll);
                    } catch (Throwable e) {
                        log.error("AllocateMessageQueueStrategy.allocate Exception. allocateMessageQueueStrategyName={}", strategy.getName(),
                            e);
                        return;
                    }

                    Set<MessageQueue> allocateResultSet = new HashSet<MessageQueue>();
                    if (allocateResult != null) {
                        allocateResultSet.addAll(allocateResult);
                    }

                    boolean changed = this.updateProcessQueueTableInRebalance(topic, allocateResultSet, isOrder);
                    if (changed) {
                        log.info(
                            "rebalanced result changed. allocateMessageQueueStrategyName={}, group={}, topic={}, clientId={}, mqAllSize={}, cidAllSize={}, rebalanceResultSize={}, rebalanceResultSet={}",
                            strategy.getName(), consumerGroup, topic, this.mQClientFactory.getClientId(), mqSet.size(), cidAll.size(),
                            allocateResultSet.size(), allocateResultSet);
                        this.messageQueueChanged(topic, mqSet, allocateResultSet);
                    }
                }
                break;
            }
            default:
                break;
        }
    }
```



> org.apache.rocketmq.client.consumer.rebalance.AllocateMessageQueueAveragely

```java
    @Override
    public List<MessageQueue> allocate(String consumerGroup, String currentCID, List<MessageQueue> mqAll,
        List<String> cidAll) {
        if (currentCID == null || currentCID.length() < 1) {
            throw new IllegalArgumentException("currentCID is empty");
        }
        if (mqAll == null || mqAll.isEmpty()) {
            throw new IllegalArgumentException("mqAll is null or mqAll empty");
        }
        if (cidAll == null || cidAll.isEmpty()) {
            throw new IllegalArgumentException("cidAll is null or cidAll empty");
        }

        List<MessageQueue> result = new ArrayList<MessageQueue>();
        if (!cidAll.contains(currentCID)) {
            log.info("[BUG] ConsumerGroup: {} The consumerId: {} not in cidAll: {}",
                consumerGroup,
                currentCID,
                cidAll);
            return result;
        }

        // 第几个Consumer，这也是我们上面为什么要排序的重要原因之一。
        int index = cidAll.indexOf(currentCID);
        // 取模，多少消息队列无法平均分配
        int mod = mqAll.size() % cidAll.size();
        // 平均分配
        int averageSize =
            mqAll.size() <= cidAll.size() ? 1 : (mod > 0 && index < mod ? mqAll.size() / cidAll.size()
                + 1 : mqAll.size() / cidAll.size());
        // 有余数的情况下，[0, mod) 平分余数，即每consumer多分配一个节点；第index开始，跳过前mod余数。
        int startIndex = (mod > 0 && index < mod) ? index * averageSize : index * averageSize + mod;
        // 分配队列数量。
        // 之所以要Math.min()的原因是，mqAll.size() <= cidAll.size()，部分consumer分配不到消息队列。
        int range = Math.min(averageSize, mqAll.size() - startIndex);
        for (int i = 0; i < range; i++) {
            result.add(mqAll.get((startIndex + i) % mqAll.size()));
        }
        return result;
    }
```



#### 负载均衡时机

- Consumer宕机后，默认最多20秒，进行负载均衡

> org.apache.rocketmq.common.ServiceThread

```java
    public void start() {
        log.info("Try to start service thread:{} started:{} lastThread:{}", getServiceName(), started.get(), thread);
        if (!started.compareAndSet(false, true)) {
            return;
        }
        stopped = false;
        this.thread = new Thread(this, getServiceName());
        this.thread.setDaemon(isDaemon);
        this.thread.start();
    }
```



RebalanceService的父类的start()方法调用了Thread的start()方法，相当于调用了RebalanceService的run()方法，实现了负载均衡。



> org.apache.rocketmq.client.impl.consumer.RebalanceService

```java
    // 等待时间的间隔，毫秒，默认是20s
    private static long waitInterval =
        Long.parseLong(System.getProperty(
            "rocketmq.client.rebalance.waitInterval", "20000"));

    @Override
    public void run() {
        log.info(this.getServiceName() + " service started");

        while (!this.isStopped()) {
            // 等待20s，然后超时自动释放锁执行doRebalance
            this.waitForRunning(waitInterval);
            this.mqClientFactory.doRebalance();
        }

        log.info(this.getServiceName() + " service end");
    }
```



- 新Consumer上线后会立即唤醒沉睡的线程，进行负载均衡

> org.apache.rocketmq.client.impl.consumer.DefaultMQPushConsumerImpl

```java
public synchronized void start() throws MQClientException {
        // ...
        // 立刻负载均衡
        this.mQClientFactory.rebalanceImmediately();
    }
```



### 消费消息

#### 消费时Broker挂了

如果 Broker 没有设置主从集群，消费者会继续从挂掉的 Broker 上拉取，这会导致拉取失败，直到 NameServer 更新了 Broker 列表。如果有从节点，在 Broker 主节点恢复前，生产者是不能往从节点发送消息的，但是消费者可以去从节点拉取消息。

Broker 挂了以后，消费组会通过向 Name Server 拉取订阅关系来更新本地缓存的 Broker 列表，因为主节点已经不在列表中了，所以会从从节点列表中选择一个 Broker 进项消息拉取。

在主节点系统压力较大的时候，消费者也会去从节点拉取消息。



```java
// DefaultMessageStore
//maxOffsetPy:最大物理偏移量
//maxPhyOffsetPulling:这次消息拉取的最大偏移量
//diff:还没有被拉取的消息总长度
long diff = maxOffsetPy - maxPhyOffsetPulling;
//TOTAL_PHYSICAL_MEMORY_SIZE:系统总的物理内存大小
//getAccessMessageInMemoryMaxRatio 默认是 40
long memory = (long) (StoreUtil.TOTAL_PHYSICAL_MEMORY_SIZE
 * (this.messageStoreConfig.getAccessMessageInMemoryMaxRatio() / 100.0));
getResult.setSuggestPullingFromSlave(diff > memory);
```



当未处理的消息超出物理内存 40% 时就会去从节点拉取。**需要注意两点：**

1. 需要设置 slaveReadEnable 参数为 true，才能去从节点读取数据；
2. 需要配置 whichBrokerWhenConsumeSlowly 参数来决定从哪个从 brokerId 读取。参考下面这段代码：

```java
if (this.brokerController.getBrokerConfig().isSlaveReadEnable()) {
 // consume too slow ,redirect to another machine
 if (getMessageResult.isSuggestPullingFromSlave()) {
     //这里配置从哪个从节点拉取
  responseHeader.setSuggestWhichBrokerId(subscriptionGroupConfig.getWhichBrokerWhenConsumeSlowly());
 }
 //...
}
```



brokerId 默认是 0，也就是主节点，如果主节点挂了并且长期启动失败，这个参数也是需要改成可以长期拉取的一个从节点。



##### 重复消费

Broker 主节点挂了，如果成功从节点拉取消息时，对于**广播模式**，消息偏移量是保存在消费者本地的，只要消费者不挂，按照内存中的偏移量去从节点拉取就行了，不会有问题。



对于**集群模式**，消息偏移量保存在 Broker，路径如下：

```
/${rocketmq.client.localOffsetStoreDir}/.rocketmq_offsets/${clientId}/${groupName}/offsets.json
```



消费者消费完一批消息后，会向 Broker 发送请求更新 Broker 内存中保存的偏移量，内存中的偏移量会定时（每 5s 一次）更新到上面文件中。如果 Broker 主节点不挂，无论消费者从主节点还是从节点拉取消息，更新偏移量的请求都会发送到主节点，从节点会每隔 10s 从主节点同步偏移量，如下图：

![图片](../../Image/2022/09/220926-13.jpg)



```java
//BrokerController 类 handleSlaveSynchronize
if (role == BrokerRole.SLAVE) {
 slaveSyncFuture = this.scheduledExecutorService.scheduleAtFixedRate(new Runnable() {
  @Override
  public void run() {
   try {
    BrokerController.this.slaveSynchronize.syncAll();
   }
  }
 }, 1000 * 3, 1000 * 10, TimeUnit.MILLISECONDS);
}
```

**也就是说，如果主节点挂了，去从节点拉取消息，可能因为偏移量没有同步到主节点，从节点保存的偏移量不正确。不过只要消费者不宕机，就会根据消费者本地保存的偏移量去拉取，并不会拉取到重复消息。**



如果 Broker 主节点重启了，主节点并不能同步从节点的最新偏移量，消费者会用本地保存的偏移量去主节点拉取消息，主节点会更新本地的偏移量，同时从节点也会去主节点同步偏移量，所以并不会拉取到重复消息。如果消费者也挂了，消费者重启后 Broker 主节点的偏移量还没有被其他消费者更新过，那确实会拉取到重复消息。 



## 代码示例

### Java

#### Maven

```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.7.0</version>
</dependency>
```



#### 发送接收消息

##### Producer

发消息肯定要必备如下几个条件：

- 指定生产组名（不能用默认的，会报错）
- 配置namesrv地址（必须）
- 指定topic name（必须）
- 指定tag/key（可选）

验证消息是否发送成功：消息发送完后可以启动消费者进行消费，也可以去管控台上看消息是否存在。



###### send（同步）

```java
public class Producer {
    public static void main(String[] args) throws Exception {
        // 指定生产组名为my-producer
        DefaultMQProducer producer = new DefaultMQProducer("my-producer");
        // 配置namesrv地址
        producer.setNamesrvAddr("124.57.180.156:9876");
        // 启动Producer
        producer.start();
        // 创建消息对象，topic为：myTopic001，消息内容为：hello world
        Message msg = new Message("myTopic001", "hello world".getBytes());
        // 发送消息到mq，同步的
        SendResult result = producer.send(msg);
        System.out.println("发送消息成功！result is : " + result);
        // 关闭Producer
        producer.shutdown();
        System.out.println("生产者 shutdown！");
    }
}
```



###### send（批量）

```java
public class ProducerMultiMsg {
    public static void main(String[] args) throws Exception {
        // 指定生产组名为my-producer
        DefaultMQProducer producer = new DefaultMQProducer("my-producer");
        // 配置namesrv地址
        producer.setNamesrvAddr("124.57.180.156:9876");
        // 启动Producer
        producer.start();

        String topic = "myTopic001";
        // 创建消息对象，topic为：myTopic001，消息内容为：hello world1/2/3
        Message msg1 = new Message(topic, "hello world1".getBytes());
        Message msg2 = new Message(topic, "hello world2".getBytes());
        Message msg3 = new Message(topic, "hello world3".getBytes());
        // 创建消息对象的集合，用于批量发送
        List<Message> msgs = new ArrayList<>();
        msgs.add(msg1);
        msgs.add(msg2);
        msgs.add(msg3);
        // 批量发送的api的也是send()，只是他的重载方法支持List<Message>，同样是同步发送。
        SendResult result = producer.send(msgs);
        System.out.println("发送消息成功！result is : " + result);
        // 关闭Producer
        producer.shutdown();
        System.out.println("生产者 shutdown！");
    }
}
```



###### sendCallBack（异步）

```java
public class ProducerASync {
    public static void main(String[] args) throws Exception {
       // 指定生产组名为my-producer
        DefaultMQProducer producer = new DefaultMQProducer("my-producer");
        // 配置namesrv地址
        producer.setNamesrvAddr("124.57.180.156:9876");
        // 启动Producer
        producer.start();

        // 创建消息对象，topic为：myTopic001，消息内容为：hello world async
        Message msg = new Message("myTopic001", "hello world async".getBytes());
        // 进行异步发送，通过SendCallback接口来得知发送的结果
        producer.send(msg, new SendCallback() {
            // 发送成功的回调接口
            @Override
            public void onSuccess(SendResult sendResult) {
                System.out.println("发送消息成功！result is : " + sendResult);
            }
            // 发送失败的回调接口
            @Override
            public void onException(Throwable throwable) {
                throwable.printStackTrace();
                System.out.println("发送消息失败！result is : " + throwable.getMessage());
            }
        });
        // 此时直接shutdown可能会抛出RemotingConnectException异常，应为异步消息可能还没有发送到Broker。
        producer.shutdown();
        System.out.println("生产者 shutdown！");
    }
}
```



###### sendOneway

```java
public class ProducerOneWay {
    public static void main(String[] args) throws Exception {
        // 指定生产组名为my-producer
        DefaultMQProducer producer = new DefaultMQProducer("my-producer");
        // 配置namesrv地址
        producer.setNamesrvAddr("124.57.180.156:9876");
        // 启动Producer
        producer.start();

        // 创建消息对象，topic为：myTopic001，消息内容为：hello world oneway
        Message msg = new Message("myTopic001", "hello world oneway".getBytes());
        // 效率最高，因为oneway不关心是否发送成功，我就投递一下我就不管了。所以返回是void
        producer.sendOneway(msg);
        System.out.println("投递消息成功！注意这里是投递成功，而不是发送消息成功哦！因为我sendOneway也不知道到底成没成功，我没返回值的。");
        producer.shutdown();
        System.out.println("生产者 shutdown！");
    }
}
```



###### 总结

几种发送消息的方式效率如下：

**sendOneway > sendCallBack > send批量 > send单条**

sendOneway不求结果，就负责投递，不管你失败还是成功，相当于中转站，来了就扔出去，不进行任何其他处理。所以最快。

而sendCallBack是异步发送肯定比同步的效率高。

send批量和send单条的效率也是分情况的，如果只有1条msg要发，那还搞毛批量，直接send单条完事。



##### Consumer

**每个consumer只能关注一个topic。**拉取消息肯定要必备如下几个条件：

- 指定消费组名（不能用默认的，会报错）
- 配置namesrv地址（必须）
- 指定topic name（必须）
- 指定tag/key（可选）



###### 集群模式

默认拉去模式即集群模式（Clustering），比如启动五个Consumer，Producer生产一条消息后，Broker会选择五个Consumer中的其中一个进行消费这条消息，所以属于点对点消费模式。

```java
public class Consumer {
    public static void main(String[] args) throws Exception {
        // 指定消费组名为my-consumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("my-consumer");
        // 配置namesrv地址
        consumer.setNamesrvAddr("124.57.180.156:9876");
        // 订阅topic：myTopic001 下的全部消息（因为是*，*指定的是tag标签，代表全部消息，不进行任何过滤）
        consumer.subscribe("myTopic001", "*");
        // 注册监听器，进行消息消息。
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                for (MessageExt msg : msgs) {
                    String str = new String(msg.getBody());
                    // 输出消息内容
                    System.out.println(str);
                }
                // 默认情况下，这条消息只会被一个consumer消费，这叫点对点消费模式。也就是集群模式。
                // ack确认
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        // 启动消费者
        consumer.start();
        System.out.println("Consumer start");
    }
}
```



###### 广播模式

广播模式（BroadCasting）指比如启动五个Consumer，Producer生产一条消息后，Broker会把这条消息广播到五个Consumer中，这五个Consumer分别消费一次，每个都消费一次。



```java
// 代码里只需要添加如下这句话即可：
consumer.setMessageModel(MessageModel.BROADCASTING); 
```



###### 总结

- 集群默认是默认的，广播模式是需要手动配置。
- 一条消息：集群模式下的多个Consumer只会有一个Consumer消费。广播模式下的每一个Consumer都会消费这条消息。
- 广播模式下，发送一条消息后，会被当前被广播的所有Consumer消费，但是后面新加入的Consumer不会消费这条消息。



##### Tag&Key

发送/消费消息的时候可以指定tag/key来进行过滤消息，支持通配符。*代表消费此topic下的全部消息，不进行过滤。

```java
public class ProducerTagsKeys {
    public static void main(String[] args) throws Exception {
        // 指定生产组名为my-producer
        DefaultMQProducer producer = new DefaultMQProducer("my-producer");
        // 配置namesrv地址
        producer.setNamesrvAddr("124.57.180.156:9876");
        // 启动Producer
        producer.start();
        // 创建消息对象，topic为：myTopic001，
        // 消息内容为：hello world，且tags为：test-tags，keys为test-keys
        Message msg = new Message("myTopic001", "test-tags", "test-keys", "hello world".getBytes());
        // 发送消息到mq，同步的
        SendResult result = producer.send(msg);
        System.out.println("发送消息成功！result is : " + result);
        // 关闭Producer
        producer.shutdown();
        System.out.println("生产者 shutdown！");
    }
}
```



消费的时候如果指定*那就是此topic下的全部消息，可以指定前缀通配符，比如：

```java
// 这样就只会消费myTopic001下的tag为test-*开头的消息。
consumer.subscribe("myTopic001", "test-*");

// 代表订阅Topic为myTopic001下的tag为TagA或TagB的所有消息
consumer.subscribe("myTopic001", "TagA||TagB");
```



还支持SQL表达式过滤，不是很常用。



##### 常见异常

###### 发送超时

```
o.a.r.r.e.RemotingTooMuchRequestException: sendDefaultImpl call timeout
```



解决方式：

1. 如果你是云服务器，首先检查安全组是否允许9876这个端口访问，是否开启了防火墙，如果开启了的话是否将9876映射了出去。
2. 修改配置文件`broker.conf`，加上：

```
brokerIP1=阿里云服务器公网IP
```

​		

启动namesrv和broker的时候加上本机IP（我用的是阿里云服务器，这里是我的公网IP）：

```
./bin/mqnamesrv -n IP:9876
./bin/mqbroker -n IP:9876 -c conf/broker.conf
```



###### 没有Topic路由信息

提示没有这个topic，如下

```
o.a.r.c.e.MQClientException: No route info of this topic: myTopic001
```



**解决方案**

1. 手动创建该Topic；
2. 启动Broker时指定参数让Broker自动创建。

```
./bin/mqbroker -n IP:9876 -c conf/broker.conf autoCreateTopicEnable=true
```



###### 批量发送消息Topic不一致

批量发送的topic必须是同一个，如果message对象指定不同的topic，那么批量发送的时候会报错：

```bash
o.a.r.c.e.MQClientException: Failed to initiate the MessageBatch
For more information, please visit the url, http://rocketmq.apache.org/docs/faq/
```



#### 事务消息

```java
public class ProducerTransaction2 {
    public static void main(String[] args) throws Exception {
        TransactionMQProducer producer = new TransactionMQProducer("my-transaction-producer");
        producer.setNamesrvAddr("124.57.180.156:9876");

        // 回调
        producer.setTransactionListener(new TransactionListener() {
            @Override
            public LocalTransactionState executeLocalTransaction(Message message, Object arg) {
                LocalTransactionState state = null;
                //msg-4返回COMMIT_MESSAGE
                if(message.getKeys().equals("msg-1")){
                    state = LocalTransactionState.COMMIT_MESSAGE;
                }
                //msg-5返回ROLLBACK_MESSAGE
                else if(message.getKeys().equals("msg-2")){
                    state = LocalTransactionState.ROLLBACK_MESSAGE;
                }else{
                    //这里返回unknown的目的是模拟执行本地事务突然宕机的情况（或者本地执行成功发送确认消息失败的场景）
                    state = LocalTransactionState.UNKNOW;
                }
                System.out.println(message.getKeys() + ",state:" + state);
                return state;
            }

            /**
             * 事务消息的回查方法
             */
            @Override
            public LocalTransactionState checkLocalTransaction(MessageExt messageExt) {
                if (null != messageExt.getKeys()) {
                    switch (messageExt.getKeys()) {
                        case "msg-3":
                            System.out.println("msg-3 unknow");
                            return LocalTransactionState.UNKNOW;
                        case "msg-4":
                            System.out.println("msg-4 COMMIT_MESSAGE");
                            return LocalTransactionState.COMMIT_MESSAGE;
                        case "msg-5":
                            //查询到本地事务执行失败，需要回滚消息。
                            System.out.println("msg-5 ROLLBACK_MESSAGE");
                            return LocalTransactionState.ROLLBACK_MESSAGE;
                    }
                }
                return LocalTransactionState.COMMIT_MESSAGE;
            }
        });

        producer.start();

        //模拟发送5条消息
        for (int i = 1; i < 6; i++) {
            try {
                Message msg = new Message("transactionTopic", null, "msg-" + i, ("测试，这是事务消息！ " + i).getBytes());
                producer.sendMessageInTransaction(msg, null);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```



执行结果如下：

```
msg-1,state:COMMIT_MESSAGE
msg-2,state:ROLLBACK_MESSAGE
msg-3,state:UNKNOW
msg-4,state:UNKNOW
msg-5,state:UNKNOW

msg-3 unknow
msg-3 unknow
msg-5 ROLLBACK_MESSAGE
msg-4 COMMIT_MESSAGE

msg-3 unknow
msg-3 unknow
msg-3 unknow
msg-3 unknow
```



- 只有msg-1和msg-4发送成功了。msg-4在msg-1前面是因为msg-1先成功的，msg-4是回查才成功的。按时间倒序来的。
- 先来输出五个结果，对应五条消息

- 然后进入了回查，msg-3还是unknow，msg-5回滚了，msg-4提交了事务。所以这时候msg-4在管控台里能看到了。
- 过了一段时间再次回查msg-3，发现还是unknow，所以一直回查。



> 回查的时间间隔和次数都是可配的，默认是回查15次还失败的话就会把这个消息丢掉了。



#### 顺序消费

##### 消息发送到同一个queue

**Producer**

```java
public class Producer5 {
    public static void main(String[] args)throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("my-order-producer");
        producer.setNamesrvAddr("124.57.180.156:9876");     
        producer.start();
        for (int i = 0; i < 5; i++) {
            Message message = new Message("orderTopic", ("hello!" + i).getBytes());
            producer.send(
                    // 要发的那条消息
                    message,
                    // queue 选择器 ，向 topic中的哪个queue去写消息
                    new MessageQueueSelector() {
                        // 手动 选择一个queue
                        @Override
                        public MessageQueue select(
                                // 当前topic 里面包含的所有queue
                                List<MessageQueue> mqs, 
                                // 具体要发的那条消息
                                Message msg,
                                // 对应到 send（） 里的 args，也就是2000前面的那个0
                                // 实际业务中可以把0换成实际业务系统的主键，比如订单号啥的，然后这里做hash进行选择queue等。能做的事情很多，我这里做演示就用第一个queue，所以不用arg。
                                Object arg) {
                            // 向固定的一个queue里写消息，比如这里就是向第一个queue里写消息
                            MessageQueue queue = mqs.get(0);
                            // 选好的queue
                            return queue;
                        }
                    },
                    // 自定义参数：0
                    // 2000代表2000毫秒超时时间
                    0, 2000);
        }
    }
}
```



**Consumer**

```java
public class ConsumerOrder {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("my-consumer");
        consumer.setNamesrvAddr("124.57.180.156:9876");
        consumer.subscribe("orderTopic", "*");
        consumer.registerMessageListener(new MessageListenerOrderly() {
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
                for (MessageExt msg : msgs) {
                    System.out.println(new String(msg.getBody()) + " Thread:" + Thread.currentThread().getName() + " queueid:" + msg.getQueueId());
                }
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });

        consumer.start();
        System.out.println("Consumer start...");
    }
}
```



执行结果：

```
Consumer start...
hello!0 Thread:ConsumeMessageThread_1 queueid:0
hello!1 Thread:ConsumeMessageThread_1 queueid:0
hello!2 Thread:ConsumeMessageThread_1 queueid:0
hello!3 Thread:ConsumeMessageThread_1 queueid:0
hello!4 Thread:ConsumeMessageThread_1 queueid:0
```



##### 单线程消费

创建消费者时设置消费线程为单线程。

```java
// 最大线程数1
consumer.setConsumeThreadMax(1);
// 最小线程数
consumer.setConsumeThreadMin(1);
```



## 其它

### 分布式消息组件的条件

- 需要考虑能快速扩容、天然支持集群
- 持久化的姿势
- 高可用性
- 数据0丢失的考虑
- 服务端部署简单、client端使用简单





# 概念和特性

## 概念[(Concept)](https://github.com/apache/rocketmq/blob/master/docs/cn/concept.md)

- Message

消息载体。Message发送或者消费的时候必须指定Topic。Message有一个可选的Tag项用于过滤消息，还可以添加额外的键值对。

**Message**（消息）就是要传输的信息。

一条消息必须有一个主题（Topic），主题可以看做是你的信件要邮寄的地址。

一条消息也可以拥有一个可选的标签（Tag）和额外的键值对，它们可以用于设置一个业务 Key 并在 Broker 上查找此消息以便在开发期间查找问题。



- Topic

消息的逻辑分类，发消息之前必须要指定一个topic才能发，就是将这条消息发送到这个topic上。消费消息的时候指定这个topic进行消费。就是逻辑分类。

**Topic**（主题）可以看做消息的规类，它是消息的第一级类型。比如一个电商系统可以分为：交易消息、物流消息等，一条消息必须有一个 Topic 。

**Topic** 与生产者和消费者的关系非常松散，一个 Topic 可以有0个、1个、多个生产者向其发送消息，一个生产者也可以同时向不同的 Topic 发送消息。

一个 Topic 也可以被 0个、1个、多个消费者订阅。



- Queue

1个Topic会被分为N个Queue，数量是可配置的。message本身其实是存储到queue上的，消费者消费的也是queue上的消息。多说一嘴，比如1个topic4个queue，有5个Consumer都在消费这个topic，那么会有一个consumer浪费掉了，因为负载均衡策略，每个consumer消费1个queue，5>4，溢出1个，这个会不工作。

在**Kafka**中叫Partition，每个Queue内部是有序的，在**RocketMQ**中分为读和写两种队列，一般来说读写队列数量一致，如果不一致就会出现很多问题。

**Message Queue**（消息队列），主题被划分为一个或多个子主题，即消息队列。

一个 Topic 下可以设置多个消息队列，发送消息时执行该消息的 Topic ，RocketMQ 会轮询该 Topic 下的所有队列将消息发出去。

消息的物理管理单位。一个Topic下可以有多个Queue，Queue的引入使得消息的存储可以分布式集群化，具有了水平扩展能力。



- Tag

Tag 是 Topic 的进一步细分，顾名思义，标签。每个发送的时候消息都能打tag，消费的时候可以根据tag进行过滤，选择性消费。

**Tag**（标签）可以看作子主题，它是消息的第二级类型，用于为用户提供额外的灵活性。使用标签，同一业务模块不同目的的消息就可以用相同 Topic 而不同的 **Tag** 来标识。比如交易消息又可以分为：交易创建消息、交易完成消息等，一条消息可以没有 **Tag** 。

标签有助于保持您的代码干净和连贯，并且还可以为 **RocketMQ** 提供的查询系统提供帮助。



- Message  Model

消息模型：集群（Clustering）和广播（Broadcasting）。

默认情况下就是集群消费，该模式下一个消费者集群共同消费一个主题的多个队列，一个队列只会被一个消费者消费，如果某个消费者挂掉，分组内其它消费者会接替挂掉的消费者继续消费。

而广播消费消息会发给消费者组中的每一个消费者进行消费。



- Message Order

消息顺序：顺序（Orderly）和并发（Concurrently）。

顺序消费表示消息消费的顺序同生产者为每个消息队列发送的顺序一致，所以如果正在处理全局顺序是强制性的场景，需要确保使用的主题只有一个消息队列。

并行消费不再保证消息顺序，消费的最大并行数量受每个消费者客户端指定的线程池限制。



- Producer Group

消息生产者组。



- Consumer Group

消息消费者组。



- Offset

在**RocketMQ** 中，所有消息队列都是持久化，长度无限的数据结构，所谓长度无限是指队列中的每个存储单元都是定长，访问其中的存储单元使用Offset 来访问，Offset 为 java long 类型，64 位，理论上在 100年内不会溢出，所以认为是长度无限。

也可以认为 Message Queue 是一个长度无限的数组，**Offset** 就是下标。



### 消息模型

RocketMQ消息模型（Message Model）主要由 Producer、Broker、Consumer 三部分组成，其中Producer 负责生产消息，Consumer 负责消费消息，Broker 负责存储消息。Broker 在实际部署过程中对应一台服务器，每个 Broker 可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的 Broker。Message Queue 用于存储消息的物理地址，每个Topic中的消息地址存储于多个 Message Queue 中。ConsumerGroup 由多个Consumer 实例构成。



### 消息生产者

消息生产者（Producer）负责生产消息，一般由业务系统负责生产消息。一个消息生产者会把业务应用系统里产生的消息发送到broker服务器。RocketMQ提供多种发送方式，同步发送、异步发送、顺序发送、单向发送。同步和异步方式均需要Broker返回确认信息，单向发送不需要。



### 消息消费者

消息消费者（Consumer）负责消费消息，一般是后台系统负责异步消费。一个消息消费者会从Broker服务器拉取消息、并将其提供给应用程序。从用户应用的角度而言提供了两种消费形式：拉取式消费、推动式消费。



### 主题

主题（Topic）表示一类消息的集合，每个主题包含若干条消息，每条消息只能属于一个主题，是RocketMQ进行消息订阅的基本单位。



### 代理服务器

代理服务器（Broker Server）是消息中转角色，负责存储消息、转发消息。代理服务器在RocketMQ系统中负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。代理服务器也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等。



### 名字服务

名称服务（Name Server）充当路由消息的提供者。生产者或消费者能够通过名字服务查找各主题相应的Broker IP列表。多个Namesrv实例组成集群，但相互独立，没有信息交换。



### 拉取式消费

拉取式消费（Pull Consumer）是Consumer消费的一种类型，应用通常主动调用Consumer的拉消息方法从Broker服务器拉消息、主动权由应用控制。一旦获取了批量消息，应用就会启动消费过程。



### 推动式消费

推动式消费（Push Consumer）是Consumer消费的一种类型，该模式下Broker收到数据后会主动推送给消费端，该消费模式一般实时性较高。



### 生产者组

生产者组（Producer Group）指同一类Producer的集合，这类Producer发送同一类消息且发送逻辑一致。如果发送的是事务消息且原始生产者在发送之后崩溃，则Broker服务器会联系同一生产者组的其他生产者实例以提交或回溯消费。



### 消费者组

消费者组（Consumer Group）指同一类Consumer的集合，这类Consumer通常消费同一类消息且消费逻辑一致。消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易。要注意的是，消费者组的消费者实例必须订阅完全相同的Topic。RocketMQ 支持两种消息模式：集群消费（Clustering）和广播消费（Broadcasting）。



### 集群消费

集群消费（Clustering）模式下，相同Consumer Group的每个Consumer实例平均分摊消息。



### 广播消费

广播消费（Broadcasting）模式下，相同Consumer Group的每个Consumer实例都接收全量的消息。



### 普通顺序消息

普通顺序消费（Normal Ordered Message）模式下，消费者通过同一个消息队列（ Topic 分区，称作 Message Queue） 收到的消息是有顺序的，不同消息队列收到的消息则可能是无顺序的。



### 严格顺序消息

严格顺序消息（Strictly Ordered Message）模式下，消费者收到的所有消息均是有顺序的。



### 消息

消息系统所传输信息的物理载体，生产和消费数据的最小单位，每条消息（Message）必须属于一个主题。RocketMQ中每个消息拥有唯一的Message ID，且可以携带具有业务标识的Key。系统提供了通过Message ID和Key查询消息的功能。



### 标签

为消息设置的标志，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签（Tag）。标签能够有效地保持代码的清晰度和连贯性，并优化RocketMQ提供的查询系统。消费者可以根据Tag实现对不同子主题的不同消费逻辑，实现更好的扩展性。



## 特性[(Features)](https://github.com/apache/rocketmq/blob/master/docs/cn/features.md)

介绍RocketMQ实现的功能特性。

### 订阅与发布

消息的发布是指某个生产者向某个topic发送消息；消息的订阅是指某个消费者关注了某个topic中带有某些tag的消息，进而从该topic消费数据。



### 消息顺序

消息有序指的是一类消息消费时，能按照发送的顺序来消费。例如：一个订单产生了三条消息分别是订单创建、订单付款、订单完成。消费时要按照这个顺序消费才能有意义，但是同时订单之间是可以并行消费的。RocketMQ可以严格的保证消息有序。

顺序消息分为全局顺序消息与分区顺序消息，全局顺序是指某个Topic下的所有消息都要保证顺序；部分顺序消息只要保证每一组消息被顺序消费即可。

- 全局顺序 对于指定的一个 Topic，所有消息按照严格的先入先出（FIFO）的顺序进行发布和消费。 适用场景：性能要求不高，所有的消息严格按照 FIFO 原则进行消息发布和消费的场景
- 分区顺序 对于指定的一个 Topic，所有消息根据 sharding key 进行区块分区。 同一个分区内的消息按照严格的 FIFO 顺序进行发布和消费。 Sharding key 是顺序消息中用来区分不同分区的关键字段，和普通消息的 Key 是完全不同的概念。 适用场景：性能要求高，以 sharding key 作为分区字段，在同一个区块中严格的按照 FIFO 原则进行消息发布和消费的场景。



### 消息过滤

RocketMQ的消费者可以根据Tag进行消息过滤，也支持自定义属性过滤。消息过滤目前是在Broker端实现的，优点是减少了对于Consumer无用消息的网络传输，缺点是增加了Broker的负担、而且实现相对复杂。



### 消息可靠性

RocketMQ支持消息的高可靠，影响消息可靠性的几种情况：

1. Broker非正常关闭
2. Broker异常Crash
3. OS Crash
4. 机器掉电，但是能立即恢复供电情况
5. 机器无法开机（可能是cpu、主板、内存等关键设备损坏）
6. 磁盘设备损坏

1)、2)、3)、4) 四种情况都属于硬件资源可立即恢复情况，RocketMQ在这四种情况下能保证消息不丢，或者丢失少量数据（依赖刷盘方式是同步还是异步）。

5)、6)属于单点故障，且无法恢复，一旦发生，在此单点上的消息全部丢失。RocketMQ在这两种情况下，通过异步复制，可保证99%的消息不丢，但是仍然会有极少量的消息可能丢失。通过同步双写技术可以完全避免单点，同步双写势必会影响性能，适合对消息可靠性要求极高的场合，例如与Money相关的应用。注：RocketMQ从3.0版本开始支持同步双写。



### 至少一次

至少一次(At least Once)指每个消息必须投递一次。Consumer先Pull消息到本地，消费完成后，才向服务器返回ack，如果没有消费一定不会ack消息，所以RocketMQ可以很好的支持此特性。



### 回溯消费

回溯消费是指Consumer已经消费成功的消息，由于业务上需求需要重新消费，要支持此功能，Broker在向Consumer投递成功消息后，消息仍然需要保留。并且重新消费一般是按照时间维度，例如由于Consumer系统故障，恢复后需要重新消费1小时前的数据，那么Broker要提供一种机制，可以按照时间维度来回退消费进度。RocketMQ支持按照时间回溯消费，时间维度精确到毫秒。



### 事务消息

RocketMQ事务消息（Transactional Message）是指应用本地事务和发送消息操作可以被定义到全局事务中，要么同时成功，要么同时失败。RocketMQ的事务消息提供类似 X/Open XA 的分布事务功能，通过事务消息能达到分布式事务的最终一致。



### 定时消息

定时消息（延迟队列）是指消息发送到broker后，不会立即被消费，等待特定时间投递给真正的topic。 broker有配置项messageDelayLevel，默认值为“1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h”，18个level。可以配置自定义messageDelayLevel。注意，messageDelayLevel是broker的属性，不属于某个topic。发消息时，设置delayLevel等级即可：msg.setDelayLevel(level)。level有以下三种情况：

- level == 0，消息为非延迟消息
- 1<=level<=maxLevel，消息延迟特定时间，例如level==1，延迟1s
- level > maxLevel，则level== maxLevel，例如level==20，延迟2h

定时消息会暂存在名为SCHEDULE_TOPIC_XXXX的topic中，并根据delayTimeLevel存入特定的queue，queueId = delayTimeLevel – 1，即一个queue只存相同延迟的消息，保证具有相同发送延迟的消息能够顺序消费。broker会调度地消费SCHEDULE_TOPIC_XXXX，将消息写入真实的topic。

需要注意的是，定时消息会在第一次写入和调度写入真实topic时都会计数，因此发送数量、tps都会变高。



### 消息重试

Consumer消费消息失败后，要提供一种重试机制，令消息再消费一次。Consumer消费消息失败通常可以认为有以下几种情况：

- 由于消息本身的原因，例如反序列化失败，消息数据本身无法处理（例如话费充值，当前消息的手机号被注销，无法充值）等。这种错误通常需要跳过这条消息，再消费其它消息，而这条失败的消息即使立刻重试消费，99%也不成功，所以最好提供一种定时重试机制，即过10秒后再重试。
- 由于依赖的下游应用服务不可用，例如db连接不可用，外系统网络不可达等。遇到这种错误，即使跳过当前失败的消息，消费其他消息同样也会报错。这种情况建议应用sleep 30s，再消费下一条消息，这样可以减轻Broker重试消息的压力。

RocketMQ会为每个消费组都设置一个Topic名称为“%RETRY%+consumerGroup”的重试队列（这里需要注意的是，这个Topic的重试队列是针对消费组，而不是针对每个Topic设置的），用于暂时保存因为各种异常而导致Consumer端无法消费的消息。考虑到异常恢复起来需要一些时间，会为重试队列设置多个重试级别，每个重试级别都有与之对应的重新投递延时，重试次数越多投递延时就越大。RocketMQ对于重试消息的处理是先保存至Topic名称为“SCHEDULE_TOPIC_XXXX”的延迟队列中，后台定时任务按照对应的时间进行Delay后重新保存至“%RETRY%+consumerGroup”的重试队列中。



### 消息重投

生产者在发送消息时，同步消息失败会重投，异步消息有重试，oneway没有任何保证。消息重投保证消息尽可能发送成功、不丢失，但可能会造成消息重复，消息重复在RocketMQ中是无法避免的问题。消息重复在一般情况下不会发生，当出现消息量大、网络抖动，消息重复就会是大概率事件。另外，生产者主动重发、consumer负载变化也会导致重复消息。如下方法可以设置消息重试策略：

- retryTimesWhenSendFailed:同步发送失败重投次数，默认为2，因此生产者会最多尝试发送retryTimesWhenSendFailed + 1次。不会选择上次失败的broker，尝试向其他broker发送，最大程度保证消息不丢。超过重投次数，抛出异常，由客户端保证消息不丢。当出现RemotingException、MQClientException和部分MQBrokerException时会重投。
- retryTimesWhenSendAsyncFailed:异步发送失败重试次数，异步重试不会选择其他broker，仅在同一个broker上做重试，不保证消息不丢。
- retryAnotherBrokerWhenNotStoreOK:消息刷盘（主或备）超时或slave不可用（返回状态非SEND_OK），是否尝试发送到其他broker，默认false。十分重要消息可以开启。



### 流量控制

生产者流控，因为broker处理能力达到瓶颈；消费者流控，因为消费能力达到瓶颈。

生产者流控：

- commitLog文件被锁时间超过osPageCacheBusyTimeOutMills时，参数默认为1000ms，返回流控。
- 如果开启transientStorePoolEnable == true，且broker为异步刷盘的主机，且transientStorePool中资源不足，拒绝当前send请求，返回流控。
- broker每隔10ms检查send请求队列头部请求的等待时间，如果超过waitTimeMillsInSendQueue，默认200ms，拒绝当前send请求，返回流控。
- broker通过拒绝send 请求方式实现流量控制。

注意，生产者流控，不会尝试消息重投。

消费者流控：

- 消费者本地缓存消息数超过pullThresholdForQueue时，默认1000。
- 消费者本地缓存消息大小超过pullThresholdSizeForQueue时，默认100MB。
- 消费者本地缓存消息跨度超过consumeConcurrentlyMaxSpan时，默认2000。

消费者流控的结果是降低拉取频率。



### 死信队列

死信队列用于处理无法被正常消费的消息。当一条消息初次消费失败，消息队列会自动进行消息重试；达到最大重试次数后，若消费依然失败，则表明消费者在正常情况下无法正确地消费该消息，此时，消息队列 不会立刻将消息丢弃，而是将其发送到该消费者对应的特殊队列中。

RocketMQ将这种正常情况下无法被消费的消息称为死信消息（Dead-Letter Message），将存储死信消息的特殊队列称为死信队列（Dead-Letter Queue）。在RocketMQ中，可以通过使用console控制台对死信队列中的消息进行重发来使得消费者实例再次进行消费。



# 架构设计

## 架构[(Architecture)](https://github.com/apache/rocketmq/blob/master/docs/cn/architecture.md)

介绍RocketMQ部署架构和技术架构。

## 设计[(Design)](https://github.com/apache/rocketmq/blob/master/docs/cn/design.md)

介绍RocketMQ关键机制的设计原理，主要包括消息存储、通信机制、消息过滤、负载均衡、事务消息等。

# 样例

## 样例 [(Example)](https://github.com/apache/rocketmq/blob/master/docs/cn/RocketMQ_Example.md) 

介绍RocketMQ的常见用法，包括基本样例、顺序消息样例、延时消息样例、批量消息样例、过滤消息样例、事务消息样例等。

# 最佳实践

## 最佳实践 [(Best Practice)](https://github.com/apache/rocketmq/blob/master/docs/cn/best_practice.md)

介绍RocketMQ的最佳实践，包括生产者、消费者、Broker以及NameServer的最佳实践，客户端的配置方式以及JVM和linux的最佳参数配置。

## 消息轨迹指南 [(Message Trace)](https://github.com/apache/rocketmq/blob/master/docs/cn/msg_trace/user_guide.md)

介绍RocketMQ消息轨迹的使用方法。

## 权限管理[(Auth Management)](https://github.com/apache/rocketmq/blob/master/docs/cn/acl/user_guide.md)

介绍如何快速部署和使用支持权限控制特性的RocketMQ集群。

## Dledger快速搭建[(Quick Start)](https://github.com/apache/rocketmq/blob/master/docs/cn/dledger/quick_start.md)

介绍Dledger的快速搭建方法。

## 集群部署[(Cluster Deployment)](https://github.com/apache/rocketmq/blob/master/docs/cn/dledger/deploy_guide.md)

介绍Dledger的集群部署方式。



# 运维管理

## 安装RocketMQ

### Docker

 ```shell
# 拉取RocketMQ镜像
$ docker pull rocketmqinc/rocketmq

# 运行RocketMQ的NameServer
$ docker run -d -p 9876:9876 
-v /opt/rocket/data/namesrv/logs:/root/logs 
-v /opt/rocket/data/namesrv/store:/root/store
--name rmqnamesrv 
-e "MAX_POSSIBLE_HEAP=100000000" rocketmqinc/rocketmq sh mqnamesrv

# 使用相同的镜像运行RocketMQ的broker
$ docker run -d -p 10911:10911 -p  10909:10909   
-v   /opt/rocket/data/broker/logs:/root/logs   
-v   /opt/rocket/rocketmq/data/broker/store:/root/store   
-v   /opt/rocket/conf/broker.conf:/opt/rocketmq/conf/broker.conf --name  rmqbroker --link rmqnamesrv:namesrv 
-e "NAMESRV_ADDR=namesrv:9876"   
-e "MAX_POSSIBLE_HEAP=200000000" rocketmqinc/rocketmq sh  mqbroker   
-c /opt/rocketmq/conf/broker.conf  

# 拉取RocketMQ控制台镜像
$ docker pull pangliang/rocketmq-console-ng
$ docker run -e "JAVA_OPTS=-Drocketmq.namesrv.addr=47.101.48.22:9876 
-Dcom.rocketmq.sendMessageWithVIPChannel=false"
-p 19876:19876 
-t pangliang/rocketmq-console-ng
 ```



## 集群部署[(Operation)](https://github.com/apache/rocketmq/blob/master/docs/cn/operation.md)

介绍单Master模式、多Master模式、多Master多slave模式等RocketMQ集群各种形式的部署方法以及运维工具mqadmin的使用方式。

# API Reference（待补充）

[DefaultMQProducer API Reference](https://github.com/apache/rocketmq/blob/master/docs/cn/client/java/API_Reference_DefaultMQProducer.md)