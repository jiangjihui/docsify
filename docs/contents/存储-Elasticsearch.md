## 简介

Elasticsearch是一个实时的分布式搜索和分析引擎。它可以帮助你用前所未有的速度去处理大规模数据。

它可以用于全文搜索，结构化搜索以及分析，当然你也可以将这三者进行组合。

Elasticsearch使用**Lucene**作为内部引擎，但是在使用它做全文搜索时，只需要使用统一开发好的API即可，而不需要了解其背后复杂的Lucene的运行原理。

 

 

## 安装

[下载](https://www.elastic.co/downloads/elasticsearch)zip包解压运行其中bin/elasticsearch.bat即可

**备注：**推荐使用PostMan进行练习而不是Google插件sense

 

 

## 对比关系型数据库

基本概念的理解，就是要知道index, type, document, field这些名词到底啥意思，看下表。

传统关系型数据库（如 MySQL）与 Elasticsearch 对比

| **Relational DB** | **Elasticsearch** | **释义**           |
| ----------------- | ----------------- | ------------------ |
| Databases         | Indices           | 索引(名词)即数据库 |
| Tables            | Types             | 类型即表名         |
| Rows              | Documents         | 文档即每行数据     |
| Columns           | Fields            | 字段               |

