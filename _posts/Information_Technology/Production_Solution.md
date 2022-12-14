

# 通用业务

## 隐藏内部接口

### 业务场景

某些接口不支持外部访问，只能内网服务调用。



### 解决方案

#### 微服务隔离

将对外暴露的接口和对内暴露的接口分别放到两个微服务上，一个服务里所有的接口均对外暴露，另一个服务的接口只能内网服务间调用。

该方案需要额外编写一个只对内部暴露接口的微服务，将所有只能对内暴露的业务接口聚合到这个微服务里，通过这个聚合的微服务，分别去各个业务侧获取资源。

该方案，新增一个微服务做请求转发，增加了系统的复杂性，增大了调用耗时以及后期的维护成本。



####  网关 + redis 实现白名单机制

在 redis 里维护一套接口白名单列表，外部请求到达网关时，从 redis 获取接口白名单，在白名单内的接口放行，反之拒绝掉。

该方案的好处是，对业务代码零侵入，只需要维护好白名单列表即可。缺点在于白名单的维护是一个持续性投入的工作，另外，每次请求进来，都需要判断白名单，增加了系统响应耗时，考虑到正常情况下外部进来的请求大部分都是在白名单内的，只有极少数恶意请求才会被白名单机制所拦截，所以该方案的性价比很低。



#### 网关 + AOP

**对所有经过网关的请求的header里添加一个字段，业务侧接口收到请求后，判断header里是否有该字段，如果有，则说明该请求来自外部，没有，则属于内部服务的调用，再根据该接口是否属于内部接口来决定是否放行该请求。**

该方案将内外网访问权限的处理分布到各个业务侧进行，消除了由网关来处理的系统性瓶颈；同时，开发者可以在业务侧直接确定接口的内外网访问权限，提升开发效率的同时，增加了代码的可读性。

该方案会对业务代码有一定的侵入性，不过可以通过注解的形式，最大限度的降低这种侵入性。



- 网关侧对进来的请求header添加外网标识符: from=public

```java
@Component
public class PublicRequestFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        return chain.filter(
            exchange.mutate().request(
                exchange.getRequest().mutate().header("id", "").header("from", "public").build())
                .build()
        );
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```



- 编写内外网访问权限判断的AOP和注解

```java
@Aspect
@Component
@Slf4j
public class OnlyIntranetAccessAspect {
    @Pointcut("@within(com.demo.spring.aop.OnlyIntranetAccess)")
    public void onlyIntranetAccessOnClass() {
    }

    @Pointcut("@annotation(com.demo.spring.aop.OnlyIntranetAccess)")
    public void onlyIntranetAccessOnMethod() {
    }

    @Before(value = "onlyIntranetAccessOnMethod() || onlyIntranetAccessOnClass()")
    public void before() {
        HttpServletRequest hsr = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        String from = hsr.getHeader("from");
        if (!StringUtils.isEmpty(from) && "public".equals(from)) {
            log.error("This api is only allowed invoked by intranet source");
            throw new ServiceException("This api is only allowed invoked by intranet source");
        }
    }
}
```



```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface OnlyIntranetAccess {
}
```



- 最后在只能内网访问的接口上加上@OnlyIntranetAccess注解



# 电商业务

## 避免重复下单

### 描述

用户快速点了两次 “提交订单”  按钮，浏览器会向后端发送两条创建订单的请求，最终会创建两条一模一样的订单。



### 解决方案

解决方案就是采用**幂等机制**，多次请求和一次请求产生的效果是一样的。



#### 主键唯一约束

利用数据库自身特性 “主键唯一约束”，在插入订单记录时，带上主键值，如果订单重复，记录插入会失败。



操作过程：

- 引入一个服务，用于生成一个“全局唯一的订单号”
- 进入创建订单页面时，前端请求该服务，预生成订单ID
- 提交订单时，请求参数除了业务参数外，还要带上这个预生成订单ID



#### JS脚本控制

前端通过js脚本控制，无法解决用户刷新提交的请求。另外也无法解决恶意提交。不建议采用该方案，如果想用，也只是作为一个补充方案。



#### 附加参数校验

前后约定附加参数校验。当用户点击购买按钮时，渲染下单页面，展示商品、收货地址、运费、价格等信息，同时页面会埋上`Token `信息，用户提交订单时，后端业务逻辑会校验token，有且匹配才认为是合理请求。



## 订单快照，减少存储成本

商品信息是可以修改的，当用户下单后，为了更好解决后面可能存在的买卖纠纷，创建订单时会同步保存一份商品详情信息，称之为订单快照。

同一件商品，会有很多用户会购买，如果热销商品，短时间就会有上万的订单。如果每个订单都创建一份快照，存储成本太高。另外商品信息虽然支持修改，但毕竟是一个低频动作。我们可以理解成，大部分订单的商品快照信息都是一样的，除非下单时用户修改过。

如何实时识别修改动作是解决快照成本的关键所在。我们采用摘要比对的方法‍。创建订单时，先检查商品信息摘要是否已经存在，如果不存在，会创建快照记录。订单明细会关联商品的快照主键。



