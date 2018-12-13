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
- 
