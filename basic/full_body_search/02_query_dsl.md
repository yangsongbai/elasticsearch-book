# [查询表达式](02_query_dsl.md)    
&emsp;&emsp;查询表达式(`Query DSL`)是一种非常灵活又富有表现力的 查询语言。 
`Elasticsearch` 使用它可以以简单的 `JSON` 接口来展现 `Lucene` 功能的绝大部分。
在你的应用中，你应该用它来编写你的查询语句。它可以使你的查询语句更灵活、更精确、易读和易调试。

要使用这种查询表达式，只需将查询语句传递给 `query` 参数：
```
GET /_search
{
    "query": YOUR_QUERY_HERE
}
```
&emsp;&emsp;空查询（`empty search`） `—{}— `在功能上等价于使用 `match_all` 查询， 正如其名字一样，匹配所有文档：
```
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```
## 查询语句的结构  
一个查询语句的典型结构：
```
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}
```
如果是针对某个字段，那么它的结构如下：
```
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
```
举个例子，你可以使用 `match `查询语句 来查询 `tweet` 字段中包含 `elasticsearch` 的 `tweet`：
```
{
    "match": {
        "tweet": "elasticsearch"
    }
}
```
完整的查询请求如下：
```
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
```
## 合并查询语句  
&emsp;&emsp;查询语句(`Query clauses`) 就像一些简单的组合块，
这些组合块可以彼此之间合并组成更复杂的查询。这些语句可以是如下形式：

 - 叶子语句（`Leaf clauses`） (就像 `match` 语句) 被用于将查询字符串和一个字段（或者多个字段）对比。
 - 复合(`Compound`) 语句 主要用于 合并其它查询语句。 比如，一个 `bool` 语句 允许在你需要的时候组合其它语句，
 无论是 `must` 匹配、 `must_not` 匹配还是 `should` 匹配，同时它可以包含不评分的过滤器（`filters`）：  
 
```
{
    "bool": {
        "must":     { "match": { "tweet": "elasticsearch" }},
        "must_not": { "match": { "name":  "mary" }},
        "should":   { "match": { "tweet": "full text" }},
        "filter":   { "range": { "age" : { "gt" : 30 }} }
    }
}
```
emsp;&emsp;一条复合语句可以合并 任何 其它查询语句，包括复合语句，了解这一点是很重要的。
这就意味着，复合语句之间可以互相嵌套，可以表达非常复杂的逻辑。

emsp;&emsp;例如，以下查询是为了找出信件正文包含 business opportunity 的星标邮件，
或者在收件箱正文包含 business opportunity 的非垃圾邮件：
```
{
    "bool": {
        "must": { "match":   { "email": "business opportunity" }},
        "should": [
            { "match":       { "starred": true }},
            { "bool": {
                "must":      { "match": { "folder": "inbox" }},
                "must_not":  { "match": { "spam": true }}
            }}
        ],
        "minimum_should_match": 1
    }
}
```
emsp;&emsp;到目前为止，你不必太在意这个例子的细节，我们会在后面详细解释。
最重要的是你要理解到，一条复合语句可以将多条语句 `— `叶子语句和其它复合语句 `—` 合并成一个单一的查询语句。