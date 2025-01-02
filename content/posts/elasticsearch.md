---
title: '初识ElasticSearch'
date: 2024-12-10T12:50:00+08:00
draft: false
layout: "post"
images: 
  - "/avatar.jpg"   # 设置封面图
description: ""     
categories: ["笔记"]
tags: ["中间件","后端"]
---
#  ElasticSearch

## 认识ElasticSearch

### 倒排索引

传统的数据库（MYSQL），采用的是正向索引，自上向下根据id对数据进行搜索。基于文档创建ID，根据Id查询，必须要先找到文档之后，最后判断是否包含目标词条。

- 文档（document）： 每一条数据（记录），就是一个文档。
- 词条（term），将文档按照**语义**分成的词语。
- 将所有的文档分词，根据分得到的每个词条创建索引，记录词条所在的文档id。得到**词条列表（包含文档id）**

#### 搜索过程

- 搜索“华为手机” -> 分词 -> “华为” + “手机”。
- 在根据分得到的两个词条，查询**词条列表**搜索文档id。
- 根据文档id查询文档。

### IK分词器

一共分为两种**分词**方式。

- `ik_smart` `ik_max_word`

```json
POST /_analyze
{
  "analyzer": "ik_smart",
  "text": "让我们草着走，来吧孩子们！"
}
```

结果

```json
{
  "tokens" : [
    {
      "token" : "让我们",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "草",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "着",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 2
    },
    {
      "token" : "走",
      "start_offset" : 5,
      "end_offset" : 6,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "来吧",
      "start_offset" : 7,
      "end_offset" : 9,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "孩子们",
      "start_offset" : 9,
      "end_offset" : 12,
      "type" : "CN_WORD",
      "position" : 5
    }
  ]
}
```

同时IK分词器允许**拓展词典**增加自定义的词库。

```xml
// /var/lib/docker/volumes/es-plugins/_data/ik/config
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict"></entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords"></entry>
        <!--用户可以在这里配置远程扩展字典 -->
        <!-- <entry key="remote_ext_dict">words_location</entry> -->
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>

```

### 基础概念

- 在ElasticSearch中，文档（记录）数据序列化为Json格式存放。
- **索引**（库）（index）：相同类型的文档集合，类似于数据库中表（table）
- **映射**（mapping）：索引中文档的约束，类似于数据库表结构（Schema）
- **Field**：Json中的字段 , 类似于数据库中的列（column）
- **DSL：**查询语言，提供的JSON风格的请求语句，用于定义搜索条件。

#### Mapping映射属性

- T**ype**
  - 字符串：text（可以进行分词的文本）、**keyword**（精确值，不能分词的文本：国家、品牌、ip地址）
  - ...
- **Index**: 是否创建索引、默认为true
- **Analyzer**：使用的分词器
- **Properties**：该字段的字子字段

### 索引库操作

```json
# 创建索引库，设置mapping映射
PUT /eraser
{
  "mappings": {
    "properties": {
      "info": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "age": {
        "type": "byte"
      },
      "email": {
        "type": "keyword",
        "index": false
      },
      "name": {
        "type": "object",
        "properties": {
          "firstName": {
            "type": "keyword"
          },
          "lastName": {
            "type": "keyword"
          }
        }
      }
    }
  }
}

# 查询索引库
GET /eraser
```

索引库和mapping一旦创建之后就无法修改，但是只能够添加新的字段

```json
# 修改索引库
PUT /eraser/_mapping
{
  "properties": {
    "home": {
      "type": "keyword"
    }
  }
}
```

### 对文档的CRUD

#### 新增文档数据

```json
# 新增文档数据
POST /easer/_doc/1 # 文档id
{
  "info": "sauber车队换胎工",
  "email": "zy@itcast.cn",
    "name": {
        "firstName": "云",
        "lastName": "赵"
    }
}
```

返回结果

```json
{
  "_index" : "easer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

#### 查询文档数据

```json
# 
GET /easer/_doc/1

