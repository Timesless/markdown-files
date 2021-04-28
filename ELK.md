

## ElasticSearch



> 分布式全文检索引擎
>
> https://www.elastic.co
>
>  
>
> **es 自动集群，默认集群名称：elasticsearch**



### 安装

#### Docker

1. 下载镜像

``` shell
docker pull elasticsearch:7.12.0
docker pull kibaba:7.12.0
```

2. 创建实例

``` shell'
# 创建 config 和 data 目录做 容器运行的映射
mkdir -p /Docker/elasticsearch/config
mkdir -p /Docker/elasticsearch/data
mkdir -p /Docker/elasticsearch/plugins
echo "http.host:0.0.0.0" >> /Docker/elasticsearch/config/elasticsearch.yml

# 9200 rest 访问暴露端口，集群模式下 9300 为节点的通信端口
# -v 挂载目录
# plugins 插件目录，如 ik 分词器

docker run --name elasticsearch -p 9200:9200 -p 9300:9300\
-e "discovery.type=single-node"\
-e ES_JAVA_OPTS="-Xms256m -Xmx256m"\
-v /d/Docker/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml\
-v /d/Docker/elasticsearch/data:/usr/share/elasticsearch/data\
-v /d/Docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins\
-d elasticsearch:7.12.0

# 查看日志
docker logs containerId

# 自动启动
docker update containerId --restart=always
```



##### windows

> 配置文件只覆盖 elasticsearch.yml

``` shell
docker run --name elasticsearch -v /d/Docker/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /d/Docker/elasticsearch/data/:/usr/share/elasticsearch/data -v /d/Docker/elasticsearch/plugins/:/usr/share/elasticsearch/plugins -v /d/Docker/elasticsearch/logs/:/usr/share/elasticsearch/logs -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -d elasticsearch:7.12.0

docker update containerId --restart=always
```



#### Kibana

``` shell
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.0.104:9200 -p 5601:5601\
-d kibana:7.12.0
```



### ==检索进阶==

> 有两种方式
>
> 1. query
> 2. request body「**query DSL**」



#### query

``` shell
GET bank/_search?q=*&sort=account_number:asc

```



#### query DSL

##### 示例一

``` shell
GET bank/_search
{
	"query": {
		"match_all": {}
	},
	# 排序
	"sort": [
		{
			"account_number": "asc",
			"balance": "desc"
		}
	],
	# 分页
	"from": 0,
	"size": 5,
	# 指定字段查询
	"_source": ["firstname", "balance"]
}
```



##### 示例二

> 倒排索引进行全文检索
>
> 1. match
> 2. match_phase
> 3. multi-match

``` shell
# 结果按 _score 排序
GET bank/_search
{
	"query": {
		# 对检索条件「mill movico」进行分词匹配
		"match": {
			"address": "mill movico"
		},
		
		# 不分词匹配
		"match_phase": {
			"address": "mill movico"
			# 精确匹配：mill movico cc 不能被查询到
			# "address.keyword": "mill movico"
		},
		
		# 多字段匹配
		"multi_match": {
			"query": "mill movico",
			# address 或者 city 字段匹配 mill movico
			"fields": ["address", "city"]
		}
	}
}
```



##### 示例三：复合查询 bool

``` shell
GET bank/_search
{
	"query": {
		"bool": {
			# 必须 类似 =
			"must": [
				{
          "match": {
            "gender": "F"
          }
				},
				{
					"match": {
						"address": "mill"
					}
				}
			],
			# 必须不 类似 !=
			"must-not": [
				{
					"match": {
						"age": "38"
					}
				}
			],
			# 应该满足 类似 or
			"should": [
				{
					"match": {
						"lastname": "Wallace"
					}
				}
			],
			"filter": {
				"range": {
					...
				}
			}
		}
	}
}
```



##### 示例四：filter

> **filter 不贡献 score**
>
> must / must-not /should 会计算 score

``` shell
GET bank/_search
{
	"query": {
		"bool": {
			# "must": {
			"filter": {
				"range": {
					"age": {
						"gte": 18,
						"lte": 30
					}
				}
			}
		}
	}
}
```



##### 示例五：term

