# 目录

[TOC]

# 前言

# 架构理念

## 领域驱动设计(DDD)



## 职责分离模式(CQRS)

CQRS 全称是 Command Query Rsponsibility Segregation，将应用程序分为两部分：命令端(Command) 和查询端 (Query)。命令端处理程序创建，更新和删除请求，并在数据更改时发出事件。查询端通过执行查询来处理查询，并且通过订阅数据更改时发出的事件流而保持最新。CQRS使用分离的接口将数据查询操作(Queries) 和数据修改操作(Commands) 分离开来，这也意味着在查询和更新过程中使用的数据模型也是不一样的。这样读和写逻辑就隔离开来了。

CQRS 里面的一些概念：

- Command (命令): 不返回任何结果，被校验成功后但会改变对象的状态。
- Query (查询): 有返回结果，但是不会改变对象的状态。
- Aggregate (聚合): 保存状态, 处理command，和改变状态。
- Event Store: 存储Events。





# 微服务组件

### 系统监控

Monitoring（监控）举例来说就是：定期体检。使用监控系统把需要关注的指标采集起来，形成报告，并对需要关注的异常数据进行分析形成告警。特点是：

- 低频

- 定期

- 定量

Monitoring的需求并没有包含非常高的并发量和通讯量。反过来说：高并发、大数据量的需求并不适用于Monitoring这个点。



### 链路追踪

Tracing（链路追踪）举例来说就是：对某一项工作的定期汇报。某个工作开始做了A，制作A事件的报告，收集起来，然后这个工作还有B、C、D等条目，一个个处理，然后都汇总进报告，最终的结果就是一个Tracing。

特点是：

- 高频
- 巨量
- 有固定格式

因为Tracing是针对某一个事件（一般来说就是一个API），而这个API可能会和很多组件进行沟通，后续的所有的组件沟通无论是内部还是外部的IO，都算作这个API调用的Tracing的一部分。

可以想见在一个业务繁忙的系统中，API调用的数量已经是天文数字，而其衍生出来的Tracing记录更是不得了的量。其特点就是高频、巨量，一个API会衍生出大量的子调用。

也因此适合用来做Monitoring的系统就不一定适合做Tracing了，用Prometheus这样的系统来做Tracing肯定完蛋（Prometheus只有拉模式，全部都是HTTP请求，高并发直接挂掉）。

一般来说Tracing系统都会在本地磁盘IO上做日志（最高效、也是最低的Cost），然后再通过本地Agent慢慢把文本日志数据发送到聚合服务器上，甚至可能在聚合服务器和本地的Agent之间还需要做消息队列，让聚合服务器慢慢消化巨量的消息。

Tracing在现在的业界是有标准的：`OpenTracing`，因此它不是很随意的日志/事件聚合，而是有格式要求的日志/事件聚合，这就是Tracing和Logging最大的不同。



### 日志系统
Logging（日志）举例来说就是：废品回收站。各种各样的物品都会汇总进入到配品回收站里，然后经过分门别类归纳整理，成为各种可回收资源分别回收到商家那里。一般来说我们在大型系统中提到Logging说的都不是简单的日志，而是日志聚合系统。
		从本质上来说，Monitoring和Tracing都是Logging，Logging是这三者中覆盖面最大的超集，而前两者则是其一部分的子集。Logging最麻烦的是，开发者也不会完全知道最后记录进入到日志系统里的一共会有哪些东西，只有在使用（检索）的时候才可能需要汇总查询总量中的一部分。
		要在一般的Loggin系统中进行Monitoring也是可以的，直接把聚合进来的日志数据提取出来，定期形成数据报告，就是监控了。Tracing也是一样，只要聚合进了Logging系统，有了原始数据，后面要做都是可以做的。因此Logging系统最为通用，其特点和Tracing基本一致，也是需要处理高频并发和巨大的数据量。

### 总结

- Monitoring系统从根本的需求和基本设计上就不可能支持Tracing和Logging：低频 vs 高频、低量 vs 高量，其从设计到实现就只为了监控服务

- Tracing系统对提供的数据有格式要求，且处理方式和一般的Logging也不同，有更限定的应用范围

- Logging系统可以处理前两者的需求，但前两者的领域有更专业的工具就不推荐直接使用普通的日志聚合系统了；Logging系统一般用来处理大型系统的日志聚合以及检索查询

  

