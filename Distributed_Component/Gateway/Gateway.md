# 概念

SpringCloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

SpringCloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Zuul，在Spring Cloud 2.0以上版本中，没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 2.0之前的非Reactor模式的老版本。而为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。

Spring Cloud Gateway 的目标，不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标，和限流。



**SpringCloud Gateway 特征**

SpringCloud官方，对SpringCloud Gateway 特征介绍如下：

（1）基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0

（2）集成 Hystrix 断路器

（3）集成 Spring Cloud DiscoveryClient

（4）Predicates 和 Filters 作用于特定路由，易于编写的 Predicates 和 Filters

（5）具备一些网关的高级功能：动态路由、限流、路径重写

从以上的特征来说，和Zuul的特征差别不大。SpringCloud Gateway和Zuul主要的区别，还是在底层的通信框架上。





## 工作流程

![图片](../../../Image/2022/09/220922-1.jpg)

① **路由判断**；客户端的请求到达网关后，先经过 Gateway Handler Mapping 处理，这里面会做断言（Predicate）判断，看下符合哪个路由规则，这个路由映射后端的某个服务。

② **请求过滤**：然后请求到达 Gateway Web Handler，这里面有很多过滤器，组成过滤器链（Filter Chain），这些过滤器可以对请求进行拦截和修改，比如添加请求头、参数校验等等，有点像净化污水。然后将请求转发到实际的后端服务。这些过滤器逻辑上可以称作 Pre-Filters，Pre 可以理解为“在...之前”。

③ **服务处理**：后端服务会对请求进行处理。

④ **响应过滤**：后端处理完结果后，返回给 Gateway 的过滤器再次做处理，逻辑上可以称作 Post-Filters，Post 可以理解为“在...之后”。

⑤ **响应返回**：响应经过过滤处理后，返回给客户端。



# 动态路由

**Route**（路由）是网关配置的基本组成模块，和Zuul的路由配置模块类似。一个**Route模块**由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配，目标URI会被访问。



**uri: lb://passjava-question**，表示将请求转发给 passjava-question 微服务，且支持负载均衡。lb 是 loadbalance（负载均衡) 单词的缩写。

当 passjava-question 服务添加一个微服务，或者 IP 地址更换了，Gateway 都是可以感知到的，但是配置是不需要更新的。这里的动态指的是微服务的集群个数、IP 和端口是动态可变的。



# 断言

## 概念

这是一个 Java 8 的 Predicate，可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。**断言的**输入类型是一个 ServerWebExchange。

```yaml
spring:
  cloud:
    gateway:
      # 路由数组：指当请求满足什么样的断言时，转发到哪个服务上
      routes:
        # 路由标识，要求唯一，名称任意
        - id: rule
          # 负载均衡，将请求转发到指定服务或地址
          uri: lb://java
          # 断言，判断是否应用这条路由规则
          predicates:
            - Path=/api/java/**
            - Weight=group1, 8
          filters: 
            - RewritePath=/api/java
```



Spring Cloud Gateway 中的断言命名都是有规范的，格式：“`xxx + RoutePredicateFactory`”，比如权重断言 `WeightRoutePredicateFactory`，那么配置时直接取前面的 “Weight”。

如果路由转发匹配到了两个或以上，则是的按照配置先后顺序转发，上面都配置了路径：“ `Path=/gateway/provider/**` ”，如果没有配置权重，则肯定是先转发到 “`http://localhost:9024`”，但是既然配置配置了权重并且相同的分组，则按照权重比例进行分配流量。



### 路由和断言的关系

- **一对多**：一个路由规则可以包含多个断言。如上图中路由 Route1 配置了三个断言 Predicate。
- **同时满足**：如果一个路由规则中有多个断言，则需要同时满足才能匹配。如上图中路由 Route2 配置了两个断言，客户端发送的请求必须同时满足这两个断言，才能匹配路由 Route2。
- **第一个匹配成功**：如果一个请求可以匹配多个路由，则映射第一个匹配成功的路由。如上图所示，客户端发送的请求满足 Route3 和 Route4 的断言，但是 Route3 的配置在配置文件中靠前，所以只会匹配 Route3。



## 内置断言

### AfterRoutePredicateFactory

### BeforeRoutePredicateFactory

### BetweenRoutePredicateFactory

### CloudFoundryRouteServiceRoutePredicateFactory

### CookieRoutePredicateFactory

### HeaderRoutePredicateFactory

### HostRoutePredicateFactory

### MethodRoutePredicateFactory

### PathRoutePredicateFactory

### QueryRoutePredicateFactory

### ReadBodyPredicateFactory

### RemoteAddrRoutePredicateFactory

### WeightRoutePredicateFactory



