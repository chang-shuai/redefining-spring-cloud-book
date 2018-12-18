# 11.1 Spring Cloud Config配置中心概述

### 1 什么是配置中心

1. 由来
- 被用作集中管理不同环境(Dev Test Stage Prod)和不同集群配置
- 在修改配之后将实时动态推送到应用上进行刷新


2. 概况
- Spring Cloud Config
- netflix archaius
- disconf
- ctrip apollo

3. 应具备的功能
- Open API
- 业务无关性
- 配置生效监控
- 一致性K、V存储
- 统一配置实时推送
- 配合灰度更新与回滚
- 配置全局恢复、备份与历史
- 高可用集群

* 运维管理体系
* 开发管理体系

### 2. Spring Cloud Config

1. 概述
- 集中化外部配置的分布式系统，由服务端和客户端组成
- 不依赖于注册中心，是一个独立的配置中心
- 支持多种存储配置信息的形式，主要有jdbc、Vault、Native、svn、git，默认为git

2. Git版工作原理
- 配置客户端启动时向服务端发起请求，服务端接收到客户端的请求后，根据配置的仓库地址，将git上的文件克隆到本地的一个临时目录，这个目录时一个git的本地仓库目录
- 服务端再读取本地文件返回给客户端
- 好处：当git服务器故障或者网络请求异常时，保证服务端仍然可以正常工作

### 3. 入门案例
1. ConfigServer配置
- @EnableConfigServer 开启Config服务功能
- spring-cloud-config-server
```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri:
          search-paths: # 搜索目录下所有满足条件的配置文件，可以根据需要添加多个子目录，目录之间用逗号隔开
          username:
          password:
  application:
    name: sc-config

```

```
/{application}/{profile}/{label}
/{application}-{profile}.yml
{label}/{application}-{profile}.yml
/{application}-{profile}.properties
{label}/{application}-{profile}.properties

# application应用名，可以理解成git上面的文件名
# profile指的是对于激活的环境名，例如dev、test、prod
# label指的是git的分支,如果不写默认为master
```
> config-server会在本地的临时目录下克隆远程仓库中的配置信息

2.Config Client配置

- spring-cloud-config-client
```yml
spring:
  cloud:
    config:
      label: master
      uri: http://localhost:9090
      name: config-info
      profile: dev
```
- label代表要请求那个git分支
- uri代表请求的config server的地址
- name代表要请求哪个名称的远程文件，可以写多个，通过逗号分隔
- profile代表哪个分支的文件，例如dev、test、prod等

> bootstrap.yml优先于application.yml加载，因此会去加载远程的配置文件信息
> 此时配置修改生效还需要重启
