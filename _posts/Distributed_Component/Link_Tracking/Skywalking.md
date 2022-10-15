# 目录

[TOC]

# 前言

![2021年1月7日183401](https://raw.githubusercontent.com/xuBigHead/pic/master/img/20210107183531.jpg)



# agent

# collector

# web

# storage

# 应用

## agent相关配置

```properties
# 代理的命名空间，默认为default-namespace
agent.namespace=skywalking_brief
# 服务名，推荐一个服务一个名称，这个将会在界面的拓补图中显示
agent.service_name=spring-boot-service-name
# 每三秒追踪链采样数量，负数表示尽可能的多采集，默认为-1
agent.sample_n_per_3_secs=-1
# 配置认证，需要和backend服务中的认证配置相符
agent.authentication=${SW_AGENT_AUTHENTICATION:XXX}
# 配置在单个segment中出现的span的最大数量，skywalking将用这个配置来估计应用的内存开销
agent.span_limit_per_segment=${SW_AGENT_SPAN_LIMIT:300}
# 配置哪些资源不会被skywalking所捕获
agent.ignore_suffix=.jpg,.jpeg,.js,.css,.png,.bmp,.gif,.ico,.mp3,.mp4,.html,.svg
# 操作名称最大长度
agent.operation_name_threshold=${SW_AGENT_OPERATION_NAME_THRESHOLD:500}
# 配置collector的地址，多个地址之间以“,”隔开
collector.backend_service=192.168.1.180:11800
### agent相关配置
logging.file_name=skywalking_luke.log
# 日志级别，默认为DEBUG
logging.level=INFO
# 日志目录
logging.dir=./logs
# 日志文件最大为300M一个
logging.max_file_size=314572800
# 历史日志文件的最大数量，当数量超过配置上限的时候，旧的日志文件将会被删除。
# 负数或者0意味着该功能将关闭，默认为-1
logging.max_history_files=${SW_LOGGING_MAX_HISTORY_FILES:-1}
# mysql插件配置，配置追踪sql参数，这样可以在sql错误的时候查看是否是参数所引起的
plugin.mysql.trace_sql_parameters=true
```



## 启动springboot服务

启动springboot服务时添加如下参数读取Skywalking的agent文件：

`-javaagent:D:\download\agent\skywalking-agent.jar`



然后添加如下参数覆盖agent中的对应配置

`-Dskywalking.agent.service_name=caibei-demo`

`-Dskywalking.agent.authentication = ***@***_test`

## 阿里云链路追踪

阿里云链路追踪承担了存储、分析的功能，因此只需要配置Skywalking的探针即可。



### 区分测试生产和环境

#### 利用地域来区分

可以选择当前地域或最近地域用于生产环境，选择一个较远的地域用于测试环境，以此达到区分环境的目的。例如，选择北京地域用于生产环境，选择请到或者张家口地域用于测试环境。

- 优点：不同地域的资源是互相隔离的，这样做可以便利地隔离各个环境。

- 缺点：阿里云的部分VPC用户无法通过内网跨地域上报数据。如果是这种情况，建议利用标签来区分环境。

  

#### 利用标签来区分

通过将`agent.authentication`配置后添加`_test`或`_prod`来区分。



## 参考文档

- [SkyWalking安装使用](https://www.jianshu.com/p/5524b4545421)
- [skywalking学习笔记](https://juejin.cn/post/6844903583893159944)



# 备注
- [SkyWalking中文文档](https://skyapm.github.io/document-cn-translation-of-skywalking/zh/8.0.0/)

- [SkyWalking的Github地址](https://github.com/apache/skywalking/blob/v5.0.0-GA/docs/cn/Deploy-skywalking-agent-CN.md?spm=a2c4g.11186623.2.46.5f65372eYJzpdY&file=Deploy-skywalking-agent-CN.md)

- [SkyWalking的Apache下载地址](http://archive.apache.org/dist/skywalking/)

- [SkyWalking国内下载地址](https://mirrors.cloud.tencent.com/apache/skywalking/)