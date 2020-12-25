# ElasticSearch			

[TOC]

​							

### ElasticSearch简介

***

Elasticsearch 是一个分布式的开源搜索和分析引擎，适用于所有类型的数据，包括文本、数字、地理空间、结构化和非结构化数据。Elasticsearch 在 Apache Lucene 的基础上开发而成，由 Elasticsearch N.V.（即现在的 Elastic）于 2010 年首次发布。Elasticsearch 以其简单的 REST 风格 API、分布式特性、速度和可扩展性而闻名，是 Elastic Stack 的核心组件；Elastic Stack 是适用于数据采集、充实、存储、分析和可视化的一组开源工具。人们通常将 Elastic Stack 称为 ELK Stack（代指 Elasticsearch、Logstash 和 Kibana），目前 Elastic Stack 包括一系列丰富的轻量型数据采集代理，这些代理统称为 Beats，可用来向 Elasticsearch 发送数据。

**官方文档**：https://www.elastic.co/guide/en/elasticsearch/reference/6.0/getting-started.html

**中文文档**：https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html

​			

​			

​			

### 基本概念

***

#### Index（索引）

​	动词的情况下，索引一条数据到es，相当于insert一条数据到es；

​	索引还相当于MySQL中的数据库

#### Type（类型)

​	在Index（索引）中，可以定义一个或多个类型。

​	类似于MySQL中的Table，存放一种Type的数据

#### Document（文档）

​	相当于MySQL表中的一条记录，在es里面所有的数据都是json格式的文档

​	<img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201216152439.png" style="zoom: 50%;" />

​				

​				

​				

### 安装ElasticSearch和Kibana

***

#### Docker安装ElasticSearch

* 拉取镜像

  ```bash
  docker pull elasticsearch:7.4.2
  docker pull kibana:7.4.2       #  可视化检索数据，类似于SqlYog，kibana 和 es的版本要统一
  ```

* 创建挂载目录

  ```bash
  mkdir -p /mydata/elasticsearch/config
  mkdir -p /mydata/elasticsearch/data
  chmod -R 777 /mydata/elasticsearch/   # 给文件全部权限
  ```

* 向配置文件中添加内容，"http:host:0.0.0.0"：允许所有主机访问

  ```bash
  echo "http: host: 0.0.0.0" >> /mydata/elasticsearch/config/elasticsearch.yml 
  echo "network.host: 0.0.0.0" >> /mydata/elasticsearch/config/elasticsearch.yml
  # 上面两行配置都是一样的，但是要注意host与地址之间要有空格。
  # 如果配置有误的话可以用yml的格式写，如下
  network:
  	host: 0.0.0.0
  ```

* 运行elasticsearch镜像，9300是集群通信端口，9200是暴露给客户端的端口

  ```bash
  docker run --name=elasticsearch -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e ES_JAVA_OPTS="-Xms64m -Xmx128m" \
  -v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
  -v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
  -v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
  -d elasticsearch:7.4.2
  # 如果启动报错，要么是文件夹没有创建，要么是缺失文件，要么是配置文件有误
  # 挂载目录，即使以后容器删了，相应数据也不会丢失
  ```

* 运行kibana镜像

  ```bash
  docker run --name kibana \
  -e ELASTICSEARCH_HOSTS=http://192.168.203.137:9200 \
  -p 5601:5601 \
  -d kibana:7.4.2
  ```

  ​			

  ​			

  ​			

### 初步检索

***

#### 一、_cat

| 请求方式 | 请求路径      | 功能                   |
| -------- | ------------- | ---------------------- |
| GET      | /             | 查看节点信息           |
| GET      | /_cat/nodes   | 查看所有节点           |
| GET      | /_cat/health  | 查看es健康状况         |
| GET      | /_cat/master  | 查看主节点             |
| GET      | /_cat/indices | 查看所有索引  show dbs |

​				

#### 二、索引一个文档到es

其实就是向es中插入一条数据

保存一个数据：需要指定保存在哪个索引的哪个类型下，指定用哪个唯一标识

| 请求方式    | 请求路径             | 功能                                        |
| ----------- | -------------------- | ------------------------------------------- |
| PUT or POST | /customer/external/1 | 在customer索引下的external类型下插入1号数据 |

<img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201216163017.png" style="zoom:80%;" />

