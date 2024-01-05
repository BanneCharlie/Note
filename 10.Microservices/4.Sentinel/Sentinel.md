# Info

简介 : Sentinel 是`面向分布式、多语言异构化`服务架构的流量治理组件，主要以流量为切入点，从流量路由、流量控制、流量整形、熔断降级、系统自适应过载保护、热点流量防护等多个维度来帮助开发者保障微服务的稳定性;

![image-20231212195508986](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231212195508986.png) 

# Dashboard

简介 : Sentinel控制台是Sentinel的一个轻量级的GUI控制台(Springboot编写JDK8以及以上版本),提供对Sentinel主机(Sentinel应用)的健康管理、动态配置服务控流、熔断、路由规则的配置与管理;

```shell
# 启动Sentinel Dashboard 
java -Dserver.port=8888 ^ # config import
-Dsentinel.dashboard.auth.username=sentinel ^ # config username
-Dsentinel.dashboard.auth.password=123456 ^ # config password
-jar sentinel-dashboard-1.8.6.jar
```

![image-20231212200530653](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231212200530653.png) 

`application.xml`  服务 provider-dept配置Sentinel

```yml
server:
  port: 9001

spring:
  # 服务名称
  application:
    name: consumer-dept

  # Nacos 相关配置
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.238.128


  # 添加Sentinel控制台的相关配置,将该服务通过Sentinel监控
    sentinel:
      transport: # 配置Sentinel的传输层
        dashboard: localhost:8888 # Sentinel的控制台地址
        port: 8719
      # 启用Sentinel的立即加载模式,Sentinel将在应用程序启动时立即记载,而不是首次请求加载
      eager: true
```

![image-20231213122739954](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231213122739954.png)

# Service Avalanche

简介 : 服务雪崩是指在一个分布式系统中，当大量的服务实例因某种原因（例如高负载、依赖服务故障、配置错误等）同时出现故障或不可用时，导致整个系统的级联故障;

- **单点故障：** 如果系统中的某个关键服务出现故障，其他服务请求可能无法正常完成，导致连锁反应。

![image-20231214112539079](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214112539079.png)  

- **依赖服务故障：** 当一个服务依赖于其他服务，如果依赖的服务出现故障，可能导致调用方服务在等待响应时产生大量阻塞，进而引发雪崩。

![image-20231214112431766](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214112431766.png) 

- **资源耗尽：** 当某个服务的资源（如数据库连接、线程池）用尽时，它可能无法及时响应请求，从而导致其他服务等待超时，形成雪崩。

![image-20231214112404497](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214112404497.png) 

- **超时和重试：** 当某个服务在处理请求时出现延迟或超时，发出请求的服务可能会发起重试。如果重试过于频繁，可能导致后端服务更大的负载，最终触发雪崩效应。 

- **大规模扩容：** 在系统负载剧增时，如果没有合理的负载均衡和流量控制机制，可能导致服务无法及时扩容，引发雪崩。
- **缓存击穿：** 如果缓存中的某个热点数据失效，而此时又有大量请求涌入，数据库或后端服务可能会因为请求集中而崩溃。
- **配置错误：** 不正确的配置可能导致系统异常，例如错误的超时设置、线程池配置等。

*处理服务雪崩的方法 :*

- 超时处理: 设置超时时间,请求到达一定时间没有响应就返回错误信息,不会一直等待;

![image-20231214112644745](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214112644745.png) 

- 线程隔离: 设定每个业务能使用的线程数,避免耗尽整个tomcat的资源,形成阻塞;

![image-20231214112802705](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214112802705.png) 

- 降级熔断: 断路器统计服务的请求数量,异常比例达到阈值会熔断此业务;

 ![image-20231214112936325](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214112936325.png)

- 流量控制: 限制业务访问的QPS(每秒钟请求数目),避免服务因流量的突增而故障,避免雪崩的预防措施;

![image-20231214113213790](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214113213790.png) 

# Service Degradation

简介 : 服务降级可以提升用户的体验,当用户请求被拒绝时,系统会返回一个事先设定好的、用户可以接受的,但又令用户并不满意的结果;

