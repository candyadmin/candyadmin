---
title: es常用命令
date: 2025-02-27 15:00:37
categories: elasticsearch
tag: 学习笔记
---

以下是 Elasticsearch (ES) 常用命令的总结和笔记：

### 1. 查看索引相关信息

- **查看指定索引的映射信息**
  ```bash
  GET kibana_sample_data_ecommerce
  ```

- **查看指定索引的文档总数**
  ```bash
  GET kibana_sample_data_ecommerce/_count
  ```

- **查看前10条文档**
  ```bash
  POST kibana_sample_data_ecommerce/_search
  {
  }
  ```

### 2. 查看索引状态与信息

- **查看索引信息**
  ```bash
  GET /_cat/indices/kibana*?v&s=index
  ```

- **查看状态为绿的索引**
  ```bash
  GET /_cat/indices?v&health=green
  ```

- **按文档个数排序查看索引**
  ```bash
  GET /_cat/indices?v&s=docs.count:desc
  ```

- **查看具体的索引字段（健康、主分片、复制分片、文档数等）**
  ```bash
  GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt
  ```

- **查看每个索引使用的内存量**
  ```bash
  GET /_cat/indices?v&h=i,tm&s=tm:desc
  ```

### 3. 查看节点信息

- **查看所有节点信息**
  ```bash
  GET _cat/nodes?v
  ```

- **查看特定节点信息**
  ```bash
  GET /_nodes/es7_01,es7_02
  ```

- **查看节点信息并显示特定字段**
  ```bash
  GET /_cat/nodes?v&h=id,ip,port,v,m
  ```

### 4. 查看集群健康状况

- **查看集群健康状况**
  ```bash
  GET _cluster/health
  ```

- **查看集群健康状况，并按分片级别显示**
  ```bash
  GET _cluster/health?level=shards
  ```

- **查看特定索引的集群健康状况**
  ```bash
  GET /_cluster/health/kibana_sample_data_ecommerce,kibana_sample_data_flights
  ```

- **查看特定索引的集群健康状况，并按分片级别显示**
  ```bash
  GET /_cluster/health/kibana_sample_data_flights?level=shards
  ```

### 5. 查看集群状态

- **查看集群状态**
  ```bash
  GET /_cluster/state
  ```

### 6. 查看集群设置

- **查看集群设置**
  ```bash
  GET /_cluster/settings
  ```

- **查看集群设置，包括默认设置**
  ```bash
  GET /_cluster/settings?include_defaults=true
  ```

### 7. 查看分片信息

- **查看集群的所有分片信息**
  ```bash
  GET /_cat/shards
  ```

- **查看特定的分片信息（包括状态、主分片/副本分片等）**
  ```bash
  GET /_cat/shards?h=index,shard,prirep,state,unassigned.reason
  ```



### 1. 创建文档

- **自动生成 `_id` 创建文档**
  ```bash
  POST users/_doc
  {
    "user" : "Mike",
    "post_date" : "2019-04-15T14:12:12",
    "message" : "trying out Kibana"
  }
  ```

- **指定 ID 创建文档（如果 ID 已存在则报错）**
  ```bash
  PUT users/_doc/1?op_type=create
  {
    "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
  }
  ```

- **指定 ID 创建文档（如果 ID 已存在则报错）**
  ```bash
  PUT users/_create/1
  {
    "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
  }
  ```

### 2. 根据 ID 获取文档

- **获取指定 ID 的文档**
  ```bash
  GET users/_doc/1
  ```

### 3. 更新文档

- **更新指定 ID 的文档（先删除再写入）**
  ```bash
  PUT users/_doc/1
  {
    "user" : "Mike"
  }
  ```

- **在原文档上增加字段**
  ```bash
  POST users/_update/1/
  {
    "doc": {
      "post_date" : "2019-05-15T14:12:12",
      "message" : "trying out Elasticsearch"
    }
  }
  ```

### 4. 删除文档

- **根据 ID 删除文档**
  ```bash
  DELETE users/_doc/1
  ```

### 5. 批量操作 (Bulk API)

- **执行批量操作**  
  第一次执行批量操作：
  ```bash
  POST _bulk
  { "index" : { "_index" : "test", "_id" : "1" } }
  { "field1" : "value1" }
  { "delete" : { "_index" : "test", "_id" : "2" } }
  { "create" : { "_index" : "test2", "_id" : "3" } }
  { "field1" : "value3" }
  { "update" : {"_id" : "1", "_index" : "test"} }
  { "doc" : {"field2" : "value2"} }
  ```

  第二次执行批量操作：
  ```bash
  POST _bulk
  { "index" : { "_index" : "test", "_id" : "1" } }
  { "field1" : "value1" }
  { "delete" : { "_index" : "test", "_id" : "2" } }
  { "create" : { "_index" : "test2", "_id" : "3" } }
  { "field1" : "value3" }
  { "update" : {"_id" : "1", "_index" : "test"} }
  { "doc" : {"field2" : "value2"} }
  ```

### 6. 多文档获取 (MGET)

- **获取多个文档**
  ```bash
  GET /_mget
  {
    "docs" : [
      { "_index" : "test", "_id" : "1" },
      { "_index" : "test", "_id" : "2" }
    ]
  }
  ```

- **URI中指定索引**
  ```bash
  GET /test/_mget
  {
    "docs" : [
      { "_id" : "1" },
      { "_id" : "2" }
    ]
  }
  ```

- **使用 `_source` 参数指定返回的字段**
  ```bash
  GET /_mget
  {
    "docs" : [
      { "_index" : "test", "_id" : "1", "_source" : false },
      { "_index" : "test", "_id" : "2", "_source" : ["field3", "field4"] },
      { "_index" : "test", "_id" : "3", "_source" : { "include": ["user"], "exclude": ["user.location"] } }
    ]
  }
  ```

### 7. 多查询操作 (MSEARCH)

- **执行多个查询**
  ```bash
  POST kibana_sample_data_ecommerce/_msearch
  {}
  {"query" : {"match_all" : {}},"size":1}
  {"index" : "kibana_sample_data_flights"}
  {"query" : {"match_all" : {}},"size":2}
  ```

### 8. 清除测试数据

