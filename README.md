# Storm概念、原理详解及其应用（一）BaseStorm

2016年07月10日 16:35:31

转自       [http://blog.csdn.net/kuring\_k/article/details/51872112](http://blog.csdn.net/kuring_k/article/details/51872112)

## 问题：Storm是什么？为什么要用Storm？为什么不用Spark？

### 对于第一个问题，Storm是什么？

**Storm是基于数据流的实时处理系统，提供了大吞吐量的实时计算能力。通过数据入口获取每条到来的数据，在一条数据到达系统的时候，立即会在内存中进行相应的计算；Storm适合要求实时性较高的数据分析场景。**

### 第二问题，为什么要用Storm？

很多场景下，我们**希望系统能够实时的处理一条数据、甚至是事务。也就是说，在处理数据、事务的过程中，到达系统，并能马上得到结果。**其次，在成万上亿条数据大量涌入系统时，也要求“实时”的得到事务处理的结果。此时，单个节点已经是杯水车薪了，而**Storm的其中一个特点是它支持分布式并行计算**！如果说，你遇到了以上相似的场景，那Storm可以当仁不让的扛起实时处理的大旗！

### 第三个问题，为什么不用Spark？

这个问题其实很难界定，因为Spark在RDD粒度上，可以满足实时计算的要求，当然，使用RDD还有其他优势；但总的来说，Storm 的实时性更强。其次，Storm的框架完全按照流式处理的思想构建，和项目场景结合性更强一些。（这是个人的一点看法，也许是因为本人Spark 用的不是很多的原因吧，欢迎吐槽。）

进入正题,

在看Storm之前，很多人都对Hadoop有一定了解，为了能更快入戏，我们以Hadoop为参照，以下是它使用yarn之前的架构，对照Storm Server框架理解。

Hadoop、Storm系统和组件接口对比表：

![](http://img.blog.csdn.net/20160711162225804?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## **Storm框架：**

先来看看Spark的架构，可以发现spark和Storm架构还是有很多相似的地方的。

![](/assets/Spark架构图.png)

                                                                                    Spark架构图：出自《Spark大数据处理》

![](http://img.blog.csdn.net/20160711162247492?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

                                                                                                             Storm框架图

上面这幅图是Stom框架图，和很多分布式系统一样，**基于zk（Zookeeper）作为集群配置运行的元数据基础平台**。

nimbus和supervisor是服务器端守护进程，守护进程的文章会在[Storm概念、原理详解及其应用（二）Storm Cluster](http://blog.csdn.net/kuring_k/article/details/51887340)进行讲解。

以下是对启动一个应用所需要的集群上JVM进程线程的简单介绍，建议记忆后再继续阅读。

**· Nodes （服务器）：指配置在一个 Storm 集群中的服务器，会执行 topology 的一部分运算。一个 Storm 集群可以包括一个或者多个工作 node 。**

**· Workers （JVM 虚拟机）：指一个 node 上相互独立运行的 JVM 进程。每个 node 可以配置运行一个或者多个 worker 。一个topology 会分配到一个或者多个 worker 上运行。**

**· Executor （线程）：指一个 worker 的jvm 进程中运行的 Java 线程。多个 task 可以指派给同一个 executer 来执行。除非是明确指定， Storm 默认会给每个 executor 分配一个 task。**

**· Task （bolt/spout 实例）： task 是 spout 和bolt 的实例， 它们的 nextTuple\(\) 和execute\(\) 方法会被executors 线程调用执行。**

例如：

```
builder.setSpout(spoutName, spout, spoutParallelism).setNumTasks(2)
```

这里可以定义spoutParallelism = 2，即对应两个executor线程，tasks为两个实例。（此处配置的原理，会在接下来会讲到worker和并发中解释。）

可以看出，虽然在这设置了多个task实例，但是并行度并没有提高（而executor在不同的worker上执行，存在并行），因为只有两个线程去运行这些实例，只有设置足够多的线程和实例才可以真正的提高并行度；在这设置多个实例主要是为了下面执行rebalance的时候用到。

### **为什么要用rebalance？**

这里一直在启动、操作的是“线程”，真正的process需要在配置中设置worker数量，也就是说topology启动时已经决定了worker数量（即并行数量）。

_因为rebalance不需要修改代码，就可以动态修改topology的并行度，这样的话就必须提前配置好多个实例，在rebalance的时候主要是对之前设置多余的任务实例分配线程去执行。_

### **在命令行动态修改并行度：**

除了使用代码进行调整，还可以在shell命令行下对并行度进行调整。

storm rebalance mytopology -w 10 -n 2 -e spout=2 -e bolt=2

表示 10秒之后对mytopology进行并行度调整。把spout调整为2个executor，把bolt调整为2个executor

注意：并行度主要就是调整executor的数量，但是调整之后的executor的数量必须小于等于task的数量，如果分配的executor的线程数比task数量多的话也只能分配和task数量相等的executor。

## **概念：   **

官方对于下列Storm名词概念的解释如下：

1、Topologies

2、Streams

3、Spouts

4、Bolts

5、Stream groupings

6、Reliability

7、Tasks

8、Workers

### 1、Topologies（拓扑）

Topology是Storm中实时应用的一种封装。其功能 analogous（模拟的，类似的） to a MapReduce job类似于MapReduce 的job，但唯一不同的是它是循环执行的——无数据流等待，有数据流执行，直到被kill progress。

一个Topology是由spouts和bolts组成并被Stream groupings连接的一副流程图，相关概念如下：

Resources:

* [TopologyBuilder](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/TopologyBuilder.html): use this class to construct topologies in Java：在java中，该类构建了topologies。
* [Running topologies on a production cluster](http://storm.apache.org/releases/1.0.0/Running-topologies-on-a-production-cluster.html)：在生产集群中，运行多个topologies。
* [Local mode](http://storm.apache.org/releases/1.0.0/Local-mode.html): Read this to learn how to develop and test topologies in local mode. 在本地模型中开发和测试topologies。

Topology结构：

![](http://img.blog.csdn.net/20160711162446026?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

2、Streams （流）

**Stream在Storm中是一个核心的抽象概念。一个流是由无数个元组序列构成，这些元组并行、分布式的被创建和执行。**

在stream的许多元组中，Streams被定义为以Fields区域命名的一种模式。默认情况下，**元组支持：integers, longs, shorts, bytes, strings, doubles, floats, booleans, and byte arrays**. 你也可以定义自己的序列化器，使这种风格类型能够被自然的使用在元组中。

每一个Stream在声明的时候都会赋予一个id。单个Stream——spouts和bolts，可以使用[OutputFieldsDeclarer](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/OutputFieldsDeclarer.html)的convenience方法声明一个stream，而不用指定一个id。但是这种方法会给予一个默认的id——default，相关概念如下：

Resources:

* [Tuple](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/tuple/Tuple.html): streams are composed of tuples：Tuple是一个interface，对应的实现类 TupleImpl。
* [OutputFieldsDeclarer](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/OutputFieldsDeclarer.html): used to declare streams and their schemas
* [Serialization](http://storm.apache.org/releases/1.0.0/Serialization.html): Information about Storm's dynamic typing of tuples and declaring custom serializations

Ps：Storm中的tuple是接口，没有具体实现，但原话是这么解释的：

_Storm needs to know how to serialize all the values in a tuple. By default, Storm \* knows how to serialize the primitive types, strings, and byte arrays._

3、Spouts

在Topology中，每个Spout都是一个Streams源，通常情况下，Spouts会从外部源读取Tuple，并输入这些Tuple到Topology中

。

Spouts既是可靠的又是不可靠的

，因为，可靠的spout会在发送Tuple失败的情况下，重复发送；相反，不可靠的spout会忘记它发送过的Tuple，无论是否成功。

Spout代码过程：

Spouts能够发送多个流：使用

[OutputFieldsDeclarer](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/OutputFieldsDeclarer.html)

\(interface\)

的

`declareStream`

方法声明多个流，并且当使用

[SpoutOutputCollector](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/spout/SpoutOutputCollector.html)

（实现2，接口模式）

的emit方法可以

指定这个流去发送Tuple。

Spouts的主要方法之一是：

nextTuple\(\) 发送tuple

，nextTuple可以发送一个新的Tuple到Topology，或者当没有新的Tuple被发送的时候，就简单的返回。

对于任何spout的实现，

nextTuple都不能阻塞，因为Storm调用的所有spout都是基于同一个线程！

其次是 ack 和 fail 方法，

它们都会被调用，

当Storm发现一个tuple被从spout发射后，要么成功地完成的通过topology，要么错误的完成。ack 和 fail 方法只有在可靠的spouts下才能被调用。spout可靠性，请搜本页下面内容，或移至代码。

Resources:

* [IRichSpout](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/IRichSpout.html)
  : this is the interface that spouts must implement.
* [Guaranteeing message processing](http://storm.apache.org/releases/1.0.0/Guaranteeing-message-processing.html)

Ps：nextTuple\(\)方法中会发送Tuple，至于那种对象能发送，请看上述。

Qu：

1、在代码中如何让声明的留和发送tuple联系起来，因为声明流的名称并不是tuple对象名？

2、是Storm中Spout的nextTuple对应一个线程，还是多个Spout对应一个线程？

answer：在集群中，应该是每个node的JVM中启动一个线程跑spout

4、Bolts

在Topologies中所有的处理都会在bolts中被执行，它能够

过滤tuple、

函数操作、合并

（连接join、聚合aggregation）

、数据库读写

等。Bolt可以做复杂的流传输，需要多步骤、多bolt的连接。

Bolt也可以发射出一个或多个流，它需要使用

[OutputFieldsDeclarer](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/OutputFieldsDeclarer.html)

类的

`declareStream`

`方法`

声明多个流，并且需要指定这个流去使用

OutputCollector

l类的

`emit`

`方法去`

发射。

当你声明一个bolt的输入流时，你需要订阅一个指定的其他组件的流。

每一个流的订阅都是一个个添加。

[InputDeclarer](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/InputDeclarer.html)

类可以声明一个流在默认的流id上。

`declarer.shuffleGrouping("1")`

`说明在组件“1”上订阅了这个默认流，等价于`

`declarer.shuffleGrouping("1", DEFAULT_STREAM_ID)。`

Bolts的主要

方法是

`execute`

方法，它会吸收作为输入的一个新Tuple。Bolts使用

[OutputCollector](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/task/OutputCollector.html)

对象发射新的Tuples。Bolts必须对每一个tuple调用

`OutputCollector`

的

`ack`

方法，以便于Storm知道什么时候元组们被处理完成（可以最终确定它的安全对于包装这个初始化spout tuples）。

共同处理一个输入元组的情况下,发射0或多个元组们基于元组，然后包装输入元组，Storm提供一个

[IBasicBolt](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/IBasicBolt.html)

接口的自动包装。

在Bolts异步处理的时候，完全可以启动新线程；同时

[OutputCollector](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/task/OutputCollector.html)

是线程安全的，可以在任何时候被调用。

Resources:

* [IRichBolt](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/IRichBolt.html)
  : this is general interface for bolts.
* [IBasicBolt](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/IBasicBolt.html)
  : this is a convenience interface for defining bolts that do filtering or simple functions.
* [OutputCollector](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/task/OutputCollector.html)
  : bolts emit tuples to their output streams using an instance of this class
* [Guaranteeing message processing](http://storm.apache.org/releases/1.0.0/Guaranteeing-message-processing.html)

Ps：bolt发送或接收的数据流都可以多对多的进行。

![](http://img.blog.csdn.net/20160711162511934?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

5、Stream groupings 流分组

定义一个拓扑部分是指定了每个bolt门闩的流都应该作为输入被接收。一个流分组定义为：

在门闩的任务之中如何区分流

。

在Storm中有8种流分组方式，通过实现

CustomStreamGrouping

j接口，你可以实现一种风格流分组方式：

Storm 定义了八种内置数据流分组的方式：

1、Shuffle grouping（随机分组）：这种方式会随机分发 tuple 给bolt 的各个 task，每个bolt 实例接收到的相同数量的 tuple 。

2、Fields grouping（按字段分组）：根据指定字段的值进行分组。比如说，一个数据流根据“ word”字段进行分组，

所有具有相同“ word ”字段值的 tuple 会路由到同一个 bolt 的 task 中

。

3、All grouping（全复制分组）：将所有的 tuple 复制后分发给所有 bolt task。每个订阅数据流的 task 都会接收到 tuple 的拷贝。

4、Globle grouping（全局分组）：这种分组方式将所有的 tuples 路由到唯一一个 task 上。Storm 按照最小的 task ID 来选取接收数据的 task 。注意，当使用全局分组方式时，

设置 bolt 的 task 并发度是没有意义的（spout并发有意义）

，因为所有 tuple 都转发到同一个 task 上了。使用全局分组的时候需要注意，因为所有的 tuple 都转发到一个 JVM 实例上，

可能会引起 Storm 集群中某个 JVM 或者服务器出现性能瓶颈或崩溃。

5、None grouping（不分组）：在功能上和随机分组相同，是为将来预留的。

6、Direct grouping（指向型分组）：数据源会调用 emitDirect\(\) 方法来判断一个 tuple 应该由哪个 Storm 组件来接收。只能在声明了是指向型的数据流上使用。

7、Local or shuffle grouping （本地或随机分组）：和随机分组类似，但是，会将 tuple 分发给同一个 worker 内的bolt task （如果 worker 内有接收数据的 bolt task ）。其他情况下，采用随机分组的方式。取决于topology 的并发度，本地或随机分组可以减少网络传输，从而提高 topology 性能。

8、Partial Key grouping

: The stream is partitioned by the fields specified in the grouping, like the Fields grouping, but are load balanced between two downstream bolts, which provides better utilization of resources when the incoming data is skewed.

[This paper](https://melmeric.files.wordpress.com/2014/11/the-power-of-both-choices-practical-load-balancing-for-distributed-stream-processing-engines.pdf)

provides a good explanation of how it works and the advantages it provides.

Resources:

* [TopologyBuilder](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/TopologyBuilder.html)
  : use this class to define topologies
* [InputDeclarer](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/InputDeclarer.html)
  : this object is returned whenever 
  `setBolt`
   is called on
  `TopologyBuilder`
   and is used for declaring a bolt's input streams and how those streams should be grouped

6、Reliability

Storm保证每一个spout tuple都将会在拓扑中完整的被处理。处理过程：它会追踪这个tuple tree被每一个

spout tuple

所触发，并且确定tuple tree已经成功完成。每个拓扑都有一个“信息超时”与之相关联。假如Storm未能检测到一个

spout tuple

已经超时完成，它将舍弃并重新执行这个tuple。

为了改善Storm的可靠性能力，你可以告诉Storm什么时候需要在元组树种创建一个新的边界，告诉Storm无论在什么时候都可以完成处理一个独立的tuple。Bolt们都使用了

[OutputCollector](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/task/OutputCollector.html)

对象去发射tuple。“锚定”（实际上就是mark）的完成于这个emit方法，你可以声明一个元组使用了ack方法而被完成。

以上详细的解释了可靠消息处理。

7、Tasks

每个喷口spout或者门闩bolt都有许多任务在集群中执行。

每一个任务对应一个执行线程，流分组定义了如何从一个任务集到另外一个任务集发送元组。你可以使用

[TopologyBuilder](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/topology/TopologyBuilder.html)

类的setSpout和setBolt方法，为每一个spout或bolt是设置并行度和并发度。

Ps：Tasks可以理解为每个节点上的任务实例，运行在对应executor线程上。

8、Workers

拓扑执行要通过一个或多个worker进程。每一个worker进程都是一个物理的JVM和这个拓扑中执行了一个所有这个任务的子集。

例子：如果拓扑的联合并发数为300，分配了50个worker，因此每一个worker将会执行6个task（task将作为worker的线程）。Storm将会均匀的分配任务到所有worker上。

Resources:

* [Config.TOPOLOGY\_WORKERS](http://storm.apache.org/releases/1.0.0/javadocs/org/apache/storm/Config.html#TOPOLOGY_WORKERS)
  : this config sets the number of workers to allocate for executing the topology

Worker结构：

![](http://img.blog.csdn.net/20160711162307977?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![](http://img.blog.csdn.net/20160711162556247?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

Topology的并发机制：

storm的Worker、Executor、Task默认配置都是1

1、增加worker（本地模式无效，只有一个JVM）

Config

对象的

setNumWorkers\(\)

方法：

Config config = new Config\(\);

config.setNumWorkers\(2\):

2、配置executor 和 task

默认都为1，setXXX指定一个Worker中有几个线程，而后面的setNumXXX指定总共需要执行的tasks数量，因此，一个Thread--Executor中需要跑tasks/threads个任务。

```
    topologyBuilder.setSpout\(SENTENCE\_SPOUT\_ID, spout, 2\);

    // StormBaseSpout -
```

&gt;

StormBaseBolt

```
    topologyBuilder.setBolt\(SPLIT\_BOLT\_ID, bolt\).setNumTasks\(2\).shuffleGrouping\(SENTENCE\_SPOUT\_ID\);

    // StormBaseBolt -
```

&gt;

StormBaseBoltSecond

```
    topologyBuilder.setBolt\(COUNT\_BOLT\_ID, boltSecond, 4\).fieldsGrouping\(SPLIT\_BOLT\_ID, new Fields\("word"\)\);

    // StormBaseBoltSecond -
```

&gt;

StormBaseBoltThird

```
    topologyBuilder.setBolt\(REPORT\_BOLT\_ID, boltThird\).globalGrouping\(COUNT\_BOLT\_ID\);
```

![](http://img.blog.csdn.net/20160711162617200?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

storm的处理保障机制：

![](http://img.blog.csdn.net/20160711162644139?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

1、spout的可靠性

spout会记录它所发射出去的tuple，当下游任意一个bolt处理失败时spout能够重新发射该tuple。在spout的nextTuple\(\)发送一个tuple时，为实现可靠消息处理需要给每个spout发出的tuple带上唯一ID，并将该ID作为参数传递给SpoutOutputCollector的emit\(\)方法：

collector.emit\(new Values\("value1","value2"\), tupleID\);

实际上Values extends ArrayList

&lt;

Object

&gt;

保障过程中，每个bolt每收到一个tuple，都要向上游应答或报错，在tuple树上的所有bolt都确认应答，spout才会隐式调用

ack\(\)方法

表明这条消息（

一条完整的流

）已经处理完毕，将会对编号ID的消息应答确认；处理报错、超时则会调用

fail\(\)方法

。

2、bolt的可靠性

bolt的可靠消息处理机制包含两个步骤：

a、当发射衍生的tuple，需要锚定读入的tuple

b、当处理消息时，需要应答或报错

可以通过OutputCollector中emit\(\)的一个重载函数锚定或tuple：

collector.emit\(tuple, new Values\(word\)\); 并且需要调用一次this.collector.ack\(tuple\)应答。

以上就是storm的基础概念，阅读完后并不能满足你去实现代码的需求，因为需要一个可demo代码，作为模仿的基础。这里就不做提供了，毕竟网上一大堆。

最近在研究Storm源代码，不想与“源码分析”一样只告诉该类代码：结构、方式、用到了什么技术，而希望写一些“特殊”的内容；当然有可能也不能免俗，但尽力写点不同的东西。

内容有不妥的地方，希望大家指正，希望能一起进步，文笔欠佳，见谅。

此处配置的原理，会在接下来会讲到worker和并发解释。



另附，浅谈Storm流式处理框架 - fanyun的博客 - CSDN博客 http://blog.csdn.net/fanyun\_01/article/details/50921678

这篇文章的Storm的架构、处理流程的图解比较容易理解。

