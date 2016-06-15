# Java NIO：浅析I/O模型

NIO是Java 4里面提供的新的API，目的是用来解决传统IO的问题。


## 为什么要使用 NIO?

NIO 的创建目的是为了让 Java 程序员可以实现高速 I/O 而无需编写自定义的本机代码。NIO 将最耗时的 I/O 操作(即填充和提取缓冲区)转移回操作系统，因而可以极大地提高速度。

## 流与块的比较

原来的 I/O 库(在 java.io.*中) 与 NIO 最重要的区别是数据打包和传输的方式。正如前面提到的，原来的 I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。

面向流 的 I/O 系统一次一个字节地处理数据。一个输入流产生一个字节的数据，一个输出流消费一个字节的数据。为流式数据创建过滤器非常容易。链接几个过滤器，以便每个过滤器只负责单个复杂处理机制的一部分，这样也是相对简单的。不利的一面是，面向流的 I/O 通常相当慢。

一个 面向块 的 I/O 系统以块的形式处理数据。每一个操作都在一步中产生或者消费一个数据块。按块处理数据比按(流式的)字节处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

## NIO中的几个基础概念

在NIO中有几个比较关键的概念：Channel（通道），Buffer（缓冲区），Selector（选择器）。

可以将NIO 中的Channel同传统IO中的Stream来类比，但是要注意，传统IO中，Stream是单向的，比如InputStream只能进行读取操作，OutputStream只能进行写操作。而Channel是双向的，既可用来进行读操作，又可用来进行写操作。Buffer（缓冲区），是NIO中非常重要的一个东西，在NIO中所有数据的读和写都离不开Buffer。

NIO中最核心的一个东西：Selector。可以说它是NIO中最关键的一个部分，Selector的作用就是用来轮询每个注册的Channel，一旦发现Channel有注册的事件发生，便获取事件然后进行处理。 
![Selector](http://images.cnitblog.com/i/288799/201408/181105032681855.jpg)

## Channel 

以下是常用的几种通道：

- FileChannel
- SocketChanel
- ServerSocketChannel
- DatagramChannel

通过使用FileChannel可以从文件读或者向文件写入数据；通过SocketChannel，以TCP来向网络连接的两端读写数据；通过ServerSocketChanel能够监听客户端发起的TCP连接，并为每个TCP连接创建一个新的SocketChannel来进行数据读写；通过DatagramChannel，以UDP协议来向网络连接的两端读写数据。

## Buffer

Buffer，故名思意，缓冲区，实际上是一个容器，是一个连续数组。Channel提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由Buffer。具体看下面这张图就理解了：
![Buffer](http://images.cnitblog.com/i/288799/201408/181521285655283.jpg)

在NIO中，Buffer是一个顶层父类，它是一个抽象类，常用的Buffer的子类有：

- ByteBuffer
- IntBuffer
- CharBuffer
- LongBuffer
- DoubleBuffer
- FloatBuffer
- ShortBuffer

如果是对于文件读写，上面几种Buffer都可能会用到。但是对于网络读写来说，用的最多的是ByteBuffer。

## Selector

Selector类是NIO的核心类，Selector能够检测多个注册的通道上是否有事件发生，如果有事件发生，便获取事件然后针对每个事件进行相应的响应处理。这样一来，只是用一个单线程就可以管理多个通道，也就是管理多个连接。这样使得只有在连接真正有读写事件发生时，才会调用函数来进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程，并且避免了多线程之间的上下文切换导致的开销。

与Selector有关的一个关键类是SelectionKey，一个SelectionKey表示一个到达的事件，这2个类构成了服务端处理业务的关键逻辑。