- **删除索引（清除数据）**
  ```bash
  DELETE users
  DELETE test
  DELETE test2
  ```

[3.4-倒排索引入门]

Elasticsearch 的 `_analyze` API，它用于分析文本，查看在特定分析器下，文本是如何被分词和处理的。下面是每个请求的说明：

### 1. **分析 "Mastering Elasticsearch"**
```bash
POST _analyze
{
  "analyzer": "standard",
  "text": "Mastering Elasticsearch"
}
```
- **分析器**：使用默认的 `standard` 分析器。
- **文本**：`"Mastering Elasticsearch"`。
- **预期效果**：`standard` 分析器会将这段文本分割成单词，通常会根据空格和标点符号进行分词。

### 2. **分析 "Elasticsearch Server"**
```bash
POST _analyze
{
  "analyzer": "standard",
  "text": "Elasticsearch Server"
}
```
- **分析器**：同样使用 `standard` 分析器。
- **文本**：`"Elasticsearch Server"`。
- **预期效果**：分析器会分割文本为单词，可能会得到类似于 `["elasticsearch", "server"]` 的分词结果。

### 3. **分析 "Elasticsearch Essentials"**
```bash
POST _analyze
{
  "analyzer": "standard",
  "text": "Elasticsearch Essentials"
}
```
- **分析器**：继续使用 `standard` 分析器。
- **文本**：`"Elasticsearch Essentials"`。
- **预期效果**：分析器会将文本分割成 `["elasticsearch", "essentials"]`。

### 结果
每个请求的结果将显示由 `standard` 分析器生成的分词结果。通常，`standard` 分析器会去除大部分标点符号并将文本转换为小写。分词的具体结果如下：

- `"Mastering Elasticsearch"` -> `["mastering", "elasticsearch"]`
- `"Elasticsearch Server"` -> `["elasticsearch", "server"]`
- `"Elasticsearch Essentials"` -> `["elasticsearch", "essentials"]`


以下是使用不同的分析器（Analyzer）对文本进行分词分析的示例：

### 1. **Standard Analyzer**
标准分析器会根据空格、标点符号等进行分词，并将所有文本转换为小写。
```bash
GET _analyze
{
  "analyzer": "standard",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

{
  "tokens" : [
    {
      "token" : "2",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<NUM>",
      "position" : 0
    },
    {
      "token" : "running",
      "start_offset" : 2,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "quick",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "brown",
      "start_offset" : 16,
      "end_offset" : 21,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "foxes",
      "start_offset" : 22,
      "end_offset" : 27,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "leap",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "over",
      "start_offset" : 33,
      "end_offset" : 37,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "lazy",
      "start_offset" : 38,
      "end_offset" : 42,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "dogs",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "<ALPHANUM>",
      "position" : 8
    },
    {
      "token" : "in",
      "start_offset" : 48,
      "end_offset" : 50,
      "type" : "<ALPHANUM>",
      "position" : 9
    },
    {
      "token" : "the",
      "start_offset" : 51,
      "end_offset" : 54,
      "type" : "<ALPHANUM>",
      "position" : 10
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "<ALPHANUM>",
      "position" : 11
    },
    {
      "token" : "evening",
      "start_offset" : 62,
      "end_offset" : 69,
      "type" : "<ALPHANUM>",
      "position" : 12
    }
  ]
}

```
- **分词结果**：`["2", "running", "quick", "brown", "foxes", "leap", "over", "lazy", "dogs", "in", "the", "summer", "evening"]`


### 2. **Simple Analyzer**
简单分析器按非字母字符切分，符号会被过滤，文本会转换为小写。
```bash
GET _analyze
{
  "analyzer": "simple",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```
- **分词结果**：`["2", "running", "quick", "brown", "foxes", "leap", "over", "lazy", "dogs", "in", "the", "summer", "evening"]`

### 3. **Stop Analyzer**
停用词分析器会将文本转换为小写，并过滤掉常见的停用词（如 "the", "a", "is"）。
```bash
GET _analyze
{
  "analyzer": "stop",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```
- **分词结果**：`["2", "running", "quick", "brown", "foxes", "leap", "lazy", "dogs", "summer", "evening"]`

### 4. **Whitespace Analyzer**
空格分析器会按空格切分文本，并且不会转换为小写。
```bash
GET _analyze
{
  "analyzer": "whitespace",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```
- **分词结果**：`["2", "running", "Quick", "brown-foxes", "leap", "over", "lazy", "dogs", "in", "the", "summer", "evening."]`

### 5. **Keyword Analyzer**
关键字分析器不进行分词，直接返回原始文本。
```bash
GET _analyze
{
  "analyzer": "keyword",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```
- **分词结果**：`["2 running Quick brown-foxes leap over lazy dogs in the summer evening."]`

### 6. **Pattern Analyzer**
模式分析器使用正则表达式进行分词，默认使用 `\W+`（非字母字符）作为分隔符。
```bash
GET _analyze
{
  "analyzer": "pattern",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```
- **分词结果**：`["2", "running", "Quick", "brown", "foxes", "leap", "over", "lazy", "dogs", "in", "the", "summer", "evening"]`

### 7. **English Analyzer**
英语分析器针对英语语言进行分词，去除停用词等。
```bash
GET _analyze
{
  "analyzer": "english",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```
- **分词结果**：`["2", "running", "quick", "brown", "foxes", "leap", "lazy", "dogs", "summer", "evening"]`

### 8. **ICU Analyzer for Chinese Text**
使用 `icu_analyzer` 来处理中文文本，适用于多语言环境中的分词。
```bash
POST _analyze
{
  "analyzer": "icu_analyzer",
  "text": "他说的确实在理"
}
```
- **分词结果**：`["他说", "的", "确实", "在理"]`

### 9. **Standard Analyzer for Chinese Text**
使用标准分析器处理中文文本，通常处理中文时可能不会像中文专用分析器那样精确。
```bash
POST _analyze
{
  "analyzer": "standard",
  "text": "他说的确实在理"
}
```
- **分词结果**：`["他说", "的", "确实", "在理"]`

