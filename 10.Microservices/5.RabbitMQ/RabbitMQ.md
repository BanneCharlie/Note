# Info

简介 : RabbitMQ是一个开源的中间件(Message Broker),实现了高级消息队列协议(AMQP),用于在分布式系统中异步传递消息、程序之间进行通信、支持解耦和可伸缩性;

*异步通信的优缺点 :*

优点：

- 吞吐量提升：无需等待订阅者处理完成，响应更快速

- 故障隔离：服务没有直接调用，不存在级联失败问题
- 调用间没有阻塞，不会造成无效的资源占用
- 耦合度极低，每个服务都可以灵活插拔，可替换
- 流量削峰：不管发布事件的流量波动多大，都由Broker接收，订阅者可以按照自己的速度去处理事件



缺点：

- 架构复杂了，业务没有明显的流程线，不好管理
- 需要依赖于Broker的可靠、安全、性能

`RabbitMQ的基本结构 :`

![image-20231218093916899](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218093916899.png) 

`RabbitMQ的工作流程图 : `

![image-20231220095954833](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231220095954833.png) 

*RabbitMQ中的角色 :*

- Publisher 生产者
- VirtualHost 虚拟主机,隔离不同租户的exchange queue 消息的隔离
- Exchange 交换器,负责消息路由
- Queue 队列,存储消息
- Binding 绑定,用于消息队列和交换器之间的关联
- Message 消息,通过消息头和消息体组成
- Connection 网路连接
- Channel 通信,多路复用连接中的一条单独的双向数据通道
- Consumer 消费者

`docker 部署RabbitMQ`

```shell
docker run \
 -e RABBITMQ_DEFAULT_USER=admin \
 -e RABBITMQ_DEFAULT_PASS=12345 \
 --name mq \
 --hostname mq1 \
 -p 15672:15672 \
 -p 5672:5672 \
 -d \
 rabbitmq:3.12-management
```

<img src="https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218095856958.png" alt="image-20231218095856958" style="zoom:80%;" /> 

`pom.xml` 添加依赖

```xml
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

`application.yml`  连接到RabbitMQ服务器(单实例)

```yml
spring:
 # 配置RabbitMQ的地址
  rabbitmq:
    host: 192.168.238.128 # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: admin # 用户名
    password: 123456 # 密码
```



`docker 部署RabbitMQ集群` 默认元数据模式

rabbitmq-cluster.sh

```shell
# 创建 rabbitMQ 数据的挂载卷
mkdir -p /myrabbitmq/rabbitmq/data

docker run  -d --hostname rabbit1 --name myrabbit1 \
-p 15672:15672  -p 5672:5672 \
-v /myrabbitmq/rabbitmq/data:/var/lib/rabbitmq \
# 集群中的 RABBITMQ_ERLANG_COOKIE 必须一致,用于集群之间的通信 
\ -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=123456 \
rabbitmq:3.12-management \

docker run -d --hostname rabbit2 --name myrabbit2 \
-p 15673:15672 -p 5673:5672 \
--link myrabbit1:rabbit1 \
-v /myrabbitmq/rabbitmq/data:/var/lib/rabbitmq \
-e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=123456 \
rabbitmq:3.12-management  \

docker run -d --hostname rabbit3 --name myrabbit3 \
-p 15674:15672 -p 5674:5672  \
--link myrabbit1:rabbit1  \
--link myrabbit2:rabbit2  \
-v /myrabbitmq/rabbitmq/data/data:/var/lib/rabbitmq \
-e RABBITMQ_ERLANG_COOKIE='rabbitcookie' -e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=123456 \
rabbitmq:3.12-management \
```

```shell
# 执行 rabbit-cluster.sh脚本
bash rabbit-cluster.sh

# 集群中的第一个节点将初始元数据代入集群中，并且无须被告知加入;
docker exec -it myrabbit1 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
exit

# 第2个和之后加入的节点将加入它并获取它的元数据。要加入节点，需要进入Docker容器，重启RabbitMQ
docker exec -it myrabbit2 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbit@rabbit1
rabbitmqctl start_app
exit

docker exec -it myrabbit3 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbit@rabbit1
rabbitmqctl start_app
exit
```

![image-20231220125645892](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231220125645892.png) 

`docker 部署RabbitMQ集群` 设置为镜像队列模式,集群中的消息数据互相备份

```shell
# 进入任意一个节点
docker exec -it myrabbit1 bash
# 可以设置镜像队列,"^"表示匹配所有队列,即所有队列在各个节点上都会有备份,自动同步到其他队列;
rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'
# 查看集群状态
rabbitmqctl cluster_status
```

`application.yml` 连接到RabbitMQ服务器(集群模式)

```yml
server:
  port: 9002

spring:
  application:
    name: message-publisher

  rabbitmq:
    # rabbitMQ的集群地址
    addresses: 192.168.238.128:5672,192.168.238.128:5673,192.168.238.128:5674
    username: user1
    password: 123456
    virtual-host: /
```

# SpringAMQP

简介 : SpringAMQP基于RabbitMQ封装的一套模版,并且利用SpringBoot对其实现了自动装配;

![image-20231218100151707](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218100151707.png) 

![image-20231218100232387](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218100232387.png) 

*SpringAMQP功能 :*

- 通过配置文件自动声明交换机 队列以及绑定关系
- 基于注解的监听器模式,异步接收消息
- 封装了RabbitTemplate工具,用于发送消息

## Basic Queue

`pom.xml`

```xml
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

`application.yml`  Publisher

```yml
spring:
 # 配置RabbitMQ的地址
  rabbitmq:
    host: 192.168.238.128 # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: admin # 用户名
    password: 123456 # 密码
```

`SpringAMQPSendMessage.java`  向队列simple.queue发送消息

