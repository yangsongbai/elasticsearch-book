# [使用查询表达式搜索](09_search_with_query_dsl.md)
&emsp;&emsp;Query-string 搜索通过命令非常方便地进行临时性的即席搜索 ，但它有自身的局限性（参见 [轻量](08_search_lite.md)搜索 ）。
Elasticsearch 提供一个丰富灵活的查询语言叫做 查询表达式 ， 它支持构建更加复杂和健壮的查询。

&emsp;&emsp;领域特定语言 （DSL）， 使用 JSON 构造了一个请求。我们可以像这样重写之前的查询所有名为 Smith 的搜索 ：  
```$xslt
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```
&emsp;&emsp;返回结果与之前的查询一样，但还是可以看到有一些变化。其中之一是，不再使用 query-string 参数，
而是一个请求体替代。这个请求使用 JSON 构造，并使用了一个 match 查询（属于查询类型之一，后面将继续介绍）。