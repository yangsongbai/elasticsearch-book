# 和Elasticsearch交互
&emsp;&emsp;和 Elasticsearch 的交互方式取决于你是否使用 Java。  

## Java Api
&emsp;&emsp;如果你正在使用 Java，在代码中你可以使用 Elasticsearch 内置的两个客户端：   

**节点客户端（Node client）**     
&emsp;&emsp;节点客户端作为一个非数据节点加入到本地集群中。换句话说，它本身不保存任何数据，但是它知道数据在集群中的哪个节点中，并且可以把请求转发到正确的节点。

**传输客户端（Transport client）**    
&emsp;&emsp;轻量级的传输客户端可以将请求发送到远程集群。它本身不加入集群，但是它可以将请求转发到集群中的一个节点上。   

&emsp;&emsp;两个 Java 客户端都是通过 9300 端口并使用 Elasticsearch 的原生 传输 协议和集群交互。集群中的节点通过端口 9300 彼此通信。如果这个端口没有打开，节点将无法形成一个集群。

>建议
>> Java 客户端作为节点必须和 Elasticsearch 有相同的 主要 版本；否则，它们之间将无法互相理解。

&emsp;&emsp;更多的 Java 客户端信息可以在 [Elasticsearch Clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html) 中找到。

## Restful API with Json over HTTP 
&emsp;&emsp;所有其他语言可以使用 RESTful API 通过端口 9200 和 Elasticsearch 进行通信，你可以用你最喜爱的 web 客户端访问 Elasticsearch 。
事实上，正如你所看到的，你甚至可以使用 curl 命令来和 Elasticsearch 交互。
>注意
>> Elasticsearch 为以下语言提供了官方客户端--Groovy、JavaScript、.NET、 PHP、 Perl、 Python 和 Ruby—​还有很多社区提供的客户端和插件，
所有这些都可以在 [Elasticsearch Clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html)
 中找到。

&emsp;&emsp;一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：  
```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -H 'Content-Type: application/json'  -d '<BODY>'
```
被 < > 标记的部件：  

|  变量   | 备注  |
|  ----  | ----  |
| VERB  | 适当的 HTTP 方法 或 谓词 : GET、 POST、 PUT、 HEAD 或者 DELETE。 |
| PROTOCOL  | http 或者 https（如果你在 Elasticsearch 前面有一个 https 代理） |
| HOST  | Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。 |
| PORT  | 运行 Elasticsearch HTTP 服务的端口号，默认是 9200 。 |
| PATH  | API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：_cluster/stats 和 _nodes/stats/jvm 。 |
| QUERY_STRING  | 任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读) |
| BODY  | 一个 JSON 格式的请求体 (如果请求需要的话) |

&emsp;&emsp;例如，计算集群中文档的数量，我们可以用这个:

```$xslt
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'
```
&emsp;&emsp;Elasticsearch 返回一个 HTTP 状态码（例如：200 OK）和（除`HEAD`请求）一个 JSON 格式的返回值。
前面的 curl 请求将返回一个像下面一样的 JSON 体：  
```$xslt
{
    "count" : 0,
    "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
    }
}
```
&emsp;&emsp;在返回结果中没有看到 HTTP 头信息是因为我们没有要求`curl`显示它们。想要看到头信息，需要结合 -i 参数来使用 curl 命令：   
```$xslt
curl -i -XGET 'localhost:9200/'
```
&emsp;&emsp;在书中剩余的部分，我们将用缩写格式来展示这些 curl 示例，
所谓的缩写格式就是省略请求中所有相同的部分，例如主机名、端口号以及 curl 命令本身。而不是像下面显示的那样用一个完整的请求：
```$xslt
curl -XGET 'localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}'
```
&emsp;&emsp;我们将用缩写格式显示：  
```$xslt
GET /_count
{
    "query": {
        "match_all": {}
    }
}
```
&emsp;&emsp;事实上， kibana DevTool 控制台 也使用这样相同的格式。











