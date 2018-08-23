# Hystrix使用入门文档

## 一、接入步骤

这里以SC (stockcenter)为例，SC会依赖IC (itemcenter)服务

比如IC的商品锁服务，接入Hystrix前服务调用方式

```
public class ItemLockServiceClient {

    @Resource
    private ItemLockService itemLockService;

    public ItemFieldLockModel getFieldLockByItemId(Long kdtId, Long itemId) {

        ItemIdParam itemIdParam = new ItemIdParam();
        itemIdParam.setItemId(itemId);
        itemIdParam.setKdtId(kdtId);

        PlainResult<ItemFieldLockModel> result = itemLockService.getFieldLockByItemId(itemIdParam);
        if (RpcResultUtil.isFailed(result)) {
            LOGGER.warn("查询失败,param=" + itemIdParam + ",code=" + result.getCode() + ",msg=" + result.getMessage());
        }
        RpcResultUtil.assertDubboFailed(result);
        return result.getData();
    }
...
```

### 1.1 封装Hystrix Command

#### HystrixCommand方式

继承HystrixCommand，业务逻辑写在run方法中

```
public class ItemLockGetFieldLockCommand extends HystrixCommand<PlainResult<ItemFieldLockModel>> {

    private static final HystrixCommand.Setter cachedSetter =
        HystrixCommand.Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ItemLock"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("GetFieldLock"))
                .andThreadPoolPropertiesDefaults(HystrixThreadPoolProperties.Setter().
                        withCoreSize(5).//
                        withMaximumSize(5).//
                        withMaxQueueSize(5).//
                        withAllowMaximumSizeToDivergeFromCoreSize(true))
                .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
                        .withExecutionTimeoutInMilliseconds(1000)
                        .withMetricsRollingStatisticalWindowInMilliseconds(50)
                        .withMetricsRollingPercentileWindowBuckets(10)
                        .withMetricsRollingPercentileBucketSize(100)
                        .withMetricsRollingStatisticalWindowInMilliseconds(10000)
                        .withCircuitBreakerEnabled(true)
                        .withCircuitBreakerErrorThresholdPercentage(50)
                        .withCircuitBreakerRequestVolumeThreshold(20));

    private ItemLockService itemLockService;
    private ItemIdParam param;

    protected ItemLockGetFieldLockCommand(ItemLockService itemLockService, ItemIdParam param) {
        super(cachedSetter);
        this.itemLockService = itemLockService;
        this.param = param;
    }

    @Override
    protected PlainResult<ItemFieldLockModel> run() throws Exception {
        return itemLockService.getFieldLockByItemId(param)
    }

    /**
     * 通过提供一个失败回退的方法，Hystrix 在主命令逻辑发生异常时能从这个方法中得到一个默认值或者一些数据作为命令的返回值，从而实现优雅的服务降级
     */
    @Override
    protected PlainResult<ItemFieldLockModel> getFallback() {
        return null;
    }
...
```

调用HystrixCommand

```
public class ItemLockServiceClientV2 {

    public ItemFieldLockModel getFieldLockByItemId(Long kdtId, Long itemId) {
        ...
        ItemLockGetFieldLockCommand command = new ItemLockGetFieldLockCommand(itemLockService, itemIdParam);
        PlainResult<ItemFieldLockModel> result = command.execute();
        if (RpcResultUtil.isFailed(result)) {
            LOGGER.warn("查询失败,param=" + itemIdParam + ",code=" + result.getCode() + ",msg=" + result.getMessage());
        }
        ...
    }
...
```

#### HystrixObservableCommand方式

继承HystrixObservableCommand，业务逻辑写在construct方法中

