# 优化聚合查询 
“elasticsearch 里面桶的叫法和 SQL 里面分组的概念是类似的，一个桶就类似 SQL 里面的一个 group，
多级嵌套的 aggregation， 类似 SQL 里面的多字段分组（group by field1,field2, …​..），
注意这里仅仅是概念类似，底层的实现原理是不一样的。 －译者注”     
  
terms 桶基于我们的数据动态构建桶；它并不知道到底生成了多少桶。 
大多数时候对单个字段的聚合查询还是非常快的， 但是当需要同时聚合多个字段时，
就可能会产生大量的分组，最终结果就是占用 es 大量内存，从而导致 OOM 的情况发生。       

假设我们现在有一些关于电影的数据集，每条数据里面会有一个数组类型的字段存储表演该电影的所有演员的名字。   
```
{
  "actors" : [
    "Fred Jones",
    "Mary Jane",
    "Elizabeth Worthing"
  ]
}
```    
如果我们想要查询出演影片最多的十个演员以及与他们合作最多的演员，使用聚合是非常简单的：    
```
{
  "aggs" : {
    "actors" : {
      "terms" : {
         "field" : "actors",
         "size" :  10
      },
      "aggs" : {
        "costars" : {
          "terms" : {
            "field" : "actors",
            "size" :  5
          }
        }
      }
    }
  }
}
```  
这会返回前十位出演最多的演员，以及与他们合作最多的五位演员。这看起来是一个简单的聚合查询，最终只返回 50 条数据！

但是， 这个看上去简单的查询可以轻而易举地消耗大量内存，我们可以通过在内存中构建一个树来查看这个 terms 聚合。 
actors 聚合会构建树的第一层，每个演员都有一个桶。然后，内套在第一层的每个节点之下， 
costar 聚合会构建第二层，每个联合出演一个桶，请参见 Figure 42, “Build full depth tree” 所示。
这意味着每部影片会生成 n2 个桶！
<img src="./images/figure_42.png" width = "800" height = "200" 
alt="Figure 42，Build full depth tree" align=center />     

用真实点的数据，设想平均每部影片有 10 名演员，每部影片就会生成 102 == 100 个桶。如果总共有 20，000 部影片，
粗率计算就会生成 2，000，000 个桶。

现在，记住，聚合只是简单的希望得到前十位演员和与他们联合出演者，总共 50 条数据。
为了得到最终的结果，我们创建了一个有 2，000，000 桶的树，
然后对其排序，取 top10。 图 Figure 43, “Sort tree” 和图 Figure 44, “Prune tree” 对这个过程进行了阐述。   
<img src="./images/figure_43.png" width = "800" height = "200" 
alt="Sort tree" align=center />     

<img src="./images/figure_44.png" width = "800" height = "200" 
alt="Sort tree" align=center />     

这时我们一定非常抓狂，在 2 万条数据下执行任何聚合查询都是毫无压力的。
如果我们有 2 亿文档，想要得到前 100 位演员以及与他们合作最多的 20 位演员，
作为查询的最终结果会出现什么情况呢？

可以推测聚合出来的分组数非常大，会使这种策略难以维持。
世界上并不存在足够的内存来支持这种不受控制的聚合查询。   

## 深度优先与广度优先   
Elasticsearch 允许我们改变聚合的 集合模式 ，就是为了应对这种状况。 我们之前展示的策略叫做 深度优先 ，
它是默认设置， 先构建完整的树，然后修剪无用节点。 深度优先 的方式对于大多数聚合都能正常工作，
但对于如我们演员和联合演员这样例子的情形就不太适用。

为了应对这些特殊的应用场景，我们应该使用另一种集合策略叫做 广度优先 。
这种策略的工作方式有些不同，它先执行第一层聚合， 再 继续下一层聚合之前会先做修剪。 
图 Figure 45, “Build first level” 和图 Figure 47, “Prune first level” 对这个过程进行了阐述。   

在我们的示例中， actors 聚合会首先执行，在这个时候，我们的树只有一层，但我们已经知道了前 10 位的演员！
这就没有必要保留其他的演员信息，因为它们无论如何都不会出现在前十位中。   
<img src="./images/figure_45.png" width = "800" height = "200" 
alt="Build first level" align=center />    

<img src="./images/figure_46.png" width = "800" height = "200" 
alt="Build first level" align=center /> 

<img src="./images/figure_47.png" width = "800" height = "200" 
alt="Prune first level" align=center />     

因为我们已经知道了前十名演员，我们可以安全的修剪其他节点。
修剪后，下一层是基于 它的 执行模式读入的，重复执行这个过程直到聚合完成，
如图 Figure 48, “Populate full depth for remaining nodes” 所示。 
这种场景下，广度优先可以大幅度节省内存。     
<img src="./images/figure_48.png" width = "800" height = "200" 
alt="Populate full depth for remaining nodes" align=center />   

要使用广度优先，只需简单 的通过参数 collect 开启：   
```
{
  "aggs" : {
    "actors" : {
      "terms" : {
         "field" :        "actors",
         "size" :         10,
         "collect_mode" : "breadth_first" 
      },
      "aggs" : {
        "costars" : {
          "terms" : {
            "field" : "actors",
            "size" :  5
          }
        }
      }
    }
  }
}
```    
按聚合来开启 breadth_first 。   

广度优先仅仅适用于每个组的聚合数量远远小于当前总组数的情况下，
因为广度优先会在内存中缓存裁剪后的仅仅需要缓存的每个组的所有数据，
以便于它的子聚合分组查询可以复用上级聚合的数据。

广度优先的内存使用情况与裁剪后的缓存分组数据量是成线性的。对于很多聚合来说，每个桶内的文档数量是相当大的。 
想象一种按月分组的直方图，总组数肯定是固定的，因为每年只有12个月，这个时候每个月下的数据量可能非常大。
这使广度优先不是一个好的选择，这也是为什么深度优先作为默认策略的原因。     

针对上面演员的例子，如果数据量越大，那么默认的使用深度优先的聚合模式生成的总分组数就会非常多，
但是预估二级的聚合字段分组后的数据量相比总的分组数会小很多所以这种情况下使用广度优先的模式能大大节省内存，
从而通过优化聚合模式来大大提高了在某些特定场景下聚合查询的成功率。    

