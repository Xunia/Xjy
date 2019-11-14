
## **（一）SQL思考树**
### 1 配置
1.	服务器配置
2.	MySQL配置
类型	配置	注解
基本配置	innodb_buffer_pool_size	缓冲池大小
	max_connections	连接数设置（默认151）
	innodb_log_file_size	redo日志大小
InnoDB配置		
		

1.2 Object-引擎
MyISAM：
1)	不支持事务，但是每次查询都是原子的；
2)	支持表级锁，即每次操作是对整个表加锁；
3)	存储表的总行数；
4)	一个MYISAM表有三个文件：索引文件、表结构文件、数据文件；
5)	采用非聚集索引，索引文件的数据域存储指向数据文件的指针。辅索引与主索引基本一致，但是辅索引不用保证唯一性。
6)	适用OLAP（On-Line Analytical Processing，联机分析处理）

InnoDb：
1)	支持ACID的事务，支持事务的四种隔离级别；
2)	支持行级锁及外键约束：因此可以支持写并发；
3)	不存储总行数；
4)	一个InnoDb引擎存储在一个文件空间（共享表空间，表大小不受操作系统控制，一个表可能分布在多个文件里），也有可能为多个（设置为独立表空，表大小受操作系统文件大小限制，一般为2G），受操作系统文件大小的限制；
5)	主键索引采用聚集索引（索引的数据域存储数据文件本身），辅索引的数据域存储主键的值；因此从辅索引查找数据，需要先通过辅索引找到主键值，再访问辅索引；
注意点：最好使用自增主键，防止插入数据时，为维持B+树结构，文件的大调整。
    6）适用OLTP（On-Line Transaction Processing，联机事务处理）

InnoDB主要特性：
    主要包括：插入缓存（insert buffer）、两次写(double write)、自适应哈希(Adaptive Hash index)、异步IO(Async IO)、刷新邻接页(Flush Neighbor Page)
1.3 Object-库
1.4 Object-表
两个隐藏字段（版本信息）
1.5 Object-Schema（表结构）
1.varchar的特点
        1） 存储空间不固定，根据字段长度决定。
2） 需要额外的1个或2个字节记录字符串的长度，字符串长度小于255字节使用1个字节，否则使用2个。
3） 最大长度为 65535 字节（这里单位是字节而非字符)
4） 如果列可以为null，则需要额外的一个字节作为标志。
5） 最大长度 = 字段长度 + [长度记录：(1或2) B] + [null标志位：1B]

2.char的特点
        1） 存储空间固定。
2） 长度不够时内部存储使用空格填充。
3） 若字段本身末尾存在空格，检索出来自动截断末尾空格（因为分不清空格是字段含有的还是填充产生的）。
4） 若字段本身前端存在空格，是不会截断的。
5） 当输入的字符长度超过指定长度时，char会截取超出的字符。

1.冗余数据的处理
    适当的数据冗余可以提高系统的整体查询性能
关系数据库的三范式:
    第一范式（1NF）是对关系模式的基本要求，不满足第一范式（1NF）的数据库就不是关系数据库，是指每一列都是不可分割的基本数据项，同一列中不能有多个值（数据项）；
    第二范式（2NF）要求数据库表中的每个实例或行必须可以被惟一地区分。 即各字段和主键之间不存在部分依赖
    第三范式（3NF）要求一个数据库表中不包含已在其它表中已包含的非主关键字信息。即在第二范式的基础上，不存在传递依赖 (不允许有冗余数据)
2.大表拆小表，有大数据的列单独拆成小表
3.根据需求的展示设置更合理的表结构
4.把常用属性分离成小表
  减少查询常用属性需要查询的列;
    便于常用属性的集中缓存;
1.6 Object-索引
定义：是帮助MySQL高效获取数据的数据结构。
底层原理：B+树