> 对于非 text 字段的精确匹配使用 term
>
> 对于 text 字段的精确匹配使用 column.keyword

``` shell
GET bank/_seach
{
	"query": {
		"term": {
			"age": 28
		}
	}
}
```



#### 聚合



### 映射



#### 创建索引的映射

> 整数数值类型默认 Long
>
> keyword 类型：不做全文检索，而是精确匹配
>
> text 类型：在保存数据的时候进行分词，检索时按照分词全文检索

``` shell
PUT /my_index
{
	"mappings": {
		"properties": {
			"age": {"type": "integer"},
			"email": {"type": "keyword"},
			"name": {"type": "text"}
		}
	}
}
```



#### 添加索引的映射

``` shell
PUT /my_index/_mapping
{
	"properties": {
		"type": "keyword",
		"index": false
	}
}
```



#### 修改索引的映射

#### reindex

**无法修改索引的映射**

1. 创建一个新的索引，指定正确的索引关系
2. 数据迁移

``` shell
# 1
PUT /newbank
{
	"mappings": {
		"properties": {
			"account_number": {
				"type": "long",
				...
			}
		}
	}
}

# 2
# 6.0 以后 _reindex
POST _reindex
{
	"source": {
		"index": "bank"
	},
	"dest": {
		"index": "newbank"
	}
}

# 6.0 以前的索引迁移到 6.0 以后
# type 都会修改为 _doc
POST _reindex
{
	"source": {
		"index": "bank",
		# 需要指定 type
		"type": "account"
	},
	"dest": {
		"index": "newbank"
	}
}
```





### ES 概念

#### 索引

对应关系型数据库的库

#### 文档

对应关系型数据库表中一条记录

#### 类型

~~默认"_doc"类型，已弃用~~

#### 分片

> 分片：是ES中所有数据的文件快，也是数据的最小单元，整个ES集群的核心就是对所有分片进行分布，索引，负载，路由，分片平均的存储整个集群的所有数据
>
> 默认5个分片（primary shard，主分片），每个主分片将有一个副本（repica shard，复制分片）
>
> 主分片与复制分片会在不同节点中，==一个分片是一个Lucene索引（一个包含倒排索引的文件目录）==
>
> **倒排索引**：倒排索引包含一个有序列表，列表包含所有文档出现过的不重复个体（term，词元），每个词元包含了它所有曾出现过的文档列表

doc1： good good study, up up every day

doc2：day day up, good good study

| term（词元） | doc1 | doc2 | 倒排索引 |
| ------------ | ---- | ---- | -------- |
| day          | √    | √    | 1，2     |
| every        | √    | ×    | 1        |
| good         | √    | √    | 1，2     |
| study        | √    | √    | 1，2     |
| up           | √    | √    | 1，2     |



### 分词

**分词：一个 tokenizer「分词器」接受一个字符流，将其分割为独立的 token「词元」，然后输出 tokens 流**



#### standard

``` shell
# 标准分词器

POST _analyze
{
	"analyzer": "standard",
	"text": "尚硅谷电商项目"
}

# 结果
"尚"
"硅"
"谷"
"电"
"商"
"项"
"目"
```



#### ik 分词器

> 中文分词器：ik
>
> + ik_smart
> + id_max_word
>

```  shell
# install 进入 es 安装目录 plugins
cd $ES_HOME$/plugins && mkdir ik
# 解压到ik文件夹
unzip elastic-search-analysis-ik-7.12.0.zip

# chmod
chmod -R 777 ik/

# 将我们需要的业务词元添加到自定义字典中
IKAnalyzer.cfg.xml中<entry key="ext_dict">my.dic</entry>
```



ik_smart

``` shell
POST _analyze
{
	"analyzer": "ik_smart",
	"text": "我是中国人"
}

# 结果
我
是
中国人
```

id_max_word

``` shell
POST _analyze
{
	"analyzer": "ik_max_word",
	"text": "我是中国人"
}

# 结果
我
是
中国人
中国
国人
```



#### 自定义词库



##### nginx 存放词库

由 ik 分词器向 nginx 发起请求获取新词库



1. 安装 nginx