### 10. **ICU Analyzer for Another Chinese Sentence**
处理另一个中文句子 `"这个苹果不大好吃"`。
```bash
POST _analyze
{
  "analyzer": "icu_analyzer",
  "text": "这个苹果不大好吃"
}
```
- **分词结果**：`["这个", "苹果", "不大", "好吃"]`

### 总结
每种分析器具有不同的分词行为：
- **Standard**：根据空格和标点符号分词并小写化。
- **Simple**：过滤非字母字符并小写化。
- **Stop**：去除停用词并小写化。
- **Whitespace**：仅按空格分词，不进行大小写转换。
- **Keyword**：不进行分词，返回整个文本。
- **Pattern**：使用正则表达式分词，默认 `\W+` 分割。
- **English**：为英语文本设计的分析器，去除停用词。
- **ICU Analyzer**：适合多语言的分词，特别适用于中文等非拉丁字母语言。

这些分析器可以根据不同的语言和需求来优化索引和搜索。


以下是您提供的一些 Elasticsearch 查询示例，涵盖了基本查询、带有分析的查询、布尔查询、范围查询、通配符查询、模糊查询等操作。下面是每种查询的详细解释和用法：

### 1. **基本查询**
查询 `movies` 索引中标题为 `2012` 的电影，并根据 `year` 字段降序排序，返回前 10 条结果：
```bash
GET /movies/_search?q=2012&df=title&sort=year:desc&from=0&size=10&timeout=1s
```
- **`q=2012`**：查询内容为 `2012`。
- **`df=title`**：指定查询字段为 `title`。
- **`sort=year:desc`**：按 `year` 字段降序排序。
- **`from=0&size=10`**：分页，返回前 10 条数据。
- **`timeout=1s`**：查询超时限制为 1 秒。

### 2. **带 Profile 的查询**
启用 `profile` 可以帮助分析查询性能，查看查询的每个阶段的时间。
```bash
GET /movies/_search?q=2012&df=title
{
	"profile":"true"
}
```
- **`"profile": "true"`**：启用查询性能分析。

### 3. **泛查询，对所有字段进行查询**
使用 `_all` 字段进行泛查询，查询所有字段。
```bash
GET /movies/_search?q=2012
{
	"profile":"true"
}
```
- **`q=2012`**：查询所有字段中包含 `2012` 的数据。

### 4. **指定字段查询**
查询 `title` 字段中包含 `2012` 的记录，并按照 `year` 字段降序排序：
```bash
GET /movies/_search?q=title:2012&sort=year:desc&from=0&size=10&timeout=1s
{
	"profile":"true"
}
```
- **`q=title:2012`**：指定查询字段为 `title`，查询内容为 `2012`。

### 5. **查找美丽心灵**
查询 `title` 字段包含 `"Beautiful Mind"` 的记录：
```bash
GET /movies/_search?q=title:Beautiful Mind
{
	"profile":"true"
}
```

### 6. **泛查询**
查询 `title` 字段包含 `2012` 的记录：
```bash
GET /movies/_search?q=title:2012
{
	"profile":"true"
}
```

### 7. **Phrase 查询（短语查询）**
查询 `title` 字段精确匹配 `"Beautiful Mind"`：
```bash
GET /movies/_search?q=title:"Beautiful Mind"
{
	"profile":"true"
}
```
- 使用引号表示精确短语匹配，要求字段内容必须完全匹配短语 `"Beautiful Mind"`。

### 8. **分组查询（使用括号）**
查询 `title` 字段中包含 `Beautiful Mind` 的记录：
```bash
GET /movies/_search?q=title:(Beautiful Mind)
{
	"profile":"true"
}
```
- 使用括号可以进行分组查询，类似逻辑上的括号优先级。

### 9. **布尔操作符查询**
- **AND 查询：**
  查询 `title` 字段同时包含 `Beautiful` 和 `Mind` 的记录：
  ```bash
  GET /movies/_search?q=title:(Beautiful AND Mind)
  {
  	"profile":"true"
  }
  ```
- **NOT 查询：**
  查询 `title` 字段包含 `Beautiful` 但不包含 `Mind` 的记录：
  ```bash
  GET /movies/_search?q=title:(Beautiful NOT Mind)
  {
  	"profile":"true"
  }
  ```
- **OR 查询：**
  查询 `title` 字段包含 `Beautiful` 或 `Mind` 的记录：
  ```bash
  GET /movies/_search?q=title:(Beautiful %2BMind)
  {
  	"profile":"true"
  }
  ```

### 10. **范围查询**
查询 `title` 字段包含 `beautiful` 且 `year` 在 2002 到 2018 年之间的记录：
```bash
GET /movies/_search?q=title:beautiful AND year:[2002 TO 2018%7D
{
	"profile":"true"
}
```
- **`year:[2002 TO 2018]`**：指定 `year` 字段的范围查询。

### 11. **通配符查询**
查询 `title` 字段以 `b` 开头的所有记录：
```bash
GET /movies/_search?q=title:b*
{
	"profile":"true"
}
```
- **`b*`**：使用通配符查询以 `b` 开头的字符串。

### 12. **模糊查询 & 近似度匹配**
- **模糊匹配**：查询 `title` 字段中的词语 `beautifl`，允许有 1 个字符的错误：
  ```bash
  GET /movies/_search?q=title:beautifl~1
  {
  	"profile":"true"
  }
  ```
- **近似度匹配**：查询 `title` 字段中的短语 `"Lord Rings"`，允许 2 个字符的误差：
  ```bash
  GET /movies/_search?q=title:"Lord Rings"~2
  {
  	"profile":"true"
  }
  ```

### 总结
这些查询展示了如何使用 Elasticsearch 提供的强大查询能力来根据不同的需求筛选和搜索数据。常见的查询类型包括：
- **基本查询**：用于查找单一词汇。
- **布尔查询**：组合多个查询条件。
- **范围查询**：查找特定范围内的记录。
- **通配符查询**：模糊匹配。
- **短语查询**：精确匹配整个短语。

同时，`profile` 参数可以帮助分析查询性能，优化查询响应。


3.8-RequestBody与QueryDSL简介

以下是您提供的 Elasticsearch 查询操作，包括分页、日期排序、源过滤、脚本字段、以及不同的查询类型（如 `match`、`match_phrase`）。这些操作展示了如何在 Elasticsearch 中进行更精细的查询和数据处理。

