# 删除一个索引   
用以下的请求来 删除索引:   
```
DELETE /my_index
```    
你也可以这样删除多个索引：   
```
DELETE /index_one,index_two
DELETE /index_*
```     
你甚至可以这样删除 全部 索引：  
```
DELETE /_all
DELETE /*
```     
> NOTE   
>> 对一些人来说，能够用单个命令来删除所有数据可能会导致可怕的后果。如果你想要避免意外的大量删除, 你可以在你的 elasticsearch.yml 做如下配置：
   
>> action.destructive_requires_name: true
   
>> 这个设置使删除只限于特定名称指向的数据, 而不允许通过指定 _all 或通配符来删除指定索引库。你同样可以通过 [Cluster State API](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_changing_settings_dynamically.html) 
>> 动态的更新这个设置。