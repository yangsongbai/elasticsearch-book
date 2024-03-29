#  过滤集提升权重  
回到 忽略 TF/IDF 里处理过的问题，我们希望根据每个度假屋的特性数量来评分，
当时我们希望能用缓存的过滤器来影响评分，现在 function_score 查询正好可以完成这件事情。      

到目前为止，我们展现的都是为所有文档应用单个函数的使用方式，
现在会用过滤器将结果划分为多个子集（每个特性一个过滤器），并为每个子集使用不同的函数。    

在下面例子中，我们会使用 weight 函数，它与 boost 参数类似可以用于任何查询。
有一点区别是 weight 没有被 Luence 归一化成难以理解的浮点数，而是直接被应用。   

查询的结构需要做相应变更以整合多个函数：  
```
GET /_search
{
  "query": {
    "function_score": {
      "filter": {               (1)
        "term": { "city": "Barcelona" }
      },
      "functions": [           (2)
        {
          "filter": { "term": { "features": "wifi" }},     (3)
          "weight": 1
        },
        {
          "filter": { "term": { "features": "garden" }},    (3)
          "weight": 1
        },
        {
          "filter": { "term": { "features": "pool" }},      (3)
          "weight": 2                                       (4)
        }
      ],
      "score_mode": "sum",                                 (5)
    }
  }
}
```   
(1) function_score 查询有个 filter 过滤器而不是 query 查询。     
(2) functions 关键字存储着一个将被应用的函数列表。     
(3) 函数会被应用于和 filter 过滤器（可选的）匹配的文档。     
(4) pool 比其他特性更重要，所以它有更高 weight 。   
(5) score_mode 指定各个函数的值进行组合运算的方式。       

这个新特性需要注意的地方会在以下小节介绍。      

## 过滤 VS 查询   
首先要注意的是 filter 过滤器代替了 query 查询，在本例中，我们无须使用全文搜索，只想找到 city 字段中包含 Barcelona 的所有文档，
逻辑用过滤比用查询表达更清晰。过滤器返回的所有文档的评分 _score 的值为 1 。 
function_score 查询接受 query 或 filter ，如果没有特别指定，则默认使用 match_all 查询。    

## 函数functions   
functions 关键字保持着一个将要被使用的函数列表。
可以为列表里的每个函数都指定一个 filter 过滤器，在这种情况下，函数只会被应用到那些与过滤器匹配的文档，
例子中，我们为与过滤器匹配的文档指定权重值 weight 为 1 （为与 pool 匹配的文档指定权重值为 2 ）。        

## 评分模式score_mode    
每个函数返回一个结果，所以需要一种将多个结果缩减到单个值的方式，然后才能将其与原始评分 _score 合并。
评分模式 score_mode 参数正好扮演这样的角色，它接受以下值：
 - multiply    
   函数结果求积（默认）。
 - sum   
函数结果求和。    
 - avg     
函数结果的平均值。    
 - max     
函数结果的最大值。     
 - min     
函数结果的最小值。      
 - first        
使用首个函数（可以有过滤器，也可能没有）的结果作为最终结果     

在本例中，我们将每个过滤器匹配结果的权重 weight 求和，并将其作为最终评分结果，所以会使用 sum 评分模式。       

不与任何过滤器匹配的文档会保有其原始评分， _score 值的为 1 。        


