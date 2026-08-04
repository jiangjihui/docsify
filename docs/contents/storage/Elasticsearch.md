# Elasticsearch

## 概述

### 什么是 Elasticsearch

Elasticsearch 是一个实时的分布式搜索和分析引擎。它基于 Apache Lucene 构建，使用 Java 编写，通过 RESTful API 提供强大的搜索和分析能力。Elasticsearch 能够处理大规模数据，支持水平扩展到数百台服务器及 PB 级别的结构化或非结构化数据。

Elasticsearch 是 Elastic Stack（也称为 ELK Stack）的核心组件：

| 组件 | 用途 |
|------|------|
| **Elasticsearch** | 搜索与分析引擎 |
| **Kibana** | 数据可视化 |
| **Logstash** | 数据收集、转换 |
| **Beats** | 轻量级数据采集器 |

### Elasticsearch 版本演进

| 版本 | 发布年份 | 主要特性 |
|------|----------|----------|
| 0.4 | 2012 | 首个公开版本，分布式搜索框架 |
| 1.x | 2014 | 1.0 正式版，分布式搜索 |
| 2.x | 2015 | 性能优化、聚合增强 |
| 5.x | 2016 | Ingest 节点、SQL 支持、安全增强 |
| 6.x | 2017 | 跨版本复制、Index 类型限制、 painless 脚本 |
| 7.x | 2019 | 新的集群协调系统、Lyves、猛犸象吉祥物 |
| 8.x | 2022 | 安全增强、ES|QL、改进的 REST Client |

> **版本说明**：ES 8.x 有较大变化，如包名从 `org.elasticsearch.*` 改为 `co.elastic.*`。

### 特性

- **分布式架构**：水平扩展，通过添加节点增加处理能力和存储容量
- **实时分析**：近实时的文档索引和检索（通常 1 秒内）
- **RESTful API**：使用 HTTP 协议和 JSON 格式交互
- **多租户能力**：支持为不同用户或应用提供独立服务
- **插件化**：支持插件扩展功能
- **丰富的查询 DSL**：强大的查询语言
- **聚合分析**：支持度量、桶、管道聚合
- **高可用性**：通过分片和副本机制保证可靠性

### 应用场景

- **全文搜索**：电商搜索、日志搜索、文档搜索
- **日志分析**：ELK 日志分析系统
- **应用性能监控**：APM 场景
- **安全分析**：SIEM、安全日志分析
- **业务分析**：用户行为分析、指标分析

---

## 核心概念

### 架构术语对照

| Elasticsearch | 关系型数据库 | 释义 |
|--------------|-------------|------|
| Cluster | - | 集群 |
| Node | - | 节点 |
| Index | Database | 索引（逻辑命名空间） |
| Type | Table | 类型（ES 7.x 后废弃） |
| Document | Row | 文档 |
| Field | Column | 字段 |
| Mapping | Schema | 映射（结构定义） |
| Shard | - | 分片 |
| Replica | - | 副本 |

### 集群与节点

**集群（Cluster）** 是由多个节点组成的分布式系统，提供联合索引和搜索能力。

**节点（Node）** 是 Elasticsearch 的运行实例，分为以下角色：

| 节点类型 | 说明 | 配置 |
|----------|------|------|
| **Master Node** | 管理集群元数据、节点加入/离开、索引创建删除 | `node.master: true` |
| **Data Node** | 存储数据、处理查询和索引操作 | `node.data: true` |
| **Ingest Node** | 数据预处理（管道处理） | `node.ingest: true` |
| **Coordinate Node** | 协调节点，分发请求和聚合结果 | `node.master: false, node.data: false` |

```yaml
# elasticsearch.yml 配置示例
# Master 节点
node.master: true
node.data: false
node.ingest: false

# Data 节点
node.master: false
node.data: true
node.ingest: false

# Ingest 节点
node.master: false
node.data: false
node.ingest: true
```

### 索引（Index）

索引是存储文档的逻辑容器，类似关系型数据库的数据库概念。

```json
// 创建索引
PUT /products
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "price": { "type": "double" },
      "category": { "type": "keyword" }
    }
  }
}
```