{
  "_index" : "easer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "info" : "sauber车队换胎工",
    "email" : "zy@itcast.cn",
    "name" : {
      "firstName" : "云",
      "lastName" : "赵"
    }
  }
}
```

#### 修改文档数据

##### **全量修改**

会替换之前的文档数据，先删除，再新增。

```json
PUT /easer/_doc/1
```

##### 增量修改

修改指定字段值(使用**_update**路径)

```json
# 增量修改
POST /easer/_update/1
{
  "doc": {
    "email": "hahahah@outlook.com"
  }
}
```

### 文档批量处理

```json
# 批量处理新增
POST /_bulk // 直接使用bulk批量路径
{"index": {"_index":"eraser", "_id": "3"}} // 要操作的索引库，指定表id
{"info": "C++讲师", "email": "ww@itcast.cn", "name":{"firstName": "五", "lastName":"王"}}
{"index": {"_index":"eraser", "_id": "4"}}
{"info": "前端讲师", "email": "zhangsan@itcast.cn", "name":{"firstName": "三", "lastName":"张"}}

# 批量删除
POST /_bulk
{"delete":{"_index":"eraser", "_id": "3"}}
{"delete":{"_index":"eraser", "_id": "4"}}

```

## JavaRestClient

初始化测试

```java
//@SpringBootTest
public class ElasticTest {
    private RestHighLevelClient client;

    @Test
    void testClient() {
        System.out.println(client);
    }

    @BeforeEach
    void setUp() throws Exception {
        client = new RestHighLevelClient(RestClient
                .builder(new HttpHost("localhost", 9200, "http")));
    }

    // 单元测试值之后销毁
    @AfterEach
    void tearDown() throws Exception {
        if (client != null) {
            client.close();
        }
    }
}

```

CURD, 创建一个商品的索引库

```java
MAPPING_TEMPLATE(json) = 
    {
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "name":{
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "price":{
        "type": "integer"
      },
      "stock":{
        "type": "integer"
      },
      "image":{
        "type": "keyword",
        "index": false
      },
      "category":{
        "type": "keyword"
      },
      "brand":{
        "type": "keyword"
      },
      "sold":{
        "type": "integer"
      },
      "commentCount":{
        "type": "integer",
        "index": false
      },
      "isAD":{
        "type": "boolean"
      },
      "updateTime":{
        "type": "date"
      }
    }
  }
}

@BeforeEach
    void setUp() throws Exception {
        client = new RestHighLevelClient(RestClient
                .builder(new HttpHost("localhost", 9200, "http")));
    }

    @Test
    void testCreateIndex() throws IOException {
        //1. prepare the Request Class
        CreateIndexRequest createIndexRequest = new CreateIndexRequest("items");
        // 将映射文件进行传递
        createIndexRequest.source(MAPPING_TEMPLATE, XContentType.JSON);
        // 客户端创建
        client.indices().create(createIndexRequest, RequestOptions.DEFAULT);
    }

    /**
     * 测试get请求
     * @throws IOException
     */
    @Test
    void testGetIndex() throws IOException {
        GetIndexRequest getIndexRequest = new GetIndexRequest("items");
        GetIndexResponse getIndexResponse = client.indices().get(getIndexRequest, RequestOptions.DEFAULT);
        System.out.println(getIndexResponse.toString());
    }
```

### 文档操作

- 需要新建对应的Doc实体对象 -> Json文件
- **source**文件需要指定相应的格式，JSON等。
- 查询api中，由于查到的最终结果，想要的查询结果是在"source"属性中，所以需要多一步处理的结果

```java
	@Test
	// 这里的index对应的就是原生请求路径中的"/items/_index"

	// 既可以修改（全量修改更新)，又可以新增
    void testIndexDoc() throws IOException {
        //0. 准备文档数据
        Item item = itemService.getById(100000011127L);
        ItemDoc itemDoc = BeanUtils.copyProperties(item, ItemDoc.class);
        // 初始化IndexRequest
        IndexRequest items = new IndexRequest("items").
                id(itemDoc.getId()).
                source(JSONUtil.toJsonStr(itemDoc), XContentType.JSON);
        // 新增 对应的是路径“_index"
        client.index(items, RequestOptions.DEFAULT);
    }

	@Test
    void testGetDoc() throws IOException {
        GetRequest getRequest = new GetRequest("items", "100000011127");
        GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);
        String sourceJson = getResponse.getSourceAsString();
        // 将JSON转换为Bean
        ItemDoc bean = JSONUtil.toBean(sourceJson, ItemDoc.class);
        System.out.println(bean);
    }