## Sentinel degrade

```xml
        <!--添加 sentinel 依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
            <version>2022.0.0.0-RC2</version>
        </dependency>
```

`UserController.java` Sentinel方法式服务降级

```java
import ..
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    DeptService deptService;

/*  Sentinel方法式服务降级 selectById()添加服务降级  
    降级方法为selectByIdFallback()返回值和参数类型必须与原方法相同,方法名随意;
*/
    @SentinelResource(fallback = "selectByIdFallback")
    // 根据id查询部门信息
    @GetMapping("/id")
    public Dept selecById(@RequestParam int id){
        return  deptService.selectById(id);
    }
    
    // 降级方法  可添加参数Throwable,获取降级时相应异常对象
    public Dept selectByIdFallback(int id,Throwable throwable){
        Dept dept = new Dept();
        dept.setId(id);
        dept.setDeptName("degrade-method-" + id + "-" + throwable.getMessage());
        return dept;
    }
}
```

![image-20231212204002287](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231212204002287.png) 



`UserServiceDegrade.java` 自定义降级类

```java
import ..

// 自定义降级类
public class UserServiceDegrade{

    // 对应 selectById() 方法,返回值和参数类型必须与原方法相同,方法名随意
    public static Dept selectByIdFallback(int id, Throwable throwable){
        Dept dept = new Dept();
        dept.setId(id);
        dept.setDeptName("degrade-class-" + id + "-" + throwable.getMessage());
        return dept;
    }
}
```

`UserController.java` Sentinel类式服务降级

```java
import ..
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    DeptService deptService;

/*  Sentinel类式服务降级  UserServiceDegrade 自定义降级类
    fallback属性指定降级类中的对应方法  fallbackClass对应降级类
*/
    @SentinelResource(fallback = "selectByIdFallback",fallbackClass = UserServiceDegrade.class)
    // 根据id查询部门信息
    @GetMapping("/{id}")
    public Dept selecById(@PathVariable int id){
        return  deptService.selectById(id);
    }

}
```

![image-20231212205033546](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231212205033546.png)

## Feign degrade

```yml
server:
  port: 9001

spring:
  application:
    name: consumer-dept

  cloud:
    nacos:
      discovery:
        server-addr: 192.168.238.128
        
      
# 开启Fegin对Sentinel的支持
feign:
  sentinel:
    enabled: true
```

`UserServiceDegrade.java` Feign式服务降级

```java
import ..

/* 
降级类必须实现Feign接口, 生命周期要交由 Spring容器管理，类上的@RequestMapping 的内容要与 Controller 处理器上的相同，但最前必须要添加/fallback 的 URI
*/
@Component
@RequestMapping("/fallback/dept")
public class UserServiceDegrade implements DeptService {
    @Override
    public List<Dept> selectAll() {
        System.out.println("执行selectAll()的服务降级处理方法");
        return null;
    }

    @Override
    public Dept selectById(int id) {
        Dept dept = new Dept();
        dept.setId(id);
        dept.setDeptName("degrade-fegin-" + id + "-" + "执行selectById()的服务降级处理方法");
        return dept;
    }

    @Override
    public Result insert(Dept dept) {
        return new Result().error("-1","执行insert()的服务降级处理方法");
    }

    @Override
    public Result delete(int id) {
        return new Result().error("-1","执行delete()的服务降级处理方法");
    }

    @Override
    public Result update(Dept dept) {
        return new Result().error("-1","执行update()的服务降级处理方法");
    }
}
```

`DeptService.interface` Feign模仿RESTful接口风格

```java
import ..
// 指定服务降级类
@FeignClient(value = "provider-dept",path = "/dept",fallback = UserServiceDegrade.class)

public interface DeptService {
    //查询所有部门信息
    @GetMapping
    public List<Dept> selectAll();

    //根据id查询部门信息
    @GetMapping("/{id}")
    public Dept selectById(@PathVariable  int id);

    //添加部门信息
    @PostMapping
    public Result insert(@RequestBody  Dept dept);

    //删除部门信息
    @DeleteMapping
    public Result delete(@RequestParam  int id);

    //修改部门信息
    @PutMapping
    public Result update(@RequestBody  Dept dept);
}
```

