---
title: Java 异步编程
date: 2019-09-01 23:25:54
categories: [开发,多线程]
tags: [Java,多线程]
---

> 本文摘抄自《Java 多线程编程实战指南》核心篇 第九章小结

**本章介绍了同步计算与异步计算的概念，并介绍了 Java 平台对异步计算所提供的相关 API。**

---

&emsp;&emsp;从单个任务的角度来看，任务的执行方式可以是同步的，也可以是异步的。**同步方式的优点是代码简单/直观，缺点是它往往意味着阻塞，因此不利于系统的吞吐率。异步方式的优点则是它往往意味着非阻塞，因此有利于系统的吞吐率，其代价是相对复杂的代码和额外的开销。**阻塞/非阻塞是任务执行方式的属性，它们与任务执行方式没有必然的联系：同步任务既可能是阻塞的，也可能是非阻塞的；异步任务既可能是非阻塞的，也可能是阻塞的。**对于同一个任务，我们既可以说它是同步任务也可以说它是异步任务，这取决于任务的执行方式以及我们的观察角度。**

&emsp;&emsp;Runnable/Callable 接口是对任务处理逻辑进行的抽象，而 Executor 接口是对任务的执行进行的抽象。Executor 接口使得我们能够对任务的提交与任务的具体执行细节进行解耦，这为更改任务的具体执行细节提供了灵活性与便利。ExecutorService 接口是对 Executor 接口的增强：它支持返回异步任务的处理结果/支持资源的管理接口/支持批量任务提交等。ThreadPoolExecutor 是 Executor/ExecutorService 接口的一个实现类。实用工具类 Executors 为线程池的创建提供了快捷方法。Completion Service 接口为异步任务的批量提交以及获取这些任务的处理结果提供了便利，其默认实现类为 ExecutorCompletionService。

&emsp;&emsp;FutureTask 是 Java 标准库提供的 Future 接口实现类，它还实现了 Runnable 接口。因此，FutureTask 可直接用来获取异步任务的处理结果，它可以交给专门的工作者线程执行，也可以交给 Executor 实例执行，甚至由当前线程直接执行（同步）。**一般来说，FutureTask 是一次性使用的，一个 FutureTask 实例代表的任务只能够被执行一次。如果需要多次执行同一个任务，那么可以考虑 AsyncTask 类。**

&emsp;&emsp;计划任务的执行方式包括延迟执行和周期性执行。ScheduledThreadPoolExecutor 是 ScheduledExecutorService 接口的默认实现类，它可以用于执行计划任务。ScheduledFuture 接口可用来获取延迟执行的计划任务的处理结果。如果要获取周期性执行的计划任务的处理结果，可以使用自定义的 AsyncTask 类。**周期性执行的计划任务，其执行周期并不是固定的，而是受任务单次执行耗时的影响：提交给 scheAtFixedRate 方法执行的计划任务，其执行周期为 max（Execution Time,period）；提交给 scheduleWithFixedDelay 方法执行的计划任务，其执行周期为 Execution Time + delay。计划任务在其执行过程中如果抛出未捕获的异常，那么该任务将不会再被执行。**

![本章知识结构图](https://raw.githubusercontent.com/Folgerjun/materials/master/blog/img/Multithreading/Multithreading-Chapter-Nine.png) 