### 1. **忽略不存在的索引（`ignore_unavailable=true`）**
在查询时，如果访问的索引不存在，使用 `ignore_unavailable=true` 可以避免报错：
```bash
POST /movies,404_idx/_search?ignore_unavailable=true
{
  "profile": true,
  "query": {
    "match_all": {}
  }
}
```
- **`ignore_unavailable=true`**：在访问不存在的索引时，忽略错误。

### 2. **分页查询**
查询 `kibana_sample_data_ecommerce` 索引并分页返回数据（从第 10 条开始，返回 20 条记录）：
```bash
POST /kibana_sample_data_ecommerce/_search
{
  "from": 10,
  "size": 20,
  "query": {
    "match_all": {}
  }
}
```
- **`from=10`** 和 **`size=20`**：分页，跳过前 10 条，返回接下来的 20 条。

### 3. **日期排序**
按 `order_date` 字段降序排序：
```bash
POST kibana_sample_data_ecommerce/_search
{
  "sort": [{"order_date": "desc"}],
  "query": {
    "match_all": {}
  }
}
```
- **`sort`**：按照 `order_date` 字段降序排序。

### 4. **源过滤**
只返回文档中的 `order_date` 字段，不返回其他字段：
```bash
POST kibana_sample_data_ecommerce/_search
{
  "_source": ["order_date"],
  "query": {
    "match_all": {}
  }
}
```
- **`_source`**：指定只返回 `order_date` 字段的数据。

### 5. **脚本字段**
使用脚本字段计算新的字段（例如，拼接 `order_date` 和 `"hello"`）：
```bash
GET kibana_sample_data_ecommerce/_search
{
  "script_fields": {
    "new_field": {
      "script": {
        "lang": "painless",
        "source": "doc['order_date'].value+'hello'"
      }
    }
  },
  "query": {
    "match_all": {}
  }
}
```
- **`script_fields`**：使用 `painless` 脚本创建新的字段 `new_field`，将 `order_date` 字段与字符串 `"hello"` 拼接。

### 6. **`match` 查询**
查询 `movies` 索引中 `title` 字段包含 `"last christmas"` 的记录：
```bash
POST movies/_search
{
  "query": {
    "match": {
      "title": "last christmas"
    }
  }
}
```
- **`match`**：进行标准的全文匹配查询。

### 7. **`match` 查询并使用 AND 操作符**
查询 `title` 包含 `"last christmas"` 且两个词都必须出现的记录（使用 AND 操作符）：
```bash
POST movies/_search
{
  "query": {
    "match": {
      "title": {
        "query": "last christmas",
        "operator": "and"
      }
    }
  }
}
```
- **`operator: "and"`**：两个词必须同时出现在字段中。

### 8. **`match_phrase` 查询（短语匹配）**
查询 `title` 字段精确匹配 `"one love"` 作为一个短语：
```bash
POST movies/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "one love"
      }
    }
  }
}
```
- **`match_phrase`**：精确匹配整个短语，必须按照指定的顺序。

### 9. **`match_phrase` 查询并使用 `slop`**
查询 `title` 字段中的短语 `"one love"`，并允许 1 个词语的偏移（即词语可以有一定的顺序变化）：
```bash
POST movies/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "one love",
        "slop": 1
      }
    }
  }
}
```
- **`slop: 1`**：允许最多 1 个词语的顺序偏移（例如，`love one` 也可以匹配）。

### 总结
这些查询展示了 Elasticsearch 中常用的查询类型：
- **分页查询**：使用 `from` 和 `size` 来实现分页。
- **日期排序**：使用 `sort` 来对结果进行排序。
- **源过滤**：通过 `_source` 限制返回的字段。
- **脚本字段**：使用 `painless` 脚本来计算新字段。
- **布尔操作符**：通过 `operator` 来定义 `match` 查询的逻辑。
- **短语匹配和偏移**：通过 `match_phrase` 和 `slop` 来进行更灵活的查询。

这些查询可以帮助你在 Elasticsearch 中实现复杂的搜索需求。

[3.9-QueryString&SimpleQueryString查询]

以下是您提供的 Elasticsearch 查询操作，涉及到 `query_string` 和 `simple_query_string` 查询的使用，以及如何通过不同的字段和操作符执行复杂查询。这里的查询包括布尔操作符、字段匹配、以及指定默认操作符等。

### 1. **向索引添加文档**
首先，向 `users` 索引添加两个用户文档：
```bash
PUT /users/_doc/1
{
  "name": "Ruan Yiming",
  "about": "java, golang, node, swift, elasticsearch"
}

PUT /users/_doc/2
{
  "name": "Li Yiming",
  "about": "Hadoop"
}
```

### 2. **`query_string` 查询：查询名称中包含 "Ruan" 和 "Yiming" 的文档**
使用 `query_string` 查询，指定默认字段为 `name`，查询条件为 `"Ruan AND Yiming"`：
```bash
POST users/_search
{
  "query": {
    "query_string": {
      "default_field": "name",
      "query": "Ruan AND Yiming"
    }
  }
}
```
- **`query_string`**：用于执行复杂的查询，可以使用逻辑操作符（如 AND、OR）。

### 3. **`query_string` 查询：多个字段和复杂查询**
查询名称或简介字段包含 `"Ruan AND Yiming"` 或 `"Java AND Elasticsearch"` 的文档：
```bash
POST users/_search
{
  "query": {
    "query_string": {
      "fields": ["name", "about"],
      "query": "(Ruan AND Yiming) OR (Java AND Elasticsearch)"
    }
  }
}
```
- **`fields`**：可以指定多个字段进行查询。
- **`query`**：包含布尔操作符（如 AND、OR）。

### 4. **`simple_query_string` 查询（默认操作符为 OR）**
执行一个简单查询，默认操作符为 `OR`，查询 `name` 字段中包含 `"Ruan"` 或 `"Yiming"` 的文档：
```bash
POST users/_search
{
  "query": {
    "simple_query_string": {
      "query": "Ruan AND Yiming",
      "fields": ["name"]
    }
  }
}
```
- **`simple_query_string`**：适用于较简单的查询，避免了复杂的查询语法。

