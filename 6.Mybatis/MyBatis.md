# MyBatis

简介 : MyBatis是一款优秀的持久层(`负责将数据保存到数据库中`)框架,简化JDBC开发;

JavaEE三层框架: 表现层(页面展示) 业务层(逻辑处理) 持久层(数据存储);

*MyBatis快速入门 :*

`通过Maven配置Mysql驱动 导入MyBatis依赖 :`

```xml
  <dependencies>
   
    <!--导入Mysql驱动-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.46</version>
    </dependency>

    <!--导入MyBatis依赖-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.5</version>
    </dependency>

  </dependencies>
```

`配置mybatis-config.xml文件,连接数据库,加载SQL映射文件 :`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>

            <dataSource type="POOLED">
            <!--数据库连接信息-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/java_testdata"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>

        </environment>
    </environments>

    <mappers>
        <!--加载SQL的映射文件-->
        <mapper resource="UserMapper.xml"/>
    </mappers>

</configuration>
```

`配置映射文件UserMapper.xml执行SQL语句 :`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace: 名称空间-->
<mapper namespace="test">

<!--id为唯一标识  resultType为返回类型-->
    <select id="selectAll" resultType="com.banne.pojo.User">
        select * from user;
    </select>

</mapper>
```

`User.java`

```java
package com.banne.pojo;
public class User
{
    private int id;
    private String username;
    private String password;
    private String gender;
    private String addr;

    public User(int id, String username, String password, String gender, String addr) {
        this.id = id;
        this.username = username;
        this.password = password;
        this.gender = gender;
        this.addr = addr;
    }

    @Override
    public String toString() {...}
}
```

`MyBatisDemoTest.java`

```java
package com.banne;
import ....
    
// MyBatis快速入门测试代码
public class MyBatisDemoTest {
    public static void main(String[] args) throws IOException {
        // 1.获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 2.获取SqlSession对象,执行SQL语句
        SqlSession sqlSession = sqlSessionFactory.openSession();

        // 3.执行sql语句
        List<User> userList = sqlSession.selectList("test.selectAll");
        for (User user : userList) {
            System.out.println(user);
        }

        // 4.释放资源
        sqlSession.close();
    }
}
```

# Mapper

简介 : Mapper代理的方式,简化SQL语句的执行过程;

1. 定义与SQL映射文件相同的Mapper接口,将Mapper接口和SQL映射文件放在同一目录下
2. 设置SQL映射文件的namespace为Mapper接口的全限定名
3. 在Mapper接口中定义方法,方法名称为SQL映射文件中sql语句的Id名,并保存参数类型和返回值一致

`配置文件mybatis-config.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
....
<configuration>
    <environments default="development">....</environments>

    <mappers>
        <!--普通加载SQL的映射文件-->
        <!--<mapper resource="com/banne/mapper/UserMapper.xml"/>-->

        <!--Mapper代理方式加载SQL的映射文件-->
        <package name="com.banne.mapper"/>
    </mappers>

</configuration>
```

`映射文件UserMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
...
<!-- namespace: 名称空间要和Mapper接口名称一致 -->
<mapper namespace="com.banne.mapper.UserMapper">

<!--id为唯一标识  resultType为返回类型-->
    <select id="selectAll" resultType="com.banne.pojo.User">
        select * from user;
    </select>

</mapper>
```

`Mapper接口UserMapper.interface`

```java
public interface UserMapper {
    // 方法名 与 UserMapper.xml 文件中id名相同
    List<User> selectAll();
}
```

`MyBatisDemoTest2.java`

```java
package com.banne;
import...

// MyBatis中的Mapper代理开发
public class MyBatisDemoTest2 {
    public static void main(String[] args) throws IOException {
        // 1.获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 2.获取SqlSession对象,执行SQL语句
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3.Mapper代理执行SQL语句
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> userList = userMapper.selectAll();
        for (User user : userList){
            System.out.println(user);
        }

        // 4.释放资源
        sqlSession.close();
    }
}
```

# MyBatis Configuration File

`配置标签顺序图 :`

![image-20230719160514046](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20230719160514046.png) 

`mybatis-config.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
...
<configuration>
    <!--typeAliases 类型别名-->
    <typeAliases>
        <package name = "com.banne.pojo"/>
    </typeAliases>
   
    <!--environments环境配置,default 可以进行更改-->
    <environments default="development">
        
        <environment id="development">
            <transactionManager type="JDBC"/>
        <!--dataSource数据源-->
            <dataSource type="POOLED">
            <!--数据库连接信息-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/java_testdata"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>

        </environment>
        
    </environments>
    <!--mappers映射器-->
    <mappers>
        <package name="com.banne.mapper"/>
    </mappers>

</configuration>
```

