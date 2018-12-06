[TOC]

# 1.2 Spring Cloud与中间件

## 1.2.1 中间件概述

- 三驾马车：中间件、操作系统、数据库
- 本质为技术架构
- 常见中间件：服务治理中间件（Dubbo等RPC框架）、配置中心、全链路监控、分布式事务、分布式定时任务、消息中间件、API网关、分布式缓存、数据库中间件

## 1.2.2 什么是SpringCloud

- 也是一个中间件，基于SpringBoot开发，提供一套完整的微服务解决方案
- 服务注册与发现、配置中心、全链路监控、API网关、熔断器
- 里程碑版本:SpringCloud Finchley版本 2018年6月19日发布

## 1.2.3 SpringCloud项目模块

- 开源项目集合<http://github.com/spring-cloud>
- 版本号管理：按伦敦地铁站的名字，按首字母排序
- 组件列表

| 组件名称 |        所属项目        |     组件分类     |
|:--------:|:----------------------:|:----------------:|
|  Eureka  |  spring-cloud-netflix  |     注册中心     |
|   Zuul   |  spring-cloud-netflix  |    第一代网关    |
| Sidccar  |  spring-cloud-netflix  |      多语言      |
|  Ribbon  |  spring-cloud-netflix  |     负载均衡     |
| Hystrix  |  spring-cloud-netflix  |      熔断器      |
| Turbine  |  spring-cloud-netflix  |     集群监控     |
|  Feign   | spring-cloud-openfeign | 声明式HTTP客户端 |
|  Consul  |  spring-cloud-consul   |     注册中心     |
| Gateway  |  spring-cloud-gateway  |    第二代网关    |
|  Sleuth  |  spring-cloud-sleuth   |     链路追踪     |
|  Config  |  spring-cloud-config   |     配置中心     |
|   Bus    |    spring-cloud-bus    |       总线       |
| Pipeline | spring-cloud-pipelines |     部署管道     |
| Dataflow | spring-cloud-dataflow  |     数据处理     |

## 1.2.4 SpringCloud与服务治理中间件

- 服务治理中间件
  - 服务注册与发现
  - 服务路由：服务上下线、在线测试、机房就近原则、A/B测试、灰度发布
  - 负载均衡：支持根据目标状态和目标权重进行负载均衡
  - 自我保护：服务降级、优雅降级、流量控制
  - 丰富的治理管理机制
- 注册中心：Eureka、Zookeeper、Consul
- 熔断自我保护：Hystrix的Fallback
- 负载均衡：Ribbon

## 1.2.5 SpringCloud与配置中心中间件

- 配置信息抽取出来统一管理
- 配置中心：支持复杂的场景配置、与公司的运维体系和权限管理体系集成、各种配置兼容支持
- spring cloud config：基于应用、环境、版本三个维度管理，配置存储支持Git和其他扩展存储
- 缺点：无可视化管控平台


## 1.2.6 SpringCloud与网关中间件

- API Gateway网关：隔离外部访问和内部系统的作用，企业级的应用防火墙
- 统一接入功能：负载均衡、容灾切换、异地多活
- 协议适配功能：对外请求协议HTTP/HTTP2 后端访问协议REST/RPC
- 流量管控功能：流量管控、调拨；熔断和服务降级、流量切片路由到不同机房
- 安全防护功能：安全防护过滤、IP黑名单、URL黑名单封禁控制；风控防刷、防恶意攻击

1. 第一代网管Zuul：根据配置的路由规则或者默认路由规则进行路由转发和负载均衡；集成Hystrix实现网关层面降级功能、集成Ribbon实现弹性伸缩功能、集成Archaius进行配置管理；最多1000至2000QPS

2. 第二代网关Spring Cloud Gateway：Spring5.0+SpringBoot2.0+ProjectReactor；底层基于Netty实现（多线程reactor模型、使用boss线程和worker线程接收异步处理请求）


## 1.2.7 SpringCloud与全链路监控中间件

- 收集、汇总并分析日志信息，进行可视化展示和监控告警
- 主要功能
  - 定位慢调用：慢web服务、慢REST/RPC服务、慢SQL
  - 定位各种错误：4XX 5XX ServiceError
  - 定位各种异常：Error Exception、Fatal Exception
  - 展现依赖和拓扑：域拓扑、服务拓扑、trace拓扑
  - Trace调用链：将端对端的调用，以及附加在这次调用的上下文信息，异常日志信息、每一个调用点的耗时都呈现给用户进行展示
  - 应用告警：根据运维设定的告警规则，扫描指标数据，将告警信息上报到中央告警平台
 - 产品：京东Hydra、阿里Eagleye、Skywalking、Pinpoint