**注意发送的请求方式和发送的数据类型 (JSON类型)** 

POST 和 PUT 的请求方式都可以保存数据，区别是PUT请求必须带上id

* PUT：PUT保存数据的时候必须要带上id，否则会报错，如果带的id已存在，则会更新对应id的数据。

* POST：POST保存数据可以不带id，默认会自动生成id，如果带的id已存在，则会更新对应id的数据。

  ​					

#### 三、查询数据

| 请求方式 | 请求路径             | 功能                                      |
| -------- | -------------------- | ----------------------------------------- |
| GET      | /customer/external/1 | 查询customer索引下external类型下的1号数据 |

​					

#### 四、更新文档

除了以上PUT、POST方式传入同一个id号的更新方式，还可以显示指定更新。

通过url地址后面带上 /_update 显示更新

<img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201216164910.png" style="zoom:70%; border:1px solid #ccc" />

**注意**：

* 这种方式只能用于post请求，且传入的json数据中需要指定 "doc" 字段

* /_update 的方式es会检查更新的数据是否与原来数据一致，如果是一样的，则不会执行更新操作。而直接使用POST方式，不管数据是否一致都会执行更新操作。

  ​				

#### 五、删除

| 请求方式 | 请求路径             | 功能                                      |
| -------- | -------------------- | ----------------------------------------- |
| DELETE   | /customer/external/1 | 删除customer索引下external类型下的1号数据 |
| DELET    | /customer            | 删除customer索引                          |

ES中不支持删除类型的操作	

​			

#### 六、批量操作api，_bulk

| 请求方式 | 请求路径                 | 功能                                       |
| -------- | ------------------------ | ------------------------------------------ |
| POST     | /customer/external/_bulk | 批量向customer索引下external类型下插入数据 |

**语法格式**：两个为一个整体

```json
{action: {metadata}}\n		// 执行的操作：添删改查
{request body}\n			// 发送的数据
```

**实例**

```json
POST /_bulk

// 删除website索引，blog类型下的123数据
{"delete": {"_index":"website", "_type":"blog", "_id":"123"}}

// 创建website索引，创建blog类型，新增123号数据，第二行是具体数据
{"create": {"_index":"website", "_type":"blog", "_id":"123"}}
{"title": "My first blog post"}	

// 向website索引，blog类型下插入一条数据
{"index": {"_index":"website", "_type":"blog"}}
{"title": "My second blog post"}	

// 更新website索引，blog类型下的123号数据，最后一个属性是如果更新失败，重试三次
{"update": {"_index":"website", "_type":"blog", "_id":"123", "_retry_on_conflict": 3}}
{"doc": {"title":"My updated blog post"}}
```

**实例，不能有空行**

<img src="https://kingwait-note.oss-cn-chengdu.aliyuncs.com/20201216171847.png" style="zoom: 80%;" />

**批量插入测试**

es github提供的测试数据

https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json

​			

​			

​			

### 进阶检索

***

#### ES支持两种基本方式检索

* 通过使用REST request URI 发送搜索参数（uri + 检索参数）

  ```javascript
  GET /bank/_search?q=*&sort=account_number:asc
  ```

* 通过使用REST request body 发送（uri + 请求体）	

  ```javascript
  GET /bank/_search
  {
      "query": {
      	"match_all": {}
  	},
      "sort": [
      	{
      		"account_number": "asc"
  		},
          {
              "balance": "desc"
          }
  	]
  }
  ```

  ​					

#### match_all						

`match_all`：查询全部，相当于 /_search 不带条件；GET

```json
{
    "query": {
        "match_all": {}
    },
    "sort": 
    [
    	{
    		"account_number": "desc"
		},
		{
            "balance": "desc"
        }
	]
}
// 排序的另一种写法
GET /bank/_search
{
    "query": {
        "match_all": {}
    },
    "sort":
    [
      {
      	"account_number": {
      		"order": "desc"
  		  }
      }
	]
}
```

​													

#### match

`match`：精准匹配（数值），模糊匹配，分词匹配；GET

```json
// 模糊匹配，分词匹配
{
  "query": {
    "match": {
      "address": "Mill"   // 不止address属性，其他属性，只要包含Mill都会被返回
    }
  }
}

// 精准匹配
{
  "query": {
    "match": {
      "balance": 45801  
    }
  }
}


// 通过 _source 返回部分字段，相当于select field1, field2
{
    "query": {
        "match_all": {}
    },
    "_source": ["balance", "firstname"]
}
```

