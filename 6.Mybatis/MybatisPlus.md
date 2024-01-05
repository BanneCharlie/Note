# Info

简介 : MybatisPlus集成了Mybatis的所有对于单表的功能,并且实现了自动装配的效果;

`pom.xml`

```xml
<!--注入 MybatisPlus 依赖-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.4</version>
        </dependency>
```

`Entity --> User`

```java
import ...

/* MybatisPlus将实体类信息转换成数据库信息的规则:
    1.类名驼峰转下划线为表名
    2.名为id的字段作为主键
    3.变量名为驼峰转下划线为表的字段名
*/
@Data
public class User {
    // id为数据库主键,设置为自增长类型
    @TableId(value = "id",type = IdType.Auto)
    private Long id;

    private String username;

    private String password;

    private String phone;

    private String info;

    private Integer status;

    private Integer balance;

    private LocalDateTime createTime;

    private LocalDateTime updateTime;
}
```

`Mapper -->  UserMapper`

```java
import ...
@Mapper
// Mapper接口继承BaseMapper<E> 泛型必须指定;  无需再通过.xml文件,编写sql语句;
// MybatisPlus通过扫描实体类,并基于反射获取实体类信息作为数据库表信息    
public interface UserMapper extends BaseMapper<User> {}
```

`Config --> application.yaml`

```yaml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/mp?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456

# mybatisplus 常见配置
mybatis-plus:
  type-aliases-package: com.banne.mybatisplus.domian # 别名扫描包
  mapper-locations: "classpath*:/mapper/**/*.xml" # 指定扫描Mapper.xml文件地址
  configuration:
    map-underscore-to-camel-case: true # 是否开启下划线和驼峰的映射

    global-config:

    db-config:
      id-type: auto # 设置 主键id的 IdType类型
      update-strategy: not_null # 更新策略: 只更新非空字段
```

`Test --> MybatisPlusApplicationTests `

```java
import ...

@SpringBootTest
class MybatisPlusApplicationTests {

    @Autowired
    UserMapper userMapper;

    @Test
    void insert() {
        User user = new User();
        user.setId(20L);
        user.setUsername("Banne");
        user.setPhone("18816243127");
        user.setPassword("123456");
        user.setInfo("{\"age\": 29, \"intro\": \"伏地魔\", \"gender\": \"male\"}");
        user.setStatus(1);
        user.setBalance(20000);
        // 父类BaseMapper的insert(entity)
        userMapper.insert(user);
    }

    @Test
    void delete(){
        // 父类BaseMapperdedeleteById(int)
        userMapper.deleteById(20L);
    }

    @Test
    void update(){
        User user = new User();
        user.setId(20L);
        user.setUsername("Ron");
        // 父类BaseMapper的updateById(entity)
        userMapper.updateById(user);
    }

    @Test
    void select(){
        // 父类BaseMapper的selectById(int)
        User user = userMapper.selectById(20L);
        System.out.println(user.toString());
    }

}
```

`常见注解 :`

![image-20231106110259442](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231106110259442.png) 

# Core

## Conditional Constructor

简介 : MybatisPlus支持各种复杂的where条件,通过`条件构造器`进行实现;

`条件构造器类图 :`

![image-20231106113947524](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231106113947524.png) 

*条件构造器的用法 :*

- QueryWrapper和LambdaQuery通常用来作为 select、update 、delete 的where条件部分;
- UpdateWrapper和LambdaWrapper通过只有在set语句比较特殊时使用;

`Test --> MybatisPlusApplicationTests`

