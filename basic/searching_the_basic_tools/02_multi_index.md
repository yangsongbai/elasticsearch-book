# [多索引](02_multi_index.md) 
&emsp;&emsp;搜索API的最基础的形式是没有指定任何查询的空搜索，它简单地返回集群中所有索引下的所有文档：
你有没有注意到之前的 `empty search` 的结果，不同类型的文档— `user` 和 `tweet` 来自不同的索引— `us` 和 `gb` ？

&emsp;&emsp;搜索API的最基础的形式是没有指定任何查询的空搜索，它简单地返回集群中所有索引下的所有文档：
如果不对某一特殊的索引或者类型做限制，就会搜索集群中的所有文档。Elasticsearch 转发搜索请求到每一个主分片或者副本分片，
汇集查询出的前10个结果，并且返回给我们。

然而，经常的情况下，你想在一个或多个特殊的索引并且在一个或者多个特殊的类型中进行搜索。
我们可以通过在URL中指定特殊的索引和类型达到这种效果，如下所示：

`/_search`     
&emsp;&emsp;在所有的索引中搜索所有的类型      
`/gb/_search`     
&emsp;&emsp;在 gb 索引中搜索所有的类型      
`/gb,us/_search`   
&emsp;&emsp;在 gb 和 us 索引中搜索所有的文档      
`/g*,u*/_search`    
&emsp;&emsp;在任何以 g 或者 u 开头的索引中搜索所有的类型    
`/gb/user/_search`   
&emsp;&emsp;在 gb 索引中搜索 user 类型       
`/gb,us/user,tweet/_search`   
&emsp;&emsp;在 gb 和 us 索引中搜索 user 和 tweet 类型    
`/_all/user,tweet/_search`    
&emsp;&emsp;在所有的索引中搜索 user 和 tweet 类型 
   
&emsp;&emsp;当在单一的索引下进行搜索的时候，Elasticsearch 转发请求到索引的每个分片中，可以是主分片也可以是副本分片，
然后从每个分片中收集结果。多索引搜索恰好也是用相同的方式工作的—​只是会涉及到更多的分片。

> 警示⚠️
>> 搜索一个索引有五个主分片和搜索五个索引各有一个分片准确来所说是等价的。
>> Elasticsearch6版本之后

&emsp;&emsp;接下来，你将明白这种简单的方式如何灵活的根据需求的变化让扩容变得简单。