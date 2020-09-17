快速聚合、灵活过滤、低延迟数据摄入

# 1、Introduction

存储和分析大量日志数据

Hadoop存在的不足：

1. Hadoop擅长存储大量数据并提供了访问大量数据的能力，但对大量数据的的访问速度并没有提供性能保障。
2. Hadoop是一个高可用的系统，当存在大量并发负载时，系统性能会降低。
3. Hadoop能很好地存储数据，但对于数据的摄入以及摄入后让数据立马可读并没有进行优化。

因此在某些高并发、高查询的环境下，Hadoop并不能满足需求，Druid因此诞生。Druid是一种开源的、分布式的、基于列式存储、实时的分析数据存储系统。Druid和其他OLAP系统很像：交互式查询系统、内存数据库、分布式数据存储。

# 2、Problem Definition

Druid最初设计用来解决有关摄入和探索大量交易数据（日志数据）的问题。在OLAP系统中，数据通常为时序数据且不断地流入。如**Table 1**所示。

<img src="E:\总结\markdown\druid\Images\table1.png" alt="table1" style="zoom: 50%;" />

**Table 1** 表示对于 Wikipedia 页面的编辑数据。当某个用户在编辑Wikipedia的某个页面时会产生一个事件，这个事件会产生x相应的元数据。**元数据包含三个部分。第一部分，时间戳（timestamp）**，用来标识事件发生的时间。**第二部分，属性维度（dimension column）用来标识事件的各种属性**，例如这里的编辑的页面名称（Page）、编辑的用户（Username）、用户的性别（Gender）、用户所在城市（City）。**第三部分，度量维度（metric column），维度通常包含数值类型的值，可以进行聚合操作**，如这里的页面增加的特性（Characters Added）、删除的特性（Characters Removed）。

**我们的目标是快速地对数据进行下钻和聚合**。例如对于 **Table 1** 中的数据，我们想回答的问题有：来自San Francisco的男性，对Justin Bieber页面的编辑一共有多少？过去一个月来自Calgary的用户在Wikipedia平均增加的特性有多少？

Druid的诞生是由于已经存在的关系型数据库（RDBMS）和基于键值对的NoSQL数据库，对于交互式应用不能提供低延迟的数据摄入和查询能力，此外系统还需支持多用户、高可用且实时响应。传统的基于Hadoop的开源的数据仓库系统，不能提供亚秒级的数据摄入延迟。

# 3、Architecture

Druid集群包含不同类型的节点，每种节点用来执行特定的任务。这种设计策略简化了系统的复杂性。不同节点独立运行且他们之间的交互很少，因此集群内部的交互错误对于数据可用性的影响很小。Druid 的架构如 **Figure 1** 所示。

<img src="E:\总结\markdown\druid\Images\figure1.png" alt="figure1" style="zoom: 50%;" />

## 3.1 Real-time Nodes

Real-time Nodes 提供数据摄入和查询事件流的能力。事件在被实时节点索引标识后可立即被查询到。Real-time Nodes 只关心一小段时间内的事件，并定期把这段时间内收集的事件流移交给Druid集群中专门处理批事件流的节点。Real-time Nodes 利用Zooker来和其它节点协作，并向Zooker报告在线状态以及当前节点所保存的数据信息。

Real-time Nodes 对所有到来的事件在内存中都有一份索引标识，这些索引在数据被摄入后逐渐递增且也能被查询到。Druid 在基于堆的JVM缓存中，提供了对于事件的行查询能力。为了避免堆内存溢出，Real-time Nodes 定期地或在达到行最大限制时，把内存中的索引持久化到本地磁盘中（列式存储）。每个被持久化的索引都是不可修改的，Real-time Nodes 把这些已经持久化的索引再次加载到非堆内存中，以便它们依然能被查询到。这个过程可以用 **Figure 2** 表示。

<img src="E:\总结\markdown\druid\Images\figure2.png" alt="figure2" style="zoom: 50%;" />

在一个基本周期，每个 Real-time Nodes 会调度一个后台任务去搜寻本地所有的被持久化索引。该任务把基本周期内所有被索引标识的事件合并为一个不可修改的块数据，这个块数据成为 **Segment**，之后 Real-time Nodes 把 Segment 传到持久的备份存储中，通常为一个分布式的文件系统，例如Amazon  S3或HDFS。Druid中，这个分布式文件系统称为 **Deep Storage** 。

**Figure 3** 阐述了一个Real-time Nodes 的运作过程。节点在13:37启动，且只会接收当前小时以及下一个小时的事件。当事件被摄入后，节点对外宣称保存了从13:00到14:00的一个Segment数据，每10分钟（时间可以配置）节点会把内存中的索引持久化到磁盘上。在快接近14:00点时，节点可能看见了14:00到15:00之间的数据。当这种情况发生的时候，节点会创建一个新的内存索引，并准备保存下一个小时的数据。该节点对外宣称同时保存了14:00到15:00之间的数据。节点并不立即持久化13:00到14:00之间的索引，而是会保持一个可配置的窗口来继续接收13:00到14:00之间的数据，窗口的使用减少了数据丢失的风险。在窗口结束时，该节点将合并所有13:00到14:00之间的索引到一个不可修改的Segment，并把这个Segment存储到Deep Storage中。当这个Segment在Druid集群中的其他地方被加载且可查询到时，Real-time Nodes 释放所有13:00到14:00之间的信息，并宣称它不在保存该时间段的数据。