# Select Data

`BrandMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
    namespace: 名称空间要和Mapper接口名称一致
-->
<mapper namespace="com.banne.mapper.BrandMapper">

    <!--解决无法自动封装数据(列名 != 属性名),通过resultMap标签解决-->
    <resultMap id="brandResultMap" type="com.banne.pojo.Brand">
        <!--id :映射唯一标识 主键
               column: 表的列名
               property : 实体类的属性名
            result : 映射一般字段
        -->
        <id column="id" property="id"/>

        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>

    </resultMap>

    <!-- Select All Data  id为唯一标识   resultType为返回类型-->
    <select id="selectAll" resultType="com.banne.pojo.Brand">
        select * from brand;
    </select>

<!-- =======================================================  -->

    <!-- Select Details Data id为唯一标识 parameterType为参数类型 resultType为返回类型-->
    <select id="selectById" parameterType="int" resultType="com.banne.pojo.Brand">
        <!--参数占位符 :
                1. #{}: 会将其替换成为?,为了防止SQL注入
                2. ${}: 拼写SQL,存在SQL注入问题
            参数类型 : parameterType 一般可以省略;
            特殊转义字符处理 :
                1. 转义字符: &it;
                2. CDATA区: <![CDATA[ ../ ]]>
        -->
        select * from brand where id <![CDATA[ < ]]> #{id};
    </select>

<!-- ======================================================= -->
    <!--Static Select By Condition -->
    <select id="selectByCondition" resultType="com.banne.pojo.Brand">
        select * from brand where
        company_name like #{companyName} and
        brand_name like #{brandName};
    </select>
    
<!-- ======================================================= -->
    <!--Dynamic Select By Condition -->
    <select id="selectByCondition" resultType="com.banne.pojo.Brand">
    <!--动态多条件查询 :
             if: 条件判断 test: 逻辑表达式
             出现and问题: 1. 恒等式解决 2.<where>标签解决-->
        select * from brand
        <where>
            <if test="brandName != null and brandName != ''"> and brand_name like #{brandName};</if>
        </where>
        
    </select>
    
<!-- =======================================================  -->
    <!--Dynamic Select By Single Condition-->
    <select id="selectBySingleCondition" resultType="com.banne.pojo.Brand">
    <!--动态单条件选择查询 : choose标签 when标签-->
        select * from brand
        <where>
            <choose> <!--类似于 switch-->
                <when test="staus != null and status != ''">
                     status = #{status}
                </when>

                <when test="brandName != null and brandName != ''">  <!--类似于 case-->
                     brand_name like #{brandName}
                </when>

                <when test="companyName != null and companyName != ''">
                     company_name  like #{companyName}
                </when>
            </choose>
        </where>

    </select>
</mapper>
```

`BrandMapper.interface`

```java
package com.banne.mapper;

import ...

public interface BrandMapper {
    // 查看所有数据
    public List<Brand> selectAll();

    // 查看详情: 根据id
    public Brand selectById(int id);

    // 静态条件查询: 根据Condition
    /*
    * 1.散装参数: 存在多个参数,需要使用@Param("")注解
    * 2.对象参数 3.map集合*/
    public List<Brand> selectByCondition(@Param("companyName") String company,@Param("brandName") String brand);

    public List<Brand> selectByCondition(Brand brand);

    public List<Brand> selectByCondition(Map map);
    
    // 动态单条件选择查询
    public Brand selectBySingleCondition(Map map);
}

```

`MybatisBrandTest.java`

