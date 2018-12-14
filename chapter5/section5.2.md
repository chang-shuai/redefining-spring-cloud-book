# 5.2 Spring Cloud Ribbon实战

### 5.2.1 Ribbon负载均衡策略与自定义配置

- Nginx：轮询RoundRibbon 权重weight ip_hash
- Ribbon负载均衡策略

|          策略类           |       命名       |                                                                       描述                                                                        |
|:-------------------------|:----------------|:-------------------------------------------------------------------------------------------------------------------------------------------------|
|        RandomRule         |     随机策略     |                                                                  随机选择server                                                                   |
|      RoundRobinRule       |     轮询策略     |                                                               按顺序循环选择server                                                                |
|         RetryRule         |     重试策略     |                                       在一个配置时间段内当选择server不成功，则一直尝试选择一个可用的server                                        |
|     BestAvailableRule     |   最低并发策略   |                                  逐个考察server，如果server断路器打不开，则忽略，再选择其中并发连接最低的server                                   |
| AvailabilityFilteringRule |   可用过滤策略   |                过滤掉一直连接失败并被标记为circuit tripped的server，过滤掉哪些高并发连接的server(active connections超过配置的阀值)                |
| ResponseTimeWeightedRule  | 响应时间加权策略 | 根据server的响应时间分配权重，响应时间越长，权重越低，被选择到的概率就越低；响应时间越短，权重越高，被选择的概率就越高 ， 综合考虑的 网络 磁盘 io |
|     ZoneAvoidanceRule     |   区域权衡策略   |         综合判断server所在区域的性能和server可用性轮询选择server，并且判定一个AWS Zone的运行性能是否可用，剔除不可用的Zone中的所有server          |

1. 全局策略设置
> 凡是通过Ribbon的请求都会按照配置的规则来进行
```java
@Configuration
public class TestConfig {
    @Bean
    public IRule ribbonRule() {
        return new RandomRule();
    }
}
```

2. 基于注释的策略设置：针对某一个源服务设置其特有的策略，使用@RibbonClient注解
```java
@Configuration
@AvoidScan
public class TestConfig {
    @Autowired
    IClientConfig iClientConfig;

    @Bean
    public IRule ribbonRule(IClientConfig iClientConfig) {
        return new RandomRule();
    }
}
```

```java
@SpringBootApplication
@RibbonClient(name = "client-a", configuration = TestConfig.class)
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION, value = {AvoidScan.class})})
public class RibbonApplication {
    public static void main(String[] args) {
        SpringApplication.run(RibbonApplication.class, args);
    }

    @Bean
    @LoadBalanced
    public RestTemplate template() {
        return new RestTemplate();
    }
}
```

> 对client-a服务使用的策略是经过TestConfig锁配置的，@ComponentScan让Spring不去扫描被@AvoidScan注解标记的配置类；配置是对单个源服务生效的，必须排除

- 也可以使用@RibbonClients注解对多个源头服务及进行策略指定

```java
@RibbonClients(value = {
    @RibbonClient(name = "client-a", configuration = TestConfig.class),
    @RibbonClient(name = "client-b", configuration = TestConfig.class)
})
```

3. 基于配置文件的策略设置

- 基本语法 <client-name>.ribbon.*
- 对client-a服务使用随机策略

```yml
client-a:
  ribbon:
    NFLoadBalanceRuleClassName: com.netflix.loadbalancer.RandomRule
```

### 5.2.2 Ribbon超时与重试

- 时限控制以及时限之后的重试，F版中Ribbon的重试机制默认是开启的，需要添加对于超时时间与重试策略的配置

```yml
client-a:
  ribbon:
    ConnectTimeout: 30000
    ReadTimeout:  30000
    MaxAutoRetries: 1 #对第一次请求的服务的重试次数
    MaxAutoRetriesNextServer: 1 #要重试下一个服务的最大数量(不包括第一个服务)
    OkToRetryOnAllOperation:  true
```

### 5.2.3 Ribbon的饥饿加载

- Ribbon在进行客户端负载均衡的时候并不是在启动时就加载上下文，而是在实际请求的时候才去创建
- 第一次调用颇为乏力，引起调用超时
- 通过指定Ribbon具体的客户端的名称来开启饥饿加载，即在启动时加载所有配置项的应用程序上下文

```yml
client-a:
  ribbon:
    enabled:  true
    clients:  client-a, cleint-b, client-c

```

### 5.2.4 利用配置文件自定义Ribbon客户端

- 实质也就是使用配置文件来指定一些默认加载类，从而更改客户端的行为方式
- 这种方式优先级最高，高于使用注解@RibbonClient指定的配置和源码中加载的相关Bean

| 配置项                                            | 说明                    |
|:--------------------------------------------------|:------------------------|
| <clientName>.ribbon.NFLoadBalanceClassName        | 指定ILoadBalancer实现类 |
| <clientName>.ribbon.NFLoadBalanceRuleClassName    | 指定IRule实现类         |
| <clientName>.ribbon.NFLoadBalancePingClassName    | 指定IPing实现           |
| <clientName>.ribbon.NIWSServerListClassName       | 指定ServerList实现      |
| <clientName>.ribbon.NIWSServerLisFiltertClassName | 指定ServerFilter实现    |

- 可以使用Ribbon自带实现类，也可以自己实现

```yml
client:
  ribbon:
    NIWSServerListClassName:  com.netflix.loadbalancer.ConfigurationBaseServerList
    NFLoadBalanceRuleClassName: com.netflix.loadbalancer.WeightResponseTimeRule

```

### 5.2.5 Ribbon脱离Eureka的使用

- 默认下，Ribbon客户端从Eureka注册中心读取服务注册信息列表，来达到一种动态负载均衡功能
- 不推荐这种方式情况：Eureka时一个供很多人使用的公共注册中心，此时容易产生服务侵入性问题，避免从Eureka中读取服务列表，应该Ribbon客户端自行指定源服务地址，让Ribbon脱离Eureka来使用
- 在Ribbon中禁用Eureka功能

```yml
ribbon:
  eureka:
    emabled:  false

client:
  ribbon:
    listOfServers:  http://localhost:7070, http://localhost:7071  # 源服务设定地址列表

```
