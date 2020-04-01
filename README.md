- ### 前言：

> 本文内容是基于 JDK 8

- ### 背景描述:

--- 

&nbsp;&nbsp;因发现公司的JVM参数配置的不合理，各项目之间JVM参数配置不统一，并且CMS GC触发时间，是交由JVM动态调控的，造成遇到GC和JVM相关问题的时候，排查较为困难，于是基于JDK8整理出一套通用的JVM参数配置。

---

- ### 注意事项

---

1. 新生代垃圾回收器是采用ParNew（标记复制算法）

2. 老年代的垃圾回收器是采用CMS（标记清除算法）

3. 备用老年代垃圾回收器（Concurrent Mode Failure)采用Serial Old（标记整理算法）

---

- ### JVM参数

``` 

设置初始内存和最大内存为5120m(具体大小可根据项目自行调节)，设置为统一，避免扩容造成的性能损失

-Xms5120m 

-Xmx5120m

设置新生代大小为1706m 推荐新生代大小：老年代大小比例为 1:2，新生代占整堆大小的1/3

-Xmn1706m

设置线程栈为512k

-Xss512k

设置元空间初始内存和最大内存为512m

-XX:MaxMetaspaceSize=512m 
-XX:MetaspaceSize=512m

指定CMS垃圾回收器作为老年代回收器

-XX:+UseConcMarkSweepGC

使用Parnew垃圾回收器作为新生代回收器

-XX:+UseParNewGC

关闭JVM自动调控的垃圾回收，不开启此参数，后续很多配置参数无意义

-XX:+UseCMSInitiatingOccupancyOnly

// FullGC之后开启压缩，配套使用CMSFullGCsBeforeCompaction的值作为FullGC几次之后 压缩一次，值为0的话就默认每次FullGC压缩一次，注意CMS GC不是Full GC，可自行百度CMS 的 foreground GC和 background GC的区别）

-XX:+UseCMSCompactAtFullCollection

-XX:CMSFullGCsBeforeCompaction=0

在老年代内存达到70%的时候，进行CMS垃圾回收

-XX:CMSInitiatingOccupancyFraction=70

CMS垃圾回收时卸载无用的class类

-XX:+CMSClassUnloadingEnabled

在做System.gc()时会做background模式CMS GC。主要因为用NIO/Netty框架的时候，会直接申请堆外内存，很多框架底层，为了释放mmap分配的空间，会调用System.gc()来回收

-XX:+ExplicitGCInvokesConcurrent

以下为打印GC日志和OOM时dump的参数

-XX:+HeapDumpOnOutOfMemoryError

-XX:HeapDumpPath=xxxx/heapdump.hprof

-XX:+PrintGC

-XX:+PrintGCDateStamps

-XX:+PrintGCDetails

-XX:+PrintGCTimeStamps

可选参数

在CMS remark之前做一次ygc

-XX:CMSScavengeBeforeRemark

并行处理Reference

-XX:+ParallelRefProcEnabled

排查GC时间过长的可选参数

-XX:+PrintStringTableStatistics

-XX:+PrintReferenceGC

-XX:+PrintHeapAtGC

-XX:+PrintGCApplicationStoppedTime

-XX:+PrintSafepointStatistics

-XX:PrintSafepointStatisticsCount=1
```
![alt](126.jpg)
