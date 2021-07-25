# [删除文档](08_deleting_document.md)   
&emsp;&emsp;删除文档的语法和我们所知道的规则相同，只是使用 DELETE 方法  
```$xslt
DELETE /website/blog/123
```
&emsp;&emsp;如果找到该文档，Elasticsearch 将要返回一个 200 ok 的 HTTP 响应码，
和一个类似以下结构的响应体。注意，字段 _version 值已经增加:  
```$xslt
{
  "found" :    true,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 3
}
```
&emsp;&emsp;如果文档没有找到，我们将得到 404 Not Found 的响应码和类似这样的响应体：
&emsp;&emsp;
```$xslt
```
&emsp;&emsp;即使文档不存在（ Found 是 false ）， _version 值仍然会增加。
这是 Elasticsearch 内部记录本的一部分，用来确保这些改变在跨多节点时以正确的顺序执行。  
> Note
>>&emsp;&emsp;正如已经在[更新整个文档](06_updating_whole_document.md)中提到的，删除文档不会立即将文档从磁盘中删除，只是将文档标记为已删除状态。
随着你不断的索引更多的数据，Elasticsearch 将会在后台清理标记为已删除的文档。