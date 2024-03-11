# 2024-3-04

- JUC P107
- SpringBoot 复习
  - 常用组件注解 
    - @Component @Controller @Service  @Repository    
  - 数据绑定注解
    - @ConfigurationProperties  +  @EnableConfigurationProperty
  - 自定义日志   Log4j2 通过XML文件自定义日志输出格式
    - @Slf4j  
  - 异常处理器   统一处理异常
    - @ControllerAdvice  +  @ExceptionHandler

# 2024-3-05

- SpringMVC  P32
  - @RequestMapping(类注解 方法注解)  设置请求映射路径
  - 请求参数注解(参数注解)    
    - @RequestParm (普通类型参数  POJO参数)    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")日期类型参数   
    - @RequestBody(json 格式参数 application/json)
  - 响应结果注解(方法注解)
    - @ResponseBody  设置当前控制器返回值作为响应体
  - REST风格  
    -  @PathVariable("id")  -->  user/{id}   接收路径变量
    - GetMapping("/users")   ===  RequestMapping(vlaue = "/users",method = RequestMethod.GET )
  - 异常处理器
    - @ControllerAdvice  +  @ExceptionHandler
    - 自定义 BusinessException(业务异常)     SystemException(系统异常)  记录日志  其他异常 Exception
  - 拦截器  底层机制为AOP
    - 实现HandlerInterceptor接口 
    - 配置类,实现WebMvcConfigur接口 / 继承WebMvcConfigurationSupport类 进行拦截器的注册

- SpringMVC  Interview  T10

  - SpringMVC的执行流程

  ![image-20240305090854310](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240305090854310.png) 



- Spring P39

  - OCP开闭原则   DIP依赖倒置原则

  - IOC设计模式  --> 将对象的创建,对象和对象之间的关系,通过第三方容器来创建和管理
  - Spring通过DI实现  IOC思想
  - Bean Scope 作用域范围 singleton(默认,单实例)   prototype(多实例)   

- 了解 Take Out项目 / EliteSelection 项目中关于SpringMVC中部分 (GlobalExceptionHandler异常处理器   JwtTokenAdminInterceptor拦截器)

# 2024-3-06

- JWT  P12     添加项目简历 (1)通过用户的JWT令牌来验证用户的身份,确保用户的访问是合法的  实现登录以及用户的认证
  - 加密后的数据载体,各个应用之间的数据传输
  - Header(alg)  Payload(注册声明 / 公共 私有声明)  Signature(Header + Payload进行Base64编码 + 私有秘钥进行加密)
  - 常用场景: 身份的验证
- 创建Springboot项目后端通用框架 

# 2024-3-07

- Spring  P53

  - Gof23种设计模式    工厂方法设计模式(简单工厂设计模式)   

    - 简单工厂设计模式  生产和消费进行分离  违背了OCP原则
      - 抽象产品  Weapon
      - 具体产品  Gun Tank
      - 工厂类角色 WeaponFactory 统一实现具体产品的创建
    - 工厂方法设计模式  实现工厂和角色的一一对应 体现了OCP原则
      - 抽象的产品 Weapon 
      - 具体的产品  Gun  Tank
      - 抽象的工厂 WeaponFactory
      - 具体的工厂 GunFactory  TankFactory
    - 抽象工厂设计模式图

    ![image-20240307210739107](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240307210739107.png) 

  - Bean的实例化方式

    - 构造器方法实例化
    - 简单工厂模式实例化 

    ```xml
    <!-- 调用工厂Bean的静态方法 创建Bean对象 -->
    <bean id="vipBean" class="com.powernode.spring6.bean.VipFactory" factory-method="get"/>
    ```

    - Factory-Bean实例化

    ```xml
    <bean id="orderFactory" class="com.powernode.spring6.bean.OrderFactory"/>
    <!-- 调用工厂Bean的方法 get方法 创建Bean对象 -->
    <bean id="orderBean" factory-bean="orderFactory" factory-method="get"/>
    ```

    - Factory-Bean接口实例化        SimpleDateFormat 格式化日期类

    ```xml
    <!-- 工厂Bean实现FactoryBean接口 重写其中方法 自动调用进行创建Bean对象 -->
    <bean id="personBean" class="com.powernode.spring6.bean.OrderFactory"/>
    ```

    - BeanFactory(创建Bean对象) 和  FactoryBean(辅助Spring创建其他Bean对象) 的区别

  - Bean的声明周期

    - 五步 --> 实例化Bean  Bean属性赋值  初始化Bean(自定义InitBean函数)  使用Bean  销毁Bean(自定义destoryBean函数)

    ```xml
        <!-- init-method属性指定初始化方法  destroy-method属性指定销毁方法。-->
        <bean id="userBean" class="com.powernode.spring6.bean.User" init-method="initBean" destroy-method="destroyBean">
            <property name="name" value="zhangsan"/>
        </bean>
    ```

    - 七步    实现BeanPostProcessor类，并且重写before和after方法

    ![image-20240307213231623](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240307213231623.png) 

    - 十步   Aware相关接口为BeanNameAware, BeanClassLoaderAware, BeanFactoryAware

    ![image-20240307213323462](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240307213323462.png) 

    - singleton作用域的Bean是完整的   prototype作用域Bean没有销毁前和销毁流程

  - ```java
        public void testBeanRegister(){
            // 自己new的对象
            User user = new User();
            System.out.println(user);
    
            // 创建 默认可列表BeanFactory 对象
            DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
            // 注册Bean
            factory.registerSingleton("userBean", user);
            // 从spring容器中获取bean
            User userBean = factory.getBean("userBean", User.class);
            System.out.println(userBean);
        }
    ```