```java
import ...

@SpringBootTest
class MybatisPlusApplicationTests {

    @Autowired
    UserMapper userMapper;

    @Test
    void queryWrapperTest(){
        // 执行sql: 查询名字中具有o,而且存款大于等于 1000 元人的 id username info balance字段
        // 1.创建 QueryWrapper
        QueryWrapper wrapper = new QueryWrapper<User>()
                .select("id","username","info","balance")
                .like("username","o")
                .ge("balance",1000);
        // 2.查询
        List<User> users = userMapper.selectList(wrapper);

        users.forEach(System.out::println);
    }

    @Test
    void LambdaQueryWrapperTest(){
        // 执行sql: 查询名字中具有o,而且存款大于等于 1000 元人的 id username info balance字段
        // 1.创建 QueryLambdaWrapper
        LambdaQueryWrapper wrapper = new LambdaQueryWrapper<User>()
                .select(User::getId,User::getUsername,User::getInfo,User::getBalance)
                .like(User::getUsername,"o")
                .ge(User::getBalance,10000);

        // 2.查询
        List<User> users = userMapper.selectList(wrapper);

        users.forEach(System.out::println);
    }



    @Test
    void updateWrapperTest1() {
        // 执行sql: 修改用户名为 jack 用户的余额为 2000
        // 1.需要更新的数据
        User user = new User();
        user.setBalance(2000);
        // 2.创建 UpdateWrapper
        UpdateWrapper<User> wrapper = new UpdateWrapper<User>()
                .eq("username", "jack");
        // 3.执行更新
        userMapper.update(user,wrapper);
    }
    
    
    @Test
    void updateWrapperTest2() {
        // 执行sql: id为1 2 4 将balance减少200
        // 1.更新的条件
  	   List<Long> ids = List.of(1,2,4);
        // 2.创建 UpdateWrapper
        UpdateWrapper<User> wrapper = new UpdateWrapper<User>()
                .setsql("balance = balance - 200")
                .in("id",ids)
        // 3.执行更新
        userMapper.update(null,wrapper);
    }
}
```

## Custom Sql

简介 : 可以利用MybatisPlus的Wrapper在业务层构建复杂的where条件,然后在DAO层自定义sql语句,简化sql的复杂程度;

`Service层`

```java
    @Autowired
    UserMapper userMapper;

    @Test
    void customSqlWrapperTest(){
        List<Long> ids = List.of(1L, 2L, 4L);
        int amount = 300;
        // 1. 构建条件
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>().in(User::getId,ids);
        // 2. 自定义SQL方法调用
        userMapper.updateBalanceByIds(wrapper,amount);
    }
```

`Mapper 层`

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    // mapper方法参数中Param注解声明wrapper变量名称必须为 ew
    void updateBalanceByIds(@Param("ew") LambdaQueryWrapper wrapper, @Param("amount") int amount);
}
```

```xml
<update id="updateBalanceByIds">
        <!-- where条件的固定格式 ${ew.customSqlSegment}-->
        update user  set balance = balance - #{amount}  ${ew.customSqlSegment}
</update>
```

## IService

简介 : IService接口对于BaseMapper接口存在更多的方法,更加方便操作者的使用;

`Service常用方法 :`

![image-20231106152625727](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231106152625727.png) 

`IService实现流程 :` 

自定义Service接口继承IService接口;自定义Service类继承ServiceImp类;

![image-20231106150249084](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231106150249084.png) 

`Service`

```java
import ....
// 自定义Service接口继承IService接口    
public interface UserService extends IService<User> {}
```

```java
import ....
@Service
// 自定义Service类继承ServiceImp类 实现自定义接口;
public class UserServiceImp extends ServiceImpl<UserMapper, User> implements UserService {

}
```

`Test`

```java
import ...
@SpringBootTest
class UserServiceImpTest {

    @Autowired
    private UserService userService;

    @Test
    void saveTest(){
        User user = new User();
        user.setUsername("Harry");
        user.setPassword("321");
        user.setPhone("18816243127");
        user.setBalance(50000);
        user.setInfo("{\"age\": 20, \"intro\": \"佛系青年\", \"gender\": \"male\"}");
        user.setCreateTime(LocalDateTime.now());
        user.setUpdateTime(LocalDateTime.now());
	   
        // save() 添加数据
        userService.save(user);
    }
}
```

`Lambda实现`

`UserController.java`

```java
import ....