### 5. **`simple_query_string` 查询（默认操作符为 AND）**
查询 `name` 字段中包含 `"Ruan"` 且 `"Yiming"` 的文档，使用 `AND` 作为默认操作符：
```bash
POST users/_search
{
  "query": {
    "simple_query_string": {
      "query": "Ruan Yiming",
      "fields": ["name"],
      "default_operator": "AND"
    }
  }
}
```
- **`default_operator`**：设置默认操作符（`AND` 或 `OR`）。在此示例中，`AND` 用于要求 `"Ruan"` 和 `"Yiming"` 必须同时出现在 `name` 字段中。

### 6. **`query_string` 查询：查询电影标题包含 "Beautiful" 和 "Mind"**
执行 `query_string` 查询，查询电影标题中包含 `"Beautiful"` 和 `"Mind"` 的文档：
```bash
GET /movies/_search
{
  "profile": true,
  "query": {
    "query_string": {
      "default_field": "title",
      "query": "Beautiful AND Mind"
    }
  }
}
```
- **`query_string`**：执行复杂查询，可以使用布尔操作符（如 `AND`）。

### 7. **`query_string` 查询：多个字段查询**
查询电影的 `title` 或 `year` 字段包含 `"2012"` 的文档：
```bash
GET /movies/_search
{
  "profile": true,
  "query": {
    "query_string": {
      "fields": ["title", "year"],
      "query": "2012"
    }
  }
}
```
- **`fields`**：指定多个字段进行查询，这里同时查询 `title` 和 `year` 字段。

### 8. **`simple_query_string` 查询：使用 `+` 进行短语匹配**
查询电影标题中包含 `"Beautiful"` 和 `"mind"` 的文档，使用 `+` 确保两个词都出现在查询中：
```bash
GET /movies/_search
{
  "profile": true,
  "query": {
    "simple_query_string": {
      "query": "Beautiful +mind",
      "fields": ["title"]
    }
  }
}
```
- **`+` 操作符**：用于强制要求某个词出现在查询结果中。

---

### 总结
这些查询展示了如何在 Elasticsearch 中使用不同类型的查询：

- **`query_string`** 查询：用于执行复杂的查询，可以包含多个字段、布尔操作符（AND、OR、NOT）以及通配符等。
- **`simple_query_string`** 查询：比 `query_string` 更简单，适用于不需要复杂语法的查询，并且支持默认操作符的配置。
- **字段查询**：可以针对多个字段同时查询，指定多个字段使用 `fields` 参数。
- **布尔操作符**：如 `AND`、`OR`、`NOT`，控制查询的逻辑。

这些查询方式非常适合处理复杂的检索需求。

[3.10-DynamicMapping和常见字段类型]

您的示例展示了如何在 Elasticsearch 中管理映射（Mapping）以及如何处理动态字段和映射更新。以下是总结和解释：

### **映射和字段定义**

映射在 Elasticsearch 中类似于关系数据库中的表结构定义，主要用于：
- **定义字段的名称**：指定文档中的字段名。
- **定义字段的类型**：比如文本（`text`）、关键字（`keyword`）、日期（`date`）等。
- **定义倒排索引相关配置**：比如是否需要索引字段，以及字段的 `Analyzer` 配置。

### **动态映射（Dynamic Mapping）**

动态映射允许 Elasticsearch 根据新文档自动推断字段的类型。可以通过 `dynamic` 属性来配置动态行为：
- **`true`**（默认）：允许新字段自动被索引和映射。
- **`false`**：禁止自动创建新字段，但已存在字段可以继续使用。
- **`strict`**：禁止任何新的字段。如果文档包含未定义的字段，将导致错误。

### **操作演示**

1. **写入文档并查看映射**
    - 向 `mapping_test` 索引中添加一个文档，查看映射：
   ```bash
   PUT mapping_test/_doc/1
   {
     "firstName": "Chan",
     "lastName": "Jackie",
     "loginDate": "2018-07-24T10:29:48.103Z"
   }

   GET mapping_test/_mapping
   ```

2. **动态映射：推断字段类型**
    - 新字段 `uid`, `isVip`, `age` 等被自动推断并加入映射：
   ```bash
   PUT mapping_test/_doc/1
   {
     "uid": "123",
     "isVip": false,
     "isAdmin": "true",
     "age": 19,
     "height": 180
   }

   GET mapping_test/_mapping
   ```

3. **新增字段（默认动态）**
    - 如果动态映射为 `true`（默认），可以动态添加新的字段：
   ```bash
   PUT dynamic_mapping_test/_doc/1
   {
     "newField": "someValue"
   }

   POST dynamic_mapping_test/_search
   {
     "query": {
       "match": {
         "newField": "someValue"
       }
     }
   }
   ```

4. **修改动态设置为 `false`**
    - 修改 `dynamic` 设置为 `false`，禁止自动推断新字段：
   ```bash
   PUT dynamic_mapping_test/_mapping
   {
     "dynamic": false
   }

   PUT dynamic_mapping_test/_doc/10
   {
     "anotherField": "someValue"
   }

   POST dynamic_mapping_test/_search
   {
     "query": {
       "match": {
         "anotherField": "someValue"
       }
     }
   }
   ```

   在此示例中，`anotherField` 将不会被索引或搜索，因为动态映射已设置为 `false`。

5. **修改为 `strict` 模式**
    - 如果将 `dynamic` 设置为 `strict`，任何新的字段都会导致错误：
   ```bash
   PUT dynamic_mapping_test/_mapping
   {
     "dynamic": "strict"
   }

   PUT dynamic_mapping_test/_doc/12
   {
     "lastField": "value"
   }
   ```
   此时，写入新字段 `lastField` 会导致错误，HTTP 返回 `400` 错误码。

6. **删除索引**
    - 删除索引：
   ```bash
   DELETE dynamic_mapping_test
   ```

### **总结**

- **动态映射（Dynamic Mapping）**：允许 Elasticsearch 自动推断并添加新的字段，方便快速处理未知数据。
- **`dynamic` 配置**：控制是否允许 Elasticsearch 自动映射新字段：
    - `true`：默认允许动态字段；
    - `false`：禁止自动映射新字段；
    - `strict`：禁止所有未知字段，如果有未定义字段则会报错。