1.	分类
分类	分类名	注解
分类1	聚簇索引	定义：该索引中的键值的逻辑顺序决定了相应行的物理顺序
创建规则：
1. InnoDB
1）如果一个主键被定义了，那么这个主键就是作为聚集索引
2）如果没有主键被定义，那么该表的第一个唯一非空索引被作为聚集索引
3）如果没有主键也没有合适的唯一索引，那么innodb内部会生成一个隐藏的主键（6个字节，自增）作为聚集索引
2.MyISAM
1）主键也不是聚集索引。
注意点：
1.不适合修改频繁的列
2.使用的列越少越好
	非聚簇索引	定义：数据存储在一个地方，索引存储在另一个地方，索引带有指针指向数据的存储位置。
分类2	普通索引（INDEX ）	注意点：
见#分类3
	唯一索引（UNIQUE ）	注意点：
允许有一个空值
	主键索引（PRIMARY KEY）	注意点：
1.	不允许有空
2.	Myisam可以没有
	全文索引（FULLTEXT）	InnoDB8.0支持
分类3	单列索引	
	多列索引	注意点：
1）	最左前缀匹配原则
2）	覆盖索引☆

2.	使用
方式	影响Object	注释
增	数据	1.	数据量
2.	区分度☆
	索引	1.	创建索引的长度尽量短
2.	尽量扩建索引，不要建新索引，影响性能（最多6个）
增/删/改	@表-@增@删@改	1.	量大的情况处理
2.	对于经常存取的列避免建立索引
查	索引	1.	一个单位查询里只使用一个索引（where和order by中选）

注意：
由查询选择器选择最优方案
	@多列索引	
	数据	Null值判断（全表扫描）- #优：索引字段不为NULL
	关键字/操作符/函数	函数（计算符）：
1.	对索引字段进行函数（计算符）操作（全表扫描）

操作符：
1.	使用 != 或 <>（全表扫描）

关键字
1.	In 和 Not In （全表扫描），- #优：Between(连续值)/Exists
2.	Like 使用注意
		

1.7 B+ 树

可以减少查询IO
1）	无需返回上层父节点重复遍历查找减少IO操作
2）	数据存储
•	结合操作系统存储结构优化处理： mysql巧妙运用操作系统存储结构(一个节点分配到一个存储页中->尽量减少IO次数) & 磁盘预读(缓存预读->加速预读马上要用到的数据).
•	B+Tree 单个节点能放多个子节点，相同IO次数，检索出更多信息。
•	B+TREE 只在叶子节点存储数据 & 所有叶子结点包含一个链指针 & 其他内层非叶子节点只存储索引数据。只利用索引快速定位数据索引范围，先定位索引再通过索引高效快速定位数据。
3）磁盘存取原理（缺页申请 -> 读取一页（4K）或几页载入内存中）