​					

#### match_phrase

`match_phrase`：短语匹配，短语不会被分割；GET

```json
{
  "query": {
    "match_phrase": {
      "address": "671 Bristol Street"  // 671 Bristol Street会一个总体，不会被分割成多个单词匹配
    }
  }
}

// 另一种方式精准匹配短语
{
  "query": {
    "match": {
      "address.keyword": "671 Bristol Street" // 任何文本都可以 .keyword
    }
  }
}

/**
 *	.keyword 和 match_phrase 的区别
 *  .keyword必须全值匹配，match_phrase可以缺值匹配。
 * 
 *  例如：address: China Beijing CY
 *  .keyword必须是China Beijing CY才能查到这条记录；
 *  match_phrase使用China Beijing依然可以查到
 */

```

​							

#### multi_match	

`multi_match`：多字段匹配；GET

```json
// 相当于 where email = "Lane" or firstname="Lane"
{
  "query": {
    "multi_match": {
      "query": "Lane",
      "fields": ["email", "firstname"]
    }
  }
}
```

​					

#### bool 

`bool`：复合查询，满足多个条件；GET

```json
{
  "query": {
    "bool": {
      "must": [				// 必须满足条件
        {
          "match": { "gender": "F" }
        },
        {
          "match": { "address": "mill" }
        }
      ],
      "must_not": [			// 必须不满足条件
        {
          "match": { "age": "38" }
        }
      ],
      "should": [			// 满足也行，不满足也行
        {
          "match": { "lastname": "Long" }
        }
      ]
    }
  }
}
```

​					

#### filter

`filter`：结果过滤；GET

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "match": { "gender": "F" }
        },
        {
          "match": { "address": "mill" }
        }
      ],
      "must_not": [
        {
          "match": { "age": "38" }
        }
      ],
      "should": [
        {
          "match": { "lastname": "Long" }
        }
      ],	
      "filter": {			// 过滤条件，年龄在 10 - 20 之间的才返回
        "range": {
          "age": { "gte": 10, "lte": 20 }
        }
      }
    }
  }
}
```

​				

#### term

`term`：前面我们用match + 数字，可以实现精准匹配；但是es中，推荐使用term来做精准匹配，match更适合用于文本内容的检索

**全文检索字段用match，其他非text字段匹配用term**

```json
{
  "query": {
    "term": {
      "age": { "value": 28 }
    }
  }
}
```

​						

#### aggregation

`aggregation`：聚合

```json
// 按照年龄聚合，并求出这些年龄段的人的平均薪资
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": {		// 求某个属性的分布情况
        "field": "age",
        "size": 100
      },
      "aggs": {
        "balanceAvg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}

// 查出所有年龄分布，并查出该年龄中每个性别的平均薪资，以及这个年龄段的总体平均薪资
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 100
      },
      "aggs": {
        "genderAgg": {
          // 这里注意gender是text字段，需要.keyword才能查询分布
          "terms": { "field": "gender.keyword", "size": 10},  
          "aggs": { 
              "balanceAvg": { "avg": { "field": "balance" } } 
          }
        },
        "ageBalanceAvg": { "avg": { "field": "balance" } }
      }
    }
  },
  "size": 0
}
```

​				

#### mapping

`mapping`：定义文档被如何处理，映射，可以理解为创建表并定义各个字段的数据类型

在es7及之后的版本中，type（类型）就会被移除

```json
// 添加索引，并指定映射
PUT /my_index
{
  "mappings": {
    "properties": {
      "age": {"type": "integer"},
      "email": {"type": "keyword", 
          "fields": {			// 子属性，大属性里面可以有子属性
          	"keyword": { "type": "keyword", "ignore_above": 256 }
          }
	  },
      "name": {"type": "text"}
    }
  }
}

// 给索引中添加映射字段
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false	// index false 标识这个字段不参与检索条件
    }
  }
}

/**
 * 在es中是不允许修改索引映射的，因为一旦修改，则这个索引下的所有数据的规则都会被破坏。
 * 想要修改索引映射唯一的方法就是，新建一个正确的索引，然后将数据迁移到新索引下
 */
