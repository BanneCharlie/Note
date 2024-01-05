# OpenFeign

简介 : OpenFeign为声明式Restful客户端,具有负载均衡功能,而且已经丢弃了Ribbon,采用Spring Cloud Loadbalancer作为负载器;

OpenFeign只涉及Consumer,且为伪客户端,不会进行任务处理,通过注解方式实现Restful请求;

OpenFeign的远程调用底层实现为`HttpClient`同时也支持`OkHttp`;

OpenFeign的默认负载均衡采用的Spring Cloud Loadbalancer的`轮询算法`,同时也具有随机策略;

`pom.xml` 导入 Loadbalancer、OpenFeign依赖

```xml
        <!--开启Spring Cloud配置loadbalancer 实现负载均衡,调用服务时使用-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
            <version>4.0.2</version>
        </dependency>

        <!--配置OpenFigen依赖,实现 RESTFUL风格的负载均衡-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>4.0.4</version>
        </dependency>
```

`application.yml` OpenFegin的常用配置

```yml
server:
  port: 8001

spring:
  application:
    name: consumer-dept

  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848

    openfeign:
      client:
        # 默认没有开启OkHttp	 传输方式
        okhttp:
         enabled: false
        # 开启的为 Httpclient   传输方式
        httpclient:
         enabled: true
      
        config:
          # 全局超时设置 连接时间 读取时间 时间单位ms
          default:
            connect-timeout: 1000
            read-timeout: 1000
          # 提供者provider-dept超时时间设置,优先级高于默认
          provider-dept:
            connect-timeout: 500
            read-timeout: 500
            
      # 对请求与响应进行压缩,开启Http压缩功能
      compression:
        request:
          enabled: true
          # 需要压缩的类型
          mime-types: ["text/html","text/xml","application/json"]
          # 压缩的 HTTP 请求的最小大小
          min-request-size: 1024
        response:
          enabled: true
```

`DeptService.java` 定义Feign接口

```java
import ...
// value为提供者名称  path为地址 遵循 Restful风格(提供者也遵循)
@FeignClient(value = "provider-dept",path = "/dept")
public interface DeptService {
    /*查询所有数据*/
    @GetMapping()
    List<Dept> selectAll();
}
```

`UserController.java` 通过Feign接口消费服务

```java
import ..

@RestController
@RequestMapping("/user")
public class UserController {
    // 注入配置好的提供者伪客户端
    @Autowired
    private DeptService deptService;

    @GetMapping()
    public void selectAll(){
        // 调用提供者的方法
        List<Dept> depts = deptService.selectAll();
        for (Dept dept :depts) {
            System.out.println(dept.toString());
        }
    }
}
```

`ConsumerApplication.java` 开启OpenFegin

```java
import ...

@SpringBootApplication
// 开启OpenFegin    
@EnableFeignClients
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(Provider01Application.class, args);
    }
}
```

`Myconfig.java` 自定义负载均衡的策略

```java
import ...

public class Myconfig {

    // 自定义 ReactorLoadBalancer 更改默认负载均衡轮询策略为随机策略
    @Bean
    public ReactorLoadBalancer<ServiceInstance> changeLoadBalancer(Environment e, LoadBalancerClientFactory loadBalancerClientFactory){
        // 获取负载均衡器的名称
        String name = e.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
        // 获取ServiceInstanceListSupplier的提供者
        ObjectProvider<ServiceInstanceListSupplier> 
        lazyProvider =loadBalancerClientFactory.getLazyProvider(name,ServiceInstanceListSupplier.class);        					
        
        return new RandomLoadBalancer(lazyProvider,name);
    }
}
```

`Provider01Application.java` 

```java
import ...

@SpringBootApplication
// 启动类上添加@LoadBalancerClients注解,自定义配置类
@LoadBalancerClients(defaultConfiguration = {Myconfig.class})
@EnableFeignClients
public class Provider01Application {
    public static void main(String[] args) {
        SpringApplication.run(Provider01Application.class, args);
    }
}
```

