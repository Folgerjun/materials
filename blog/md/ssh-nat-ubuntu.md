---
title: 使用 ssh 反向隧道穿透 NAT 访问 Linux 内网主机
date: 2018-11-03 11:15:10
categories: [开发,运维]
tags: [Ubuntu,ssh,脚本,开机自启]
---

# 使用 ssh 反向隧道穿透 NAT 访问内网主机

## 前言
由于公司经常会有项目需要去业主那边搭建服务器，基本不需要什么流量所以就准备用 4G 网卡搭建。而该网卡无固定公网 ip，只有内网 ip，我们目的就是为了可以远程操控以避免有时因业务需要往业主那边跑，要是地方比较远来回一趟也得花个把星期，不划算。所以就研究了下 ssh 隧道穿透来满足我们的需求。

## 场景

现在我们有三台机子：

- A：公司内网电脑（Win 10）
- B：公司内网服务器（Linux，固定外网 ip：58.247.33.44，ssh 开放端口：8862）
- C：业主那边 4G 网卡搭建的服务器（Linux，无固定外网 ip，ssh 开放端口：22）

操作前：A 可以访问 B，C 能上网也能访问 B，但是 B 不能访问 C，A 不能访问 C
我们需要满足：A 可以访问 C

**当然，很多远程桌面软件可以满足我们这个需求，但是由于太不方便和不稳定，还是考虑 ssh。（安装 ssh 可见底文的[附加安装指令](#附加安装)）**

## 配置

### 配置 B 服务器

- 修改服务器上的 sshd 设置

`vim /etc/ssh/shhd_config` 建议使用 vim，vim 比 vi 更强大（安装 vim 可见底文的[附加安装指令](#附加安装)）

- 把 GatewayPorts 打开(去掉前面的 # 号注释，没有就直接在底下添加)

`GatewayPorts yes` 打开允许映射端口

- 存盘后退出，并重新启动 sshd (不知如何操作文件的自行上网查阅)

`service ssh restart` 这个具体得看你 service 的实际名字（Ubuntu 下 `service --status-all` 可以查看所有服务状态）

### 配置 C 服务器

#### 可以进行映射操作了

`ssh -NfR 1234:localhost:22 user@58.247.33.44 -p8862`

>N 参数，表示只连接远程主机，不打开远程shell。<br>
>f 参数，表示后台运行。<br>
>R 参数接受三个值，分别是"远程主机端口:目标主机:目标主机端口"。<br>
>p 参数，表示指定 ssh 对外开放的端口号。<br>
>user 是 B 服务器的用户。<br>
>这条命令的意思，就是让 B 服务器监听它自己的 1234 端口，然后将所有数据经由 B 服务器转发到 C 服务器的 22 端口。这就被称为"远程端口绑定"。

这里每次连接需要输入 B 服务器的密码，不太方便，待会再详细介绍。

映射操作后我们可以发现在 B 服务器上已经开启了 1234 端口的监听，已经可以通过 1234 端口进行 ssh 连接到 C 服务器了。

#### 密钥验证，直接登录

接回上面说的每次需要输入密码的问题，我们可以用 ssh 密钥来实现自动登录。

**在 C 服务器上生成公钥和私钥**

`ssh-keygen` （一直 enter）

`ls ~ /.ssh/` 查看是否生成，显示如下

>id_rsa id_rsa.pub known_hosts

`ssh-copy-id user@58.247.33.44` 复制到 B 服务器中

好了，这样就不需要每次都输入 B 服务器密码了。

#### autossh 实现自动重连

由于上述的 ssh 反向连接十分不稳定，可能随时断开，一旦断开就无法进行访问了。所以我们需要 autossh，它可以实现自动重连。（安装 autossh 可见底文的[附加安装指令](#附加安装)）

`autossh -M 5678 -NR 1234:localhost:22 user@58.247.33.44 -p8862`

>M 参数，指定了 autossh 的监听端口，监听是否断开然后进行重连操作<br>
>autossh 本身就是后台执行，所以就省去了 f 参数。

到这里就很完美了，可是还不够。要是 C 服务器宕机重启了怎么办，autossh 又不会自动执行。

#### 实现开机自启

这里以 Ubuntu 18.04 为例。

`ls /lib/systemd/system` 执行该指令我们可以看到许多启动脚本，我们需要操作的就是 **rc.local.service**

打开脚本内容，我们在最后面加上一段：
```
[Install]  
WantedBy=multi-user.target  
Alias=rc-local.service
```

保存退出。

由于 ubuntu-18.04 默认是没有 /etc/rc.local 这个文件的，需要自己创建。

```
sudo touch /etc/rc.local
chmod 755 /etc/rc.local
```

**这里千万别忘记给 rc.local 文件设置可执行权限，不然没用。**

在 `rc.local` 中你就可以编写脚本了。**注意开头 #!/bin/bash 不可少**

例如我的是：
```
#!/bin/bash
LOG_TIME=`date "+%Y-%m-%d %H:%M:%S"`
echo '123456' | sudo -S autossh -M 5678 -NR 1234:localhost:2223 user@58.247.33.44 -p8862
echo "autossh restart:"$LOG_TIME >>/usr/local/autossh.log
```

这里我是用 root 权限执行，123456 是 C 服务器的用户密码。 `>>` 表示追加，不覆盖。同时打印了执行的时间。

最后一步，前面我们说 systemd 默认读取 /etc/systemd/system 下的配置文件, 所以还需要在 /etc/systemd/system 目录下创建软链接。

`ln -s /lib/systemd/system/rc.local.service /etc/systemd/system/`

可以了，重启系统查看脚本是否执行，日志中是否有内容。

## 参考博文

笔者参考了如下博文并进行了整理实验：

- [使用ssh隧道穿透NAT访问内网主机(超干货)](https://blog.csdn.net/cayman_mg/article/details/79527207)
- [SSH反向连接及Autossh](https://www.cnblogs.com/eshizhan/archive/2012/07/16/2592902.html)
- [ubuntu-18.04 设置开机启动脚本](http://www.r9it.com/20180613/ubuntu-18.04-auto-start.html)
- [ubuntu 18.04 配置 rc.local](https://blog.csdn.net/a912952381/article/details/81205095)

## 附加安装

- 更新源列表

`sudo apt-get update`

- 安装 ssh

`sudo apt-get install openssh-client` 安装客户端 （反向隧道需要）

`sudo apt-get install openssh-server` 安装服务端

- 安装 vim

```
sudo apt-get remove vim-common
sudo apt-get install vim
```

Ubuntu 18.04 中 vi 方向键有点问题，vim 很好用。

- 安装 autossh

`sudo apt-get install autossh`

- 安装 net-tools

当发现输入 `ifconfig` 不可用时

`sudo apt install net-tools` 装之

