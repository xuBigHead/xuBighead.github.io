# Nacos 注册中心

## 实现原理

### 服务注册

- 服务启动后会将自己的IP和端口号通知注册中心
- Nacos每5秒发送心跳来获取服务的状态，15秒没有收到心跳的服务会被设置为不健康，30秒没收到心跳的服务会被临时摘除。



![img](../../../Image/2022/09/220926-5.jpg)



### 服务发现

服务注册到注册中心后，服务的消费者就可以向注册中心订阅某个服务，并提交一个监听器，当注册中心中服务发生变更时，监听器会收到通知，这时消费者更新本地的服务实例列表，以保证所有的服务均是可用的。

服务提供者与服务消费者之间是通过feign+ribbon进行配合调用的，feign提供http请求的封装以及调用，ribbon提供负载均衡。负载均衡有很多种实现方式，包括轮询法，随机方法法，对请求ip做hash后取模等等。 Nacos的客户端在获取到服务的完整实例列表后，会在客户端进行负载均衡算法来获取一个可用的实例，默认使用的是随机获取的方式。



# Nacos 配置中心

## 实现原理

对于nacosconfig来讲其实就是提供了一系列的crud的访问接口使应用可以完成配置的增删改查的操作。

当config配置进行更新变化的时候就需要相关的应用进行跟着相关变化，这里就有个问题了那客户端怎么知道配置变化了呢，客户端是什么时候进行更新的呢？涉及更新方式就有2种，推和拉；

- 客户端主动从服务端定时拉取配置，如果有变化则进行替换。
- 服务端主动把变化的内容发送给客户端。

两种方式各有利弊，比如对于推的模式来讲，就需要服务端与客户端进行长连接，那么这种就会出现服务端需要耗费大量资源维护这个链接，并且还得加入心跳机制来维护连接有效性。而对于拉的模式则需要客户端定时去服务端访问，那么就会存在时间间隔，也就保证不了数据的实时性。那nacos采用哪种模式呢？nacos是采用了拉模式是一种特殊的拉模式，也就是我们通常听的长轮询机制。



### 长连接

1. 如果服务端配置与客户端一直没有变化

如果客户端拉取发现客户端与服务端配置是一致的（其实是通过MD5判断的）那么服务端会先拿住这个请求不返回，直到这段时间内配置有变化了才把刚才拿住的请求返回。他的步骤是nacos服务端收到请求后检查配置是否发生变化，如果没有则开启定时任务，延迟29.5s执行。同时把当前客户端的连接请求放入队列。那么此时服务端并没有将结果返回给客户端，当有以下2种情况的时候才触发返回。

- 就是等待29.5s后触发自动检查
- 在29.5s内有配置进行了更改

经过这2种情况才完成这次的pull操作。这种的好处就是保证了客户端的配置能及时变化更新，也减少了轮询给服务端带来的压力。所以之前文章我们说过这个长链接回话超时时间默认是30s。

![img](../../../Image/2022/09/220926-6.jpg)



# 扩展

## 基于nacos优雅停服的小坑

在采用金丝雀灰度发布的过程中发现服务已经把原来的注册的服务进行下线操作了，但是仍然有流量请求进来，这样就违背了官方宣称的秒级上下线特点。其中我们服务的负载均衡使用的ribbon这个负载均衡组件。我们通过源码看一下：



### 注册中心更新核心机制

```text
public String update(HttpServletRequest request) throws Exception {
        String serviceName = WebUtils.required(request, CommonParams.SERVICE_NAME);
        String namespaceId = WebUtils.optional(request, CommonParams.NAMESPACE_ID, Constants.DEFAULT_NAMESPACE_ID);
        String agent = request.getHeader("Client-Version");
        if (StringUtils.isBlank(agent)) {
            agent = request.getHeader("User-Agent");
        }
        ClientInfo clientInfo = new ClientInfo(agent);
        if (clientInfo.type == ClientInfo.ClientType.JAVA &&
                clientInfo.version.compareTo(VersionUtil.parseVersion("1.0.0")) >= 0) {
            serviceManager.updateInstance(namespaceId, serviceName, parseInstance(request));
      n "ok";
}
```

