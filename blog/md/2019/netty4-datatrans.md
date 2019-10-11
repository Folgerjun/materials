---
title: Netty4 实现数据传输中间层处理
date: 2019-10-11 10:59:49
categories: [开发,总结]
tags: [Java,Netty4]
---

> Netty4 实现数据报文的接收/拆包/重组/转发<br><br>
> 完整代码：[netty4-datatrans](https://github.com/Folgerjun/netty4-datatrans)

---

## 前言
由于项目中有对建筑的 GPS 定位模块，而 GPS 仪器作为客户端连接，传输的是标准的 GPGGA 语句，也就是多个客户端对一个服务端发送数据，节约端口资源故配置的是同一个端口，此时服务端接收到的 GPGGA 数据却并不能分辨出到底是哪一个客户端发送的，由此决定写一个数据中间层处理，给报文重组根据规则加上唯一标识符。

## 正题
根据实际需求我这写了服务端和客户端，即该脚本部署的机器同时作为 server 和 client。

可以进行对接收数据的拆包/逻辑重组/添加数据标识符等等 DIY 操作，再进行定向转发。

### 客户端处理

- 添加了 Listener 启动时可监听判断 client 是否正常启动，即对应 server 端口是否启用监听
    +   若通道连通，正常连接进行数据传输
    +   若通道未连通，则调用 schedule 进行定时重连操作

**GPSTransClientConnectionListener.java**
```
if (!future.isSuccess()) {
    final EventLoop loop = future.channel().eventLoop();
    loop.schedule(new Runnable() {
        @Override
        public void run() {
            System.err.println("client reconnecting ...");
            try {
                client.connect(GPSTransConsts.REMOTE_IP, Integer.parseInt(GPSTransConsts.REMOTE_PORT));
            } catch (NumberFormatException | InterruptedException e) {
                System.out.println("restart err...");
                e.printStackTrace();
            }
        }
    }, 5L, TimeUnit.SECONDS);
} else {
    System.out.println("client connected ...");
}
```

- 同时若是启动成功但是运行一段时间后 server 端口关闭监听了，那也要进行重连处理，可以根据实际需求更改

**GPSTransClientHandler.java**
```
@Override
public void channelInactive(ChannelHandlerContext ctx) throws Exception {
    System.err.println("server disconnect ...");
    success = false;
    // 使用过程中断线重连
    final EventLoop eventLoop = ctx.channel().eventLoop();
    eventLoop.schedule(new Runnable() {
        @Override
        public void run() {
            try {
                client.connect(GPSTransConsts.REMOTE_IP, Integer.parseInt(GPSTransConsts.REMOTE_PORT));
            } catch (Exception e) {
                System.out.println("restart err...");
                e.printStackTrace();
            }
        }
    }, 5L, TimeUnit.SECONDS);
    super.channelInactive(ctx);
}
```

由于是不停的进行转发操作，所以需要循环处理。

定义了 `private static volatile boolean success;` 作为数据发送线程的循环标志符。

当连接成功时，success 置为 true，当连接断开时，success 置为 false。

`volatile` 修饰故保证了其可见性。

```
@Override
public void channelActive(ChannelHandlerContext ctx) throws Exception {
    System.out.println("channelActive ...");
    success = true;
    System.out.println("send data to server ...");
    // 必须另开线程处理，否则会在这个方法中出不去
    new Thread() {
        @Override
        public void run() {
            while (success) {
                if (!GPSTransConsts.NAME_MESS.isEmpty()) {
                    StringBuilder sb = new StringBuilder();
                    GPSTransConsts.NAME_MESS.values().forEach(value -> {
                        sb.append(value);
                    });
                    ByteBuf resp = Unpooled.copiedBuffer(sb.toString(), CharsetUtil.UTF_8);
                    ctx.writeAndFlush(resp);
                    try {
                        Thread.sleep(1000L);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            System.out.println("client thread exit ...");
        };
    }.start();
    super.channelActive(ctx);
}
```

### 服务端处理

接收多个客户端数据，根据其 IP 来定位设备，再进行报文拆包重组 DIY，存储到内存中便于 client 模块进行转发。

**GPSTransServerHandler.java**
```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
  
    InetSocketAddress ipsocket = (InetSocketAddress) ctx.channel().remoteAddress();
    // 获取客户端 IP
    String clientIP = ipsocket.getAddress().getHostAddress();
    int index = clientIP.lastIndexOf(".");
    String ipNum = clientIP.substring(index + 1);
    ByteBuf in = (ByteBuf) msg;
    String message = in.toString(CharsetUtil.UTF_8);
    if (message.startsWith("$")) {
        message = message.replace("$", "#");
        if (!GPSTransConsts.IP_NAME.containsKey(ipNum)) {
            System.err.println(ipNum + "未配置!");
            return;
        }
        String name = GPSTransConsts.IP_NAME.get(ipNum);
        message = "#" + GPSTransConsts.IP_NAME.get(ipNum) + message + "\r";
        GPSTransConsts.NAME_MESS.put(name, message);
    }
    // 释放
    super.channelRead(ctx, msg);
}
```
因为我们没有进行 write 和 flush 操作，所以需要进行释放。

### 配置文件

为了方便配置的修改，可以把项目打成 jar 包，然后在同目录下新建一个 config 文件夹，把 gps.properties 丢进去，完事。