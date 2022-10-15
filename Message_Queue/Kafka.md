# 分区状态

Kafka内部分区的运转机制具体实现为PartitionStateMachine，从这个类的注释上来看可以得知Kafka分区的状态共有四个，它们分别是：

- NonExistentPartition 表示分区不存在，通常是该分区从未创建过或者创建后被删除。

- NewPartition 分区已创建，即分配完成了副本，但还未进行分区Leader选举，即还不存在Leader分区与ISR集合，前一个有效状态为NonExistentPartition。

- OnlinePartition 分区处于在线时的状态，表示已经完成了分区选举，成功选举出Leader，此时可以进行消息发送与消息消费，前一个有效状态为NewPartition/OfflinePartition。

- OfflinePartition

	分区处于离线时状态，表示选举出来的Leader失效了，例如Leader所在的Broker宕机，前一个有效状态为NewPartition/OnlinePartition。

关于分区的状态变如下所示：

![图片](../../Image/2022/10/221010-1.png)



## 总结

- 分区的状态主要包括NonExistentPartition、NewPartition、OnlinePartition、OfflinePartition四个状态，只有分区状态为OnlinePartition才能对外提供读与写。
- Kafka启动时，在选举好集群的控制器(Kafka Controller)后会启动分区状态机(PartitionStateMachine),Kafka会根据/brokers/topics/{topicName}/partitions/{partition_no}/state中的信息，驱动分区状态向OnlineParttion转换。
- 当新创建主题时，Kafka会根据当前集群的负载情况，主题需要创建的分区数量、副本数量，机架信息等，进行负载均衡，生成分区的意向leader，已经分区副本的分布情况，写入到/brokers/topics/{topicName}节点上，此时会触发PartitionModifications，从而触发分区创建流程，即从NewPartition向OnlineParttion转换。



