# [集群健康](02_cluster_health.md)  
&emsp;&emsp;Elasticsearch 的集群监控信息中包含了许多的统计数据，其中最为重要的一项就是 集群健康 ， 
它在 status 字段中展示为 green 、 yellow 或者 red 。
```$xslt
GET /_cluster/health
```
&emsp;&emsp;在一个不包含任何索引的空集群中，它将会有一个类似于如下所示的返回内容：
```$xslt
{
  "cluster_name": "elasticsearch",
  "status": "yellow", (a)
  "timed_out": false,
  "number_of_nodes": 10,
  "number_of_data_nodes": 9,
  "active_primary_shards": 6921,
  "active_shards": 13661,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 181,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 98.6923854934258
}
```
&emsp;&emsp;*(a)status 字段是我们最关心的。*     
&emsp;&emsp;*status* 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：

**green**    
&emsp;&emsp;所有的主分片和副本分片都正常运行。    
**yellow**    
&emsp;&emsp;所有的主分片都正常运行，但不是所有的副本分片都正常运行。    
**red**    
&emsp;&emsp;有主分片没能正常运行。        

&emsp;&emsp;在本章节剩余的部分，我们将解释什么是 *主* 分片和 *副本* 分片，以及上面提到的这些颜色的实际意义。