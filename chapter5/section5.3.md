# 5.3 Spring Cloud Ribbon进阶

### 5.3.1 核心工作原理

1. 上层接口
2. Ribbon从启动到负载均衡器选择服务实例的过程


- Ribbon中的核心接口,Ribbon骨架，基于这些接口建立Ribbon负载均衡

| 接口                     | 描述                                                      | 默认实现类                     |
|:-------------------------|:----------------------------------------------------------|:-------------------------------|
| IClientConfig            | 定义Ribbon中管理配置的接口                                | DefaultClientConfigImpl        |
| IRule                    | 定义Ribbon中负载均衡器策略的接口                          | ZoneAvoidanceRule              |
| IPing                    | 定义定期ping服务检查可用性的接口                          | DummyPing                      |
| ServerList<Server>       | 定义获取服务列表接口的方法                                | ConfigurationBasedServerList   |
| ServerListFilter<Server> | 定义特定期望获取服务列表接口的方法                        | ZonePreferenceServerListFilter |
| ILoadBalancer            | 定义负载均衡选择服务的核心方法的接口                      | ZoneAwareLoadBalancer          |
| ServerListUpdater        | 为DynamicServerListLoadBalancer定义动态更新服务列表的接口 | PollingServerListUpdater       |



- 如何利用RestTemplate达到负载均衡的：RestTemplate的Bean与@LoadBalanced注解
- @LoadBalanced:标记一个RestTemplate来使用LoadBalancerClient
- ServiceInstance choose(String serviceId):根据serviceId结合负载均衡器选择一个服务实例
- <T> T execute(String serviceId, LoadBalancerRequest<T> request) 使用来自LoadBalancer的ServiceInstance为指定的服务执行请求
- <T> T execute(String serviceId, ServiceInstance serviceInstance, LoadBalancerRequest<T> request)：使用来自LoadBalancer的ServiceInstance为指定的服务执行请求，是上一个方法的重载，前一个方法的细节实现
- URI reconstructURI(ServiceInstance instance, URI original)：使用主机ip和port构建特定的URI以供Ribbon内部使用。Ribbon使用具有逻辑服务名词的URI作为host，例如`http://myservice/path/to/service`
- 初始化：LoadBalancerAutoConfiguration Ribbon功能的核心配置类
- 加载时机：必须有RestTemplate的实例，必须初始化了LoadBalancerClient实现类
- LoadBalancerRequestFactory用于创建LoadBalancedRequest供LoadBalancerInterceptor使用，低版本没有
- LoadBalancerInterceptorConfig中维护了LoadBalancerInterceptor与RestTemplateCustomizer的实例
- LoadBalancerInterceptor：拦截每一次HTTP请求，将请求绑定进行Ribbon负责均衡的生命周期
- RestTemplateCustomizer：为每个RestTemplate绑定LoadBalancerInterceptor拦截器
- 利用ClientHttpRequestInterceptor来对每次Http请求进行拦截，Spring中维护的请求拦截器，实现intercept方法可以使请求进入方法体
- loadBalancer.execute方法来处理请求
- RibbonLoadBalancerClient：首先得到ILoadBalancer，再使用它取得到一个Server，具体服务实例的封装
-  getServer(loadBalancer)发生负载均衡的过程
- ILoadBalancer.chooseServer(Object key)方法
- rule.choose(key)中的rule就是IRule

### 5.3.2 负载均衡策略源码导读

- IRule时定义Ribbon负载均衡策略的父接口，所有策略都是基于它实现的
- 内部一共7种可选的负载均衡策略
- IRule接口一共定义了三个方法：
  - chose方法会加入具体的负载均衡策略逻辑
  - 另外两个与ILoadBalancer关联
  - 调用过程中，Ribbon时通过ILoadBalancer来关联IRule的，ILoadBalancer的chooseServer方法会转换为调用IRule的choose方法
  - 抽象的AbstractLoadBalancerRule实现了这两个方法，从而将ILoadBalancer与IRule关联起来
