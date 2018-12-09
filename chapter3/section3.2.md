# 3.2 服务的核心操作

### 3.2.1 概述

- 服务注册 register

- 服务下线 cancel

- 服务租约 renew

- 服务剔除 evict

### 3.2.2 LeaseManager

- com.netflix.eureka.lease.LeaseManager
- 接口定义了应用服务实例在服务中心的几个操作

- register：用于注册服务实例信息
- cancel：用于删除服务实例信息
- renew：用于与EurekaServer进行心跳操作，维持租约
- evict：是Server端的一个方法，用于剔除租约过期的服务实例信息

### LookupService

- com.netflix.discovery.shared.LookupService
- 接口定义了EurekaClient从服务中心获取服务实例的查询方法
- 主要给Client端使用


- getApplication(String appName):
- getApplications():获取所有应用实例信息
- getInstancesById(String id)：根据应用id获取所有服务实例
- getNextServerFromEureka(String virtualHostname, boolean secure)：根据virtualHostname使用round-ribbon方式获取下一个服务实例
