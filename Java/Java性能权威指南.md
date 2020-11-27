## 





### JVM配置



#### 配置1

> PrintNMTStatistics：退出时打印所有原生内存的分配详情
>
> ​	对OS来说，堆，栈空间，Native Memory都是原生内存

``` shell
-Xms1024m
-Xmx1024m
-Xmn384m
-XX:SurvivorRatio=6
-XX:MetaspaceSize=128m
-XX:ConcGCThreads=2
-XX:+PrintStringTableStatistics
-XX:+PrintGCDetails
-XX:+UnlockDiagnosticVMOptions
-XX:+PrintNMTStatistics
-XX:NativeMemoryTracking=detail
-Dfile.encoding=UTF-8
```



#### idea配置

> idea操作时需要新生代分配大一点

```shell
-Xms1200m
-Xmx1200m
-Xmn600m
-XX:MetaspaceSize=600m
-XX:MaxMetaspaceSize=600m
```

