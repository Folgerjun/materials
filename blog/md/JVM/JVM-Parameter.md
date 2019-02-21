---
title: JVM 常用调优参数
date: 2019-2-21 14:20:12
categories: [开发,JVM]
tags: [Java,JVM]
---

> 记录下 JVM 常用的一些调优参数。


```
// 常见参数
-Xms1024m 初始堆大小 
-Xmx1024m 最大堆大小  一般将 Xms 和 Xmx 设置为相同大小，防止堆扩展，影响性能。
-XX:NewSize=n:设置年轻代大小 
-XX:NewRatio=n:设置年轻代和年老代的比值.如:为 3,表示年轻代与年老代比值为 1:3,年轻代占整个年轻代年老代和的 1/4 
-XX:SurvivorRatio=n:年轻代中 Eden 区与两个 Survivor 区的比值.注意 Survivor 区有两个.如: 3,表示 Eden:Survivor=3:2,一个 Survivor 区占整个年轻代的 1/5 
-XX:MaxPermSize=n:设置持久代大小
-XX:+HeapDumpOnOutOfMemoryError OOM 时自动保存堆文件，可以用 visualvm 分析堆文件

// 收集器设置 
-XX:+UseSerialGC:设置串行收集器
-XX:+UseParallelGC:设置并行收集器
-XX:+UseParalledlOldGC:设置并行年老代收集器
-XX:+UseConcMarkSweepGC:设置并发收集器

// 垃圾回收统计信息
-XX:+PrintGC 
-XX:+PrintGCDetails 
-XX:+PrintGCTimeStamps 
-Xloggc:filename

// 并行收集器设置 
-XX:ParallelGCThreads=n:设置并行收集器收集时使用的CPU数 
-XX:MaxGCPauseMillis=n:设置并行收集最大暂停时间 
-XX:GCTimeRatio=n:设置垃圾回收时间占程序运行时间的百分比.公式为 1/(1+n)

// 并发收集器设置 
-XX:+CMSIncrementalMode:设置为增量模式.适用于单 CPU 情况. 
-XX:ParallelGCThreads=n:设置并发收集器年轻代收集方式为并行收集时,使用的 CPU 数.并行收集线程数.
```