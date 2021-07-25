# [空查询](01_empty_search.md)   
让我们以最简单的 search API 的形式开启我们的旅程，空查询将返回所有索引库（indices)中的所有文档：
```
GET /_search
{} (a)
```
`(a)这是一个空的请求体。`  
&emsp;&emsp;只用一个查询字符串，你就可以在一个、多个或者 `_all` 索引库（`indices`）中查询：
```
GET /index_2014*/type/_search
{}
```
&emsp;&emsp;同时你可以使用 from 和 size 参数来分页：
```
GET /_search
{
  "from": 30,
  "size": 10
}
```
>一个带请求体的 GET 请求？
>>&emsp;&emsp;某些特定语言（特别是 JavaScript）的 HTTP 库是不允许 GET 请求带有请求体的。
事实上，一些使用者对于 GET 请求可以带请求体感到非常的吃惊。
  
>>&emsp;&emsp;而事实是这个RFC文档 RFC 7231— 一个专门负责处理 `HTTP` 语义和
内容的文档 — 并没有规定一个带有请求体的 GET 请求应该如何处理！结果是，一些 `HTTP` 服务器允许这样子，
而有一些 — 特别是一些用于缓存和代理的服务器 — 则不允许。
  
>>&emsp;&emsp;对于一个查询请求，Elasticsearch 的工程师偏向于使用 GET 方式，
因为他们觉得它比 `POST` 能更好的描述信息检索（`retrieving information`）的行为。
然而，因为带请求体的 `GET` 请求并不被广泛支持，所以 search API同时支持 `POST` 请求：  
```
POST /_search
{
  "from": 30,
  "size": 10
}
```
>>&emsp;&emsp;类似的规则可以应用于任何需要带请求体的 GET API。

&emsp;&emsp;我们将在聚合 `聚合` 章节深入介绍聚合（`aggregations`），而现在，我们将聚焦在查询。

&emsp;&emsp;相对于使用晦涩难懂的查询字符串的方式，一个带请求体的查询允许我们使用 
查询领域特定语言（`query domain-specific language`） 或者 `Query DSL` 来写查询语句。