# Info



简介 : Spring Cloud Gateway 提供了一个建立在Spring生态系统之上的API网关,旨在提供一种简单有效的方法来路由到API,并为它们提供跨领域的关注点,可以处理安全、监控、限流、熔断等跨领域的问题;

*Spring Cloud Gateway关键部分 :*

- route 路由 : 网关的最基本组成, 路由id + 目标url + predicate工厂(断言为true,请求经filter被路由到目标url) + filter工厂
- predicate 断言 : 判断当前Http请求是否满足指定的断言规则;满足后路由到指定规则或过滤器链
- filter 过滤器 : 通过断言后,会被路由到设置好的过滤器,对请求或者响应进行处理

![image-20231202143844610](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202143844610.png) 

`pom.xml` 导入 Spring Cloud gateway 依赖

```xml
        <!--配置 SpringCloud Gateway 依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
            <version>4.0.6</version>
        </dependency>
```

`application.yml` 配置式路由

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id:  route-config
          uri: https://www.baidu.com
          # 1.shortcut config predicate
          predicates:
            # 设置所有路径皆可访问
            - Path=/**
            - Cookie=mycookie,mycookievlaue
          
          # 2.fully expanded config predicate 
            - Path=/**
            # 仅针对 key [filed - value]
            - name: Cookie
              args:
               name: mycookie
               regexp: mycookievlaue
          
```

![image-20231202151859708](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202151859708.png) 

`application.yml` Api式路由

```yml
server:
  port: 9002

spring:
  application:
    name: gateway-api
```

`GateWayApi.java` config配置类

```java
import ..

@Configuration
public class GatewayApi {

    @Bean
    public RouteLocator testRouterLocator(RouteLocatorBuilder builder){
        return builder.routes()
                // 链式编程 设置route的id  断言predicate 目标路径 uri
                .route("gateway-provider",
                        predicateSpec -> predicateSpec
                                .path("/**")
                                .uri("https://www.tencent.com"))
                .build();
    }
}
```

![image-20231202152314202](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202152314202.png)

# Custom Error Attribute

![image-20231202204635045](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202204635045.png)

```java
import ...

@Component
// 更换系统的异常信息,自定义一个DefaultErrorAttribute的子类,重写getErrorAttributes()方法即可;
public class CustomErrorAttributes extends DefaultErrorAttributes {
    @Override
    public Map<String, Object> getErrorAttributes(ServerRequest request, ErrorAttributeOptions options) {
        HashMap<String, Object> map = new HashMap<>();
        // map可设置的key值为 exception trace message errors
        map.put("status", HttpStatus.NOT_FOUND.value());
        map.put("message","没有找到匹配的信息");
        return map;
    }
}
```

# Predicate

简介 : Spring Cloud Gateway将路由匹配为最基本的功能;

该功能通过路由断言工厂完成,SpringCloud Gateway包含了许多内置的路由断言工厂,可以匹配HTTP请求的不同属性,而且

可以根据逻辑与状态,进行复用;

## Time route predicate 

简介 : 该断言工厂的参数对应的为UTC格式的时间,将请求访问到GateWay的时间与该参数进行比较;

- After    请求时间在参数时间之后
- Before  请求时间在参数时间之前
- Between 请求时间在参数时间之间

`application.yml` 

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      # 默认第一个优先级别最高
      routes:
        - id:  route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置Before断言工厂 
            - Before=2024-01-20T17:42:47.789-07:00
           
           
        - id:  route-config2
          uri: https://www.douyin.com
          predicates:
            # 设置After断言工厂 
            - After=2017-01-20T17:42:47.789-07:00
             
        - id: route-config3
          uri: https://www.huohu.com
          predicates:
            # 设置Between断言工厂 
            - Between=2018-01-20T17:42:47.789-07:00,2024-01-20T17:42:47.789-07:00
```

![image-20231202161626307](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202161626307.png) 

## Cookie route predicate 

简介 : Cookie断言工厂包含两个参数,分别为cookie的 key 和 vlaue;请求头中具有指定的key 与 value 的cookie,匹配成功;

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id:  route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置Cookie断言工厂 Headers的Cookie下  key为 city value为 Shanghai
            - Cookie=city,Shanghai
```

![image-20231202162433088](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202162433088.png) 

## Header route predicate 

简介 : Header断言工厂中包含两个参数,分别为请求头的 key 与 value;请求中带有指定的key与value的header,匹配成功;

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id:  route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置Header断言工厂 Headers下 key为 X-Request-Id value为 \d+ 1~n个数字
            - Header=X-Request-Id,\d+
```

![image-20231202163303690](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202163303690.png) 

## Host route predicate 

简介 : Host断言工厂包含参数为请求头中的Host属性;请求中带有指定的host属性,匹配成功;

`修改C:\Windows\System32\drivers\etc中的hosts文件`

![image-20231202164414745](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202164414745.png) 

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id:  route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置Host断言工厂 必须携带对应的端口号
             - Host=bannelocalhost1:9001,bannelocalhost2:9001,bannelocalhost3:9001
```

![image-20231202165044358](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202165044358.png) 

## Method route predicate 

简介 : Method断言工厂判断请求是否使用了指定的请求方法,POST GET PUT等

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id:  route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置Method断言工厂 符合一个即可
             - Method=GET,POST
```

![image-20231202165647835](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202165647835.png) 

## Path route predicate

简介 : Path断言工厂中具有指定的路径,请求中包含指定的路径,匹配成功;将匹配的uri拼接到目标uri进行跳转;

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id:  route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置Path断言工厂  符合后uri会拼接到目标uri进行跳转
             - Path=/**
```

![image-20231202171209327](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202171209327.png) 

## Query route predicate

简介 : Query断言工厂设置请求参数,可以仅查看参数名称,也可以同时查看参数名和参数值;请求中包含指定的参数名或者参数值,匹配成功;

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id:  route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置Query断言工厂  参数必须同时包含 color(必须具有参数值且满足正则) 和 size(参数值可有可无)
             - Query=color,gr.+
             - Query=size
```

![image-20231202183243038](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202183243038.png) 

## RemoteAddr route predicate 

简介 : RemoteAddr断言工厂设置请求客户端的IP地址范围或者列表;请求的IP地址满足IP地址范围和IP列表,匹配成功;

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id:  route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置RemoveAddr断言工厂 IP的范围为 192.168.238.1~254
             - RemoteAddr=192.168.238.1/24
```

![image-20231202191046716](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202191046716.png) 

## Weight route predicate 

简介 : Weight断言工厂用于对同一组中不同uri实现指定权重的负载均衡,路由中包含两个参数,分别为group、weight;

对于同一组多个uri地址,路由器会根据设置的权重实现负载均衡;

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id:  route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置Weight断言工厂 参数为 组别(banne) 和 权重(8),实现负载均衡 80%
            - Weight=banne,8

        - id: route-config2
          uri: https://www.douyin.com
          predicates:
            # 设置Weight断言工厂 参数为 组别(banne) 和 权重(2),实现负载均衡 20%
            - Weight=banne,2
```

## XForwardRemoteAddr route predicate

简介 : XForwardRemoteAddr断言工厂设置XForwardRemoteAddr的IP列表;请求头中带有X-Forwarded-For的IP出现在路由指定的 IP 列表中,则匹配成功;

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id:  route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置XForwardedRemoteAddr断言工厂 参数为IP列表 1~254
            - XForwardedRemoteAddr=192.168.128.1/24
```

![image-20231202190738526](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202190738526.png) 

## Custom route predicate 

简介 : 自定义predicate路由工厂,继承AbstractRoutePredicateFactory<对应的配置类>类重写apply方法实现自定义断言功能;

### Token

```java
import ...

@Component
/*自定义路由断言工厂 名称为 Token(功能前缀) + RoutePredicateFactory,前缀名称将用在配置文件中;
* 请求中具有一个token请求参数,且包含参数值 参数名或值与Token断言工厂指定的相同匹配成功*/
public class TokenRoutePredicateFactory extends AbstractRoutePredicateFactory<TokenRoutePredicateFactory.Config> {

