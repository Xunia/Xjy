<!-- TOC -->

- [**概述**](#概述)
- [**1. Redis的数据结构和相关常用命令**](#1-redis的数据结构和相关常用命令)
    - [1.1 Key](#11-key)
    - [1.2 String](#12-string)
    - [1.3 List](#13-list)
    - [1.4 Hash](#14-hash)
    - [1.5 Set](#15-set)
    - [1.6 Sorted Set](#16-sorted-set)
    - [1.6 Bitmap和HyperLogLog](#16-bitmap和hyperloglog)
    - [1.7 其他常用命令](#17-其他常用命令)
- [2. **数据持久化**](#2-数据持久化)
    - [2.1 RDB](#21-rdb)
    - [2.2 AOF](#22-aof)
- [3. **内存管理与数据淘汰机制**](#3-内存管理与数据淘汰机制)
- [4. **Pipelining**](#4-pipelining)
- [5. **事务与Scripting**](#5-事务与scripting)
- [6. **Redis性能调优**](#6-redis性能调优)
- [7. **主从复制与集群分片**](#7-主从复制与集群分片)

<!-- /TOC -->

## **概述**
* Redis是一个开源的，基于内存的结构化数据存储媒介，可以作为数据库、缓存服务或消息服务使用。
* Redis支持多种数据结构，包括字符串、哈希表、链表、集合、有序集合、位图、Hyperloglogs等。
* Redis具备LRU淘汰、事务实现、以及不同级别的硬盘持久化等能力，支持副本集和通过Redis Sentinel实现的高可用方案，同时还支持通过Redis Cluster实现的数据自动分片能力。
* Redis的主要功能都基于单线程模型实现，同时Redis采用了非阻塞式IO，并精细地优化各种命令的算法时间复杂度，这些信息意味着：
    + Redis是线程安全的（因为只有一个线程），其所有操作都是原子的，不会因并发产生数据异常
    + Redis的速度非常快（因为使用非阻塞式IO，且大部分命令的算法时间复杂度都是O(1))
    + 使用高耗时的Redis命令是很危险的，会占用唯一的一个线程的大量处理时间，导致所有的请求都被拖慢。
## **1. Redis的数据结构和相关常用命令**
### 1.1 Key
Redis采用Key-Value型的基本数据结构，
关于Key的一些注意事项：
* 不要使用过长的Key。会消耗更多的内存，还会导致查找的效率降低
* Key短也是不好的，可读性和可维护性上的考虑
* 最好使用统一的规范来设计Key
* Redis允许的最大Key长度是512MB（对Value的长度限制也是512MB）
### 1.2 String
String是Redis的基础数据类型，Redis没有Int、Float、Boolean等数据类型的概念，所有的基本类型在Redis中都以String体现。  

与String相关的常用命令：
* **SET**：为一个key设置value，可以配合EX/PX参数指定key的有效期，通过NX/XX参数针对key是否存在的情况进行区别操作，时间复杂度O(1)
* GET：获取某个key对应的value，时间复杂度O(1)
* GETSET：为一个key设置value，并返回该key的原value，时间复杂度O(1)
* MSET：为多个key设置value，时间复杂度O(N)
* MSETNX：同MSET，如果指定的key中有任意一个已存在，则不进行任何操作，时间复杂度O(N)
* MGET：获取多个key对应的value，时间复杂度O(N)  

把String作为整型或浮点型数字来使用，只对可以转换为整型的String数据起作用，主要体现在INCR、DECR类的命令上：
* INCR：value值自增1，并返回自增后的值。时间复杂度O(1)
* INCRBY：value值自增指定的整型数值，并返回自增后的值。。时间复杂度O(1)
* DECR/DECRBY：同INCR/INCRBY，自增改为自减。  

INCR/DECR系列命令要求操作的value类型为String，必须在[-2^63 ~ 2^63 - 1]范围内。
使用场景：
+ 例1：库存控制，在高并发场景下实现库存余量的精准校验，确保不出现超卖的情况。
+ 例2：自增序列生成，生成一系列唯一的序列号

### 1.3 List
Redis的List是**链表型**的数据结构。
与List相关的常用命令：
* LPUSH：向List的左侧（即头部）插入1个或多个元素，返回插入后的List长度。时间复杂度O(N)
* RPUSH：同LPUSH，向List的右侧（即尾部）插入1或多个元素
* LPOP：从List的左侧（即头部）移除一个元素并返回，时间复杂度O(1)
* RPOP：同LPOP，从指定List的右侧（即尾部）移除1个元素并返回
* LPUSHX/RPUSHX：与LPUSH/RPUSH类似，区别在于，LPUSHX/RPUSHX操作的key如果不存在，则不会进行任何操作
* LLEN：返回指定List的长度，时间复杂度O(1)
* LRANGE：返回指定List中指定范围的元素（双端包含，即LRANGE key 0 10会返回11个元素），时间复杂度O(N)。应尽可能控制一次获取的元素数量，一次获取过大范围的List元素会导致延迟，同时对长度不可预知的List，避免使用LRANGE key 0 -1这样的完整遍历操作。

应谨慎使用的List相关命令：
* LINDEX：返回指定List指定index上的元素，如果index越界，返回nil。index数值是回环的，即-1代表List最后一个位置，-2代表List倒数第二个位置。时间复杂度O(N)
* LSET：将指定List指定index上的元素设置为value，如果index越界则返回错误，时间复杂度O(N)，如果操作的是头/尾部的元素，则时间复杂度为O(1)
* LINSERT：向指定List中指定元素之前/之后插入一个新元素，并返回操作后的List长度。如果指定的元素不存在，返回-1。如果指定key不存在，不会进行任何操作，时间复杂度O(N)
* BLPOP/BRPOP等，能够实现类似于BlockingQueue的能力，即在List为空时，阻塞该连接，直到List中有对象可以出队时再返回。
### 1.4 Hash
Hash即哈希表，可以理解成将HashMap搬入Redis。  
Hash非常适合用于表现对象类型的数据，用Hash中的field对应对象的field即可。  
Hash的优点包括：
* 可以实现二元查找，如"查找ID为1000的用户的年龄"
* 比起将整个对象序列化后作为String存储的方法，Hash能够有效地减少网络传输的消耗
* 当使用Hash维护一个集合时，提供了比List效率高得多的随机访问命令

与Hash相关的常用命令：
* HSET：将key对应的Hash中的field设置为value。如果该Hash不存在，会自动创建一个。时间复杂度O(1)
* HGET：返回指定Hash中field字段的值，时间复杂度O(1)
* HMSET/HMGET：同HSET和HGET，可以批量操作同一个key下的多个field，时间复杂度：O(N)，N为一次操作的field数量
* HSETNX：同HSET，但如field已经存在，HSETNX不会进行任何操作，时间复杂度O(1)
* HEXISTS：判断指定Hash中field是否存在，存在返回1，不存在返回0，时间复杂度O(1)
* HDEL：删除指定Hash中的field（1个或多个），时间复杂度：O(N)，N为操作的field数量
* HINCRBY：同INCRBY命令，对指定Hash中的一个field进行INCRBY，时间复杂度O(1)
应谨慎使用的Hash相关命令：
* HGETALL：返回指定Hash中所有的field-value对。返回结果为数组，数组中field和value交替出现。时间复杂度O(N)
* HKEYS/HVALS：返回指定Hash中所有的field/value，时间复杂度O(N)  

### 1.5 Set
Redis Set是无序的，不可重复的String集合。
与Set相关的常用命令：
* SADD：向指定Set中添加1个或多个member，如果指定Set不存在，会自动创建一个。时间复杂度O(N)，N为添加的member个数
* SREM：从指定Set中移除1个或多个member，时间复杂度O(N)，N为移除的member个数
* SRANDMEMBER：从指定Set中随机返回1个或多个member，时间复杂度O(N)，N为返回的member个数
* SPOP：从指定Set中随机移除并返回count个member，时间复杂度O(N)，N为移除的member个数
* SCARD：返回指定Set中的member个数，时间复杂度O(1)
* SISMEMBER：判断指定的value是否存在于指定Set中，时间复杂度O(1)
* SMOVE：将指定member从一个Set移至另一个Set  

慎用的Set相关命令：
* SMEMBERS：返回指定Hash中所有的member，时间复杂度O(N)
* SUNION/SUNIONSTORE：计算多个Set的并集并返回/存储至另一个Set中，时间复杂度O(N)，N为参与计算的所有集合的总member数
* SINTER/SINTERSTORE：计算多个Set的交集并返回/存储至另一个Set中，时间复杂度O(N)，N为参与计算的所有集合的总member数
* SDIFF/SDIFFSTORE：计算1个Set与1或多个Set的差集并返回/存储至另一个Set中，时间复杂度O(N)，N为参与计算的所有集合的总member数

### 1.6 Sorted Set
Redis Sorted Set是有序的、不可重复的String集合。Sorted Set中的每个元素都需要指派一个分数(score)，Sorted Set会根据score对元素进行升序排序。如果多个member拥有相同的score，则以字典序进行升序排序。
Sorted Set非常适合用于实现排名。 

Sorted Set的主要命令：
* ZADD：向指定Sorted Set中添加1个或多个member，时间复杂度O(Mlog(N))，M为添加的member数量，N为Sorted Set中的member数量
* ZREM：从指定Sorted Set中删除1个或多个member，时间复杂度O(Mlog(N))，M为删除的member数量，N为Sorted Set中的member数量
* ZCOUNT：返回指定Sorted Set中指定score范围内的member数量，时间复杂度：O(log(N))
* ZCARD：返回指定Sorted Set中的member数量，时间复杂度O(1)
* ZSCORE：返回指定Sorted Set中指定member的score，时间复杂度O(1)
* ZRANK/ZREVRANK：返回指定member在Sorted Set中的排名，ZRANK返回按升序排序的排名，ZREVRANK则返回按降序排序的排名。时间复杂度O(log(N))
* ZINCRBY：同INCRBY，对指定Sorted Set中的指定member的score进行自增，时间复杂度O(log(N))
慎用的Sorted Set相关命令：
* ZRANGE/ZREVRANGE：返回指定Sorted Set中指定排名范围内的所有member，ZRANGE为按score升序排序，ZREVRANGE为按score降序排序，时间复杂度O(log(N)+M)，M为本次返回的member数
* ZRANGEBYSCORE/ZREVRANGEBYSCORE：返回指定Sorted Set中指定score范围内的所有member，返回结果以升序/降序排序，min和max可以指定为-inf和+inf，代表返回所有的member。时间复杂度O(log(N)+M)
* ZREMRANGEBYRANK/ZREMRANGEBYSCORE：移除Sorted Set中指定排名范围/指定score范围内的所有member。时间复杂度O(log(N)+M)  

上述几个命令，应尽量避免传递[0 -1]或[-inf +inf]这样的参数，来对Sorted Set做一次性的完整遍历，特别是在Sorted Set的尺寸不可预知的情况下。可以通过ZSCAN命令来进行游标式的遍历，或通过LIMIT参数来限制返回member的数量（适用于ZRANGEBYSCORE和ZREVRANGEBYSCORE命令），以实现游标式的遍历。

### 1.6 Bitmap和HyperLogLog
Bitmap在Redis中不是一种实际的数据类型，而是一种将String作为Bitmap使用的方法。可以理解为将String转换为bit数组。使用Bitmap来存储true/false类型的简单数据极为节省空间。

HyperLogLogs是一种主要用于数量统计的数据结构，它和Set类似，维护一个不可重复的String集合，但是HyperLogLogs并不维护具体的member内容，只维护member的个数。也就是说，HyperLogLogs只能用于计算一个集合中不重复的元素数量，所以它比Set要节省很多内存空间。

### 1.7 其他常用命令
* EXISTS：判断指定的key是否存在，返回1代表存在，0代表不存在，时间复杂度O(1)
* DEL：删除指定的key及其对应的value，时间复杂度O(N)，N为删除的key数量
* EXPIRE/PEXPIRE：为一个key设置有效期，单位为秒或毫秒，时间复杂度O(1)
* TTL/PTTL：返回一个key剩余的有效时间，单位为秒或毫秒，时间复杂度O(1)
* RENAME/RENAMENX：将key重命名为newkey。使用RENAME时，如果newkey已经存在，其值会被覆盖；使用RENAMENX时，如果newkey已经存在，则不会进行任何操作，时间复杂度O(1)
* TYPE：返回指定key的类型，string, list, set, zset, hash。时间复杂度O(1)
* CONFIG GET：获得Redis某配置项的当前值，可以使用*通配符，时间复杂度O(1)
* CONFIG SET：为Redis某个配置项设置新值，时间复杂度O(1)
* CONFIG REWRITE：让Redis重新加载redis.conf中的配置

## 2. **数据持久化**
Redis的数据持久化机制是可以关闭的。但通常来说，仍然建议至少开启RDB方式的数据持久化，因为：
* RDB方式的持久化几乎不损耗Redis本身的性能，在进行RDB持久化时，Redis主进程唯一需要做的事情就是fork出一个子进程，所有持久化工作都由子进程完成
* Redis无论因为什么原因crash掉之后，重启时能够自动恢复到上一次RDB快照中记录的数据。这省去了手工从其他数据源（如DB）同步数据的过程，而且要比其他任何的数据恢复方式都要快
* 现在硬盘那么大，真的不缺那一点地方

### 2.1 RDB
采用RDB持久方式，Redis会定期保存数据快照至一个rbd文件中，并在启动时自动加载rdb文件，恢复之前保存的数据。可以在配置文件中配置Redis进行快照保存的时机：
```
save [seconds] [changes]
意为在[seconds]秒内如果发生了[changes]次数据修改，则进行一次RDB快照保存，例如
save 60 100
会让Redis每60秒检查一次数据变更情况，如果发生了100次或以上的数据变更，则进行RDB快照保存。
可以配置多条save指令，让Redis执行多级的快照保存策略。
Redis默认开启RDB快照，默认的RDB策略如下：
save 900 1
save 300 10
save 60 10000
也可以通过BGSAVE命令手工触发RDB快照保存。
```
RDB的优点：
* 对性能影响最小。如前文所述，Redis在保存RDB快照时会fork出子进程进行，几乎不影响Redis处理客户端请求的效率。
* 每次快照会生成一个完整的数据快照文件，所以可以辅以其他手段保存多个时间点的快照（例如把每天0点的快照备份至其他存储媒介中），作为非常可靠的灾难恢复手段。
* 使用RDB文件进行数据恢复比使用AOF要快很多。
RDB的缺点：
* 快照是定期生成的，所以在Redis crash时或多或少会丢失一部分数据。
* 如果数据集非常大且CPU不够强（比如单核CPU），Redis在fork子进程时可能会消耗相对较长的时间（长至1秒），影响这期间的客户端请求。
### 2.2 AOF
采用AOF持久方式时，Redis会把每一个写请求都记录在一个日志文件里。在Redis重启时，会把AOF文件中记录的所有写操作顺序执行一遍，确保数据恢复到最新。  
AOF默认是关闭的，如要开启，进行如下配置：  
appendonly yes  
AOF提供了三种fsync配置，always/everysec/no，通过配置项[appendfsync]指定：
* appendfsync no：不进行fsync，将flush文件的时机交给OS决定，速度最快
* appendfsync always：每写入一条日志就进行一次fsync操作，数据安全性最高，但速度最慢
* appendfsync everysec：折中的做法，交由后台线程每秒fsync一次  

随着AOF不断地记录写操作日志，必定会出现一些无用的日志，例如某个时间点执行了命令SET key1 "abc"，在之后某个时间点又执行了SET key1 "bcd"，那么第一条命令很显然是没有用的。大量的无用日志会让AOF文件过大，也会让数据恢复的时间过长。 

所以Redis提供了AOF rewrite功能，可以重写AOF文件，只保留能够把数据恢复到最新状态的最小写操作集。
AOF rewrite可以通过BGREWRITEAOF命令触发，也可以配置Redis定期自动进行： 

auto-aof-rewrite-percentage 100  
auto-aof-rewrite-min-size 64mb  
上面两行配置的含义是，Redis在每次AOF rewrite时，会记录完成rewrite后的AOF日志大小，当AOF日志大小在该基础上增长了100%后，自动进行AOF rewrite。同时如果增长的大小没有达到64mb，则不会进行rewrite。   
 
AOF的优点：
* 最安全，在启用appendfsync always时，任何已写入的数据都不会丢失，使用在启用appendfsync everysec也至多只会丢失1秒的数据。
* AOF文件在发生断电等问题时也不会损坏，即使出现了某条日志只写入了一半的情况，也可以使用redis-check-aof工具轻松修复。
* AOF文件易读，可修改，在进行了某些错误的数据清除操作后，只要AOF文件没有rewrite，就可以把AOF文件备份出来，把错误的命令删除，然后恢复数据。  

AOF的缺点：
* AOF文件通常比RDB文件更大
* 性能消耗比RDB高
* 数据恢复速度比RDB慢


## 3. **内存管理与数据淘汰机制**
最大内存设置
默认情况下，在32位OS中，Redis最大使用3GB的内存，在64位OS中则没有限制。
在使用Redis时，应该对数据占用的最大空间有一个基本准确的预估，并为Redis设定最大使用的内存。否则在64位OS中Redis会无限制地占用内存（当物理内存被占满后会使用swap空间），容易引发各种各样的问题。
通过如下配置控制Redis使用的最大内存：
maxmemory 100mb
在内存占用达到了maxmemory后，再向Redis写入数据时，Redis会：
* 根据配置的数据淘汰策略尝试淘汰数据，释放空间
* 如果没有数据可以淘汰，或者没有配置数据淘汰策略，那么Redis会对所有写请求返回错误，但读请求仍然可以正常执行
在为Redis设置maxmemory时，需要注意：
* 如果采用了Redis的主从同步，主节点向从节点同步数据时，会占用掉一部分内存空间，如果maxmemory过于接近主机的可用内存，导致数据同步时内存不足。所以设置的maxmemory不要过于接近主机可用的内存，留出一部分预留用作主从同步。
数据淘汰机制

Redis提供了5种数据淘汰策略：
* volatile-lru：使用LRU算法进行数据淘汰（淘汰上次使用时间最早的，且使用次数最少的key），只淘汰设定了有效期的key
* allkeys-lru：使用LRU算法进行数据淘汰，所有的key都可以被淘汰
* volatile-random：随机淘汰数据，只淘汰设定了有效期的key
* allkeys-random：随机淘汰数据，所有的key都可以被淘汰
* volatile-ttl：淘汰剩余有效期最短的key  

最好为Redis指定一种有效的数据淘汰策略以配合maxmemory设置，避免在内存使用满后发生写入失败的情况。
一般来说，推荐使用的策略是volatile-lru，并辨识Redis中保存的数据的重要性。对于那些重要的，绝对不能丢弃的数据（如配置类数据等），应不设置有效期，这样Redis就永远不会淘汰这些数据。对于那些相对不是那么重要的，并且能够热加载的数据（比如缓存最近登录的用户信息，当在Redis中找不到时，程序会去DB中读取），可以设置上有效期，这样在内存不够时Redis就会淘汰这部分数据。
配置方法：  
maxmemory-policy volatile-lru   #默认是noeviction，即不进行数据淘汰  

## 4. **Pipelining**
实现在一次交互中执行多条命令。

Pipelining的局限性：Pipelining只能用于执行连续且无相关性的命令，当某个命令的生成需要依赖于前一个命令的返回时，就无法使用Pipelining了。

通过Scripting功能，可以规避这一局限性

## 5. **事务与Scripting**
WATCH的机制是：在事务EXEC命令执行时，Redis会检查被WATCH的key，只有被WATCH的key从WATCH起始时至今没有发生过变更，EXEC才会被执行。如果WATCH的key在WATCH命令到EXEC命令之间发生过变化，则EXEC命令会返回失败。

## 6. **Redis性能调优**
尽管Redis是一个非常快速的内存数据存储媒介，也并不代表Redis不会产生性能问题。
前文中提到过，Redis采用单线程模型，所有的命令都是由一个线程串行执行的，所以当某个命令执行耗时较长时，会拖慢其后的所有命令，这使得Redis对每个任务的执行效率更加敏感。
针对Redis的性能优化，主要从下面几个层面入手：
* 最初的也是最重要的，确保没有让Redis执行耗时长的命令
* 使用pipelining将连续执行的命令组合执行
* 操作系统的Transparent huge pages功能必须关闭：
echo never > /sys/kernel/mm/transparent_hugepage/enabled
* 如果在虚拟机中运行Redis，可能天然就有虚拟机环境带来的固有延迟。可以通过./redis-cli --intrinsic-latency 100命令查看固有延迟。同时如果对Redis的性能有较高要求的话，应尽可能在物理机上直接部署Redis。
* 检查数据持久化策略
* 考虑引入读写分离机制
长耗时命令

避免在使用这些O(N)命令时发生问题主要有几个办法：
* 不要把List当做列表使用，仅当做队列来使用
* 通过机制严格控制Hash、Set、Sorted Set的大小
* 可能的话，将排序、并集、交集等操作放在客户端执行
* 绝对禁止使用KEYS命令
* 避免一次性遍历集合类型的所有成员，而应使用SCAN类的命令进行分批的，游标式的遍历
Redis提供了SCAN命令，可以对Redis中存储的所有key进行游标式的遍历，避免使用KEYS命令带来的性能问题。
网络引发的延迟
* 尽可能使用长连接或连接池，避免频繁创建销毁连接
* 客户端进行的批量数据操作，应使用Pipeline特性在一次交互中完成。具体请参照本文的Pipelining章节
数据持久化引发的延迟
Redis的数据持久化工作本身就会带来延迟，需要根据数据的安全级别和性能要求制定合理的持久化策略：
* AOF + fsync always的设置虽然能够绝对确保数据安全，但每个操作都会触发一次fsync，会对Redis的性能有比较明显的影响
* AOF + fsync every second是比较好的折中方案，每秒fsync一次
* AOF + fsync never会提供AOF持久化方案下的最优性能
* 使用RDB持久化通常会提供比使用AOF更高的性能，但需要注意RDB的策略配置
* 每一次RDB快照和AOF Rewrite都需要Redis主进程进行fork操作。fork操作本身可能会产生较高的耗时，与CPU和Redis

## 7. **主从复制与集群分片**
主从复制  
借助Redis的主从复制，可以实现读写分离和高可用：
* 实时性要求不是特别高的读请求，可以在Slave上完成，提升效率。
* 借助Redis Sentinel可以实现高可用，当Master crash后，Redis Sentinel能够自动将一个Slave晋升为Master，继续提供服务

集群分片（Redis Cluster）  
* 能够自动将数据分散在多个节点上
* 当访问的key不在当前分片上时，能够自动将请求转发至正确的分片
* 当集群中部分节点失效时仍能提供服务  