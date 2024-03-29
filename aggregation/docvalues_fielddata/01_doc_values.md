# Doc Values   
聚合使用一个叫 doc values 的数据结构（在 [Doc Values 介绍](https://www.elastic.co/guide/cn/elasticsearch/guide/current/docvalues-intro.html) 
里简单介绍）。 Doc values 可以使聚合更快、更高效并且内存友好，所以理解它的工作方式十分有益。

Doc values 的存在是因为倒排索引只对某些操作是高效的。 倒排索引的优势 在于查找包含某个项的文档，
而对于从另外一个方向的相反操作并不高效，即：确定哪些项是否存在单个文档里，聚合需要这种次级的访问模式。

对于以下倒排索引：    
|  Term   | Doc_1  |  Doc_2   | Doc_3 | 
|  ----  | ----  |  ----  | ----  |
| brown    |   X  |   X    |   |
| dog      |   X  |        |  x |
| dogs     |      |   X    |  x |
| fox      |   X  |        |  x |
| foxes    |      |   X    |   |
| in       |      |   X    |   |
| jumped   |   X  |        | x  |
| lazy     |   X  |   X    |   |
| leap     |      |   X    |   |
| over     |   X  |   X    |  x |
| quick    |   X  |   X    |  x |
| summer   |      |   X    |   |
| the      |   X  |        |  x |

如果我们想要获得所有包含 brown 的文档的词的完整列表，我们会创建如下查询：    
```
GET /my_index/_search
{
  "query" : {
    "match" : {
      "body" : "brown"
    }
  },
  "aggs" : {
    "popular_terms": {
      "terms" : {
        "field" : "body"
      }
    }
  }
}
```     
查询部分简单又高效。倒排索引是根据项来排序的，所以我们首先在词项列表中找到 brown ，然后扫描所有列，找到包含 brown 的文档。
我们可以快速看到 Doc_1 和 Doc_2 包含 brown 这个 token。

然后，对于聚合部分，我们需要找到 Doc_1 和 Doc_2 里所有唯一的词项。 
用倒排索引做这件事情代价很高： 我们会迭代索引里的每个词项并收集 Doc_1 和 Doc_2 列里面 token。
这很慢而且难以扩展：随着词项和文档的数量增加，执行时间也会增加。

Doc values 通过转置两者间的关系来解决这个问题。倒排索引将词项映射到包含它们的文档，doc values 将文档映射到它们包含的词项：     
|  Doc    | Terms  |
|  ----  | ----  |  
| Doc_1     |  brown, dog, fox, jumped, lazy, over, quick, the | 
| Doc_2     |  brown, dogs, foxes, in, lazy, leap, over, quick, summer | 
| Doc_3     |  dog, dogs, fox, jumped, over, quick, the |   

当数据被转置之后，想要收集到 Doc_1 和 Doc_2 的唯一 token 会非常容易。获得每个文档行，获取所有的词项，然后求两个集合的并集。

因此，搜索和聚合是相互紧密缠绕的。搜索使用倒排索引查找文档，聚合操作收集和聚合 doc values 里的数据。   
> 注意  
>> Doc values 不仅可以用于聚合。 任何需要查找某个文档包含的值的操作都必须使用它。 
>> 除了聚合，还包括排序，访问字段值的脚本，父子关系处理（参见 [父-子关系文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/parent-child.html) ）。
