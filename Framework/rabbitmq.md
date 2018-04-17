## 重要概念

1.
ConnectionFactory、Connection、Channel都是RabbitMQ对外提供的API中最基本的对象。Connection是RabbitMQ的socket链接，它封装了socket协议相关部分逻辑。ConnectionFactory为Connection的制造工厂。
Channel是我们与RabbitMQ打交道的最重要的一个接口，我们大部分的业务操作是在Channel这个接口中完成的，包括定义Queue、定义Exchange、绑定Queue与Exchange、发布消息等。

2.
消息回执

在实际应用中，可能会发生消费者收到Queue中的消息，但没有处理完成就宕机（或出现其他意外）的情况，这种情况下就可能会导致消息丢失。为了避免这种情况发生，我们可以要求消费者在消费完消息后发送一个回执给RabbitMQ，RabbitMQ收到消息回执（Message acknowledgment）后才将该消息从Queue中移除；如果RabbitMQ没有收到回执并检测到消费者的RabbitMQ连接断开，则RabbitMQ会将该消息发送给其他消费者（如果存在多个消费者）进行处理。

3.
Message durability

Queue与Message都设置为可持久化的（durable）

Queue的持久化

设置为持久化的队列，queue中的消息会在server本地硬盘存储一份，防止系统crash，数据丢失
设置为临时队列，queue中的数据在系统重启之后就会丢失
设置为自动删除的队列，当不存在用户连接到server，队列中的数据会被自动删除


4.
prefetchCount来限制Queue每次发送给每个消费者的消息数

5.
producer consumer
producing consuming

exchange queue bind

生产者在将消息发送给Exchange的时候，一般会指定一个routing key，来指定这个消息的路由规则routingkey 

在绑定（Binding）Exchange与Queue的同时bindkey

6.direct fanout topic headers

### RPC

MQ本身是基于异步的消息处理，前面的示例中所有的生产者（P）将消息发送到RabbitMQ后不会知道消费者（C）处理成功或者失败（甚至连有没有消费者来处理这条消息都不知道）。
但实际的应用场景中，我们很可能需要一些同步处理，需要同步等待服务端将我的消息处理完成后再进行下一步处理。这相当于RPC（Remote Procedure Call，远程过程调用）。在RabbitMQ中也支持RPC。

RabbitMQ中实现RPC的机制是：

replyTo: 一个Queue名称，用于告诉服务器处理完成后将通知我的消息发送到这个Queue中

correlationId: 此次请求的标识号，服务器处理完成后需要将此属性返还，客户端将根据这个id了解哪条请求被成功执行了或执行失败

- 客户端发送请求（消息）时，在消息的属性（MessageProperties，在AMQP协议中定义了14中properties，这些属性会随着消息一起发送）中设置两个值replyTo（一个Queue名称，用于告诉服务器处理完成后将通知我的消息发送到这个Queue中）和correlationId（此次请求的标识号，服务器处理完成后需要将此属性返还，客户端将根据这个id了解哪条请求被成功执行了或执行失败）

- 服务器端收到消息并处理

- 服务器端处理完消息后，将生成一条应答消息到replyTo指定的Queue，同时带上correlationId属性
客户端之前已订阅replyTo指定的Queue，从中收到服务器的应答消息后，根据其中的correlationId属性分析哪条请求被执行了，根据执行结果进行后续业务处理