# [取回一个文档](04_retrieving_document.md)  
&emsp;&emsp;为了从 Elasticsearch 中检索出文档，我们仍然使用相同的 `_index` , `_type` , 和 `_id` ，
但是 `HTTP` 谓词更改为 `GET` :     
```$xslt
GET /website/blog/123?pretty
```
&emsp;&emsp;响应体包括目前已经熟悉了的元数据元素，再加上 _source 字段，
这个字段包含我们索引数据时发送给 Elasticsearch 的原始 JSON 文档
```$xslt
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out...",
      "date":  "2014/01/01"
  }
}
```
>注意
>>&emsp;&emsp;在请求的查询串参数中加上 pretty 参数，正如前面的例子中看到的，这将会调用 Elasticsearch 的 pretty-print 功能，
该功能 使得 JSON 响应体更加可读。
但是， _source 字段不能被格式化打印出来。相反，我们得到的 _source 字段中的 JSON 串，刚好是和我们传给它的一样。 

&emsp;&emsp;GET 请求的响应体包括 {"found": true} ，这证实了文档已经被找到。 
如果我们请求一个不存在的文档，我们仍旧会得到一个 JSON 响应体，
但是 found 将会是 false 。 此外， HTTP 响应码将会是 404 Not Found ，而不是 200 OK 。 

&emsp;&emsp;我们可以通过传递 -i 参数给 curl 命令，该参数能够显示响应的头部：  
```$xslt
curl -i -XGET http://localhost:9200/website/blog/124?pretty
``` 
&emsp;&emsp;显示响应头部的响应体现在类似这样：    
```$xslt
HTTP/1.1 404 Not Found
Content-Type: application/json; charset=UTF-8
Content-Length: 83

{
  "_index" : "website",
  "_type" :  "blog",
  "_id" :    "124",
  "found" :  false
}
```
# 返回文档的一部分  
&emsp;&emsp;默认情况下， GET 请求会返回整个文档，
这个文档正如存储在 _source 字段中的一样。
但是也许你只对其中的 title 字段感兴趣。
单个字段能用 _source 参数请求得到，多个字段也能使用逗号分隔的列表来指定。  
```$xslt
GET /website/blog/123?_source=title,text
```
&emsp;&emsp;该 _source 字段现在包含的只是我们请求的那些字段，并且已经将 date 字段过滤掉了。
```$xslt
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```
&emsp;&emsp;或者，如果你只想得到 _source 字段，不需要任何元数据，你能使用 _source 端点：
```$xslt
GET /website/blog/123/_source
```
&emsp;&emsp;那么返回的的内容如下所示：
```$xslt
{
   "title": "My first blog entry",
   "text":  "Just trying this out...",
   "date":  "2014/01/01"
}
```