### 文档（Document）

文档是 Elasticsearch 中可被索引的基本数据单元，以 JSON 格式表示。

```json
// 文档结构示例
{
  "_id": "1",
  "_source": {
    "name": "iPhone 15",
    "price": 6999.00,
    "category": "phone",
    "tags": ["apple", "smartphone"],
    "create_time": "2024-01-01T00:00:00Z"
  }
}
```

### 映射（Mapping）

映射定义了索引中文档的结构，类似数据库的表结构。

```json
// 映射示例
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "price": { "type": "float" },
      "stock": { "type": "integer" },
      "tags": { "type": "keyword" },
      "create_time": {
        "type": "date",
        "format": "yyyy-MM-dd||epoch_millis"
      }
    }
  }
}
```

**字段类型**：

| 类型 | 说明 |
|------|------|
| text | 全文搜索类型，会分词 |
| keyword | 关键词类型，不分词，用于精确匹配 |
| integer/long | 整数类型 |
| float/double | 浮点数类型 |
| boolean | 布尔类型 |
| date | 日期类型 |
| object | 嵌套对象 |
| nested | 嵌套对象数组 |
| ip | IP 地址 |

### 分片与副本

**分片（Shard）** 是索引数据的水平分割，每个分片是一个独立的 Lucene 索引。

**副本（Replica）** 是分片的拷贝，用于提高可用性和查询吞吐量。

```
索引 (5 分片, 1 副本)
├── 主分片 0
│   └── 分片 0 副本
├── 主分片 1
│   └── 分片 1 副本
├── 主分片 2
│   └── 分片 2 副本
├── 主分片 3
│   └── 分片 3 副本
└── 主分片 4
    └── 分片 4 副本
```

```json
// 索引设置
{
  "settings": {
    "number_of_shards": 5,    // 主分片数量，创建后不可更改
    "number_of_replicas": 1  // 每个主分片的副本数量
  }
}
```

> **注意**：`number_of_shards` 在索引创建后无法修改，修改 `number_of_replicas` 可以动态调整。

---

## Elasticsearch 工作原理

### 数据索引过程

```
文档 → 分析(Analyzer) → 分词(Token) → 写入分片 → 持久化
```

1. **文档分析**：将文档内容分解为词项（Token），进行分词、过滤
2. **写入分片**：根据文档 `_id` 哈希值路由到对应主分片
3. **数据持久化**：写入 Lucene 索引文件

### 数据搜索过程

```
查询 → 解析 → 分片查询 → 结果聚合 → 返回结果
```

1. **查询解析**：解析查询条件
2. **分片查询**：向所有相关分片发送查询请求
3. **结果聚合**：收集各分片结果，排序后返回

### 倒排索引

Elasticsearch 使用倒排索引实现快速搜索：

| 词项 | 文档ID列表 |
|------|------------|
| elasticsearch | [1, 3] |
| search | [1, 2, 3] |
| engine | [1, 2] |

---

## REST API

### 索引操作

```bash
# 创建索引
PUT /my-index

# 创建索引（带设置和映射）
PUT /my-index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "content": { "type": "text" }
    }
  }
}

# 查看索引信息
GET /my-index

# 删除索引
DELETE /my-index

# 关闭/打开索引
POST /my-index/_close
POST /my-index/_open
```

### 文档操作

```bash
# 添加文档（指定 ID）
PUT /my-index/_doc/1
{
  "title": "Elasticsearch 入门",
  "content": "Elasticsearch 是一个分布式搜索引擎"
}

# 添加文档（自动生成 ID）
POST /my-index/_doc
{
  "title": "Elasticsearch 入门",
  "content": "Elasticsearch 是一个分布式搜索引擎"
}

# 根据 ID 查询
GET /my-index/_doc/1

# 查询文档是否存在
HEAD /my-index/_doc/1

# 更新文档
POST /my-index/_update/1
{
  "doc": {
    "title": "Elasticsearch 入门（更新版）"
  }
}

# 删除文档
DELETE /my-index/_doc/1
```

### 批量操作

