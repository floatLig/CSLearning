### 在项目中缓存是如何使用的

秒杀系统里面有：商品信息、用户token，id、订单、中间变量（判断秒杀是不是已经结束）

### 为啥在项目里要用缓存呢

![高性能高并发.jpg](../../_img/高性能高并发.jpg)

高性能和高并发。

高性能：不用每次都查数据库，数据库的复杂查询时间也很慢。

### 用了缓存之后会有啥不良的后果

1. 缓存与数据库双写不一致
1. 缓存雪崩
1. 缓存穿透
1. 缓存并发竞争

### redis和memcached有什么区别

1. redis的数据结构比memcached要多
2. redis可以支持集群

### Redis线程模型

IO多路复用（底层用select等）。

文件事件处理器，是单线程的，redis才叫做单线程的模型

![01_redis单线程模型.png](../../_img/01_redis单线程模型.png)

### 为啥redis单线程模型也能效率这么高

1）`纯内存`操作【相对于MySQL】
2）核心是基于非阻塞的IO多路复用机制
3）单线程反而避免了多线程的频繁上下文切换问题【相对于memcache】

### redis都有哪些数据类型？分别在哪些场景下使用比较合适

1. key-value
2. List：队列、栈
3. Set：交集差集并集
4. Map：对象
5. zSet：排序

### redis的过期策略都有哪些？内存淘汰机制都有哪些？手写一下LRU代码实现

> 定性删除、惰性删除、内存淘汰机制

如果假设你设置一个一批key只能存活1个小时，那么接下来1小时后，redis是怎么对这批key进行删除的？

`定期删除+惰性删除。`

为什么这么做呢？