- **更改映射**：映射一旦建立后不能直接修改字段类型，要修改字段配置或添加新字段需要使用 `reindex` 操作。

这些操作有助于管理索引和字段的生命周期，确保数据能够根据业务需求灵活适应。


您的示例展示了 Elasticsearch 中显式设置映射（Mapping）的一些常见操作和参数。以下是各个设置项的详细介绍和使用案例。

### **1. `index: false` 设置**
通过设置字段的 `index: false`，可以阻止某些字段被索引。这样可以避免该字段参与倒排索引，也不会进行搜索。

**示例**:
```bash
DELETE users
PUT users
{
    "mappings" : {
      "properties" : {
        "firstName" : {
          "type" : "text"
        },
        "lastName" : {
          "type" : "text"
        },
        "mobile" : {
          "type" : "text",
          "index": false  # 禁止索引该字段
        }
      }
    }
}

PUT users/_doc/1
{
  "firstName": "Ruan",
  "lastName": "Yiming",
  "mobile": "12345678"
}

POST /users/_search
{
  "query": {
    "match": {
      "mobile": "12345678"  # 搜索时不会命中mobile字段
    }
  }
}
```
在上面的示例中，`mobile` 字段被设置为 `index: false`，因此无法用于搜索查询。

### **2. `null_value` 设置**
通过设置 `null_value`，可以在文档中存储 `null` 值，并用指定的替代值替换 `null`。这对于查询时需要处理 `null` 数据特别有用。

**示例**:
```bash
DELETE users
PUT users
{
    "mappings" : {
      "properties" : {
        "firstName" : {
          "type" : "text"
        },
        "lastName" : {
          "type" : "text"
        },
        "mobile" : {
          "type" : "keyword",
          "null_value": "NULL"  # null值替代为"NULL"
        }
      }
    }
}

PUT users/_doc/1
{
  "firstName": "Ruan",
  "lastName": "Yiming",
  "mobile": null  # 实际值是null
}

PUT users/_doc/2
{
  "firstName": "Ruan2",
  "lastName": "Yiming2"  # 没有mobile字段
}

GET users/_search
{
  "query": {
    "match": {
      "mobile": "NULL"  # 搜索时会匹配到null值字段
    }
  }
}
```
此设置可以确保查询时用 `NULL` 代替存储的 `null` 值。

### **3. `copy_to` 设置**
`copy_to` 可以将多个字段的内容复制到另一个字段，方便进行联合查询或索引。

**示例**:
```bash
DELETE users
PUT users
{
  "mappings": {
    "properties": {
      "firstName": {
        "type": "text",
        "copy_to": "fullName"  # 将firstName复制到fullName字段
      },
      "lastName": {
        "type": "text",
        "copy_to": "fullName"  # 将lastName复制到fullName字段
      }
    }
  }
}

PUT users/_doc/1
{
  "firstName": "Ruan",
  "lastName": "Yiming"
}

GET users/_search?q=fullName:(Ruan Yiming)

POST users/_search
{
  "query": {
    "match": {
      "fullName": {
        "query": "Ruan Yiming",
        "operator": "and"  # 联合查询fullName字段
      }
    }
  }
}
```
这里，`firstName` 和 `lastName` 字段的内容会被复制到 `fullName` 字段，从而使得查询变得更加方便。

### **4. 数组类型处理**
Elasticsearch 支持数组字段。数组中的每个元素会被视为一个单独的值，这对于多值字段非常有用。

**示例**:
```bash
PUT users/_doc/1
{
  "name": "onebird",
  "interests": "reading"  # 字段为单一值
}

PUT users/_doc/2
{
  "name": "twobirds",
  "interests": ["reading", "music"]  # 字段为数组
}

POST users/_search
{
  "query": {
    "match_all": {}
  }
}

GET users/_mapping
```
在此示例中，`interests` 字段既可以是单个值也可以是数组，Elasticsearch 会将每个数组元素视为独立的值进行索引。

---

### **总结常见的 Mapping 参数**

- **`type`**: 字段的数据类型，如 `text`, `keyword`, `date` 等。
- **`index`**: 控制是否索引该字段，`false` 表示不索引，默认为 `true`。
- **`null_value`**: 控制 `null` 值的替代值。可以设置为任何替代值，如 `"NULL"`。
- **`copy_to`**: 将多个字段的内容复制到一个目标字段中，便于联合查询。
- **`dynamic`**: 控制动态映射，`true` 表示允许自动推断新字段，`false` 表示禁止新字段自动映射，`strict` 则表示严格模式下不允许任何新字段。

通过这些映射设置，可以更好地控制数据如何存储和索引，以及如何在查询时处理和优化字段。


您的示例展示了 Elasticsearch 中关于多字段特性以及自定义分析器（Analyzer）的配置。以下是对各个分析器和相关配置的详细解释。

### **1. 自定义 Analyzer 和 Char Filter**
Elasticsearch 提供了灵活的机制来配置分词器（Tokenizer）和字符过滤器（Char Filter）。这些配置用于在索引和搜索过程中控制文本的处理方式。

#### **自定义 Tokenizer 和 Char Filter 示例**
- **Keyword Tokenizer**: `keyword` tokenizer 将整个文本当作一个单独的 token 进行处理。
- **HTML Strip Char Filter**: `html_strip` char filter 用于从文本中去除 HTML 标签。

```bash
POST _analyze
{
  "tokenizer": "keyword",  # 使用关键词分词器
  "char_filter": ["html_strip"],  # 使用html_strip字符过滤器
  "text": "<b>hello world</b>"  # 输入带有HTML标签的文本
}
```
输出将会是：
```json
{
  "tokens": [
    {
      "token": "hello world",
      "start_offset": 3,
      "end_offset": 15,
      "type": "word",
      "position": 0
    }
  ]
}
```
此配置移除了文本中的 `<b></b>` HTML 标签，只保留了纯文本 "hello world"。

#### **Path Hierarchy Tokenizer**
`path_hierarchy` tokenizer 用于处理文件路径等层级结构的数据，将路径按分隔符（如 `/`）切割。