```bash
# 批量索引
POST /my-index/_bulk
{"index": {"_index": "my-index", "_id": "1"}}
{"title": "文档1", "content": "内容1"}
{"index": {"_index": "my-index", "_id": "2"}}
{"title": "文档2", "content": "内容2"}

# 混合操作
POST /my-index/_bulk
{"index": {"_index": "my-index", "_id": "1"}}
{"title": "文档1"}
{"delete": {"_index": "my-index", "_id": "2"}}
{"update": {"_index": "my-index", "_id": "3"}}
{"doc": {"title": "文档3更新"}}
```

### 搜索 API

```bash
# 查询所有
GET /my-index/_search

# 简单条件查询
GET /my-index/_search
{
  "query": {
    "match": {
      "title": "Elasticsearch"
    }
  }
}

# 分页查询
GET /my-index/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "match_all": {}
  }
}

# 排序
GET /my-index/_search
{
  "sort": [
    { "create_time": { "order": "desc" } },
    "_score"
  ]
}

# 返回指定字段
GET /my-index/_search
{
  "_source": ["title", "price"],
  "query": {
    "match": { "title": "Elasticsearch" }
  }
}
```

---

## 查询 DSL

### 简单查询

```json
// match 查询（全文搜索）
{
  "query": {
    "match": {
      "title": "Elasticsearch 教程"
    }
  }
}

// term 查询（精确匹配）
{
  "query": {
    "term": {
      "category": "book"
    }
  }
}

// terms 查询（多值精确匹配）
{
  "query": {
    "terms": {
      "tags": ["python", "programming"]
    }
  }
}

// range 查询（范围查询）
{
  "query": {
    "range": {
      "price": {
        "gte": 10,
        "lte": 100
      }
    }
  }
}
```

### 布尔查询

```json
// bool 查询
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "Elasticsearch" } }
      ],
      "should": [
        { "match": { "content": "入门" } }
      ],
      "must_not": [
        { "term": { "status": "deleted" } }
      ],
      "filter": [
        { "term": { "category": "tech" } }
      ]
    }
  }
}
```

**布尔查询子句**：

| 子句 | 说明 |
|------|------|
| must | 必须匹配，贡献评分 |
| should | 应该匹配，贡献评分 |
| must_not | 必须不匹配，不贡献评分 |
| filter | 必须匹配，不贡献评分（更快） |

### 复合查询

```json
// 短语匹配
{
  "query": {
    "match_phrase": {
      "title": "Elasticsearch 教程"
    }
  }
}

// 前缀查询
{
  "query": {
    "prefix": {
      "title": " Elas "
    }
  }
}

// 通配符查询
{
  "query": {
    "wildcard": {
      "title": "E*a"
    }
  }
}

// 模糊查询
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "Elasticserch",
        "fuzziness": 2
      }
    }
  }
}
```

### 高亮显示

```json
{
  "query": {
    "match": { "title": "Elasticsearch" }
  },
  "highlight": {
    "fields": {
      "title": {},
      "content": {}
    },
    "pre_tags": ["<em>"],
    "post_tags": ["</em>"]
  }
}
```

---

## 聚合分析

### 度量聚合

```json
// avg 平均值
{
  "size": 0,
  "aggs": {
    "avg_price": {
      "avg": { "field": "price" }
    }
  }
}

// sum 求和
{
  "size": 0,
  "aggs": {
    "total_sales": {
      "sum": { "field": "sales" }
    }
  }
}

// min/max 最小/最大值
{
  "size": 0,
  "aggs": {
    "min_price": { "min": { "field": "price" } },
    "max_price": { "max": { "field": "price" } }
  }
}

// stats 统计
{
  "size": 0,
  "aggs": {
    "price_stats": {
      "stats": { "field": "price" }
    }
  }
}

// cardinality 去重计数
{
  "size": 0,
  "aggs": {
    "unique_users": {
      "cardinality": { "field": "user_id" }
    }
  }
}
```

### 桶聚合

```json
// terms 按字段值分组
{
  "size": 0,
  "aggs": {
    "categories": {
      "terms": {
        "field": "category",
        "size": 10
      }
    }
  }
}

// range 范围分组
{
  "size": 0,
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 500 },
          { "from": 500 }
        ]
      }
    }
  }
}

// date_histogram 日期直方图
{
  "size": 0,
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "month"
      }
    }
  }
}
```

