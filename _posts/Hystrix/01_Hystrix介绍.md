# Hystrix介绍

<img src="https://netflix.github.com/Hystrix/images/hystrix-logo-tagline-850.png">

## 一、认识Hystrix

### Hystrix是什么？

Hystrix是一个库，它对来自依赖的延迟和故障进行防护和控制，可以做到

- 阻止故障的级联失败
- 允许应用快速失败和迅速恢复
- 回退并优雅降级
- 支持近实时监控、告警和操作控制

Hystrix翻译为中文是豪猪，一种浑身长满刺的动物，充满了防御感，跟我们使用别人的api和服务一样，要谨记墨菲定律（凡是可能出错的事必定会出错）进行防御式编程

```
官方文档 [https://github.com/Netflix/Hystrix/wiki]
```

### Hystrix产生背景

当前互联网应用通常都是比较复杂的分布式架构，复杂架构导致应用往往会依赖很多外部服务，这样不可避免的部分服务会出错。如果一个应用不能对来自依赖的故障进行隔离，那该应用就存在被拖垮的风险。在一个高流量的网站中，某个单一的后端一旦发生延迟，将会在数秒内导致所有应用资源被耗尽

下图是各依赖服务正常情况下的服务调用

<img src="https://raw.githubusercontent.com/wiki/Netflix/Hystrix/images/soa-1-640.png">

下图是部分依赖服务异常情况下的服务调用，可以看到当某个依赖服务调用延迟会导致用户请求被阻塞

<img src="https://github.com/Netflix/Hystrix/wiki/images/soa-2-640.png">

当大流量访问时，如果有依赖服务出现延迟会导致所有应用Server上的资源饱和，无法对外正常提供服务

<img src="https://github.com/Netflix/Hystrix/wiki/images/soa-3-640.png">

```
例如，一个应用依赖了30个外部服务，其中每个依赖服务的可用性为99.99%，则有：
    - 服务可用性：99.99%^30 = 99.7%
    - 每月服务不可用时间：24 * 30 * 0.3% = 2.16小时

随着服务依赖数量的变多，服务不稳定的概率会成指数性提高
```

使用Hystrix后的服务调用情况

<img src="https://raw.githubusercontent.com/wiki/Netflix/Hystrix/images/soa-4-isolation-640.png">


## 二、Hystrix相关设计原则及实现原理

### Hystrix设计原则

- 使用线程保护应用不被单个依赖耗尽资源
- 减轻负载并快速失败，而不是排队
- 在可行的情况下提供回退以保护用户访问失败
- 使用隔离技巧（舱壁、泳道和断路器模式）来限制外部依赖的冲击
- 通过近实时指标，监控和警报优化故障实时发现机制
- 保护整个依赖客户端执行中的故障，而不仅仅网络流量

### Hystrix实现原理

- 使用`HystrixCommand`或者`HystrixObservableCommand`来封装所有外部系统调用，这些调用通常是在单独的一个线程中执行
- 为每个依赖服务设置调用超时调用，可以自定义超时时间，也可以动态调整，通常超时时间设置的比测量的第99.5个百分位的值稍高一些
- 为每个依赖服务维护一个小的线程池，当线程池满了，直接拒绝请求
- 测量记录请求成功数，失败数，超时数和被拒绝数
- 触发断路器，针对某个特定的服务阻止一切请求一段时间（可以手动触发也可以自动触发，自动触发规则是这个依赖服务的错误百分比超过阀值）
- 当请求失败、超时或者短路时执行fallback逻辑
- 监控测量值和准实时配置变更

### Hystrix流程分析

<img src="https://raw.githubusercontent.com/wiki/Netflix/Hystrix/images/hystrix-command-flow-chart.png">

```
流程说明:
1:每次调用创建一个新的HystrixCommand,把依赖调用封装在run()方法中.
2:执行execute()/queue做同步或异步调用.
3:如果支持相应结果缓存并且缓存结果存在，则直接返回现存结果
4:判断熔断器(circuit-breaker)是否打开,如果打开跳到步骤8,进行降级策略,如果关闭进入步骤.
5:判断线程池/队列/信号量是否跑满，如果跑满进入降级步骤8,否则继续后续步骤.
6:调用HystrixCommand的run方法.运行依赖逻辑
6a:依赖逻辑调用超时,进入步骤8.
7:判断逻辑是否调用成功
7a:返回成功调用结果
7b:调用出错，进入步骤8.
7c:计算熔断器状态,所有的运行状态(成功, 失败, 拒绝,超时)上报给熔断器，用于统计从而判断熔断器状态.
8:getFallback()降级逻辑.
  以下四种情况将触发getFallback调用：
 (1):run()方法抛出非HystrixBadRequestException异常。
 (2):run()方法调用超时
 (3):熔断器开启拦截调用
 (4):线程池/队列/信号量是否跑满
8a:没有实现getFallback的Command将直接抛出异常
8b:fallback降级逻辑调用成功直接返回
8c:降级逻辑调用失败抛出异常
9:返回执行成功结果
```
