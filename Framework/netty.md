## Redis学习日记

Redis 是一个开源（BSD许可）的，非关系型内存数据库，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。

### Redis安装

Ubuntu上安装

```sh
$sudo apt-get update
$sudo apt-get install redis-server
```

启动 Redis

```sh
$redis-server
```

查看 redis 是否还在运行

```sh
$redis-cli
```

这将打开一个 Redis 提示符，如下图所示：

```sh
redis 127.0.0.1:6379>
```

在上面的提示信息中：127.0.0.1 是本机的IP地址，6379是 Redis 服务器运行的端口。现在输入 PING 命令，如下图所示：

```sh
redis 127.0.0.1:6379> ping
PONG
```

这说明现在你已经成功地在计算机上安装了 Redis。

在Ubuntu上安装Redis桌面管理器
要在Ubuntu 上安装 Redis桌面管理，可以从 http://redisdesktop.com/download 下载包并安装它。
Redis 桌面管理器会给你用户界面来管理 Redis 键和数据。

### 数据类型

### 数据类型String

1\.

```sh
exists 存在1，否则0 
append
get
set
strlen
```

2\.

```sh
incr
decr
incrby
decrby
del
```

3\.

```sh
GETSET
SETEX 过期时间

SETNX
如果指定的Key不存在，则设定该Key持有指定字符串Value，此时其效果等价于SET命令。相反，如果该Key已经存在，该命令将不做任何操作并返回。

SETRANGE
GETRANGE
```

4\.

```sh
setbit
getbit
```

5\. 

```sh
mset
mget
msetnx
如果在这一批Keys中有任意一个Key已经存在了，那么该操作将全部回滚，即所有的修改都不会生效。
```

### 数据类型List

1\.

```sh
lpush e.g.lpush mykey a b c d e
lpushx
lrange e.g. lrange mykey 0 -1
```

2\.

```sh
lpop
llen
```

3\.

```sh
lrem e.g. lrem mykey 2 a
lset 
lindex
ltrim
linsert
```

4\.

```sh
rpush e.g.rpush mykey a b c d
rpushx
rpop e.g.rpop mykey
rpoplpush 尾部弹出插入到另一个头部，原子操作
```

### 数据类型Hash

1\.

```sh
hset
hget
hdel
hexists
hlen
hsetnx
```

2\.

```sh
hincrby
```

3\.

```sh
hgetall
hkeys
hvals
hmget
hmset
```

### 数据类型SET

1\.

```sh
sadd
smembers
scard 数量
sismember
```

2\.

```sh
spop
srem
srandmember
smove
```

3\.

```sh
sdiff e.g.sdiff myset myset2 myset3
sdiffstore e.g.sdiffstore diffkey myset myset2 myset3
sinter e.g.sinter myset myset2 myset3 交集
sinterstore e.g.sinterstore interkey myset myset2 myset3 存储交集
sunion e.g.sunion myset myset2 myset3 并集
sunionstore e.g.sunionstore unionkey myset myset2 myset3
```

### 数据类型SortedSet

1\.

```sh
zadd
zcard
zcount
zrem
zincrby
zscore
zrange
zrank
```

2\.

```sh
zrangebysocre
zremrangebyrank
zremrangebyscore
```

3\.

```sh
zrevrange
zrevrangebyscore
zrevrank
```

### Key命令

1\.

```
flushdb
set
sadd
hset
del e.g.del key1 key2
exists
move
keys hello*
select 0切换数据库
rename 改名 改名的存在被覆盖，成功
renamenx 改名的存在操作无效
quit
info
dbsize
flushdb
```

2\.

```sh
persist
expire
expireat
ttl
```

3\.

```sh
type
randomkey
sort
```


### 事务

1\. 事务被正常执行：