// 数据迁移
POST /_reindex
{
    "source": { "index": "twitter", "type": "account" },
    "dest": { "index": "new_twitter" }		// 在以后的版本中不使用type
}
```

​				

### 分词

****

分词器接收一个字符流，将之分割为独立的词元，然后输出词元流。

例如：分词器遇到空白字符时分割文本，会将文本 “Happy New Year" 分成[Happy, New, Year]；

ES内置了很多分词器，可以用来构建costom analyzers（自定义分词器）

​			

#### 小测试

```json
POST /_analyze
{
  "analyzer": "standard",
  "text": "Happy New Year"
}

// 输出
{
  "tokens" : [
    {
      "token" : "happy",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "new",
      "start_offset" : 6,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "year",
      "start_offset" : 10,
      "end_offset" : 14,
      "type" : "<ALPHANUM>",
      "position" : 2
    }
  ]
}
```

但是es内置的分词器对中文不太友好，例如："刘俊威真帅"，这句话会被分成[刘,俊,威,真,帅]；不太符合中文正常分词。所以在涉及到中文分词的时候我们一般使用开源的**ik分词器**。

​				

#### 安装ik分词器

* 下载地址：https://github.com/medcl/elasticsearch-analysis-ik

* 找到相应版本，版本要和es版本一一对应。去到plugins目录，使用wget命令下载即可

* 解压到插件目录即为安装成功。把分词器解压到es的plugins目录，然后用 **elasticsearch-plugin list** 命令看看是否安装成功

* 安装好后记得重启

```json
// 智能分词
{
    "analyzer": "ik_smart", 
    "text": "我是中国人"
}

// 最多的词语组合
{
    "analyzer": "ik_max_word", 
    "text": "我是中国人"
}
```

​					

#### 自定义词库

官方提供的分词器有很多次识别不到，例如当下的流行语。所以通常我们会自定义词库

#### 扩展分词器的方式

* 修改ik分词器的配置文件，可以指定一个远程词库，让分词器自己发请求获取新词库

* 把新词库放到nginx里面，让分词器向nginx发送请求获取新词库

  ​			

#### 使用nginx的方式自定义词库

##### 小技巧

使用docker挂载目录的时候，经常会遇到主机目录配置文件不齐全的问题，可以启动一个容器，然后拷贝里面的配置文件到主机目录

* 启动一个nginx实例

  ```bash
  docker run -p 80:80 --name nginx -d nginx:1.10
  ```

* 拷贝其配置文件

  ```bash
  docker container cp nginx:/etc/nginx .
  				   #容器名称:容器内的目录 
  ```

  ​		

##### docker安装nginx

```bash
docker run -p 80:80 --name nginx \
-v /mydata/nginx/html:/usr/share/nginx/html \
-v /mydata/nginx/logs:/var/log/nginx \
-v /mydata/nginx/conf:/etc/nginx \
-d nginx:1.10

