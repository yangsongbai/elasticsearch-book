# 索引设置   
你可以通过修改配置来自定义索引行为，详细配置参照 [索引模块](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/index-modules.html)       
> Tip 
>> Elasticsearch 提供了优化好的默认配置。 除非你理解这些配置的作用并且知道为什么要去修改，否则不要随意修改。    

下面是两个 最重要的设置：

 - number_of_shards    
每个索引的主分片数，默认值是 5 。这个配置在索引创建后不能修改。    
 - number_of_replicas    
每个主分片的副本数，默认值是 1 。对于活动的索引库，这个配置可以随时修改。    
例如，我们可以创建只有 一个主分片，没有副本的小索引：     
```
PUT /my_temp_index
{
    "settings": {
        "number_of_shards" :   1,
        "number_of_replicas" : 0
    }
}
```
然后，我们可以用 update-index-settings API 动态修改副本数：    
```
PUT /my_temp_index/_settings
{
    "number_of_replicas": 1
}
``` 