```sh
    #在Shell命令行下执行Redis的客户端工具。
    /> redis-cli
    #在当前连接上启动一个新的事务。
    redis 127.0.0.1:6379> multi
    OK
    #执行事务中的第一条命令，从该命令的返回结果可以看出，该命令并没有立即执行，而是存于事务的命令队列。
    redis 127.0.0.1:6379> incr t1
    QUEUED
    #又执行一个新的命令，从结果可以看出，该命令也被存于事务的命令队列。
    redis 127.0.0.1:6379> incr t2
    QUEUED
    #执行事务命令队列中的所有命令，从结果可以看出，队列中命令的结果得到返回。
    redis 127.0.0.1:6379> exec
    1) (integer) 1
    2) (integer) 1
```

2\. 事务中存在失败的命令：

```sh
    #开启一个新的事务。
    redis 127.0.0.1:6379> multi
    OK
    #设置键a的值为string类型的3。
    redis 127.0.0.1:6379> set a 3
    QUEUED
    #从键a所关联的值的头部弹出元素，由于该值是字符串类型，而lpop命令仅能用于List类型，因此在执行exec命令时，该命令将会失败。
    redis 127.0.0.1:6379> lpop a
    QUEUED
    #再次设置键a的值为字符串4。
    redis 127.0.0.1:6379> set a 4
    QUEUED
    #获取键a的值，以便确认该值是否被事务中的第二个set命令设置成功。
    redis 127.0.0.1:6379> get a
    QUEUED
    #从结果中可以看出，事务中的第二条命令lpop执行失败，而其后的set和get命令均执行成功，这一点是Redis的事务与关系型数据库中的事务之间最为重要的差别。
    redis 127.0.0.1:6379> exec
    1) OK
    2) (error) ERR Operation against a key holding the wrong kind of value
    3) OK
    4) "4"
```

3\. 回滚事务：

```sh
    #为键t2设置一个事务执行前的值。
    redis 127.0.0.1:6379> set t2 tt
    OK
    #开启一个事务。
    redis 127.0.0.1:6379> multi
    OK
    #在事务内为该键设置一个新值。
    redis 127.0.0.1:6379> set t2 ttnew
    QUEUED
    #放弃事务。
    redis 127.0.0.1:6379> discard
    OK
    #查看键t2的值，从结果中可以看出该键的值仍为事务开始之前的值。
    redis 127.0.0.1:6379> get t2
    "tt"
```

4\. WATCH命令和基于CAS的乐观锁：

在Redis的事务中，WATCH命令可用于提供CAS(check-and-set)功能。假设我们通过WATCH命令在事务执行之前监控了多个Keys，倘若在WATCH之后有任何Key的值发生了变化，EXEC命令执行的事务都将被放弃，同时返回Null multi-bulk应答以通知调用者事务执行失败。例如，我们再次假设Redis中并未提供incr命令来完成键值的原子性递增，如果要实现该功能，我们只能自行编写相应的代码。其伪码如下：

```sh
      val = GET mykey
      val = val + 1
      SET mykey $val
```

以上代码只有在单连接的情况下才可以保证执行结果是正确的，因为如果在同一时刻有多个客户端在同时执行该段代码，那么就会出现多线程程序中经常出现的一种错误场景--竞态争用(race condition)。比如，客户端A和B都在同一时刻读取了mykey的原有值，假设该值为10，此后两个客户端又均将该值加一后set回Redis服务器，这样就会导致mykey的结果为11，而不是我们认为的12。为了解决类似的问题，我们需要借助WATCH命令的帮助，见如下代码：

```sh
      WATCH mykey
      val = GET mykey
      val = val + 1
      MULTI
      SET mykey $val
      EXEC
```

和此前代码不同的是，新代码在获取mykey的值之前先通过WATCH命令监控了该键，此后又将set命令包围在事务中，这样就可以有效的保证每个连接在执行EXEC之前，如果当前连接获取的mykey的值被其它连接的客户端修改，那么当前连接的EXEC命令将执行失败。这样调用者在判断返回值后就可以获悉val是否被重新设置成功。

### 主从复制

一、Redis的Replication：

这里首先需要说明的是，在Redis中配置Master-Slave模式真是太简单了。相信在阅读完这篇Blog之后你也可以轻松做到。这里我们还是先列出一些理论性的知识，后面给出实际操作的案例。下面的列表清楚的解释了Redis Replication的特点和优势。