# 保证映射的目录存在
mkdir -p {/mydata/nginx/html,/mydata/nginx/logs,/mydata/nginx/conf}
```

##### 配置nginx静态资源目录

让外部可以通过url访问到这个分词词库的静态资源就可以了

```bash
# 例如在 /mydata/nginx/html 目录下创建es文件夹，在es文件夹中创建fenci.txt文件
# 通过这样的配置之后，就可以通过 http://f.com/es/fenci.txt 地址访问到这个文件了
# fenci.txt的内容如下（案例）
尚硅谷
刘恺威
刘俊威	
```

##### 修改分词器配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict"></entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords"></entry>
        <!--用户可以在这里配置远程扩展字典 -->
        <entry key="remote_ext_dict">http://192.168.203.137/es/fenci.txt</entry>  <!--主要就是修改这里-->
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

修改了之后记得重启一下es，修改了分词词库也要重启es

​			

​				

​			

### ElasticSearch整合SpringBoot

***

#### 使用Elasticsearch-Rest-Client

操作es的方式有很多，例如：HttpClient、RestClient等等发送请求的工具都可以，因为es本身已经封装到rest请求了。但是这个需要我们自己写DSL语句，和直接发送异步请求没什么区别。所以我们采用 Elasticsearch-Rest-Client 操作es，它帮我们封装了很多，可以简化我们的代码，且Elasticsearch-Rest-Client紧随es的脚步更新。

**官方文档**：https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high.html

* 引入POM

  ```xml
  <dependency>
      <groupId>org.elasticsearch.client</groupId>
      <artifactId>elasticsearch-rest-high-level-client</artifactId>
      <version>7.4.2</version> <!--版本要和使用的es配套-->
  </dependency>
  ```

  SpringBoot父项目对ES版本作了管理，记得修改一下默认的es版本号

  ```xml
  <properties>
      <java.version>1.8</java.version>
      <elasticsearch.version>7.4.2</elasticsearch.version>
  </properties>
  ```

* 编写配置

  ```java
  @Configuration
  public class GulimallElasticSearchConfig {
      @Bean
      public RestHighLevelClient esRestClient() {
          RestHighLevelClient client = new RestHighLevelClient(
                  RestClient.builder(
                          new HttpHost("192.168.203.137", 9200, "http")));
          return client;
      }
  
  	// 这个是RequestOptions，要求每次请求都需要携带RequestOptions
      public static final RequestOptions COMMON_OPTIONS;
      static {
          RequestOptions.Builder builder = RequestOptions.DEFAULT.toBuilder();
  //        builder.addHeader("Authorization", "Bearer " + TOKEN);
  //        builder.setHttpAsyncResponseConsumerFactory(
  //                new HttpAsyncResponseConsumerFactory
  //                        .HeapBufferedResponseConsumerFactory(30 * 1024 * 1024 * 1024));
          COMMON_OPTIONS = builder.build();
      }
  }
  ```

* 测试

  ```java
  @Autowired
  private RestHighLevelClient client;
  
  // 索引一条数据
  public void indexData() throws IOException {
      IndexRequest request = new IndexRequest("users");
      request.id("1");            // 数据id，即使是数字类型的数字也是要转成字符串类型
      User user = new User();
      user.setUserName("zhangsan");
      user.setGender("男");
      user.setAge(18);
      String jsonString = JSON.toJSONString(user);
      request.source(jsonString, XContentType.JSON); // 要保存的内容，记得传第二个参数数据类型
  
      // 执行操作，这个是同步执行，还有异步执行
      IndexResponse index = client.index(request, GulimallElasticSearchConfig.COMMON_OPTIONS);
  
      // 返回值
      System.out.println(index);
  }
  
  // 检索数据
  public void testSearchData() throws IOException {
      SearchRequest searchRequest = new SearchRequest();
      searchRequest.indices("users");
      SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
      searchSourceBuilder.query(QueryBuilders.matchQuery("userName", "zhangsan"));
      searchRequest.source(searchSourceBuilder);
  
      SearchResponse searchResponse = 
          client.search(searchRequest, GulimallElasticSearchConfig.COMMON_OPTIONS);
      System.out.println(searchResponse);
  }
  
  // 聚合
  public void testAggregationData() throws IOException {
      SearchRequest request = new SearchRequest();
      SearchSourceBuilder builder = new SearchSourceBuilder();
      // 按照年龄的分布进行聚合
      TermsAggregationBuilder ageAgg = AggregationBuilders.terms("aggAgg").field("age").size(10);
      // ageAgg.subAggregation(***);   // 子聚合
      builder.aggregation(ageAgg);
  
      // 计算平均薪资
      AvgAggregationBuilder balanceAgg = AggregationBuilders.avg("balanceAvg").field("balance");
      builder.aggregation(balanceAgg);
  
      // System.out.println(builder.toString());
  
      request.source(builder);
      SearchResponse search = client.search(request, GulimallElasticSearchConfig.COMMON_OPTIONS);
  
      // 获取数据并封装对象
      SearchHit[] hits = search.getHits().getHits();
      for (SearchHit hit : hits) {
          // System.out.println(hit.getSourceAsString());
          User user = JSON.parseObject(hit.getSourceAsString(), User.class);
          System.out.println(user);
      }
  
      // 获取聚合信息
      Aggregations aggregations = search.getAggregations();
      Terms ageAgg1 = aggregations.get("ageAgg"); // 根据聚合名字获取聚合
      for (Terms.Bucket bucket : ageAgg1.getBuckets()) {
          System.out.println(bucket.getKeyAsString());     
          System.out.println(bucket.getDocCountError());
      }
      /**
       * 其实就和手写DSL发请求一样，请求响应回来的数据都有api能够获取
       */
  }
  ```

  





