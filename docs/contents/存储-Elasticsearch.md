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



## IK分词器  
Elasticsearch的默认分词器对英文支持较好，对中文分词不太友好，会将每个汉字单独分词。所以我们可以通过安装IK分词器来实现对中文的分词。

### 安装  

进入[下载页面](https://github.com/medcl/elasticsearch-analysis-ik/tags)选择对应与当前 Elasticsearch 版本一致的版本下载之后解压到Elasticsearch程序目录下的 plugins 文件夹即可。

### 模式

IK分词器包含2种模式：

- `ik_smart`：最少切分
- `ik_max_word`：最细切分

### 词库

要扩展或者禁用某些敏感词条，只需要修改一个ik分词器目录中的config目录中的IkAnalyzer.cfg.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <comment>IK Analyzer扩展配置</comment>
    <!--用户可以在这里配置自己的扩展字典-->
    <entry key="ext_dict">ext.dic</entry>

    <!--用户可以在这里配置自己的扩展停止词字典***添加停用词词典-->
    <entry key="ext_stopwords">stopword.dic</entry>
</properties>
```

备注：字典文件中以换行来区分每个词。







