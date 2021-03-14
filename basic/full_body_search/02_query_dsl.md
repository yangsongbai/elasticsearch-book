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