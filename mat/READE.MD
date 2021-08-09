# 内存分析   
官网下载连接：https://www.eclipse.org/mat/downloads.php    

## 修改MAT的内存大小     

```
vi MemoryAnalyzer.ini
```
## jmap dump整个堆  
```
jmap -dump:format=b,file=[hprof文件] [进程ID]
```
## MAT分析 dump

```
 ./ParseHeapDump.sh [hprof文件]  
org.eclipse.mat.api:suspects org.eclipse.mat.api:overview org.eclipse.mat.api:top_components
```

分析报告
结果会生产如下三个zip文件，很小可以直接拷贝到本机  

jmap_Leak_Suspects.zip   
jmap_System_Overview.zip  
jmap_Top_Components.zip   

## 官方网站   
eclipse.org/mat/  