@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserService userService;

    // 查询所有数据
    @GetMapping()
    public String  queryAll(){
        List<User> users = userService.list();
        users.forEach(user -> System.out.println(user.toString()));
        return "Ok";
    }

    // 通过id查询数据
    @GetMapping("/{id}")
    public String queryById(@PathVariable("id") Long id){
        User user = userService.getById(id);
        System.out.println(user.toString());
        return "Ok";
    }

    // 根据id删除数据  form格式
    @DeleteMapping()
    public String deleteById(Long id){
        userService.removeById(id);
        return "OK";
    }

    // 复杂查询数据  --> 通过 lambdaQuery() 类实现
    @PostMapping()
    public String complexQuery(String username,Integer status,Integer minBalance , Integer maxBalance){
        List<User> users =  userService.querySection(username,status,minBalance,maxBalance);
        users.forEach(user -> System.out.println(user.toString()));
        return "OK";
    }

    // 复杂修改数据 --> 通过 lambdaUpdate() 类实现
    @PutMapping()
    public String complexUpdate(Long id,Integer money){
        userService.complexUpdate(id,money);
        return "OK";
    }

}
```

`UserServiceImp.java`

```java
import ....

@Service
public class UserServiceImp extends ServiceImpl<UserMapper, User> implements UserService {

    @Override
    public List<User> querySection(String name, Integer status, Integer minBalance,Integer maxBalance) {
        // 通过lambdaQuery()查询User表中所有字段的信息
        List<User> list = lambdaQuery()
                .like(name != null, User::getUsername, name)
                .eq(status != null, User::getStatus, status)
                .ge(minBalance != null, User::getBalance, minBalance)
                .le(maxBalance != null, User::getBalance, maxBalance)
                .list();

        return list;
    }

    @Override
    public void complexUpdate(Long id, Integer money) {
        // 1.判断当前用户是否存在
        User user = getById(id);
        if(user == null || user.getStatus() == 2){
            throw new RuntimeException("当前用户出现异常");
        }
        // 2.判断当前用户余额
        if (user.getBalance() <= 0){
            throw new RuntimeException("当前用户余额已经为空");
        }

        // 3.扣除完用户的余额为0,将status进行修改
        int balance = user.getBalance() - money;

        lambdaUpdate()
                // SET
                .set(User::getBalance,balance)
                .set(balance <= 0 ,User::getStatus,2)
                // WHERE
                .eq(User::getId,id)
                // execute
                .update();
    }
}
```

# Extend

## Code Generation

简介 : 通过Mybatis插件,连接数据库后可自动生成 实体类 Controller Service Mapper;

`Test Connection`

![image-20231107095715714](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231107095715714.png) 

`Create`

![image-20231107095914064](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231107095914064.png) 

## Db Static Tool

简介 :  Db静态工具,具有的方法和IService接口的方法一致,为了解决业务层循环调用的问题(UserService中同时需要使用 UserMapper 和 AddressMapper等);

![image-20231107100619756](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231107100619756.png) 

`UserController`

```java
import ...

@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {

    @Autowired
    private UserService userService;

    // 通过id查询用户信息 和 地址信息
    @GetMapping("/{id}")
    public String queryById(@PathVariable("id") Long id){
        UserVO userVO = userService.queryUserAndAddress(id);
        System.out.println(userVO.toString());
        return "OK";
    }
}
```

`UserService`

```java
import ....

public interface UserService extends IService<User> {

    UserVO queryUserAndAddress(Long id);
}
```

`UserServiceImp`

```java
import ...

@Service
public class UserServiceImp extends ServiceImpl<UserMapper, User> implements UserService {

    // 查询用户的同时查询用户对应的所有地址  --> 通过 静态工具类 实现
    @Override
    public UserVO queryUserAndAddress(Long id) {
        // 1.查询当前用户的 id
        User user = getById(id);
        // 2.判断当前用户是否存在
        if (user == null  || user.getStatus() == 2){
            throw new RuntimeException("用户出现异常");
        }
        // 3.根据用户id 查询出 对应的地址

        // 通过 静态工具类 进行 Db进行查询
        List<Address> addressLists = Db.lambdaQuery(Address.class)
                .eq(Address::getUserId, id).list();
        // 4.封装UserVO
        UserVO userVO = new UserVO();
        BeanUtils.copyProperties(user,userVO);
        userVO.setAddressList(addressLists);

        return userVO;

    }
}
```

## Logic Delete

简介 : MybatisPlus提供了逻辑删除的功能,`无需改变方法的调用的方式`,通过配置文件自动修改CEUD的语句;

`application.yaml`

```yaml
# mybatisplus 状态的修改
mybatis-plus:
    global-config:
      db-config:
        logic-delete-field: delete # 全局逻辑删除的实体字段名称