```java
import ...

@Component
public class SpringAMQPSendMessage {

    // 注入RabbitTemplate工具类,发送消息
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private AmqpAdmin amqpAdmin;

    /**
     * 向基础队列发送消息
     */
    public void sendMessageToBasicQueue(){
        // 队列名称
        String queueName = "simple.queue";
        // 创建队列
        Queue queue = new Queue(queueName);
        amqpAdmin.declareQueue(queue);

        // 创建消息
        String message = "Basic Queue Test !";
        // 发送消息
        rabbitTemplate.convertAndSend(queueName,message);
    }
}
```

![image-20231218110413930](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218110413930.png) 



`application.yml` Consumer

```yml
server:
  port: 9001

spring:
  application:
    name: message-consumer


  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: /
```

`SpringAMQPTakeMessage.java` 获取队列simple.queue的消息

```java
import ..
@Component
@Slf4j
public class SpringAMQPTakeMessage {

    @Autowired
    RabbitTemplate rabbitTemplate;

    // 接收消息
    public void takeMessageFromBasicQueue(){
        log.info("接收消息");
        String message = (String)rabbitTemplate.receiveAndConvert("simple.queue");
        System.out.println("SpringAMQP消费者接收到的消息为: " + message);
    }
}
```

![image-20231218111944171](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218111944171.png) 

## Task Queue

简介 : 任务模型,让多个消费者绑定一个队列,共同消费队列中的消息,默认为平均分配;

通过修改配置`spring.rabbitmq.listener.simple.prefetch: 1`根据消费能力分配;

![image-20231218112538196](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218112538196.png) 

`application.yml`  Publisher

```yml
spring:
 # 配置RabbitMQ的地址
  rabbitmq:
    host: 192.168.238.128 # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: admin # 用户名
    password: 123456 # 密码
```

`SpringAMQPSendMessage.java`  向队列simple.queue发送多条消息,模拟消息堆积

```java
import....

@Component
public class SpringAMQPSendMessage {

    // 注入RabbitTemplate工具类,发送消息
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private AmqpAdmin amqpAdmin;

    /**
     * 工作队列发送消息,模拟消息堆积
     */
    public void sendMessageToTaskQueue() throws InterruptedException {
        // 队列名称
        String queueName = "simple.queue";
        // 创建队列
        Queue queue = new Queue(queueName);
        amqpAdmin.declareQueue(queue);

        // 创建消息
        String message = "Basic Queue Test !";
        for (int i = 1; i <= 60 ; i++) {
            message+= i;
            // 发送消息
            rabbitTemplate.convertAndSend(queueName,message);
            // 20ms 发送一次
            Thread.sleep(20);
        }
    }
}
```

![image-20231218113053783](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218113053783.png) 



`application.yml` Consumer

```yml
server:
  port: 9001

spring:
  application:
    name: message-consumer


  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: /

    listener:
     simple:
      # 每次只能获取一条消息，处理完成才能获取下一个消息;能者多劳
      prefetch: 1
```

`SpringAMQPTakeMessage.java` 多个消费者获取队列simple.queue的消息

```java
import ...
@Component
@Slf4j
public class SpringAMQPTakeMessage {

    @Autowired
    RabbitTemplate rabbitTemplate;

    /**
     * 通过sleep(),消费者1的处理速度慢与消费者2,模拟消费者处理能力差距;
     * @throws InterruptedException
     */
    public void takeMessageFromTaskQueue1() throws InterruptedException {
        String message = (String)rabbitTemplate.receiveAndConvert("simple.queue");
        System.out.println("SpringAMQP消费者接1收到的消息为: " + message);
        Thread.sleep(20);
    }

    public void takeMessageFromTaskQueue2() throws InterruptedException {
        String message = (String)rabbitTemplate.receiveAndConvert("simple.queue");
        System.err.println("SpringAMQP消费者接2收到的消息为: " + message);
        Thread.sleep(200);
    }
}
```

## Publish /  Subscribe

`发布订阅模型图 :`

![image-20231218130912371](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218130912371.png) 

*Excahnge的三种类型 :*

- Fanout: 广播,将消息交给所有绑定到交换机的队列
- Direct: 定向,把消息交给符合指定routing key的队列
- Topic: 通配符,把消息交给符合指定routing pattern(路由模式)的队列

Exchange(交换机)只负责转发消息,不具备存储消息的功能;当没有任何队列与Exchange绑定,或者没有符合路由规则的队列,消息就会丢失;

`交换机的接口`

![image-20231218131854048](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218131854048.png) 

## Fanout

简介 : Fanout将消息广播到与之绑定的所有队列,不考虑消息的路由键等;

![image-20231218131526442](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218131526442.png) 

`application.yml` Publisher

```yml
server:
  port: 9002

spring:
  application:
    name: message-publisher

  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: test
```

`FanoutConfig.java` fanout类型交换机配置

```java
import ...

@Configuration
public class FanoutConfig {
    /**
     *  声明交换机banne.fanout  和  队列fanout.queue1 fanout.queue2
     */
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("banne.fanout");
    }

    @Bean
    public Queue Queue1(){
        return  new Queue("fanout.queue1");
    }

    @Bean
    public Queue Queue2(){
        return  new Queue("fanout.queue2");
    }

    /**
     *  交换机和队列进行绑定
     *  banne.fanout交换机 具有队列 fanout.queue1 ,fanout.queue2
     *  所有队列皆具有发送的消息
     */
    @Bean
    public Binding bindingQueue1(Queue Queue1,FanoutExchange fanoutExchange){
        return BindingBuilder.bind(Queue1).to(fanoutExchange);
    }

    @Bean
    public Binding bindingQueue2(Queue Queue2,FanoutExchange fanoutExchange){
        return BindingBuilder.bind(Queue2).to(fanoutExchange);
    }
}
```

