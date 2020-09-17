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

Historical Nodes从Deep Storage中加载 Real-time Node创建的Segment，并保证在Historical Nodes的内存中提供对历史数据的查询。在真实环境中，大多数在Druid集群中加载的数据是不可修改的，因此Historical Nodes是Druid集群中的主要成分。Historical Nodes不共享的架构设计决定了Historical Nodes并不存在单点故障。每个节点并不知道其他节点，历史Node只知道如何加载、删除、保存不可修改的Segment。

和实时节点类似，历史节点也向Zookeeper报告节点状态以及当前所保存的信息。加载和删除Segment的指令通过Zookeeper来发送，并包含Segment在Deep Storage的位置信息以及如何进行加压缩处并理。当历史节点下载一个特定的Segment时，它首先检查本地缓存中是否存在，如果不存在，历史节点则会去Deep Storage中下载该Segment。该过程如**Figure 5**所示。该过程完成后，该Segment会在Zoomkeeper中声明，此时该Segment变得可查询。Historical Nodes的本地缓存同样允许历史节点进行快速地更新和重启。在启动时Historical Nodes节点检查它的缓存，然后立即将数据保存再内存中。由于Segment是不可修改的，因此Historical Nodes可以保持数据一致性，且可进行并发访问与聚合。

<img src="E:\总结\markdown\druid\Images\figure5.png" alt="figure5" style="zoom: 50%;" />

### 3.2.1 Tiers

Historical Nodes可以进行分层，每一层的的配置相同，不同的层可以设置不同的性能和错误容忍度参数。分层的目的是根据Segment的重要程度给予一定的高低优先级。

### 3.2.2 Availability

Historical Nodes依靠Zookeeper进行数据的加载与删除，如果Zoomkeeper宕机，Historical Nodes依然可以从本地所保存的Segment来做出响应。

## 3.3 Broker Nodes

Broker Nodes充当Historical Nodes和Real-Time Nodes的查询路由。Broker Nodes从Zookeeper保存的元数据知道哪些Segment是可查的，以及这些Segment在存储位置。当查询请求发生时，Broker Nodes在Historical Nodes以及Real-Time Nodes之间进行路由。Historical Nodes在返回数据的时候，可以合并来自Historical Nodes与Real-Time Nodes之间的数据并返回。

### 3.3.2 Caching

Broker Nodes的缓存使用LRU算法，缓存可以使用本地的堆内存或使用外部的基于键值对的存储，例如Memcache，Redis等。当Broker Nodes收到一个请求候，首先在本地缓存中进行查询，当本地缓存中不存在时，代理节点会把请求路由到Historical Node或Real-Time Nodes。当数据从Historical Nodes返回时，Broker Nodes的本地缓存中会保存一份。该过程如**Figure 6**所示。从Real-Time Node返回的数据并不会进行缓存操作，因为Real-Time Node的数据在时刻改变，缓存实时节点的数据并不可靠。

![figure6](E:\总结\markdown\druid\Images\figure6.png)

### 3.3.2 Availability

如果Zookeeper完全宕机，数据依然可查询。Broker Nodes使用最近知道的集群信息来路由查询到Historical Nodes或Real-Time Nodes。Broker Node假设集群的结构和之前是一样的。

## 3.4 Coordinator Nodes

Coordinator Nodes主要对Historical Nodes中的数据进行管理和分发。Coordinator Nodes告诉Historical Nodes加载新数据、删除旧数据、复制数据、数据移动以实现负载均衡。Druid使用多版本的并发控制交换协议来管理Segment以保持稳定的视图。如果一个不可修改的Segment中的数据完成过时了，且该Segment可以被另一个Segment替换时，这个Segment会从集群中删除掉。协调节点也有一个备节用来充当备份。

Coordinator Nodes周期地运行来决定当前集群的状态。通过比较期望的集群状态与实际的集群状态来做出决策。和其他Druid节点一样，Coordinator Nodes和Zoomkeeper保持一个连接来交换集群的信息。Coordinator Nodes同时保持了一个到MySQL数据库的连接来保存额外的操作和配置信息。在MySQL中保持了一种关键信息，在MySQL中有一张表，保持了Historical Nodes中应该保存的所有Segment，这张表可以被任何创建Segment的节点进行更新，例如实时节点。MySQL中还保存了一张规则表用来管理Segment在集群中如何创建、销毁、复制。