> 1. 随意启动一个 nginx 实例，只是为了复制其配置
>
>    `docker run -p 80:80 --name nginx -d nginx:1.10`
>
> 2. 拷贝 nginx 运行的文件到宿主机
>
>    `docker container cp nginx:/etc/nginx ./conf`
>
>    **复制 nginx 容器下 /etc/nginx 文件夹到当前目录的 conf 下**
>
> 3. 删除 nginx 实例
>
>    `docker stop nginx docker rm nginx `
>
> 4. 执行以下命令

``` shell
mkdir nginx
mv conf nginx/

docker run -p 80:80 --name nginx\
-v /yangzl/docker/nginx/html:/usr/share/niginx/html\
-v /yangzl/docker/nginx/logs:/var/log/nginx\
-v /yangzl/docker/nginx/conf:/etc/nginx\
-d nginx:1.10

```

> 此时，nginx 目录下有三个文件夹
>
> + conf
> + html
> + logs

2. 继续创建文件夹 es

``` shell
vim html/index.html

mkdir es
cd es
vim terms.txt

尚硅谷
巧碧螺

shift zz
```

3. 继续回到 ik 分词器的 conf

``` shell
cd ik/conf

vim IKAnalyzer.cfg.xml

# 远程字典配置
<entry key="remote_ext_dict">http://192.168.0.104:80/es/terms.txt</entry>

# 本地字典配置
```





### Rest API

==**API 地址：**==[es-api](elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-document-index.html)

+ PUT

创建（指定文档ID）：`localhost:9200/idx/_doc/id`

+ POST

创建（随机文档ID）：`localhost:9200/idx/_doc`

修改：`localhost:9200/idx/_doc/doctId/_update`

查询所有数据：`localhost:9200/idx/_doc/_search`

+ DELETE

删除：`localhost:9200/idx/_doc/docId`

+ GET

通过文档ID查询数据：`localhost:9200/idx/type/docId`



#### \_cat

``` shell
# 所有节点
GET /_cat/nodes
# es 健康状况
GET /_cat/health
# 查看 master 节点
GET /_cat/master
# 查看所有索引 show database;
GET /_cat/indices
```



#### 保存文档

+ PUT「必须携带 ID」
+ POST

``` shell
# index1 索引名称
# _doc「typeless」
# 1001 文档 id
PUT index1/_doc/1001
```



+ Postman

+ IDEA HTTP Client

``` shell
###
POST http://127.0.0.1:9200/index1/_doc/1001
Content-Type: application/json

{
  "name": "Jone Doe"
}
```



##### 乐观锁机制

``` shell
### 乐观锁更新请求1
POST http://127.0.0.1:9200/index1/_doc/1001?if_seq_no=3&if_primary_term=1
Content-Type: application/json

{
  "name": "Jone Doe2"
}

### 乐观锁更新请求2
POST http://127.0.0.1:9200/index1/_doc/1001?if_seq_no=3&if_primary_term=1
Content-Type: application/json

{
  "name": "Jone Doe2"
}


# 第二个更新请求结果 409 
{
  "error": {
    "type": "version_conflict_engine_exception",
    "reason": "[1001]: version conflict, required seqNo [3], primary term [1]. current document has seqNo [4] and primary term [1]",
    "index_uuid": "e-eWHvqrRx2FDF4Fsyo4ng",
    "shard": "0",
    "index": "index1"
  },
  "status": 409
}
```



#### 查询文档

+ Postman
+ IDEA HTTP Client

``` shell
###
GET http://127.0.0.1:9200/index1/_doc/1001

HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8

{
  "_index": "index1",
  "_type": "_doc",
  "_id": "1001",
  "_version": 3,
  "_seq_no": 3,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Jone Doe"
  }
}

# 业务可根据该字段自己做乐观更新
# ?if_seq_no=1&if_primary_term=1
_seq_no：并发控制字段，每个事务请求都会 +1，提供乐观锁机制

```



#### 更新文档

``` shell
# 一
# 对比数据是否变化，如果数据无变化，则返回 noop
# 需要将数据放在 doc 中
POST index1/_doc/1001/_update

{
	"doc": {
		"name": "John3"
	}
}

# 二
# 版本号 seq_no 变化
POST index1/_doc/1001

# 三
PUT index1/_doc/1001
```