`SpringAMQPSendMessage.java` 向交换机banne.fanout发送消息

```java
import ...

@Component
public class SpringAMQPSendMessage {

    // 注入RabbitTemplate工具类,发送消息
    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 向FanoutExchange发送消息
     */
    public void sendMessageToFanoutExchange() throws InterruptedException {
        // 交换机名称
        String exchangeName = "banne.fanout";
        // 创建消息
        String msg = "FanoutExchange Test !";
        Message message = new Message(msg.getBytes());

        // 发送消息 交换机名称 路由键  消息内容
        rabbitTemplate.convertAndSend(exchangeName,"",message);
    }
}
```

![image-20231218143759287](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218143759287.png)  





`application.yml` Consumer

```yml
server:
  port: 9001

spring:
  application:
    name: message-consumer


  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: test
```

`SpringAMQPTakeMessage.java` 通过注解接收消息

```java
import ..

@Component
@Slf4j
public class SpringAMQPTakeMessage {
    // 通过 RabbitListener注解,可以直接读取对应队列的信息 无需调用
    @RabbitListener(queues = "fanout.queue1")
    public void takeMessageFromFanoutQueue1(Message message){
        byte[] body = message.getBody();
        String msg = new String(body);
        System.out.println("fanout.queue1队列中的消息为: "+ msg);
    }

    @RabbitListener(queues = "fanout.queue2")
    public void takeMessageFromFanoutQueue2(Message message){
        byte[] body = message.getBody();
        String msg = new String(body);
        System.out.println("fanout.queue2队列中的消息为: "+ msg);
    }
}
```

![image-20231218144344878](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218144344878.png) 

![image-20231218144359698](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218144359698.png) 

## Direct

简介 : DirectExchange会根据路由键,来选择当前消息由那个队列来订阅;

![image-20231218150421518](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218150421518.png)

`application.yml` Publisher

```yml
server:
  port: 9002

spring:
  application:
    name: message-publisher

  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: test
```

`DirectConfig.java` direct交换机配置

```java
import ....

@Configuration
public class DirectConfig {
    public static final String DIRECT_EXCHANGE_NAME = "banne.direct";
    public static final String QUEUE_A_NAME = "queueA";
    public static final String QUEUE_B_NAME = "queueB";

    /**
     * 创建直连交换机
     * @return DirectExchange
     */
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange(DIRECT_EXCHANGE_NAME);
    }

    /**
     * 创建队列 queueA queueB 
     * @return Queue
     */
    @Bean
    public Queue queueA() {
        return new Queue(QUEUE_A_NAME);
    }

    @Bean
    public Queue queueB() {
        return new Queue(QUEUE_B_NAME);
    }

    /**
     * 将队列queueA与交换机directExchange绑定,并添加路由键为routing.key.a 仅接收具有此路由键的消息
     * @param queueA
     * @param directExchange
     * @return Binding
     */
    @Bean
    public Binding bindingQueueA(Queue queueA, DirectExchange directExchange) {
        return BindingBuilder.bind(queueA).to(directExchange).with("routing.key.a");
    }

    /**
     * 将队列queueB与交换机directExchange绑定,并添加路由键为routing.key.b 仅接收具有此路由键的消息
     * @param queueB
     * @param directExchange
     * @return Binding
     */
    @Bean
    public Binding bindingQueueB(Queue queueB, DirectExchange directExchange) {
        return BindingBuilder.bind(queueB).to(directExchange).with("routing.key.b");
    }
}
```



`SpringAMQPSendMessage.java` 向交换机banne.direct发送消息

```java
import ...

@Component
public class SpringAMQPSendMessage {

    // 注入RabbitTemplate工具类,发送消息
    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 向DirectExchange发送消息
     */
    public void sendMessageToDirectExchange() throws InterruptedException {
        // 交换机名称
        String exchangeName = "banne.direct";
        // 创建消息
        Message message = MessageBuilder.withBody("DirectExchange Test !".getBytes()).build();

        // 发送消息 交换机名称 指定路由键为routing.key.a   消息内容 仅有queueA存在该消息
        rabbitTemplate.convertAndSend(exchangeName,"routing.key.a",message);
    }

}
```

![image-20231218151056166](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218151056166.png) 



`application.yml` Consumer

```yml
server:
  port: 9001

spring:
  application:
    name: message-consumer


  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: test
```

`SpringAMQPTakeMessage.java `通过注解接收消息

```java
package com.banne.consumer.take;

import ...

@Component
@Slf4j
public class SpringAMQPTakeMessage {

    @Autowired
    RabbitTemplate rabbitTemplate; 
    // 通过注解的方式读取队列 queueA中的消息	
    @RabbitListener(queues = "queueA")
    public void takeMessageFromDirectQueueA(Message message){
        byte[] body = message.getBody();
        String msg = new String(body);
        log.info("queueA队列中的消息为: "+ msg);
    }
}
```

![image-20231218151536058](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218151536058.png) 

## Topic 

简介 : Topic类型的交换机与Direct相比,都是根据路由键把消息路由到不同的队列中,只不过Topic类型可以让队列在绑定路由键时使用通配符;

*通配符规则：*

- `#`：代表0或者多个词
- `*`：代表1个词

![image-20231218152915459](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218152915459.png) 

`application.yml` Publisher

```yml
server:
  port: 9002

spring:
  application:
    name: message-publisher

  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: test
```

`TopicConfig.java` topic类型交换机配置

