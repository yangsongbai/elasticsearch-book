# [最重要的查询](04_most_important_queries.md) 
&emsp;&emsp;虽然 Elasticsearch 自带了很多的查询，但经常用到的也就那么几个。
我们将在 深入搜索 章节详细讨论那些查询的细节，接下来我们对最重要的几个查询进行简单介绍。  
## match_all 查询
&emsp;&emsp;`match_all` 查询简单的匹配所有文档。在没有指定查询方式时，它是默认的查询：  
```
{ "match_all": {}}
```
它经常与 `filter` 结合使用—​例如，检索收件箱里的所有邮件。
所有邮件被认为具有相同的相关性，所以都将获得分值为 `1` 的中性 `_score`。

## match 查询
&emsp;&emsp;无论你在任何字段上进行的是全文搜索还是精确查询，match 查询是你可用的标准查询。

&emsp;&emsp;如果你在一个全文字段上使用 match 查询，在执行查询前，它将用正确的分析器去分析查询字符串：
```
{ "match": { "tweet": "About Search" }}
```
&emsp;&emsp;如果在一个精确值的字段上使用它，例如数字、日期、布尔或者
一个 `not_analyzed `字符串字段，那么它将会精确匹配给定的值：
```
{ "match": { "age":    26           }}
{ "match": { "date":   "2014-09-01" }}
{ "match": { "public": true         }}
{ "match": { "tag":    "full_text"  }}
```
>Tip
>>对于精确值的查询，你可能需要使用 `filter` 语句来取代 `query`，
因为 `filter` 将会被缓存。接下来，我们将看到一些关于 `filter` 的例子。

&emsp;&emsp;不像我们在 轻量 搜索 章节介绍的字符串查询（query-string search）， 
match 查询不使用类似 +user_id:2 +tweet:search 的查询语法。
它只是去查找给定的单词。这就意味着将查询字段暴露给你的用户是安全的；
你需要控制那些允许被查询字段，不易于抛出语法异常。
## multi_match 查询
&emsp;&emsp;`multi_match` 查询可以在多个字段上执行相同的 `match` 查询：
```
{
    "multi_match": {
        "query":    "full text search",
        "fields":   [ "title", "body" ]
    }
}
```
## range查询 
&emsp;&emsp;range 查询找出那些落在指定区间内的数字或者时间：
```
{
    "range": {
        "age": {
            "gte":  20,
            "lt":   30
        }
    }
}
```
被允许的操作符如下：   
`gt`   
&emsp;&emsp;大于   
`gte`    
&emsp;&emsp;大于等于    
`lt`   
&emsp;&emsp;小于   
`lte`    
&emsp;&emsp;小于等于    

## term查询
&emsp;&emsp;`term` 查询被用于精确值匹配，这些精确值可能是数字、时间、布尔或者那些 `not_analyzed` 的字符串：
```
{ "term": { "age":    26           }}
{ "term": { "date":   "2014-09-01" }}
{ "term": { "public": true         }}
{ "term": { "tag":    "full_text"  }}
```
&emsp;&emsp;`term` 查询对于输入的文本不 分析 ，所以它将给定的值进行精确查询。

## terms查询  
&emsp;&emsp;`terms` 查询和 `term` 查询一样，但它允许你指定多值进行匹配。
如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件：
```
{ "terms": { "tag": [ "search", "full_text", "nosql" ] }}
```
&emsp;&emsp;和 `term` 查询一样，terms 查询对于输入的文本不分析。
它查询那些精确匹配的值（包括在大小写、重音、空格等方面的差异）。

## exists 查询和 missing 查询
&emsp;&emsp;`exists` 查询和 `missing` 查询被用于查找那些指定字段中有值 (`exists`) 或无值 (`missing`) 的文档。
这与`SQL`中的 `IS_NULL` (`missing`) 和 `NOT IS_NULL` (`exists`) 在本质上具有共性：  
```
{
    "exists":   {
        "field":    "title"
    }
}
```
&emsp;&emsp;这些查询经常用于某个字段有值的情况和某个字段缺值的情况。

