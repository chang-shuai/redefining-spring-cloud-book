[TOC]

# 1.1 微服务架构概述

## 1.1.1 应用架构的发展

1. 三种软件架构类型
  - 业务架构
  - 应用架构
  - 技术架构

> 业务架构决定应用架构，技术架构支撑应用架构

---

2. 架构发展历程
 - 单体架构
 - 分布式架构
 - SOA架构
 - 微服务架构

--- 

- 单体应用架构
  - 一个Java Web应用程序，包括表现层、业务层、数据访问层；即controller、Service、Dao
  - 优点：易于开发、测试、部署
  - 缺点：灵活度不够、降低系统性能、系统扩展性差


- 分布式架构
  - 传统分布式架构：按照业务垂直切分，每个应用都是单体架构，通过API互相调用
  - 优点：依赖解耦、理解清晰
  - 缺点：进程间调用可靠性降低、实现技术复杂


- 面向服务的SOA架构
  - 是一种软件体系架构，其应用程序的不同组建通过网络上的通信协议向其他组件提供服务或消费服务，也是一种分布式架构
  - 两个主要角色：服务提供者Provider 服务消费者Consumer
  - 优点：
     1. 把模块拆分，使用接口通信、降低模块之间的耦合度
     2. 把项目拆分成若干个子项目，不同团队负责不同的子项目
     3. 增加功能时只需要增加一个子项目，调用其他系统的接口即可
  - 缺点：系统之间使用远程通信，接口开发增加工作量

***

## 1.1.2 微服务架构

- 是一种架构风格，业务功能拆分为多个相互独立的微服务，各个微服务之间松散耦合，通过远程协议进行同步/异步通信，各个微服务可以被独立部署、扩/缩容、升/降级
- 技术选型：
  - SpringCloud：微服务完整解决方案，基于REST/Http，Eureka(AP)、Ribbon、Hystrix、Config、Hystrix+Turbine、Seluth+Zipkin
  - Dubbo：服务治理框架，RPC协议，ZK/Nacos、Nacos、Dubbo+Monitor
  - Motan：服务治理框架，RPC/Hessian2，ZK/Consul
  - MSEC：服务开发运营框架，Protocol buffer、Monitor
  - 其他：grpc/thrift，Etcd，Nginx+Lua、keepalived/HeartBeat、Apollo/Nacos、Kong、ELK、Pinpoint

***

## 1.1.3 微服务解决方案

- 基于SpringCloud的微服务解决方案
  - 方案1：Eureka(服务发现)、服务间调用组件Feign+负载均衡组件Ribbon+熔断器Hystrix（共用组件）、网关组件Zuul(或性能高SpringCloudGateway)、配置中心SpringCloudConfig(或者携程Apollo、阿里Nacos)、全链路监控Skywalking（或者Pinpoint、zikpin)、搭配组件：分布式事物+容器化+SpringCloud与DDD+gRPC
  - 方案2：Consul(服务发现)，其他组件同方案1
  - 方案3：etcd或阿里Nacos(服务发现)、网关为自研发中间件，其他同方案1

- 基于Dubbo实现微服务解决方案
  - 2012年 阿里巴巴推出 分布式服务治理框架；专注于RPC领域
  - Nacos，动态服务发现、配置、服务管理平台
  -  Dubbo+Nacos+其他