上边这段代码说明的是nacos进行上线下线的逻辑，可以看到核心逻辑是parseInstance()方法，里面的的实例信息就是从request中获取的。再看一下updateInstance方法：

```text
public void updateInstance(String namespaceId, String serviceName, Instance instance) throws NacosException {
	Service service = getService(namespaceId, serviceName);
	if (service == null) {
		throw new NacosException(NacosException.INVALID_PARAM, "service not found, namespace: " + namespaceId + ", service: " + serviceName);
	}
	if (!service.allIPs().contains(instance)) {
		throw new NacosException(NacosException.INVALID_PARAM, "instance not exist: " + instance);
	}
	addInstance(namespaceId, serviceName, instance.isEphemeral(), instance);
}
public void addInstance(String namespaceId, String serviceName, boolean ephemeral, Instance... ips) throws NacosException {
	String key = KeyBuilder.buildInstanceListKey(namespaceId, serviceName, ephemeral);
	Service service = getService(namespaceId, serviceName);
	List<Instance> instanceList = addIpAddresses(service, ephemeral, ips);
	Instances instances = new Instances();
	instances.setInstanceList(instanceList);
	consistencyService.put(key, instances);
}
```

可以看到实例下线，是立马更新server中的实例信息数据的。



### Ribbon负载均衡更新机制

NacosServerList继承了AbstractServerList最终是在DynamicServerListLoadBalancer这类中进行负载所有的server的

