# [乐观并发控制](10_optimistic_concurrency_control.md)   
&emsp;&emsp;Elasticsearch 是分布式的。当文档创建、更新或删除时， 新版本的文档必须复制到集群中的其他节点。
Elasticsearch 也是异步和并发的，这意味着这些复制请求被并行发送，并且到达目的地时也许 顺序是乱的 。
 Elasticsearch 需要一种方法确保文档的旧版本不会覆盖新的版本。

&emsp;&emsp;当我们之前讨论 `index` ， `GET` 和 `delete` 请求时，我们指出每个文档都有一个 _version （版本）号，
当文档被修改时版本号递增。 Elasticsearch 使用这个 _version 号来确保变更以正确顺序得到执行。
如果旧版本的文档在新版本之后到达，它可以被简单的忽略。

&emsp;&emsp;我们可以利用 `_version` 号来确保 应用中相互冲突的变更不会导致数据丢失。我们通过指定想要修改文档的 `version` 号来达到这个目的。
如果该版本不是当前版本号，我们的请求将会失败。

&emsp;&emsp;让我们创建一个新的博客文章：
```$xslt
PUT /website/blog/1/_create
{
  "title": "My first blog entry",
  "text":  "Just trying this out..."
}
```
&emsp;&emsp;响应体告诉我们，这个新创建的文档 _version 版本号是 1 。
现在假设我们想编辑这个文档：我们加载其数据到 web 表单中，
 做一些修改，然后保存新的版本。

&emsp;&emsp;首先我们检索文档:
```$xslt
GET /website/blog/1
```
&emsp;&emsp;响应体包含相同的 _version 版本号 1 ：  
```$xslt
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out..."
  }
}
```
&emsp;&emsp;现在，当我们尝试通过重建文档的索引来保存修改，我们指定 version 为我们的修改会被应用的版本：
```$xslt
PUT /website/blog/1?version=1  (a)
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```
`(a)我们想这个在我们索引中的文档只有现在的 _version 为 1 时，本次更新才能成功。`

&emsp;&emsp;此请求成功，并且响应体告诉我们 `_version` 已经递增到 2 ：
```$xslt
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "1",
  "_version": 2
  "created":  false
}
```
&emsp;&emsp;然而，如果我们重新运行相同的索引请求，仍然指定 `version=1` ， 
Elasticsearch 返回 `409 Conflict HTTP` 响应码，和一个如下所示的响应体：    
```$xslt
{
   "error": {
      "root_cause": [
         {
            "type": "version_conflict_engine_exception",
            "reason": "[blog][1]: version conflict, current [2], provided [1]",
            "index": "website",
            "shard": "3"
         }
      ],
      "type": "version_conflict_engine_exception",
      "reason": "[blog][1]: version conflict, current [2], provided [1]",
      "index": "website",
      "shard": "3"
   },
   "status": 409
}
```
&emsp;&emsp;这告诉我们在 Elasticsearch 中这个文档的当前 `_version` 号是 `2` ，但我们指定的更新版本号为 `1 `。

&emsp;&emsp;我们现在怎么做取决于我们的应用需求。我们可以告诉用户说其他人已经修改了文档，并且在再次保存之前检查这些修改内容。 
或者，在之前的商品 `stock_count` 场景，我们可以获取到最新的文档并尝试重新应用这些修改。

&emsp;&emsp;所有文档的更新或删除 `API`，都可以接受 `version` 参数，这允许你在代码中使用乐观的并发控制，这是一种明智的做法。

# 通过外部系统使用版本控制  
&emsp;&emsp;一个常见的设置是使用其它数据库作为主要的数据存储，使用 `Elasticsearch` 做数据检索， 
这意味着主数据库的所有更改发生时都需要被复制到 `Elasticsearch` ，如果多个进程负责这一数据同步，
你可能遇到类似于之前描述的并发问题。

&emsp;&emsp;如果你的主数据库已经有了版本号 — 或一个能作为版本号的字段值比如 `timestamp `— 那么你就可以在 `Elasticsearch `
中通过增加 `version_type=external` 到查询字符串的方式重用这些相同的版本号， 
版本号必须是大于零的整数， 且小于 9.2E+18 — 一个 Java 中 long 类型的正值。

&emsp;&emsp;外部版本号的处理方式和我们之前讨论的内部版本号的处理方式有些不同， 
Elasticsearch 不是检查当前 `_version` 和请求中指定的版本号是否相同， 
而是检查当前 `_version` 是否 小于 指定的版本号。 如果请求成功，外部的版本号作为文档的新 `_version` 进行存储。

&emsp;&emsp;外部版本号不仅在索引和删除请求是可以指定，而且在 创建 新文档时也可以指定。

&emsp;&emsp;例如，要创建一个新的具有外部版本号 5 的博客文章，我们可以按以下方法进行： 
```$xslt
PUT /website/blog/2?version=5&version_type=external
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
```
&emsp;&emsp;在响应中，我们能看到当前的 `_version` 版本号是 5 ：
```$xslt
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 5,
  "created":  true
}
```
&emsp;&emsp;现在我们更新这个文档，指定一个新的 version 号是 10 ：
```$xslt
PUT /website/blog/2?version=10&version_type=external
{
  "title": "My first external blog entry",
  "text":  "This is a piece of cake..."
}
```
&emsp;&emsp;请求成功并将当前 _version 设为 10 ：
```$xslt
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 10,
  "created":  false
}
```
&emsp;&emsp;如果你要重新运行此请求时，它将会失败，并返回像我们之前看到的同样的冲突错误， 
因为指定的外部版本号不大于 Elasticsearch 的当前版本号。

