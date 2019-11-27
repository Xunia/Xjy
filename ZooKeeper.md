<!-- TOC -->

- [概述](#概述)
    - [1. 重要概念：](#1-重要概念)
    - [1.1 Session](#11-session)
    - [1.2 Znode](#12-znode)
    - [1.3 Stat](#13-stat)
    - [1.4 Watcher](#14-watcher)
    - [1.5 ACL（AccessControlLists）权限控制](#15-aclaccesscontrollists权限控制)
    - [2. ZooKeeper 特点](#2-zookeeper-特点)
    - [3. ZooKeeper 设计目标](#3-zookeeper-设计目标)
    - [4. ZooKeeper 集群角色介绍](#4-zookeeper-集群角色介绍)
    - [5. ZooKeeper &ZAB 协议&Paxos算法](#5-zookeeper-zab-协议paxos算法)

<!-- /TOC -->
## 概述
ZooKeeper 是一个开源的分布式协调服务。

ZooKeeper 的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。

原语： *操作系统或计算机网络用语范畴。是由若干条指令组成的，用于完成一定功能的一个过程。具有不可分割性·即原语的执行必须是连续的，在执行过程中不允许被中断。*

ZooKeeper 是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于 ZooKeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

Zookeeper 一个最常用的使用场景就是用于担任服务生产者和服务消费者的注册中心。

主要功能：  
1、集群管理：容错、负载均衡。  
2、配置文件的集中管理  
3、集群的入口。  


### 1. 重要概念：

ZooKeeper 本身就是一个分布式程序（只要半数以上节点存活，ZooKeeper 就能正常服务，最好是以集群形态来部署）。

ZooKeeper 将数据保存在内存中，这也就保证了 高吞吐量和低延迟（*但是内存限制了能够存储的容量不太大，此限制也是保持znode中存储的数据量较小的进一步原因*）。

ZooKeeper 是高性能的。 *在“读”多于“写”的应用程序中尤其地高性能，因为“写”会导致所有的服务器间同步状态。（“读”多于“写”是协调服务的典型场景。）*

ZooKeeper有临时节点的概念。 *当创建临时节点的客户端会话一直保持活动，期间节点就一直存在。而当会话终结时，瞬时节点被删除。持久节点是指一旦这个ZNode被创建了，除非主动进行ZNode的移除操作，否则这个ZNode将一直保存在Zookeeper上。*

ZooKeeper 底层其实只提供了两个功能：
* ①管理（存储、读取）用户程序提交的数据；
* ②为用户程序提交数据节点监听服务。


### 1.1 Session   
服务端首先会为每个客户端都分配一个sessionID  
一个客户端连接是指客户端和服务器之间的一个 TCP 长连接

### 1.2 Znode

第一类同样是指构成集群的机器，我们称之为机器节点；
第二类则是指数据模型中的数据单元，我们称之为数据节点一一ZNode。

### 1.3 Stat 
version（当前ZNode的版本）、
cversion（当前ZNode子节点的版本）和 
cversion（当前ZNode的ACL版本）。

### 1.4 Watcher
Watcher（事件监听器），是Zookeeper中的一个很重要的特性。Zookeeper允许用户在指定节点上注册一些Watcher，并且在一些特定事件触发的时候，ZooKeeper服务端会将事件通知到感兴趣的客户端上去，该机制是Zookeeper实现分布式协调服务的重要特性。

### 1.5 ACL（AccessControlLists）权限控制
create,delete 子节点

read，write，admin

### 2. ZooKeeper 特点

* 顺序一致性： 从同一客户端发起的事务请求，最终将会严格地按照顺序被应用到 ZooKeeper 中去。

* 原子性： 所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，也就是说，要么整个集群中所有的机器都成功应用了某一个事务，要么都没有应用。

* 单一系统映像 ： 无论客户端连到哪一个 ZooKeeper 服务器上，其看到的服务端数据模型都是一致的。

* 可靠性： 一旦一次更改请求被应用，更改的结果就会被持久化，直到被下一次更改覆盖。


### 3. ZooKeeper 设计目标
* 数据模型
* 可构建集群
* 顺序访问（zxid）
* 高性能

### 4. ZooKeeper 集群角色介绍
Leader、Follower 和 Observer

ZooKeeper 集群中的所有机器通过一个 Leader 选举过程来选定一台称为 “Leader” 的机器，Leader 既可以为客户端提供写服务又能提供读服务。除了 Leader 外，Follower 和 Observer 都只能提供读服务。Follower 和 Observer 唯一的区别在于 Observer 机器不参与 Leader 的选举过程，也不参与写操作的“过半写成功”策略，因此 Observer 机器可以在不影响写性能的情况下提升集群的读性能。

重新选举：  

Leader election（选举阶段）：节点在一开始都处于选举阶段，只要有一个节点得到超半数节点的票数，它就可以当选准 leader。

Discovery（发现阶段）：在这个阶段，followers 跟准 leader 进行通信，同步 followers 最近接收的事务提议。

Synchronization（同步阶段）:同步阶段主要是利用 leader 前一阶段获得的最新提议历史，同步集群中所有的副本。同步完成之后 准 leader 才会成为真正的 leader。

Broadcast（广播阶段） 到了这个阶段，Zookeeper 集群才能正式对外提供事务服务，并且 leader 可以进行消息广播。同时如果有新的节点加入，还需要对新节点进行同步。


### 5. ZooKeeper &ZAB 协议&Paxos算法
ZooKeeper 并没有完全采用 Paxos算法 ，而是使用 ZAB 协议作为其保证数据一致性的核心算法    

ZAB（ZooKeeper Atomic Broadcast 原子广播） 协议是为分布式协调服务 ZooKeeper 专门设计的一种支持崩溃恢复的原子广播协议。