假设redis里放了10万个key，都设置了过期时间，你每隔几百毫秒，`就检查10万个key`，那redis基本上就死了，cpu负载会很高的，消耗在你的检查过期key上了。redis是每隔100ms`随机抽取一些key`来检查和删除的。【[隔多少时间扫描是通过Hz参数设定的](https://www.cnblogs.com/chenpingzhao/p/5022467.html?utm_source=tuicool&utm_medium=referral)】

并不是key到时间就被删除掉，`而是你查询这个key的时候，redis再懒惰的检查一下`

但是实际上这还是有问题的，如果定期删除漏掉了很多过期key，然后你也没及时去查，也就没走惰性删除，此时会怎么样？如果大量过期key堆积在内存里，导致redis内存块耗尽了，咋整？

答案是：`走内存淘汰机制`。

1）noeviction：当内存不足以容纳新写入数据时，新写入操作`会报错【只是报错而已】`，这个一般没人用吧，实在是太恶心了    
2）allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key（这个是最常用的）  
3）allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key，这个一般没人用吧，为啥要随机，肯定是把最近最少使用的key给干掉啊  
4）volatile-lru：当内存不足以容纳新写入数据时，在`设置了过期时间的键空间中`，移除最近最少使用的key（这个一般不太合适）  
5）volatile-random：当内存不足以容纳新写入数据时，在`设置了过期时间的键空间中`，随机移除某个key  
6）volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除  

> [leetcode -- LRU算法](https://floatlig.github.io/JavaLearning/#/_source/%E7%AE%97%E6%B3%95/146.LRU%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6)

```java
class LRUCache extends LinkedHashMap<Integer, Integer>{

    private final int capacity;

    //这里就是传递进来的最多能缓存多少数据
    public LRUCache(int capacity) {
        //这块就是设置一个hashmap的初始大小，loadFactor是负载因子，同时最后一个true指按照访问顺序进行排序，
        //最近访问的放在头，最老访问的放在尾
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        //这个意思就是说：当map中的数据量大于缓存的这个数，就自动删除最老的数据
        return size() > capacity;
    }
}
```

### Redis如何保证高并发和高可用

> 主从复制 读写分离 + 哨兵

- 首先要明白`Redis高并发`跟`整个系统的高并发`之间的关系：

如果要搞高并发的话，作为缓存功能的Redis一定要搞好，使用`主从复制、读写分离横向拓展Redis,提高QPS，使用哨兵模式，提高Redis的可用性，还是可以使用集群`。

但是真正的高并发的话，除了Redis，MySQL也要进行相应的处理，使用`分库分表`，使用多级缓存架构、热点缓存。

---

- Redis不能支撑高并发的瓶颈在哪里？

一般来说，`单机的Redis` QPS几乎不太可能超过10W+，单机的Redis QPS一般在 1w ~ 几万之间。要达到10W+只有在特殊情况下才能达到，比如说：机器性能特别好，配置特别高，物理机维护做的特别好，而且你的整体操作不是太复杂。

所以，使用主从复制、读写分离用来提高QPS。【大量的请求都是读的，少量请求才是写的】

---

- Redis的Replication的流程

> <https://juejin.im/post/5cb003e51d456e2b15f5da#heading-0>

![16a10acc8f77f36d.png](../../_img/16a10acc8f77f36d.png)

1. 一个Master节点可以配置多个Slave node, Slave node也可以连接其他的Slave node
2. slave node启动，仅仅保留master node的信息（这些信息在配置文件中 slaveof 说明了），包括master node的IP，host，密码
3. slave node内部有一个定时任务，每秒检查是否有新的master node要连接和复制，如果发现，就跟master node建立socket网络连接
4. slave node发送ping命令给master node
5. 口令认证，如果master设置了requirepass，那么salve node必须发送masterauth的口令过去进行认证
6. 如果这个slave node是第一次连接到master node，那么就会触发一次full resynchronization。
7. 【full resynchronization的底层实现】开始full synchronization的时候，`master会启动一个后台线程`，开始生成一份`RDB快照文件(这个RDB快照文件是在内存中的)`，同时还会将从客户端收到的`所有写命令 (新接收到的写命令) 缓存在内存中`。RDB文件生成完毕之后，master会将这个RDB文件发送给slave，`slave会写入本地磁盘，然后再从本地磁盘加载到内存中`。然后master会将内存中的写命令发送给slave，slave也会同步这些数据。slave node如果跟master node有网络故障，断开了连接，会自动重连。master如果发现有多个slave node都来重新连接，仅仅会启动一个rdb save 操作，用一份数据服务所有的slave node。
8. Redis采用异步方式复制数据到Slave节点，从Redis2.8开始，Slave node会周期性的向Master确认自己每次复制的数据量
9. Slave做复制的时候，是不会block master node的正常工作，也不会block对自己的查询工作，它会用旧的数据来提供服务；但是复制完成的时候，需要删除旧数据集，加载新数据集，这个时候就会暂停对外服务
10. Slave node主要用来进行横向扩容，做读写分离，扩容的Slave node可以提高吞吐量。Slave对于高可用有很大的关系。

---

- 主从复制的断点续传

从Redis2.8开始，就支持主从复制的断点续传，如果主从复制过程中，网络断掉了，那么可以接着上次复制的地方，继续复制下去，而不是从头开始复制一份。

master node会在内存中保存一个backlog，master和slave都会保存一个replica offset，还有一个master id，offset就是保存在backlog中。如果master和slave网络连接断掉了，slave会让master从上一次的offset开始复制。

但是如果没有找到对应的offset，那么会执行下一次recynchronization.

---

- 无磁盘化复制

```bash
repl-diskless-sync yes  # 默认为no，master在内存中直接创建rdb，然后发送给slave，不会在自己本地落地磁盘
repl-diskless-sync-delay 5 # 等待一定时长再开始复制，因为要等更多slave重新连接过来
```

---

- master持久化对于主从结构的安全保障和意义：

  1. 如果开启了主从架构，那么建议开启master node持久化，不建议用slave node作为master node的数据热备。如果你关掉master的持久化，可能在master宕机重启的时候是空的，那再经过一次主从复制，slave的数据也丢了。
  2. 详细说明一下，slave数据丢的过程：首先，master的RDB跟AOF都关闭了，就将数据全部存在内存中，那么这个时候，假设master宕机，然后又快速重启，master就没有本地数据可以恢复，mastet就会以为数据是空的，master就会将空的数据同步到slave上面去，这样子所有的slave数据就都被清掉了，造成100%数据丢失。所以master必须做持久化。即使采用哨兵模式，如果sentinal没有检测到master failture，master node就重启了，还是会导致100%数据丢失。
  3. 除此之外，slave node也会做`本地备份`，以防万一，本地数据也丢了

---

- 过期键的处理

slave不会过期键，只会等待master过期key.如果master过期了一个key，或者通过LRU淘汰了一个key，那么会模拟一个del命令发送给slave。

---

- 数据同步的核心机制：

在第一次slave连接master的时候，执行的是全量复制，那个过程里面需要注意的细节。

1. master和slave都会维护一个offset。master会在自身不断累加offset，slave也会在自身不断累加offset。slave每秒都会上报自己的offset给master，同时master也会保存每个slave的offset。【通过这个offset，就能够知道master和slave的数据会不会不一样。当然offset不仅仅用于全量复制】

3. backlog：

master node会有一个backlog[积压，相当于缓冲]，默认是1MB大小。master node给slave node复制数据的时候，也会将数据在backlog中同步写一份。backlog主要是用来做全量复制意外中断，使用增量复制继续完成全量复制没有完成的部分。

3. master run id

通过命令`info server`，可以看到master run id。

### Redis如何做到99.99%的高可用

首先解释一下，什么叫99.99%的高可用。你的系统，在这一年中99.99%的时间都能对外提供服务，这就叫做高可用。

影响高可用性有很多种情况，以下情况可能会导致你的系统不能对外提供服务，比如：

1. 机器死了，宕机了
2. jvm进程oom了，挂了
3. 机器CPU打满了，不能工作了，hang死了
4. 磁盘突然满了，系统IO报错了，不工作了

Redis在什么情况下会影响高可用？

当Redis使用主从复制时，有一个slave宕机了，是不用影响可用性的，因为这个时候还有其他slave可以顶。

但是如果master宕机了，而且master短时间内恢复不了，这个时候大量的写请求过来，不会打到缓存上，而是直接打到MySQL上，MySQL宕机就完了，不可用了。

所以，当master宕机了，必须能够在短时间内出现另一个master，用来接收写请求。sentinal node哨兵模式，就是用来做这种事的。当一个master宕机了，sentinal会通过选举，将一个slave选举为master，这个过程叫做故障转移（failover，主备切换）

---

- 经典的三节点哨兵模式

一台服务器上运行一个Redis + 哨兵

```
       +----+
       | M1 |
       | S1 |
       +----+
          |
+----+    |    +----+
| R2 |----+----| R3 |
| S2 |         | S3 |
+----+         +----+
```

为什么redis哨兵集群只有1个或2个节点无法正常工作？

如果是一个的话，哨兵down了,如果master 再down了，就完了呀。这不是高可用

如果是两个的话，如果一个服务器宕机了，上面的redis和master都down了，只剩下一台服务器，虽然quorum=1满足，但是majority不满足，也不能主备切换。（2的majority=2，3的majority=2，4的majority=2，5的majority=3）

---

1、两种数据丢失的情况

主备切换的过程，可能会导致数据丢失

（1）异步复制导致的数据丢失

因为master -> slave的复制是异步的，所以可能有部分数据还没复制到slave，master就宕机了，此时这些部分数据就丢失了

（2）脑裂导致的数据丢失

脑裂，也就是说，某个master所在机器突然脱离了正常的网络，跟其他slave机器不能连接，但是实际上master还运行着

此时哨兵可能就会认为master宕机了，然后开启选举，将其他slave切换成了master

这个时候，集群里就会有两个master，也就是所谓的脑裂

此时虽然某个slave被切换成了master，但是可能client还没来得及切换到新的master，还继续写向旧master的数据可能也丢失了

因此旧master再次恢复的时候，会被作为一个slave挂到新的master上去，自己的数据会清空，重新从新的master复制数据

2、解决异步复制和脑裂导致的数据丢失

```bash
min-replicas-to-write 3
min-replicas-max-lag 10
```

要求至少有1个slave，数据复制和同步的延迟不能超过10秒

如果说一旦所有的slave，数据复制和同步的延迟都超过了10秒钟，那么这个时候，master就不会再接收任何请求了

上面两个配置可以减少异步复制和脑裂导致的数据丢失

（1）减少异步复制的数据丢失

有了min-slaves-max-lag这个配置，就可以确保说，一旦slave复制数据和ack延时太长，就认为可能master宕机后损失的数据太多了，那么就拒绝写请求，这样可以把master宕机时由于部分数据未同步到slave导致的数据丢失降低的可控范围内

（2）减少脑裂的数据丢失

如果一个master出现了脑裂，跟其他slave丢了连接，那么上面两个配置可以确保说，如果不能继续给指定数量的slave发送数据，而且slave超过10秒没有给自己ack消息，那么就直接拒绝客户端的写请求

这样脑裂后的旧master就不会接受client的新数据，也就避免了数据丢失

上面的配置就确保了，如果跟任何一个slave丢了连接，在10秒后发现没有slave给自己ack，那么就拒绝新的写请求

因此在脑裂场景下，最多就丢失10秒的数据

如果有大量数据涌进来怎么办？

一般来说，会在client做`降级`，写到本地磁盘里面，在client对外接收请求的请求，再做降级，`做限流`，减慢请求涌入的速度。或者client可能会采取将数据临时`灌入一个Kafka消息队列`，每个10分钟去队列里面取一次，尝试重新发挥master。

1、sdown和odown转换机制

sdown和odown两种失败状态

sdown是主观宕机，就一个哨兵如果自己觉得一个master宕机了，那么就是主观宕机

odown是客观宕机，如果quorum数量的哨兵都觉得一个master宕机了，那么就是客观宕机

sdown达成的条件很简单，如果一个哨兵ping一个master，超过了is-master-down-after-milliseconds指定的毫秒数之后，就主观认为master宕机

sdown到odown转换的条件很简单，如果一个哨兵在指定时间内，收到了quorum指定数量的其他哨兵也认为那个master是sdown了，那么就认为是odown了，客观认为master宕机

2、哨兵集群的自动发现机制

哨兵互相之间的发现，是通过redis的pub/sub系统实现的，每个哨兵都会往__sentinel__:hello这个channel里发送一个消息，这时候所有其他哨兵都可以消费到这个消息，并感知到其他的哨兵的存在

每隔两秒钟，每个哨兵都会往自己监控的某个master+slaves对应的__sentinel__:hello channel里发送一个消息，内容是自己的host、ip和runid还有对这个master的监控配置

每个哨兵也会去监听自己监控的每个master+slaves对应的__sentinel__:hello channel，然后去感知到同样在监听这个master+slaves的其他哨兵的存在

每个哨兵还会跟其他哨兵交换对master的监控配置，互相进行监控配置的同步

3、slave配置的自动纠正

哨兵会负责自动纠正slave的一些配置，比如slave如果要成为潜在的master候选人，哨兵会确保slave在复制现有master的数据; 如果slave连接到了一个错误的master上，比如故障转移之后，那么哨兵会确保它们连接到正确的master上

4、slave->master选举算法

如果一个master被认为odown了，而且majority哨兵都允许了主备切换，那么某个哨兵就会执行主备切换操作，此时首先要选举一个slave来

会考虑slave的一些信息

（1）跟master断开连接的时长
（2）slave优先级
（3）复制offset
（4）run id

如果一个slave跟master断开连接已经超过了down-after-milliseconds的10倍，外加master宕机的时长，那么slave就被认为不适合选举为master

(down-after-milliseconds * 10) + milliseconds_since_master_is_in_SDOWN_state

接下来会对slave进行排序

（1）按照slave优先级进行排序，slave priority越低，优先级就越高
（2）如果slave priority相同，那么看replica offset，哪个slave复制了越多的数据，offset越靠后，优先级就越高
（3）如果上面两个条件都相同，那么选择一个run id比较小的那个slave

5、quorum和majority

每次一个哨兵要做主备切换，首先需要quorum数量的哨兵认为odown，然后选举出一个哨兵来做切换，这个哨兵还得得到majority哨兵的授权，才能正式执行切换

如果quorum < majority，比如5个哨兵，majority就是3，quorum设置为2，那么就3个哨兵授权就可以执行切换

但是如果quorum >= majority，那么必须quorum数量的哨兵都授权，比如5个哨兵，quorum是5，那么必须5个哨兵都同意授权，才能执行切换

6、configuration epoch

哨兵会对一套redis master+slave进行监控，有相应的监控的配置

执行切换的那个哨兵，会从要切换到的新master（salve->master）那里得到一个configuration epoch，这就是一个version号，每次切换的version号都必须是唯一的

如果第一个选举出的哨兵切换失败了，那么其他哨兵，会等待failover-timeout时间，然后接替继续执行切换，此时会重新获取一个新的configuration epoch，作为新的version号

7、configuraiton传播

哨兵完成切换之后，会在自己本地更新生成最新的master配置，然后同步给其他的哨兵，就是通过之前说的pub/sub消息机制

这里之前的version号就很重要了，因为各种消息都是通过一个channel去发布和监听的，所以一个哨兵完成一次新的切换之后，新的master配置是跟着新的version号的

其他的哨兵都是根据版本号的大小来更新自己的master配置的

### Redis持久化的意义

redis持久化的意义，在于`故障恢复`。如果通过持久化将数据搞一份儿在磁盘上去，然后定期比如说同步和备份到`一些云存储服务上去`，那么就可以保证数据不丢失全部，还是可以恢复一部分数据回来的

AOF,RDB持久化：

如果我们想要redis仅仅作为`纯内存的缓存`来用，那么可以禁止RDB和AOF所有的持久化机制

通过RDB或AOF，都可以将redis内存中的数据给`持久化到磁盘上面来`，然后可以将这些数据备份到别的地方去，比如说阿`里云，云服务`

如果redis挂了，服务器上的内存和磁盘上的数据都丢了，可以从云服务上拷贝回来之前的数据，放到指定的目录中，然后重新启动redis，redis就会自动根据持久化数据文件中的数据，去恢复内存中的数据，继续对外提供服务

如果同时使用`RDB和AOF两种持久化机制，那么在redis重启的时候，会使用AOF来重新构建数据，因为AOF中的数据更加完整`

2、RDB持久化机制的优点

（1）RDB对redis对外提供的读写服务，影响非常小，`可以让redis保持高性能`，因为redis主进程只需要fork一个子进程，让子进程执行磁盘IO操作来进行RDB持久化即可

（2）RDB会生成多个数据文件，每个数据文件都代表了某一个时刻中redis的数据，这种多个数据文件的方式，非常适合`做冷备`，可以将这种完整的数据文件发送到一些远程的安全存储上去，比如说Amazon的S3`云服务`上去，在国内可以是阿里云的ODPS分布式存储上，以预定好的备份策略来定期备份redis中的数据

（3）相对于AOF持久化机制来说，直接基于`RDB数据文件来重启和恢复redis进程，更加快速`

3、缺点

可能会造成太多的数据丢失。RDB数据备份的时间：

```text
save 900 1
save 300 10
save 60 10000
```

4、AOF的优点

AOF可以更好的保护数据不丢失，一般AOF会每隔1秒，通过一个后台线程执行一次`fsync操作`，最多丢失1秒钟的数据

5、AOF的缺点

1. 对于同一份数据来说，AOF日志文件通常比RDB数据`快照文件更大`
2. AOF开启后，支持的写QPS会比RDB支持的写QPS低

AOF的rewrite：如果AOF的文件过大，会再次根据Redis内存中的数据，在生成新的AOF文件。

6、总和选择

用AOF来保证数据不丢失，作为数据恢复的第一选择; 用RDB来做不同程度的冷备，在AOF文件都丢失或损坏不可用的时候，还可以使用RDB来进行快速的数据恢复

### 了解什么是redis的雪崩和穿透，如何解决

雪崩就是某些情况下Redis全部失效，大量的数据打在数据库上。

穿透是恶意用户攻击数据库，使用大量数据库没有的key，绕过redis，查询数据库

**缓存雪崩的事前事中事后的解决方案：**

1. Redis一定要做高可用，使用主从复制、读写分离，再加上哨兵模式
2. Redis要做持久化，以便在Redis挂掉之后可以快速的恢复
3. 系统层面，可以使用ehcache小缓存，同时使用hystrix限流

![02_如何解决缓存雪崩.png](../../_img/02_如何解决缓存雪崩.png)

**穿透的解决方案：**

1. 查询数据库，查询不到数据也在缓存中设置空值
2. 布隆过滤器

![03_缓存穿透现象以及解决方案.png](../../_img/03_缓存穿透现象以及解决方案.png)

### 布隆过滤器

**布隆过滤器：**

![布隆过滤器-hash运算.png](../../_img/布隆过滤器-hash运算.png)

布隆过滤器的本质上是“位(bit)数组”。当一个元素加入布隆过滤器的时候，会进行如下操作：

1. 使用布隆过滤器中的哈希函数对元素值进行计算，得到哈希值（有几个哈希函数就有几个哈希值）。假设元素经过布隆过滤器计算后的值为10
2. 假设布隆过滤器的位数组为bf[], 根据得到的哈希值10，将`bf[10]`置为1

当我们需要判断一个元素是否存在于布隆过滤器的时候，会进行如下操作：

1. 对给定元素再次进行相同的哈希计算
2. 得到值之后判断位数组中的每个元素是否都为1，如果都为1，那么说明这个值可能在布隆过滤器中，运行通过。如果为0，说明这个元素一定不在布隆过滤器中，不允许通过。

> <https://github.com/Snailclimb/JavaGuide/blob/master/docs/dataStructures-algorithms/data-structure/bloom-filter.md>

使用Google

```java

        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>28.0-jre</version>
        </dependency>
```
```java
        // 创建布隆过滤器对象
        BloomFilter<Integer> filter = BloomFilter.create(
                Funnels.integerFunnel(),
                1500,
                0.01);
        // 判断指定元素是否存在
        System.out.println(filter.mightContain(1));
        System.out.println(filter.mightContain(2));
        // 将元素添加进布隆过滤器
        filter.put(1);
        filter.put(2);
        System.out.println(filter.mightContain(1));
        System.out.println(filter.mightContain(2));
```

Redis中

```bash
127.0.0.1:6379> BF.ADD myFilter java
(integer) 1
127.0.0.1:6379> BF.ADD myFilter javaguide
(integer) 1
127.0.0.1:6379> BF.EXISTS myFilter java
(integer) 1
127.0.0.1:6379> BF.EXISTS myFilter javaguide
(integer) 1
127.0.0.1:6379> BF.EXISTS myFilter github
(integer) 0
```
