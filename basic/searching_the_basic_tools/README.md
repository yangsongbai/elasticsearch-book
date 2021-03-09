# [搜索——最基本的工具](README.md)  
&emsp;&emsp;现在，我们已经学会了如何使用 Elasticsearch 作为一个简单的 NoSQL 风格的分布式文档存储系统。
我们可以将一个 JSON 文档扔到 Elasticsearch 里，然后根据 ID 检索。
但 Elasticsearch 真正强大之处在于可以从无规律的数据中找出有意义的信息——从“大数据”到“大信息”。

&emsp;&emsp;Elasticsearch 不只会_存储（stores）_ 文档，为了能被搜索到也会为文档添加_索引（indexes）_ ，
这也是为什么我们使用结构化的 JSON 文档，而不是无结构的二进制数据。

&emsp;&emsp;文档中的每个字段都将被索引并且可以被查询 。不仅如此，在简单查询时，Elasticsearch 可以使用 所有（all） 这些索引字段，
以惊人的速度返回结果。这是你永远不会考虑用传统数据库去做的一些事情。

搜索（search） 可以做到：

 - 在类似于 gender 或者 age 这样的字段上使用结构化查询，join_date 这样的字段上使用排序，就像SQL的结构化查询一样。    
 - 全文检索，找出所有匹配关键字的文档并按照_相关性（relevance）_ 排序后返回结果。    
 - 以上二者兼而有之。    

很多搜索都是开箱即用的，为了充分挖掘 Elasticsearch 的潜力，你需要理解以下三个概念：

*映射（Mapping）*     
&emsp;&emsp;描述数据在每个字段内如何存储    
*分析（Analysis）*     
&emsp;&emsp;全文是如何处理使之可以被搜索的      
*领域特定查询语言（Query DSL）*    
&emsp;&emsp;Elasticsearch 中强大灵活的查询语言     

&emsp;&emsp;以上提到的每个点都是一个大话题，我们将在 深入搜索 一章详细阐述它们。
本章节我们将介绍这三点的一些基本概念——仅仅帮助你大致了解搜索是如何工作的。

&emsp;&emsp;我们将使用最简单的形式开始介绍 search API。
>测试数据
>>&emsp;&emsp;本章节的测试数据可以在这里找到： https://gist.github.com/clintongormley/8579281 。    
  
>>&emsp;&emsp;你可以把这些命令复制到终端中执行来实践本章的例子。   
