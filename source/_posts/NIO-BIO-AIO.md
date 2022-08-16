---
title: 'NIO,BIO,AIO'
date: 2022-08-15 18:03:23
tags:
categories: java
---

### 1.BIO、NIO、AIO定义

Java共支持3种[网络编程](https://so.csdn.net/so/search?q=网络编程&spm=1001.2101.3001.7020)模型IO模式

- **BIO**：(blocking io）同步并阻塞（传统阻塞型），服务器实现模式为一个连接一个线程，即客户端有连接请求时，服务器就要启动一个线程进行处理，如果这个链接不做任何事，就容易造成不必要的线程开销。当一个线程调用 `read()` 或 `write()` 时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。

- **NIO**：(non blocking io 或 new io)异步非阻塞，NIO相对于BIO来说出现了几个核心的组件，分别是 Channle（通道） 和 Buffer（缓冲区）、Selector（选择器）  。NIO出现于JDK 1.4之后。

  - Channel：NIO的所有IO操作都从Channl开始： 
    - 从通道进行数据读取 ：创建一个缓冲区，然后请求通道读取数据。
    - 从通道进行数据写入 ：创建一个缓冲区，填充数据，并要求通道写入数据。

  - Buffer: 缓冲区的出现导致了NIO和BIO的不同：读数据时可以先读一部分到缓冲区中，然后处理其他事情；写数据时可以先写一部分到缓冲区中，然后处理其他事情。读和写操作可以不再持续，所以不会阻塞。当缓冲区满后才会将其放入真正地读/写。

- **Selector**：选择器可以让单个线程处理多个通道，达到复用的目的。

  <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h57mhubk5zj210y0u0acq.jpg" style="zoom:50%;" />

- **AIO**:异步非阻塞：AIO出现于JDK 1.7，是NIO的改进版。它的特点是由操作系统完成后，才通知服务器启动线程去处理。