1.8	Object-关键字/操作符/函数
1) limit 优化
1.9 InnoDB-事务
1.	ACID
•原子性(Atomicity):原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。undo log
•一致性(Consistency):如果事务执行之前数据库是一个完整性的状态,那么事务结束后,无论事务是否执行成功,数据库仍然是一个完整性状态。 (数据库的完整性状态:当一个数据库中的所有的数据都符合数据库中所定义的所有的约束,此时可以称数据库是一个完整性状态。MVCC + redo log（重做日志）
•隔离性(Isolation):事务的隔离性是指多个用户并发访问数据库时，一个用户的事务不能被其它用户的事务所干扰，多个并发事务之间数据要相互隔离。锁机制
•持久性(durability):持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响。redo log（重做日志）

2.	隔离级别及产生的问题
Read Uncommitted（读取未提交内容）
    事务可以看到其他未提交事务的执行结果。很少用于实际应用，因为它的性能也不比其他级别好多少。读取未提交的数据，也被称之为脏读（Dirty Read）。
    Read Committed（读取提交内容）
    事务只能看见已经提交事务所做的改变。 这种隔离级别也支持所谓的不可重复读（Nonrepeatable Read），因为同一事务的其他实例在该实例处理其间可能会有新的commit，所以同一select可能返回不同结果。
    Repeatable Read（可重读）
    这是MySQL的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：幻读 （Phantom Read）。 简单的说，幻读指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行。InnoDB和Falcon存储引擎通过多版本并发控制（MVCC，Multiversion Concurrency Control）机制解决了该问题。
    Serializable（可串行化）
    这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。 这四种隔离级别采取不同的锁类型来实现，若读取的是同一个数据的话，就容易发生问题。

    脏读(Drity Read)：某个事务已更新一份数据，另一个事务在此时读取了同一份数据，由于某些原因，前一个RollBack了操作，则后一个事务所读取的数据就会是不正确的。

    不可重复读(Non-repeatable read):在一个事务的两次查询之中数据不一致，这可能是两次查询过程中间插入了一个事务更新的原有的数据。

    幻读(Phantom Read):在一个事务的两次查询中数据笔数不一致，例如有一个事务查询了几列(Row)数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就会发现有几列数据是它先前所没有的。

    读不影响写：事务以排他锁的形式修改原始数据，读时不加锁，因为 MySQL 在事务隔离级别Read committed 、Repeatable Read下，InnoDB 存储引擎采用非锁定性一致读－－即读取不占用和等待表上的锁。即采用的是MVCC中一致性非锁定读模式。 因读时不加锁，所以不会阻塞其他事物在相同记录上加 X锁来更改这行记录。

    写不影响读：事务以排他锁的形式修改原始数据，当读取的行正在执行 delete 或者 update 操作，这时读取操作不会因此去等待行上锁的释放。相反地，InnoDB 存储引擎会去读取行的一个快照数据。

间隙锁：间隙锁主要用来防止幻读，用在repeatable-read隔离级别下，指的是当对数据进行条件，范围检索时，对其范围内也许并存在的值进行加锁！ 当查询的索引含有唯一属性（唯一索引，主键索引）时，Innodb存储引擎会对next-key lock进行优化，将其降为record lock,即仅锁住索引本身，而不是范围！若是普通辅助索引，则会使用传统的next-key lock进行范围锁定！

意向锁：
是一种表锁，意向锁不会与行级的共享 / 排他锁互斥！！！ 与其它表锁有兼容互斥性

SELECT column FROM table ... LOCK IN SHARE MODE; （S锁）
SELECT column FROM table ... FOR UPDATE; （X锁）

3.	MVCC
MVCC的全称是“多版本并发控制”。这项技术使得InnoDB的事务隔离级别下执行一致性读操作有了保证，换言之，就是为了查询一些正在被另一个事务更新的行，并且可以看到它们被更新之前的值。 这是一个可以用来增强并发性的强大的技术，因为这样的一来的话查询就不用等待另一个事务释放锁。这项技术在数据库领域并不是普遍使用的。一些其它的数据库产品，以及mysql其它的存储引擎并不支持它。
注：MVCC只在RC和RR两个隔离级别下工作，其他两个隔离级别都和MVCC不兼容
    
3.1	什么是多版本并发控制呢MVCC ？
mysql的innodb采用的是行锁，而且采用了多版本并发控制来提高读操作的性能。
    其实就是在每一行记录的后面增加两个隐藏列，记录创建版本号和删除版本号，而每一个事务在启动的时候，都有一个唯一的递增的版本号。 在InnoDB中，给每行增加两个隐藏字段来实现MVCC，两个列都用来存储事务的版本号，每开启一个新事务，事务的版本号就会递增。

3.2默认的隔离级别（REPEATABLE READ）下，增删查改？
    SELECT
    读取创建版本小于或等于当前事务版本号，并且删除版本为空或大于当前事务版本号的记录。这样可以保证在读取之前记录是存在的
    INSERT
    将当前事务的版本号保存至行的创建版本号
    UPDATE
    新插入一行，并以当前事务的版本号作为新行的创建版本号，同时将原记录行的删除版本号设置为当前事务版本号
    DELETE
    将当前事务的版本号保存至行的删除版本号

3.3 什么是快照读和当前读？
    快照读：读取的是快照版本，也就是历史版本
    当前读：读取的是最新版本
    普通的SELECT就是快照读，而UPDATE、DELETE、INSERT、SELECT …  LOCK IN SHARE MODE（S锁）、SELECT … FOR UPDATE是当前读。

3.4 什么是锁定读？(意向锁)
    在一个事务中，标准的SELECT语句是不会加锁，但是有两种情况例外。
    SELECT ... LOCK IN SHARE MODE 给记录假设共享锁，这样一来的话，其它事务只能读不能修改，直到当前事务提交
    SELECT ... FOR UPDATE 给索引记录加锁，这种情况下跟UPDATE的加锁情况是一样的

3.5 什么是一致性非锁定读？
    consistent read （一致性读），InnoDB用多版本来提供查询数据库在某个时间点的快照。如果隔离级别是REPEATABLE READ，那么在同一个事务中的所有一致性读都读的是事务中第一个这样的读读到的快照； 如果是READ COMMITTED，那么一个事务中的每一个一致性读都会读到它自己刷新的快照版本。Consistent read（一致性读）是READ COMMITTED和REPEATABLE READ隔离级别下普通SELECT语句默认的模式。 一致性读不会给它所访问的表加任何形式的锁，因此其它事务可以同时并发的修改它们。
    MVCC实现一致性非锁定读，这就有保证在同一个事务中多次读取相同的数据返回的结果是一样的，解决了不可重复读的问题。

3.6 什么是悲观锁和乐观锁？
    悲观锁：正如它的名字那样，数据库总是认为别人会去修改它所要操作的数据，因此在数据库处理过程中将数据加锁。其实现依靠数据库底层。
    乐观锁：如它的名字那样，总是认为别人不会去修改，只有在提交更新的时候去检查数据的状态。通常是给数据增加一个字段来标识数据的版本。

3.7 select时怎么加排它锁？
    使用锁定读，普通select不会引起加锁，而是去读取最新的快照。同上4
    事务以排他锁的形式修改原始数据，当读取的数据正在进行更新等操作，则直接去读取快照，而不是等锁释放

2. 集群/并发
mysql并发情况下怎么解决？
        代码中sql语句优化； 数据库字段优化，索引优化；加缓存，redis/memcache等；主从，读写分离 集群 分流 横向扩展；分区；垂直拆分，解耦模块；水平切分 分片

数据库的主从复制 ？
1）	就算MYSQL拆成了多个,也必须分出主和从,所有的写操作都必须要在主MYSQL 上完成;
    2）所有的从MYSQL的数据都来自于(同步于)主MYSQL;

    既然涉及到同步,那一定有延迟;有延迟,就一定可能在读的时候产生脏数据;所以,能够在从MYSQL上进行的读操作,一定对实时性和脏数据有一定容忍度的数据;比如,登陆日志,后台报表,首页统计信息来源;文章;资讯;SNS消息;

    完成主从同步,第一个事情就是把在主服务器上的bin-log(二进制文件)打开,bin-log文件就可以记录在MYSQL上执行的所有的改动
    MYSQL使用被动注册的方式来让从MYSQL请求同步主MYSQL的binlog;原因:被动请求的方式,主的MYSQL不需要知道有哪些从的MYSQL,我额外添加/去掉从MYSQL服务器,对主MYSQL服务器的正常运行没有任何影响;
    第二步,从MYSQL后台一个线程发送一个请求,到主服务器请求更新数据;最重要的数据
    第三步,主MYSQL后台一个线程接收到从MYSQL发送的请求,然后读取bin-log文件中指定的内容,并放在从MYSQL的请求响应中;
    第四步,从MYSQL的请求带回同步的数据,然后写在从MYSQL中的relay-log(重做日志)中;relay-log中记录的就是从主MYSQL中请求回来的哪些SQL数据;
    第五步,从MYSQL后台一个线程专门用于从relay-log中读取同步回来的SQL,并写入到从MYSQL中,完成同步;
    MYSQL的主从同步是经过高度优化的,性能非常高;


（二）问题
1.关系型数据库和非关系型数据库区别？
非关系型数据库
	1）优点：
    查询速度：nosql数据库将数据存储于缓存之中，关系型数据库将数据存储在硬盘中，自然查询速度远不及nosql数据库。
    存储数据的格式：nosql的存储格式是key,value形式、文档形式、图片形式等等，所以可以存储基础类型以及对象或者是集合等各种格式，而数据库则只支持基础类型。
扩展性：关系型数据库有类似join这样的多表查询机制的限制导致扩展很艰难。
	2) 缺点：
    不提供对sql的支持
    不提供关系型数据库对事务的处理。
    
