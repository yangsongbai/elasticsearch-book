# [代价较小的批量操作](13_cheaper_in_bulk.md)     
&emsp;&emsp;与 `mget` 可以使我们一次取回多个文档同样的方式， `bulk API` 
允许在单个步骤中进行多次 `create `、 `index` 、 `update` 或 `delete` 请求。
如果你需要索引一个数据流比如日志事件，它可以排队和索引数百或数千批次。  

&emsp;&emsp;`bulk` 与其他的请求体格式稍有不同，如下所示：   
```$xslt
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
```
&emsp;&emsp; 这种格式类似一个有效的单行 JSON 文档 流 ，它通过换行符(\n)连接到一起。注意两个要点：

 - 每行一定要以换行符(\n)结尾， 包括最后一行 。这些换行符被用作一个标记，可以有效分隔行。
 - 这些行不能包含未转义的换行符，因为他们将会对解析造成干扰。这意味着这个 JSON 不 能使用 pretty 参数打印。
 
>  Tip
>> 在 为什么是有趣的格式？ 中， 我们解释为什么 bulk API 使用这种格式。  

`action/metadata` 行指定 哪一个文档 做 什么操作 。     

 `action` 必须是以下选项之一:      
`create`   
&emsp;&emsp; 如果文档不存在，那么就创建它。详情请见 [创建新文档](07_creating_new_document.md)。     
`index`    
&emsp;&emsp; 创建一个新文档或者替换一个现有的文档。详情请见 [索引文档](03_indexing_document.md) 和 [更新整个文档](06_updating_whole_document.md)。   
`update`    
&emsp;&emsp; 部分更新一个文档。详情请见 [文档的部分更新](11_partial_updates_documents.md)。     
`delete`    
&emsp;&emsp; 删除一个文档。详情请见 删除文档。     
`metadata` 应该指定被索引、创建、更新或者删除的文档的 `_index` 、 `_type` 和 `_id` 。

&emsp;&emsp;例如，一个 `delete` 请求看起来是这样的：
```$xslt
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}
```
&emsp;&emsp;`request body` 行由文档的 `_source` 本身组成—​文档包含的字段和值。它是 `index` 和 `create` 操作所必需的，这是有道理的：你必须提供文档以索引。
它也是 `update` 操作所必需的，并且应该包含你传递给 `update API` 的相同请求体：  
 `doc` 、 `upsert` 、 `script` 等等。 删除操作不需要 `request body` 行。
```$xslt
{ "create":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
```
&emsp;&emsp;如果不指定 _id ，将会自动生成一个 ID ：  
```$xslt
{ "index": { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
```
&emsp;&emsp;为了把所有的操作组合在一起，一个完整的 bulk 请求 有以下形式:
```$xslt
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}  （a）
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} } (b)
```
`(a)请注意 delete 动作不能有请求体,它后面跟着的是另外一个操作。`

`(b)谨记最后一个换行符不要落下。`

&emsp;&emsp;这个 `Elasticsearch` 响应包含 `items `数组，
这个数组的内容是以请求的顺序列出来的每个请求的结果。
```$xslt
{
   "took": 4,
   "errors": false, (a)
   "items": [
      {  "delete": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 2,
            "status":   200,
            "found":    true
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 3,
            "status":   201
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "EiwfApScQiiy7TIKFxRCTw",
            "_version": 1,
            "status":   201
      }},
      {  "update": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 4,
            "status":   200
      }}
   ]
}
```
`(a)所有的子请求都成功完成。`   
&emsp;&emsp;每个子请求都是独立执行，因此某个子请求的失败不会对其他子请求的成功与否造成影响。
如果其中任何子请求失败，最顶层的 error 标志被设置为 true ，并且在相应的请求报告出错误明细：
```$xslt
POST /_bulk
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "Cannot create - it already exists" }
{ "index":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "But we can update it" }
```
&emsp;&emsp;在响应中，我们看到 create 文档 123 失败，因为它已经存在。
但是随后的 index 请求，也是对文档 123 操作，就成功了：
```$xslt
{
   "took": 3,
   "errors": true, (a)
   "items": [
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "status":   409, (b)
            "error":    "DocumentAlreadyExistsException (c)
                        [[website][4] [blog][123]:
                        document already exists]"
      }},
      {  "index": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 5,
            "status":   200 (d)
      }}
   ]
}
```
`(a)一个或者多个请求失败。`    
`(b) 这个请求的HTTP状态码报告为 409 CONFLICT 。`    
`(c)解释为什么请求失败的错误信息。`    
`(d)第二个请求成功，返回 HTTP 状态码 200 OK 。`   
&emsp;&emsp;这也意味着 bulk 请求不是原子的： 不能用它来实现事务控制。
每个请求是单独处理的，因此一个请求的成功或失败不会影响其他的请求。
## 不要重复指定Index和Type 
&emsp;&emsp;也许你正在批量索引日志数据到相同的 index 和 type 中。 
但为每一个文档指定相同的元数据是一种浪费。相反，
可以像 mget API 一样，在 bulk 请求的 URL 中接收默认的 /_index 或者 /_index/_type 
```$xslt
POST /website/_bulk
{ "index": { "_type": "log" }}
{ "event": "User logged in" }
```
&emsp;&emsp;你仍然可以覆盖元数据行中的 _index 和 _type , 
但是它将使用 URL 中的这些元数据值作为默认值：
```$xslt
POST /website/log/_bulk
{ "index": {}}
{ "event": "User logged in" }
{ "index": { "_type": "blog" }}
{ "title": "Overriding the default type" }
```
## 多大是太大？
&emsp;&emsp;整个批量请求都需要由接收到请求的节点加载到内存中，因此该请求越大，其他请求所能获得的内存就越少。 
批量请求的大小有一个最佳值，大于这个值，性能将不再提升，甚至会下降。 
但是最佳值不是一个固定的值。它完全取决于硬件、文档的大小和复杂度、索引和搜索的负载的整体情况。

&emsp;&emsp;幸运的是，很容易找到这个 最佳点 ：通过批量索引典型文档，并不断增加批量大小进行尝试。 
当性能开始下降，那么你的批量大小就太大了。一个好的办法是开始时将 1,000 到 5,000 个文档作为一个批次, 
如果你的文档非常大，那么就减少批量的文档个数。

&emsp;&emsp;密切关注你的批量请求的物理大小往往非常有用，一千个 1KB 的文档是完全不同于一千个 1MB 文档所占的物理大小。 
一个好的批量大小在开始处理后所占用的物理大小约为 5-15 MB。