    // 定义对应配置文件Token的参数名称
    @Data
    public static class Config{
        private String token;
    }

    // 定义配置文件中参数的顺序
    @Override
    public List<String> shortcutFieldOrder() {
        return Collections.singletonList("token");
    }
    // 向父类传入配置类
    public TokenRoutePredicateFactory(){
        super(Config.class);
    }
    // 覆盖apply方法,返回Predicate对象; 重写test方法(会被SpringCloudGateway调用)
    public Predicate<ServerWebExchange> apply(Config config){
        return serverWebExchange -> {
            // 1.获取请求中的参数列表
            MultiValueMap<String, String> map = serverWebExchange.getRequest().getQueryParams();
            // 2.得到参数名称token对应的参数值
            List<String> values = map.get("token");
            // 3.判断请求中token的参数值是否与配置文件设置的参数值一致
            if (values.contains(config.getToken())){
                return true;
            }
            return false;
        };
    }
}
```

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id: route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置自定义断言工厂 Token的值
           - Token=BanneCharlie
```

![image-20231202201843332](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202201843332.png) 

### Auth

```java
import ...

@Component
 /*自定义路由断言工厂 名称为 Auth(功能前缀) + RoutePredicateFactory,前缀名称将用在配置文件中;
 * 请求头中包含 用户名和密码的key-value对,用户名和密码与Auth断言工厂指定的相同匹配成功
 * */
public class AuthRoutePredicateFactory extends AbstractRoutePredicateFactory<AuthRoutePredicateFactory.Config> {
    // 断言工厂的配置属性
    @Data
    public static class Config{
        private String username;

        private String password;
    }
    // 定义配置文件中参数的顺序
    @Override
    public List<String> shortcutFieldOrder() {
        return Arrays.asList("username","password");
    }
    // 向父类传入配置类,spring cloud gateway得知
    public AuthRoutePredicateFactory(){
        super(Config.class);
    }
    // 覆盖apply方法,返回Predicate对象; 重写test方法(会被SpringCloudGateway调用)
    public Predicate<ServerWebExchange> apply(Config config){
        return serverWebExchange -> {
            // 1.获取请求头
            HttpHeaders headers = serverWebExchange.getRequest().getHeaders();
            // 2.获取请求头中和配置文件username对应的密码
            List<String> content = headers.get(config.getUsername());
            // 3.判断请求头中是否存在符合配置文件的密码
            if (content.contains(config.getPassword())){
                return true;
            }
            return false;
        };
    }

}
```

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    # 配置网关 路由id  目标地址 url  断言设置 predicate
    gateway:
      routes:
        - id: route-config1
          uri: https://www.baidu.com
          predicates:
            # 设置自定义断言工厂 Auth
           - Auth=banne,123456