# 配置中心

# 注册中心

# 系统监控





# 链路追踪

**链路追踪技术**主要是**收集、存储、分析**分布式系统中的调用事件数据，协助开发运营人员进行故障诊断、容量预估、性能瓶颈定位以及调用链路梳理。 链路追踪技术包含了**数据埋点、收集、存储、分析**等相关技术，是一套技术体系。 



**链路追踪的主要功能**：

- 分布式调用链查询和诊断：追踪分布式架构中的所有微服务用户请求，并将它们汇总成分布式调用链。
- 应用性能实时汇总：通过追踪整个应用程序的用户请求，来实时汇总组成应用程序的单个服务和资源。
- 分布式拓扑动态发现：用户的所有分布式微服务应用和相关 PaaS 产品可以通过链路追踪收集到的分布式调用信息。
- 多语言开发程序接入：基于 OpenTracing 标准，全面兼容开源社区，例如 Jaeger 和 Zipkin。
- 丰富的下游对接场景：收集的链路可直接用于日志分析，且可对接到 MaxCompute 等下游分析平台。



## 基础概念

- `Span`：基本工作单元，发送一个远程调度任务 就会产生一个Span，Span是一个64位ID唯一标识的，Trace是用另一个64位ID唯一标识的，Span还有其他数据信息，比如摘要、时间戳事件、Span的ID、以及进度ID。

- `Trace`：一系列Span组成的一个树状结构。请求一个微服务系统的API接口，这个API接口，需要调用多个微服务，调用每个微服务都会产生一个新的Span，所有由这个请求产生的Span组成了这个Trace。

- `Annotation`：用来及时记录一个事件的，一些核心注解用来定义一个请求的开始和结束 。

  - `cs` - Client Sent(客户端发送响应)客户端发送一个请求，这个注解描述了这个Span的开始。
  
  - `sr` - Server Received(服务端接收响应)服务端获得请求并准备开始处理它，如果将其sr减去cs时间戳便可得到网络传输的时间。
  
- `ss`- Server Sent(服务端发送响应)–该注解表明请求处理的完成(当请求返回客户端)，如果ss的时间戳减去sr时间戳，就可以得到服务器请求的时间。
  
  - `cr` - Client Received(客户端接收响应)-此时Span的结束，如果cr的时间戳减去cs时间戳便可以得到整个请求所消耗的时间。
  
## Opentracing规范

Opentracing是分布式链路追踪的一种规范标准，是CNCF（云原生计算基金会）下的项目之一。和一般的规范标准不同，Opentracing不是传输协议，消息格式层面上的规范标准，而是一种语言层面上的API标准。以Go语言为例，只要某链路追踪系统实现了Opentracing规定的接口（interface），符合Opentracing定义的表现行为，那么就可以说该应用符合Opentracing标准。这意味着开发者只需修改少量的配置代码，就可以在符合Opentracing标准的链路追踪系统之间自由切换。

[OpenTracing官方标准-中文版](https://github.com/opentracing-contrib/opentracing-specification-zh)

[OpenTracing文档中文版](https://wu-sheng.gitbooks.io/opentracing-io/content/)



## 链路追踪组件

### 组件比较

Zipkin是Twitter基础Dapper论文开源的，遵循但是不完全匹配Opentracing规范的链路分析工具。

Zipkin的Span没有Log Event相关的配置，无法使用阿里云链路追踪的异常诊断功能。

>**提示：**异常数据的统计依据是Log Event中的field 'event'='error'和field 'stack'='**Exception:***' 。请参见OpenTracing规范中的[Log Field部分。](https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/semantic_conventions.md)

Zipkin结合Sleuth可以实现自动埋点，需要添加相关依赖，支持手动埋点。



Skywalking是国产的，被Apache管理的链路分析和性能监控的工具。

### Zipkin

### Skywalking




## 参考文档

- [【剖析 | SOFARPC 框架】之SOFARPC 链路追踪剖析](https://developer.aliyun.com/article/662497)
- [分布式服务跟踪及Spring Cloud的实现](http://daixiaoyu.com/distributed-tracing.html)
- [Spring Cloud Sleuth + Zipkin 实现服务追踪](https://www.huaweicloud.com/articles/93178d251c9b1545b3a1e49d2465a4f1.html)



# 日志系统