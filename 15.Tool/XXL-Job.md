# Info

简介 : 任务调度是为了自动完成特定任务,在约定的特定时刻去执行任务的过程;

Spring框架中的`@Schedule + @EnableScheduling`注解也可实现调度的任务,但是为了满足更多的场景和高可用的特点引出了分布式任务.

```java
@Scheduled(cron = "0/20 * * * * ? ")
 public void doWork(){
 	//doSomething   
 }
```

*分布式事务的使用原因 :*

1. 高可用：单机版的定式任务调度只能在一台机器上运行, 如果程序或者系统出现异常就会导致功能不可用.
2. 防止重复执行: 在单机模式下, 定时任务是没什么问题的; 但当我们部署了多台服务, 同时又每台服务又有定时任务时, 若不进行合理的控制在同一时间, 只有一个定时任务启动执行, 定时执行的结果就可能存在混乱和错误了.
3. 单机处理极限：原本1分钟内需要处理1万个订单，但是现在需要1分钟内处理10万个订单；原来一个统计需要1小时，现在业务方需要10分钟就统计出来。你也许会说，你也可以多线程、单机多进程处理。的确，多线程并行处理可以提高单位时间的处理效率，但是单机能力毕竟有限（主要是CPU、内存和磁盘），始终会有单机处理不过来的情况。

简介 : 分布式调度的常用组件为 XXL-Job是大众点评的分布式任务调度中心, 该系统内部已调度约100万次, 表现优异;

`系统架构图`

![image-20240404141339073](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404141339073.png) 

# Deploy

简介 : XXL-Job 通过docker进行部署,部署前需要将对应的数据库进行配置;

![image-20240404142628525](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404142628525.png) 

```shell
docker run -d  -e PARAMS="--spring.datasource.url=jdbc:mysql://192.168.238.128:3306/xxl_job?useUnicode=true&characterEncoding=utf-8&useSSL=false \
--spring.datasource.username=root \
--spring.datasource.password=123456 \
--spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver \
--xxl.job.accessToken=xdclass.net" \
-p 8080:8080 \
--name xxl-job --restart=always  xuxueli/xxl-job-admin:2.4.0
```

`客户端 默认账号密码 admin / 123456`

![image-20240404153204767](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404153204767.png)  

# Config Project

`xxl-job-core依赖`

```xml
<dependency>
    <groupId>com.xuxueli</groupId>
    <artifactId>xxl-job-core</artifactId>
    <version>2.4.0</version>
</dependency>
```

`application.properties`

```properties
### 调度中心部署根地址 [选填]：如调度中心集群部署存在多个地址则用逗号分隔。执行器将会使用该地址进行"执行器心跳注册"和"任务结果回调"；为空则关闭自动注册；
xxl.job.admin.addresses=http://192.168.238.128:8080/xxl-job-admin
### 执行器通讯TOKEN [选填]：非空时启用；
xxl.job.accessToken=xdclass.net
### 执行器AppName [选填]：执行器心跳注册分组依据；为空则关闭自动注册
xxl.job.executor.appname=xxl-job-executor-sample
### 执行器注册 [选填]：优先使用该配置作为注册地址，为空时使用内嵌服务 ”IP:PORT“ 作为注册地址。从而更灵活的支持容器类型执行器动态IP和动态映射端口问题。
xxl.job.executor.address=
### 执行器IP [选填]：默认为空表示自动获取IP，多网卡时可手动设置指定IP，该IP不会绑定Host仅作为通讯实用；地址信息用于 "执行器注册" 和 "调度中心请求并触发任务"；
xxl.job.executor.ip=
### 执行器端口号 [选填]：小于等于0则自动获取；默认端口为9999，单机部署多个执行器时，注意要配置不同执行器端口；
xxl.job.executor.port=9999
### 执行器运行日志文件存储磁盘路径 [选填] ：需要对该路径拥有读写权限；为空则使用默认路径；
xxl.job.executor.logpath=/data/applogs/xxl-job/jobhandler
### 执行器日志文件保存天数 [选填] ： 过期日志自动清理, 限制值大于等于3时生效; 否则, 如-1, 关闭自动清理功能；
xxl.job.executor.logretentiondays=30
```