<img src="E:\总结\markdown\druid\Images\figure3.png" alt="figure3" style="zoom:50%;" />

### 3.1.1 Availability and Scalability

实时节点是一个数据消费者，需要一个与之相应的生产者来提供数据流。通常为了数据持久性目的，消息总线Kafka位于生产者和实时节点之间，如**Figure 4**所示。实时节点从Kafka中读取事件并摄入数据。事件的创建与事件的消耗在几百毫秒。Kafka的作用有两个，第一，充当到来事件的缓冲区。第二，多个Real-time Nodes可以同时从Kafka中摄入数据。

<img src="E:\总结\markdown\druid\Images\figure4.png" alt="figure4" style="zoom:50%;" />

## 3.2 Historical Nodes

历史节点从Deep Storage中加载Segment，并保证在历史节的内存中提供对历史数据的查询。在真实环境中，大多数在Druid中的数据是不可修改的，因此历史节点是Druid集群中的主要成分。历史节点架构不共享，因此并比存在单点故障。每个节点并不知道其他节点，历史Node只知道如何加载、删除、保存不可修改的Segment。

和实时节点类似，历史节点也向Zoomkeeper报告节点状态以及当前所保存的信息。加载和删除Segment的指令通过Zoomkeeper来发送，并包含Segment在Deep Storage的位置信息以及如何进行加压缩与处理。当历史节点下载一个特定的Segment时候，它受限检查本地缓存中是否以及存在，如果不存在，历史节点则会去Deep Storage中下载该Segment。该过程如Figure 5所示。该过程完成后，该Segment会在Zoomkeeper中声明，此时，该Segment变得可查询。本地缓存同样允许历史节点进行快速地更新和重启。在重启时，该节点检查它的缓存，然后历史保存它所找见的节点。

<img src="E:\总结\markdown\druid\Images\figure5.png" alt="figure5" style="zoom: 50%;" />

## 3.3 Broker Nodes

代理节点充当历史节点和实时节点的查询路由。代理节点从Zoomkeeper保存的元数据知道，哪些Segment是可查的以及这些Segment在哪里。当查询请求发生时，代理节点对查询请求在历史节点以及实时节点之间进行路由。代理节点在返回数据的时候，可以合并来自历史节点与实时节点之家的数据并返回。

代理节点的缓存使用LRU算法，缓存可以使用本地的堆内存或使用外部的基于键值对的存储，例如Memcache，Redis等。当代理节点收到一个请求时候，首先在本地缓存中进行查询，当本地缓存中不存在时，代理节点会把请求路由到历史节点或实时节点。当数据从历史节点返回时，本地缓存中会保存一份。该过程如Figure 6所示。从实时节点返回的数据并不会进行缓存操作，因为实时节点的数据在时刻改变，缓存实时节点的数据并不可靠。

## 3.4 Coordinator Nodes

协调节点主要对历史节点中的数据进行管理和分发。协调节点告诉历史节点加载新数据、删除旧数据、复制数据、把数据移动到负载均衡。Druid使用多版本的并发控制交换协议来管理不可变的Segment以保持稳定的视图。如果一个不可修改的Segment中的数据完成过时了，则这个Segment会从集群中删除掉。协调节点也有一个备节用来充当备份。

协调节点周期地运行来决定当前集群的状态。通过比较期望的集群状态与实际的集群状态来做出决策。和其他Druid节点一样，协调节点和Zoomkeeper保持一个连接来交换集群的信息。协调节点同时保持了一个到MySQL数据库的链接来保持额外的操作和配置信息。在MySQL中保持的一种关键信息是，MySQL中有一张表，保持了历史节点中应该保存的所有Segment，这种表可以被任何创建Segment的节点进行更新，例如实时节点。MySQL中还保存了一张规则表用来管理Segment在集群中如何创建、销毁、复制。

# 4、Storage Format

Druid中的数据表又被称为Data Source，Data Source是时序事件的集合，又被分为很多Segment，每个Segment通常包含500到1000W行。Segment是跨越某个时间段的数据行的集合。Segment是Druid中的基本存储单元。数据的复制和分发是以Segment进行的。

Druid包含多种列类型来表示各种数据，基于列的数据类型，各种压缩算法用来减少在内存和磁盘中的存储空间。例如在Table 1中，page，user，gender，city列是字符串类型，直接存储字符串代价很高，可以采用字典编码对对其进行编码。例如

```
Justin Bieber	->	0
Ke$ha	->	1
```

这样Page列，可以用整形数组来表示，即

```
[0,0,1,1]
```

同样的方法可以用来对数值类型的列进行压缩，例如：

```
Characters Add	->	[1800,2912,1953,1314]
Characters Removed	->	[25,42,17,170]
```

## 4.2 Indices for Filtering Data

在很多真实的OLAP系统中，通常还需要对维度进行过滤。在很多真实的数据集中，dimension column包含string类型的值，metric column包含number类型的值，Druid对String类型的列创建额外的indice，来加速filter的速度。

```
Justin Bieber -> rows [0, 1] -> [1][1][0][0]
Ke$ha	-> rows [2, 3] -> [0][0][1][1]
```

查询哪些行包含Justin Bieber或Ke$ha，我们可以直接进行或操作

```
[0][1][0][1] OR [1][0][1][0] = [1][1][1][1]
```

