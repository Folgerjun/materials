---
title: 线程管理
date: 2019-09-01 22:25:54
categories: [开发,多线程]
tags: [Java,多线程]
---

> 本文摘抄自《Java 多线程编程实战指南》核心篇 第八章小结

**本章介绍了如何将线程管控起来以便高效/可靠地利用线程这种有限地资源。**

---

&emsp;&emsp;线程组是 Thread.UncaughtExceptionHandler 的一个实现类，它可以帮助我们检测线程的异常终止。多数情况下，我们可以忽略线程组这一概念以及线程组的存在。

&emsp;&emsp;Thread.UncaughtExceptionHandler 接口使得我们能够侦测到线程运行过程中抛出的未捕获的异常，以便做出相应的补救措施，例如创建并启动相应的替代线程。**一个线程在其抛出未捕获的异常而终止前，总有一个 UncaughtExceptionHandler 实例会被选中。被选中的 UncaughtExceptionHandler 实例的 uncaughtException 方法会被该线程在其终止前执行。**UncaughtExceptionHandler 实例 > 线程所在线程组 > 默认 UncaughtExceptionHandler。

&emsp;&emsp;线程工厂 ThreadFactory 能够封装线程的创建与配置的逻辑，这使得我们能够对线程的创建与配置进行统一的控制。

&emsp;&emsp;利用条件变量我们能实现线程的暂挂与恢复，用于替代 Thread.suspend()/resume() 这两个废弃的方法。

&emsp;&emsp;**线程池是生产者——消费者模式的一个具体例子，它能够摊销线程的创建/启动与销毁的开销，并在一定程度上有利于减少线程调度的开销。**线程池使得我们能够充分利用有限的线程资源。ThreadPoolExecutor 支持核心线程大小以及最大线程池大小这两种阈值来控制线程池中的工作者线程总数。ThreadPoolExecutor 支持对核心线程以外的空闲了指定时间的工作者线程进行清理，以减少不必要的资源消耗。RejectedExecutionHandler 接口使得我们能够对被线程池拒绝的任务进行重试以提高系统的可靠性。Future 接口使得我们可以获取提交给线程池执行的任务的处理结果/侦测任务处理异常以及取消任务的执行。当一个线程池实例不再被需要的时候，我们需要主动将其关闭以节约资源。ThreadPoolExecutor 提供了一组能够对线程池进行监控的方法，通过这些方法我们能够了解线程池的当前线程池大小/工作队列的情况等数据。同一个线程池只能用于执行相互独立的任务，彼此有依赖关系的任务需要提交给不同的线程池执行以避免死锁。**我们可以通过线程工厂为线程池中的工作者线程关联 UncaughtExceptionHandler，但是这些 UncaughtExceptionHandler 只会对通过 ThreadPoolExecutor.execute 方法提交给线程池的任务起作用。**

![本章知识结构图](https://raw.githubusercontent.com/Folgerjun/materials/master/blog/img/Multithreading/Multithreading-Chapter-Eight.png) 