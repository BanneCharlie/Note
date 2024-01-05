# Info

简介 : Elasticsearch是开源的`分布式搜索和分析引擎`,支持多种数据类型;采用RESTful API接口通过HTTP协议进行通信,使用特定的查询语言(DSL)支持复杂的查询;

Elasticsearch底层基于`Lucene实现`;

`ELK技术栈 :` Elasticsearch为核心,负责存储 搜索 分析数据

![image-20231223185029627](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231223185029627.png) 

`ElasticSearch的部署(单实例)`

```shell
# 创建网关,ES需搭配Kibana使用 在同一个网络下
docker network create elastic-network

docker run -d \
	--name elasticsearch \
    -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
    -e "discovery.type=single-node" \
    -v es-data:/usr/share/elasticsearch/data \
    -v es-plugins:/usr/share/elasticsearch/plugins \
    --privileged \
    --network elastic-network \
    -p 9200:9200 \
    -p 9300:9300 \
elasticsearch:7.12.1
```

![image-20231224162048399](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231224162048399.png) 

`Kibana的部署`

```shell
docker run -d \
--name kibana \
# 通过容器名访问elasticsearch,必须同一个网络下
-e ELASTICSEARCH_HOSTS=http://elasticsearch:9200 \
--network=elastic-network \
-p 5601:5601  \
kibana:7.12.1

# 查看启动日志
docker logs -f kibana 
```

![image-20231224163026590](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231224163026590.png) 

![image-20231224163152028](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231224163152028.png) 

`Elasticsearch配置IK分词器插件`

方法1 :

```shell
# 进入容器内部
docker exec -it elasticsearch /bin/bash

# 在线下载并安装
./bin/elasticsearch-plugin  install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.1/elasticsearch-analysis-ik-7.12.1.zip

#退出
exit
#重启容器
docker restart elasticsearch
```

方法2 :

```shell
# 查看elasticsearch挂载的数据卷
docker volume inspect es-plugins

[
    {
        "CreatedAt": "2023-12-23T20:04:33+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/es-plugins/_data",
        "Name": "es-plugins",
        "Options": null,
        "Scope": "local"
    }
]

# 进入插件的挂载位置,导入准备好的IK分词器文件
cd /var/lib/docker/volumes/es-plugins/_data

# 重启elasticsearch容器
docker restart elasticsearch
```

 ![image-20231224164254590](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231224164254590.png)

*IK分词器两种模式：*

* `ik_smart`：最少切分

* `ik_max_word`：最细切分

![image-20231224164826808](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231224164826808.png)  

# Auto Complete

简介 : 自动补全机制,当用户输入字母时会提示完整的词条,根据拼音字符来推断词条,需要使用到拼音的分词功能;

`Elasticsearch配置PY分词器插件`

```shell
# 查看elasticsearch挂载的数据卷
docker volume inspect es-plugins

[
    {
        "CreatedAt": "2023-12-23T20:04:33+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/es-plugins/_data",
        "Name": "es-plugins",
        "Options": null,
        "Scope": "local"
    }
]

# 进入插件的挂载位置,导入准备好的IK分词器文件
cd /var/lib/docker/volumes/es-plugins/_data

# 重启elasticsearch容器
docker restart elasticsearch
```

![image-20231231144839143](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231231144839143.png) 

*PY分词器模式 :*

![image-20231231145125164](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231231145125164.png)

## Custom tokenizer

简介 : 默认的拼音分词器会将每个汉字单独分为拼音,需要每个词条形成一组拼音,需要对拼音分词器做个性化定制;

*ElasticSearch中分词器的组成部分 ;*

- character filters: 在分词器之前对文本进行处理,例如: 删除字符 替换字符
- tokenizer: 将文本按照一定的规则切割成词条,例如: keyword不分词 ik_smart
- tokenizer filter: 将tokenizer输出的词条做进一步的处理; 例如: 大小写的转换 同义词的处理 拼音的处理

![image-20231231150017565](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231231150017565.png) 