```
public class ItemLockGetFieldLockObservableCommand extends HystrixObservableCommand<PlainResult<ItemFieldLockModel>> {

    private ItemLockService itemLockService;
    private ItemIdParam param;

    protected ItemLockGetFieldLockObservableCommand(ItemLockService itemLockService, ItemIdParam param) {
        super(HystrixCommandGroupKey.Factory.asKey("ItemLock"));
        this.itemLockService = itemLockService;
        this.param = param;
    }


    @Override
    protected Observable<PlainResult<ItemFieldLockModel>> construct() {
        return Observable.create(new Observable.OnSubscribe<PlainResult<ItemFieldLockModel>>() {
            @Override
            public void call(Subscriber<? super PlainResult<ItemFieldLockModel>> observer) {
                try {
                    PlainResult<ItemFieldLockModel> result = itemLockService.getFieldLockByItemId(param);
                    if (!observer.isUnsubscribed()) {
                        observer.onNext(result);
                        observer.onCompleted();
                    }
                } catch (Exception e) {
                    observer.onError(e);
                }
            }
        }).subscribeOn(Schedulers.io());
    }

    /**
     * 对应于HystrixCommand#getFallback
     */
    @Override
    protected Observable<PlainResult<ItemFieldLockModel>> resumeWithFallback() {
        return Observable.create(
                new Observable.OnSubscribe<PlainResult<ItemFieldLockModel>>(){
                    public void call(Subscriber<? super PlainResult<ItemFieldLockModel>> subscriber) {
                        if (!subscriber.isUnsubscribed()) {
                            subscriber.onNext(null);
                            subscriber.onCompleted();
                        }
                    }
                }
        ).subscribeOn(Schedulers.io());
    }
...
```

调用HystrixObservableCommand

```
public class ItemLockServiceClientV3 {

    public ItemFieldLockModel getFieldLockByItemId(Long kdtId, Long itemId) {
        ...
        ItemLockGetFieldLockObservableCommand command = new ItemLockGetFieldLockObservableCommand(itemLockService, itemIdParam);
        PlainResult<ItemFieldLockModel> result = command.observe().toBlocking().single();
        if (RpcResultUtil.isFailed(result)) {
            LOGGER.warn("查询失败,param=" + itemIdParam + ",code=" + result.getCode() + ",msg=" + result.getMessage());
        }
        ...
    }
...
```

### 1.2 搜集Hystrix Metrix信息

这一步是收集metric信息，用于展示应用及服务状态，以帮助做决策判断

#### 实现HystrixMetricsInitializingBean

```
public class HystrixMetricsInitializingBean {

    public void init() throws Exception {
        Observable<HystrixDashboardStream.DashboardData> dataObservable =
                HystrixDashboardStream.getInstance().observe();
        dataObservable.subscribe(dashboardData -> {

            dashboardData.getCommandMetrics().forEach(commandMetric -> {
                processCommandMetric(commandMetric);
            });

            dashboardData.getThreadPoolMetrics().forEach(threadPoolMetric -> {
                processThreadPoolMetric(threadPoolMetric);
            });
        });
    }

    public static void processThreadPoolMetric(HystrixThreadPoolMetrics threadPoolMetric) {

        ThreadPoolMetricDO metricDO = new ThreadPoolMetricDO();
        metricDO.setCurrentTime(System.currentTimeMillis());
        metricDO.setThreadPoolKey(threadPoolMetric.getThreadPoolKey().name());

        metricDO.setRollingCountThreadsExecuted(threadPoolMetric.getRollingCountThreadsExecuted());
        metricDO.setRollingCountThreadsRejected(threadPoolMetric.getRollingCountThreadsRejected());

        metricDO.setCumulativeCountThreadsExecuted(threadPoolMetric.getCumulativeCountThreadsExecuted());
        metricDO.setCumulativeCountThreadsRejected(threadPoolMetric.getCumulativeCountThreadsRejected());

        metricDO.setCurrentActiveCount(threadPoolMetric.getCurrentActiveCount().intValue());
        metricDO.setCurrentCompletedTaskCount(threadPoolMetric.getCurrentCompletedTaskCount().longValue());

        metricDO.setCurrentCorePoolSize(threadPoolMetric.getCurrentCorePoolSize().intValue());
        metricDO.setCurrentPoolSize(threadPoolMetric.getCurrentPoolSize().intValue());

        metricDO.setCurrentQueueSize(threadPoolMetric.getCurrentQueueSize().intValue());
        metricDO.setCurrentTaskCount(threadPoolMetric.getCurrentTaskCount().intValue());

        System.out.println(metricDO);
        //TODO send to MQ or write to storage
    }

    public static void processCommandMetric(HystrixCommandMetrics commandMetric) {

        CommandMetricDO metricDO = new CommandMetricDO();
        metricDO.setCurrentTime(System.currentTimeMillis());
        metricDO.setCommandGroup(commandMetric.getCommandGroup().name());
        metricDO.setCommandName(commandMetric.getCommandKey().name());
        metricDO.setThreadPoolKey(commandMetric.getThreadPoolKey().name());

        metricDO.setTotalTimeMean(commandMetric.getTotalTimeMean());
        metricDO.setExecutionTimeMean(commandMetric.getExecutionTimeMean());
        metricDO.setCurrentConcurrentExecutionCount( commandMetric.getCurrentConcurrentExecutionCount());

        HystrixCommandMetrics.HealthCounts health = commandMetric.getHealthCounts();
        metricDO.setTotalRequests(health.getTotalRequests());
        metricDO.setErrorCount(health.getErrorCount());
        metricDO.setErrorPercentage(health.getErrorPercentage());

        System.out.println(metricDO);
        //TODO send to MQ or write to storage
    }
...
```

