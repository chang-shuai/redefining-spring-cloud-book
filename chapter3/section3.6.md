# 3.6 Eureka故障演练

###  概述

- Chaos Engineering：混沌工程，通过一系列可控的实验模拟真实世界可能出现的故障，来暴露系统的缺陷，从而驱动工程师取构建更具有弹性的服务
- 常见的手段有：随机关闭依赖服务、模拟增加依赖服务的延时、模拟机器故障、网络中断

### 3.6.1 EurekaServer不可用

> 假设 EurekaServer 全部挂掉，即同时挂掉

1. 应用服务启动前不可用：
  - 应用可以正常启动，但是会有报错信息
  - EurekaServer设计了一个eurkea.client.backup-registry-impl属性，以配置在启动时EurekaServer访问不到的情况下，从这个back registry读取服务注册信息，作为fallback
  - 该backup-registry-impl比较适合服务端提供负载均衡或者服务ip地址相对固定的场景
2. 应用服务运行时不可用
  - EurekaClient在本地内存中有个AtomicReference<Applications>类型的localRegionApps变量
  - 来维护从EurekaServer拉取回来的注册信息
  - Client端有个定时任务CacheRefreshThread会定时从Server端拉取注册信息更新到本地
  - 如果EurekaServer在应用服务运行时挂掉的话，本地的CacheRefreshThread会抛出异常，本地的localRegionApps变量不会得到更新

### 3.6.2 EurekaServer 部分不可用

1. Client端
  - Client端有个定时任务AsyncResolver.updateTask取拉取serverUrl的变更
  - 如果配置文件有改动，运行时可以动态变更，拉取完之后，Client端会随机化Server的list
  - Client端在请求Server的时候，维护了一个不可用的Eureka Server列表(quarabtineSet,在Connection error或者5xx的情况下会被列入该列表，当该列表的大小超过指定阀值则会重新清空）
  - 对可用的Server列表（一般为拉取回来的Server列表剔除不可用的列表，如果剔除之后为空，则不会做剔除处理）
  - 采用RetryableEurekaHttpClient进行请求，numberOfRetries为3
  - 也就是说 如果EurekaServer有一台挂掉，则会被纳入不可用列表，那么此时服务注册信息来自健康的EurekaServer
2. Server端
  - EurekaServer之间时peer node，如果一台挂掉，则EurekaServer之间的replication会受影响
  - PeerEurekaNodes有个定时任务peersUpdateTask会去从配置文件拉取availabilityZones即serviceUrl信息
  - 然后在运行时更新peerEurekaNodes信息
  - 如果其中一台挂掉，人工介入更改EurekaServer的serviceURL信息，则可以主动剔除挂掉的peerNode
  - 如果未能及时剔除，则会报错
  - 目前没有对peerEurekaNodes进行健康检查

### 3.6.3 Eureka高可用原理

- Region：默认下Region之间不会复制，不过Eureka提供了fetch-remote-regions-registry配置，这个配置在dataCenterInfo时Amazon时才生效，其作用时拉取远程Region的注册信息到本地
- AvailabilityZone，默认EurekeClient的eureka.client.prefer-same-zone-eureka配置为true，也就是在拉取serviceURL的时候，优先选择与应用实例同处于一个zone的EurekaServer列表
- 如果client.prefer-same-zone-eureka为false，则默认返回ZoneOffset为0，即取availZones的第一个zone

1. Client端高可用
  - 在Client端启动之前，如果没有EurekaServer，则可以通过配置backup-registry-impl从备份registry读取关键服务的信息
  - 在client启动之后，若运行时出现EurekaServer全部挂掉的情况，本地内存有localRegion之前获取的数据；
  - 在EurkeaServer都不可用的情况下，从Server端定时拉取注册信息回来更新线程CacheRefereshThread会执行失败，本地LocalREgion变量信息不会被更新
  - 在client启动之后，若运行时出现EurekaServer出现部分挂掉的情况，可以人工介入，通过配置文件剔除挂掉的EurekaServer地址
  - Client端会定时刷新serviceUrl
  - 因为Client端维护了一个EurekaServer的不可用列表，一旦请求发生了Connections error或者5xx的情况会被列入该列表
  - 当列表的大小超过指定阀值则会重新清空
  - 在重新清空的情况下，Client默认采用RetryableEurekaHttpClient进行请求，numberOfRetries为3
2. Server端高可用
  - 采用peer to peer的架构模式，因而无所谓的server端的高可用，主要高可用在client端处理
  - server端支持了跨region基本的高可用，可以通过配置remoteRegionUrlsWithName来支持拉取远程region的实例信息
  - 如果当前region要访问的服务实例挂了，那么server端就会fallback到远程region的该服务实例
  - SELF PRESERVATION机制，引入一个阀值，如果最近一分钟接收到的续约的次数小与指定阀值的话，则关闭租约失效剔除，禁止定时任务剔除失效的实例，从而保护注册信息