```text
public class NacosServerList extends AbstractServerList<NacosServer> {
	private NacosDiscoveryProperties discoveryProperties;
	private String serviceId;
	public NacosServerList(NacosDiscoveryProperties discoveryProperties) {
		this.discoveryProperties = discoveryProperties;
	}
	@Override
	public List<NacosServer> getInitialListOfServers() {
		return getServers();
	}
	@Override
	public List<NacosServer> getUpdatedListOfServers() {
		return getServers();
	}
	private List<NacosServer> getServers() {
		try {
			List<Instance> instances = discoveryProperties.namingServiceInstance()
					.selectInstances(serviceId, true);
			return instancesToServerList(instances);
		}
		catch (Exception e) {
			throw new IllegalStateException(
					"Can not get service instances from nacos, serviceId=" + serviceId,
					e);
		}
	}
	private List<NacosServer> instancesToServerList(List<Instance> instances) {
		List<NacosServer> result = new ArrayList<>();
		if (null == instances) {
			return result;
		}
		for (Instance instance : instances) {
			result.add(new NacosServer(instance));
		}
		return result;
	}
	public String getServiceId() {
		return serviceId;
	}
	@Override
	public void initWithNiwsConfig(IClientConfig iClientConfig) {
		this.serviceId = iClientConfig.getClientName();
	}
}
protected final ServerListUpdater.UpdateAction updateAction = new ServerListUpdater.UpdateAction() {
        @Override
        public void doUpdate() {
            updateListOfServers();
        }
    };
public DynamicServerListLoadBalancer(IClientConfig clientConfig) {
	initWithNiwsConfig(clientConfig);
}

@Override
public void initWithNiwsConfig(IClientConfig clientConfig) {
	try {
		super.initWithNiwsConfig(clientConfig);
		String niwsServerListClassName = clientConfig.getPropertyAsString( CommonClientConfigKey.NIWSServerListClassName, DefaultClientConfigImpl.DEFAULT_SEVER_LIST_CLASS);
		ServerList<T> niwsServerListImpl = (ServerList<T>) ClientFactory
                    .instantiateInstanceWithClientConfig(niwsServerListClassName, clientConfig);
                //得到所有的server实现
		this.serverListImpl = niwsServerListImpl;

		if (niwsServerListImpl instanceof AbstractServerList) {
			AbstractServerListFilter<T> niwsFilter = ((AbstractServerList) niwsServerListImpl)
                        .getFilterImpl(clientConfig);
			niwsFilter.setLoadBalancerStats(getLoadBalancerStats());
			this.filter = niwsFilter;
		}

		String serverListUpdaterClassName = clientConfig.getPropertyAsString( CommonClientConfigKey.ServerListUpdaterClassName, DefaultClientConfigImpl.DEFAULT_SERVER_LIST_UPDATER_CLASS);
               // 获取Updater对象
		this.serverListUpdater = (ServerListUpdater) ClientFactory.instantiateInstanceWithClientConfig(serverListUpdaterClassName, clientConfig);

		restOfInit(clientConfig);
	} catch (Exception e) {
		throw new RuntimeException(
                    "Exception while initializing NIWSDiscoveryLoadBalancer:"
                            + clientConfig.getClientName()
                            + ", niwsClientConfig:" + clientConfig, e);
	}
}
void restOfInit(IClientConfig clientConfig) {
	boolean primeConnection = this.isEnablePrimingConnections();
	// turn this off to avoid duplicated asynchronous priming done in BaseLoadBalancer.setServerList()
	this.setEnablePrimingConnections(false);
        //采用定时任务进行定时刷新实例信息缓存
	enableAndInitLearnNewServersFeature();//最重要的点
        //进行一次实例拉取操作
	updateListOfServers();
	if (primeConnection && this.getPrimeConnections() != null) {
		this.getPrimeConnections() .primeConnections(getReachableServers());
	}
	this.setEnablePrimingConnections(primeConnection);
	LOGGER.info("DynamicServerListLoadBalancer for client {} initialized: {}", clientConfig.getClientName(), this.toString());
}
// 这里就是进行实例信息缓存更新的操作
@VisibleForTesting
public void updateListOfServers() {
	List<T> servers = new ArrayList<T>();
	if (serverListImpl != null) {
    // 调用拉取新实例信息的方法
		servers = serverListImpl.getUpdatedListOfServers();
		LOGGER.debug("List of Servers for {} obtained from Discovery client: {}", getIdentifier(), servers);
    // 用Filter对拉取的servers列表进行更新
		if (filter != null) {
			servers = filter.getFilteredListOfServers(servers);
			LOGGER.debug("Filtered List of Servers for {} obtained from Discovery client: {}", getIdentifier(), servers);
		}
	}
  // 更新实例列表
	updateAllServerList(servers);
}
```

之后我们看最重要的enableAndInitLearnNewServersFeature这个方法的操作

```text
@Override
public synchronized void start(final UpdateAction updateAction) {
	if (isActive.compareAndSet(false, true)) {
		final Runnable wrapperRunnable = new Runnable() {
			@Override
			public void run() {
				if (!isActive.get()) {
					if (scheduledFuture != null) {
						scheduledFuture.cancel(true);
					}
					return;
				}
				try {
                   // 这里就是在DynamicServerListLoadBalancer中的Servers实现
					updateAction.doUpdate();
					lastUpdated = System.currentTimeMillis();
				} catch (Exception e) {
					logger.warn("Failed one update cycle", e);
				}
			}
		};
                // 默认定时任务执行时间间隔为30s
		scheduledFuture = getRefreshExecutor().scheduleWithFixedDelay(
                    wrapperRunnable,
                    initialDelayMs,
                    refreshIntervalMs,
                    TimeUnit.MILLISECONDS);
	} else {
		logger.info("Already active, no-op");
	}
}
```

最终虽然实现了秒级的实例上下线，但是由于在Spring Cloud中，负载组件rabbion的实例信息更新是采用了定时任务的形式，有可能这个任务上一秒刚刚执行完，下一秒你就执行实例上下线操作，那么ribbion要感知这个变化，就必须要等待refreshIntervalMs秒后才可以感知到。所以了解了根源那么我们才能从底层更好地去使用这些组件。



# 参考资料

- [Nacos 官方文档](https://nacos.io/zh-cn/docs/quick-start.html)