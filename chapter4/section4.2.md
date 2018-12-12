# 4.2 Feign的基础功能

### 4.2.1 FeignClient注解解析

- @Target(ElementType.Type)修饰，表示FeignClient注解的作用目标是在接口上
- 常用属性

| 属性名          | 说明                                                                                                                     |
|:----------------|:-------------------------------------------------------------------------------------------------------------------------|
| name            | 指定FeignClient的名称，如果项目使用了Ribbon，name属性作为微服务的名称，用于服务发现                                      |
| url             | url一般用用于调试，可以手动指定@FeignClient调用的地址                                                                    |
| decode404       | 当发生404错误时，如果该字段为true，会代噢用decoder进行解码，否则抛出FeifnException                                       |
| configuration   | Feign配置类，可以自定义Feign的Encoder，Decoder、LogLevel、Contract                                                       |
| fallback        | 定义容错的处理类，当调用远程接口失败或者超时时，会调用对应接口的容错逻辑，fallback指定的类必须实现@FeignClient标记的接口 |
| fallbackFactory | 工厂类，用于生成fallback类实例，通过这个属性可以实现每个接口通用的容错逻辑，减少重复代码                                 |
| path            | 定义当前FeignClient的统一前缀                                                                                            |


### 4.2.2 Feign开启GZIP压缩

- 支持对请求和响应的GZIP压缩，以提高通信效率
- 两种不同的配置方式
- application.yml
- application.properties
- 由于开启了GZIP压缩之后，Feign之间的调用通过二进制协议进行传输，返回值需要改为ResponseEntity<byte[]>才可以正常显示，否则会导致服务之间的调用结果乱码
-

### 4.2.3 Feign支持属性文件配置

- 对单个指定特定名称的Feign进行配置：@Feign的配置信息可以通过application.properties或application.yml配置

```yml
feign:
  client:
    github-client:
      connectTimeOut: 500
      loggerLevel:  full
      errorDecoder: com.example.SimpleErrorDecoder
      retryer:  com.example.SimpleRetryer
      requestInterceptors:
        - com.example.FooRequestInterceptor
        - com.example.BarRequestInterceptor
      decode404:  false
      encoder:  com.example.SimpleEncoder
      decoder:  com.example.SimpleDecoder
      contract: com.example.SimpleContract

```

- 作用于所有Feign的配置方式：@EnableFeignClients注解有个defaultConfiguration属性，可以将默认配置写成一个类`@EnableFeignClients(defaultConfiguration = DefaultFeignConfiguration.clsss)`

> 也可以使用application.yml来作用于所有Feign也是可以的

```yml
feign:
  client:
    config:
      defalut:
        connectTimeOut: 5000
        readTiemOut:  5000
        loggerLevel:  basic

```

> 属性文件中Feign配置会覆盖Java代码的配置，但是可以配置Feign.client.default-to-properties=false来改变Feign配置生效的优先级

### 4.2.4 Feign Client开启日志

- Feign为每一个FeignClient都提供了一个feign.Logger实例，可以在配置中开启日志
  - 在applicaiton.yml中配置日志输出
  ```yml

    logging:
      level:
        com.cloud.feign.service.FeignService: debug

  ```
  -  通过java代码的方式在主程序入口类中配置日志bean；也可以创建带有@Configuration注解的类，区配置日志Bean

```java
  @Bean
  Logger.Level feignLoggerLevel() {
    return Logger.Level.full;
  }

```

```java

@Configuration
public class HelloFeignServiceConfig {
    /**
     * Logger.Level 的具体级别如下：
     * NONE：不记录任何信息
     * BASIC：仅记录请求方法、URL以及响应状态码和执行时间
     * HEADERS：除了记录 BASIC级别的信息外，还会记录请求和响应的头信息
     * FULL：记录所有请求与响应的明细，包括头信息、请求体、元数据
     *
     * @return
     */
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}


```


### 4.2.5 Feign的超时设置

- Feign调用分两层,即Ribbon的调用和Hystrix的调用
- 高版本的Hystrix默认是关闭
- `feign.RetryableException`：Read time out Ribbon超时，此时配置Ribbon的配置信息

```yml

  # 请求处理的超时时间
  ribbon.ReadTimeout: 120000
  # 请求连接 的超时时间
  ribbon.ConnectTimeout:  30000
```

- 如果开启Hystrix,Hystrix报错`com.netflix.hystrix.HystrixRuntimeException：timedout`
- 设置Hystrix的配置信息

```yml

feign.hystrix.enabled:  true
hystrix:
  shareSecurityContext: true
  command:
    default:
      circuitBreaker:
        sleepWindowInMilliseconds:  100000
        forceClosed:  true
      execution:
        isolation:
          thread:
            thread:
              timeoutInMilliseconds:  600000  

```