非关系型数据库的优势
    1）性能NOSQL是基于键值对的，可以想象成表中的主键和值的对应关系，而且不需要经过SQL层的解析，所以性能非常高。
    2）可扩展性同样也是因为基于键值对，数据之间没有耦合性，所以非常容易水平扩展。

关系型数据库的优势
    1）复杂查询可以用SQL语句方便的在一个表以及多个表之间做非常复杂的数据查询。
    2）事务支持使得对于安全性能很高的数据访问要求得以实现。对于这两类数据库，对方的优势就是自己的弱势，反之亦然。

2. Mysql主从复制是怎么工作的呢？说说各个线程具体做了什么吧？

5. 索引有B+索引和hash索引，各自的区别？
主要区别
键值唯一，如果是等值查询，哈希索引有绝对优势
重复键值情况下，哈希索引的效率也是极低的，因为存在所谓的哈希碰撞问题。
范围查询检索，模糊查询（like），不支持多列联合索引的最左匹配规则，无法使用哈希索引

11、schema(表结构)对性能的影响？


13、explain和join
14、内连接、外连接、交叉连接、笛卡儿积等
内连接 
        只有两个表相匹配的行才能在结果集中出现 分为三种：等值连接、自然连接、不等连接 

外连接 
        左外连接(LEFT OUTER JOIN或LEFT JOIN) 以左边为准，右边没用则为空
        右外连接(RIGHT OUTER JOIN或RIGHT JOIN) 以右边为准，左边没有则为空
        全外连接(FULL OUTER JOIN或FULL JOIN) 左右均可能为空 