1). 同一个Master可以同步多个Slaves。

2). Slave同样可以接受其它Slaves的连接和同步请求，这样可以有效的分载Master的同步压力。因此我们可以将Redis的Replication架构视为图结构。

3). Master Server是以非阻塞的方式为Slaves提供服务。所以在Master-Slave同步期间，客户端仍然可以提交查询或修改请求。

4). Slave Server同样是以非阻塞的方式完成数据同步。在同步期间，如果有客户端提交查询请求，Redis则返回同步之前的数据。

5). 为了分载Master的读操作压力，Slave服务器可以为客户端提供只读操作的服务，写服务仍然必须由Master来完成。即便如此，系统的伸缩性还是得到了很大的提高。

6). Master可以将数据保存操作交给Slaves完成，从而避免了在Master中要有独立的进程来完成此操作。
    
二、Replication的工作原理：

在Slave启动并连接到Master之后，它将主动发送一个SYNC命令。此后Master将启动后台存盘进程，同时收集所有接收到的用于修改数据集的命令，在后台进程执行完毕后，Master将传送整个数据库文件到Slave，以完成一次完全同步。而Slave服务器在接收到数据库文件数据之后将其存盘并加载到内存中。此后，Master继续将所有已经收集到的修改命令，和新的修改命令依次传送给Slaves，Slave将在本次执行这些数据修改命令，从而达到最终的数据同步。

如果Master和Slave之间的链接出现断连现象，Slave可以自动重连Master，但是在连接成功之后，一次完全同步将被自动执行。
    
三、如何配置Replication：

见如下步骤：

1). 同时启动两个Redis服务器，可以考虑在同一台机器上启动两个Redis服务器，分别监听不同的端口，如6379和6380。

2). 在Slave服务器上执行一下命令：

```sh
    /> redis-cli -p 6380   #这里我们假设Slave的端口号是6380
    redis 127.0.0.1:6380> slaveof 127.0.0.1 6379 #我们假设Master和Slave在同一台主机，Master的端口为6379
    OK
```

上面的方式只是保证了在执行slaveof命令之后，redis_6380成为了redis_6379的slave，一旦服务(redis_6380)重新启动之后，他们之间的复制关系将终止。
如果希望长期保证这两个服务器之间的Replication关系，可以在redis_6380的配置文件中做如下修改：

```sh
    /> cd /etc/redis  #切换Redis服务器配置文件所在的目录。
    /> ls
    6379.conf  6380.conf
    /> vi 6380.conf
    将
    # slaveof <masterip> <masterport>
    改为
    slaveof 127.0.0.1 6379
```

保存退出。这样就可以保证Redis_6380服务程序在每次启动后都会主动建立与Redis_6379的Replication连接了。
    
四、应用示例：

这里我们假设Master-Slave已经建立。

```sh
    #启动master服务器。
    [root@Stephen-PC redis]# redis-cli -p 6379
    redis 127.0.0.1:6379>
    #情况Master当前数据库中的所有Keys。
    redis 127.0.0.1:6379> flushdb
    OK
    #在Master中创建新的Keys作为测试数据。
    redis 127.0.0.1:6379> set mykey hello
    OK
    redis 127.0.0.1:6379> set mykey2 world
    OK
    #查看Master中存在哪些Keys。
    redis 127.0.0.1:6379> keys *
    1) "mykey"
    2) "mykey2"
    
    #启动slave服务器。
    [root@Stephen-PC redis]# redis-cli -p 6380
    #查看Slave中的Keys是否和Master中一致，从结果看，他们是相等的。
    redis 127.0.0.1:6380> keys *
    1) "mykey"
    2) "mykey2"
    
    #在Master中删除其中一个测试Key，并查看删除后的结果。
    redis 127.0.0.1:6379> del mykey2
    (integer) 1
    redis 127.0.0.1:6379> keys *
    1) "mykey"
    
    #在Slave中查看是否mykey2也已经在Slave中被删除。
    redis 127.0.0.1:6380> keys *
    1) "mykey"
```

