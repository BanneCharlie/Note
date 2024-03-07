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

- Spring  P61
- ![image-20240307184228309](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240307184228309.png)