```

#### 增量更新

```java
@Test
    void testUpdateDoc() throws IOException {
        UpdateRequest updateRequest = new UpdateRequest("items", "100000011127");
        // 增量更新
        // 也可以使用Map的方式
        updateRequest.doc(
                "price", 18,
                "name", "Rose"
        );
        UpdateResponse update = client.update(updateRequest, RequestOptions.DEFAULT);
    }
```

#### 文档批处理（bulk）

```java
    @Test
    void testBulk() throws IOException {
        // 分页拿取的数据库中数以万计的数据
        // 防止多次对数据库进行请求
        int pageNo = 1, pageSize = 500;
        Page<Item> page = itemService.lambdaQuery()
                .eq(Item::getStatus, 1)
                .page(Page.of(pageNo, pageSize));
        List<Item> records = page.getRecords();
        if (records == null || records.isEmpty()) {
            // 并没有查询得到
            return;
        }

        BulkRequest bulkRequest = new BulkRequest();
        for (Item item : records) {
            ItemDoc itemDoc = BeanUtils.copyProperties(item, ItemDoc.class);
            bulkRequest.add(new IndexRequest("items").id(itemDoc.getId())
                    .source(JSONUtil.toJsonStr(itemDoc), XContentType.JSON));
        }
        client.bulk(bulkRequest, RequestOptions.DEFAULT);
    }
```

## DSL查询

### 基本语法

```json
GET /items/_search // GET/index_name/_search
{
  "query": {
    "match_all": {}
  }
}
```

### 叶子查询

#### 全文检索（full text）查询

利用分词器对用户输入内容进行分词搜索，在词条列表中去匹配（等同于倒排索引）。

- `match_query` 根据一个字段进行查询
- `multi_match_query` 根据多个字段查询，参与的字段越多，查询的性能越差。

##### match查询

```JSON
# 单字段查询match query
GET /items/_search
{
  "query": {
    "match": {
      "name": "脱脂牛奶"
    }
  }
} 

# 多字段查询
GET /items/_search
{
  "query": {
    "multi_match": {
      "query": "脱脂牛奶",
       "fields": ["name", "desc"],
    }
  }
} 
```

多字段查询`multi_match_query`格式如上，可以对多个字段进行查询目标。

单字段查询结果

```json
{
  "took" : 27,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1149,
      "relation" : "eq"
    },
    "max_score" : 16.198345, # 查询的得分
    "hits" : [
      {
        "_index" : "items",
        "_type" : "_doc",
        "_id" : "33449279171",
        "_score" : 16.198345,
        "_source" : {
          "id" : "33449279171",
          "name" : "意大利 进口牛奶 葛兰纳诺脱脂纯牛奶 成人牛奶  进口脱脂纯牛奶1Lx6盒",
          "price" : 3500,
          "image" : "https://m.360buyimg.com/mobilecms/s720x720_jfs/t1/25045/9/2656/164517/5c20699dE9b7f4c9c/1a05e9bdd2c5d59e.jpg!q70.jpg.webp",
          "category" : "牛奶",
          "brand" : "葛兰纳诺",
          "sold" : 0,
          "commentCount" : 0,
          "isAD" : false,
          "updateTime" : 1556640000000
        }
      },
```

#### 精确查询

Term-level qeury 

**不对用户输入内容分词，直接精确匹配，一般是用来查找keyword、数值、日期等类型。**

只能搜单个词条，不能搜被分词了的词条，比如"脱脂牛奶"，就是被分词了的词条，不能是用term精确查询

- `ids` `range` `term`

```json
# 精确term查询 query
GET /items/_search
{
  "query": {
    "term": {
      "name": {
        "value": "牛奶" # 牛奶是单个词条
      }
    }
  }
}

# 精确range范围查询 query
GET /items/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 500000,
        "lte": 1000000
      }
    }
  }
}