### 管道聚合

```json
// 桶内度量
{
  "size": 0,
  "aggs": {
    "categories": {
      "terms": { "field": "category", "size": 10 },
      "aggs": {
        "avg_price": {
          "avg": { "field": "price" }
        }
      }
    }
  }
}

// 桶排序 + 桶脚本
{
  "size": 0,
  "aggs": {
    "categories": {
      "terms": {
        "field": "category",
        "size": 5,
        "order": { "avg_price": "desc" }
      },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    }
  }
}
```

---

## IK 分词器

Elasticsearch 默认分词器对中文支持不佳，需要安装 IK 分词器。

### 安装

```bash
# ES 7.x 及以下
cd $ES_HOME/plugins
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.17.0/elasticsearch-analysis-ik-7.17.0.zip
unzip elasticsearch-analysis-ik-*.zip
rm elasticsearch-analysis-ik-*.zip

# ES 8.x（使用 plugin 命令）
cd $ES_HOME
bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.11.0/elasticsearch-analysis-ik-8.11.0.zip
```

> **注意**：IK 版本需要与 Elasticsearch 版本匹配。

### 使用方式

```json
// ik_smart 智能分词（最少切分）
{
  "analyzer": "ik_smart",
  "text": "中华人民共和国"
}
// 结果：["中华人民共和国"]

// ik_max_word 最细切分
{
  "analyzer": "ik_max_word",
  "text": "中华人民共和国"
}
// 结果：["中华人民共和国", "中华人民", "中华", "华人", "人民共和国", "人民", "人", "民", "共和国", "共和", "和"]
```

### 配置自定义词库

编辑 `plugins/ik/config/IKAnalyzer.cfg.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <comment>IK Analyzer 扩展配置</comment>
    <!-- 扩展字典 -->
    <entry key="ext_dict">ext.dic</entry>
    <!-- 扩展停止词字典 -->
    <entry key="ext_stopwords">stopword.dic</entry>
</properties>
```

创建词库文件（每行一个词）：

```text
# ext.dic 示例
Elasticsearch
分布式搜索引擎
大数据

# stopword.dic 示例
的
了
和
```

### 定义 IK 分词器索引

```json
PUT /my-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_ik_analyzer": {
          "type": "custom",
          "tokenizer": "ik_max_word",
          "filter": ["my_synonym_filter"]
        }
      },
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms_path": "analysis/synonym.txt"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

---

## Java 客户端

### REST Client（推荐 ES 8.x）

```java
// pom.xml 依赖
<dependency>
    <groupId>co.elastic.clients</groupId>
    <artifactId>elasticsearch-java</artifactId>
    <version>8.11.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.2</version>
</dependency>
```

```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.*;
import co.elastic.clients.elasticsearch.core.search.Hit;
import co.elastic.clients.json.jackson.JacksonJsonpMapper;
import co.elastic.clients.transport.ElasticsearchTransport;
import co.elastic.clients.transport.rest_client.RestClientTransport;
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;

public class ElasticsearchDemo {

    public static void main(String[] args) throws Exception {
        // 创建 Low Level REST Client
        RestClient restClient = RestClient.builder(
                new HttpHost("localhost", 9200, "http")
        ).build();

        // 创建传输层
        ElasticsearchTransport transport = new RestClientTransport(
                restClient, new JacksonJsonpMapper()
        );

        // 创建 ES 客户端
        ElasticsearchClient client = new ElasticsearchClient(transport);

        // 索引文档
        String indexName = "products";
        Product product = new Product("1", "iPhone 15", 6999.0);
        IndexResponse response = client.index(i -> i
                .index(indexName)
                .id(product.getId())
                .document(product)
        );
        System.out.println("Indexed: " + response.result());

        // 查询文档
        GetResponse<Product> getResponse = client.get(g -> g
                        .index(indexName)
                        .id("1"),
                Product.class
        );
        System.out.println("Found: " + getResponse.source());

        // 搜索文档
        SearchResponse<Product> searchResponse = client.search(s -> s
                        .index(indexName)
                        .query(q -> q
                                .match(m -> m
                                        .field("name")
                                        .query("iPhone")
                                )
                        ),
                Product.class
        );

        for (Hit<Product> hit : searchResponse.hits().hits()) {
            System.out.println("Found: " + hit.source());
        }

        // 关闭客户端
        transport.close();
        restClient.close();
    }
}

