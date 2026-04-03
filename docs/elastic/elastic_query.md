# Elastic中的查询

elastic的查询请求，是一个Json结构的内容，最简单的查询全部，如下：

```json
{
  "query": {
    "match_all": {}
  }
}
```
返回内容为

![](/assets/img/tutorial/elastic/5-1.jpg)

从elasticsearch-head的`复合查询[+]`标签左边的 操作内容来看，主要是3个输入部分：

1. 第一个输入框，elastic的节点地址，IP+端口，这里是`127.0.0.1:9200`
2. 第二个输入框，是操作类型，当下只讨论查询，所以以`_search`结尾
3. 第三个输入文本框，是请求体，json格式

## 理解返回的Json结构

我觉得，有必要先理解一下返回的Json结构体，以上图为例：

|名称|含义|
|---|---|
|took|查询耗时毫秒数|
|timed_out|是否有分片超时，如果为true，表示只返回了部分结果，有的分片超时未返回|
|_shards.total|总共的分片数量|
|_shards.successful|成功响应的分片数量|
|_shards.skipped|跳过未处理的分片数量|
|_shards.failed|响应超时的分片数量|
|htis.total.value|返回结果的总共数量|
|htis.total.relation|查询的方式|
|htis.max_score|这个返回结果中的最大得分数|
|htis.hits._index|文档所在的索引|
|htis.hits._id|文档的id|
|htis.hits._score|文档的相关性得分|
|htis.hits._source.xxx|文档中的字段|

## 明确查询范围

在第二个操作输入框中，对于查询来说，完整的格式，是这样的
```txt
/[_all|*|{[+|-]index_name[*]+}]/[{type_name+}]/_search
```
这里，我用了`正则表达式`的符号，具体例子如下：

```txt
/_search: 搜索整个集群
/_all/_search: 搜索全部索引
/*/_search: 搜全部索引
/book_idnex/_search: 只在索引book_index中搜索
/+book*,-book_index/_search: 在已book开头，但不包含book_index索引中搜索
```

第二个文本框中的各种输入，目的是限制搜索范围，即：

搜索的第一步是：**先明确自己的搜索范围**

## 构建查询Json查询体

Json体的第一层中，允许出现以下字段：

|名称|含义|
|---|---|
|query|查询体，最核心部分，包含**查询上下文（query context）**和**过滤上下文（filter context）**，它俩最大的区别是，前者会计算相关度得分，后者不会，后者只是简单的判断是否匹配|
|size|返回数量|
|from|分页操作，从0开始，类似数组下标索引|
|_source|指定返回结果中_source字段的内容，可以是json数组，支持通配符，也可以是json对象，包含**include**和**exclude**两个json数组对象|
|sort|排序字段，默认按评分(即：相关度)排序|

例子1：
```json
{
  "query": {
    "match_all": {}
  },
  "size": 3,
  "from": 2,
  "sort": ["year","pageNum"],
  "_source": ["name","year"]
}
```
其中，如果sort和_source只有一个字段，可以直接指定字符串，不用数组。

例子2：
```json
{
  "query": {
    "match_all": {}
  },
  "size": 3,
  "from": 2,
  "sort": {
    "year": "desc"
  },
  "_source": {
    "include": ["name","year"],
    "exclude": ["introduction"]
  }
}
```
这种格式显得稍微复杂些，但功能相对来说更全面

## 支持的查询类型

elastic将查询条件分为两类
1. 节点条件（Leaf query clauses）：只包含查询字段内容，如，`match`、`term`、`range`等
2. 组合条件（Compound query clauses）：包含其他的查询条件或节点条件，如，`bool`等

#### 节点条件

|名称|含义|
|---|---|
|match||
|term||
|range|按范围进行查询|

#### 组合条件

|名称|含义|
|---|---|
|bool|包含`must`、`should`、`must_not`、`filter`子条件，`must`和`should`会累加其评分，`must_not`和`filter`在filter上下文中执行|
|boosting|包含`positive`、`negative`、`negative_boost`字段|
|constant_score|包含`filter`、`boost`字段|
|dix_max|包含`queries`、`tie_breaker`字段|
|function_score|包含`query`、`boost`、`random_score`、`boost_mode`、`functions`、`max_boost`、`score_mode`、`boost_mode`、`min_score`、`script_score`等字段，相比较最为复杂|

1. bool条件中允许出现的字段名称：must、should、must_not、filter、minimum_should_match、boost
1. boosting条件中允许出现的字段名称：positive、negative、negative_boost
1. constant_score条件中允许出现的名称：filter、boost
1. dix_max条件中允许出现的名称：queries、tie_breaker
1. function_socre条件中允许出现的名称：query、boost、random_score、boost_mode、functions、max_boost、score_mode、boost_mode、min_score、script_score

## 全文查询

|名称|含义|
|---|---|
|intervals query||
|match query||
|match_bool_prefix query||
|match_phrase query||
|match_phrase_prefix query||
|multi_match query||
|combined_fields query||
|query_string query||
|simple_query_string query||

## 其他

1. 节点查询中，可以定义一个名为`_name`的字段，用来跟踪 返回内容属于哪个查询
