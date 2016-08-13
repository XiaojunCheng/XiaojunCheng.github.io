# Netty学习笔记

## 基本概念

- Bootstrap & ServerBootstrap
- EventLoop & EventLoopGroup
- Channel
  - need to register to one eventloop
- ChannelPipeline
- ChannelHandler
- ByteBuf
- ChannelPromise
- ChannelFuture

## NIO

隐喻

Channel 公路，双向道，分为去路和来路。隐喻为网络连接。

ChannelPipeline，公路管理部门，可以设置收费站，关卡等设施对车辆进行检查。

ByteBuf 车辆。隐喻为网络连接中的数据。

Handler 收费站，关卡；可以对公路上的车辆进行各种处理

## Document

- [Netty实战精髓](https://www.gitbook.com/book/waylau/essential-netty-in-action/details) (Essential Netty in Action) `√`
- [Netty学习笔记](http://skyao.github.io/leaning-netty/buffer/buffer.html)
