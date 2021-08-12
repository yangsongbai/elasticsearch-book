# 构建源码   

./gradlew idea   

进去项目根目录     
```
./gradlew assemble   
# 去除snaphot
./gradlew assemble -Dbuild.snapshot=false  
```
由根目录buildd.gradle文件内容可知 安装包在 ./distribution/package目录下   
https://blog.csdn.net/u013066244/article/details/73927756      
https://www.iteye.com/blog/fishboy-2391750     
https://segmentfault.com/a/1190000017831398    