```java
import ...

@Configuration
public class TopicConfig {
    public static final String TOPIC_EXCHANGE_NAME = "banne.topic";
    public static final String QUEUE_C_NAME = "queueC";
    public static final String QUEUE_D_NAME = "queueD";

    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange(TOPIC_EXCHANGE_NAME);
    }

    @Bean
    public Queue queueC() {
        return new Queue(QUEUE_C_NAME);
    }

    @Bean
    public Queue queueD() {
        return new Queue(QUEUE_D_NAME);
    }

    /**
     * 路由键的格式为模糊匹配 *代表有且仅存在一个  #代表多个
     * @param queueC
     * @param topicExchange
     * @return Binding
     */
    @Bean
    public Binding bindingQueueC(Queue queueC, TopicExchange topicExchange) {
        return BindingBuilder.bind(queueC).to(topicExchange).with("topic.key.*");
    }

    @Bean
    public Binding bindingQueueD(Queue queueD, TopicExchange topicExchange) {
        return BindingBuilder.bind(queueD).to(topicExchange).with("topic.key.#");
    }
}
```

`SpringAMQPSendMessage.java ` 向交换机banne.topic发送消息

```java
import ...

@Component
public class SpringAMQPSendMessage {

    // 注入RabbitTemplate工具类,发送消息
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    /**
     * 向TopicExchange发送消息
     */
    public void sendMessageToTopicExchange() throws InterruptedException {
        // 交换机名称
        String exchangeName = "banne.topic";
        // 创建消息
        Message message = MessageBuilder.withBody("TopicExchange Test !".getBytes()).build();
	   
        // 队列 queueC满足一个,queueD皆满足
        // 发送消息 交换机名称 指定路由键为routing.key.abc   消息内容  
        rabbitTemplate.convertAndSend(exchangeName,"topic.key.abc",message);

        // 发送消息 交换机名称 指定路由键为routing.key.abc.123   消息内容
        rabbitTemplate.convertAndSend(exchangeName,"topic.key.abc.123",message);
    }

}
```

![image-20231218154129908](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218154129908.png) 



`application.yml` Consumer

```yml
server:
  port: 9001

spring:
  application:
    name: message-consumer


  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: test
```

`SpringAMQPTakeMessage.java` 通过注解接收消息

```java
package com.banne.consumer.take;

import ...

@Component
@Slf4j
public class SpringAMQPTakeMessage {

    @Autowired
    RabbitTemplate rabbitTemplate; 
    // 通过注解的方式读取队列 queueD中的消息	
    @RabbitListener(queues = "queueD")
    public void takeMessageFromDirectQueueD(Message message){
        byte[] body = message.getBody();
        String msg = new String(body);
        log.info("queueD队列中的消息为: "+ msg);
    }
}
```

![image-20231218154439562](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218154439562.png) 

# Dead Letter Exchange

简介 : 死信交换机是一种用于处理`消息无法被消费、消息队列超过最大长度、被拒绝或者过期的情况机制`;

当消息满足一定条件时,被路由到一个死信交换机上,而不是被直接丢弃;

`死信交换机工作流程`

![image-20231219085241601](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219085241601.png) 

## Expired message 

简介 : 队列中的消息超时未消费,会变成死信消息;

*超时的两种情况 :*

- 消息本身设置了超时时间

```java
// 发送消息时为当前消息设置超时时间 
public void sendMessageToDirectExchange() throws InterruptedException {
        // 交换机名称
        String exchangeName = "banne.direct";
        // 创建消息属性
        MessageProperties messageProperties = new MessageProperties();
        // 设置消息超时为 12秒
        messageProperties.setExpiration("12000");

        // 创建消息 添加设置的消息属性
        Message message = MessageBuilder.withBody("DirectExchange Test !".getBytes())
                                        .andProperties(messageProperties).build();

        rabbitTemplate.convertAndSend(exchangeName,"info",message);
    }
```

![image-20231218165719807](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218165719807.png) 

- 在队列中为消息设置了超时时间

```java
    @Bean
    public Queue queueTTL() {
        // 方式1 new Queue的方式
        Map<String,Object> arguments = new HashMap<>();
        String queueName = "queueTTL";
        // 设置过期时间
        arguments.put("x-message-ttl",30000);
        return new Queue(queueName,true,false,false,arguments);

        // 方式2 构建者模式
        //return QueueBuilder.durable(queueName).withArgument(arguments).build();
```

![image-20231218171059925](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231218171059925.png) 

- 当消息本身和队列皆具有超时时间,选择最小的超时时间;

## Queue length

简介 : 队列本身存在最大长度,当消息的数目超过队列的最大长度时,先进的消息会变成死信消息;

```java
    @Bean
    public Queue queueMaxLength() {
        // 方式1 new Queue的方式
        Map<String,Object> arguments = new HashMap<>();
        String queueName = "queueMaxLength";
	   //设置队列的最大长度,对头的消息会被挤出变成死信消息
        arguments.put("x-max-length", 5);
        return new Queue(queueName,true,false,false,arguments);

        // 方式2 构建者模式
        //return QueueBuilder.durable(queueName).withArgument(arguments).build();
    }
```

## Timeout trigger dead letter

`application.yml`

```yml
server:
  port: 9002

spring:
  application:
    name: message-publisher

  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: dead
```

`RabbitConfig.java`  设置队列中消息过期时间以及交换机和队列的绑定

