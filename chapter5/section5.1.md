# 5.1 Spring Cloud Ribbon概述

### 5.1.1 Ribbon与负载均衡

- 负载均衡Load Balance：即利用特定方式将流量分摊到多个操作单元上的一种手段，对系统吞吐量与系统处理能力有着质的提升
- Nginx LVS 本质流量的疏导
- 软负载与硬负载：代表产品时Nginx与F5
- 集中式负载与进程内负载均衡
- 集中式负载均衡：是指位于因特网与服务提供者之间，并负责把网络请求转发到各个提供单位，Nginx与F5是一类，也称服务端负载均衡
- 进程内负载均衡：是指从一个实例库进行流量导入，在微服务的范畴内，实例库一般存储在Eureka、Consul、Zookeeper、etcd等注册中心，此时的负载均衡器就是类似Ribbon的IPC（Inter-Process Communication 进程间通信）组件，也叫客户端负载均衡
- Ribbon是一个客户端负载均衡器(后端负载均衡、进程内负载均衡)，赋予应用一些支配HTTP与TCP行为的能力；SpringCloud横向扩展必须组件
- Feign、Zuul已默认继承Ribbon

### 5.1.2 入门案例

- spring-cloud-starter-netflix-ribbon

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}

```

- Ribbon默认使用轮询的方式访问源服务，此外Ribbon对服务实例节点的增减能动态感知