class Product {
    private String id;
    private String name;
    private Double price;

    // 构造方法、getter、setter
    public Product(String id, String name, Double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public Double getPrice() { return price; }
}
```

### High Level Client（ES 7.x 推荐）

```java
// pom.xml 依赖（ES 7.x）
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.17.0</version>
</dependency>
```

```java
// 创建客户端
RestHighLevelClient client = new RestHighLevelClient(
        RestClient.builder(
                new HttpHost("localhost", 9200, "http")
        ));

// 索引文档
IndexRequest request = new IndexRequest("products");
request.id("1");
request.source(Map.of(
        "name", "iPhone 15",
        "price", 6999.0
), XContentType.JSON);
IndexResponse response = client.index(request, RequestOptions.DEFAULT);

// 搜索文档
SearchRequest searchRequest = new SearchRequest("products");
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.matchQuery("name", "iPhone"));
searchRequest.source(sourceBuilder);
SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
```

---

## Spring Boot 集成

### Spring Data Elasticsearch

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
    <version>3.2.0</version>
</dependency>
```

```yaml
# application.yml
spring:
  elasticsearch:
    uris: http://localhost:9200
    username: elastic
    password: password
```

```java
// 实体类
@Document(indexName = "products")
public class Product {

    @Id
    private String id;

    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String name;

    @Field(type = FieldType.Double)
    private Double price;

    @Field(type = FieldType.Keyword)
    private String category;

    // getter、setter
}
```

```java
// Repository
public interface ProductRepository extends ElasticsearchRepository<Product, String> {

    List<Product> findByName(String name);

    @Query("{\"match\": {\"name\": {\"query\": \"?0\"}}}")
    List<Product> searchByName(String name);

    List<Product> findByPriceBetween(Double min, Double max);
}
```

```java
// Service
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public Product save(Product product) {
        return productRepository.save(product);
    }

    public List<Product> search(String keyword) {
        return productRepository.findByName(keyword);
    }

    public void delete(String id) {
        productRepository.deleteById(id);
    }
}
```

### ElasticsearchRestTemplate

```java
@Configuration
public class ElasticsearchConfig {

    @Bean
    public ElasticsearchRestTemplate elasticsearchRestTemplate() {
        return new ElasticsearchRestTemplate(client());
    }

    @Bean
    public RestClient client() {
        return RestClient.builder(
                new HttpHost("localhost", 9200, "http")
        ).build();
    }
}
```

```java
@Autowired
private ElasticsearchRestTemplate template;

// 创建索引
template.createIndex(Product.class);

// 删除索引
template.deleteIndex(Product.class);

// 查询
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
        .withQuery(QueryBuilders.matchQuery("name", "iPhone"))
        .build();
SearchHits<Product> hits = template.search(searchQuery, Product.class);
```

---

## 性能优化

### 分片策略

```json
// 分片分配策略
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "routing": {
      "allocation": {
        "total_shards_per_node": 2
      }
    }
  }
}
```

**分片数量建议**：

| 数据量 | 分片数建议 |
|--------|-----------|
| < 1GB | 1 个分片 |
| 1GB - 20GB | 1-3 个分片 |
| 20GB - 100GB | 3-5 个分片 |
| > 100GB | 按 20-50GB/分片 |

> **公式**：分片数 ≈ 数据总量 / 目标分片大小

### 路由优化

```json
// 自定义路由
{
  "routing": {
    "field": "user_id"
  }
}

// 查询时指定路由（减少搜索分片数）
GET /my-index/_search?routing=user_123
{
  "query": {
    "term": { "user_id": "user_123" }
  }
}
```

### 缓存策略