### 数据持久化

一、Redis提供了哪些持久化机制：

1). RDB持久化：

该机制是指在指定的时间间隔内将内存中的数据集快照写入磁盘。 

2). AOF持久化:

该机制将以日志的形式记录服务器所处理的每一个写操作，在Redis服务器启动之初会读取该文件来重新构建数据库，以保证启动后数据库中的数据是完整的。

3). 无持久化：

我们可以通过配置的方式禁用Redis服务器的持久化功能，这样我们就可以将Redis视为一个功能加强版的memcached了。

4). 同时应用AOF和RDB。
    
二、RDB机制的优势和劣势：

RDB存在哪些优势呢？

1). 一旦采用该方式，那么你的整个Redis数据库将只包含一个文件，这对于文件备份而言是非常完美的。比如，你可能打算每个小时归档一次最近24小时的数据，同时还要每天归档一次最近30天的数据。通过这样的备份策略，一旦系统出现灾难性故障，我们可以非常容易的进行恢复。

2). 对于灾难恢复而言，RDB是非常不错的选择。因为我们可以非常轻松的将一个单独的文件压缩后再转移到其它存储介质上。

3). 性能最大化。对于Redis的服务进程而言，在开始持久化时，它唯一需要做的只是fork出子进程，之后再由子进程完成这些持久化的工作，这样就可以极大的避免服务进程执行IO操作了。

4). 相比于AOF机制，如果数据集很大，RDB的启动效率会更高。
    
RDB又存在哪些劣势呢？

1). 如果你想保证数据的高可用性，即最大限度的避免数据丢失，那么RDB将不是一个很好的选择。因为系统一旦在定时持久化之前出现宕机现象，此前没有来得及写入磁盘的数据都将丢失。

2). 由于RDB是通过fork子进程来协助完成数据持久化工作的，因此，如果当数据集较大时，可能会导致整个服务器停止服务几百毫秒，甚至是1秒钟。
    
三、AOF机制的优势和劣势：

AOF的优势有哪些呢？

1). 该机制可以带来更高的数据安全性，即数据持久性。Redis中提供了3中同步策略，即每秒同步、每修改同步和不同步。事实上，每秒同步也是异步完成的，其效率也是非常高的，所差的是一旦系统出现宕机现象，那么这一秒钟之内修改的数据将会丢失。而每修改同步，我们可以将其视为同步持久化，即每次发生的数据变化都会被立即记录到磁盘中。可以预见，这种方式在效率上是最低的。至于无同步，无需多言，我想大家都能正确的理解它。

2). 由于该机制对日志文件的写入操作采用的是append模式，因此在写入过程中即使出现宕机现象，也不会破坏日志文件中已经存在的内容。然而如果我们本次操作只是写入了一半数据就出现了系统崩溃问题，不用担心，在Redis下一次启动之前，我们可以通过redis-check-aof工具来帮助我们解决数据一致性的问题。

3). 如果日志过大，Redis可以自动启用rewrite机制。即Redis以append模式不断的将修改数据写入到老的磁盘文件中，同时Redis还会创建一个新的文件用于记录此期间有哪些修改命令被执行。因此在进行rewrite切换时可以更好的保证数据安全性。

4). AOF包含一个格式清晰、易于理解的日志文件用于记录所有的修改操作。事实上，我们也可以通过该文件完成数据的重建。
    
AOF的劣势有哪些呢？

1). 对于相同数量的数据集而言，AOF文件通常要大于RDB文件。

2). 根据同步策略的不同，AOF在运行效率上往往会慢于RDB。总之，每秒同步策略的效率是比较高的，同步禁用策略的效率和RDB一样高效。
    
四、其它：

1\. Snapshotting:

缺省情况下，Redis会将数据集的快照dump到dump.rdb文件中。此外，我们也可以通过配置文件来修改Redis服务器dump快照的频率，在打开6379.conf文件之后，我们搜索save，可以看到下面的配置信息：