![image-20231212211804542](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231212211804542.png)

# Cluster Link

简介 : 请求进入微服务时,首先访问DispatcherServlet,进入Controller Service Mapper这样的调用链称为`簇点链路`;

默认情况下Sentinel会监控SpringMVC的Endpoint(Controller中的方法),SpringMVC中的每个Endpoint就是调用链路中的资源;

![image-20231214114325832](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214114325832.png) 

流控、熔断等都是针对簇点链路中的资源来设置的,因此我们可以点击对应资源后面的按钮来动态设置规则(规则存放在内存中)：

- 流控：流量控制
- 降级：降级熔断
- 热点：热点参数限流，是限流的一种
- 授权：请求的权限控制

# Flow Control Rule

简介 : 流量控制是避免服务因突发的流量而发生故障,是对微服务雪崩问题的预防;

![image-20231214125228720](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214125228720.png) 

```java
// API式 创建流控规则
FlowRule rule = new FlowRule();
rule.setResource("/user"); // 资源名称，可以是方法名或 URL 等
rule.setGrade(RuleConstant.FLOW_GRADE_QPS); // QPS 限流
rule.setCount(3); // 每秒允许通过的请求数

// 加载规则
FlowRuleManager.loadRules(Collections.singletonList(rule));

// loadRules()底层源码
public static void loadRules(List<ParamFlowRule> rules) {
 try {
      currentProperty.updateValue(rules);
	 } catch (Throwable var2) {
       RecordLog.info("[ParamFlowRuleManager] Failed to load rules", var2);
	}
}

// singletonList()底层源码
public static <T> List<T> singletonList(T o) {
	return new SingletonList<>(o);
}
```

![image-20231214195552725](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214195552725.png) 

```java
// API式 创建流控规则
FlowRule rule = new FlowRule();
rule.setResource("yourResourceName"); // 资源名称
rule.setGrade(RuleConstant.FLOW_GRADE_THREAD); // 线程数限流
rule.setCount(5); // 允许的最大并发线程数

// 加载规则
FlowRuleManager.loadRules(Collections.singletonList(rule));
```



*流控模式 :*

- 直接: 统计当前资源的请求,触发阈值对当前资源直接限流,也是默认模式;

![image-20231214125648035](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214125648035.png) 

- 关联: 统计与当前资源相关的另一个资源请求,触发阈值,对当前资源进行限流;

![image-20231214155353958](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214155353958.png)  

eg: 比如用户支付时需要修改订单状态，同时用户要查询订单。查询和修改操作会争抢数据库锁，产生竞争。业务需求是优先支付和更新订单的业务，因此当修改订单业务触发阈值时，需要对查询订单业务限流。

- 链路: 统计从指定链路访问到本资源的请求,触发阈值,对指定链路限流;

 ![image-20231214160546118](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214160546118.png) 

*流控效果 :*

- 快速失败: 达到阈值后,新的请求会被立刻拒绝并抛出FlowException异常,是默认的处理方式
- Warm Up: 预热模式,对超出阈值的请求同样是拒绝并抛出异常;但这种模式阈值会发生动态变化,从较小的阈值增加到最大阈值

![image-20231214161437963](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214161437963.png) 

![image-20231214161609135](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214161609135.png) 

- 排队等待: 让所有的请求按照先后次序排队执行,按照阈值允许的时间间隔依次执行请求;如果请求预期等待时长大于超时时间,直接拒绝;

![image-20231214163020073](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214163020073.png) 

# Hotspot  Rule

简介 : 热点参数限流是分别统计参数值相同的请求,判断是否超过QPS阈值,进行限流;

热点参数限流对默认的`SpringMVC资源无效`,需要利用@SentinelResource注解标记资源;

![image-20231214164226784](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214164226784.png) 

`全局参数限流 :`

![image-20231214164510301](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214164510301.png) 