```json
// 开启查询缓存
{
  "settings": {
    "indices.queries.cache.size": "20%"
  }
}

// 查看缓存状态
GET /_stats/query_cache
```

### 写入优化

```json
{
  "settings": {
    "refresh_interval": "30s",  // 刷新间隔（默认 1s）
    "translog": {
      "sync_interval": "30s",  // 事务日志同步间隔
      "durability": "async"    // 异步刷新（提升写入性能）
    }
  }
}
```

### 合并优化

```json
{
  "settings": {
    "merge": {
      "scheduler": {
        "max_thread_count": 1
      }
    }
  }
}
```

---

## 集群部署

### 生产环境配置

```yaml
# elasticsearch.yml
cluster.name: my-cluster
node.name: node-1

# 绑定地址
network.host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300

# 节点发现
discovery.seed_hosts:
  - 192.168.1.1:9300
  - 192.168.1.2:9300
  - 192.168.1.3:9300

# 集群协调
cluster.initial_master_nodes:
  - node-1
  - node-2
  - node-3

# 内存锁定
bootstrap.memory_lock: true

# JVM 堆内存
-Xms4g
-Xmx4g

# 线程池
thread_pool.write.size: 30
thread_pool.search.size: 30
```

### 集群规划建议

| 节点数 | 数据量 | 建议配置 |
|--------|--------|---------|
| 3 节点 | < 10TB | 8核 16GB |
| 5 节点 | 10-50TB | 16核 32GB |
| 10+ 节点 | > 50TB | 32核 64GB+ |

### 监控 API

```bash
# 集群健康
GET /_cluster/health

# 集群状态
GET /_cluster/state

# 节点统计
GET /_nodes/stats

# 索引统计
GET /_stats

# 任务管理
GET /_tasks
```

---

## 常见问题

### 1. 集群健康为 yellow 或 red

- **yellow**：副本分片未分配，检查节点状态
- **red**：主分片未分配，检查索引和节点

```bash
# 查看未分配的分片
GET /_cluster/allocation/explain

# 重新分配
POST /_cluster/reroute
{
  "commands": [
    {
      "move": {
        "index": "my-index",
        "shard": 0,
        "from_node": "node-1",
        "to_node": "node-2"
      }
    }
  ]
}
```

### 2. 文档无法索引

- 检查索引是否存在
- 检查映射是否正确
- 检查分片是否健康

```bash
# 查看索引状态
GET /my-index/_status
```

### 3. 查询性能差

- 优化查询：使用 filter 而非 must
- 减少返回字段：使用 `_source` 限制
- 合理使用分页：避免深度分页
- 开启慢查询日志

```json
{
  "settings": {
    "index.search.slowlog.threshold.query.warn": "10s"
  }
}
```

### 4. 内存溢出（OOM）

- 增加 JVM 堆内存
- 优化查询语句
- 减少分片数量

```bash
# 查看节点内存
GET /_nodes/stats/jvm
```

### 5. IK 分词器不生效

- 确认 IK 版本与 ES 版本匹配
- 确认 IK 插件已正确安装
- 检查分词器配置

```bash
# 查看已安装插件
GET /_cat/plugins

# 测试分词器
GET /my-index/_analyze
{
  "analyzer": "ik_max_word",
  "text": "中华人民共和国"
}
```

### 6. 深度分页问题

避免使用 `from + size` 进行深度分页，推荐使用：

- **Search After**：基于上一页最后一条排序值
- **Scroll API**：用于导出大量数据
- **PIT（Point In Time）**：ES 7.10+ 推荐

```json
// Search After 示例
{
  "size": 10,
  "query": { "match_all": {} },
  "sort": [
    { "create_time": "desc" },
    { "_id": "asc" }
  ]
}
```

### 7. 集群脑裂问题

配置最小主节点数避免脑裂：

```yaml
# elasticsearch.yml
discovery.zen.minimum_master_nodes: 2
```

---

## 参考资料

- [Elasticsearch 官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Elasticsearch Java Client](https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/index.html)
- [Spring Data Elasticsearch](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/)
- [IK Analysis for Elasticsearch](https://github.com/medcl/elasticsearch-analysis-ik)
