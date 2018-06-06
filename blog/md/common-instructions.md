---
title: Windows/CMD/Linux/Git常用命令整理
date: 2018-6-6 13:01:04
categories: [开发,总结]
tags: [Windows,CMD,Linux,Git]
---
Windows/CMD/Linux/Git常用命令整理，由于平常经常会使用故趁热把这些都一起整理下。
## 总览
- [Windows Win+R 快捷操作](#1)
- [CMD常用命令](#2)
- [Linux常用命令](#3)
- [Git常用命令](#4)
- [参考资料](#5)

---
## <span id = 1>Windows Win+R 快捷操作</span>
> 有时候不用去挨个点开什么控制面板啊再计算机什么的直接命令快速打开，列举平常使用率较高的几个
### 打开本地服务
`services.msc`这个对于开发人员是经常需要操作的
### 打开注册表
`regedit`有时候卸载或者重装软件的时候会需要操作注册表
### 打开CMD界面
`cmd`这个就不用说了，用管理员方式打开则需要右击操作
### 打开组策略
`gpedit.msc`可以查看计算机配置和用户配置
### 打开记事本
`notepad`
### 磁盘清理
`cleanmgr`可以快速清理一些回收垃圾
### 查看win版本
`winver`
### 打开画图板
`mspaint`
### 远程连接
`mstsc`
### shutdown
- `shutdown -r`表示重启
- `shutdown -s -t 60`表示60秒后正常关机
- `shutdown -s -t 60 -c 关机`则多了个提示消息<br>
常用参数有：`"-s"正常关机、"-f"强制关机、"-r"重启、"-t"定时关机、"-c" 设置提示信息、"-a" 是取消定时关机`

## <span id = 2>CMD常用命令</span>
> 由于本人使用的都是Windows系统，平时难免会需要经常接触CMD来方便一些操作，列举平常使用率较高的几个(cd/dir/切换磁盘什么的这些就不说了...)
### 查看IP信息
`ipconfig`
### 修改本机IP
`netsh interface ip set address "以太网" static 192.168.30.100 255.255.255.0 192.168.30.1`<br>
**注：这里`以太网`是网络名称，`192.168.30.100`是你要修改的IP`255.255.255.0`为子网掩码`192.168.30.1`为网关**
### 网络诊断
`ping putop.top`就是给putop.top发送数据包看有没有返回
### 无线WLAN操作
只要你输入`netsh wlan`就会出现许多提示，只要能看懂字就会操作了<br>
如`netsh wlan connect wuxian`就是连接无线名为`wuxian`的无线网络了，`netsh wlan disconnect`就是断开无线连接
### 查看端口状态
- `netstat -ano`查看所有端口占用情况<br>
- `netstat -aon|findstr "8080"`查看指定端口占用情况`8080`为端口号<br>
- `tasklist|findstr "2018"`查看指定PID对应的进程`2018`为PID<br>
- `taskkill /f /t /im java.exe`删除指定进程<br>
完整语法为：
```
TASKKILL [/S system [/U username [/P [password]]]]
         { [/FI filter] [/PID processid | /IM imagename] } [/T] [/F]

    /S    system           指定要连接的远程系统。
    /U    [domain\]user    指定应该在哪个用户上下文执行这个命令。
    /P    [password]       为提供的用户上下文指定密码。如果忽略，提示输入。
    /FI   filter           应用筛选器以选择一组任务。允许使用 "*"。例如，映像名称 eq acme*
    /PID  processid        指定要终止的进程的 PID。使用 TaskList 取得 PID。
    /IM   imagename        指定要终止的进程的映像名称。通配符 '*'可用来指定所有任务或映像名称。
    /T                     终止指定的进程和由它启用的子进程。
    /F                     指定强制终止进程。
    /?                     显示帮助消息。
```

## <span id = 3>Linux常用命令</span>
>平时部署项目也会用到Linux系统，整理下平时常用的几个(cd/ls/ll这些也不说了...)说来惭愧，经常用ftp操作感觉还挺方便还是Windows用多了啊
### 系统显示
- `date`显示系统时间
- `cal 2018`显示2018年的日历表
- `date 060610502018.00`设置日期和时间 月日时分年.秒
- `clock -w`将时间修改保存到BIOS中
- `cat /proc/cpuinfo`显示CPU info的信息
- `lsusb -tv`显示USB设备

### 关机、重启
- `shutdown -h now`关闭系统
- `shutdown -h hours:minutes &`按预定时间关闭系统
- `shutdown -c`取消按预定时间关闭系统
- `shutdown -r now`重启
- `logout`注销

### 文件和目录
- `mkdir dir`创建名为dir的目录
- `rmdir dir`删除名为dir的目录
- `rm -rf dir`删除名为dir的目录并同时删除其内容
- `rm -f file`删除名为file的文件
- `mv dir new_dir`重命名/移动名为dir的目录
- `cp file same_file`复制一个名为file的文件叫做same_file
- `cp -a dir same_dir`复制一个名为dir的目录叫做same_dir
- `cp dir/* .`复制一个目录下的所有文件到当前工作目录
- `cp -a /tmp/dir`复制一个目录到当前工作目录

### 磁盘空间
- `df -h`显示已经挂载的分区列表
- `du -sh dir`估算目录`dir`已经使用的磁盘空间

### 文件权限
- `ls -lh`显示权限
- `chmod ugo+rwx dir`设置目录的所有人(u)、群组(g)以及其他人(o)以读（r）、写(w)和执行(x)的权限，使用 "+" 设置权限，使用 "-" 用于取消
- `chattr +a file`只允许以追加方式读写文件
- `chattr +i file1`设置成不可变的文件，不能被删除、修改、重命名或者链接
- `chattr +S file`一旦应用程序对这个文件执行了写操作，使系统立刻把修改的结果写到磁盘
- `chattr +u file`若文件被删除，系统会允许你在以后恢复这个被删除的文件 
- `lsattr`显示特殊的属性

### 解压/压缩文件
- 压缩
    + `tar –cvf jpg.tar *.jpg`将目录里所有jpg文件打包成tar.jpg
    + `tar –czf jpg.tar.gz *.jpg`将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
    + `tar –cjf jpg.tar.bz2 *.jpg`将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2
    + `tar –cZf jpg.tar.Z *.jpg`将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z
    + `rar a jpg.rar *.jpg`rar格式的压缩，需要先下载rar for linux
    + `zip jpg.zip *.jpg`zip格式的压缩，需要先下载zip for linux
- 解压
    + `tar –xvf file.tar`解压 tar包
    + `tar -xzvf file.tar.gz`解压tar.gz
    + `tar -xjvf file.tar.bz2`解压 tar.bz2
    + `tar –xZvf file.tar.Z`解压tar.Z
    + `unrar e file.rar`解压rar
    + `unzip file.zip`解压zip

### 查看文件内容
- `cat file`从第一个字节开始正向查看文件`file`的内容
- `tac file`从最后一行开始反向查看文件`file`的内容
- `more file`查看一个长文件的内容
- `head -2 file`查看一个文件的前两行
- `tail -2 file`查看一个文件的最后两行
- `tail -f /var/log/messages`实时查看被添加到一个文件中的内容，这个平时查看日志经常会用到

### 进程
- `ps -ef`查看所有运行进程
- `ps -ef|grep java`指定查看java进程，java开发人员经常使用，比如鄙人
- `kill PID`停止该PID表示进程，必要时加上`- 9`强制

### 部署
- `nohup java -jar xx.jar >/dev/null &`这是jar包的后台部署，加上`&`就是后台运行

### top视图
- `top`进入top视图<br>
视图数据详解：
```
第一行：
10:01:23 — 当前系统时间
126 days, 14:29 — 系统已经运行了126天14小时29分钟（在这期间没有重启过）
2 users — 当前有2个用户登录系统
load average: 1.15, 1.42, 1.44 — load average后面的三个数分别是1分钟、5分钟、15分钟的负载情况。

第二行：
Tasks — 任务（进程），系统现在共有183个进程，其中处于运行中的有1个，182个在休眠（sleep），stoped状态的有0个，zombie状态（僵尸）的有0个。

第三行：cpu状态
6.7% us — 用户空间占用CPU的百分比。
0.4% sy — 内核空间占用CPU的百分比。
0.0% ni — 改变过优先级的进程占用CPU的百分比
92.9% id — 空闲CPU百分比
0.0% wa — IO等待占用CPU的百分比
0.0% hi — 硬中断（Hardware IRQ）占用CPU的百分比
0.0% si — 软中断（Software Interrupts）占用CPU的百分比

第四行：内存状态
8306544k total — 物理内存总量（8GB）
7775876k used — 使用中的内存总量（7.7GB）
530668k free — 空闲内存总量（530M）
79236k buffers — 缓存的内存量 （79M）

第五行：swap交换分区
2031608k total — 交换区总量（2GB）
2556k used — 使用的交换区总量（2.5M）
2029052k free — 空闲交换区总量（2GB）
4231276k cached — 缓冲的交换区总量（4GB）

第六行是空行

第七行以下：各进程（任务）的状态监控
PID — 进程id
USER — 进程所有者
PR — 进程优先级
NI — nice值。负值表示高优先级，正值表示低优先级
VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
SHR — 共享内存大小，单位kb
S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
%CPU — 上次更新到现在的CPU时间占用百分比
%MEM — 进程使用的物理内存百分比
TIME+ — 进程使用的CPU时间总计，单位1/100秒
COMMAND — 进程名称（命令名/命令行）
```

## <span id = 4>Git常用命令</span>
> 不免要使用git来版本管理

- `git init`创建版本库
- `git clone`克隆仓库
- `git add`添加文件到暂存区
- `git commit`提交更改,加上`- m`增加提交说明
- `git status`查看状态
- `git rm`删除文件
- `git diff`文件对比
- `git log`日志显示
- `git push`上传
- `git merge`合并
- `git branch -d`删除分支
- `git fetch`获取分支
- `git pull`更新

## <span id = 5>参考资料</span>
- [Windows下常用的100个CMD指令以及常见的操作](https://blog.csdn.net/CDersTeam/article/details/51346911)
- [Linux常用命令大全](https://www.cnblogs.com/yjd_hycf_space/p/7730690.html)
- [Linux下的压缩zip,解压缩unzip命令详解及实例](https://www.cnblogs.com/zdz8207/p/3765604.html)
- [Linux top命令的用法详细详解](https://blog.csdn.net/dxl342/article/details/53507673)
- [Git教程 - 廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
- [详解git fetch与git pull的区别](https://blog.csdn.net/riddle1981/article/details/74938111)