```java
import ...

@Configuration
public class RabbitConfig {
    public static final String NORMAL_EXCHANGE_NAME = "normal.exchange";
    public static final String NORMAL_QUEUE_NAME = "normal.queue";
    public static final String NORMAL_QUEUE_ROUTE_KEY = "order";

    public static final String DEAD_LETTER_EXCHANGE_NAME = "dead-letter.exchange";
    public static final String DEAD_LETTER_QUEUE_NAME = "dead-letter.queue";
    public static final String DEAD_QUEUE_ROUTE_KEY = "error";

    public static final Integer TIME_TO_LIVE = 20000;

    @Bean
    public DirectExchange normalExchange() {
        return new DirectExchange(NORMAL_EXCHANGE_NAME);
    }

    @Bean
    public Queue normalQueue() {
        // 为队列设置参数
        Map<String,Object> arguments = new HashMap<>();
        // 指定队列的过期时间
        arguments.put("x-message-ttl",TIME_TO_LIVE);
        // 指定死信交换机
        arguments.put("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE_NAME);
        // 指定死信交换机的路由键
        arguments.put("x-dead-letter-routing-key", DEAD_QUEUE_ROUTE_KEY);

        return QueueBuilder.durable(NORMAL_QUEUE_NAME)
                .withArguments(arguments)
                .build();
    }

    @Bean
    public Binding bindingNormalQueue(Queue normalQueue, DirectExchange normalExchange) {
        return BindingBuilder.bind(normalQueue).to(normalExchange).with(NORMAL_QUEUE_ROUTE_KEY);
    }

    @Bean
    public DirectExchange deadLetterExchange() {
        return new DirectExchange(DEAD_LETTER_EXCHANGE_NAME);
    }

    @Bean
    public Queue deadLetterQueue() {
        return new Queue(DEAD_LETTER_QUEUE_NAME);
    }

    @Bean
    public Binding bindingDeadLetterQueue(Queue deadLetterQueue, DirectExchange deadLetterExchange) {
        return BindingBuilder.bind(deadLetterQueue).to(deadLetterExchange).with(DEAD_QUEUE_ROUTE_KEY);
    }
}
```

`SendMessageService.java`

```java
import ...

@Service
@Slf4j
public class SendMessageService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendMessageToNomalExchange(){
        // 设置发送的消息的内容
        Message message = MessageBuilder.withBody("Dead Message !".getBytes()).build();

        // 发送消息
        rabbitTemplate.convertAndSend("normal.exchange","order",message);

        log.info("消息发送成功: " + new Date());
    }
}
```

![image-20231219085441964](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219085441964.png) 

![image-20231219085524877](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219085524877.png) 



`RabbitConfig.java` 设置交换机和队列的绑定

```java
import ...
@Configuration
public class RabbitConfig {
    ....
}
```

`SendMessageService.java` 消息本身设置超时时间

```java
import ...

@Service
@Slf4j
public class SendMessageService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendMessageToNomalExchange(){
        // 设置此消息过期时间
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setExpiration("20000");
        // 设置发送的消息的内容
        Message message = MessageBuilder.withBody("Dead Message !".getBytes())
                .andProperties(messageProperties)
                .build();

        // 发送消息
        rabbitTemplate.convertAndSend("normal.exchange","order",message);

        log.info("消息发送成功: " + new Date());
    }
}
```

![image-20231219085441964](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219085441964.png) 

![image-20231219090419179](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219090419179.png) 

# Delay Queue

简介 : RabbitMQ本身不支持延迟队列,通过`TTL和DLX`的结合方式实现消息的延迟投递;消费者从死信队列中获取的消息就是延迟信息;

`RabbitMQ中延迟队列的工作流程`

![image-20231219144459067](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219144459067.png) 

`application.yml`

```yml
server:
  port: 9002

spring:
  application:
    name: message-publisher

  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: dead
```

`DelayConfig.java`

```java
import ...

@Configuration
public class DelayConfig {
    public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";

    public static final String NORMAL_QUEUE_NAME = "normal.queue";
    public static final String DEADED_QUEUE_NAME = "dead.queue";
    
    // 设置延迟交换机
    @Bean
    public DirectExchange directExchange(){
        return new DirectExchange(DELAYED_EXCHANGE_NAME);
    }

    // 正常队列
    @Bean
    public Queue normalQueue() {
        // 设置消息过期的时间 死信交换机 死信队列
        return QueueBuilder.durable(NORMAL_QUEUE_NAME)
                .ttl(20000)
                .deadLetterExchange(DELAYED_EXCHANGE_NAME).deadLetterRoutingKey("error")
                .build();
    }

    @Bean
    public Binding bindingNormalQueue(Queue normalQueue, DirectExchange delayedExchange) {
        return BindingBuilder.bind(normalQueue).to(delayedExchange).with("order");
    }

    // 死信队列
    @Bean
    public Queue deadedQueue(){
        return  QueueBuilder.durable(DEADED_QUEUE_NAME).build();
    }

    @Bean
    public Binding bindingDeadedQueue(Queue deadedQueue,DirectExchange delayedExchange){
        return BindingBuilder.bind(deadedQueue).to(delayedExchange).with("error");
    }
}
```

`SendMessageService.java`

```java
import ...

@Service
@Slf4j
public class SendMessageService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendMessageToDelayExchange(String message,long delayMillis){
        // 设置发送的消息内容
        MessageBuilder.withBody("Delay Message !".getBytes()).build();

        // 发送消息
        rabbitTemplate.convertAndSend("delayed.exchange","order",message);

        log.info("消息发送成功 ! 当前时间" + new Date() );
    }
}
```

![image-20231219150130657](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219150130657.png) 

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: message-consumer


  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: dead
```

`TakeMessageService.java` 从死信队列中获取消息,形成延迟队列的效果;

```java
    // 获取dead.queue队列的消息就是延迟20s发送的信息
    @RabbitListener(queues = "dead.queue")
    public void takeMessageFromDelayQueueD(Message message){
        byte[] body = message.getBody();
        String msg = new String(body);
        log.info("dead.queue队列中的消息为: "+ msg);
    }
