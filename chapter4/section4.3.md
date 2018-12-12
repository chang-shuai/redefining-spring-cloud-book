# 4.3 Feign的实战应用

### 4.3.1 Feign默认Client的替换

- 默认使用JDK原生的URL Connection发送HTTP请求，没有连接池，但是对每个地址会保持一个长连接，即利用http的persistence connection
- 可以用Apache HttpClient替换Feign原始的 HttpClient，通过设置连接池，超时时间等对服务之间的调用调优

1. 使用Apache Http Client替换Feign默认的Client
2. 使用okhttp替换Feign默认的Client
