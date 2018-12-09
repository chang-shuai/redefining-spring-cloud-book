# 3.1 Eureka的核心类

### 3.1.1 InstanceInfo

- Eureka:com.netflix.appinfo.InstanceInfo:代表注册的服务实例

|             字段              |                                                              说明                                                              |
|:-----------------------------:|:------------------------------------------------------------------------------------------------------------------------------:|
|          instanceId           |                                                             实例Id                                                             |
|            appName            |                                                             应用名                                                             |
|         appGroupName          |                                                          应用所属群组                                                          |
|            ipAddr             |                                                             ip地址                                                             |
|              sid              |                                                             已废弃                                                             |
|             port              |                                                             端口号                                                             |
|          securePort           |                                                         Https的端口号                                                          |
|          homePageUrl          |                                                       应用实例的首页url                                                        |
|         statusPageUrl         |                                                      应用实例的状态页url                                                       |
|        healthCheckUrl         |                                                     应用实例的健康检查url                                                      |
|     secureHealthCheckUrl      |                                                 应用实例的健康检查的https url                                                  |
|          vipAddress           |                                                           虚拟ip地址                                                           |
|       secureVipAddress        |                                                       https的虚拟ip地址                                                        |
|           countryId           |                                                     已废弃 国家id 默认未US                                                     |
|        dataCenterInfo         |                                             dataCebter信息 Netflix或Amazon或MyOwn                                              |
|           hostName            |                                                            主机名称                                                            |
|            status             |                                       实例状态  UP DOWN STARTING OUT_OF_SERVICE UNKNOWN                                        |
|       overriddenStatus        |                                            外界需要强制覆盖的状态值 默认为 UNKNOWN                                             |
| isCoordinatingDiscoveryServer |                         首先标识是否是discoveryServer，其次标识该discoveryServer是否是响应你的请求实例                         |
|           leaseInfo           |                                                            租约信息                                                            |
|           metadata            |                                                      应用实例的元数据信息                                                      |
|     lastUpdatedTimestamp      |                                                      状态信息最后更新时间                                                      |
|      lastDirtyTimestamp       | 实例信息最新的过期时间，在Client端用于标识该实例信息是否与EurekaServer一致，在Server端则用于多个EurekaServer之间的信息同步处理 |
|          actionType           |                                标识EurekaServer对该实例执行的操作，包括 ADDED MODIFIED DELETED                                 |
|            asgName            |                                                  在AWS的autoscaling group名称                                                  |


### 3.1.2 LeaseInfo：com.netflix.appinfo.LeaseInfo

- 标识应用实例的租约信息

|         字段          |                  说明                  |
|:---------------------:|:--------------------------------------:|
| renewalIntervalInSecs |         Client端续约的时间间隔         |
|    durationInSecs     |    Client端需要设定的租约的有效时长    |
| registrationTimestamp |  Server端设置的该租约的第一次注册时间  |
| lastRenewalTimestamp  | Server端设置的该租约的最后一次续约时间 |
|   evictionTimestamp   |    Server端设置的该租约被剔除的时间    |
|  serviceUpTimestamp   | Server端设置的该服务实例标记为UP的时间 |

### 3.1.3 ServiceInstance：org.springframework.cloud.client.ServiceInstance

- 是Spring Cloud对service discovery的实例信息的抽象接口
- 约定了服务发现的实例应用有哪些通用的信息
- Spring Cloud对该接口的实现类为 org.springframework.cloud.netflix.eureka.serviceregistry.EurekaRegistration
- 该实现类 实例了 Registration（该接口实现了ServiceInstance）同时还实现了Closeable接口，实现了优雅关闭EurekaClient(eurekaClient.shutdown())

|     方法     |       说明       |
|:------------:|:----------------:|
| getServiceId |      服务id      |
|   getHost    |    实例的host    |
|   getPort    |    实例的port    |
|   isSecure   |  是否开启https   |
|    getUri    |  实例的uri地址   |
| getMetadata  | 实例的元数据信息 |
|  getScheme   |   实例的scheme   |

### 3.1.4 InstanceStatus
- com.netflix.appinfo.InstanceInfo.InstanceStatus
- 用于标识服务实例的状态，是一个枚举类型
- OUT_OF_SERVICE暂停服务，即停止接收请求，处于这个状态的服务实例将不会被路由到，常用于升级部署的场景
