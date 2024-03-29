# [添加故障转移](04_add_failover.md)
&emsp;&emsp;当集群中只有一个节点在运行时，意味着会有一个单点故障问题——没有冗余。
幸运的是，我们只需再启动一个节点即可防止数据丢失。    
>启动第二个节点
>>&emsp;&emsp;为了测试第二个节点启动后的情况，你可以在同一个目录内，完全依照启动第一个节点的方式来启动一个新节点
（参考安装并运行 Elasticsearch）。多个节点可以共享同一个目录。

>>&emsp;&emsp;当你在同一台机器上启动了第二个节点时，只要它和第一个节点有同样的 cluster.name 配置，
它就会自动发现集群并加入到其中。 但是在不同机器上启动节点的时候，为了加入到同一集群，
你需要配置一个可连接到的单播主机列表。 详细信息请查看最好使用单播代替组播

&emsp;&emsp;如果启动了第二个节点，我们的集群将会如Figure 3, “拥有两个节点的集群——所有主分片和副本分片都已被分配”所示。  

> Figure 3. 拥有两个节点的集群——所有主分片和副本分片都已被分配

<img src="./images/03_two_node_cluster.png" width = "800" height = "200" 
alt="拥有两个节点的集群——所有主分片和副本分片都已被分配" align=center />

&emsp;&emsp;当第二个节点加入到集群后，3个 副本分片 将会分配到这个节点上——每个主分片对应一个副本分片。 
这意味着当集群内任何一个节点出现问题时，我们的数据都完好无损。

&emsp;&emsp;所有新近被索引的文档都将会保存在主分片上，然后被并行的复制到对应的副本分片上。
这就保证了我们既可以从主分片又可以从副本分片上获得文档。

&emsp;&emsp;*cluster-health* 现在展示的状态为 *green* ，这表示所有6个分片（包括3个主分片和3个副本分片）都在正常运行。

```$xslt
{
  "cluster_name": "elasticsearch",
  "status": "green",  (a)
  "timed_out": false,
  "number_of_nodes": 2,
  "number_of_data_nodes": 2,
  "active_primary_shards": 3,
  "active_shards": 6,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100
}
```
&emsp;&emsp;*(a)集群 status 值为 green* 。

&emsp;&emsp;我们的集群现在不仅仅是正常运行的，并且还处于 始终可用 的状态。