# 精确根据id属性查询 query
GET /items/_search
{
  "query": {
    "ids": {
      "values": ["1861099","1861100"]
    }
  }
}


```

#### 地理查询

主要用于搜索地理位置查询。

### 复合查询

- 基于逻辑运算组合的叶子查询。**布尔查询**
- 基于某种算法修改对查询时的文档相关性**算分**，从而改变文档的搜索结果排名
  - function_score
  - dis_max

#### 布尔查询

就是一个或者多个查询子句的组合，方式：

- must -> & 必须匹配每一个子查询
- should -> |  选择性匹配子查询
- must_not -> ~ 不需要算分
- filter = 必须匹配，不参与算分

**查询一个智能手机，品牌必须是华为，价格在一个价格区间中**  <- 使用布尔查询

```json
GET /items/_search
{
  "query": {
    "bool": {
      "must": [ # 需要有匹配度，参与算分
        {
          "match": {
            "name": "智能手机"
          }
        }
      ],
      "filter": [ # 效果同must，但是只有查到或者不查到两种情况，所以不参与算分。
        {
          "term": {
            "brand": "华为"
          }
        },
        {
          "range": {
            "price": {
              "gte": 90000,
              "lte": 159900
            }
          }
        }
      ]
    }
  }
}
```

### 排序和分页

默认按照相关度算分进行排序`_score`,也可以指定字段进行排序。

**搜索商品，按照销量排序，销量相同的情况下，按照价格升序排序**

```json
GET /items/_search 
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "sold": "desc"
    },
    {
      "price": "asc"
    }
  ]
}
```

#### 分页

es的搜索返回结果，默认只返回TOP10的数据，如果需要更多，则需要加入分页的参数 `from` `size`

```json
GET /items/_search 
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "sold": "desc"
    },
    {
      "price": "asc"
    }
  ],
 "from": 3, // 从第三个开始 但这里from + size 的搜索方式拥有搜索窗口上限 10000
  "size": 2 // 查出的页面大小
}
```

#### 深度分页问题

es中的数据会采用**分片存储`shard`,**把一个索引中的数据分成N份，存储到不同的节点上，查询数据的时候需要汇总各个分片的数据

##### ES集群

由一个索引的所有的shard构成的shard集群。

**深度分页问题：**当查的页数越深，性能就越差。**找总的1000名，需要从每一个分片中找出当前片的前1000名，最终将结果结合起来排序，得到总的前1000名。**

##### Search After解决方案

每一次分页的时候都需要对`shard`（片）中的数据进行排序，当查完第一页的所有数据，记录第一页的最后一个值，在查询下一页的时候从该值开始查询这一页的数据。

- 没有查询上限，支持深度分页
- 只能向后逐页查询，不能够随机翻页
- 场景：数据迁移、手机滚动的查询

### 高亮显示

将搜索的关键字进行高亮显示。默认加上`<em>`标签。

原理：es在分词的词条的时候，不仅会记录词条所在的文档的id，而且还会记录该词条在文档中所在的详细位置。

```json
# GET /items/_search
{
  "query": {
    "match": {
      "name": "脱脂牛奶"
    }
  },
  "highlight": {
    "fields": {
      "name": {
        "pre_tags": "<em>",
        "post_tags": "</em>"
      }
    }
  }
}
// 查询结果
"hits" : [
      {
        "_index" : "items",
        "_type" : "_doc",
        "_id" : "33449279171",
        "_score" : 16.198345,
        "_source" : {
          "id" : "33449279171",
          "name" : "意大利 进口牛奶 葛兰纳诺脱脂纯牛奶 成人牛奶  进口脱脂纯牛奶1Lx6盒",
          "price" : 3500,
          "image" : "https://m.360buyimg.com/mobilecms/s720x720_jfs/t1/25045/9/2656/164517/5c20699dE9b7f4c9c/1a05e9bdd2c5d59e.jpg!q70.jpg.webp",
          "category" : "牛奶",
          "brand" : "葛兰纳诺",
          "sold" : 0,
          "commentCount" : 0,
          "isAD" : false,
          "updateTime" : 1556640000000
        },
        "highlight" : { // 在这里
          "name" : [
            "意大利 进口<em>牛奶</em> 葛兰纳诺<em>脱脂</em>纯<em>牛奶</em> 成人<em>牛奶</em>  进口<em>脱脂</em>纯<em>牛奶</em>1Lx6盒"
          ]
        }
      },
```

## JavaRestClient查询

### 初识api

- 所有的`query`查询条件都使用`QueryBuilders`进行创建的。

```java
    @Test
    void testMatchAll() throws IOException {
        // 创建request对象（需要在这里指定搜索的目标index）
        SearchRequest searchRequest = new SearchRequest("items");
        // 主要方法就是source
        searchRequest.source()
                .query(QueryBuilders.matchQuery("name", "牛奶"))
                .sort("id", SortOrder.ASC);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        // 解析结果
        SearchHits hits = searchResponse.getHits();
        // 得到总条数
        TotalHits totalHits = hits.getTotalHits();
        // 命中的数据
        SearchHit[] hits1 = hits.getHits();
        for (SearchHit hit : hits1) {
            // 获取原始文档source的结果
            String json = hit.getSourceAsString();
            ItemDoc bean = JSONUtil.toBean(json, ItemDoc.class);
            System.out.println(bean);
        }
    }
```

### 布尔查询示例

```java
@Test
    void testProblem() throws IOException {
        //  搜索关键字为脱脂牛奶
        // 品牌必须是德亚
        // 价格为300
        SearchRequest searchRequest = new SearchRequest("items");
        searchRequest.source().query(
                QueryBuilders.boolQuery()
                        .must(QueryBuilders.matchQuery("name", "脱脂牛奶"))
                        .filter(QueryBuilders.termQuery("brand.keyword", " 徳亚"))
                        .filter(QueryBuilders.rangeQuery("price").lt(30000))
        );
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
        resolveResponse(searchResponse);
    }
```

### 高亮处理对象示例

```java
@Test
    void testProblem() throws IOException {
        SearchRequest searchRequest = new SearchRequest("items");
        searchRequest.source()
                .query(
                QueryBuilders.boolQuery()
                        .must(QueryBuilders.matchQuery("name", "脱脂牛奶")))
                .highlighter(new HighlightBuilder().field("name").preTags("<em>").postTags("</em>"));
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
        SearchHit[] hits = searchResponse.getHits().getHits();
        for (SearchHit hit : hits) {
            ItemDoc item = JSONUtil.toBean(hit.getSourceAsString(), ItemDoc.class);

            // 获得高亮字段map
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            
            if (CollUtils.isNotEmpty(highlightFields)) {
                HighlightField name = highlightFields.get("name");
                // Fragments可能会有多个数组元素
                String highlightName = name.getFragments()[0].string();
                // 更新当前hit的name，也就是设置高亮
                item.setName(highlightName);
            }
        }
    }
```

## 数据聚合(aggregations)

**参与聚合的字段必须是keyword等没有进行分词的字段**

- 桶（Bucket）聚合
  - 按照文档字段值进行分组
  - 按照日期阶梯分组，一周为一组，一月为一组。
- 度量（metric）聚合
- 管道聚合

### DSL

```json
GET /items/_search
{
    "query": {
        "bool": {
            "filter": [
                {"term": {"category": "手机"}},
                {"range": {"price": {"gte": 30000}}}
            ]
        }
    }
    "size": 0, // 设置返回的文档数据为0,只返回聚合不返回文档数据
    "aggs": {
        "cate_agg": { // 给聚合取名字
            "terms": { // 设置terms聚合
                "field": "category", // 依据聚合的字段名
                "size": 5  // 返回的聚合数
            },
// 先聚合再在聚合中的再嵌套聚合实现计算
			// 先分组再度量
			"aggs": {
                // 统计函数，返回min max arg sum的统计情况
                "price_stats": {
                    "stats": {
						"field": "price"
                    }
                }
                
			}
        }
    }
}
```




