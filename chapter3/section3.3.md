# 3.3 Eureka的设计理念

### 3.3.1 概述

- 作为一个注册服务中心，主要解决这几个问题
  1. 服务实例如何注册到服务中心：使用Spring-Cloud-Starter-Netflix-Eureka-Client，基于SpringBoot的自动配置，自动实现服务信息的注册；调用EurekaServer的REST API的register方法，取注册该应用实例的信息
  2. 服务实例如何从服务中心剔除：正常情况下的关闭，客户端使用钩子方法或其他生命周期回调方法取调用EurekaServer的REST API的de-register方法，来删除自身的服务实例信息；另外为了服务实例挂掉，或者其他异常情况没有及时删除自身信息的问题，EurekaServer要求Client进行续约，也就是发送心跳，来证明该服务实例是否还存活，是健康的，是可以调用的。如果租约超过一定时间没有进行续约操作，EurekaServer主动剔除。EurekaServer采用的就是分布式应用里头经典的心跳模式
  3. 服务实例信息的一致性问题：服务实例注册信息如何在这个集群里保持一致性。

### 3.3.2 AP由于CP

- 分布式系统领域的CAP理论：加州大学伯克利分校的EricBrewer教授提出，由痲省理工学院的SethGilbert和NancyLynch进行理论证明

- Consistency：数据一致性；即数据在存在多副本的情况下，由于网络等原因导致数据写入部分副本失败，造成副本之间数据不一致，存在冲突；满足一致性则要求对数据的更新操作成功之后，多副本的数据保持一致
- Availability：在任何时候客户端对集群进行读写操作，请求能够正常响应，即在一定的延时内完成
- PartitionTolerance：分区容忍性；即发生通信故障的时候，整个集群被分割为多个无法相互通信的分区时，集群仍然可用

- Zookeeper：CP 保持强一致，必须先执行sync操作，同步下数据；极端情况下，发生网络分区错误是否，如果leader节点不在non-quorum分区，那么对这个分区上节点的读写请求将会报错，无法满足Availability特性
- Eureka：AP 在实际生产环境中，服务注册即发现中心保留可用及过期的数据总比丢失掉可用的数据好。无法保证强一致性，需要客户端能够支持负载均衡和失败重试，在Eureka中由Ribbon提供这个功能

### 3.3.3 Peer To Peer结构

- 主从复制
  1. Master-Slave模式 即有一个主副本，其他副本为从副本；所有对数据的写操作都提交到主副本，最后再由从主副本更新到其他从副本；同步更新、异步更新、同步及异步混合
  2. 整个系统的瓶颈在于主副本的写操作压力

- 对等复制
  1. 副本之间不分主从，任何副本都可以接收写操作，各个副本之间的数据同步及冲突处理时关键
  2. EurekaServer采用Peer to Peer的复制模式

- 客户端
  1. perferSameZoneEureka：即有多个分区的话，优先选择与应用实例所在分区一样的其他服务的实例，如果没有找到则默认使用defaultZone
  2. 客户端使用quarantineSet维护一个不可用的EurekaServer列表，进行请求的时候，优先从可用的列表中进行选择，如果失败，切换下一个EurekaServer重试，默认重试次数为3
  3. 防止分布不均衡情况，Client端有个定时任务，默认每五分钟执行一次，来刷新并随机化EurekaServer列表

- 服务端
  1. 单个EurekaServer启动的时候，会有一个syncUp操作，通过EurekaClient请求其他EurekaServer节点中的一个节点获取注册的应用实例信息，然后复制到其他peer节点
  2. 进行复制操作，使用HEADER_REPLICATION的http header来将这个请求与普通应用实例的正常请求取分开来
  3. 其他peer节点接收到请求的时候，旧不会再对他的peer节点进行服务操作，从而避免死循环

- 数据复制的冲突问题
  1. lastDirtyTimestamp标识：默认开启SyncWhenTimestampDiffers配置，当lastDirtyTimestamp不为空时，进行相应处理。
    > 如果请求的lastDirtyTimestamp大于Server本地的lastDirtyTimestamp，则表示EurekaServer之间数据出现冲突，返回404，要求应用实例重新进行register操作
    > 如果请求的lastDirtyTimestamp小于Server本地的lastDirtyTimestamp，如果时peer节点的复制请求，表示数据出现冲突，返回409给peer节点，要求其同步自己最新的数据信息
  2. heartbeat
    > 应用实例与Server端的heartbeat也就是renewLease操作来进行数据的最终修复，即如果发现数据冲突，Server返回404，应用实例重新进行register操作

### 3.3.4 Zone及Region设计

- Region代表一个独立的地理区域
- 每个region下面，还分了多个AvailabilityZone
- 一个Region对应多个 AvailabilityZone，每个Region之间时相互独立及隔离的
- 默认情况下资源只有在单个Region之间的AvailabilityZone进行复制
- 跨Region之间不会进行资源复制
- EurekaClient支持perferSameZone，也就是EurekaServer的serviceURL优先拉取跟应用实例同处一个AvailabilityZone的EurekaServer地址列表
- 一个AvailabilityZone可以设置多个EurekaServer实例，它们之间是peer节点，然后采用peer to peer的复制模式
- Netfilx的Ribbon组件针对多个AvailabilityZone提供了ZoneAffinity的支持，允许在客户端路由或网关路由时，优先选择与自身实例处与同一个AvailabilityZone的服务实例

### 3.3.5 SELF PRESERVATION设计

- Eureka Server与Client端 之间有个租约，Clietn端要定时发送心跳来维持这个租约
- Eureka通过当前注册的实例数，取计算每分钟应该从应用实例接收到的心跳数
- 如果最近一分钟接收到的续约的次数小与等于指定阀值的话，则关闭租约的失效剔除，禁止定时任务剔除失效的实例，从而保护注册信息
