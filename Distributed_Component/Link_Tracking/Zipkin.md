# 目录

[TOC]

# 前言

[Zipkin](https://zipkin.io/) 是 Twitter 公司开发贡献的一款开源的分布式实时数据追踪系统（Distributed Tracking System），基于 Google Dapper 的论文设计而来，其主要功能是聚集各个异构系统的实时监控数据。

它可以收集各个服务器上请求链路的跟踪数据，并通过 Rest API 接口来辅助我们查询跟踪数据，实现对分布式系统的实时监控，及时发现系统中出现的延迟升高问题并找出系统性能瓶颈的根源。除了面向开发的 API 接口之外，它还提供了方便的 UI 组件，每个服务向 Zipkin 报告计时数据，Zipkin 会根据调用关系生成依赖关系图，帮助我们直观的搜索跟踪信息和分析请求链路明细。Zipkin 提供了可插拔数据存储方式：In-Memory、MySql、Cassandra 以及 Elasticsearch。

分布式跟踪系统还有其他比较成熟的实现，例如：Naver 的 PinPoint、Apache 的 HTrace、阿里的鹰眼 Tracing、京东的 Hydra、新浪的 Watchman，美团点评的 CAT，Apache 的 SkyWalking 等。



# 基础概念

![](https://raw.githubusercontent.com/xuBigHead/pic/master/img/20210105161027.jpg)

## 工作原理

![](https://raw.githubusercontent.com/xuBigHead/pic/master/img/20210105161201.jpg)

　　共有四个组件构成了 Zipkin：

- `Collector`：收集器组件，处理从外部系统发送过来的跟踪信息，将这些信息转换为 Zipkin 内部处理的 Span 格式，以支持后续的存储、分析、展示等功能。

- `Storage`：存储组件，处理收集器接收到的跟踪信息，默认将信息存储在内存中，可以修改存储策略使用其他存储组件，支持 MySQL，Elasticsearch 等。

- `RESTful API`：API 组件，为 Web 界面提供查询存储中数据的接口。

- `Web UI`：UI 组件，基于 API 组件实现的上层应用，提供 Web 页面，用来展示 Zipkin 中的调用链和系统依赖关系等。

  

Zipkin 分为两端，一个是 Zipkin 服务端，一个是 Zipkin 客户端，客户端也就是微服务的应用，客户端会配置服务端的 URL 地址，一旦发生服务间的调用的时候，会被配置在微服务里面的 Sleuth 的监听器监听，并生成相应的 Trace 和 Span 信息发送给服务端。发送的方式有两种，一种是消息总线的方式如 RabbitMQ 发送，还有一种是 HTTP 报文的方式发送。



# 采集数据

## RabbitMQ

## HTTP



# 存储数据

## In-Memory(内存)

## MySql

## Cassandra

## Elasticsearch



# 分析数据



# 参考文档

- [Spring Cloud 系列之 Sleuth 链路追踪（一）](https://www.cnblogs.com/mrhelloworld/p/sleuth1.html)