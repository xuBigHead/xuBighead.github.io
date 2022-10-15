# 简介

## 引入方式

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
</dependency>
```



# OKHttp
OKHttp 是现在比较常用的一个 HTTP 客户端访问工具，具有以下特点：

支持 SPDY，可以合并多个到同一个主机的请求。
使用连接池技术减少请求的延迟（如果SPDY是可用的话）。
使用 GZIP 压缩减少传输的数据量。
缓存响应避免重复的网络请求。