## 生产实践

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: route_qq
          uri: http://www.qq.com
          predicates: 
            - Query=url,qq
        - id: route_baidu
          uri: http://www.baidu.com
          predicates:
            - Query=url,baidu
```



第一条路由规则：断言为 Query=url,qq，表示当请求路径中包含 url=qq，则跳转到 http://www.qq.com。

第二条路由规则：当请求路径中包含 url=baidu，则跳转到 http://www.baidu.com。



# 过滤器

## 概念

**Filter**（过滤器）和Zuul的过滤器在概念上类似，可以使用它拦截和修改请求，并且对上游的响应，进行二次处理。过滤器为org.springframework.cloud.gateway.filter.GatewayFilter类的实例。

过滤器名称只需要写前缀，过滤器命名必须是 "`xxx+GatewayFilterFactory`“（包括自定义）。



### 过滤器类型

#### 按请求响应划分

过滤器 Filter 按照请求和响应可以分为两种：`Pre` 类型和 `Post` 类型。

**Pre 类型**：在请求被转发到微服务之前，对请求进行拦截和修改，例如参数校验、权限校验、流量监控、日志输出以及协议转换等操作。

**Post 类型**：微服务处理完请求后，返回响应给网关，网关可以再次进行处理，例如修改响应内容或响应头、日志输出、流量监控等。

由于过滤器有 pre 和 post 两种类型，pre 类型过滤器如果 order 值越小，那么它就应该在pre过滤器链的顶层，post 类型过滤器如果 order 值越小，那么它就应该在 post 过滤器链的底层。



#### 按作用范围划分

另外一种分类是按照过滤器 Filter 作用的范围进行划分：

**GatewayFilter**：局部过滤器，应用在单个路由或一组路由上的过滤器。标红色表示比较常用的过滤器。局部过滤器需要在指定路由配置才能生效，默认是不生效的。

**GlobalFilter**：全局过滤器，应用在所有路由上的过滤器。无需配置，全局生效。



`GlobalFilter` 的功能其实和 `GatewayFilter` 是相同的，只是 `GlobalFilter` 的作用域是所有的路由配置，而不是绑定在指定的路由配置上。多个 `GlobalFilter` 可以通过 `@Order` 或者` getOrder()` 方法指定执行顺序，order值越小，执行的优先级越高。



## 内置局部过滤器

### LoadBalancerClientFilter

匹配uri配置中的关键字 `lb`（LoadBalancer），表示支持负载均衡转发。



## 自定义局部过滤器

```java
/**
 * 名称必须是xxxGatewayFilterFactory形式
 * todo：模拟授权的验证，具体逻辑根据业务完善
 */
@Component
@Slf4j
public class AuthorizeGatewayFilterFactory extends AbstractGatewayFilterFactory<AuthorizeGatewayFilterFactory.Config> {
 
    private static final String AUTHORIZE_TOKEN = "token";
 
    //构造函数，加载Config
    public AuthorizeGatewayFilterFactory() {
        //固定写法
        super(AuthorizeGatewayFilterFactory.Config.class);
        log.info("Loaded GatewayFilterFactory [Authorize]");
    }
 
    //读取配置文件中的参数 赋值到 配置类中
    @Override
    public List<String> shortcutFieldOrder() {
        //Config.enabled
        return Arrays.asList("enabled");
    }
 
    @Override
    public GatewayFilter apply(AuthorizeGatewayFilterFactory.Config config) {
        return (exchange, chain) -> {
            //判断是否开启授权验证
            if (!config.isEnabled()) {
                return chain.filter(exchange);
            }
 
            ServerHttpRequest request = exchange.getRequest();
            HttpHeaders headers = request.getHeaders();
            //从请求头中获取token
            String token = headers.getFirst(AUTHORIZE_TOKEN);
            if (token == null) {
                //从请求头参数中获取token
                token = request.getQueryParams().getFirst(AUTHORIZE_TOKEN);
            }
 
            ServerHttpResponse response = exchange.getResponse();
            //如果token为空，直接返回401，未授权
            if (StringUtils.isEmpty(token)) {
                response.setStatusCode(HttpStatus.UNAUTHORIZED);
                //处理完成，直接拦截，不再进行下去
                return response.setComplete();
            }
            /**
             * todo chain.filter(exchange) 之前的都是过滤器的前置处理
             *
             * chain.filter().then(
             *  过滤器的后置处理...........
             * )
             */
            //授权正常，继续下一个过滤器链的调用
            return chain.filter(exchange);
        };
    }
 
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Config {
        // 控制是否开启认证
        private boolean enabled;
    }
}
```



局部过滤器需要在路由中配置才能生效，配置如下：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: gateway-provider_1
          uri: http://localhost:9024
          predicates:
            - Path=/gateway/provider/**
          ## 配置过滤器（局部）
          filters:
            - AddResponseHeader=X-Response-Foo, Bar
            ## AuthorizeGatewayFilterFactory自定义过滤器配置，值为true需要验证授权，false不需要
            - Authorize=true
```