`XxlJobConfig.java`  创建配置对象

```java
@Configuration
public class XxlJobConfig {
    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;
    @Value("${xxl.job.accessToken}")
    private String accessToken;
    @Value("${xxl.job.executor.appname}")
    private String appname;
    @Value("${xxl.job.executor.address}")
    private String address;
    @Value("${xxl.job.executor.ip}")
    private String ip;
    @Value("${xxl.job.executor.port}")
    private int port;
    @Value("${xxl.job.executor.logpath}")
    private String logPath;
    @Value("${xxl.job.executor.logretentiondays}")
    private int logRetentionDays;

    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);
        return xxlJobSpringExecutor;
    }
}
```

`XxlJobTest.java`  

```java
@Component
public class SimpleXxlJob {
    // @XxlJob 注解 指定需要调度的任务名称
    @XxlJob("demoJobHandler")
    public void demoJobHandler() throws Exception {
        System.out.println("执行定时任务,执行时间:"+new Date());
    }
}
```

![image-20240404163521332](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404163521332.png) 

![image-20240404164729370](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404164729370.png) 

![image-20240404164826627](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404164826627.png) 

# GLUE Pattern(Java)

简介 :  任务以源码方式维护在调度中心, 支持通过Web IDE在线更新, 实时编译和生效, 因此不需要指定JobHandler;

`(GLUE模式(Java) 运行模式`的任务实际上是一段继承自IJobHandler的Java类代码, 它在执行器项目中运行, 可使用[@Resource](https://github.com/Resource)/[@Autowire](https://github.com/Autowire)注入执行器里中的其他服务.

`新建GLUE运行模式`

![image-20240404171401025](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404171401025.png) 

`线上添加定时任务 :` 

![image-20240404171513169](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404171513169.png) 

# Actuator Clusters

简介 : 执行器集群的创建可以对调度的任务采取多种执行的方式, 提高调度的速度. 

`配置服务器集群`

启动两个SpringBoot程序,需要修改Tomcat端口和执行器端口

- Tomcat端口8090程序的命令行参数如下:

  ```shell
  -Dserver.port=8000 -Dxxl.job.executor.port=9999
  ```

- Tomcat端口8090程序的命令行参数如下:

  ```sh
  -Dserver.port=8001 -Dxxl.job.executor.port=9998
  ```

![image-20240404172757610](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404172757610.png) 

`修改路由策略为轮询`

![image-20240404173051270](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404173051270.png) 

`路由策略解释 :`

![image-20240404171926061](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404171926061.png)

# Sharding

简介 : 采取分片广播的形式的话, 一次任务调度将会广播触发对应集群中所有执行器执行一次任务, 同时系统自动传递分片参数; 可根据分片参数开发分片任务；

获取分片参数方式.

```java
// 分片的索引
int shardIndex = XxlJobHelper.getShardIndex();
// 分片的总数
int shardTotal = XxlJobHelper.getShardTotal();

// id越有序 分布越均匀
@Select("select * from t_user_mobile_plan where mod(id,#{shardingTotal}) = #{shardingIndex}")
```

![image-20240404195246470](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404195246470.png) 

![image-20240404201753597](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240404201753597.png)  

数据库插入100w条数据 

Mybatis批量插入 + 开启事务(确保执行正确) + XXL-Job(调度中心通过DB锁保证集群中执行任务的唯一性,短任务较多,随着调度中心集群数量增加, 锁竞争激烈.)

可以通过分布式任务调度进行,使用分片广播的形式进行, 配置10个执行器 每个执行器向数据库插入 10w条数据,10个执行器一起进行插入; 每个需要插入的位置根据分片索引 和 分片总数来确定;