```java
// API 创建热点参数限流规则
ParamFlowRule rule = new ParamFlowRule();
rule.setResource("/user");
rule.setParamIdx(0); // 参数的索引，假设业务参数在方法的第一个位置
rule.setCount(5);
rule.setGrade(RuleConstant.FLOW_GRADE_QPS); // QPS模式
rule.setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER);

// 加载规则
ParamFlowRuleManager.loadRules(Collections.singletonList(rule));
```

`热点参数限流 :`

![image-20231214164839095](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214164839095.png)  

# Circuit Breaker Rule

简介 : 熔断规则用于完成服务熔断,服务熔断为: `在分布式系统中,服务熔断的目的是通过在服务之间引入断路器，来防止由于某个服务故障而引起的级联故障。服务熔断器会监控对远程服务的调用，并在发现异常或故障时暂时中断对该服务的调用，而不是一直尝试执行可能会失败的操作;`处理服务雪崩;

`状态机工作流程 :`

![image-20231214165405195](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214165405195.png)

- 慢调用比例熔断策略: 

请求的响应时间大于慢调用RT则统计为慢调用;

单位统计时长内请求数目大于最小请求数目并且慢调用的比例大于阈值,会触发熔断;

![image-20231213135641819](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231213135641819.png) 

```java
// API式 创建慢调用熔断规则
DegradeRule degradeRule = new DegradeRule();
// 配置熔断服务的资源
degradeRule.setResource("/user/{id}");
// 配置熔断的级别
degradeRule.setGrade(RuleConstant.DEGRADE_GRADE_RT);
// 配置慢调用RT
degradeRule.setCount(2);
// 配置比例阈值
degradeRule.setSlowRatioThreshold(0.2);
// 配置熔断时长
degradeRule.setTimeWindow(5);
// 配置最小请求数目
degradeRule.setMinRequestAmount(10);
// 配置统计时长
degradeRule.setStatIntervalMs(1000);

// 加载熔断规则
DegradeRuleManager.loadRules(Collections.singletonList(degradeRule));
```

![](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231213150135083.png) 

- 异常比例熔断策略

单位统计时间内请求数目大于设置的最小请求数目并且异常的比例大于阈值,会触发熔断;

![image-20231213152226908](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231213152226908.png) 

```java
// API式 创建异常比例熔断规则
DegradeRule degradeRule = new DegradeRule();
// 配置熔断服务的资源
degradeRule.setResource("/user/{id}");
// 配置熔断的级别
degradeRule.setGrade(RuleConstant.DEGRADE_GRADE_EXCEPTION_RATIO);
// 配置异常比例阈值
degradeRule.setCount(0.3);
// 配置熔断时长
degradeRule.setTimeWindow(5);
// 配置最小请求数目
degradeRule.setMinRequestAmount(10);
// 配置统计时长
degradeRule.setStatIntervalMs(1000);

// 加载熔断规则
DegradeRuleManager.loadRules(Collections.singletonList(degradeRule));
```

![image-20231213152457010](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231213152457010.png) 

- 异常数熔断策略

单位统计时间内异常数目超过阈值,会触发熔断;

![image-20231213152401787](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231213152401787.png)

```java
// API式 创建异常数熔断规则
DegradeRule degradeRule = new DegradeRule();
// 配置熔断服务的资源
degradeRule.setResource("/user/{id}");
// 配置熔断的级别
degradeRule.setGrade(RuleConstant.DEGRADE_GRADE_EXCEPTION_COUNT);
// 配置异常数目
degradeRule.setCount(3);
// 配置熔断时长
degradeRule.setTimeWindow(5);
// 配置最小请求数目
degradeRule.setMinRequestAmount(10);
// 配置统计时长
degradeRule.setStatIntervalMs(1000);

// 加载熔断规则
DegradeRuleManager.loadRules(Collections.singletonList(degradeRule));
```

![image-20231213152526578](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231213152526578.png) 

`SpringApplication.java`

```java
import ...
@SpringBootApplication
@EnableFeignClients
public class SpringApplication 
{
    public static void main( String[] args ){
        SpringApplication.run(App.class,args);
        // 启动服务时,调用该方法 生成自定义的熔断规则
        initRule();
    }
} 
```