由于订单快照属于非核心操作，即使失败也不应该影响用户正常购买流程，所以通常采用异步流程执行。



## 购物车，混合存储

购物车是电商系统的标配功能，暂存用户想要购买的商品。分为添加商品、列表查看、结算下单三个动作。

服务端在用户登录态校验时，做了分支路由，当用户未登录时，会创建一个临时`Token`，作为用户的唯一标识，购物车数据挂载在该`Token`下，为了避免购物车数据相互影响以及设计的复杂度，这里会有一个临时购物车表。

临时购物车表的数据量并不会太大，用户不会一直闲着添加购物车玩，当用户登录后，查看自己的购物车，服务端会从请求的cookie里查找购物车`Token`标识，并查询临时购物车表是否有数据，然后合并到正式购物车表里。



## 库存超卖

### 描述

常见的库存扣减方式有：

- 下单减库存：即当买家下单后，在商品的总库存中减去买家购买数量。下单减库存是最简单的减库存方式，也是控制最精确的一种，下单时直接通过数据库的事务机制控制商品库存，这样一定不会出现超卖的情况。但是你要知道，有些人下完单可能并不会付款。
- 付款减库存：即买家下单后，并不立即减库存，而是等到有用户付款后才真正减库存，否则库存一直保留给其他买家。但因为付款时才减库存，如果并发比较高，有可能出现买家下单后付不了款的情况，因为可能商品已经被其他人买走了。
- 预扣库存：这种方式相对复杂一些，买家下单后，库存为其保留一定的时间（如 30 分钟），超过这个时间，库存将会自动释放，释放后其他买家就可以继续购买。在买家付款前，系统会校验该订单的库存是否还有保留：如果没有保留，则再次尝试预扣；如果库存不足（也就是预扣失败）则不允许继续付款；如果预扣成功，则完成付款并实际地减去库存。

至于采用哪一种减库存方式更多是业务层面的考虑，减库存最核心的是大并发请求时保证数据库中的库存字段值不能为负数。



### 解决方案

#### 行级锁

通常在扣减库存的场景下使用行级锁，通过数据库引擎本身对记录加锁的控制，保证数据库的更新的安全性，并且通过`where`语句的条件，保证库存不会被减到 `0` 以下，也就是能够有效的控制超卖的场景。

```
update ... set amount = amount - 1 where id = $id and amount - 1 >=0
```



#### 无符号整数

设置数据库的字段数据为无符号整数，这样减后库存字段值小于零时 SQL 语句会报错。



## MySQL读写分离带来的数据不一致问题

### 描述

互联网业务大部分都是 `读多写少`，为了提升数据库集群的吞吐性能，我们通常会采用 `主从架构`、`读写分离`。

部署一个主库实例，客户端请求`所有写操作`全部写到主库，然后借助 MySQL 自带的 `主从同步` 功能，做一些简单配置，可以近乎实时的将主库的数据同步给 `多个从库实例`，主从延迟非常小，一般**不超过 1 毫秒**。

主从同步虽然近乎实时，但还是有个 `时间差` ，主库数据刚更新完，但数据还没来得及同步到从库，后续`读请求`直接访问了从库，看到的还是旧数据，影响用户体验。



### 解决方案

支付成功后，并没有立即跳到 `订单详情页`，而是增加了一个 无关紧要的 `中间页（支付成功页）`，一是告诉你支付的结果是成功的，钱没丢，不要担心；另外也可以增加一些推荐商品，引流提升网站的GMV。最重要的，增加了一个缓冲期，为 `订单的主从库数据同步` 争取了更多的时间。



## 历史订单归档

### 描述

电商网站，一般只能查询3个月内的订单，如果你想看看3个月前的订单，需要访问历史订单页面。



### 解决方案

1、冷热数据区分的标准是什么？要结合业务思考，可能要找产品同学一块讨论才能做决策，切记不要拍脑袋。以电商订单为例：

- 方案一：以“下单时间”为标准，将3 个月前的订单数据当作冷数据，3 个月内的当作热数据。
- 方案二：根据“订单状态”字段来区分，已完结的订单当作冷数据，未完结的订单当作热数据。
- 方案三：组合方式，把下单时间 > 3 个月且状态为“已完结”的订单标识为冷数据，其他的当作热数据。

2、如何触发冷热数据的分离

- 方案一：直接修改业务代码，每次业务请求触发冷热数据判断，根据结果路由到对应的冷数据表或热数据表。缺点：如果判断标准是 `时间维度`，数据过期了无法主动感知。
- 方案二：如果觉得修改业务代码，耦合性高，不易于后期维护。可以通过监听数据库变更日志 binlog 方式来触发
- 方案三：常用的手段是跑定时任务，一般是选择凌晨系统压力小的时候，通过跑批任务，将满足条件的冷数据迁移到其他存储介质。在途业务表中只留下来少量的热点数据。

3、如何实现冷热数据分离，过程大概分为三步：

- 判断数据是冷、还是热
- 将冷数据插入冷数据表中
- 然后，从原来的热库中删除迁移的数据

4、如何使用冷热数据

- 方案一：界面设计时会有选项区分，如上面举例的电商订单
- 方案二：直接在业务代码里区分。



