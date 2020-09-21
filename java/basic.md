# basic knowledge

## NIO
### Buffer
* 面向刘的I/O中，可以将数据直接写入或者将数据直接读到Stream对象中。
* 在NIO库中，所有数据都是用缓冲区处理的。任何时候访问NIO中的数据，都是通过缓冲区进行的。
* NIO是面向块的IO。

### 通道Channel
* 全双工，可以同时用于读写
* 流只能在一个方向上移动（必须是InputStream或者OutputStream的子类）
* 主要分为两类：用于网络读写的SelectableChannel和用于文件操作的FileChannel
* SocketChannel和ServerSocketChannel对应BIO的Socket类和ServerSocket类

### 多路复用器 Selector
* Selector不断轮询注册在其上的Channel，当Channel上有新的TCP连接、du和写事件，Channel就处于就绪状态，会被Selector轮询出来，返回SelectionKey，从中获取Channel集合，进行后续的IO操作

### Reactor线程的多路复用器Selector


## netty
NioEventLoopGroup 线程组，包含一组NIO线程，专门用于网络事件的处理，实际上他们是Reactor线程组  

ServerBootstrap netty用于启动NIO服务端的辅助启动类

## jvm内存
JVM可以使用的内存分为2种：堆内存和堆外内存  

堆内存完全由JVM负责分配和释放，如果程序没有缺陷代码导致内存泄露，那么就不会遇到java.lang.OutOfMemoryError这个错误  
 
使用堆外内存，就是为了能直接分配和释放内存，提高效率。JDK5.0之后，代码中能直接操作本地内存的方式有2种：使用未公开的Unsafe和NIO包下ByteBuffer。

使用ByteBuffer分配本地内存则非常简单，直接ByteBuffer.allocateDirect(10 * 1024 * 1024)即可。

堆外内存的好处是：

(1)可以扩展至更大的内存空间。比如超过1TB甚至比主存还大的空间;

(2)理论上能减少GC暂停时间;

(3)可以在进程间共享，减少JVM间的对象复制，使得JVM的分割部署更容易实现;

(4)它的持久化存储可以支持快速重启，同时还能够在测试环境中重现生产数据

## 多线程
### ThreadPoolExecutor

## MyBatis

## Servlet

##