# Authorization Rule

简介 : 授权规则可以对请求方来源做判断和控制;

![image-20231214183158077](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214183158077.png) 

![image-20231214183318892](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214183318892.png) 

```java
// API式 创建权限规则
AuthorityRule rule = new AuthorityRule();
rule.setResource("/user");
rule.setStrategy(RuleConstant.AUTHORITY_WHITE);// 设置策略为白名单
rule.setLimitApp("appA,appB"); // 设置限制访问的应用列表，用逗号分隔

// 加载规则
AuthorityRuleManager.loadRules(Collections.singletonList(rule));

// 调用方信息通过 ContextUtil.enter(resourceName,origin)方法中的 origin 参数传入
ContextUtil.enter("/user","appA");
```

![image-20231214203015055](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214203015055.png)  

# Custom Exception Result

简介 : 默认情况下,发生限流 、降级、授权拦截时,都会抛出异常到调用方,异常结果皆为 flow limmiting;

通过实现 BlockExceptionHandler接口的handle()方法自定义异常结果;

```java
public interface BlockExceptionHandler {
    /**
     * 处理请求被限流、降级、授权拦截时抛出的异常：BlockException
     */
    void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception;
}
```

*BlockException异常的子类 :*

- FlowException  限流异常
- ParamFlowException 热点参数限流异常
- DegradeException 降级异常
- AuthorityException 授权规则异常
- SystemBlockException 系统规则异常

`CustomSentinelExceptionHandler.java` 自定义异常类

```java
import ...

@Component
public class CustomSentinelExceptionHandler implements BlockExceptionHandler {
    @Override
    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, BlockException e) throws Exception {
        String message  = "异常信息";
        int status = 429;

        if (e instanceof FlowException){
            message = "请求被限流了";
        } else if (e instanceof ParamFlowException) {
            message = "请求被热点参数限流";
        } else if (e instanceof DegradeException) {
            message = "请求被降级了";
        } else if (e instanceof AuthorityException){
            message = "没有访问权限";
            status = 401;
        }
	   // 响应的内容类型为 JSON 格式,制定字符集为 UTF-8	
        httpServletResponse.setContentType("application/json;charset=utf-8");
        httpServletResponse.setStatus(status);
        // 获取响应的 Writer 对象 并打印为 JSON格式的消息和状态码
        httpServletResponse.getWriter().println("{\"message\":" + message + ", \"status\" :" + status + "}");
    }
}
```

![image-20231214192852067](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214192852067.png) 

![image-20231214192959676](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214192959676.png) 

# Gateway Flow Control

简介 : Sentinel1.6版本之后,对主流网关限流的实现;主要是通过对网关的入口流量进行实时监控和控制，确保网关和后端服务的稳定性，以防止潜在的流量过载和系统崩溃;

![image-20231215102248296](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231215102248296.png) 

`pom.xml` 

```xml
<!--sentinel 与 spring cloud gateway 整合依赖-->
<dependency>
 <groupId>com.alibaba.cloud</groupId>
 <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
</dependency>

<!--sentinel 依赖-->
<dependency>
 <groupId>com.alibaba.cloud</groupId>
 <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```



*限流维度 :*

- Route维度: 根据网关路由中指定路由id进行限制

![image-20231215110558244](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231215110558244.png) 

- API维度: 使用 Sentinel 提供的 API 自定义分组进行限流





# Rule Persistence

简介 : Sentinel的所有规则都是存放在内存中存储,重启后都会丢失;生产环境下必须确保这些规则的持久化,避免丢失;

*持久化模式 :*

- 原始模式：Sentinel的默认模式，将规则保存在内存，重启服务会丢失。
- pull模式: 控制台将配置的规则推送到Sentinel客户端，而客户端会将配置规则保存在本地文件或数据库中;

![image-20231214193334990](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214193334990.png) 

- push模式: 控制台将配置规则推送到远程配置中心, 例如Nacos;Sentinel客户端监听Nacos,获取配置变更的推送消息,完成本地配置更新;

![image-20231214193414376](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231214193414376.png) 