```sh
    save 900 1              #在900秒(15分钟)之后，如果至少有1个key发生变化，则dump内存快照。
    save 300 10            #在300秒(5分钟)之后，如果至少有10个key发生变化，则dump内存快照。
    save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，则dump内存快照。
```

2\. Dump快照的机制：

```sh
    1). Redis先fork子进程。
    2). 子进程将快照数据写入到临时RDB文件中。
    3). 当子进程完成数据写入操作后，再用临时文件替换老的文件。
```

3\. AOF文件：

上面已经多次讲过，RDB的快照定时dump机制无法保证很好的数据持久性。如果我们的应用确实非常关注此点，我们可以考虑使用Redis中的AOF机制。对于Redis服务器而言，其缺省的机制是RDB，如果需要使用AOF，则需要修改配置文件中的以下条目：
将**appendonly no改为appendonly yes**
从现在起，Redis在每一次接收到数据修改的命令之后，都会将其追加到AOF文件中。在Redis下一次重新启动时，需要加载AOF文件中的信息来构建最新的数据到内存中。
    
4\. AOF的配置：

在Redis的配置文件中存在三种同步方式，它们分别是：

```sh
    appendfsync always     #每次有数据修改发生时都会写入AOF文件。
    appendfsync everysec  #每秒钟同步一次，该策略为AOF的缺省策略。
    appendfsync no          #从不同步。高效但是数据不会被持久化。
```

5\. 如何修复坏损的AOF文件：

```sh
1). 将现有已经坏损的AOF文件额外拷贝出来一份。
2). 执行"redis-check-aof --fix <filename>"命令来修复坏损的AOF文件。
3). 用修复后的AOF文件重新启动Redis服务器。
```

6\. Redis的数据备份：

在Redis中我们可以通过copy的方式在线备份正在运行的Redis数据文件。这是因为RDB文件一旦被生成之后就不会再被修改。Redis每次都是将最新的数据dump到一个临时文件中，之后在利用rename函数原子性的将临时文件改名为原有的数据文件名。因此我们可以说，在任意时刻copy数据文件都是安全的和一致的。鉴于此，我们就可以通过创建cron job的方式定时备份Redis的数据文件，并将备份文件copy到安全的磁盘介质中。

### 管道

一、请求应答协议和RTT：

Redis是一种典型的基于C/S模型的TCP服务器。在客户端与服务器的通讯过程中，通常都是客户端率先发起请求，服务器在接收到请求后执行相应的任务，最后再将获取的数据或处理结果以应答的方式发送给客户端。在此过程中，客户端都会以阻塞的方式等待服务器返回的结果。见如下命令序列：

```sh
    Client: INCR X
    Server: 1
    Client: INCR X
    Server: 2
    Client: INCR X
    Server: 3
    Client: INCR X
    Server: 4
```

在每一对请求与应答的过程中，我们都不得不承受网络传输所带来的额外开销。我们通常将这种开销称为RTT(Round Trip Time)。现在我们假设每一次请求与应答的RTT为250毫秒，而我们的服务器可以在一秒内处理100k的数据，可结果则是我们的服务器每秒至多处理4条请求。要想解决这一性能问题，我们该如何进行优化呢？
    
二、管线(pipelining)：

Redis在很早的版本中就已经提供了对命令管线的支持。在给出具体解释之前，我们先将上面的同步应答方式的例子改造为基于命令管线的异步应答方式，这样可以让大家有一个更好的感性认识。

```sh
    Client: INCR X
    Client: INCR X
    Client: INCR X
    Client: INCR X
    Server: 1
    Server: 2
    Server: 3
    Server: 4
```

从以上示例可以看出，客户端在发送命令之后，不用立刻等待来自服务器的应答，而是可以继续发送后面的命令。在命令发送完毕后，再一次性的读取之前所有命令的应答。这样便节省了同步方式中RTT的开销。
    最后需要说明的是，如果Redis服务器发现客户端的请求是基于管线的，那么服务器端在接受到请求并处理之后，会将每条命令的应答数据存入队列，之后再发送到客户端。