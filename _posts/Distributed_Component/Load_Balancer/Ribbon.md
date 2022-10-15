## Ribbon是如何实现失败重试的？

`Ribbon`提供`RetryHandler`接口，并且默认使用`DefaultLoadBalancerRetryHandler`。`LoadBalancerCommand`的`submit`方法中（在`FeignLoadBalancer`的`executeWithLoadBalancer`方法中调用），如果配置重试次数大于`0`，则会调用`RxJava`的`API`支持重试。

```java
public Observable<T> submit(final ServerOperation<T> operation) {
        // .......
        final int maxRetrysSame = retryHandler.getMaxRetriesOnSameServer();
        final int maxRetrysNext = retryHandler.getMaxRetriesOnNextServer();
        // Use the load balancer
        Observable<T> o = (server == null ? selectServer() : Observable.just(server))
                .concatMap(new Func1<Server, Observable<T>>() {
                    @Override
                    public Observable<T> call(Server server) {
                        //.......
                        // 调用相同节点的重试次数
                        if (maxRetrysSame > 0) 
                            o = o.retry(retryPolicy(maxRetrysSame, true));
                        return o;
                    }
                });
        // 调用不同节点的重试次数
        if (maxRetrysNext > 0 && server == null) 
            o = o.retry(retryPolicy(maxRetrysNext, false));
        return o.onErrorResumeNext(...);
    }
复制代码
```



默认`maxRetrysSame`（调用相同的重试次数）为`0`，默认`maxRetrysNext`（调用不同节点的重试次数）为`1`。`retryPolicy`方法是返回的是一个判断是否重试的决策者，由该决策者决定是否需要重试（抛出的异常是否允许重试，是否达到最大重试次数）。

```typescript
private Func2<Integer, Throwable, Boolean> retryPolicy(final int maxRetrys, final boolean same) {
        return new Func2<Integer, Throwable, Boolean>() {
            @Override
            public Boolean call(Integer tryCount, Throwable e) {
                if (e instanceof AbortExecutionException) {
                    return false;
                }
                // 大于最大重试次数
                if (tryCount > maxRetrys) {
                    return false;
                }
                if (e.getCause() != null && e instanceof RuntimeException) {
                    e = e.getCause();
                }
                // 调用RetryHandler判断是否重试
                return retryHandler.isRetriableException(e, same);
            }
        };
    }
```