```json
PUT /test
{
  "settings": {
    "analysis": {
      "analyzer": { // 自定义分词器 实现先分词  再根据分词添加拼音
        "my_analyzer": {  // 分词器名称
          "tokenizer": "ik_max_word", //分词
            
          "filter": "py" // 添加拼音
        }
      },
      "filter": { // 自定义tokenizer filter
        "py": { // 过滤器名称
          "type": "pinyin", // 过滤器类型，这里是pinyin
		"keep_full_pinyin": false,
          "keep_joined_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
    
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "my_analyzer",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

![image-20231231151348291](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231231151348291.png) 

# Inverted Index

简介 : 倒排索引用于`快速查找文档中包含特定词汇`的数据结构;

正向索引通过文档到词汇的映射即通过文档的标识找到对应的词汇;

倒排索引则是通过词汇到文档的映射即通过词汇找到包含该词汇的文档列表; 

*正向索引的优缺点 :*

- 直观可理解 支持顺序访问  更容易实现
- 检索速度慢(复杂的查询需要逐条遍历) 占用内存空间大(存储每个文档的全部数据) 不支持复杂的查询(布尔查询 短语句查询) 

*倒排索引的优缺点 :*

- 快速检索 高效存储(只存储词汇和文档的对应关系) 支持复杂查询 大规模数据 全文搜索
- 构建成本高(需要对文档进行分词) 不适合顺序访问 不适合实时更新

![image-20231226132532690](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226132532690.png)

![image-20231226132752016](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226132752016.png)

`Mysql与Elasticsearch的比较`

![image-20231226135141596](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226135141596.png) 

`Myqsl和Elasticsearch结合`

![image-20231226135105803](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226135105803.png) 

# Index Database

简介 : 索引库表类似于数据库表,Mapping映射类似于表的结构,向Elasticsearch中存储数据,必须创建库和表;

*Mapping映射属性 :*

- type：字段数据类型，常见的简单类型有：
  - 字符串：text（可分词的文本）、keyword（精确值，例如：品牌、国家、ip地址）
  - 数值：long、integer、short、byte、double、float、
  - 布尔：boolean
  - 日期：date
  - 对象：object
- index：是否创建索引，默认为true
- analyzer：使用哪种分词器
- properties：该字段的子字段

*索引库的操作 :*

- 创建索引库：PUT /索引库名
- 查询索引库：GET /索引库名
- 删除索引库：DELETE /索引库名
- 添加字段：PUT /索引库名/_mapping

## Create

- 请求方式: PUT 
- 请求路径为 /"索引库名称",自定义 
- 请求参数: mapping映射

![image-20231226141331688](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226141331688.png) 

## Query

- 请求方式: GET
- 请求路径: /索引库名称
- 请求参数: 无

![image-20231226141545802](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226141545802.png)

## Alter

简介 : 倒排索引一但数据结构发生改变,需要重新创建倒排索引;因此索引库一但创建,无法修改Mapping映射属性;

但是可以添加新的字段在Mapping中;

- 请求方式: PUT
- 请求路径: /索引库名/_mapping
- 请求参数; mapping映射

![image-20231226142021555](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226142021555.png)

## Drop

- 请求方式：DELETE

- 请求路径：/索引库名

- 请求参数：无

![image-20231226142137563](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226142137563.png) 

# Document

*文档的操作 :*

- 创建文档：POST /{索引库名}/_doc/文档id   { json文档 }
- 查询文档：GET /{索引库名}/_doc/文档id
- 删除文档：DELETE /{索引库名}/_doc/文档id
- 修改文档：
  - 全量修改：PUT /{索引库名}/_doc/文档id { json文档 }
  - 增量修改：POST /{索引库名}/_update/文档id { "doc": {字段}}

## Insert

![image-20231226144158400](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226144158400.png) 

## Select

![image-20231226144335916](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226144335916.png) 

## Delete

![image-20231226144901988](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226144901988.png)

## Update

- 全量修改：PUT /{索引库名}/_doc/文档id { json文档 }

![image-20231226144644450](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226144644450.png) 

- 增量修改：POST /{索引库名}/_update/文档id { "doc": {字段}}

![image-20231226144759010](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231226144759010.png) 

# RestClient Operate

简介 : ES官方提供了不同语言的客户端,来操作ES;这些客户端的本质为组装DSL语句,发送http请求发送给ES;

Java Rest Client包括两种:

- Java Low Level Rest Client
- Java High Level Rest Client (常用)

`pom.xml` 导入ElasticSearch相关依赖

```xml
    	<!--设置elasticsearch版本号,与服务器版本号相同-->
 	    <properties>
        	<java.version>1.8</java.version>
        	<elasticsearch.version>7.12.1</elasticsearch.version>
    	</properties>

	   <!--导入Elasticsearch依赖-->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
        </dependency>
```

`ElasticSearchConfig.java` 向IOC中注入 RestHighLevelClient类

```java
import ..

@Configuration
public class ElasticSearchConfig {

    /**
     * 注入 RestHighLevelClient类
     * @return
     */
    @Bean
    public RestHighLevelClient restHighLevelClientFactory(){
        return new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://192.168.238.128:9200")
        ));
    }
}
```

## API Index database 

简介 : 通过RestHighlevelClient的相关API操作索引库,实现创建 更改 删除 查询等操作;

*索引库操作的基本步骤：*

- 初始化RestHighLevelClient
- 创建XxxIndexRequest。XXX是Create、Get、Delete
- 准备DSL（ Create时需要，其它是无参）
- 发送请求。调用RestHighLevelClient#indices().xxx()方法，xxx是create、exists、delete



`创建索引库 hotel`

![image-20231229162036675](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229162036675.png)

`删除索引库 hotel`

```java
@SpringBootTest
class HotelDemoApplicationTests {

    @Autowired
    private RestHighLevelClient restHighLevelClient;

    /**
     * 删除 indexDatabase --> hotel
     * @throws IOException
     */
    @Test
    public void deleteHotelIndex() throws IOException {
        // 1.创建DeleteIndexRequest对象,删除请求
        DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("hotel");
        // 2.发送请求,进行删除
        restHighLevelClient.indices().delete(deleteIndexRequest, RequestOptions.DEFAULT);
    }
}
```

`查看索引库是否存在`

```java
@SpringBootTest
class HotelDemoApplicationTests {