```

![image-20231219150135742](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219150135742.png) 

# Exchange Attribute

![image-20231219182053237](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219182053237.png) 

*交换机的常见属性 :*

- Name: 交换机的名称
- Type: 交换机的类型 fanout、direct、topic、headers;
- Durability: 持久化(默认为持久化),声明交换机是否持久化,代表交换机在服务器重启后任然还是存在的;
- Auto delete: 自动删除(默认不自动删除),交换机中所有的队列解绑后,就会自动删除该交换机;
- Internal: 内部使用(默认为NO),客户端无法直接发送消息到此交换机,它只能用于交换机之间的绑定;
- Arguments: 属性 "alternate-exchange"备用交换机;

```java
// 交换机常用属性设置  
    @Bean
    public DirectExchange directExchange(){
        String exchangeName = "confirm.exchange";
        boolean durable = true;
        boolean autoDelete = true;
        Map<String,Object> arguments = new HashMap<>();
        arguments.put("alternate-exchange","alter.exchange");
        return new DirectExchange(exchangeName,durable,autoDelete,arguments);
    }
```

## Standby exchange

简介 : 消息经过交换机准备路由到队列时,发现没有对应的队列可以匹配该路由键,在RabbitMQ中默认会丢弃消息;通过设置交换机的属性参数"alternate-exchange",可以创建备用交换机,来接收无法此消息,记录日志或者发送报警信息;

`application.yml`

```yml
server:
  port: 9002

spring:
  application:
    name: message-publisher

  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: dead
```

`DefaultConfig.java` 配置备用交换机

```java
package com.banne.publisher.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class TestConfig {

    @Bean
    public DirectExchange directExchange(){
        String exchangeName = "default.exchange";
        // 设置备用交换机
        Map<String,Object> arguments = new HashMap<>();
        arguments.put("alternate-exchange","alter.exchange");
        return new DirectExchange(exchangeName,true,true,arguments);
    }

    @Bean
    public Queue confirmQueue(){
        return QueueBuilder.durable("default.queue").build();
    }

    @Bean
    public Binding bindingConfirmQueue(Queue confirmQueue,DirectExchange directExchange){
        return BindingBuilder.bind(confirmQueue).to(directExchange).with("default");
    }

    // 创建备用交换机,备用交换机一般为 FanoutExchange
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("alter.exchange");
    }

    // 创建备用队列
    @Bean
    public Queue alterQueue(){
        return QueueBuilder.durable("alter.queue").build();
    }

    @Bean
    public Binding bingingAlterQueue(Queue alterQueue,FanoutExchange fanoutExchange){
        return BindingBuilder.bind(alterQueue).to(fanoutExchange);
    }
}
```

`SendMessageService.java`

```java
import ...

@Service
@Slf4j
public class SendMessageService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendMessageToDefaultExchange(){
        Message message = MessageBuilder.withBody("Default Message !".getBytes()).build();
        // 发送的路由键不配默认交换机
        rabbitTemplate.convertAndSend("default.exchange","test", message);
    }
}
```

![image-20231219184806311](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219184806311.png) 

# Queue Attribute

![image-20231219185852168](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219185852168.png) 

*队列的常用属性 :*

- Type: 队列类型
- Name: 队列名称
- Durability: 声明持久化,队列在服务器重启服务后是否还存在;
- Auto delete: 自动删除(默认关闭),开启时当没有消费者连接此队列时,队列会进行删除;
- Exclusive: 队列只对首次声明它的连接可见,并且在断开连接时自动删除,基本上不进行设置;
- Arguments: 队列的参数
  - "x-expires": Number  指定时间未访问,队列自动删除
  - "x-message-ttl":Number 指定时间内,队列的消息会自动取消
  - "x-overflow":String 队列消息溢出时,默认Drop Head删除头部消息,可设置为Reject Publish删除尾部消息;
  - "x-single-active-consumer" 指定队列为单消费者,默认为false
  - "x-dead-letter-exchange":String 指定消息的死信交换机
  - "x-dead-letter-routing-key":String 指定死信交换机的路由键 
  - "x-max-length":Number 指定队列最大长度,可以搭配"x-overflow"属性设置覆盖原则

# Reliability

简介 : 消息的可靠性投递确保消息投递的每一个环节都要成功,失败会返回对应的日志方便修复;

![image-20231219204336050](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219204336050.png) 

## Confirm

简介 : 消息的Confirm确认机制,生产者投递消息后,到达服务器Broker中的exchange交换机,会给生产者发送应答,来确定这条消息是否正确的发送到Broker的Exchange中,确保消息可靠性的投递;

`Confirm确认机制`

![image-20231219165158797](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219165158797.png)  

`application.yml`

```yml
server:
  port: 9002

spring:
  application:
    name: message-publisher

  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: dead
    # 开启确认模式
    publisher-confirm-type: correlated
```

`MyConfirmCallback.java` 

```java
import ...    
@Configuration
@Slf4j
public class MyConfirmCallback implements RabbitTemplate.ConfirmCallback {