备注：这一步只是简单的输出了应用相关的统计信息，真实应用中这一步一般的处理策略是将统计信息封装成对象发送到MQ中，由对应的处理程序接受MQ消息处理后进行展示

在application-context.xml文件中配置

```
<bean id="hystrixMetricsInitializingBean" class="xxx.hystix.HystrixMetricsInitializingBean" init-method="init"/>
```

```
更详细的使用方式参见：https://github.com/Netflix/Hystrix/wiki/How-To-Use
```

## 二、Hystrix配置说明

```
配置说明 [https://github.com/Netflix/Hystrix/wiki/Configuration]
```

#### 基本参数

参数 | 作用 | 默认值 | 备注
--- | --- | --- | ---
groupKey | 标识服务分组，比如IC就是一个服务分组 | | 相同分组的服务会聚合在一起，此参数必填
commandKey | 标识服务 | getClass().getSimpleName() |
threadPoolKey | 标识线程池 | 默认是分组名 | 

#### 执行参数

参数 | 作用 | 默认值 | 备注
--- | --- | --- | ---
execution.isolation.strategy | 控制HystrixCommand.run()的执行策略 | THREAD | 有THREAD和SEMAPHORE THREAD，以下几种可以使用SEMAPHORE模式：只想控制并发度 外部的方法已经做了线程隔离 调用的是本地方法或者可靠度非常高、耗时特别小的方法（如medis）
execution.isolation.thread.timeoutInMilliseconds | 超时时间 | 1000ms | 在THREAD模式下，达到超时时间，可以中断；在SEMAPHORE模式下，会等待执行完成后，再去判断是否超时 设置标准：有retry，99meantime+avg meantime 没有retry，99.5 meantime
execution.timeout.enabled | 是否打开超时 | true | 
execution.isolation.thread.interruptOnTimeout | 是否打开超时线程中断 | true | THREAD模式有效
execution.isolation.semaphore.maxConcurrentRequests | 信号量最大并发度 | 10 | SEMAPHORE模式有效

#### 回退降级参数

参数 | 作用 | 默认值 | 备注
--- | --- | --- | ---
fallback.isolation.semaphore.maxConcurrentRequests | 设置当fallback降级发生时的最大并发度 | 10
fallback.enabled | fallback是否可用 | true | 

#### 熔断参数

参数 | 作用 | 默认值 | 备注
--- | --- | --- | ---
circuitBreaker.enabled | 是否开启熔断 | true    
circuitBreaker.requestVolumeThreshold | 一个统计窗口内熔断触发的最小个数/10s | 20 | 
circuitBreaker.sleepWindowInMilliseconds | 熔断多少秒后去尝试请求 | 5000ms | 
circuitBreaker.errorThresholdPercentage  | 失败率达到多少百分比后熔断 | 50 | 主要根据依赖重要性进行调整
circuitBreaker.forceOpen | 是否强制开启熔断 | |
circuitBreaker.forceClosed | 是否强制关闭熔断 | |

