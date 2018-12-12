# 4.3 Feign的实战应用

### 4.3.1 Feign默认Client的替换

- 默认使用JDK原生的URL Connection发送HTTP请求，没有连接池，但是对每个地址会保持一个长连接，即利用http的persistence connection
- 可以用Apache HttpClient替换Feign原始的 HttpClient，通过设置连接池，超时时间等对服务之间的调用调优

1. 使用Apache Http Client替换Feign默认的Client

```xnl
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.4</version>
</dependency>

<dependency>
    <groupId>com.netflix.feign</groupId>
    <artifactId>feign-httpclient</artifactId>
    <version>8.17.0</version>
</dependency>

```

```yml
feign:
  httpclient:
    enable: true

```

2. 使用okhttp替换Feign默认的Client

> 支持SPDY 可以合并多个到同一个主机的请求
> 使用连接池技术减少请求的延迟 如果SPDY时可用的话
> 使用GZIP压缩减少传输的数据量
> 缓存响应避免重复的网络请求


```xml
<dependency>
           <groupId>io.github.openfeign</groupId>
           <artifactId>feign-okhttp</artifactId>
       </dependency>
```

```yml

feign:
  httpclient:
    enable: false
  okhttp:
    enabled: true

```

```java

@Configuration
@ConditionalOnClass(Feign.class)
@AutoConfigureBefore(FeignAutoConfiguration.class)
public class FeignOkhttpConfig {
    @Bean
    public okhttp3.OkHttpClient okHttpClient() {
        return new OkHttpClient().newBuilder()
                // 设置连接超时
                .connectTimeout(60, TimeUnit.SECONDS)
                // 设置读超时
                .readTimeout(60, TimeUnit.SECONDS)
                // 设置写超时
                .writeTimeout(60, TimeUnit.SECONDS)
                // 是否自动重连
                .retryOnConnectionFailure(true)
                .connectionPool(new ConnectionPool())
                // 构建okHttpClient对象
                .build();
    }
}

```
