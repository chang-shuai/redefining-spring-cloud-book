# 5.2 Spring Cloud Ribbon实战

### 5.2.1 Ribbon负载均衡策略与自定义配置

- Nginx：轮询RoundRibbon 权重weight ip_hash
- Ribbon负载均衡celve

|          策略类           |       命名       |                                                                       描述                                                                        |
|:-------------------------|:----------------|:-------------------------------------------------------------------------------------------------------------------------------------------------|
|        RandomRule         |     随机策略     |                                                                  随机选择server                                                                   |
|      RoundRobinRule       |     轮询策略     |                                                               按顺序循环选择server                                                                |
|         RetryRule         |     重试策略     |                                       在一个配置时间段内当选择server不成功，则一直尝试选择一个可用的server                                        |
|     BestAvailableRule     |   最低并发策略   |                                  逐个考察server，如果server断路器打不开，则忽略，再选择其中并发连接最低的server                                   |
| AvailabilityFilteringRule |   可用过滤策略   |                过滤掉一直连接失败并被标记为circuit tripped的server，过滤掉哪些高并发连接的server(active connections超过配置的阀值)                |
| ResponseTimeWeightedRule  | 响应时间加权策略 | 根据server的响应时间分配权重，响应时间越长，权重越低，被选择到的概率就越低；响应时间越短，权重越高，被选择的概率就越高 ， 综合考虑的 网络 磁盘 io |
|     ZoneAvoidanceRule     |   区域权衡策略   |         综合判断server所在区域的性能和server可用性轮询选择server，并且判定一个AWS Zone的运行性能是否可用，剔除不可用的Zone中的所有server          |