#### Metrics参数

参数 | 作用 | 默认值 | 备注
--- | --- | --- | ---
metrics.rollingStats.timeInMilliseconds | 设置统计滚动窗口的长度，以毫秒为单位 |   10000 | 滚动窗口被分隔成桶(bucket)，并且进行滚动。 例如这个属性设置10s(10000)，一个桶是1s    
metrics.rollingStats.numBuckets | 设置统计窗口的桶数量 | 10 | metrics.rollingStats.timeInMilliseconds必须能被这个值整除
metrics.rollingPercentile.enabled | 设置执行时间是否被跟踪，并且计算各个百分比，50%,90%等的时间 | true | 
metrics.rollingPercentile.timeInMilliseconds | 设置执行时间在滚动窗口中保留时间，用来计算百分比 | 60000ms | 
metrics.rollingPercentile.numBuckets | 设置rollingPercentile窗口的桶数量 | 6 | metrics.rollingPercentile.timeInMilliseconds必须能被这个值整除
metrics.rollingPercentile.bucketSize | 此属性设置每个桶保存的执行时间的最大值 | 100 | 如果设置为100，但是有500次求情，则只会计算最近的100次
metrics.healthSnapshot.intervalInMilliseconds | 采样时间间隔 | 500 | 

#### Request Context参数（设置HystrixCommand使用的HystrixRequestContext相关的属性）

参数 | 作用 | 默认值 | 备注
--- | --- | --- | ---
requestCache.enabled | 设置是否缓存请求，request-scope内缓存 | true |  
requestLog.enabled | 设置HystrixCommand执行和事件是否打印到HystrixRequestLog中 | |

#### ThreadPool Properties参数

参数 | 作用 | 默认值 | 备注
--- | --- | --- | ---
coreSize | 设置线程池的core size,这是最大的并发执行数量 | 10 | 设置标准：coreSize = requests per second at peak when healthy × 99th percentile latency in seconds + some breathing room 大多数情况下默认的10个线程都是值得建议的
maxQueueSize | 最大队列长度。设置BlockingQueue的最大长度 | -1 | 如果使用正数，建议将队列从SynchronousQueue改为LinkedBlockingQueue
queueSizeRejectionThreshold | 设置拒绝请求的临界值 | 5 | 此属性不适用于maxQueueSize = - 1时 设置设个值的原因是maxQueueSize值运行时不能改变，我们可以通过修改这个变量动态修改允许排队的长度
keepAliveTimeMinutes | 设置keep-live时间 | 1分钟 | 

### 启动之后无法修改生效的参数配置

- `metrics.rollingStats.timeInMilliseconds`
- `metrics.rollingStats.numBuckets`
- `metrics.rollingPercentile.timeInMilliseconds`
- `metrics.rollingPercentile.numBuckets`
- `metrics.rollingPercentile.bucketSize`


## 附录

### 参考文档

- [官方文档：Hystrix文档-如何使用](https://github.com/Netflix/Hystrix/wiki/How-To-Use)
- [Hystrix参数详解](http://tietang.wang/2016/02/25/hystrix/Hystrix%E5%8F%82%E6%95%B0%E8%AF%A6%E8%A7%A3/)
- [使用Hystrix实现自动降级与依赖隔离](http://www.jianshu.com/p/138f92aa83dc)
- [Hystrix 那些事](https://juejin.im/entry/58d4d8f5b123db3f6b6485ec)
- [弹性应用的开发利器Hystrix](https://www.gitbook.com/book/stonetingxin/hystrix)
- [Hystrix 使用与分析](http://hot66hot.iteye.com/blog/2155036)
- [Hystrix使用入门手册（中文）](http://www.jianshu.com/p/b9af028efebb)
- [【翻译】Hystrix文档-如何使用](http://youdang.github.io/2016/08/15/translate-hystrix-wiki-how-to-use/#request-collapsing)


