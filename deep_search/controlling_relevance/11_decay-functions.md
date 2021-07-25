# 越近越好   
很多变量都可以影响用户对于度假屋的选择，也许用户希望离市中心近点，但如果价格足够便宜，
也有可能选择一个更远的住处，也有可能反过来是正确的：愿意为最好的位置付更多的价钱。    

如果我们添加过滤器排除所有市中心方圆 1 千米以外的度假屋，或排除所有每晚价格超过 £100 英镑的，
我们可能会将用户愿意考虑妥协的那些选择排除在外。    

function_score 查询会提供一组 衰减函数（decay functions） ，
让我们有能力在两个滑动标准，如地点和价格，之间权衡。    

有三种衰减函数—— linear 、 exp 和 gauss （线性、指数和高斯函数），
它们可以操作数值、时间以及经纬度地理坐标点这样的字段。所有三个函数都能接受以下参数：    

 - origin    
中心点 或字段可能的最佳值，落在原点 origin 上的文档评分 _score 为满分 1.0 。   
 - scale    
衰减率，即一个文档从原点 origin 下落时，评分 _score 改变的速度。（例如，每 £10 欧元或每 100 米）。
 - decay      
从原点 origin 衰减到 scale 所得的评分 _score ，默认值为 0.5 。      
 - offset           
以原点 origin 为中心点，为其设置一个非零的偏移量 offset 覆盖一个范围，而不只是单个原点。
在范围 -offset <= origin <= +offset 内的所有评分 _score 都是 1.0 。

这三个函数的唯一区别就是它们衰减曲线的形状，用图来说明会更为直观（参见 Figure 33, “衰减函数曲线” ）。     

<img src="./images/figure_33.png" width = "800" height = "200" 
alt="衰减函数曲线" align=center />    

图 Figure 33, “衰减函数曲线” 中所有曲线的原点 origin （即中心点）的值都是 40 ， offset 是 5 ，
也就是在范围 40 - 5 <= value <= 40 + 5 内的所有值都会被当作原点 origin 处理——所有这些点的评分都是满分 1.0 。

在此范围之外，评分开始衰减，衰减率由 scale 值（此例中的值为 5 ）和 衰减值 decay （此例中为默认值 0.5 ）共同决定。
结果是所有三个曲线在 origin +/- (offset + scale) 处的评分都是 0.5 ，即点 30 和 50 处。

linear 、 exp 和 gauss （线性、指数和高斯）函数三者之间的区别在于范围（ origin +/- (offset + scale) ）之外的曲线形状：

 - linear 线性函数是条直线，一旦直线与横轴 0 相交，所有其他值的评分都是 0.0 。   
 - exp 指数函数是先剧烈衰减然后变缓。   
 - gauss 高斯函数是钟形的——它的衰减速率是先缓慢，然后变快，最后又放缓。  
 
选择曲线的依据完全由期望评分 _score 的衰减速率来决定，即距原点 origin 的值。   

回到我们的例子：用户希望租一个离伦敦市中心近（ { "lat": 51.50, "lon": 0.12} ）且每晚不超过 £100 英镑的度假屋，而且与距离相比，
我们的用户对价格更为敏感，这样查询可以写成：  
```
GET /_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "gauss": {
            "location": {     (1)
              "origin": { "lat": 51.5, "lon": 0.12 },
              "offset": "2km",
              "scale":  "3km"
            }
          }
        },
        {
          "gauss": {
            "price": {       (2)
              "origin": "50",    (3)
              "offset": "50",
              "scale":  "20"
            }
          },
          "weight": 2     (4)   
        }
      ]
    }
  }
}
```  
(1) location 字段以地理坐标点 geo_point 映射。
(2)  price 字段是数值。      
(3) 参见 理解价格语句 ，理解 origin 为什么是 50 而不是 100 。    
(4) price 语句是 location 语句权重的两倍。      

location 语句可以简单理解为：  
 - 以伦敦市中作为原点 origin 。      
 - 所有距原点 origin 2km 范围内的位置的评分是 1.0 。   
 - 距中心 5km （ offset + scale ）的位置的评分是 0.5 。   