#### 删除

+ 删除文档
+ 删除索引

``` shell
DELETE index1/_doc/1001

# 删除索引
DELETE index1
```



#### bulk API

``` shell
POST index1/_doc/_bulk
Content-Type:application/json

# 两行为一个整体
{"index": {"_id": "1"}}
{"name": "John Doe"}
{"index": {"_id": "2"}}
{"name": "Jane Done"}
```



##### bulk CRUD

``` shell
POST /_bulk
{"delete":{"_index": "index1", "_id": "1001"}}
{"create":{"_index": "index1", "_id": "1002"}}
{"title": "My first blog"}
{"index": {"_index": "index1"}}
{"title": "My second blog"}
{"update": {"_index": "index1", "_id": "1002"}}
{"doc": {"title": "My first update blog"}}
```

> **delete 单独一行**
>
> 其他有提交数据的每两行为一个整体



### 数据类型

+ 字符串
    + text
    + keyword，分词器不解析

+ 数值类型

    long，integer，short，byte，double，float，half_float，scaled_float

+ 日期类型 date

+ 布尔类型 boolean

+ 二进制 binary



###  CRUD

``` shell
# 在Kibana中发送Rest请求

# RESTful风格的创建

# 不指定数据类型
PUT /myidx/_doc/1
{
	"name": "yangzl",
	"age": 24
}

# 指定数据类型，keyword不被分词器解析
PUT /myidx
{
	”mappings“: {
		"properties": {
			"name": { "type": "keyword" },
			"desc": { "type": "text" }
		}
	}
}
PUT /myidx/_doc/1
{
	"name": "yangzl",
	"desc": "我是一颗小虎牙Python"
}
```

创建之后数据结构如下：

| _index | _type | _id  | _score | name   | age  |
| ------ | ----- | ---- | ------ | ------ | ---- |
| myidx  | _doc  | 1    | 1      | yangzl | 24   |



``` shell
# 修改
# PUT也可以，当数据缺失时会覆盖掉原数据
POST /myidx/_doc/1/_update
{
	"doc": {
		"name": "法外狂徒"
	}
}

# RESTful风格删除
DELETE /myidx # 删除索引
DELETE /myidx/_doc/1	# 删除文档

# 查询
# Query parameters
# h代表要查询的字段，以逗号分隔
# format: JSON, YAML, etc
# local: if true requests retrieve from local node only, else from the master node
# v: includes column headings or not
# s: 以列的别名排序字段名，多个以逗号分隔
GET /myidx/_doc/1
GET /myidx/_doc/1/_search?q=name:yangzl


#*****************************************
# 复杂查询（构建查询，使用Client时对应API）
GET /myidx/_doc/_search
{
	"query": {
    	"match": { "name": "yangzl" }
	},
	# 查询指定字段
	"_source": ["name", "age"]，
	# 排序
	“sort”: [
		{
			"age": { "order": "asc" }
		}
	],
	# 分页，同LIMIT
	"from": 0,
	"size": 10,
	# 高亮结果
	"highlight": {
		"pre_tags": "<em>",
		"post_tags": "</em>",
		"fields": {
			"name": {}
		}
	}
}

# 组合查询
# 使用 bool 过滤器，通过and、or、not逻辑，组合多个过滤器，判断文档是否应该包含在结果中

GET /myidx/_doc/_search
{
  "query": {
    "bool": {
      # =，多个条件以空格分隔(quick fast)
      "must":     { "match": { "title": "quick fast" }},
      # !=
      "must_not": { "match": { "title": "lazy"  }},
      # or
      "should": [
                  { "match": { "title": "brown" }},
                  { "match": { "title": "dog"   }}
      ],
      # 过滤
      "filter": {
      	"range": {
      	  "age": {
      	    "gte": 10,
      	    "lte": 25
      	  }
      	}
      }
    }
  }
}

```



### bulk API

> bulk
>
> `POST index/type/_bulk`