# 2024-3-08

- Spring  P80
  - 循环依赖   A对象中有B的属性,B对象中有A的属性
    - singleton + setter 模式下循环依赖没有任何问题   实例化对象(立刻曝光)和对象的属性赋值分别为两个阶段
      - 一级缓存(实例化Bean并赋值)  二级缓存(实例化Bean)  三级缓存(工厂Bean)
    - prototype + setter 模式循环依赖存在问题
    - singleton + 构造器模式存在循环依赖问题    实例化对象的过程和对象属性赋值的过程没有分离开，必须在一起完成导致的
    - Spring通过 singleton + setter模式解决循环依赖的原因
  - P62 - P69  手写Spring的IOC容器的Setter注入框架    解析XML文件 + 工厂模式 + 反射机制
  - Spring IOC的注解开发       工厂模式 + 反射机制
    - 声明Bean的注解  实例化Bean对象
      - @Component
      - @Controller
      - @Service
      - @Repository
    - Bean对象属性注入的注解  
      - @Value  简单类型注入
      - @Autowired 非简单类型 默认根据类型进行注入  搭配@Qualifier 可以设置为根据名称进行注入
      - @Resource  非简单类型 默认根据名称进行注入,没有名称根据属性名称注入 最后才会根据类型进行注入 (JDK扩展包提供) 
      - @Configuration  + @Component  --> 实现纯注解开发
  - @Resource 和 @Autowired 注解的区别
- 后端项目通用模板搭建

# 2024-3-09

- Spring  P117

  - Gof23种设计模式  代理模式  结构设计模式

    ![image-20240309135318660](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240309135318660.png) 

    - 目标对象  代理对象   公共接口

    - 代理模式的应用场景 :

      - 程序中A对象和B对象无法直接交互
      - 程序中进行功能的增加
      - 对目标对象的保护,代码复用的提升

    - 代理模式区分 :

      - 静态代理模式  --> OrderServie(公共接口)  OrderServiceImpProxy(代理对象)  OrderServiceImp(目标对象)

      - 动态代理模式 --> 减少代理类的使用,解决代码复用问题  

        - JDK动态代理机制只能代理接口   

          Proxy.newProxyInstance(classLoader,类接口,调用处理器)  

          调用处理器实现InvocationHandler接口重写invoke(代理对象,目标方法,目标参数)

        - CGLIB可代理接口和类,底层通过继承方法实现(底层具有小而快的字节码处理器框架ASM)

  - 面向切面编程 AOP  业务逻辑无关的代码称为切面

    - 切面通常指的是业务系统服务中,与业务逻辑无关的服务比如 日志记录 事务管理 安全等,也称为交叉业务; 交叉业务几乎是通用的
    - AOP 面向切面编程将与核心业务无关的代码独立抽取出来,形成独立的组件,然后横向交叉的方式应用的业务流程中的过程为AOP
    - 连接点  切入点  通知 切面(切入点 + 通知) 织入  Proxy  Target
    - @Aspect 切面类注解  @AroundAdvice("execution(访问修饰符 返回类型 全类名 方法名(参数))") 

- Hutool工具类   Commons工具类  Redis Session分布式登录的原理

- 后端项目通用模板搭建  

# 2024-3-10

- 后端通用模版完成 80%
- 了解HuTool工具包  https://doc.hutool.cn/pages/index/#%F0%9F%93%9A%E7%AE%80%E4%BB%8B 