    @Autowired
    private RestHighLevelClient restHighLevelClient;

    /**
     * 判断 indexDatabase 是否存在
     * @throws IOException
     */
    @Test
    public void existHotelIndex() throws IOException {
        // 1.创建GetIndexRequest对象,查看请求
        GetIndexRequest getIndexRequest = new GetIndexRequest("hotel");
        // 2.发送请求查看,hotel索引库是否存在
        boolean exists = restHighLevelClient.indices().exists(getIndexRequest, RequestOptions.DEFAULT);

        System.out.println(exists ?"hotel索引库存在" : "不存在!");
    }
}
```

## API Document

简介 : 通过RestHighLevelClient的相关API操作文档,对文档进行添加 批量添加 删除 全量修改 局部修改 查询文档;

Elasticsearch中的数据来自于Mysql数据库;

*文档操作的基本步骤 :*

- 初始化RestHighLevelClient
- 创建XxxRequest。XXX是Index、Get、Update、Delete、Bulk
- 准备参数（Index、Update、Bulk时需要）
- 发送请求。调用RestHighLevelClient#.xxx()方法，xxx是index、get、update、delete、bulk
- 解析结果（Get时需要）



`Hotel.java` 对应数据库中表tb_hotel

```java
@Data
@TableName("tb_hotel")
public class Hotel {
    @TableId(type = IdType.INPUT)
    private Long id;
    private String name;
    private String address;
    private Integer price;
    private Integer score;
    private String brand;
    private String city;
    private String starName;
    private String business;
    private String longitude;
    private String latitude;
    private String pic;
}
```

`HotelDoc.java` 对应ES中索引库中的文档

```java
@Data
@NoArgsConstructor
public class HotelDoc {
    private Long id;
    private String name;
    private String address;
    private Integer price;
    private Integer score;
    private String brand;
    private String city;
    private String starName;
    private String business;
    private String location;
    private String pic;
    
    public HotelDoc(Hotel hotel) {
        this.id = hotel.getId();
        this.name = hotel.getName();
        this.address = hotel.getAddress();
        this.price = hotel.getPrice();
        this.score = hotel.getScore();
        this.brand = hotel.getBrand();
        this.city = hotel.getCity();
        this.starName = hotel.getStarName();
        this.business = hotel.getBusiness();
        // 将经纬度合成为一个属性,进行操作
        this.location = hotel.getLatitude() + ", " + hotel.getLongitude();
        this.pic = hotel.getPic();
    }
}
```

`创建文档 hotel/_doc/36934`

![image-20231229165042705](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229165042705.png)

`批量创建文档`

```java
@SpringBootTest
class HotelDemoApplicationTests {

    @Autowired
    private RestHighLevelClient restHighLevelClient;

    @Autowired
    private IHotelService hotelService;

    /**
     * 批量创建文档
     */
    @Test
    public void bulkCreateDocument() throws IOException {
        // 0.获取数据库中tb_user表中的所有信息
        List<Hotel> list = hotelService.list();

        // 1.创建BulkRequest对象,根据Id批量插入文档中
        BulkRequest bulkRequest = new BulkRequest();

        // 2.遍历list集合将 Hotel对象转化为插入ES中的HotelDoc对象,放入参数体中;
        for(Hotel hotel : list){
            HotelDoc hotelDoc = new HotelDoc(hotel);

            bulkRequest.add(new IndexRequest("hotel")
                    .id(hotelDoc.getId().toString())
                    .source(JSON.toJSONString(hotelDoc),XContentType.JSON)
            );
        }
        // 3.发送请求
        restHighLevelClient.bulk(bulkRequest,RequestOptions.DEFAULT);
    }
}
```

`根据Id删除文档`

```java
@SpringBootTest
class HotelDemoApplicationTests {

    @Autowired
    private RestHighLevelClient restHighLevelClient;

    @Test
    public void deleteDocument() throws IOException {
        // 1.根据索引库名 id,删除指定的文档
        DeleteRequest deleteRequest = new DeleteRequest("hotel","36934");
        // 2.发送请求
        restHighLevelClient.delete(deleteRequest,RequestOptions.DEFAULT);
    }
}
```

`根据Id修改文档(增量修改)`

```java
@SpringBootTest
class HotelDemoApplicationTests {

    @Autowired
    private RestHighLevelClient restHighLevelClient;
    
    /**
     * 根据Id增量修改文档,全量修改文档基本上和添加文档一样不常用
      * @throws IOException
     */ 
   @Test
   public void updateDocument() throws IOException {
       // 1.根据索引库名 id,修改指定的文档
       UpdateRequest updateRequest = new UpdateRequest("hotel","36934");
       // 2.增量修改,准备修改的参数
       updateRequest.doc(
               "price","988"
       );
       // 3.发送请求进行修改
       restHighLevelClient.update(updateRequest,RequestOptions.DEFAULT);
   }
}
```

`根据Id查询文档`

```java
@SpringBootTest
class HotelDemoApplicationTests {

