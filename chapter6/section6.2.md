# 6.2 Spring Cloud Hystrix 实战运用

### 6.2.1 入门实例

- `spring-cloud-starter-netflix-hystrix`
- `@EnableHystrix`

```java
public class UserService implements IUserService {
    @Override
    @HystrixCommand(fallbackMethod = "defaultUser")
    public String getUser(String username) throws Exception {
        if (username.equals("string")) {
            return "this is real user";
        }else {
            throw new Exception();
        }
    }

    /**
     * 出错调用该方法返回友好提示
     * @param username
     * @return
     */
    public String defaultUser(String username) {
        return "The user does not exist in this system";
    }
}
```

### 6.2.2 Feign中使用断路器

- 新版本中Feign已经默认关闭Hystrix
- 通过配置文件打开
```java
feign:
  hystrix:
    enabled: true
```

### 6.2.3 Hystrix Dashbord

- Hystrix Dashbord仪表盘时根据系统一段时间内发生的请其来展示的可视化面板，这些信息时每个HystrixCommand执行过程中的信息，这些信息是一个指标集合和具体的系统运行情况。
- 
