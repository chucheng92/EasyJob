Netty笔记整理
=============

- [x] Netty基础架构
- [x] I/O模型
- [x] 线程模型
- [x] Reactor模型
- [x] 心跳
- [x] 整流
- [x] 序列化
- [x] 资源回收
- [x] 粘连包解决方法
- [x] 断线重连解决方法
- [x] Future-Listener机制
- [ ] 责任链机制
- [x] ByteBuf And Reference-Count
- [x] Netty解包组包
- [ ] 滑动窗口协议


Netty是一个高性能、异步事件驱动的NIO框架，它提供了对TCP、UDP和文件传输的支持，作为一个异步NIO框架，Netty的所有IO操作都是异步非阻塞的，通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。
![](http://i.imgur.com/wuUdMQp.png)

作为当前最流行的NIO框架，Netty在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，一些业界著名的开源组件也基于Netty的NIO框架构建。

为什么选择Netty
----------
Netty是业界最流行的NIO框架之一，它的健壮性、功能、性能、可定制性和可扩展性在同类框架中都是首屈一指的，它已经得到成百上千的商用项目验证，例如Hadoop的RPC框架avro使用Netty作为底层通信框架；很多其他业界主流的RPC框架，也使用Netty来构建高性能的异步通信能力。
通过对Netty的分析，我们将它的优点总结如下:
- API使用简单，开发门槛低；
- 功能强大，预置了多种编解码功能，支持多种主流协议；
- 定制能力强，可以通过ChannelHandler对通信框架进行灵活地扩展；
- 性能高，通过与其他业界主流的NIO框架对比，Netty的综合性能最优；
- 成熟、稳定，Netty修复了已经发现的所有JDK NIO BUG，业务开发人员不需要再为NIO的BUG而烦恼；
- 社区活跃，版本迭代周期短，发现的BUG可以被及时修复，同时，更多的新功能会加入；
- 经历了大规模的商业应用考验，质量得到验证。在互联网、大数据、网络游戏、企业应用、电信软件等众多行业得到成功商用，证明了它已经完全能够满足不同行业的商业应用了。

Netty架构分析
---------
Netty 采用了比较典型的三层网络架构进行设计，逻辑架构图如下所示：
![](http://i.imgur.com/I9KMx9X.png)

1. **第一层**，Reactor 通信调度层，它由一系列辅助类完成，包括 Reactor 线程 NioEventLoop 以及其父类、NioSocketChannel/NioServerSocketChannel 以及其父 类、ByteBuffer 以及由其衍生出来的各种 Buffer、**Unsafe以及其衍生出的各种内部类**等。该层的主要职责就是监听网络的读写和连接操作，负责将网络层的数据 读取到内存缓冲区中，然后触发各种网络事件，例如连接创建、连接激活、读事 件、写事件等等，将这些事件触发到 PipeLine 中，由 PipeLine 充当的职责链来 进行后续的处理。
2. **第二层**,职责链 PipeLine，它负责事件在职责链中的有序传播，同时负责动态的 编排职责链，职责链可以选择监听和处理自己关心的事件，它可以拦截处理和向 后/向前传播事件，不同的应用的 Handler 节点的功能也不同，通常情况下，往往 会开发编解码 Hanlder 用于消息的编解码，它可以将外部的协议消息转换成内部 的 POJO 对象，这样上层业务侧只需要关心处理业务逻辑即可，不需要感知底层 的协议差异和线程模型差异，实现了架构层面的分层隔离。
3. **第三层**，业务逻辑处理层。可以分为两类:
	1. 纯粹的业务逻辑 处理，例如订单处理。
	2. 应用层协议管理，例如HTTP协议、FTP协议等。

> 接下来，我从影响通信性能的三个方面（I/O模型、线程调度模型、序列化方式）来谈谈Netty的架构。

I/O模型
-----
传统同步阻塞I/O模式如下图所示:
![](http://i.imgur.com/Q8PW111.png)
它的弊端有很多：
- 性能问题：一连接一线程模型导致服务端的并发接入数和系统吞吐量受到极大限制；
- 可靠性问题：由于I/O操作采用同步阻塞模式，当网络拥塞或者通信对端处理缓慢会导致I/O线程被挂住，阻塞时间无法预测；
- 可维护性问题：I/O线程数无法有效控制、资源无法有效共享（多线程并发问题），系统可维护性差；

*几种I/O模型的功能和特性对比*:
![](http://i.imgur.com/JYHI5mH.png)
Netty的I/O模型基于非阻塞I/O实现，底层依赖的是JDK NIO框架的Selector。
Selector提供选择已经就绪的任务的能力。简单来讲，**Selector会不断地轮询注册在其上的Channel，如果某个Channel上面有新的TCP连接接入、读和写事件，这个Channel就处于就绪状态，会被Selector轮询出来，然后通过SelectionKey可以获取就绪Channel的集合，进行后续的I/O操作。**
一个多路复用器Selector可以同时轮询多个Channel，由于JDK1.5_update10版本（+）使用了epoll()代替传统的select实现，所以它并没有最大连接句柄1024/2048的限制。这也就意味着只需要一个线程负责Selector的轮询，就可以接入成千上万的客户端，这确实是个非常巨大的技术进步。
使用非阻塞I/O模型之后，Netty解决了传统同步阻塞I/O带来的性能、吞吐量和可靠性问题。

线程调度模型
------
常用的Reactor线程模型有三种，分别如下：
- **Reactor单线程模型**：Reactor单线程模型，指的是所有的I/O操作都在同一个NIO线程上面完成。对于一些小容量应用场景，可以使用单线程模型。
- **Reactor多线程模型**：Rector多线程模型与单线程模型最大的区别就是有一组NIO线程处理I/O操作。主要用于高并发、大业务量场景。
- **主从Reactor多线程模型**：主从Reactor线程模型的特点是服务端用于接收客户端连接的不再是个1个单独的NIO线程，而是一个独立的NIO线程池。利用主从NIO线程模型，可以解决1个服务端监听线程无法有效处理所有客户端连接的性能不足问题。

事实上，Netty的线程模型并非固定不变，通过在启动辅助类中创建不同的EventLoopGroup实例并通过适当的参数配置，就可以支持上述三种Reactor线程模型.

在大多数场景下，并行多线程处理可以提升系统的并发性能。但是，如果对于共享资源的并发访问处理不当，会带来严重的锁竞争，这最终会导致性能的下降。为了尽可能的避免锁竞争带来的性能损耗，可以通过串行化设计，即消息的处理尽可能在同一个线程内完成，期间不进行线程切换，这样就避免了多线程竞争和同步锁。

为了尽可能提升性能，Netty采用了串行无锁化设计，在I/O线程内部进行串行操作，避免多线程竞争导致的性能下降。表面上看，串行化设计似乎CPU利用率不高，并发程度不够。但是，通过调整NIO线程池的线程参数，可以同时启动多个串行化的线程并行运行，这种局部无锁化的串行线程设计相比一个队列-多个工作线程模型性能更优。

![](http://i.imgur.com/zPCb5xC.png)

Reactor模型
---------
Java NIO非堵塞技术实际是采取反应器模式，或者说是观察者(observer)模式为我们监察I/O端口，如果有内容进来，会自动通知我们，这样，我们就不必开启多个线程死等，从外界看，实现了流畅的I/O读写，不堵塞了。

**同步和异步区别**：有无通知（是否轮询）
**堵塞和非堵塞区别**：操作结果是否等待（是否马上有返回值），只是设计方式的不同

NIO 有一个主要的类Selector，这个类似一个观察者，只要我们把需要探知的socketchannel告诉Selector，我们接着做别的事情，当有事件发生时，他会通知我们，传回一组SelectionKey，我们读取这些Key，就会获得我们刚刚注册过的socketchannel，然后，我们从这个Channel中读取数据，接着我们可以处理这些数据。

反应器模式与观察者模式在某些方面极为相似：**当一个主体发生改变时，所有依属体都得到通知。不过，观察者模式与单个事件源关联，而反应器模式则与多个事件源关联 。**
### 一般模型###
	我们想象以下情形：长途客车在路途上，有人上车有人下车，但是乘客总是希望能够在客车上得到休息。

**传统的做法是**：每隔一段时间（或每一个站），司机或售票员对每一个乘客询问是否下车。
**反应器模式做法是**：汽车是乘客访问的主体（Reactor），乘客上车后，到售票员（acceptor）处登记，之后乘客便可以休息睡觉去了，当到达乘客所要到达的目的地后，售票员将其唤醒即可。

![](http://i.imgur.com/jXgJbLi.png)
#### EventLoopGroup ####
对应于Reactor模式中的定时器的角色，不断地检索时候有事件可用(I/O线程-BOSS)，然后交给分离者将事件分发给对应的事件绑定的handler(WORK线程)。
```
	ServerBootstrap b = new ServerBootstrap();
	b.group(boss,//负责I/O操作
			work//负责事件分发
			);
```
**经验分享:**
1. 在客户端编程中经常容易出现在EVENTLOOP上做定时任务的，如果定时任务耗时很长或者存在阻塞，那么可能会将I/O操作挂起(因为要等到定时任务做完才能做别的操作)。解决方法:**用独立的EventLoopGroup**


序列化方式
-----
影响序列化性能的关键因素总结如下：
- 序列化后的码流大小（网络带宽占用）；
- 序列化&反序列化的性能（CPU资源占用）;
- 并发调用的性能表现：稳定性、线性增长、偶现的时延毛刺等;

对Java序列化和二进制编码分别进行性能测试，编码100万次，测试结果表明：Java序列化的性能只有二进制编码的6.17%左右。
![](http://i.imgur.com/EWZ5bR1.png)
![](http://i.imgur.com/2ef7ySd.png)

Netty默认提供了对Google Protobuf的支持，通过扩展Netty的编解码接口，用户可以实现其它的高性能序列化框架，例如Thrift的压缩二进制编解码框架。

不同的应用场景对序列化框架的需求也不同，对于高性能应用场景Netty默认提供了Google的Protobuf二进制序列化框架，如果用户对其它二进制序列化框架有需求，也可以基于Netty提供的编解码框架扩展实现。

**推荐阅读项目源码:**
1. **二进制协议**：[SMPP](https://github.com/twitter/cloudhopper-smpp)，该项目可以学习动态二进制大小协议的解码、编码、心跳以及整流&数据完整性协议(滑动窗口协议)等；
2. **文本协议**:[Motan](https://github.com/weibocom/motan),轻量级RPC，Future-Linstener模型，序列化框架Hessian2、fastjson；


Netty架构剖析之可靠性
-------------
Netty面临的可靠性挑战：
- 作为RPC框架的基础网络通信框架，一旦故障将导致无法进行远程服务（接口）调用。
- 作为应用层协议的基础通信框架，一旦故障将导致应用协议栈无法正常工作。
- 网络环境复杂（例如手游或者推送服务的GSM/3G/WIFI网络），故障不可避免，业务却不能中断。

从应用场景看，Netty是基础的通信框架，一旦出现Bug，轻则需要重启应用，重则可能导致整个业务中断。它的可靠性会影响整个业务集群的数据通信和交换，在当今以分布式为主的软件架构体系中，通信中断就意味着整个业务中断，分布式架构下对通信的可靠性要求非常高。

从运行环境看，Netty会面临恶劣的网络环境，这就要求它自身的可靠性要足够好，平台能够解决的可靠性问题需要由Netty自身来解决，否则会导致上层用户关注过多的底层故障，这将降低Netty的易用性，同时增加用户的开发和运维成本。

Netty的可靠性是如此重要，它的任何故障都可能会导致业务中断，蒙受巨大的经济损失。因此，Netty在版本的迭代中不断加入新的可靠性特性来满足用户日益增长的高可靠和健壮性需求。

### 链路有效性检测 ###
Netty提供的心跳检测机制分为三种：
- 读空闲，链路持续时间t没有读取到任何消息；
- 写空闲，链路持续时间t没有发送任何消息；
- 读写空闲，链路持续时间t没有接收或者发送任何消息。

![](http://i.imgur.com/gDyqeou.png)

当网络发生单通、连接被防火墙Hang住、长时间GC或者通信线程发生非预期异常时，会导致链路不可用且不易被及时发现。特别是异常发生在凌晨业务低谷期间，当早晨业务高峰期到来时，由于链路不可用会导致瞬间的大批量业务失败或者超时，这将对系统的可靠性产生重大的威胁。

从技术层面看，要解决链路的可靠性问题，必须周期性的对链路进行有效性检测。目前最流行和通用的做法就是心跳检测。

心跳检测机制分为三个层面：

- TCP层面的心跳检测，即TCP的Keep-Alive机制，它的作用域是**整个TCP协议栈**；
- 协议层的心跳检测，主要存在于长连接协议中。例如SMPP协议；
- 应用层的心跳检测，它主要由各业务产品通过约定方式定时给对方发送心跳消息实现。

> Keep-Alive仅仅是TCP协议层会发送连通性检测包，但并不代表设置了Keep-Alive就是长连接了。


心跳检测的目的就是确认当前链路可用，对方活着并且能够正常接收和发送消息。做为高可靠的NIO框架，Netty也提供了基于链路空闲的心跳检测机制：
- 读空闲，链路持续时间t没有读取到任何消息；
- 写空闲，链路持续时间t没有发送任何消息；
- 读写空闲，链路持续时间t没有接收或者发送任何消息。
	> netty自带心跳处理Handler
	IdleStateHandler

代码片段:
```
 	//心跳
    ch.pipeline().addLast("idle",new IdleStateHandler(0, 0, 55));
    ch.pipeline().addLast("heartBeat",new HeartBeatHandler());

    public class HeartBeatHandler extends ChannelDuplexHandler {

	@Override
	public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
		if (evt instanceof IdleStateEvent) {
            IdleStateEvent e = (IdleStateEvent) evt;
            if (e.state() == IdleState.ALL_IDLE) {
            	CMPPActive active = new CMPPActive();
            	ctx.writeAndFlush(active);
            }
        }

	}
}
```
补充：
### 客户端和服务端之间连接断开机制 ###
TCP连接的建立需要三个分节(三次握手)，终止则需要四个分节。
1. 某个应用进程首先调用close，称该端执行*主动关闭(active close)*。该端的TCP于是发送一个FIN分节，表示**数据发送完毕**。
2. 接收到这个FIN分节的对端执行*被动关闭(passive close)* 。这个FIN由TCP确认。他的接收也作为**一个文件结束符传递给接收端应用程序**，因为FIN的接收意味着接收端应用程序在相应连接上再无额外数据可以收取;
3. 一段时间后，接收到这个文件结束符的**应用程序**调用close管理他的套接字。这导致他的TCP也发送一个FIN。
4. 接收这个最终FIN的原发送端TCP(执行主动关闭的那一端)确认这个FIN。

既然每个方向都需要一个FIN和一个ACK，因此通常需要4个分节。但是某些情况下步骤1的FIN随数据一起发送；另外，步骤2和步骤3发送的分节都处在执行被动关闭的那一端，有可能合并成一个分节发送。
![](http://i.imgur.com/hxUN15V.png)
TCP关闭连接时对应的状态图如下:
![](http://i.imgur.com/YEIdmdI.jpg)
>对于大量短连接的情况下，经常出现卡在FIN_WAIT2和TIMEWAIT状态的连接，等待系统回收，但是操作系统底层回收的时间频率很长，导致SOCKET被耗尽。解决方案如下:

**Linux平台:**
```
	#!/bin/sh

	echo "Now,config system parameters..."

	echo "#config for MWGATE" >> /etc/sysctl.conf
	#当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击
	echo 'net.ipv4.tcp_syncookies = 1' >> /etc/sysctl.conf
	#允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
	echo 'net.ipv4.tcp_tw_reuse = 1' >> /etc/sysctl.conf
	#TIM_WAIT_2也就是FIN_SYN_2的等待时间
	echo 'net.ipv4.tcp_fin_timeout = 30' >> /etc/sysctl.conf
	#表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
	echo 'net.ipv4.tcp_tw_recycle = 1' >>/etc/sysctl.conf
	FILEMAX=$(cat /proc/sys/fs/file-max)
	PORTRANGE="net.ipv4.ip_local_port_range = 1024 $FILEMAX"
	echo $PORTRANGE >> /etc/sysctl.conf
	echo "#end config for MWGATE" >> /etc/sysctl.conf

	echo '#config for MWGATE' >> /etc/security/limits.conf
	SOFTLIMIT="* soft nofile $FILEMAX"
	HARDLIMIT="* hard nofile $FILEMAX"
	echo "$SOFTLIMIT" >> /etc/security/limits.conf
	echo "$HARDLIMIT" >> /etc/security/limits.conf
	echo '#end config for MWGATE' >> /etc/security/limits.conf

	echo "#config for MWGATE" >> /etc/pam.d/login
	echo 'session required /lib/security/pam_limits.so' >> /etc/pam.d/login
	echo "#end config for MWGATE" >> /etc/pam.d/login

	sysctl -p

	echo "#config for MWGATE" >> /etc/profile
	echo 'ulimit -S -c unlimited > /dev/null 2>&1' >> /etc/profile
	echo "#end config for MWGATE" >> /etc/profile

	echo 'core-%p-%t' >> /proc/sys/kernel/core_pattern

	source /etc/profile

	echo "Config is OK..."
```
**Windows平台:**
```
	Windows Registry Editor Version 5.00
	[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters]
	"MaxUserPort"=dword:0000fffe
	"TCPTimedWaitDelay"=dword:00000005
```

#### TCP状态图 ####
![TCP状态转化图](http://i.imgur.com/2AbvNs6.png)
### TCP/IP半关闭 ###
从上述讲的TCP关闭的四个分节可以看出，被动关闭执行方，发送FIN分节的前提是TCP套接字对应应用程序调用close产生的。如果服务端有数据发送给客户端那么可能存在服务端在接受到FIN之后，需要将数据发送到客户端才能发送FIN字节。这种处于业务考虑的情形通常称为半关闭。
>半关闭可能导致大量socket处于CLOSE_WAIT状态


#### 谁负责关闭连接合理 ####
连接关闭触发的条件通常分为如下几种:
1. 数据发送完成(发送到对端并且收到响应)，关闭连接；
2. 通信过程中产生异常；
3. 特殊指令强制要求关闭连接；

对于第一种，通常关闭时机是，数据发送完成方发起(客户端触发居多)；
对于第二种，异常产生方触发(例如残包、错误数据等)发起。但是此种情况可能也导致压根无法发送FIN。
对于第三种，通常是用于运维等。由命令发起方产生。


流量整形
------

流量整形（Traffic Shaping）是一种主动调整流量输出速率的措施。Netty的流量整形有两个作用：
- 防止由于上下游网元性能不均衡导致下游网元被压垮，业务流程中断；
- 防止由于通信模块接收消息过快，后端业务线程处理不及时导致的“撑死”问题。

流量整形的原理示意图如下：

![](http://i.imgur.com/A9mP7QC.png)

流量整形（Traffic Shaping）是一种主动调整流量输出速率的措施。一个典型应用是基于下游网络结点的TP指标来控制本地流量的输出。流量整形与流量监管的主要区别在于，流量整形对流量监管中需要丢弃的报文进行缓存——通常是将它们放入缓冲区或队列内，也称流量整形（Traffic Shaping，简称TS）。当令牌桶有足够的令牌时，再均匀的向外发送这些被缓存的报文。流量整形与流量监管的另一区别是，整形可能会增加延迟，而监管几乎不引入额外的延迟。

Netty支持两种流量整形模式：
- **全局流量整形**：全局流量整形的作用范围是进程级的，无论你创建了多少个Channel，它的作用域针对所有的Channel。用户可以通过参数设置：报文的接收速率、报文的发送速率、整形周期。[GlobalChannelTrafficShapingHandler]
- **链路级流量整形**：单链路流量整形与全局流量整形的最大区别就是它以单个链路为作用域，可以对不同的链路设置不同的整形策略。[ChannelTrafficShapingHandler针对于每个channel]

优雅停机
----
Netty的优雅停机三部曲：
1. 不再接收新消息
2. 退出前的预处理操作
3. 资源的释放操作

![](http://i.imgur.com/SleE1N1.png)

Java的优雅停机通常通过注册JDK的ShutdownHook来实现，当系统接收到退出指令后，首先标记系统处于退出状态，不再接收新的消息，然后将积压的消息处理完，最后调用资源回收接口将资源销毁，最后各线程退出执行。

通常优雅退出需要有超时控制机制，例如30S，如果到达超时时间仍然没有完成退出前的资源回收等操作，则由停机脚本直接调用kill -9 pid，强制退出。

在实际项目中，Netty作为高性能的异步NIO通信框架，往往用作基础通信框架负责各种协议的接入、解析和调度等，例如在RPC和分布式服务框架中，往往会使用Netty作为内部私有协议的基础通信框架。
当应用进程优雅退出时，作为通信框架的Netty也需要优雅退出，主要原因如下：
- 尽快的释放NIO线程、句柄等资源；
- 如果使用flush做批量消息发送，需要将积攒在发送队列中的待发送消息发送完成；
- 正在write或者read的消息，需要继续处理；
- 设置在NioEventLoop线程调度器中的定时任务，需要执行或者清理

Netty架构剖析之安全性
-------------

Netty面临的安全挑战：
- 对第三方开放
- 作为应用层协议的基础通信框架

![](http://i.imgur.com/Cm8CgHY.png)


安全威胁场景分析：
- **对第三方开放的通信框架**：如果使用Netty做RPC框架或者私有协议栈，RPC框架面向非授信的第三方开放，例如将内部的一些能力通过服务对外开放出去，此时就需要进行安全认证，如果开放的是公网IP，对于安全性要求非常高的一些服务，例如在线支付、订购等，需要通过SSL/TLS进行通信。
- **应用层协议的安全性**：作为高性能、异步事件驱动的NIO框架，Netty非常适合构建上层的应用层协议。由于绝大多数应用层协议都是公有的，这意味着底层的Netty需要向上层提供通信层的安全传输功能。

### SSL/TLS ###
Netty安全传输特性：
- 支持SSL V2和V3
- 支持TLS
- 支持SSL单向认证、双向认证和第三方CA认证。

SSL单向认证流程图如下：
![](http://i.imgur.com/dzTDjSy.png)

Netty通过SslHandler提供了对SSL的支持，它支持的SSL协议类型包括：SSL V2、SSL V3和TLS。
- **单向认证**：单向认证，即客户端只验证服务端的合法性，服务端不验证客户端。
- **双向认证**：与单向认证不同的是服务端也需要对客户端进行安全认证。这就意味着客户端的自签名证书也需要导入到服务端的数字证书仓库中。
- **CA认证**：基于自签名的SSL双向认证，只要客户端或者服务端修改了密钥和证书，就需要重新进行签名和证书交换，这种调试和维护工作量是非常大的。因此，在实际的商用系统中往往会使用第三方CA证书颁发机构进行签名和验证。我们的浏览器就保存了几个常用的CA_ROOT。每次连接到网站时只要这个网站的证书是经过这些CA_ROOT签名过的。就可以通过验证了。

可扩展的安全特性
--------
通过Netty的扩展特性，可以自定义安全策略：

- IP地址黑名单机制
- 接入认证
- 敏感信息加密或者过滤机制

IP地址黑名单是比较常用的弱安全保护策略，它的特点就是服务端在与客户端通信的过程中，对客户端的IP地址进行校验，如果发现对方IP在黑名单列表中，则拒绝与其通信，关闭链路。

接入认证策略非常多，通常是较强的安全认证策略，例如基于用户名+密码的认证，认证内容往往采用加密的方式，例如Base64+AES等。

Netty架构剖析之扩展性
-------------

通过Netty的扩展特性，可以自定义安全策略：
- 线程模型可扩展
- 序列化方式可扩展
- 上层协议栈可扩展
- 提供大量的网络事件切面，方便用户功能扩展

Netty的架构可扩展性设计理念如下：

- 判断扩展点，事先预留相关扩展接口，给用户二次定制和扩展使用；
- 主要功能点都基于接口编程，方便用户定制和扩展。


粘连包解决方案
-------
TCP粘包是指发送方发送的若干包数据到接收方接收时粘成一包，从接收缓冲区看，后一包数据的头紧接着前一包数据的尾。

出现粘包现象的原因是多方面的，它既可能由发送方造成，也可能由接收方造成。发送方引起的粘包是由TCP协议本身造成的，TCP为提高传输效率，发送方往往要收集到足够多的数据后才发送一包数据。若连续几次发送的数据都很少，通常TCP会根据优化算法把这些数据合成一包后一次发送出去，这样接收方就收到了粘包数据。接收方引起的粘包是由于接收方用户进程不及时接收数据，从而导致粘包现象。这是因为接收方先把收到的数据放在系统接收缓冲区，用户进程从该缓冲区取数据，若下一包数据到达时前一包数据尚未被用户进程取走，则下一包数据放到系统接收缓冲区时就接到前一包数据之后，而用户进程根据预先设定的缓冲区大小从系统接收缓冲区取数据，这样就一次取到了多包数据。

粘包情况有两种:
- 粘在一起的包都是完整的数据包;
- 粘在一起的包有不完整的包。

解决粘连包的方法大致分为如下三种:
1. 发送方开启TCP_NODELAY；
2. 接收方简化或者优化流程尽可能快的接收数据；
3. 认为强制分包每次只读一个完整的包。

对于以上三种方式，第一种会加重网络负担，第二种治标不治本，第三种算比较合理的。
第三种又可以分两种方式:
- 每次都只读取一个完整的包，不如不足一个完整的包，就等下次再接收，如果缓冲区有N个包要接受，那么需要分N次才能接收完成；
- 有多少接收多少，将接収的数据缓存在一个临时的缓存中，交由后续的专门解码的线程/进程处理。

以上两种分包方式，如果强制关闭程序，数据会存在丢失，第一种数据丢失在接收缓冲区；第二种丢失在程序自身缓存。

Netty自带的几种粘连包解决方案:
	1. DelimiterBasedFrameDecoder
	2. FixedLengthFrameDecoder
	3. LengthFieldBasedFrameDecoder


Netty Client断网重连机制
------------------
对于长连接的程序断网重连几乎是程序的标配。断网重连具体可以分为两类:
1. CONNECT失败，需要重连；
2. 程序运行过程中断网、远程强制关闭连接、收到错误包必须重连；

对于第一种解决方案是:

	实现ChannelFutureListener 用来启动时监测是否连接成功，不成功的话重试
```
	private void doConnect() {
    Bootstrap b = ...;
    b.connect().addListener((ChannelFuture f) -> {
        if (!f.isSuccess()) {
            long nextRetryDelay = nextRetryDelay(...);
            f.channel().eventLoop().schedule(nextRetryDelay, ..., () -> {
                doConnect();
            }); // or you can give up at some point by just doing nothing.
        }
    });
}
```
或者
```
public class ConnectionListener implements ChannelFutureListener {
  private Client client;
  public ConnectionListener(Client client) {
    this.client = client;
  }
  @Override
  public void operationComplete(ChannelFuture channelFuture) throws Exception {
    if (!channelFuture.isSuccess()) {
      System.out.println("Reconnect");
      //因为是建立网络连接所以可以共用EventLoop
      final EventLoop loop = channelFuture.channel().eventLoop();
      loop.schedule(new Runnable() {
        @Override
        public void run() {
          client.createBootstrap(new Bootstrap(), loop);
        }
      }, 1L, TimeUnit.SECONDS);
    }
  }
}
```

对于第二种:
```
public Bootstrap createBootstrap(Bootstrap bootstrap, EventLoopGroup eventLoop) {
     if (bootstrap != null) {
       final MyInboundHandler handler = new MyInboundHandler(this);
       bootstrap.group(eventLoop);
       bootstrap.channel(NioSocketChannel.class);
       bootstrap.option(ChannelOption.SO_KEEPALIVE, true);
       bootstrap.handler(new ChannelInitializer<SocketChannel>() {
         @Override
         protected void initChannel(SocketChannel socketChannel) throws Exception {
           socketChannel.pipeline().addLast(handler);
         }
       });
       bootstrap.remoteAddress("localhost", 8888);
       bootstrap.connect().addListener(new ConnectionListener(this));
     }
     return bootstrap;
   }

public class MyInboundHandler extends SimpleChannelInboundHandler {
   private Client client;
   public MyInboundHandler(Client client) {
     this.client = client;
   }
   @Override
   public void channelInactive(ChannelHandlerContext ctx) throws Exception {
     final EventLoop eventLoop = ctx.channel().eventLoop();
     eventLoop.schedule(new Runnable() {
       @Override
       public void run() {
         client.createBootstrap(new Bootstrap(), eventLoop);
       }
     }, 1L, TimeUnit.SECONDS);
     super.channelInactive(ctx);
   }
 }
```
Future-Listener机制
-----------------
Future在netty中位于io.netty.concurrent包中，其依赖关系如下:
![](http://i.imgur.com/COTiO1G.png)

  在并发编程中，我们通常会用到一组非阻塞的模型:Promise,Future,Callback。其中的Future表示**一个可能还没有实际完成的异步任务的结果**,针对这个结果添加Callback以便在执行任务**成功或者失败**后做出响应的操作。
  而经由Promise交给执行者，任务执行者通过Promise可以标记任务完成或者失败。以上这套模型是很多异步非阻塞框架的基础。
  具体的理解可参见JDK的FutureTask和Callable。JDK的实现版本，在获取最终结果的时候，不得不做一些阻塞的方法等待最终结果的到来。
  Netty的Future机制是JDK机制的一个子版本，它支持给Future添加Listener，以方便EventLoop在任务调度完成之后调用。

这里借用[Scala中Future机制](https://code.csdn.net/DOC_Scala/chinese_scala_offical_document/file/Futures-and-Promises-cn.md)来讲解一下Future、Callback、Promise具体是什么.
### 简介 ###
Future提供了一套高效便捷的非阻塞并行操作管理方案。其基本思想很简单，所谓Future，指的是一类占位符对象，用于指代某些尚未完成的计算的结果。一般来说，由Future指代的计算都是并行执行的，计算完毕后可另行获取相关计算结果。以这种方式组织并行任务，便可以写出高效、异步、非阻塞的并行代码。

默认情况下，future和promise并不采用一般的阻塞操作，而是依赖回调进行非阻塞操作。为了在语法和概念层面更加简明扼要地使用这些回调。当然，future仍然支持阻塞操作——必要时，可以阻塞(sync、await、awaitUninterruptibly)等待future（不过并不鼓励这样做）。
### Future ###
所谓Future，是一种用于指代某个尚未就绪的值的对象。而这个值，往往是某个计算过程的结果：
- 若该计算过程尚未完成，我们就说该Future未就位；
- 若该计算过程正常结束，或中途抛出异常，我们就说该Future已就位。

Future的就位分为两种情况：
- 当Future带着某个值就位时，我们就说该Future携带计算结果成功就位。
- 当Future因对应计算过程抛出异常而就绪，我们就说这个Future因该异常而失败。

Future的一个重要属性在于**它只能被赋值一次**。一旦给定了某个值或某个异常，future对象就变成了不可变对象——无法再被改写。

### Callbacks(回调函数) ###
Callback用于对计算的最终结果Future做一些后续的处理，以便我们能够用它来做一些有用的事。我们经常对计算结果感兴趣而不仅仅是它的副作用。

### Promises ###
如果说futures是为了一个还没有存在的结果，而当成一种只读占位符的对象类型去创建，那么Promise就被认为是一个可写的，可以实现一个Future的单一赋值容器。这就是说，promise通过这种success方法可以成功去实现一个带有值的future。相反的，因为一个失败的promise通过failure方法就会实现一个带有异常的future。
### Future和Promise的区别 ###
Promise与Future的区别在于，Future是Promise的一个只读的视图，也就是说Future没有设置任务结果的方法，只能获取任务执行结果或者为Future添加回调函数。
参考文档:
[WiKi](https://en.wikipedia.org/wiki/Futures_and_promises)
[Scala](http://docs.scala-lang.org/overviews/core/futures.html)


责任链机制
-----
TODO


ByteBuf
-------
ByteBuf类似与JDK的ByteBuffer，但是在ByteBuffer的基础上做了优化，例如去除了讨厌的flip()；
ByteBuf对应的继承关系如下:
![](http://i.imgur.com/rWTdrha.png)

### ReferenceCount ###
ByteBuf可以使用池化的ByteBuf(PooledHeapByteBuf)。详情参见官方文档[Reference counted objects](https://github.com/netty/netty/wiki/Reference-counted-objects)。

#### Basics of reference counting ####
初始状态下ByteBuf的reference count值是1；
```
	ByteBuf buf = ctx().alloc().buf();
	assert buf.refCnt() == 1
```
当Realease ByteBuf的时候他的reference coutn值会减一。
```
	//调用release()方法返回为True的唯一条件是引用计数变为0
	boolean released = buf.release();
	assert released;
	assert buf.refCnt() == 0
```
#### Dangling reference(指针悬挂) ####

对一个应用技术的对象进行访问会抛出IllegalReferenceCountException异常。
```
	assert buf.refCnt() == 0;
	try {
	  buf.writeLong(0xdeadbeef);
	  throw new Error("should not reach here");
	} catch (IllegalReferenceCountExeception e) {
	  // Expected
	}
```
#### 增加应用计数 ####
对于一个还未被销毁的对象可以使用retain()增加起引用计数值。
```
	ByteBuf buf = ctx.alloc().directBuffer();
	assert buf.refCnt() == 1;

	buf.retain();
	assert buf.refCnt() == 2;

	boolean destroyed = buf.release();
	assert !destroyed;
	assert buf.refCnt() == 1;
```
#### 谁来销毁它 ####
通常来说最后一个调用这个对象的人负责销毁这个对象。但也有很多特殊情况:
- 如果一个发送模块将数据传递给一个接收模块，那么发送模块不需要回收，而是将回收权利交给接收模块。
- 如果一个模块消费了引用计数对象，并且明确知道该对象不会被传递到别的模块，那么该模块必须销毁此引用计数对象。

下边是一个简单的例子:
```
	/**传递input到别的地方*/
	public ByteBuf a(ByteBuf input) {
	    input.writeByte(42);
	    return input;
	}
	/**消费了Input并做了新的封装*/
	public ByteBuf b(ByteBuf input) {
	    try {
	        output = input.alloc().directBuffer(input.readableBytes() + 1);
	        output.writeBytes(input);
	        output.writeByte(42);
	        return output;
	    } finally {
	        input.release();
	    }
	}
	/**单纯消费并且无需传递*/
	public void c(ByteBuf input) {
	    System.out.println(input);
	    input.release();
	}

	public void main() {
	    ...
	    ByteBuf buf = ...;
	    // This will print buf to System.out and destroy it.
	    c(b(a(buf)));
	    assert buf.refCnt() == 0;
	}

```

| Action                         | Who should release?  | Who released?     |
|--------------------------------|----------------------|-------------------|
| 1. main() creates buf          | buf→main()           | x                 |
| 2. main() calls a() with buf   | buf→a()              | x                 |
| 3. a() returns buf merely.     | buf→main()           | x                 |
| 4. main() calls b() with buf   | buf→b()              | x                 |
| 5. b() returns the copy of buf | buf→b(), copy→main() | b() releases buf  |
| 6. main() calls c() with copy  | copy→c()             | x                 |
| 7. c() swallows copy           | copy→c()             | c() releases copy |

#### Reference-counting in ChannelHandler ####

#### Inbound messages ####
当EventLoop读取到ByteBuf并且触发channelRead()事件，此时应该是跟pipeline相关的ChannelHandler销毁ByteBuf，因为，Handler消费了ByteBuf所以应该*channelReadl()*方法调用release()。
```
	public void channelRead(ChannelHandlerContext ctx, Object msg) {
	    ByteBuf buf = (ByteBuf) msg;
	    try {
	        ...
	    } finally {
	        buf.release();
	    }
	}
```
但是如果你将这个ByteBuf或者其他的任何引用计数的对象传递给下一个Handler，那么该handler不用销毁。
```
	public void channelRead(ChannelHandlerContext ctx, Object msg) {
	    ByteBuf buf = (ByteBuf) msg;
	    ...
	    ctx.fireChannelRead(buf);
	}
```
*需要主意的是:*在Netty中ByteBuf不是唯一一个引用计数类型。如果你在使用由**Decoders**生成的消息的时候，那么极有可能该对象也是引用计数的。
```
// Assuming your handler is placed next to `HttpRequestDecoder`
 public void channelRead(ChannelHandlerContext ctx, Object msg) {
    if (msg instanceof HttpRequest) {
        HttpRequest req = (HttpRequest) msg;
        ...
    }
    if (msg instanceof HttpContent) {
        HttpContent content = (HttpContent) msg;
        try {
            ...
        } finally {
            content.release();
        }
    }
 }
```
如果你对到底该不该release()感到疑惑那么可以使用ReferenceCountUtil.release():
```
	public void channelRead(ChannelHandlerContext ctx, Object msg) {
	    try {
	        ...
	    } finally {
	        ReferenceCountUtil.release(msg);
	    }
	}
```
**另外，可以考虑继承SimpleChannelHanlder，它对你收到的所有消息调用ReferenceCountUtil.release()**

#### Outbound messages ####
不同于Inbound Messages，Outbound Message是由引用程序创建的。理论上应该由netty在他们将数据写入到链路层。当时的调用write()方法的Handler也应该release()任何可能的引用计数对象。
```
	// Simple-pass through
	public void write(ChannelHandlerContext ctx, Object message, ChannelPromise promise) {
	    System.err.println("Writing: " + message);
	    ctx.write(message, promise);
	}

	// Transformation
	public void write(ChannelHandlerContext ctx, Object message, ChannelPromise promise) {
	    if (message instanceof HttpContent) {
	        // Transform HttpContent to ByteBuf.
	        HttpContent content = (HttpContent) message;
	        try {
	            ByteBuf transformed = ctx.alloc().buffer();
	            ....
	            ctx.write(transformed, promise);
	        } finally {
	            content.release();
	        }
	    } else {
	        // Pass non-HttpContent through.
	        ctx.write(message, promise);
	    }
	}
```

Netty解包组包
---------
对于TCP编程最常遇到的就是根据具体的协议进行组包或者解包。根据协议的不同大致可以分为如下几种类型:
1. JAVA平台之间通过JAVA序列化进行解包组包(object->byte->object)；
2. 固定长度的包结构(定长每个包都是M个字节的长度)；
3. 带有明确分隔符协议的解包组包(例如HTTP协议\r\n\r\n)；
4. 可动态扩展的协议，此种协议通常遵循消息头+消息体的机制，其中消息头的长度是固定的，消息体的长度根据具体业务的不同长度可能不同。例如(SMPP协议、CMPP协议);

#### 序列化协议组包解包 ####
可以使用的有:
[MessagePack](http://msgpack.org/)
[Google Protobuf](https://github.com/google/protobuf) 跨语言
[JBOSS Marshalling](http://jbossmarshalling.jboss.org/)
[Kryo](https://github.com/EsotericSoftware/kryo) 短小精悍
[Hessian2](http://hessian.caucho.com/) 跨语言

### 固定长度解包组包 ###
 参见Netty ：
 [FixedLengthFrameDecoder ](http://netty.io/4.1/api/index.html) 解包
[ MessageToByteEncoder](http://netty.io/4.1/api/index.html) 组包

### 带有分隔符协议的解包组包 ###
参见Netty:
[DelimiterBasedFrameDecoder](http://netty.io/4.1/api/index.html) 解包
[MessageToByteEncoder](http://netty.io/4.1/api/index.html) 组包

### HTTP ###
参见Netty:
io.netty.codec.http
### 消息头固定长度，消息体不固定长度协议解包组包 ###
参见Netty:
[LengthFieldBasedFrameDecoder](http://netty.io/4.1/api/index.html)

*需要注意的是*：

	对于解码的Handler必须做到在将ByteBuf解析成Object之后，需要将ByteBuf release()。

数据安全性之滑动窗口协议
------------
我们假设一个场景，客户端每次请求服务端必须得到服务端的一个响应，由于TCP的数据发送和数据接收是异步的，就存在必须存在一个等待响应的过程。该过程根据实现方式不同可以分为一下几类(部分是错误案例):
- 每次发送一个数据包，然后进入休眠(sleep)或者阻塞(await)状态，直到响应回来或者超时，整个调用链结束。此场景是典型的一问一答的场景，效率极其低下；
- 读写分离，写模块只负责写，读模块则负责接收响应，然后做后续的处理。此种场景能尽可能的利用带宽进行读写。但是此场景不坐控速操作可能导致大量报文丢失或者重复发送。
- 实现类似于Windowed Protocol。此窗口是以上两种方案的折中版，即允许一定数量的批量发送，又能保证数据的完整性。

以下引用一段对滑动窗口协议作解释(PS.英文版，不想翻译了):
参见Twitter的*com.cloudhopper.commons.util.window*包。
A utility class to support "windowed" protocols that permit requests to be sent asynchronously and the responses to be processed at a later time. Responses may be returned in a different order than requests were sent.

Windowed protocols generally provide high throughput over high latency links such as TCP/IP connections since they allow requests one after the other without waiting for a response before sending the next request. This allows the underlying TCP/IP socket to potentially buffer multiple requests in one packet.

The "window" is the amount of unacknowledged requests that are permitted to be outstanding/unacknowledged at any given time. This implementation allows a max window size to be defined during construction. This represents the number of open "slots". When a response is received, it's up to the user of this class to make sure that response is added so that any threads waiting for a response are properly signaled.

The life cycle of a request in a Window has 3 steps:
1. Request offered
	1. If free slot exists then goto 2
	2. If no free slot exists, offer now "pending" and block for specified time.  May either timeout or if free slot opens, then goto 2
2. Request accepted (caller may optionally await() on returned future till completion)
3. Request completed/done (either success, failure, or cancelled)

----------


精彩问答(摘抄自 [一篇文章，读懂Netty的高性能架构之道](http://mp.weixin.qq.com/s?__biz=MzA5Nzc4OTA1Mw==&mid=2659597788&idx=1&sn=8b226411fdf8d3f803d08d0c067e4ac2&scene=25&srcid=07266rCehjPwWwVM94nGeUqn#wechat_redirect) )
---------

**
问：据我之前了解到，Java的NIO selector底层在Windows下的实现是起两个随机端口互联来监测连接或读写事件，在Linux上是利用管道实现的；我有遇到过这样的需求，需要占用很多个固定端口做服务端，如果在Windows下，利用NIO框架（Mina或Netty）就有可能会造成端口冲突，这种情况有什么好的解决方案吗？**
>Linux使用Pipe实现网络监听，Windows要启动端口。目前没有更好的办法，建议的方式是作为服务端的端口可以规划一个范围，然后根据节点和进程信息动态生成，如果发现端口冲突，可以在规划范围内基于算法重新生成一个新的端口。

**问：能不能讲解一下Netty的串行无锁化设计，如何在串行和并行中达到最优？**
>为了尽可能提升性能，Netty采用了串行无锁化设计，在IO线程内部进行串行操作，避免多线程竞争导致的性能下降。表面上看，串行化设计似乎CPU利用率不高，并发程度不够。但是，通过调整NIO线程池的线程参数，可以同时启动多个串行化的线程并行运行，这种局部无锁化的串行线程设计相比一个队列-多个工作线程模型性能更优。Netty的NioEventLoop读取到消息之后，直接调用ChannelPipeline的fireChannelRead(Object msg)，只要用户不主动切换线程，一直会由NioEventLoop调用到用户的Handler，期间不进行线程切换，这种串行化处理方式避免了多线程操作导致的锁的竞争，从性能角度看是最优的。

本文引自 https://github.com/jackoll8868/net-learn/edit/master/netty.MD