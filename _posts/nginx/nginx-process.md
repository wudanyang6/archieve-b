---
title: "Nginx 进程模型-整体架构"
date: 2021-04-10T14:26:32+08:00
draft: false
tags: ["nginx", "进程"]
---
![进程模型图示](20210410155033.png)


从网上找了一个非常好的图片，从图中可以看到很多东西

1. `Nginx` 会生成多个进程
2. `worker` 使用了`io` 多路复用的事件驱动框架
3. `worker` 内部有很多模块
4. `worker` 处理磁盘`I/O` 时，使用了标准`I/O` ，`sendfile` ，`AIO` ，`mmap` 等`I/O` 技术
5. `Cache loader` 和`Cache manager` 操作 proxy cache
6. 后端支持多种基于`tcp` 的网络协议
