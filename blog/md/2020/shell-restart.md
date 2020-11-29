---
title: Shell 脚本定时重启项目
date: 2020-11-29 17:10:55
categories: [开发,运维]
tags: [Shell]
---

> 是时候解放双手了。

## 前言
自己很早之前就买了一台阿里云服务器，当时新用户买的时候很便宜后来每年续费简直贵得离谱。要不是我已经装了很多东西，还有我的懒（换服务器还需要重新配置和备案）。当然这跟我下面要说的关系不是很大。

我一直以来用这服务器只是为了挂自己的博客和几个很早之前练手的小项目，结果发现这破服务器总是会把我的项目进程给干掉，等我偶尔访问的时候发现早已经打不开小网站了，相当郁闷。

终于，今天我实在是忍不了了。

## 解决方案
想了想，最简单的解决方案就是我写个定时脚本去定时重启项目。这下我管你正不正常，直接给你重启了。通过网上资料的搜集和我自己的测试，不一会就搞定了，问题不大（以前也就懒，早应该弄了）。

实现脚本具体如下：

```
#!/bin/bash

echo "===start==="
tt=$(date "+%Y-%m-%d %H:%M:%S")
echo $tt

# kill
faceshow_pid=$(ps -ef | grep 'faceshow.jar' | grep -v grep |  awk '{print $2}')
if [ -z $faceshow_pid ] ;then
    echo "faceshow not exist"
else
    echo "faceshow_pid: $faceshow_pid"
    kill -9 ${faceshow_pid}
    echo "faceshow killed"
fi
mishow_pid=$(ps -ef | grep 'mishow.jar' | grep -v grep | awk '{print $2}')
if [ -z $mishow_pid ] ;then
    echo "mishow not exist"
else
    echo "mishow_pid: $mishow_pid"
    kill -9 ${mishow_pid}
    echo "mishow killed"
fi

# start
nohup /usr/local/java/jdk1.8.0_171/bin/java -jar /usr/local/jar/faceshow.jar >/usr/local/jar/log/faceshow.log 2>&1 &
echo "faceshow restarted"

nohup /usr/local/java/jdk1.8.0_171/bin/java -jar /usr/local/jar/mishow.jar >/usr/local/jar/log/mishow.log 2>&1 &
echo "mishow restarted"

echo "===end==="

```

当然还有很多可以简洁的地方，可以慢慢搞。

**说明点：**

-  `$(date "+%Y-%m-%d %H:%M:%S")` 格式化当前时间 如：2020-11-29 16:45:01
- `grep -v grep` 过滤 `grep` 自身进程
- 空格该有的有，不该有的不要有
- 执行脚本内指定需要写全路径，不然可能会报错找不到某某指令

> 脚本写好后 用 [crontab ](https://www.runoob.com/linux/linux-comm-crontab.html) 定时执行就可以了。

## 最后
脚本还是很重要的，平时绝大多数事情脚本都可以帮我们实现，科技不就是脚本实现化吗。