``` shell
# 批量操作
POST /index/type/_bulk
# 每两行为一组数据，_id指定文档ID
# 第二行为传输的数据
{"index": {"_id": "1101"}}
{"name": "doug lea"}
```



> ”match“：模糊匹配
>
> “term”：精确匹配



### 集成Spring Boot

> Spring Data ElasticSearch
>
> [spring-elasticsearch](https://www.github.com/Timessless/spring-in-action5)

**API 地址：**[es-api](elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-document-index.html)



#### TCP 操作 es

9300 端口

> + 建立长连接
>
> + 官方不推荐使用



#### HTTP 操作 es

9200 端口

##### Elasticsearch-Rest-Client



###### Java Low Level REST Client

对 es api 封装度较低



###### Java High Level REST Client

**==对 es api，query DSL 封装较高，推荐使用==**



#### 集成

```
mall-search

选择 web
```

1. 添加依赖 high-level-client

``` xml
<dependency>
	<groupId>org.elasticsearch.client</groupId>
  <artifactId>elasticsearch-rest-high-level-client</artifactId>
  <version>7.12.0</version>
</dependency>
```

``` groovy
dependencies {
  compile 'org.elasticsearch.client:elasticsearch-rest-high-level-client:7.12.0'
}
```

2. mall-search.pom

``` xml
<properties>
	<elasticsearch.version>7.12.0</elasticsearch.version>
</properties>
```

3. 配置

``` java
@Configuration
class ElasticSearchConfig {
  
  /**
   * 添加一些通用请求头
   */
  public static final RequestOptions COMMON_OPTIONS;
  static {
    RequestOptions.Builder builder = RequestOptions.DEFAULT.toBuilder();
    builder.addHeader("Authorization", TOKEN);
    
    COMMON_OPTIONS = builder.build();
  }
  
  @Bean
  public RestHighLevelClient restHighLevelClient() {
    RestClientBuilder builder = RestClient.builder(new HttpHost("192.168.0.104", 9200, "http"));
    return new RestHighLevelClient(builder);
  }
}
```







###  2. 测试

> POST  /accounts/\_doc/\_bulk
>
> es查询领域特定语言：`queryDSL`



#### 2.2 检索

``` shell
# github测试数据练习

# match_phrase全文检索，不分词
GET accounts/_search
{
  "query": {
    "match_phrase": { "address": "mill lane"  }
  }
}

# multi_match，会分词
GET accounts/_search
{
  "query": {
    "multi_match": {
      "query": "mill lane",
      "fields": ["address", "city"]
    }
  }
}


# 复合查询 bool
# must 相当于 =
# must_not 相当于 != ，不会贡献得分
# should 相当于 or
GET accounts/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "gender": "F"}
          
        }, {
          "match": { "address": "mill" }
        }
      ],
      "must_not": [
        {
          "match": { "age": "38" }
        }
      ]
    }
  }
}


# filter过滤，不计算score
GET accounts/_search
{
  "query": {
    "bool": {
      "filter": [
        {"range": {
          "age": {
            "gte": 20,
            "lte": 30
          }
        }}
      ]
    }
  }
}

# match 全文检索，term精确匹配 == 字段.keyword == match_phrase

# TODO
GET accounts/_search
{
	"query": {
		"term": {
			"age": "28"
		}
	}
}

GET accounts/_search
{
	"query": {
		"match": {
			"address.keyword": "mill lane"
		}
	}
}
```



#### 2.3 聚合

``` shell

# 聚合

# 平均年龄
GET accounts/_search
{
  "query": {
    "match": { "address": "mill lane" }
  },
  "aggs": {
  	"aggAgg": {
  		"terms":{
  			"field": "age",
  			"size": 10
  		}
  	},
    "ageAvg": {
      "avg": {
        "field": "age"
      }
    }
  }
}
```



#### 2.4 映射

> `GET acccounts/_mapping`



``` shell
# 数据迁移

POST _reindex
{
	"source": { "index": "accounts" },
	"dest": { "index": "new_acccounts" }
}
```



#### 2.5 分词

> `POST _analyze`



``` shell
POST _analyze
{
	"analyzier": "stardard",
	"text": "this is a english word"
}
```





















































## Kibana

// TODO



## Logstash