    /**
     *
     * @param correlationData 相关联的数据
     * @param ack   是否正确到达Broker中的交换机
     * @param cause 未到达交换机的原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        log.info("发送的关联属性值为: " + correlationData.getId());
        if (ack){
            log.info("消息从Publisher正常发送到交换机 !");
            return;
        }

        log.error("消息从Publisher未正常发送到交换机,出现的原因为: " + cause);
    }
}
```

`RabbitConfig.java`

```java
import java.util.Map;

@Configuration
public class RabbitConfig {

    @Bean
    public DirectExchange directExchange(){
        return new DirectExchange("confirm.exchange");
    }

    @Bean
    public Queue confirmQueue(){
        return QueueBuilder.durable("confirm.queue").build();
    }

    @Bean
    public Binding bindingConfirmQueue(Queue confirmQueue,DirectExchange directExchange){
        return BindingBuilder.bind(confirmQueue).to(directExchange).with("info");
    }
}
```

`SendMessageService.java `方法1: 自定义类实现RabbitTemplate.ConfirmCallback接口

```java
import ...

@Service
@Slf4j
public class SendMessageService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    // 自定义的确认回调方法
    private MyConfirmCallback myConfirmCallback;


    @PostConstruct // Bean在初始化的时候,会调用一次该方法;
    public void init(){
        // 设置RabbitTemplate的确认回调方法,Publisher发送消息给Exchange服务器自动回调该方法;
        rabbitTemplate.setConfirmCallback(myConfirmCallback);
    }


    public void sendMessageToConfirmExchange(){
        // 设置发送的消息的内容
        Message message = MessageBuilder.withBody("Confirm Message !".getBytes()).build();

        // 设置关联属性
        CorrelationData correlationData = new CorrelationData();
        correlationData.setId("Confirm_ID_1234");

        // 发送消息
        rabbitTemplate.convertAndSend("confirm.exchange","info",message,correlationData);
    }
}
```

![image-20231219171643806](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219171643806.png) 

`SendMessageService.java` 方法2: 通过Lambda表达式,实现ConfirmCallback接口的confirm方法

```java
import ...

@Service
@Slf4j
public class SendMessageService {
    @Autowired
    private RabbitTemplate rabbitTemplate;


    @PostConstruct // Bean在初始化的时候,会调用一次该方法;
    public void init(){
        // 设置RabbitTemplate的确认回调方法,通过Lambda表达式实现;
        rabbitTemplate.setConfirmCallback((correlationData,ack,cause)->{
            log.info("发送的关联属性值为: " + correlationData.getId());
            if (ack){
                log.info("消息从Publisher正常发送到交换机 !");
                return;
            }

            log.error("消息从Publisher未正常发送到交换机,出现的原因为: " + cause);
        });
    }
   
    public void sendMessageToConfirmExchange(){
        // 设置发送的消息的内容
        Message message = MessageBuilder.withBody("Confirm Message !".getBytes()).build();

        // 设置关联属性
        CorrelationData correlationData = new CorrelationData();
        correlationData.setId("Confirm_ID_1234");

        // 发送消息
        rabbitTemplate.convertAndSend("confirm.exchange1","order",message,correlationData);
    }
}
```

![image-20231219170940488](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219170940488.png) 

## Return

简介 : 消息的Return退回机制,交换机将消息投递给队列,投递成功不会有任何应答,投递失败会发送应答,确保消息的可靠性;

`Return退回机制`

![image-20231219185739609](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219185739609.png) 

`application.yml`

```yml
server:
  port: 9002

spring:
  application:
    name: message-publisher

  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: dead
    # 开启退回模式
    publisher-returns: true
```

`MyConfigReturnsCallback.java` 

```java
import ...

@Configuration
@Slf4j
public class MyConfigReturnsCallback implements RabbitTemplate.ReturnsCallback {
    /**
     * 
     * @param returnedMessage 发送失败的原因
     */
    @Override
    public void returnedMessage(ReturnedMessage returnedMessage) {
        log.error("交换机未正常将消息发送给队列,出现的原因为: " + returnedMessage.getReplyText());
    }
}
```

`RabbitConfig.java`

```java
import java.util.Map;

@Configuration
public class RabbitConfig {

    @Bean
    public DirectExchange directExchange(){
        return new DirectExchange("confirm.exchange");
    }

    @Bean
    public Queue confirmQueue(){
        return QueueBuilder.durable("confirm.queue").build();
    }

    @Bean
    public Binding bindingConfirmQueue(Queue confirmQueue,DirectExchange directExchange){
        return BindingBuilder.bind(confirmQueue).to(directExchange).with("info");
    }
}
```

`SendMessageService.java` 方法1: 自定义类实现RabbitTemplate.ReturnsCallback接口

```java
import ...

@Service
@Slf4j
public class SendMessageService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    // 自定义的退回回调方法
    @Autowired
    private MyConfigReturnsCallback myConfigReturnsCallback;

    @PostConstruct // Bean在初始化的时候,会调用一次该方法;
    public void init(){
        //设置RabbitTemplate的退回回调方法,通过自定义类来实现
        rabbitTemplate.setReturnsCallback(myConfigReturnsCallback);
    }


    public void sendMessageToConfirmExchange(){
        // 设置发送的消息的内容
        Message message = MessageBuilder.withBody("Confirm Message !".getBytes()).build();

        // 设置关联属性
        CorrelationData correlationData = new CorrelationData();
        correlationData.setId("Confirm_ID_1234");

        // 发送消息
        rabbitTemplate.convertAndSend("confirm.exchange","info",message,correlationData);
    }
}
```

![image-20231219171900946](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219171900946.png) 

`SendMessageService.java` 方法2: 通过Lambda表达式,实现ReturnsCallback接口的returnedMessage方法

```java
import ...

@Service
@Slf4j
public class SendMessageService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct // Bean在初始化的时候,会调用一次该方法;
    public void init(){
        //设置RabbitTemplate的退回回调方法,通过Lambda表达式实现
        rabbitTemplate.setReturnsCallback((returnedMessage)->{
            log.error("交换机未正常将消息发送给队列,出现的原因为: " + returnedMessage.getReplyText());
        });
    }
    public void sendMessageToConfirmExchange(){
        // 设置发送的消息的内容
        Message message = MessageBuilder.withBody("Confirm Message !".getBytes()).build();

        // 设置关联属性
        CorrelationData correlationData = new CorrelationData();
        correlationData.setId("Confirm_ID_1234");

        // 发送消息
        rabbitTemplate.convertAndSend("confirm.exchange","order1",message,correlationData);
    }
}
```

![image-20231219171413676](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219171413676.png) 

## Durability

简介 :  RabbitMQ中设置交换机 队列 消息的持久化可提高消息的可靠性,服务器重启时它们的数据信息不会丢失,确保安全性;

ExchangeBuilder   QueueBuilder  MessageBuilder  构建者模式创建,默认配置持久化; 

![image-20231219205216294](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219205216294.png) 

![image-20231219205610005](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219205610005.png) 

![image-20231219210556882](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231219210556882.png) 

## Affirm Message

简介 : RabbitMQ中默认情况下接收消息后会自动确认删除消息;

通过配置`spring.rabbitmq.listener.simple.acknowledge-mode: manual`开启手动确认消息;业务中出现异常可不确认消息,可重新投放或发送到死信队列,提升消息的可靠性;

`application.yml`

```yml
server:
  port: 9001

