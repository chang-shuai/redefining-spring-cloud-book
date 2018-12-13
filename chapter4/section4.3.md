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


### 4.3.2 Feign的POST和Get的多参数传递

- 如何在GET或者POST情况下，使用Feign进行多参数传递
- 通过Feign拦截器的方式处理
- 通过实现Feign的RequestInterceptor的apply方法来进行统一拦截处理Feign中的GET方法多参数传递的问题
- 解决方案
  - 把POJO拆散成一个一个单独的属性放在方法参数里
  - 把方法参数变成Map传递
  - 使用GET传递@RequestBody，此方式违反Restful规范
  - 通过Feign拦截器方式处理

```java
@Component
public class FeignRequestInterceptor implements RequestInterceptor {

    @Autowired
    public ObjectMapper objectMapper;

    @Override
    public void apply(RequestTemplate template) {
        // feign不支持GET方法传POJO json body转query
        if (template.method().equals("GET") && template.body() != null) {
            try {
                JsonNode jsonNode = objectMapper.readTree(template.body());
                template.body(null);

                Map<String, Collection<String>> queries = new HashMap<>();
                buildQuery(jsonNode, "", queries);
                template.queries(queries);
            } catch (IOException e) {
                // 提示：根据实际项目情况处理
                e.printStackTrace();
            }
        }
    }


    private void buildQuery(JsonNode jsonNode, String path, Map<String, Collection<String>> queries) {
        if (!jsonNode.isContainerNode()) {
            if (jsonNode.isNull()) {
                return;
            }
            Collection<String> values = queries.get(path);
            if (null == values) {
                values = new ArrayList<>();
                queries.put(path, values);
            }

            values.add(jsonNode.asText());
            return;
        }
        if (jsonNode.isArray()) {
            Iterator<JsonNode> it = jsonNode.elements();
            while (it.hasNext()) {
                buildQuery(it.next(), path, queries);
            }
        } else {
            Iterator<Map.Entry<String, JsonNode>> it = jsonNode.fields();
            while (it.hasNext()) {
                Map.Entry<String, JsonNode> entry = it.next();
                if (StringUtils.hasText(path)) {
                    buildQuery(entry.getValue(), path + "." + entry.getKey(), queries);
                } else {
                    buildQuery(entry.getValue(), entry.getKey(), queries);
                }
            }
        }
    }
}

```

### 4.3.3 Feign的文件上传

- feign-form 实现了文件上传所需的Encoder
- 使用feign-form结合Feign实现文件的上传

```xml
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form</artifactId>
    <version>3.0.3</version>
</dependency>

<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form-spring</artifactId>
    <version>3.0.3</version>
</dependency>

```

```java
@Configuration
public class FeignMultipartSupportConfig {
    @Bean
    @Primary
    @Scope("prototype")
    public Encoder multipartFormEncoder() {
        return new SpringFormEncoder();
    }
}

```


```java
@FeignClient(value = "provider", configuration = FeignMultipartSupportConfig.class)
public interface FileUploadService {

    /***
     * 1.produces,consumes必填
     * 2.注意区分@RequestPart和RequestParam，不要将
     * @RequestPart(value = "file") 写成@RequestParam(value = "file")
     * @param file
     * @return
     */
    @RequestMapping(method = RequestMethod.POST, value = "/uploadFile/server",
            produces = {MediaType.APPLICATION_JSON_UTF8_VALUE},
            consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public String fileUpload(@RequestPart(value = "file") MultipartFile file);
}

```

```java
@FeignClient(value = "provider", configuration = FeignMultipartSupportConfig.class)
public interface FileUploadService {

    /***
     * 1.produces,consumes必填
     * 2.注意区分@RequestPart和RequestParam，不要将
     * @RequestPart(value = "file") 写成@RequestParam(value = "file")
     * @param file
     * @return
     */
    @RequestMapping(method = RequestMethod.POST, value = "/uploadFile/server",
            produces = {MediaType.APPLICATION_JSON_UTF8_VALUE},
            consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
     String fileUpload(@RequestPart(value = "file") MultipartFile file);
}


```

```java
@PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    @ApiOperation(value = "文件上传", notes = "请选择文件上传")
    public String imageUpload(@ApiParam(value = "文件上传", required = true) MultipartFile file) {
        return fileUploadService.fileUpload(file);
    }

```

```java
@PostMapping(value = "/uploadFile/server", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public String fileUploadServer(MultipartFile file) {
        return file.getOriginalFilename();
    }

```



### 4.3.4 解决Feign首次请求失败问题

- 当Feign和Ribbon整合了Hystrix之后，可能会出现首次调用失败的问题，出现该问题的原因
  - Hystrix默认超时时间为1秒，如果超过这个时间未做出响应，将会进入fallback代码
  - 由于Bean的装配以及懒加载机制等，Feign首次请求会比较慢，如果请求响应时间大于1秒，会出现请求失败的问题

- 解决方法
  - 将Hystrix的超时时间改为5秒`hystrix.command.default.execution.isolation.thread.timeoutInMillseconds: 5000`
  - 禁用Hystrix超时时间`hystrix.command.default.execution.timeout.enabled: false`
  - 使用Feign的时候直接关闭Hystrix，该方式不推荐使用`feign.hystrix.enabled: false`


### 4.3.5 Feign返回图片流处理方式

- 通过Feign返回图片一般为字节数组
```java
@RequestMapping(value = "createImageCode")
public byte[] createImageCode(@RequestParam("imageKey") String imageKey);
```

- 使用Feign的过程中可以将流转成字节数组传递，但是因为Controller层的返回不能直接返回byte，需要将Feign返回值修改为response

```java
@RequestMapping(value = "createImageCode")
public Response createImageCode(@RequestParam("imageKey") String imageKey);
```

### 4.3.6 Feign调用传递Token

- 防止Feign远程调用Token丢失，向请求头里添加需要传递的Token
- 实现Feign提供的接口RequestInterceptor

```java
/**
 * Feign统一Token拦截器
 */
@Component
public class FeignTokenInterceptor implements RequestInterceptor {
    @Override
    public void apply(RequestTemplate template) {
        if (null == getHttpServletRequest()) {
            return;
        }
        // 将获取Token对应的值往下面传
        template.header("oauthToken", getHeaders(getHttpServletRequest()).get("oauthToken"));
    }

    private HttpServletRequest getHttpServletRequest() {
        return ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
    }

    /**
     * Feign拦截器拦截请求获取Token对应的值
     * @param request
     * @return
     */
    private Map<String, String> getHeaders(HttpServletRequest request) {
        Map<String, String> map = new LinkedHashMap<>();
        Enumeration<String> enumeration = request.getHeaderNames();
        while (enumeration.hasMoreElements()) {
            String key = enumeration.nextElement();
            String value = request.getHeader(key);
            map.put(key, value);
        }
        return map;
    }
}
```
