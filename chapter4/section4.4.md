# 4.4 venus-cloud-feign设计与使用

### 4.4.1 venus-cloud-feign设计

- SpringMVC不支持实现接口中方法参数上的注解(支持继承类，方法上的注解)
- Get请求多参数传递的问题


1. Spring MVC Controller 中的方法不支持继承实现Feign接口中方法参数上的注解问题
- cn.springcloud.feign.VenusFeignAutoConfig
- cn.springcloud.feign.VenusSpringMvcContract

2. Feign不支持GET方法传递POJO的问题
- 方法一：把POJO拆散成一个一个单独的属性放在方法参数里传输
- 方法二：把方法参数变成Map传输，
- 方法三：违反Restful规范，Get传递@RequestBody
- 方法四：Feign拦截器(cn.springcloud.feign.VenusRequestInterceptor)


### 4.4.2 venus-cloud-feign的使用

- 通过Feign编写服务提供者，并将服务注册到EurekaServer上
```xml
<dependency>
    <groupId>cn.springcloud.feign</groupId>
    <artifactId>venus-cloud-starter-feign</artifactId>
    <version>1.0.0</version>
</dependency>
```
- 创建provider-client，定义服务之间调用的Feign接口
```java
@FeignClient(name = "provider")
public interface UserFeignService {
    @RequestMapping(value = "/user/add", method = RequestMethod.GET)
    String addUser(User user);

    @RequestMapping(value = "/user/update", method = RequestMethod.POST)
    String updateUser(@RequestBody User user);
}
```

- 服务提供者实现工程,编写Feign接口对应的服务提供者实现

```java
@RestController
public class UserController implements UserFeignService {


    @Override
    public String addUser(User user) {
        return "hello," + user.getName();
    }

    @Override
    public String updateUser(User user) {
        return "hello," + user.getName();
    }
}
```

2. 创建服务消费者工程

- 依赖注入服务提供着提供的接口，进行服务间调用

```java
@RestController
public class UserController {
    @Autowired
    private UserFeignService userFeignService;

    @RequestMapping(value = "/add", method = RequestMethod.POST)
    public String addUser(@RequestBody
                          @ApiParam(name = "用户", value = "传入json格式", required = true) User user) {
        return this.userFeignService.addUser(user);
    }

    @RequestMapping(value = "/update", method = RequestMethod.POST)
    public String updateUser(@RequestBody
                          @ApiParam(name = "用户", value = "传入json格式", required = true) User user) {
        return this.userFeignService.updateUser(user);
    }
}
```