```bash
POST _analyze
{
  "tokenizer": "path_hierarchy",  # 使用路径层级分词器
  "text": "/user/ymruan/a/b/c/d/e"
}
```
输出会将路径按 `/` 切分：
```json
{
  "tokens": [
    {
      "token": "/user/ymruan/a/b/c/d/e",
      "start_offset": 0,
      "end_offset": 21,
      "type": "word",
      "position": 0
    },
    {
      "token": "/user/ymruan/a/b/c/d",
      "start_offset": 0,
      "end_offset": 16,
      "type": "word",
      "position": 1
    },
    ...
  ]
}
```

### **2. Char Filter 替换符号**
字符过滤器可以用于替换文本中的特定符号或字符。例如，使用 `mapping` 类型的 `char_filter`，可以替换文本中的字符（如将 `-` 替换为 `_`）。

```bash
POST _analyze
{
  "tokenizer": "standard",  # 使用标准分词器
  "char_filter": [
    {
      "type": "mapping",  # 使用字符映射过滤器
      "mappings": ["- => _"]
    }
  ],
  "text": "123-456, I-test! test-990 650-555-1234"
}
```
此配置将文本中的 `-` 替换为 `_`，输出将是：
```json
{
  "tokens": [
    {
      "token": "123_456",
      "start_offset": 0,
      "end_offset": 7,
      "type": "word",
      "position": 0
    },
    ...
  ]
}
```

### **3. Char Filter 替换表情符号**
你还可以使用字符过滤器将表情符号替换为特定的文本（如将 `:)` 替换为 `happy`，`:(` 替换为 `sad`）。

```bash
POST _analyze
{
  "tokenizer": "standard",
  "char_filter": [
    {
      "type": "mapping",
      "mappings": [":) => happy", ":( => sad"]
    }
  ],
  "text": ["I am felling :)", "Feeling :( today"]
}
```
输出：
```json
{
  "tokens": [
    {
      "token": "I",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "am",
      "start_offset": 2,
      "end_offset": 4,
      "type": "word",
      "position": 1
    },
    {
      "token": "felling",
      "start_offset": 5,
      "end_offset": 12,
      "type": "word",
      "position": 2
    },
    {
      "token": "happy",  # 替换表情符号:) 为 happy
      "start_offset": 13,
      "end_offset": 18,
      "type": "word",
      "position": 3
    }
  ]
}
```

### **4. 使用 Stop 和 Snowball 过滤器**
使用 `stop` 过滤器来删除常见的停用词，使用 `snowball` 过滤器进行词干提取（如将 "playing" 转换为 "play"）。

```bash
GET _analyze
{
  "tokenizer": "whitespace",  # 使用空格分词器
  "filter": ["stop", "snowball"],  # 使用停用词和词干提取
  "text": ["The girls in China are playing this game!"]
}
```
输出：
```json
{
  "tokens": [
    {
      "token": "girl",
      "start_offset": 4,
      "end_offset": 9,
      "type": "word",
      "position": 0
    },
    {
      "token": "china",
      "start_offset": 14,
      "end_offset": 19,
      "type": "word",
      "position": 1
    },
    ...
  ]
}
```

### **5. 正则表达式替换**
通过 `pattern_replace` 字符过滤器，您可以根据正则表达式来替换文本中的特定部分。

```bash
POST _analyze
{
  "tokenizer": "standard",
  "char_filter": [
    {
      "type": "pattern_replace",
      "pattern": "http://(.*)",
      "replacement": "$1"
    }
  ],
  "text": "http://www.elastic.co"
}
```
此配置将 `http://` 替换为一个空字符串，结果为：
```json
{
  "tokens": [
    {
      "token": "www.elastic.co",
      "start_offset": 7,
      "end_offset": 23,
      "type": "word",
      "position": 0
    }
  ]
}
```

### **总结：**
- **Tokenizer**: 负责将输入文本切割成词条（token）。如 `keyword`, `standard`, `path_hierarchy`。
- **Char Filter**: 处理文本中的字符替换、去除特殊符号或HTML标签等，如 `html_strip`, `mapping`, `pattern_replace`。
- **Filter**: 用于文本处理，如去除停用词（`stop`）或词干提取（`snowball`）。

这些工具的组合允许您灵活地处理和分析文本数据，并针对特定的业务需求优化索引和搜索行为。

在 Elasticsearch 中，**Index Templates** 和 **Dynamic Templates** 是两种强大的工具，用于控制如何定义和管理索引的映射和设置。以下是这两者的详细解释和示例。

### **1. Index Templates (索引模板)**

**Index Templates** 是一种配置模板，用于为匹配特定模式的索引自动配置设置、映射和其他配置。当创建一个新索引时，如果该索引匹配模板中的 `index_patterns` 配置，模板中的设置、映射和其他配置就会自动应用到新索引上。

#### **创建默认模板**
下面是创建一个默认模板的示例：
```bash
PUT _template/template_default
{
  "index_patterns": ["*"],  # 匹配所有索引
  "order": 0,
  "version": 1,
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  }
}
```
这会为所有索引设置一个模板，其中包含分片数量为1、副本数量为1。

#### **为特定模式创建模板**
另一个例子是为 `test*` 开头的索引创建模板：
```bash
PUT /_template/template_test
{
  "index_patterns": ["test*"],  # 匹配所有以test开头的索引
  "order": 1,
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 2
  },
  "mappings": {
    "date_detection": false,  # 禁止自动检测日期字段
    "numeric_detection": true  # 自动检测数字字段
  }
}
```

#### **查看模板**
你可以查看已定义的模板：
```bash
GET /_template/template_default
GET /_template/temp*  # 查看所有模板
```

#### **删除模板**
删除模板的命令如下：
```bash
DELETE /_template/template_default
DELETE /_template/template_test
```

---

### **2. Dynamic Templates (动态模板)**

**Dynamic Templates** 用于在创建索引时自动匹配字段类型和名称，根据这些动态模板来生成字段的映射规则。可以根据字段的类型（例如字符串、数字）和字段名称（例如以某些字符开头的字段）来定义映射。

