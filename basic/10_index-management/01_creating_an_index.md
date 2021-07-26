# 创建一个索引   
到目前为止, 我们已经通过索引一篇文档创建了一个新的索引 。这个索引采用的是默认的配置，
新的字段通过动态映射的方式被添加到类型映射。现在我们需要对这个建立索引的过程做更多的控制：
我们想要确保这个索引有数量适中的主分片，并且在我们索引任何数据 之前 ，分析器和映射已经被建立好。

为了达到这个目的，我们需要手动创建索引，在请求体里面传入设置或类型映射，如下所示：   
```
PUT /my_index
{
    "settings": { ... any settings ... },
    "mappings": {
        "type_one": { ... any mappings ... },
        "type_two": { ... any mappings ... },
        ...
    }
}
```
如果你想禁止自动创建索引，你 可以通过在 config/elasticsearch.yml 的每个节点下添加下面的配置：    
```
action.auto_create_index: false
```    
> NOTE 
>> 我们会在之后讨论你怎么用 [索引模板](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index-templates.html) 
>> 来预配置开启自动创建索引。
>> 这在索引日志数据的时候尤其有用：你将日志数据索引在一个以日期结尾命名的索引上，
>> 子夜时分，一个预配置的新索引将会自动进行创建。
