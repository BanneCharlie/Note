# Info

简介 : Nacos(Dynamic Naming and Configuration Service),致力于发现、管理、配置微服务;Nacos是Spring Cloud Alibaba架构中最重要的组件,提供注册中心、配置中心和动态DNS服务(Nacos使用动态DNS服务来实现`中间层负载均衡、灵活的路由策略、流量控制以及简单的DNS解析`服务)三大功能;
	
`DDNS服务 :`DNS只提供了域名和IP地址之间的静态对应关系,无法动态更新;DDNS将用户IP地址映射到固定的域名解析服务器上,实现动态更新IP地址;

![image-20231129131209233](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231129131209233.png) 

*Nacos工作流程 :*

![image-20231129202825369](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231129202825369.png) 

Linux安装Nacos: [https://developer.aliyun.com/article/786617]

![image-20231129132236524](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231129132236524.png) 

![image-20231201154233322](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231201154233322.png) 

*配置或服务隔离模型 :*

![image-20231130182149190](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231130182149190.png) 

# Persistence

简介 : Nacos默认情况下是内置storage,数据存放在内存中,无法持久化而且无法搭建集群;当Nacos作为配置中心时,必须要外置的DBMS;

Nacos目前可外置的DBMS只有Mysql,Nacos配置文件config中具有mysql-schema.sql文件来创建Nacos对应的表格(没有数据库的创建语句建议使用`nacos_config`为数据库名称),运行脚本创建数据库;

`mysql-schema.sql` 创建nacos_config数据库,运行脚本

![image-20231130173812137](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231130173812137.png) 

`config/application.properties` 修改配置开启外置数据库的使用

```properties
#*************** Config Module Related Configurations ***************#
### If use MySQL as datasource:
### Deprecated configuration property, it is recommended to use `spring.sql.init.platform` replaced.
spring.datasource.platform=mysql
spring.sql.init.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
 db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
 db.user.0=root
 db.password.0=123456
```

```shell
# 单机启动
startup.cmd -m standalone
```

![image-20231130174957319](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231130174957319.png)

# SDK

简介 : 通过Java的SDK可以向Nacos拉取对应的配置或者注册服务; 

`pom.xml`

```xml
<dependency>
    <groupId>com.alibaba.nacos</groupId>
    <artifactId>nacos-client</artifactId>
    <version>2.3.0-BETA</version>
</dependency>
```

`PullConfig.java`

```java
import ...

public class PullConfig {
    public static void main(String[] args) throws NacosException, IOException {
        String serverAddr = "localhost:8848";
        String dataId = "BaseConfig";
        String group = "DEFAULT_GROUP";
        Long timeoutMs = 5000L;

        Properties properties = new Properties();
        properties.put("serverAddr", serverAddr);
        ConfigService configService = NacosFactory.createConfigService(properties);

        String content = configService.getConfig(dataId, group, timeoutMs);
        System.out.println("获取配置文件内容: " + content);

        //监听配置文件,随时获取更新的数据;
        configService.addListener(dataId, group, new Listener() {
            @Override
            // 返回null时使用内置的线程池执行，内置线程池最多5个，如果满了则占用拉取线程执行
            public Executor getExecutor() {
                return null;
            }

            @Override
            public void receiveConfigInfo(String configInfo) {
                System.out.println("recieve:" + configInfo);
            }
        });

        System.in.read();

    }
}
```

![image-20231129145205238](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231129145205238.png) 

`Config 相关 API`

```java
// 获取配置
String getConfig(String dataId, String group, long timeoutMs) throws NacosException;

// 发布配置
boolean publishConfig(String dataId, String group, String content, String type) throws NacosException;

// 删除配置
boolean removeConfig(String dataId, String group) throws NacosException;

// 设置监听
void addListener(String dataId, String group, Listener listener);

// 移除监听
void removeListener(String dataId, String group, Listener listener);
```

`PushService.java`

```java
import ...

public class PushService {
    public static void main(String[] args) throws NacosException, IOException {
        // 向Nacos服务器注册服务
        NamingService namingService = NamingFactory.createNamingService("localhost:8848");

        Instance instance = new Instance();
        instance.setEnabled(true);
        instance.setHealthy(true);
        instance.setIp("192.168.238.128");
        instance.setPort(8887);
        
        // 通过instance设置服务状态,注册服务 serviceName Instance
        namingService.registerInstance("user",instance);
        
        // 阻塞进程,模拟服务一直在运行
        System.in.read();
    }
}

```

*Instance相关API设置服务各个属性 :*

![image-20231129211721463](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231129211721463.png) 

![image-20231129205805819](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231129205805819.png)

`PullService.java`

```java
import ...

public class PullService {
    public static void main(String[] args) throws NacosException {
        NamingService namingService = NamingFactory.createNamingService("localhost:8848");
        // 获取服务名为 user的所有实例
        List<Instance> user = namingService.getAllInstances("user");
        for (Instance user1 :user) {
            System.out.println(user1);
        }
    }
}
```

![image-20231129211404034](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231129211404034.png) 

`Service 相关 API`

```java
// 注册实例
void registerInstance(String serviceName, String ip, int port) throws NacosException;

void registerInstance(String serviceName, Instance instance) throws NacosException;

// 注销实例
void deregisterInstance(String serviceName, String ip, int port, String clusterName) throws NacosException;

// 获取全部实例
List<Instance> getAllInstances(String serviceName, List<String> clusters) throws NacosException;

// 获取健康或不健康实例列表
List<Instance> selectInstances(String serviceName, List<String> clusters, boolean healthy) throws NacosException;

// 根据负载均衡算法随机获取一个健康实例
Instance selectOneHealthyInstance(String serviceName, List<String> clusters) throws NacosException;

// 监听服务下的实例列表变化
void subscribe(String serviceName, List<String> clusters, EventListener listener) throws NacosException;

// 取消监听
void unsubscribe(String serviceName, List<String> clusters, EventListener listener) throws NacosException;
```

# SpringCloud

## Config

简介 : Nacos具有配置管理页面,将配置文件放置到Nacos服务器上;通过SpringCloud读取配置文件信息;

本地配置文件优先级高于Nacos配置文件;bootstrap.yml文件优先级别高于application.yml;

`pom.xml`

```xml
 	   <!--开启Spring Cloud 应用程序启动时加载bootstrap配置文件-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
            <version>3.1.4</version>
        </dependency>

        <!--配置SpringCloudAlibaba nacos-config依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
            <version>2022.0.0.0-RC2</version>
        </dependency>
```

*Nacos Config常用配置 :*

![image-20231130150344408](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231130150344408.png) 

`bootstrap.yml`

```yml
spring:
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1 #Nacos服务器IP地址
        group: DEFAULT_GROUP #配置分组
        
        
        # dataId正式格式为 ${prefix} -${spring.profiles.active}.${file-extension}
        prefix: user  # 配置文件的dataId的前缀  ${prefix},不写默认为${spring.application.name}   
        file-extension: properties #必须添加配置文件的格式后缀 --> 获取的配置文件 Type 为 properties 后缀格式
 
 # 设置spring的环境配置
  profiles:
  	active: dev # dataId中缀 -${spring.profiles.active}
```

`OperationConfiguration.java`

```java
import ....
    
@RestController
@RequestMapping("/cloud/nacos")
// 实现自动刷新,获取配置文件修改后的数据  --> @RefreshScope + @Value 动态刷新获取配置文件信息
@RefreshScope
public class PullConfiguration {
    // @Value 获取 配置文件中对应的value值
    @Value(value = "${banne.name}")
    private String name;

    @Value(value = "${banne.age}")
    private int age;

    @GetMapping("/config")
    public void get(){
        System.out.println("SpringCloud获取Nacos配置文件中的信息:" + "\n姓名:" + name + "\n年龄:" + age );
    }
}
```

*Nacos历史版本回滚 :*

![image-20231129200307209](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231129200307209.png) 

*Nacos监听文件 :*

![image-20231129200511151](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231129200511151.png) 

*Nacos中配置文件优先级别顺序 :*

- dataId为 user的配置
- dataId为 user.properties 的配置
- dataId为 user-${spring.profiles.active}.properties 的配置

<img src="https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231129193152326.png" alt="image-20231129193152326" style="zoom: 150%;" /> 

![image-20231129204400859](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231129204400859.png) 

*Nacos中获取多个配置文件 :*

- extension-configs拉取额外的配置文件,表示本应用特有的
- shared-configs拉取额外的配置文件,表示多个应用共同享有的

```yml
spring:
  cloud:
    nacos:
      # 配置中心
      config:
        server-addr: 127.0.0.1 #Nacos服务器IP地址
        timeout: 1000
        # 主配置文件有优先级最高
        prefix: user
        file-extension: properties

        # 读取多个配置文件(本应用文件特有的) 优先级其次 ,形成数组形式;进行读取 索引越大优先级别越高
        extension-configs[0]:
            data-id: common1.properties # 配置dataId,必须添加文件后缀
            refresh: true #支持刷新
            
        # 读取多个配置文件(应用共享的文件) 优先级再次 ,形成数组形式;进行读取 索引越大优先级别越高
        shared-configs[0]:
          data-id: common2.properties # 配置dataId,必须添加文件后缀
          refresh: true #支持刷新

```

*配置的自动刷新 :*

- 自动刷新配置开不开启,基本上都无法实现应用端的数据动态更新为服务端更改的数据;

- @Value 注解 获取服务端配置文件的信息,服务端更改数据 应用端无法获取新的数据,

  需要@RefreshScope注解创建新的Bean对象,获取新的数据实现动态更新效果;

## Service

简介 : Nacos具有服务管理页面作为注册中心,将服务列表和对应的订阅者列表放置到Nacos服务器上;

![image-20231130152928469](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231130152928469.png) 

`pom.xml`

```xml
        <!--开启Spring Cloud 应用程序启动时加载bootstrap配置文件-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
            <version>3.1.4</version>
        </dependency>

        <!--配置SpringCloudAlibaba nacos-discovery依赖,注册服务-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2022.0.0.0-RC2</version>
        </dependency>

        <!--开启Spring Cloud配置loadbalancer 实现负载均衡,调用服务时使用-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
            <version>4.0.2</version>
        </dependency>
```

### Register service

简介 : SpringCloud导入nacos-discovery依赖,配置文件设置服务名称 端口号和对应的nacos服务地址,会实现自动注册服务;

`bootstrap.yml`

```yml
# 服务端口号
server:
  port: 8001
 
spring:
  # 服务名称 
  application:
    name: provider
    
  cloud:
    nacos:
      discovery:
        # 注册的Nacos服务中心 IP地址
        server-addr: 127.0.0.1:8848
        # 服务的权重
        weight: 1
        # 区分不同的区域范围
        namespace: Test1
	   group: one
```

*Nacos Service常用 :*

![image-20231130153530222](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231130153530222.png) 

`ProvideService.java`

```java
import ...
// Nacos服务器上注册了 服务名称为 provider prot为8001 Ip为 10.20.202.244,通过Ip地址访问它提供的API
@RestController
@RequestMapping("/cloud/service/provider")
public class ProviderService {
    
    @GetMapping("/test")
    public String Get(){
        return "Provide Successfully!";
    }
}
```

![image-20231130113647694](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231130113647694.png) 

### Invoke service

简介 :   SpringCloud导入nacos-discovery依赖 loadbalancer依赖,通过配置文件获取Nacos服务器Ip地址,调用服务;

`bootstrap.yml`

```yml
server:
  port: 8002

spring:
  application:
    name: consumer

  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```

`ConsumerService.java`

```java
import ..

@RestController
@RequestMapping("/consumer")
public class ConsumerService {
    
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/test")
    public String Get(){
        // 通过ip地址,调用Nacos服务器上的注册服务 以GET请求的方式
        String result = restTemplate.getForObject("http://provider/test1", String.class);
        return result;
    }
}
```

`ConsumerApplication.java`

```java
package com.example.springcloudconsumer;

import ....

@SpringBootApplication
@EnableDiscoveryClient
public class SpringCloudConsumerApplication {

    @Bean
    @LoadBalanced
    // 创建 RestTemplate bean对象 + @LoadBalanced实现负载均衡
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }


    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConsumerApplication.class, args);
    }
}
```

*临时实例和持久化实例 :*

- Nacos注册的实例都是临时实例,默认情况下客户端每5秒发送一次心跳,15秒没收到客户端心跳,实例标记为不健康状态,30秒将会删除实例;
- 通过配置spring.cloud.nacos.discovery.ephemeral = false 设置为非临时实例,就算下线也不会进行删除只是标记为不健康状态,无法使用;

*保护阈值 :*

![image-20231130155009791](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231130155009791.png) 

*权重 :*

- 消费服务时,通过RestTemplate实现负载均衡,Nacos设置权重无法启到作用
- 通过设置ribbonRule利用Nacos所配权重

![image-20231130155408036](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231130155408036.png) 

# Cluster

docker部署集群: https://developer.aliyun.com/article/1045806 

![image-20231130193137823](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231130193137823.png) 