# 2.1 服务发现概述

## 2.1.1 服务发现由来

- 服务发现、注册中心、名字服务
- 单体架构时代：配置域名方式访问 appId appKey
- SOA架构时代：Nginx
- 微服务时代：Consul-template+Nginx方案、又拍云slardar

### 2.1.2 Eureka简介

- Eureka Server 和 Eureka Client
- 基于REST提供服务，java编写客户端
- Eureka(服务发现) + Ribbon(负载均衡)


### 2.1.3 服务发现技术选型

- Zookeeper: CP Java
- Doozer: CP Go
- Concul: CP Go
- Etcd: CP or Mixed Go
- SmartStack: AP Rubby
- Eureka: AP Java
- NSQ(looupd): AP Go
- Serf: AP Go
- Spotify(DNS): AP N/A
- SkyDNS: Mixed Go
