# redis

推荐阅读[《redis源码日志》](http://daoluan.net/redis-source-notes)


1. **什么是redis?**

    Redis是一个基于内存的高性能key-value数据库

2. **Redis的特点？**

    Redis本质上是一个key-value类型的内存数据库，类似memcached，整个数据库统统加载在内存当中进行操作，定期通过异步操作把数据库数据刷新到硬盘上进行保存。因为纯内存操作，redis性能非常出色。
    
    Reids最大的魅力是支持保存多种数据结构，此外但个value的最大限制是1GB，不像memcached只能保存1MB的数据。

    Redis的主要缺点是数据库容量容易受到物理内存的限制，不能用作海量数据的高性能读写，因此redis适合的场景主要局限在较小数据量的高性能操作和运算上。

3. **使用redis有哪些好处？**

    * 速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)。

    * 支持丰富的数据类型，支持string,list,set,sorted set,hash

    * 支持事务，操作都是原子性

    * 丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除

4. **redis 数据结构？**

    1. String
    
        常用命令：set/get/decr/incr/mget等

        应用场景：String是最常用的一种数据类型，普通的key/value存储都可以归为此类

        实现方式：String在redis内存存储默认就是一个字符串，被redisObject所引用，但遇到incr、decr等操作时会转成数值型进行计算，此时redisObject的encoding字段为int。

    2. Hash

        常用命令：hget/hset/hgetall

        应用场景：我们要存储一个用户信息对象数据，其中包括用户ID、用户姓名、年龄和生日，通过用户ID我们希望获取该用户的姓名或者年龄生日等。

        实现方式：redis的Hash实际是内部存储的alue为一个HashMap，并提供了直接存取这个Map成员的接口。key是用户ID，value是一个Map。这个Map的key是成员的属性名，value是属性值。这样对数据的修改和存储都可以直接通过其内部Map的key（redis里称内部Map的key为field），也就是通过key（用户ID）+ field（属性标签）就可以操作对应属性数据。

        当前HashMap的实现方式有两种，当HashMap的成员比较少时，redis为了节省内存会采用类似一维数组的方式紧凑存储，而不会采用真正的Hashap结构，这时对应的alue的redisObject的encoding为zipmap，当成员数量增大时会自动转成真正的HashMap，此时encoding为int。

    3. List

        常用命令：lpush/rpush/lpop/rpop/lrange等

        应用场景：redis list的应用场景非常多，也是redis最重要的数据结构之一，比如twitter的关注列表，粉丝列表都可以用redis的list结构实现；

        实现方式：list的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开小，redis内部的很多实现，包括发送缓冲队列等也都是可以用这个数据结构。

    4. Set

        常用命令：sadd/smembers/sunion等

        应用场景：set对外提供的功能与list类似，是一个列表的功能，特殊之处在于set可以自动排重，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员时候在一个set集合内的重要接口。

    5. Sorted Set

        常用命令：zadd/zrange/zrem/zcard等

        应用场景：sorted set的使用场景与set类似，区别时set不是自动有序的，sorted set可以通过用户额外提供一个优先级的参数score来为成员排序，并且是插入有序，即自动排序。当你需要一个有序的并且不重复的集合列表，那么可以选择sorted set数据结构，比如twitter的public timeline可以以发表时间为score来存储，这样获取时就是自动按时间排序的。

        实现方式：内部使用HashMap和跳跃表来保证数据的存储和有序，HashMap里放的是成员到score的映射，而跳跃表里存放的是所有的成员，排序依据是HashMap里存的socre，使用跳跃表的结构可以获得比较高的查找效率，并且在实现上比较简单。
    
5. **redis适用于的场景？**

    redis最适合所有数据in-memery的场景，如：

    1. 会话缓存

        最常用的一种使用redis的情景是会话缓存，用redis缓存会话比其它存储(如memcached)的优势在于：redis提供持久化。

    2. 全页缓存

        除基本的会话token之外，redis还提供很简单的FPC平台。回到一致性问题，即使重启了redis实例，因为有磁盘的持久化，用户也不会看到页面加载速度的下降，这是一个极大的改进，类似PHP本地FPC。

    3. 队列
        redis在内存存储引擎领域的一大优点是提供list和set操作，这使得redis能作为一个很好的消息队列平台来使用。redis作为队列使用的操作，就类似于python对list的push/pop操作。

    4. 排行榜/计数器
        
        redis在内存中对数字进行递增或递减的操作实现的非常好。集合set和有序集合sorted set也使得我们在执行这些操作时变得非常简单。

6. **redis是单进程单线程的？**

    redis利用队列技术键并发访问变为串行访问，消除了传统数据库串行控制带来的开销

7. **redis和memcached区别？**

    * redis支持更丰富的数据类型：redis不仅支持简单的k/v类型的数据，同时还提供list、set、zset、hash等数据结构。memcached只支持简单的字符串类型。

    * redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用，而memcached把所有数据全部存在内存之中。

    * 集群模式：memcached没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；但是redis目前是原生支持cluster模式的。

    * memcached是多线程，非阻塞IO复用的网络模型；redis使用单线程的多路IO复用模型。

8. **redis持久化**

    为防止数据丢失，需要将redis中的数据从内存dump到磁盘，这就是持久化。reids提供两种持久化方式：RDB、AOF。redis允许两者结合，也允许两者同时关闭。
    
    RDB可以定时备份内存中的数据集，服务器启动的时候，可以从RDB文件中恢复数据集。redis支持两种方式进行RDB持久化：1）当前进程 2）后台执行（BGSAVE）。BGSAVE策略是fork出一个子进程，把内存中的数据集整个dump到硬盘上。

    AOF（append only file）可以记录服务器的所有写操作，在服务器重新启动的时候，会把所有的写操作重新执行一遍，从而实现数据备份。当写操作集过大（比原有的数据集还大），redis会重写写操作。AOF持久化是类似于生成一个关于redis写操作的文件，写操作总是以追加的方式追加到文件中。