```

`AddressController.java`

```java
import ....
@RestController
@RequestMapping("/address")
public class AddressController {

    @Autowired
    IAddressService addressService;

    // 根据id逻辑地址信息  逻辑删除的sql语句为: UPDATE address SET deleted = 1 WHERE id = 1 AND deleted = 0
    @DeleteMapping
    public String delete(Long id){
        addressService.removeById(id);
        return "OK";
    }
}
```

## Enum Processor

简介 : MybatisPlus中的枚举处理器,自动处理实体类中定义的枚举属性,使其和数据库中的记录进行匹配;

![image-20231107153538374](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231107153538374.png) 

`配置枚举处理器`

```yaml
mybatis-plus:
  configuration:
    # 配置统一的枚举处理器,实现类型转换
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
```

`Enum --> UserStatus`

```java
import ...
// 定义枚举类 --> 规定用户存在的状态
public enum UserStatus {
    NORMAL(1,"正常"),
    FROZEN(2,"冻结"),
    ;
    // 给枚举中的与数据库对应 value值添加@EnumValue注解
    @EnumValue
    private final  int value;
    @JsonValue
    private  final String desc;

    UserStatus(int value,String desc){
        this.value = value;
        this.desc = desc;
    }
}
```

`Entity --> User`

```java
import ...

@Data
public class User {
    .... other attribute
    // 状态定义为 枚举类型 （1正常 2冻结）
    private UserStatus status;
}
```

## Json Processor

简介 : 数据库中存在Json格式的数据,一般对应的实体类中的属性为 String 类型;可通过MybatisPlus的Json处理器将其处理为对应的类型;

`Json --> UserInfo`

```java
import ..
@Data
// json 格式 数据对应的类
public class UserInfo {
    private  Integer age;
    private  String info;
    private  String gender;
}
```

`Entity --> User`

```java
import ...

@Data
// 规定实体类对应的表, 开启 autoResultMap 可进行 Json 处理
@TableName(value = "user", autoResultMap = true)
public class User {
	... other attribute
    // 选定处理Json格式的处理器
    @TableField(typeHandler = JacksonTypeHandler.class)
    private UserInfo info;

}
```

# Plugin

## Page

简介 : 通过注册MybatisPlus的核心插件,实现分页功能;

`Config --> MybatisPlusConfig`

```java
import ..

@Configuration
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        // 1.初始化核心插件
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        // 2.添加分页插件
        PaginationInnerInterceptor pageInterceptor = new PaginationInnerInterceptor(DbType.MYSQL);
        // 2.1设置分页上限
        pageInterceptor.setMaxLimit(1000L);
        // 3.添加上分页插件
        mybatisPlusInterceptor.addInnerInterceptor(pageInterceptor);
        return  mybatisPlusInterceptor;
    }
}
```

`Test --> UserTest`

```java
import ..

@SpringBootTest
class UserServiceImpTest {

    @Autowired
    private UserService userService;
    
    @Test
    void pageTest(){
        //1.查询
        int pageNo = 1, pageSize = 5;
        //1.1分页参数
        Page<User> page = Page.of(pageNo, pageSize);
        //1.2分页的排序参数
        OrderItem balance = new OrderItem("balance", false);
        page.addOrder(balance);
        //1.3分页查询
        Page<User> page1 = userService.page(page);
        // 2.获取查询的数据
        System.out.println("查询的页数" + page1.getPages());
        System.out.println("查询的条数" + page1.getTotal());
        // 分页数据
        List<User> records = page.getRecords();
        records.forEach(System.out::println);
    }
}
```

## Page Practice

简介 : 在后端实现分页操作的流程;

`Query --> PageQuery`

```java
import ..

