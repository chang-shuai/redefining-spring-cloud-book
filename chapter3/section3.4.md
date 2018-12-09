# 3.4 Eureka参数调优和监控

### 3.4.1 核心参数

- Client端：
  - 基本参数
  - 定时任务参数
  - http参数
- Server端
  - 基本参数
  - response cache参数：为了提升自身REST API接口性能，提供了两个缓存，一个时基于ConcurrentMap的readOnlyCacheMap，一个基于GuavaCache的readWriteCacheMap
  - peer相关参数
  - http参数

### 3.4.2 参数调优

- 为什么服务下线了，EurekaServer接口返回的信息还会存在
- 为什么服务上线了，EurekaClient不能及时获取到
- 为什么有时会出现如下提示：RENEW ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEGING EXPIRED JUST TO BE SAFE

- 解决方法
  - Eureka并不是强一致，因此register中会保留过期的实例信息
    > 应用实例异常挂掉，没能在挂掉之前告知EurekaServer要下线该服务实例信息，这个需要依赖EurekaServer的EvictionTask去剔除
    > 应用实例下线时有告知EurekaServer下线，但是由于EurekaServer的REST API有response cache，因此需要等待缓存过期才能更新
    > 由于开启了SELF PRESERVATION模式，导致registry的信息不会因为过期而被剔除，直到退出SELF PRESERVATION模式
  - 针对Clietn下线没有通知EurekaServer问题，可以调整EvictionTask的调度频率
  - 针对Response Cache问题，可以根据情况关闭readOnlyCacheMap；或者调整readWriteCacheMap的过期时间
  - 针对SELF PRESERVATION的问题，在测试环境可以将enable-serf-preservation设置为false
  - 针对新服务上线，EurekaClient获取不及时问题，在测试环境，可以适当提高Client端拉取Server注册信息的频率
  - 针对SELF PRESERVATION问题，eureka.server.enableSelfPreservation = preservation设置为false；在生产环境里，可以把renewalPercentThreshold及leaseRenewalIntervalInSeconds参数调小一点，进而提高触发SELF PRESERVATION机制的门槛

### 3.4.3 指标监控

- Eureka内置了基于servo的指标特性 详见com.netflix.eureka.util.EurekaMonitors
- SpringBoot2.x版本该为Micrometer，不支持Nerflix Servo，转而支持Servo替代着Netflix Spectator
- 对于Servo，可以通过DefaultMonitorsRegistry.getInstance().getRegisteredMonitors()来获取所有注册了的Monitors，进而获取其指标值