spring:
  application:
    name: message-consumer


  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: test
    listener:
      simple:
        acknowledge-mode: manual # 开启手动确认模式
```

`SendMessageService.java` 手动确认消息;出现异常时不再投放,放入死信队列中 

```java
    @RabbitListener(queues = "normal.queue")
    public void takeMessageFromNormalQueueD(Message message, Channel channel) throws IOException {
        // 获取消息的属性值
        MessageProperties messageProperties = message.getMessageProperties();
        // 获取消息的唯一标识
        long deliveryTag = messageProperties.getDeliveryTag();

        try{
            String msg = new String(message.getBody());
            log.info("queueD队列中的消息为: "+ msg);
            // 配置文件改为手动确认,消费者手动确认消息 参数为 1.消息标识符 2.确认消息(false为确认当前消息,true批量确认消息)
            channel.basicAck(deliveryTag,false);
        }catch (Exception e){
            log.error("接收消息出现问题: " ,e.getMessage());
            
            //方法1: 出现异常消费者手动不确认该消息(可批量处理),参数1: 消息标识符 参数2 false处理当前消息 参数3 重新入队投放 
            // 不继续投放; 直接进入死信队列
            channel.basicNack(deliveryTag,false,false);
            
            //方法2: 出现异常消费拒绝此消息(只能处理单条消息) 参数1: 消息标识符  参数2: 不继续投放 直接进入死信队列
            channel.basicReject(deliveryTag,false);
        }
   }
```

# Idempotency

简介 : 对一个资源,不管你请求一次还是请求多次,对资源本身造成的影响应该是相同的,不能因为重复的请求对该资源重复造成影响;

消息的幂等性同一个消息,第一次接收,正常处理业务,第二次接收,将不再执行业务;

`application.yml `Publisher

```yml
server:
  port: 9002

spring:
  application:
    name: message-publisher

  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: dead
```

`Order.java` 实现Serializable接口,添加注解创建构建者模式;

```java
import ...

@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder(toBuilder = true)
public class Order implements Serializable {

    private String  id;

    private String  orderName;

    private String  orderInfo;
}
```

`SendMessageService.java` 发送两条一样的订单消息

```java
import ...
@Service
@Slf4j
public class SendMessageService {
    // 注入RabbitTemplate工具类,发送消息
    @Autowired
    private RabbitTemplate rabbitTemplate;


    // 注入ObjectMapper工具类,进行序列化/反序列化(json格式)
    @Autowired
    private ObjectMapper objectMapper;

    // 向confirm.queue队列发送两条消息
    public void sendMessageToConfirmExchange() throws JsonProcessingException {
        // 发送第一个订单
        {
            // 创建订单
            Order order = Order.builder().id("6379").orderInfo("购买VivoX100 Pro").orderName("手机订单").build();
            // 序列化为json格式字符串
            String str = objectMapper.writeValueAsString(order);
            // 创建消息进行发送
            Message message = MessageBuilder.withBody(str.getBytes()).build();
            // 模拟发送两次相同的订单
            for (int i = 0; i < 2; i++) {
                rabbitTemplate.convertAndSend("confirm.exchange", "info", message);
            }
        }
    }
}
```

![image-20231220114545101](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231220114545101.png) 

`pom.xml` 导入Redis相关依赖

```xml
		<!--导入Redis相关依赖-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>

		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-pool2</artifactId>
		</dependency>
```

`application.yml` Consumer

```yml
server:
  port: 9001

spring:
  application:
    name: message-consumer


  rabbitmq:
    addresses: 192.168.238.128
    port: 5672
    username: admin
    password: 123456
    virtual-host: dead

  # 设置redis配置
  data:
    redis:
      host: localhost
      port: 6379
      database: 0

      lettuce:
        pool:
          max-active: 8
          max-wait: -1
          max-idle: 8
          min-idle: 0
```

`TakeMessageService.java` 接收消息进行处理,接收的消息通过全局唯一Id存入Redis中,确保消息是第一次接收,仅处理一次;

```java
import ...

@Component
@Slf4j
public class SpringAMQPTakeMessage {
    //注入ObjectMapper工具类,处理序列化/反序列化
    @Autowired
    private ObjectMapper objectMapper;

    //注入RedisTemplate工具类,操作redis数据库
    @Autowired
    private RedisTemplate redisTemplate;

    // 获取confirm.queue队列的消息进行处理,相同的订单只处理一次
    @RabbitListener(queues = "confirm.queue")
    public void takeMessageFromDelayQueueD(Message message) throws IOException {
        byte[] body = message.getBody();
        String msg = new String(body);
        // 进行反序列化,将json格式的字符串转换为对象
        Order order = objectMapper.readValue(message.getBody(), Order.class);
        // 确保相同的订单只处理一次,将订单Id存入到redis数据库中
        Boolean result = redisTemplate.opsForValue().setIfAbsent("confrim.queue." + order.getId(), order.getId());
        // 存入成功,当前消息第一次进入可进行处理
        if (result){
            // TODO 订单存入数据库中
            log.info("队列中的消息为: " + order.toString());
            return;
        }
        log.error("队列中的消息已经被处理 !");
    }
}
```

![image-20231220114355093](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231220114355093.png) 