@Data
public class PageQuery {
    private Integer pageNo;
    private Integer pageSize;
    private String sortBy;
    private Boolean isAsc;
	
    // 1.分页的条件  --> 可复用
    public <T>  Page<T> toMpPage(OrderItem ... orders){
        // 1.分页条件
        Page<T> p = Page.of(pageNo, pageSize);
        // 2.排序条件
        // 2.1.先看前端有没有传排序字段
        if (sortBy != null) {
            p.addOrder(new OrderItem(sortBy, isAsc));
            return p;
        }
        // 2.2.再看有没有手动指定排序字段
        if(orders != null){
            p.addOrder(orders);
        }
        return p;
    }
    // 一般调用 这个 方法 判断是否存在排序的
    public <T> Page<T> toMpPage(String defaultSortBy, boolean isAsc){
        return this.toMpPage(new OrderItem(defaultSortBy, isAsc));
    }

    public <T> Page<T> toMpPageDefaultSortByCreateTimeDesc() {
        return toMpPage("create_time", false);
    }

    public <T> Page<T> toMpPageDefaultSortByUpdateTimeDesc() {
        return toMpPage("update_time", false);
    }
}

```

`Query --> UserQuery` 

```java
import ..

@Data
@ApiModel(description = "用户查询条件实体")
public class UserQuery extends  PageQuery{
    @ApiModelProperty("用户名关键字")
    private String name;
    @ApiModelProperty("用户状态：1-正常，2-冻结")
    private Integer status;
    @ApiModelProperty("余额最小值")
    private Integer minBalance;
    @ApiModelProperty("余额最大值")
    private Integer maxBalance;
}
```

`VO --> UserVO`

```java
import ..
    
@Data
@ApiModel(description = "用户VO实体")
public class UserVO {

    @ApiModelProperty("用户id")
    private Long id;

    @ApiModelProperty("用户名")
    private String username;

    @ApiModelProperty("详细信息")
    private String info;

    @ApiModelProperty("使用状态（1正常 2冻结）")
    private Integer status;

    @ApiModelProperty("账户余额")
    private Integer balance;

    private List<Address> addressList;
}
```

`Controller --> UserController`

```java
import ..

@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {

    @Autowired
    private UserService userService;

    // 根据分页条件查询用户接口
    @GetMapping("/page")
    public PageDTO<UserVO> queryUesrsPage(UserQuery query){
        return userService.queryUsersPage(query);
    }

}
```

`Service --> UserServiceImp`

```java
import ...

@Service
public class UserServiceImp extends ServiceImpl<UserMapper, User> implements UserService {

    @Override
    public PageDTO<UserVO> queryUsersPage(UserQuery query) {
        // 1.创建分页条件
        // 1.1分页的页数 条数
        Page<User> page = Page.of(query.getPageNo(), query.getPageSize());
        // 1.2分页的排序规则
        page.addOrder(new OrderItem(query.getSortBy(),query.getIsAsc()));

        // 2.分页查询
        Page<User> page1 = lambdaQuery()
                .like(query.getName() != null, User::getUsername, query.getName())
                .eq(query.getStatus() != null, User::getStatus, query.getStatus())
                .page(page);
        // 3.封装成VO结果
        PageDTO<UserVO> userVOPageDTO = new PageDTO<>();
        // 3.1 总条数
        userVOPageDTO.setTotal(page1.getTotal());
        // 3.2 总页数
        userVOPageDTO.setPages(page1.getPages());
        // 3.3 当前页数信息
        List<User> records = page1.getRecords();

        if (CollectionUtil.isEmpty(records)){
            userVOPageDTO.setList(Collections.emptyList());
            return userVOPageDTO;
        }

        // 3.4将数据进行拷贝
        List<UserVO> userVOS = BeanUtil.copyToList(records, UserVO.class);
        userVOPageDTO.setList(userVOS);

        return userVOPageDTO;
    }
}
```