## 内置全局过滤器

### ReactiveLoadBalancerClientFilter



## 自定义全局过滤器

```java
@Slf4j
@Component
@Order(value = Integer.MIN_VALUE)
public class AccessLogGlobalFilter implements GlobalFilter {
 
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        //filter的前置处理
        ServerHttpRequest request = exchange.getRequest();
        String path = request.getPath().pathWithinApplication().value();
        InetSocketAddress remoteAddress = request.getRemoteAddress();
        return chain
                //继续调用filter
                .filter(exchange)
                //filter的后置处理
                .then(Mono.fromRunnable(() -> {
            ServerHttpResponse response = exchange.getResponse();
            HttpStatus statusCode = response.getStatusCode();
            log.info("请求路径:{},远程IP地址:{},响应码:{}", path, remoteAddress, statusCode);
        }));
    }
}
```



# 实践

## 依赖引入

```xml

```



## 配置

### 自动路由配置

```yaml
spring:
  application:
    name: gateway
  cloud:
    gateway:
      discovery:
        locator:
          # 设置为true表明开启服务发现和路由的功能，
          # 网关自动根据注册中心的服务名为每个服务创建一个router
          # 将以服务名开头的请求路径转发到对应的服务
          enabled: true
          # 路由的路径默认会使用大写ID，
          # 若想要使用小写ID，可将lowerCaseServiceId设置为true
          lower-case-service-id: true
```



## 自定义全局异常处理

一旦路由的微服务下线或者失联了，Spring Cloud Gateway直接返回了一个错误页面，这种异常信息不友好，前后端分离架构中必须定制返回的异常信息。传统的Spring Boot 服务中都是使用 `@ControllerAdvice` 来包装全局异常处理的，但是由于服务下线，请求并没有到达。因此必须在网关中也要定制一层全局异常处理，这样才能更加友好的和客户端交互。



```java
@Slf4j
@Order(-1)
@Component
@RequiredArgsConstructor
public class GlobalErrorExceptionHandler implements ErrorWebExceptionHandler {
    private final ObjectMapper objectMapper;

    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
        ServerHttpResponse response = exchange.getResponse();
        if (response.isCommitted()) {
            return Mono.error(ex);
        }

        // JSON格式返回
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);
        if (ex instanceof ResponseStatusException) {
            response.setStatusCode(((ResponseStatusException) ex).getStatus());
        }

        return response.writeWith(Mono.fromSupplier(() -> {
            DataBufferFactory bufferFactory = response.bufferFactory();
            try {
                //todo 返回响应结果，根据业务需求，自己定制
                Map<String, String> result = new HashMap<>(6);
                result.put("code", "500");
                return bufferFactory.wrap(objectMapper.writeValueAsBytes(result));
            } catch (JsonProcessingException e) {
                log.error("Error writing response", ex);
                return bufferFactory.wrap(new byte[0]);
            }
        }));
    }
}
```



## 集成Nacos

- 网关服务需要知道所有服务的域名或IP地址，另外，一旦服务的域名或IP地址发生修改，路由配置中的 uri 就必须修改
- 服务集群中无法实现负载均衡



那么此时可以集成的注册中心，使得网关能够从注册中心自动获取uri，并实现负载均衡。



- 引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```



- 添加 @EnableDiscoveryClient 注解开启注册中心功能

```java
@EnableDiscoveryClient
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```



- 配置nacos注册中心配置

```yaml
spring:
  cloud:
    nacos:
      discovery:
       server-addr: 120.76.129.106:80
        namespace: 856a40d7-6548-4494-bdb9-c44491865f63
        register-enabled: true
```



- 服务路由配置

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: gateway-provider_1
          ## 使用了lb形式，从注册中心负载均衡的获取uri
          uri: lb://gateway-provider
          ## 配置断言
          predicates:
            - Path=/gateway/provider/**
          filters:
            - AddResponseHeader=X-Response-Foo, Bar
```



路由配置中唯一不同的就是路由的 uri，格式：`lb://service-name`，这是固定写法：

- lb：固定格式，指的是从nacos中按照名称获取微服务，并遵循负载均衡策略
- service-name：nacos注册中心的服务名称，这里并不是IP地址形式的

全局过滤器 `LoadBalancerClientFilter` 就是负责路由寻址和负载均衡的。





# 参考资料

- [Spring Cloud Gateway官方文档](https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/)
