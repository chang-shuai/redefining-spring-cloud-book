# 3.5 Eureka实战

### 3.5.1 EurekaServer在线扩容

- 需要依赖配置中心的动态刷新功能，具体就是/actuator/refresh这个endpoint
- config-server时native的profile，因此修改后重启才能生效，如果是使用Git仓库，则无需重其config-server


### 3.5.2 构建Multi Zone Eureka Server

- Gateway路由时对Region是ZoneAffinity的

### 3.5.3 支持Remote Region

- 异地多活自动转移请求
- 跨Region的fallback

### 3.5.4 开启HTTP Basic认证

- Eureka Server
  - Eureka Server引入 spring-boot-starter-security
  - spring-boot-starter-security默认开启了csrf校验，需要禁用

- Eureka Client
  - 需要相应的帐号信息来传递，通过配置文件来指定，结合config-server的加密功能来加密

### 3.5.5 启用https

- 证书生成
  - keytool生成 server和client端证书：keytool -genkeypair
  - 导出证书：keytool -export
  - 将server.crt文件导入client.p12中，使Client端信任Server的证书，输入的是client.p12密钥：keytool -import
  - 将clietn.crt导入server.p12中，使得Server信任Client的证书，输入的是server.p12密钥

- Eureka Server
  - 把生成的server.p12放入到Maven工程的resources目录下，然后指定相关配置
  - 指定server.ssl配置，以及eureka.instance的securePortEnable及eureka.instance.securePort配置
  - https访问

- Eureka Client
  - 把生成的client.p12放入到Maven工程的resources目录下
  - 仅仅开启访问EurekaServer的https配置，通过自定义的eureka.client.ssl.key-store以及eureka.client.ssl.key-store-password两个属性
  - 指定Eureka Client访问EurekaServer的sslContext配置，需要在代码指定DiscoveryClient,DiscoveryClientOptionalArgs

### 3.5.6 Eureka Admin

- SpringCloud中国社区EurekaServer在线服务访问地址：http://eureka.springcloud/cn/
- EurekaServer开源了一个节点监控，服务动态启停的管控平台Eureka Admin
- <url>https://github.com/SpringCloud/eureka-admin</url>
- 通过管控平台的界面对注册的服务实例进行上下线，停止等操作

### 3.5.7 基于metedata路由实例

- 通过metedata属性进行灰度测试或者不宕机升级
- ILoadBalancer接口：在Ribbon中，ILoadBalancer选取Server的逻辑主要有一系列的IRule来实现
- IRule接口：RoundRobinRule，采用轮询调度算法规则来选取Server
- MetadataAwarePredicate：可以基于Guava的Predicate进行过滤，Predicate将Server的metedata跟上下文传递的attribute信息进行匹配，全部匹配才返回true
