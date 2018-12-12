# 4.1 Feign概述

### 4.1.1 什么是Feign

- Feign是一个声明式的WebService客户端
- 只需要创建一个接口加上对应的注解，比如FeignClient注解
- 可插拔注解，包括Feign注解和JAX-RS注解；Feign也支持编码器和解码器；可以使用HttpMessageConverters
- Feign是一种声明式、模版化的HTTP客户端
- spring-cloud-openfeign

| 特性                                        |
|:--------------------------------------------|
| 可插拔的注解支持，包括Feign注解和JAX-RS注解 |
| 支持可插拔的HTTP编码器和解码器              |
| 支持Hystrix和它的Fallback                   |
| 支持Ribbon的负载均衡                        |
| 支持HTTP请求和响应的压缩                    |


### 4.1.2 Feign入门案例


### 4.1.3 Feign工作原理

- 在开发微服务应用时，会在主程序入口添加@EnableFeignClients注解开启对Feign Client的扫描加载处理，根据FeignClient的开发规范，定义接口并加@FeignClient注解
- 当程序启动时，会进行包扫描，扫描所有@FeignClient的注解的类，并将这些信息注入SpringIOC容器中。当定义的Feign接口中的方法被调用时，通过JDK的代理的方式，来生成具体的RequestTemplate；当生成代理时，Feign会为每个接口方法创建一个RequestTemplate对象，该对象封装了HTTP请求需要的全部信息，入请求参数名，请求方法等信息都是在这个过程中确定的
- 然后由RequestTemplate生成Request，然后把Request交给Client去处理，这里指的Client可以是JDK原生的URLConnection，Apache的HttpClient，也可以是Okhttp
- 最后Client被封装到LoadBalanceClient类，这个类结合Ribbon负载均衡发起服务之间的调用