### 3.4.1 Rules

Rules用来管理历史Segment在集群中如何进行加载与删除。Rules表明Segment如何指定到Historical Nodes的不同层中，以及不同层中存在多少个副本。Rules还可以指明集群中的Segment什么时候被删除。Rules通常会设置一个有效期。

### 3.4.2 Load Balancing

在一个典型的生产环境中，一次查询可能会返回成百上千的Segment。由于每个Historical Nodes的资源有限，Segment必须在集群中进行有效的分发，以使集群的负载不会太大。

### 3.4.3 Replication

Coordinator Nodes可以Hist告诉orical Nodes加载相同Segment的一个副本。在Historical Nodes的每一层中，副本的数量是可以配置的。在一个高容错的系统中，副本的数量配置得高一些。

### 3.4.4 Availability

Coordinator Nodes依靠ZooKeeper来决定哪些Historical Nodes已经在集群中存在，如果ZooKeeper不可用，Coordinator不能发送指令来指定、平衡、删除Segment。但这些操作并不影响数据的可用性。

# 4、Storage Format

Druid中的数据表又被称为Data Source，Data Source是时序事件的集合，这些事件又被分为很多Segment，每个Segment通常包含500到1000万行。Segment是跨越某个时间段的数据的集合，且Segment是Druid中的基本存储单元。数据的复制和分发是以Segment进行的。

Druid要求一个时间戳列来简化数据的分发、保留策略，以及对第一阶段的查询进行剪枝。Druid将Data Source中的数据按照时间段进行分区，通常为一小时或一天，并可以借助其它列来完成理想的分区大小。Segment可以使用Data Source标识符、时间段、Segment创建时的版本字符串来唯一标识。Segment是按列进行存储的，因此有利于对事件流进行聚合操作。

Druid包含多种列类型来表示各种数据，不同的类型可以使用不同的压缩算法用来减少在内存和磁盘中的存储空间。例如在**Table 1**中，page，user，gender，city列是字符串类型，直接存储字符串代价很高，可以采用字典编码对其进行编码。例如

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

## 4.1 Indices for Filtering Data

在很多真实的OLAP系统中，通常还需要对维度进行过滤。在很多真实的数据集中，dimension column包含string类型的值，metric column包含number类型的值，Druid对String类型的列创建额外的查询索引以加速过滤的速度。例如Table 1中，Page列中包含Justin Bieber和Ke$ha的行可以用如下数据表示。

```
Justin Bieber -> rows [0, 1] -> [1][1][0][0]
Ke$ha	-> rows [2, 3] -> [0][0][1][1]
```

当查询哪些行包含Justin Bieber或Ke$ha，我们可以直接进行**或**操作，这样提高了查询效率。

```
[0][1][0][1] OR [1][0][1][0] = [1][1][1][1]
```

## 4.2 Storage Engine

Druid的持久化组件允许使用不同的存储引擎，这些存储引擎可以完全在内存中存储数据，例如堆内存，也可以在内存映射结构中存储数据。当使用内存映射结果存储引擎时，Druid依靠操作系统来调度Segment的进出内存。

# 5、Query API

Druid拥有自己的查询语言，接收POST类型的查询请求。Druid集群中的节点使用相同的查询API。POST请求体是一个包含各种查询参数的JSON对象。一个典型的查询包含数据源的名称、结果数据的粒度，查询的时间范围，请求的类型，以及聚合的Metric。查询结果也是一个JSON对象。很多查询还支持过滤操作。一个典型的查询如下：

```
{
    "queryType" : "timeseries",
    "dataSource" : "wikipedia",
    "intervals" : "2013-01-01/2013-01-08",
    "filter" : {
        "type" : "selector",
        "dimension" : "page",
        "value" : "Ke$ha"
    },
    "granularity" : "day",
    "aggregations" : [{"type":"count", "name":"rows"}]
}
```

查询结果如下：

```
[ {
        "timestamp": "2012-01-01T00:00:00.000Z",
        "result": {"rows":393298}
    },
    {
        "timestamp": "2012-01-02T00:00:00.000Z",
        "result": {"rows":382932}
    },
    ...
    {
        "timestamp": "2012-01-07T00:00:00.000Z",
        "result": {"rows": 1337}
} ]
```