交叉连接
        没有WHERE 子句，它返回连接表中所有数据行的笛卡尔积 
笛卡儿积


（三） 延伸（不计复习）

3.CAP 分布式系统不可能同时满足一致性（C：Consistency）、可用性（A：Availability）和分区容忍性（P：Partition Tolerance），最多只能同时满足其中两项

dubbo+zookeeper 主要实现CP
springcloud eureka [hystrix] 主要实现AP
以上与服务注册细节相关

4.BASE 是基本可用（Basically Available）、软状态（Soft State）和最终一致性（Eventually Consistent）三个短语的缩写。 BASE 理论是对 CAP 中一致性和可用性权衡的结果，它的理论的核心思想是：即使无法做到强一致性，但每个应用都可以根据自身业务特点，采用适当的方式来使系统达到最终一致性。


8、B+索引数据结构，和B树的区别 ？

    +树索引的关键字检索效率比较平均，不像B树那样波动幅度大，在有大量
、为什么B+树适合作为索引的结构？
    B树：有序数组+平衡多叉树
    B+树：有序数组链表+平衡多叉树 叶子存储数据，空间占用小，且是双链表，修改效率快
    不同于B树只适合随机检索，B+树同时支持随机检索和顺序检索

    数据库索引采用B+树的主要原因是B树在提高了磁盘IO性能的同时并没有解决元素遍历的效率低下的问题。 正是为了解决这个问题，B+树应运而生。B+树只要遍历叶子节点就可以实现整棵树的遍历。而且在数据库中基于范围的查询是非常频繁的，而B树不支持这样的操作（或者说效率太低）。

    平衡二叉树没能充分利用磁盘预读功能，而B树是为了充分利用磁盘预读功能来而创建的一种数据结构，也就是说B树就是为了作为索引才被发明出来的的。