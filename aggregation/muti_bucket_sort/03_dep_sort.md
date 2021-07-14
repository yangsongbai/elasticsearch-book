# 基于“深度”度量排序  
在前面的示例中，度量是桶的直接子节点。平均售价是根据每个 term 来计算的。 在一定条件下，我们也有可能对 更深 的度量进行排序，比如孙子桶或从孙桶。

我们可以定义更深的路径，将度量用尖括号（ > ）嵌套起来，像这样： my_bucket>another_bucket>metric 。

需要提醒的是嵌套路径上的每个桶都必须是 单值 的。 filter 桶生成 一个单值桶：所有与过滤条件匹配的文档都在桶中。 多值桶（如：terms ）动态生成许多桶，无法通过指定一个确定路径来识别。

目前，只有三个单值桶： filter 、 global 和 reverse_nested 。让我们快速用示例说明，
创建一个汽车售价的直方图，但是按照红色和绿色（不包括蓝色）车各自的方差来排序：   
```
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : {
        "colors" : {
            "histogram" : {
              "field" : "price",
              "interval": 20000,
              "order": {
                "red_green_cars>stats.variance" : "asc" 
              }
            },
            "aggs": {
                "red_green_cars": {
                    "filter": { "terms": {"color": ["red", "green"]}}, 
                    "aggs": {
                        "stats": {"extended_stats": {"field" : "price"}} 
                    }
                }
            }
        }
    }
}
```
按照嵌套度量的方差对桶的直方图进行排序。   
因为我们使用单值过滤器 filter ，我们可以使用嵌套排序。  
按照生成的度量对统计结果进行排序。     

本例中，可以看到我们如何访问一个嵌套的度量。 stats 度量是 red_green_cars 聚合的子节点，
而 red_green_cars 又是 colors 聚合的子节点。 为了根据这个度量排序，我们定义了路径 red_green_cars>stats.variance 。
我们可以这么做，因为 filter 桶是个单值桶。