```

![image-20231202200356833](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231202200356833.png) 

# Filter

简介 : 网关的过滤器工厂允许以某种方式修改特定路由(必须满足断言)传入的HTTP请求或者返回的HTTP响应;

## AddRequestHeader gateway filter

简介 :  该过滤器工厂会添加上指定的请求头,可以多个请求头或者一个请求头多个信息;

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    gateway:
      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/info/**
          uri: http://localhost:8080
          filters:
            # 设置AddRequestHeader过滤工厂,可以同时添加多个请求头或一个请求头具有多个值
            - AddRequestHeader=X-Request-Color,blue
            - AddRequestHeader=X-Request-Color,red
            - AddRequestHeader=X-Request-Size,1024
```

![image-20231203153656711](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231203153656711.png) 

## AddRequestHeadersIfNotPresent gateway filter

简介 : 该过滤器工厂仅仅对不存在的请求头添加指定的header;

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    gateway:
      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/info/**
          uri: http://localhost:8080
          filters:
            # 设置AddRequestHeader过滤工厂,可以同时添加多个请求头或一个请求头具有多个值
            - AddRequestHeader=X-Request-Color,blue
            - AddRequestHeader=X-Request-Color,red
            - AddRequestHeader=X-Request-Size,1024
            # 设置AddRequestHeadersIfNotPresent过滤工厂,仅仅对不存在的请求头添加内容 且是一次添加多个内容,格式为 k:v 
            - AddRequestHeadersIfNotPresent=X-Request-Name:Banne,X-Request-Size:2048
```

![image-20231203155030821](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231203155030821.png) 

## AddResponseHeader gatway filter

简介 : 该过滤器会给从网关返回的响应添加上指定的header;

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    gateway:
      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/info/**
          uri: http://localhost:8080
          filters:
            # 设置AddResponseHeader过滤工厂,给返回的响应添加上指定的header,同时可以添加多个响应头或一个响应头多个值
              - AddResponseHeader=X-Response-Name,Banne
              - AddResponseHeader=X-Response-Name,Harvey
              - AddResponseHeader=X-Response-Age,22
```

![image-20231203161039118](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231203161039118.png)

## AddRequestParameter gateway filter

简介 : 该过滤器会在请求上添加指定的请求参数;

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    gateway:
      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/info/**
          uri: http://localhost:8080
          filters:
            # 设置AddRequestParameter过滤工厂,请求中添加参数 可以添加多个参数或一个参数对应多个值
             - AddRequestParameter=color,blue
             - AddRequestParameter=color,red
             - AddRequestParameter=name,banne
```

![image-20231203160353544](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231203160353544.png) 

## CircuitBreaker gateway filter

简介 : 该过滤器完成网关层的服务熔断与降级;

`pom.xml` 添加resilience依赖,实现服务的降级和熔断

```xml
        <!--配置 SpringCloud Gateway 断路器的依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
        </dependency>
```

`FallbackController.java` 定义降级处理器

```java
package com.banne.gateway.config;
import ...
@RestController
@RequestMapping("/fallback")
public class FallbackController {

    @GetMapping("/first")
    public String  firstFallback(){
        return "路由定向时出现错误,降级到当前页面!";
    }
}
```

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    gateway:
      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/info/**
          uri: http://localhost:8080
          filters:
            # 设置CircuitBreaker过滤工厂,路由定向出现问题时进行降级处理 去当前服务控制器中找匹配的URI
            - name: CircuitBreaker
              args:
                name: myCircuitBreaker
                fallbackUri: forward:/fallback/first
```

![image-20231203162716351](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231203162716351.png) 

## PrefixPath gateway filter

简介 : 该过滤器工厂会为指定的 URI 自动添加上一个指定的 URI 前辍;

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    gateway:
      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/**
          uri: http://localhost:8080
          filters:
            # 设置PrefixPath过滤工厂,会为指定的uri添加前缀uri 目标uri= http://localhost:8080/info/**
            - PrefixPath=/info
```

![image-20231203163538360](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231203163538360.png) 

## StripPrefix gateway filter

简介 : 该过滤器工厂会为指定的URL去掉指定的长度前缀;

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    gateway:
      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/aaa/bbb/info/**
          uri: http://localhost:8080
          filters:
            # 设置StripPrefix过滤工厂,去掉指定长度的前缀 uri=http://localhost:8080/info/**
            - StripPrefix=2
```

![image-20231203164051999](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231203164051999.png)

## RewritePath gateway filter

简介 : 该过滤器工厂会将请求URL替换为另一个指定的URL进行访问;RewritePath 有两个参数,第一个是正则表达式,第二个是要替换为的目标表达式;

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    gateway:
      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/test/**
          uri: http://localhost:8080
          filters:
            # 设置 RewritePath过滤工厂,重新定义URL去向 uri= localhost:8080/info/**
            - RewritePath=/test,/info
```

![image-20231203165448081](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231203165448081.png)

## RequestRateLimiterGateway gateway filter

简介 : 该过滤器工厂通过令牌桶算法对进来的请求路由实现限流的操作,限流后会的到HTTP ERROR 429;

`令牌桶算法分析图:`

![image-20231204150628417](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231204150628417.png) 

`漏斗算法分析:`

![image-20231204150919222](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231204150919222.png) 

`pom.xml` 该限流器基于Redis,需要导入Redis依赖

```xml
        <!--导入redis-reactive依赖 使用限流器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
        </dependency>
```

`GatewayRateConfig.java` 添加限流解析器,从请求中解析出需要限流的Key(用户、URI,目标服务器host或ip等)

```java
package com.banne.gateway.config;

import org.springframework.cloud.gateway.filter.ratelimit.KeyResolver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import reactor.core.publisher.Mono;

@Configuration
public class GatewayRateConfig {

    // 目标服务器host进行限流处理
    @Bean
    public KeyResolver hostKeyResolver(){
        return exchange ->
            Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
    }
}
```

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  data:
#redis 相关配置
    redis:
      host: 192.168.238.128
      port: 6379
      password: 123456

  cloud:
    gateway:
      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/info/**
          uri: http://localhost:8080
          filters:
            # 设置RequestRateLimiter过滤工厂,定义限流的Key值当下定义的为 目标host
            - name: RequestRateLimiter
              args:
                key-resolver: "#{@pathKeyResolver}" # Spring EL表达式,解析请求的键
                redis-rate-limiter.replenishRate:  2 # 每秒令牌桶添加的令牌数目
                redis-rate-limiter.burstCapacity: 5 # 令牌桶的容量
                redis-rate-limiter.requestedTokens: 1 # 每个请求需要的令牌数目
```

![image-20231204154313911](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231204154313911.png) 

## Default gateway filter

简介 : 默认的过滤工厂,对所有的路由都起到作用;但是特定的过滤工厂优先级别高于默认的过滤工厂;

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    gateway:
      # 设置默认的过滤工厂,对所有路由都起作用; 配置的为断路器,目标uri出现错误,进行降级
      default-filters:
        - name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/fallback/first
            
      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/info/param
          uri: http://localhost:8080
          filters:
            # 设置RequestRateLimiter过滤工厂,定义限流的Key值当下定义的为 host
            - name: RequestRateLimiter
              args:
                key-resolver: "#{@pathKeyResolver}" # Spring EL表达式,解析请求的键
                redis-rate-limiter.replenishRate:  1 # 每秒令牌桶添加的令牌数目
                redis-rate-limiter.burstCapacity: 5 # 令牌桶的容量
                redis-rate-limiter.requestedTokens: 2 # 每个请求需要的令牌数目
                
        - id: config2
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/info/name
          uri: http://localhost:8080
          filters:
            - RewritePath=/info/name,/info/header
```

## Custom gateway filter

简介 : 自定义filter路由工厂,前缀为功能名称用在配置文件中,后缀格式为GatewayFilterFactory;继承AbstractNameValueGatewayFilterFactory类或者AbstractGatewayFilterFactory类重写apply自定义功能;

`AddHeaderGatewayFilterFactory.java`

```java
import ...

@Configuration
public class AddHeaderGatewayFilterFactory extends AbstractGatewayFilterFactory<AddHeaderGatewayFilterFactory.Config> {

    // 定义配置类，包含请求头的名称和值
    @Data
    public static class Config{
        private String name;

        private String value;
    }

    // 构造函数，调用父类的构造函数并传入配置类的类型 
    public AddHeaderGatewayFilterFactory(){
        super(Config.class);
    }

    // 定义配置类中字段的顺序
    @Override
    public List<String> shortcutFieldOrder() {
        return Arrays.asList("name","value");
    }

    // 根据给定的配置创建一个新的自定义的GatewayFilter 
    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            // 创建一个新的ServerHttpRequest，包含一个新的请求头
            ServerHttpRequest changedRequest = exchange.getRequest().mutate().header(config.getName(), config.getValue()).build();

            // 使用新的请求创建一个新的ServerWebExchange
            ServerWebExchange webExchange = exchange.mutate().request(changedRequest).build();

            // 继续处理链
            return chain.filter(webExchange);

        };
    }
}
```

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config
    
  cloud:
    gateway:
      # 设置默认的过滤工厂,对所有路由都起作用; 配置的为断路器,目标uri出现错误,进行降级
      default-filters:
        - name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/fallback/first
        - PrefixPath=/info

      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/**
          uri: http://localhost:8080
          filters:
          # 设置自定义AddHeader过滤工厂,功能与AddRequestHeader过滤工厂相同
            - AddHeader=Color,pink
```

![image-20231204173216859](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231204173216859.png) 

## Pre and post

简介 : Spring Cloud Gateway具有pre 和 post 两种方式的filter;

客户端的请求按照filter的优先级顺序先执行pre部分,然后转发到目标服务器,收到响应后执行post部分,最后响应到客户端;

关于filter1 filter2 filter3等的顺序以配置文件的顺序为标准;

![image-20231204173925159](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231204173925159.png) 

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config
    
  cloud:
    gateway:
      routes:
        - id: config1
          # 设置Path断言 将path地址添加到uri地址上进行定位
          predicates:
            - Path=/**
          uri: http://localhost:8080
          filters:
            # 设置的顺序为 filter1 filter2 filter3 
            - OneFilter=filter1,111
            - TwoFilter=filter2,222
            - ThreeFilter=filter3,333
```



![](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231204174038770.png) 

# Global Filter

简介 : 全局过滤器是应用在所有路由策略上的Filter;Spring Cloud Gateway已经定义好了许多Global filter,这些全局过滤无需任何声明与配置,当符合应用条件时会自动生效;

## ReactiveLoadBalancer client filter

简介 : 根据微服务名称进行负载均衡策略;

`pom.xml` 配置Spring Cloud loadbalancer依赖,负载均衡过滤将会自动生效

```xml
        <!--配置 Spring Cloud loadbalancer 负载均衡依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>

        <!--配置 Spring Cloud Alibab的 nacos-discovery 依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2022.0.0.0-RC2</version>
        </dependency>
```

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config


  cloud:

    nacos:
      # 配置Nacos的服务器ip 将gateway-config注册到服务中心
      discovery:
        server-addr: 192.168.128.238:8848


    gateway:
      # 开启自动服务发现功能,允许Gateway自动从Nacos服务器获取服务列表,动态创建路由规则;
      discovery:
        locator:
          enabled: true

      routes:
        - id: lb_route
          # 目标地址为Nacos注册的服务名称 uri = lb://服务名称
          uri: lb://provider-dept
          predicate:
            - Path=/dept
```

## Custom global filter

简介 : Global filter无需在任何具体的路由规则中注册,只需在类上添加@Compoment注解,将生命周期交给IOC容器进行管理;

`URLValidateGlobalFIlter.java `通过实现GlobalFilter 和 Ordered接口重写filter getOrder方法,实现自定义

```java
import ...

@Component
/* 设置的全局filter为 当前系统的任意URL都必须为一个合法的URL即请求中携带token请求参数*/
public class URLValidateGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 获取请求中的参数token
        String token = exchange.getRequest().getQueryParams().getFirst("token");
        // 若token为空,则响应客户端状态码401未授权不让通过;
        if (!StringUtils.hasText(token)){
            exchange.getResponse().setRawStatusCode(HttpStatus.UNAUTHORIZED.value());
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    // 设置顶级优先级别
    public int getOrder(){
        return Ordered.HIGHEST_PRECEDENCE;
    }
}
```

![image-20231204212901738](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231204212901738.png) 

![image-20231204212945297](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231204212945297.png) 

# CORS Configuration

简介 : CORS跨域资源共享,是一种允许当前域的资源(例如，html、js、web service)被其他域的脚本请求访问的机制;Gateway可以通过配置控制CORS行为实现跨域;

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: gateway-config

  cloud:
    gateway:
      # 全局控制CORS行为
      globalcors:
        cors-configurations:
          '[/**]': # 匹配所有的请求路径
            allowedOrigins: "*" # 设置允许所有的源向你的服务发送请求
            allowedMethods: # 发送请求的方式
              - GET
              - POST


      routes:
        - id: config
          uri: http://localhost:8080
          predicates:
            - Path=/**
          # 局部控制CORS行为
          metadata:
            cors:
            allowedOrigins: '*'
            allowedMethods:
              - GET
              - POST
            allowedHeaders: '*' # 允许所有的HTTP请求头
            maxAge: 30
```

![image-20231207155447448](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231207155447448.png) 

