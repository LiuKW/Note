## Netty

### Netty是什么

* Netty是一个异步的，基于事件驱动的网络应用框架，用以快速开发高性能、高可靠性的网络IO框架。

* Netty主要针对在TCP协议下，面向Clients端的高并发应用，或者P2P场景下的大量数据持续传输的应用。

* Netty本质是一个NIO框架，适用于服务器通讯相关的多种应用场景。

  

### NIO

* Java NIO 全称 java non-blocking IO，是指 JDK 提供的新API。从 JDK 1.4 开始，Java提供了一系列改进的输入 / 输出的新特性，被统称为NIO（即New IO) ，是同步非阻塞的。
* NIO相关类都被放在java.nio包及子包下。
* NIO有三大核心：**Channel（通道），Buffer（缓冲区），Selector（选择器）**；
* NIO是面向缓冲区编程的。数据读取到一个稍后处理的缓冲区，需要时可在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞式的高伸缩性网络。

**NIO和BIO的区别**

* BIO以流的方式处理数据，而NIO以块的方式处理数据，块I/O的效率比流I/O高很多
* BIO基于字节流和字符流进行操作。而NIO基于Channel和Buffer进行操作，数据总是从通道读取到缓冲区，或者从缓冲区写入到通道中。Selector用于监听多个通道的事件（例如：连接请求，数据到达），因此使用单个线程就可以监听多个客户端通道



**NIO三大核心组件关系图**

![](../img/netty/Channel-Buffer-Selector关系图.png)

* 关系说明
  * 每个Channel都对应一个Buffer
* 一个Selector对应一个线程，一个线程可以对应多个Channel（Channel可以理解为一个连接)
  * Channel要注册到Selector中，该图中，有三个Channel注册到了Selector中
* 程序切换到哪个Channel是由事件决定的，Event是一个重要的概念
  * Selector会根据不同的事件，在各个通道上切换
* Buffer就是一个内存块，底层是一个数组
  * 数据的读取写入是通过Buffer的，而在BIO中都是通过输入流或者输出流（单向）。但是NIO的Buffer是可以读，也可以写的，需要通过flip方法切换。
* Channel是双向的，可以返回底层操作系统的情况。（Buffer是双向的，Channel也是双向的）
  





**Buffer**







**Channel**

* NIO的通道类似于流，但有些区别如下：
  * 通道可以同时进行读写，而流只能读或者只能写
  * 通道可以实现异步读写数据
  * 通道可以从缓冲读数据，也可以写数据到缓冲，通道是双向的，而流只是单向的
* read，从通道读取数据并发到缓冲区中
* write，把缓冲区的数据写到通道中







