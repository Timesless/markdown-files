

## ElasticSearch



> 分布式全文检索引擎
>
> https://www.elastic.co
>
>  
>
> 自动集群，默认elasticsearch

#### 1.0 ES 概念

+ 索引

对应关系型数据库的库

+ 文档

对应关系型数据库表中一条记录

+ 类型

~~默认"_doc"类型，已弃用~~

+ 分片

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



#### 1.1 分词器

> 中文分词器：ik
>
> + ik_smart
> + id_max_word
>
> 将我们需要的业务词元添加到自定义字典中
>
> IKAnalyzer.cfg.xml中<entry key="ext_dict"> my.dic </entry>

``` shell
# install
cd es_home/plugins && mkdir ik
# 解压到ik文件夹
```



#### 1.2 Rest API

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



#### 1.3 数据类型

+ 字符串
    + text
    + keyword，分词器不解析

+ 数值类型

    long，integer，short，byte，double，float，half_float，scaled_float

+ 日期类型 date

+ 布尔类型 boolean

+ 二进制 binary



####  1.4 CRUD

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



#### 1.5 批量API

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



#### 1.6 集成Spring Boot

> Spring Data ElasticSearch
>
> [spring-elasticsearch](https://www.github.com/Timessless/spring-in-action5)



### 2 测试

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