    @Autowired
    private RestHighLevelClient restHighLevelClient;

    /**
     * 根据Id查询文档,返回查询数据
     * @throws IOException
     */
    @Test
    public void getDocument() throws IOException {
        // 1.根据索引库名 id,查询指定的文档
        GetRequest getRequest = new GetRequest("hotel", "36934");
        // 2.发送请求,返回查询的数据
        GetResponse getResponse = restHighLevelClient.get(getRequest, RequestOptions.DEFAULT);
        // 3.解析响应的结果
        String result = getResponse.getSourceAsString();
        // 4.将Json格式字符串,转换为HotelDoc对象
        HotelDoc hotelDoc = JSON.parseObject(result, HotelDoc.class);
        System.out.println(hotelDoc.toString());
    }
}
```

#  Query

简介 : ES的查询仍然基于JSON风格的DSL来实现;

```json
// 查询的语法基本一致
GET /index database name/_search
{
    "query":{
        "查询类型" : {
            "查询条件" : "条件值"
        }
    }
}
```

*DSL常见的查询类型 :*

- 查询所有: 查询所有数据,一般测试使用;  match_all

  ```json
  // 查询所有,没有查询条件
  GET /hotel/_search
  {
      "query":{
          "match_all":{
              
          }
      }
  }
  ```

- 全文检索查询: 利用分词器对用户输入的内容进行分词,去倒排索引库中匹配,得到文档id 根据文档id找到文档,返回结果;

  - match查询: 单字段查询  将常用的字段copy_to复制到 all字段中,也可以实现多字段查询效果,还提升性能;

  ```json
  // 单字段查询
  GET /hotel/_search
  {
      "query":{
          "match":{
              "name":"如家"
          }
      }
  }
  ```

  ![image-20231229191053222](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229191053222.png) 

  - multi_match查询: 多字段查询,任意一个字段符合条件就算符合查询条件

  ```json
  // 查询多个字段
  GET /hotel/_search
  {
      "query":{
       	"multi_match":{
  		   "query" : "上海如家",
            	"fields":["city","name"]
          }
      }
  }
  ```

  ![image-20231229191510999](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229191510999.png)

  ​	 

- 精确查询: 根据精确词条值查询数据,一般查找keyword 数值 日期 boolean等类型字段
  - range: 根据值的范围查询

  ```json
  // range 根据范围值查询
  GET /hotel/_search
  {
     "query":{
         "range":{
             "price":{
                 "gte":500, // gt代表大于 gte代表大于等于
                 "lte":1500 // lt代表小于 lte代表小于等于
             }
         }
     }   
  }
  ```

  ![image-20231229192619148](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229192619148.png) 

  - term: 根据词条精准确定值查询

  ```json
  // term 精准确定值查询
  GET /hotel/_search
  {
      "query":{
          "term":{
              "city":{
                  "value":"北京"
              }
          }
      }
  }
  ```

  ![image-20231229192057965](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229192057965.png)

  `内容为词条无法搜索到一一对应的结果`

  ![image-20231229192254065](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229192254065.png) 
- 地理查询: 根据经纬度查询
  - geo_distance

  `圆形范围`

  ![image-20231229192730529](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229192730529.png) 

  ```json
  GET /hotel/_search
  {
      "query":{
          "geo_distance":{
              "distance":"15km", // 半径
              "location": "31.21,121.5" // 圆心
          }
      }
  }
  ```

  ![image-20231229193113091](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229193113091.png)

  - geo_bounding_box

  `矩形范围`

  ![image-20231229192708102](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229192708102.png) 

  ```json
  GET /hotel/_search
  {
      "query":{
          "geo_bounding_box":{
              "location":{
                       "top_left": { // 左上点
            			"lat": 31.1,
            			"lon": 121.5
          	},
          		"bottom_right": { // 右下点
            			"lat": 30.9,
            			"lon": 121.7
        	    }
           }
      }
  }
  ```

  ![image-20231229193546381](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229193546381.png) 
- 复合查询: 复合查询可将上述各种条件查询组合起来,合并为查询条件;
  - bool: 布尔查询,利用逻辑关系组合多个其它的查询,实现复杂查询

  *布尔查询的组合方式 :*

  1. must：必须匹配每个子查询，类似“与”
  2. should：选择性匹配子查询，类似“或”
  3. must_not：必须不匹配，**不参与算分**，类似“非”
  4. filter：必须匹配，**不参与算分**

  ```json
  GET /hotel/_search
  {
    "query": {
      "bool": {
        // 必须匹配的每个子查询  
        "must": [
          {"term": {"city": "上海" }}
        ],
        // 选择性匹配子查询  
        "should": [
          {"term": {"brand": "皇冠假日" }},
          {"term": {"brand": "华美达" }}
        ],
        // 必须不匹配每个子查询,不计分  
        "must_not": [
          { "range": { "price": { "lte": 500 } }}
        ],
        // 必须匹配每个子查询,不计分  
        "filter": [
          { "range": {"score": { "gte": 45 } }}
        ]
      }
    }
  }
  ```

  - function_score: 算分函数查询,可以控制文档相关性算分,控制文档排名;

  *算分函数查询的关键部分 :*

  1. 过滤条件：决定哪些文档的算分被修改
  2. 算分函数：决定函数算分的算法
  3. 运算模式：决定最终算分结果

  ```json
  // 通过算分函数查询,控制文档相关性算分
  GET /hotel/_search
  {
    "query":{
      "function_score": {
        // 原始查询条件
        "query": {"match": {"name": "上海"}},
        
        "functions": [
          {
            // 过滤条件,符合条件的文档会重新算分
            "filter":{"term": {"brand": "如家"}},
            // 算分函数  
            "weight":10
          }
        ],
        // 算分函数 function_score 与 query分值的运算方式,默认为乘法
        "boost_mode": "sum"
      }
    }
  }
  ```

![image-20231229204825655](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229204825655.png) 

`相关性算分`

![image-20231229194215596](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229194215596.png) 

*相关度打分的算法 :*

- TF-IDF算法

![image-20231229194724002](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229194724002.png) 

- BM25算法,ES5.1版本之后采用

![image-20231229194802498](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229194802498.png) 

`TF-IDF算法与BM25算法的曲线图`

![image-20231229194913905](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231229194913905.png) 

#  Search Result

简介 : 搜索的结果可以按照用户指定的方式去处理或展示;

*DSL常见的处理 :*

- 排序: ES默认的搜索方式根据相关度算分,可自定义方式对搜索结果排序,例如: keyword 数值 地理坐标 日期类型等

  ```json
  // 普通字段查询
  GET /hotel/_search
  {
      "query" : {
          "match_all":{}
      },
      "sort" :{
          "score":"desc",
          "price":"asc"  //排序方式 desc降序 asc升序
      }
  }
  ```

  ![image-20231230111749274](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230111749274.png) 

  ```json
  // 地理坐标字段查询
  GET /hotel/_search
  {
      "query":{
          "match_all":{}
      },
      "sort":[
          {
              "_geo_distance":{
                  "location":{   //文档中geo_point类型的字段名 目标坐标点
                      "lat": 31.210759,
                      "lon": 121.275392
                  }, 
                  "order" : "asc", //排序方式
                  "unit" : "km" //排序的距离单位
              }
          }
      ]
  }
  ```

  ![image-20231230112805372](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230112805372.png) 

- 分页: ES默认情况下只返回top10的数据,通过修改ES中的from size参数来控制返回的分页结果;

  - from: 偏移量
  - size: 分页数目

  ```json
  GET /hotel/_search
  {
      "query":{
          "match_all":{}
      },
      "sort":[
          {"price":"asc"}
      ],
      "from": 990, //分页开始的位置 获取的数据为 990 - 1000条文档
      "size": 10 // 分页的数目
  }
  ```

  `单点模式,没有太大影响`

  ![image-20231230113340122](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230113340122.png)

  `集群模式,性能下降巨大`

  ![image-20231230113601730](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230113601730.png)

  *分页查询的常见实现方案以及优缺点：*

  - from + size：
    - 优点：支持随机翻页
    - 缺点：深度分页问题，默认查询上限（from + size）是10000
    - 场景：百度、京东、谷歌、淘宝这样的随机翻页搜索
  - after search：分页时进行排序,原理从上一次的排序值开始,查询下一页数据;
    - 优点：没有查询上限（单次查询的size不超过10000）
    - 缺点：只能向后逐页查询，不支持随机翻页
    - 场景：没有随机翻页需求的搜索，例如手机向下滚动翻页

  - scroll： 原理将排序后的文档id形成快照,保存在内存;
    - 优点：没有查询上限（单次查询的size不超过10000）
    - 缺点：会有额外内存消耗，并且搜索结果是非实时的
    - 场景：海量数据的获取和迁移。从ES7.1开始不推荐，建议用 after search方案。

- 高亮显示: 给文档中所有关键字添加一个标签<em>页面给<em>标签添加样式

  ```json
  GET /hotel/_search
  {
      "query":{
          "match":{
              "all":"如家" 
          }
      },
      "highlight": {
        "fields": {
          "name": {
            "require_field_match": "false"
          }
        }
      }
  }
  ```

  ![image-20231230115042518](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230115042518.png) 

`DSL查询属性`

![image-20231230115150420](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230115150420.png) 

# RestClient Query 

简介 : 通过RestHighLevelClient的相关API实现`全文检索查询 精确查询 地理查询 复合查询等查询操作和分页 排序 高亮操作`;

*文档查询的基本步骤 :*

- 初始化RestHighLevelClient对象

- 准备SearchRequest对象

- 为SearchRequest对象添加参数, QueryBuilders对象设置查询参数;

  ![image-20231230130406633](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230130406633.png) 

- 发送请求

- 解析响应结果

`快速入门 match_all查询`

![image-20231230125856511](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230125856511.png) 

`解析返回的结果`

![image-20231230130116088](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230130116088.png)

## Full Text query

![image-20231230131100060](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230131100060.png) 

```java
    /**
     * 全文检索查询 match, multi_match
     * @throws IOException
     */
    @Test
    public void queryMatch() throws IOException {
        // 1.创建请求
        SearchRequest searchRequest = new SearchRequest("hotel");
        // 2.设置查询参数
        searchRequest.source().query(QueryBuilders.matchQuery("name","如家"));
        // 3.发送请求
        SearchResponse searchResponse = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
        // 4.解析结果
        handle(searchResponse);
    }
```

## Precise query

![image-20231230131112689](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230131112689.png) 

```java
    /**
     * 精确查询 term,range
     * @throws IOException
     */
    @Test
    public void queryPrecise() throws IOException {
        // 1.创建请求
        SearchRequest searchRequest = new SearchRequest("hotel");
        // 2.设置查询参数
        searchRequest.source().query(QueryBuilders.termQuery("brand","汉庭"));
        // 3.发送请求
        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
        // 4.解析结果
        handle(search);
    }
```

## Bool query

![image-20231230131510482](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230131510482.png) 

```java
    /**
     * 布尔查询 条件为 must should filter must_not 
     * @throws IOException
     */
    @Test
    public void queryBool() throws IOException {
        // 1.创建请求
        SearchRequest searchRequest = new SearchRequest("hotel");
        // 2.设置查询参数
        // 2.1创建布尔查询
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        // 2.2设置布尔查询条件
        boolQuery.must(QueryBuilders.termQuery("brand","万豪"));
        boolQuery.filter(QueryBuilders.rangeQuery("price").lte(500));
        searchRequest.source().query(boolQuery);
        // 3.发送请求
        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
        // 4.解析结果
        handle(search);
    }
```

## Function Score query

![image-20231230153243915](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230153243915.png) 

`复合函数的全面使用`

```java
private void buildBasicQuery(RequestParam params, SearchRequest searchRequest) {
        // 1.构建BooleanQuery
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        // 2.关键字搜索
        String key = params.getKey();
        if (key == null || key.equals("")) {
            boolQuery.must(QueryBuilders.matchAllQuery());
        } else {
            boolQuery.must(QueryBuilders.matchQuery("all", key));
        }
        // 3.城市条件
        if (params.getCity() != null && !params.getCity().equals("")) {
            boolQuery.filter(QueryBuilders.termQuery("city", params.getCity()));
        }
        // 4.品牌条件
        if (params.getBrand() != null && !params.getBrand().equals("")) {
            boolQuery.filter(QueryBuilders.termQuery("brand", params.getBrand()));
        }
        // 5.星级条件
        if (params.getStarName() != null && !params.getStarName().equals("")) {
            boolQuery.filter(QueryBuilders.termQuery("starName", params.getStarName()));
        }
        // 6.价格
        if (params.getMinPrice() != null && params.getMaxPrice() != null) {
            boolQuery.filter(QueryBuilders
                    .rangeQuery("price")
                    .gte(params.getMinPrice())
                    .lte(params.getMaxPrice())
            );
        }

        // 2.算分控制
        FunctionScoreQueryBuilder functionScoreQuery =
                QueryBuilders.functionScoreQuery(
                        // 原始查询，相关性算分的查询
                        boolQuery,
                        // function score的数组
                        new FunctionScoreQueryBuilder.FilterFunctionBuilder[]{
                                // 其中的一个function score 元素
                                new FunctionScoreQueryBuilder.FilterFunctionBuilder(
                                        // 过滤条件
                                        QueryBuilders.termQuery("isAD", true),
                                        // 算分函数
                                        ScoreFunctionBuilders.weightFactorFunction(10)
                                )
                        }).scoreMode(FunctionScoreQuery.ScoreMode.SUM); // 设置评分方式

        // 7.放入source
        searchRequest.source().query(functionScoreQuery);
    }
```

## Auto complete query

简介 : ElasticSearch提供了Completion Suggester查询来实现自动补全功能,此查询会匹配用户输入内容开头的词条并返回;

*自动补全约束 :*

- 参与补全查询的字段类型必须为completion类型
- 字段的内容一般是用来补全的多个词条形成的数组

```json
// 创建索引库
PUT test
{
  "mappings": {
    "properties": {
      "title":{
        "type": "completion"
      }
    }
  }
}

// completion类型添加数据
POST test/_doc
{
  "title": ["Sony", "WH-1000XM3"]
}
POST test/_doc
{
  "title": ["SK-II", "PITERA"]
}
POST test/_doc
{
  "title": ["Nintendo", "switch"]
}

// 自动补全查询
GET /test/_search
{
  "suggest": {
    "title_suggest": {
      "text": "s", // 关键字
      "completion": {
        "field": "title", // 补全查询的字段
        "skip_duplicates": true, // 跳过重复的
        "size": 10 // 获取前10条结果
      }
    }
  }
}
```

`RestClient自动补全查询`

![image-20231231160026400](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231231160026400.png) 

`RestClient自动补全的查询结果`

![image-20231231160459832](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231231160459832.png) 

# RestClient Result

## Sort  Pagination

简介 : ES中搜索结果的排序和分页与query同级参数,通过request.source()设置;

![image-20231230135825275](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230135825275.png)

`默认排序和距离排序	`

![image-20231230153311621](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230153311621.png) 

## Highlighter

`查询字段添加高亮`

![image-20231230140858178](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230140858178.png) 

`解析添加高亮的字段`

![image-20231230141721325](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230141721325.png) 

```java
     // 解析查询结果的方法
	public void handle(SearchResponse searchResponse){
        // 获取命中的hits
        SearchHits hits = searchResponse.getHits();
        // 1获取返回的total数目
        long total = hits.getTotalHits().value;
        // 2获取hits中的hit数组,将获取的数据转化为HotelDoc存入List集合中
        SearchHit[] arrHits = hits.getHits();

        List<HotelDoc> lists = new ArrayList<HotelDoc>();

        for (SearchHit hit : arrHits) {
            // arrHits数组中的_source数据转换为字符串
            String result = hit.getSourceAsString();
            // 将Json格式字符串,转换为HotelDoc对象
            HotelDoc hotelDoc = JSON.parseObject(result, HotelDoc.class);
            // 获取高亮的数据
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            
            if (!CollectionUtils.isEmpty(highlightFields)){
                // 获取到对应的高亮字符串
                HighlightField highlightField = highlightFields.get("name");
                
                if (highlightField != null){
                    String string = highlightField.getFragments()[0].string();
                    // 替换高亮字符串
                    hotelDoc.setName(string);
                }
            }
            lists.add(hotelDoc);
        }
        System.out.println(lists.toString());
    }
```

# Data Aggregation

简介 :   ES中的数据聚合方便对数据的统计 分析 运算,类似与数据库中的 Group By 和 一些常见的聚合函数Avg Sum Max Min等;

参加聚合的字段必须为` keyword 日期 数值 布尔类型`;

*常见的聚合类型 :*

- 桶聚合: 对文档进行分组

  - TermAggregation: 按照文档字段值进行分组,例如: 品牌 国家等

  - Date Histogram: 按照日期阶梯分组,例如: 一周为一组 一月为一组等

- 管道聚合: 其他聚合的结果为基础做聚合

- 度量聚合: 计算一些值,例如: 最大值 最小值 平均值等

  - Avg: 平均值
  - Max: 最大值
  - Min: 最小值
  - Stats: 同时求取max min avg sum等 



*聚合的三要素 :*

- 聚合名称
- 聚合类型
- 聚合字段

## DSL aggregation

简介 : 通过DSL语法实现聚合操作,参加聚合的字段必须为 keyword 日期 数值 布尔类型等;

```json
GET /hotel/_search
{
    "size": 0, //设置size为0 结果中不包含稳当,只包含聚合结果
    // 定义聚合  目前操作为桶聚合
    "aggs": {
        // 聚合名称
        "brandAggs":{
            // 聚合类型 term
           "terms":{
               "field":"brand", // 参与聚合的字段 默认的排序按照_count降序排列
               "order":{
                 "_count":"asc" // 设置为按照_count降序排列  
               },
               "size":20 // 获取聚合结果的数量
           } 
        }
    }
} 
```

![image-20231230192147620](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230192147620.png) 

```json
GET /hotel/_search
{
    "size": 0, 
    
    "aggs": {

        "brandAggs":{
          
           "terms":{
               "field":"brand", 
               "order":{
                 "_count":"asc"
               },
               "size":20 
           },
           // brands聚合的子聚合,针对分组后每组分别计算 度量聚合	
           "aggs":{
             // 度量聚合的名称  
             "score_stats":{
                // 度量聚合的类型 计算 min max sum avg
               "stats": {
                 // 聚合字段  
                 "field": "score"
               }
             }
           }
           
        }
    }
}
```

![image-20231230192817401](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230192817401.png)

## RestClient aggregation

![image-20231230195242759](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230195242759.png)

![image-20231230195435988](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231230195435988.png) 

```java
public class HotelAggregationDemoApplicationTests {

    @Autowired
    private RestHighLevelClient client;


    @Test
    public void aggregation() throws IOException {
        // 1.创建请求
        SearchRequest searchRequest = new SearchRequest("hotel");

        // 2.设置请求参数
        searchRequest.source().size(0);

        searchRequest.source().aggregation(
                // 设置桶聚合名称
                AggregationBuilders.terms("brand_agg")
                        .field("brand")
                        .size(20)
                        // 默认为false降序 true为升序
                        .order(BucketOrder.count(true))

        );

        searchRequest.source().aggregation(
                // 设置度量聚合
                AggregationBuilders.stats("score_stats")
                        .field("score")
        );

        // 3.发送请求
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        // 4.解析请求的结果
        Aggregations aggregations = searchResponse.getAggregations();

        // 根据名称获取聚合结果
        Terms brandTerms = aggregations.get("brand_agg");

        // 获取桶
        List<? extends Terms.Bucket> buckets = brandTerms.getBuckets();

        for (Terms.Bucket bucket : buckets) {
            // 获取Key值,也就是品牌信息
            String  brandName = bucket.getKeyAsString();
            System.out.println(brandName);
        }
    }
}
```

# Data Synchronization

简介 : ElasticSearch中的数据来源于Mysql,因此当数据库发生改变时,ElasticSearch也必须跟着改变,实现数据之间的同步;

*数据同步的三种策略 :*

- 同步调用: 耦合度高 简单粗暴 

![image-20231231165315087](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231231165315087.png) 

- 异步通知: 耦合度低 难度一般,依赖MQ的可靠性(RabbitMQ推荐)

![image-20231231165327678](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231231165327678.png) 

- 监听biglog: 完全解除服务间的耦合,开启binlog增加数据库负担、实现复杂度高;
  - Mysql开启biglog功能
  - 对数据库的操作会记录在biglog中
  - hotel-demo基于canal监听biglog的变化,实时更新ES数据;

![image-20231231165342954](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20231231165342954.png) 

#  Elasticsearch Cluster

简介 : ElasticSearch单机存储必然`面临海量数据存储和单点故障问题`,通过ES集群可以有效解决这些问题;

*ElasticSearch 集群 :*

- 节点(node): 集群中的ES实例    
- 分片(shard): 索引可以被拆分为不同的部分进行存储,称为分片
- **最佳分片 副本数目+1 <= 节点数目   分片数目 <= 节点数目**

![image-20240101133724835](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240101133724835.png) 

`ES集群的搭建 docker-compose`

```shell
version: '2.2'
services:
  es01:
    image: elasticsearch:7.12.1
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: elasticsearch:7.12.1
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - data02:/usr/share/elasticsearch/data
    ports:
      - 9201:9200
    networks:
      - elastic
  es03:
    image: elasticsearch:7.12.1
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - data03:/usr/share/elasticsearch/data
    networks:
      - elastic
    ports:
      - 9202:9200
volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local

networks:
  elastic:
    driver: bridge
```

`扩展虚拟机的内存大小`

```shell
vi /etc/sysctl.conf

# 添加内容
vm.max_map_count = 262144

# 执行命令,配置生效
sysctl -p

# docker-compose启动集群
docker compose up -d
```

![image-20240101135805016](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240101135805016.png) 

`创建集群下索引库`

![image-20240101140041282](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240101140041282.png) 

`DSL语法创建集群下的索引库`

```json
PUT /cluster
{
  "settings": {
    "number_of_shards": 3, // 分片数量
    "number_of_replicas": 1 // 副本数量
  },
  "mappings": {
    "properties": {
      // mapping映射定义 ...
    }
  }
}
```

![image-20240101140346769](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240101140346769.png) 

 `集群节点的职责划分 :`

![image-20240101151256674](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240101151256674.png) 

- master节点: 对CPU要求高;
- data节点: 对CPU和内存要求都高;
- coordinating节点: 对网络带宽 CPU要求高;

`ES集群职责划分图 :`

![image-20240101151559248](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240101151559248.png) 

## Distributed storage

简介 : ElasticSearch 集群通过hash算法,计算文档应该存储在那个分片中;

![image-20240101151756851](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240101151756851.png) 

- _routing默认为文档的id
- 算法与分片的数量相关,因此索引库一但创建,分片数量不能修改;

`文档存储流程 :`

![image-20240101151918545](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240101151918545.png) 

## Distributed query

简介 : ElasticSearch 集群的查询分为两个阶段`scatter phase(分散阶段) gather phase(聚集阶段)`;

- 分散阶段: coordinating noed 会把请求分发到每一个分片
- 聚集阶段: coordinating node 汇总data node的搜索结果,并处理后返回给用户

 ![image-20240101152343319](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240101152343319.png) 

## Fail Over

简介 : ElasticSearch 集群中的Master node会监控集中的节点状态,发现节点宕机,会将宕机节点的分片数据迁移到其他节点中,确保数据安全;

`集群结构图`

![image-20240101152644682](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240101152644682.png) 

`发生故障后,选举新的主节点,进行数据迁移`

![image-20240101152732949](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20240101152732949.png) 

*Elastic Search集群主节点宕机 :*

- 主节点宕机
- 选举新的主节点
- 分片处理: 主节点的主分片会在其他节点的副分片中,选举新的主分片
- 集群状态变化: 节点宕机会导致集群中绿色变为黄色或红色,若副本分片数量足够,那么集群状态会自动恢复
- 节点恢复: 宕机节点恢复后会接受新的副本分片,并与主分片进行数据同步;

*Elastic Search集群子节点宕机 :*

- 主节点会定期检测节点宕机
- 主分片提升: 宕机节点存在主分片,主节点会将主分片的副本提升为主分片
- 副分片的创建: 宕机节点存在副分片,主节点会在其他节点上创建新的副分片
- 数据同步: 新创建的主和副分片,会从其他副本分片中复制数据保证数据的一致性
- 节点恢复: 宕机节点恢复加入到集群中,这个节点的旧分片和副本会变成副本分片,然后与主分片同步;