```java
package com.banne.test;
import .....

public class MybatisBrandTest {
   public static void main(String[] args) throws IOException {
   }

   // 获取数据库表中所有数据
   public void testSelectAll() throws IOException {
      // 1.获取SqlSessionFactory对象
      String resource = "mybatis-config.xml";
      InputStream inputStream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

      // 2.获取SqlSession对象,执行SQL语句
      SqlSession sqlSession = sqlSessionFactory.openSession();

      //3.Mapper代理执行SQL语句
      BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
      List<Brand> brandList = brandMapper.selectAll();
      for (Brand brand : brandList){
         System.out.println(brand);
      }

      // 4.释放资源
      sqlSession.close();
   }

   // 根据id获取数据库表中,一行详细信息
   public void testSelectById() throws IOException {
      // 1.获取SqlSessionFactory对象
      String resource = "mybatis-config.xml";
      InputStream inputStream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

      // 2.获取SqlSession对象,执行SQL语句
      SqlSession sqlSession = sqlSessionFactory.openSession();

      //3.Mapper代理执行SQL语句
      BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
      // 查询id为2的一行数据
      Brand detailBrand = brandMapper.selectById(2);

      System.out.println(detailBrand);

      // 4.释放资源
      sqlSession.close();
   }

   // 根据多条件静态/动态查询数据表中信息
   public void testSelectByCondition() throws IOException {
      // 接受参数
      String companyBrand = "Oppo";
      String brandName = "oppo";

      // 处理参数
      companyBrand = "%"+companyBrand+"%";
      brandName = "%" + brandName + "%";

      // 封装Map对象
      Map map = new HashMap();
      map.put("brandName",brandName);
      map.put("companyBrand",companyBrand);

      // 1.获取SqlSessionFactory对象
      String resource = "mybatis-config.xml";
      InputStream inputStream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

      // 2.获取SqlSession对象,执行SQL语句
      SqlSession sqlSession = sqlSessionFactory.openSession();

      //3.Mapper代理执行SQL语句
      BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
      List<Brand> brandList = brandMapper.selectByCondition(map);
      for (Brand brands : brandList){
         System.out.println(brands);
      }

      // 4.释放资源
      sqlSession.close();
   }
     
    // 根据单条件动态查询数据表中信息
    public void testSelectBySingleCondition() throws IOException {
        // 获取参数
        String status = "1";
        String brandName = "vivo";
        String companyName = "Vivo";

        brandName = "%"+ brandName +"%";
        companyName = "%"+companyName+"%";

        Map map = new HashMap();
        //map.put("status",status);
        map.put("brandName",brandName);
        map.put("companyName",companyName);
        System.out.println(map);

        // 1.获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 2.获取SqlSession对象,执行SQL语句
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3.Mapper代理执行SQL语句
        BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
        Brand brand = brandMapper.selectBySingleCondition(map);
        System.out.println(brand);

        // 4.释放资源
        sqlSession.close();
    } 
}
```

# Add Data

`BrandMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
    namespace: 名称空间要和Mapper接口名称一致
-->
<mapper namespace="com.banne.mapper.BrandMapper">
    
        <!--Add Data 添加数据 设置属性 useGeneratedKeys,keyProperty 返回添加数据的主键给Brand对象-->
        <insert id="addData" useGeneratedKeys="true" keyProperty="id">
            insert into brand (brand_name,company_name,ordered,description,status) values
            (#{brandName},#{companyName},#{ordered},#{description},#{status});
        </insert>

</mapper>
```

`BrandMapper.interface`

```java
package com.banne.mapper;

import ...;

public interface BrandMapper {
    // Add Data , Return bool
    public boolean  addData(Brand brand);
}
```

`MybatisBrandTest.java`

```java
package com.banne.test;

import ....

public class MybatisBrandTest {
   public static void main(String[] args) throws IOException {
      MybatisBrandTest mybatisBrandTest = new MybatisBrandTest();
      mybatisBrandTest.testAddData();
   }

   public void testAddData() throws IOException {
      // 获取参数
      Brand brand = new Brand("iphone", "IPhone科技公司", "50", "IPhone change world", "1");

      // 1.获取SqlSessionFactory对象
      String resource = "mybatis-config.xml";
      InputStream inputStream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

      // 2.获取SqlSession对象,执行SQL语句 Mybatis自动关闭事务,设置为自动提交事务
      SqlSession sqlSession = sqlSessionFactory.openSession(true);

      //3.Mapper代理执行SQL语句
      BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
      boolean  bool = brandMapper.addData(brand);
      System.out.println(bool);
	 
      // 映射文件中,insert标签设置属性 useGeneratedKeys  keyProperty 返回添加数据的主键给当前对象
      System.out.println(brand.getId());
      // 4.释放资源
      sqlSession.close();
   }

}
```

# Update Data

`BrandMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
    namespace: 名称空间要和Mapper接口名称一致
-->
<mapper namespace="com.banne.mapper.BrandMapper">
        <!--Update All Data-->
        <update id="updateAllData">
            update brand set brand_name = #{brandName},company_name = #{companyName},ordered = #{ordered},
            description = #{description}, status = #{status} where id = #{id};
        </update>

        <!--Dynamic Update Data-->
        <update id="updateDynamicData">
            update brand
            <set>
                <if test="brandName != null and brandName != ''">brand_name = #{brandName}, </if>

                <if test="companyName != null and companyName != ''">company_name = #{companyName}, </if>

                <if test="ordered != null and ordered != ''">ordered = #{ordered} ,</if>

                <if test="description != null and description != ''">description = #{description}, </if>

                <if test="status != null and status != ''">status = #{status} </if>
            </set>
            where id = #{id};
        </update>
</mapper>
```