#### **自动映射数字字符串和日期字符串**
例如，数字字符串会被映射为 `text` 类型，日期字符串会被映射为日期类型：
```bash
PUT ttemplate/_doc/1
{
  "someNumber": "1",        # 字符串数字
  "someDate": "2019/01/01"  # 字符串日期
}
GET ttemplate/_mapping
```
在这个例子中，`someNumber` 会被映射成 `text` 类型，而 `someDate` 会被映射成 `date` 类型。

#### **定义动态模板映射规则**
使用 `dynamic_templates` 可以根据字段类型和名称定义映射。例如，以下动态模板将以 `is*` 开头的字符串字段映射为 `boolean` 类型，其他字符串字段映射为 `keyword` 类型：
```bash
DELETE my_index

PUT my_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_as_boolean": {
          "match_mapping_type": "string",   # 匹配所有字符串类型
          "match": "is*",  # 字段名称以is开头
          "mapping": {
            "type": "boolean"
          }
        }
      },
      {
        "strings_as_keywords": {
          "match_mapping_type": "string",   # 匹配所有字符串类型
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}
```
在这个示例中，`isVIP` 字段会被映射为 `boolean` 类型，而其他字符串类型的字段则会被映射为 `keyword` 类型。

#### **结合路径匹配**
动态模板不仅可以基于字段名称，还可以基于字段的路径。例如，以下模板会将 `name.*` 路径下的所有字段映射为 `text` 类型，并将这些字段复制到 `full_name` 字段中：
```bash
PUT my_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "full_name": {
          "path_match": "name.*",  # 匹配name下的所有字段
          "path_unmatch": "*.middle",  # 排除middle字段
          "mapping": {
            "type": "text",
            "copy_to": "full_name"  # 将值复制到full_name字段
          }
        }
      }
    ]
  }
}
```
然后，插入以下数据：
```bash
PUT my_index/_doc/1
{
  "name": {
    "first": "John",
    "middle": "Winston",
    "last": "Lennon"
  }
}
```

在这个例子中，`first` 和 `last` 字段将被映射为 `text` 类型，并且它们的内容会被复制到 `full_name` 字段中。

#### **搜索字段**
可以通过 `full_name` 字段来搜索：
```bash
GET my_index/_search?q=full_name:John
```

---

### **总结：**
- **Index Templates** 是为匹配特定索引模式（`index_patterns`）的所有索引自动配置设置和映射的模板。
- **Dynamic Templates** 用于根据字段类型和字段名称自动映射索引字段。你可以根据字段类型（如 `string`）或字段路径（如 `name.*`）来定义映射规则。

这些配置使得索引的创建和管理更加灵活，可以根据实际数据自动推断和调整映射，从而提高了 Elasticsearch 的可用性和效率。



Elasticsearch 提供了强大的 **聚合** 功能，用于对数据进行汇总和分析。聚合是 Elasticsearch 搜索的一个核心功能，允许你通过多种方式对数据进行分组、统计、求最大/最小值、计算平均值等操作。以下是一些常见的聚合分析操作和示例：

### **1. 基本的聚合分析：按目的地分桶统计**
在这个示例中，我们按航班的目的地（`DestCountry`）进行分桶统计，查看每个目的地的航班数量：
```bash
GET kibana_sample_data_flights/_search
{
  "size": 0,  # 不返回实际的文档数据，只返回聚合结果
  "aggs": {
    "flight_dest": {
      "terms": {
        "field": "DestCountry"  # 按目的地国家分桶
      }
    }
  }
}
```
在这个聚合查询中，`terms` 聚合用于对 `DestCountry` 字段的值进行分桶，返回每个目的地国家的航班数量。

### **2. 增加更多统计：平均、最大、最小价格**
你可以在分桶内部添加更多聚合来获取其他统计信息，如每个目的地的平均票价、最高票价和最低票价：
```bash
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flight_dest": {
      "terms": {
        "field": "DestCountry"
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "AvgTicketPrice"  # 计算平均票价
          }
        },
        "max_price": {
          "max": {
            "field": "AvgTicketPrice"  # 计算最高票价
          }
        },
        "min_price": {
          "min": {
            "field": "AvgTicketPrice"  # 计算最低票价
          }
        }
      }
    }
  }
}
```
在这个查询中，我们为每个目的地添加了三个聚合：`avg_price`、`max_price` 和 `min_price`，分别计算每个目的地的平均票价、最大票价和最小票价。

### **3. 聚合价格与天气信息：统计与多重聚合**
如果你还想结合天气信息（如 `DestWeather` 字段）与航班价格统计一起进行聚合，可以使用嵌套聚合：
```bash
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flight_dest": {
      "terms": {
        "field": "DestCountry"
      },
      "aggs": {
        "stats_price": {
          "stats": {
            "field": "AvgTicketPrice"  # 计算票价的多项统计信息
          }
        },
        "weather": {
          "terms": {
            "field": "DestWeather",  # 按天气类型分桶
            "size": 5  # 只返回前5个天气类型
          }
        }
      }
    }
  }
}
```
在这个示例中，我们不仅计算了每个目的地的票价统计信息（如平均票价、最大票价等），还将天气信息作为另一个 `terms` 聚合，显示每个目的地的天气类型，并限制返回的天气类型数量为 5 个。

---

### **常见的聚合类型：**
- **`terms` 聚合**：将文档按某个字段的值进行分桶，常用于统计词频或其他分类数据。
- **`avg` 聚合**：计算某个数值字段的平均值。
- **`sum` 聚合**：计算某个数值字段的总和。
- **`min` 聚合**：返回某个数值字段的最小值。
- **`max` 聚合**：返回某个数值字段的最大值。
- **`stats` 聚合**：返回字段的统计信息（包括：最小值、最大值、平均值、总和和文档数量）。
- **`date_histogram` 聚合**：按日期进行分桶，适用于时间序列数据。
- **`histogram` 聚合**：按数值区间进行分桶，常用于数值数据的统计。

---

### **相关阅读：**
- [Elasticsearch 聚合文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/search-aggregations.html) 详细介绍了各种聚合的使用方式和示例。

聚合功能非常强大，能帮助你从海量数据中提取有价值的信息，适用于各种数据分析场景。


