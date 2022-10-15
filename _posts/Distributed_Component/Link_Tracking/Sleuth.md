# 目录

[TOC]

# Sleuth

[官方文档](https://cloud.spring.io/spring-cloud-static/spring-cloud-sleuth/2.0.1.RELEASE/single/spring-cloud-sleuth.html#_running_examples)

Spring Cloud Sleuth 为 Spring Cloud 实现了分布式跟踪解决方案。兼容 Zipkin，HTrace 和其他基于日志的追踪系统，例如 ELK（Elasticsearch 、Logstash、 Kibana）。

Spring Cloud Sleuth 提供了以下功能：

- `链路追踪`：通过 Sleuth 可以很清楚的看出一个请求都经过了那些服务，可以很方便的理清服务间的调用关系等。
- `性能分析`：通过 Sleuth 可以很方便的看出每个采样请求的耗时，分析哪些服务调用比较耗时，当服务调用的耗时随着请求量的增大而增大时， 可以对服务的扩容提供一定的提醒。
- `数据分析，优化链路`：对于频繁调用一个服务，或并行调用等，可以针对业务做一些优化措施。
- `可视化错误`：对于程序未捕获的异常，可以配合 Zipkin 查看。



## yaml配置

因为微服务所有的请求都从网关进行分发，因此需要在网关中配置采集策略。

```yacas
spring:
  sleuth:
    sampler:
      # 设置抽样采集率为100%，默认为0.1，即10%
      probability: 1.0f
      # 每秒允许的采集数，和probability同时存在时以rate为准
      rate: 10000
```



如果同时配置了rate和probability属性，则优先读取rate属性。

`org.springframework.cloud.sleuth.sampler.SamplerAutoConfiguration`

```java
static Sampler samplerFromProps(SamplerProperties config) {
    if (config.getRate() != null) {
        return new RateLimitingSampler(config);
    }
    return new ProbabilityBasedSampler(config);
}
```



设置probability为`1.0f`时，表示100%采集。

`org.springframework.cloud.sleuth.sampler.ProbabilityBasedSampler`

```java
@Override
public boolean isSampled(long traceId) {
    if (this.configuration.getProbability() == 0) {
        return false;
    }
    else if (this.configuration.getProbability() == 1.0f) {
        return true;
    }
    synchronized (this) {
        final int i = this.counter.getAndIncrement();
        boolean result = this.sampleDecisions.get(i);
        if (i == 99) {
            this.counter.set(0);
        }
        return result;
    }
}
```



## 集成Zipkin

**maven依赖**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```



**yaml配置**

```yaml
spring:
  zipkin:
    enabled: true
    # Zipkin地址，可以使用自建的Zipkin服务或阿里云的链路追踪的服务地址
    base-url: http://tracing-analysis-dc-sz.aliyuncs.com/***
    # 关闭服务发现，否则Spring Cloud会把zipkin的url当做服务名称
    discoveryClientEnabled: false
    sender:
      # 设置使用http的方式传输数据
      type: web
```



## 添加自定义信息



## 参考文档

- [[spring boot 2.0.3+spring cloud （Finchley）7、服务链路追踪Spring Cloud Sleuth](https://www.cnblogs.com/cralor/p/9246582.html)]