`BrandMapper.interface`

```java
package com.banne.mapper;

import ...
public interface BrandMapper {
    // Updata All Data
    public boolean updateAllData(Brand brand);

    // Dynamic Update Data
    public boolean updateDynamicData(Brand brand);
}
```

`MybatisBrandTest.java`

```java
package com.banne.test;

import  ....

public class MybatisBrandTest {
   public static void main(String[] args) throws IOException {
      MybatisBrandTest mybatisBrandTest = new MybatisBrandTest();
      mybatisBrandTest.testUpdateDynamicData();
   }

   public void testUpdateDynamicData() throws IOException {
      // 获取参数
      Brand brand = new Brand(null, "IPhone科技有限公司", "20", "IPhone change world", "1");
      brand.setId(7);
      System.out.println(brand);

      // 1.获取SqlSessionFactory对象
      String resource = "mybatis-config.xml";
      InputStream inputStream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

      // 2.获取SqlSession对象,执行SQL语句 Mybatis自动关闭事务,设置为自动提交事务
      SqlSession sqlSession = sqlSessionFactory.openSession(true);

      //3.Mapper代理执行SQL语句
      BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
      boolean bool = brandMapper.updateDynamicData(brand);
      System.out.println(bool);

      // 4.释放资源
      sqlSession.close();
   }

}
```

# Delete Data

`BrandMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
    namespace: 名称空间要和Mapper接口名称一致
-->
<mapper namespace="com.banne.mapper.BrandMapper">

        <!--Delete Much Data-->
        <delete id="deleteByIds">
            delete from brand where id in
            <!--通过 foreach 标签 来实现-->
            <!--Mybatis会将数组参数,封装成为一个Map集合, 默认 array = 数组; 可以在接口中使用@Param注解改变map集合默认的key名称-->
            <foreach collection="ids" item="id" separator="," open="(" close=")">
                    #{id}
            </foreach>
        </delete>

</mapper>
```

`BrandMapper.interface`

```java
package com.banne.mapper;

import ....

public interface BrandMapper {
    // Delete Much Data
    public boolean deleteByIds(@Param("ids") int[] arrs);
}
```

`MybatisBrandTest.java`

```java
package com.banne.test;

import ....

public class MybatisBrandTest {
   public static void main(String[] args) throws IOException {
      MybatisBrandTest mybatisBrandTest = new MybatisBrandTest();
      mybatisBrandTest.testDeleteByIds();
   }

   public void testDeleteByIds() throws IOException {
      // 获取参数
      int [] arrs = {1,2,9};

      // 1.获取SqlSessionFactory对象
      String resource = "mybatis-config.xml";
      InputStream inputStream = Resources.getResourceAsStream(resource);
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

      // 2.获取SqlSession对象,执行SQL语句 Mybatis自动关闭事务,设置为自动提交事务
      SqlSession sqlSession = sqlSessionFactory.openSession(true);

      //3.Mapper代理执行SQL语句
      BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);
      boolean bool = brandMapper.deleteByIds(arrs);
      System.out.println(bool);

      // 4.释放资源
      sqlSession.close();
   }
}
```

# Pass Parameters

简介 : MyBatis接口方法中可以传递各种各样的参数,MyBatis通过ParamNameResolver类来进行参数的封装;

*MyBatis 参数封装 :*

- 单个参数 

  - POJO类型 直接使用, 属性名 和 参数占位符名称一致
  - Map集合 直接使用, 属性名 和 参数占位符名称一致
  - Collection 封装为Map集合

  ```java
  map.put("arg0",Collection集合);
  map.put("collection",Collection集合);
  ```

  - List 封装为Map集合

  ```java
  map.put("arg0",list集合);
  map.put("collection",list集合);
  map.put("list",list集合);
  ```

  - Array 封装成Map集合

  ```java
  map.put("arg0",数组);
  map.put("array",数组);
  ```

  - 其他类型 : 直接使用

- 多个参数 : 封装为Map集合,可以使用 @Param注解,替换Map集合中默认的arg键名 

```java
	map.put("arg0",参数1);
	map.put("param1",参数1);

	map.put("arg1",参数2);
	map.put("param2",参数2);
```

# Annotation SQL

简介 : 注解开发会被配置文件开发更加方便,一般推荐简单功能;

*常用注解 :*

- 查询: @Select
- 添加: @Insert
- 修改: @Update
- 删除: @Delete

`BrandMapper.interface`

```java
package com.banne.mapper;

import ....

public interface BrandMapper {
    // Annotation Address SQL 无需映射配置文件
    @Select("select * from brand where id= #{id}")
    public Brand selectBynewId(int i);	
}
```

