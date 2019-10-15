## 1、Storm的集群架构

Nimbus，Supervisor，ZooKeeper，Worker，Executor，Task

它们之间的关系见下图

![](D:\File\studyNote\大数据\storm\images\1570676931(1).jpg)

## 2、Storm的核心概念

Topology，Spout，Bolt，Tuple，Stream

### 2.1、Topology

Topology 是对实时计算应用逻辑的封装，会一直运行在集群中，直到你手动去终止他。

### 2.2、Spout

数据源的一个代码组件，就是我们可以写一个 Java 类，实现一个 spout 接口，然后可以自己尝试去数据源获取数据，比如说从 kafka 中消费数据。

数据源（Spout）是拓扑中数据流的来源。一般 Spout 会从一个外部的数据源读取元组然后将他们发送到拓扑中。根据需求的不同，Spout 既可以定义为**可靠的**数据源，也可以定义为**不可靠的**数据源。一个可靠的 Spout 能够在它发送的元组处理失败时重新发送该元组，以确保所有的元组都能得到正确的处理；相对应的，不可靠的 Spout 就不会在元组发送之后对元组进行任何其他的处理。

Spout 中的关键方法是 `nextTuple`。顾名思义，`nextTuple` 要么会向拓扑中发送一个新的元组，要么会在没有可发送的元组时直接返回。需要特别注意的是，由于 Storm 是在同一个线程中调用所有的 Spout 方法，`nextTuple` 不能被 Spout 的任何其他功能方法所阻塞，否则会直接导致数据流的中断（关于这一点，阿里的 JStorm 修改了 Spout 的模型，使用不同的线程来处理消息的发送，这种做法有利有弊，好处在于可以更加灵活地实现 Spout，坏处在于系统的调度模型更加复杂，如何取舍还是要看具体的需求场景

### 2.3、bolt

一个业务处理的代码组局，spout 会将数据传送给 bolt，各种 bolt 还可以串联成一个计算链条，Java 类实现了一个 bolt 接口

一堆 Spout + bolt ，就组成一个Topology，就是一个拓补，实时计算作业，一个拓补涵盖数据源获取/生产+数据处理的所有的代码逻辑

### 2.4、tuple

就是一条数据，每条数据都会被封装在tuple中，在多个 spout 和 bolt 之间传递

### 2.5、stream

就是一个流，抽象的概念。一个数据流指的是在分布式环境中并行创建、处理的一组元组（tuple）的无界序列。

数据流可以由一种能够表述数据流中元组的域（fileds）的模式来定义，默认情况下，元组（tuple）包含基本对象类型

## 3、Storm的并行度及流分组

学习目的：

1、了解如何将storm拓扑打包后提交到storm集群上去运行

2、掌握如何能够通过storm ui去查看你的实时计算拓扑的运行现状

**数据流分组：**在 bolt 的不同任务（task）中划分数据流的方式。简单的说，就是task 与 task之间的数据流向关系。

**并行度：**对于一个拓补来说，并行度其实就是task。有些人会以为是executor，那是因为默认情况下，一个executor只有一个task，executor的数量和task是相等的。最小的计算单元是task，每个 spout/bolt 的代码副本都会运行在一个task中。

在 Storm 中有八种内置的数据流分组方式，而且你还可以通过 `CustomStreamGrouping`接口实现自定义的数据流分组模型。

1. **随机分组（Shuffle grouping）：**这种方式下元组（tuple）会被尽可能随机地分配到 Bolt 的不同任务（tasks）中，使得每个任务所处理元组（tuple）数量能够能够保持基本一致，以确保集群的负载均衡。
2. **域分组（Fields grouping）：**这种方式下数据流根据定义的“域”来进行分组。例如，如果某个数据流是基于一个名为“user-id”的域进行分组的，那么所有包含相同的“user-id”的元组都会被分配到同一个任务中，这样就可以确保消息处理的一致性。
3. 部分关键字分组（Partial Key grouping）：这种方式与域分组很相似，根据定义的域来对数据流进行分组，不同的是，这种方式会考虑下游 Bolt 数据处理的均衡性问题，在输入数据源关键字不平衡时会有更好的性能1。感兴趣的读者可以参考[这篇论文](https://melmeric.files.wordpress.com/2014/11/the-power-of-both-choices-practical-load-balancing-for-distributed-stream-processing-engines.pdf)，其中详细解释了这种分组方式的工作原理以及它的优点。
4. 完全分组（All grouping）：这种方式下数据流会被同时发送到 Bolt 的所有任务中（也就是说同一个元组会被复制多份然后被所有的任务处理），使用这种分组方式要特别小心。
5. 全局分组（Global grouping）：这种方式下所有的数据流都会被发送到 Bolt 的同一个任务中，也就是 id 最小的那个任务。
6. 非分组（None grouping）：使用这种方式说明你不关心数据流如何分组。目前这种方式的结果与随机分组完全等效，不过未来 Storm 社区可能会考虑通过非分组方式来让 Bolt 和它所订阅的 Spout 或 Bolt 在同一个线程中执行。
7. 直接分组（Direct grouping）：这是一种特殊的分组方式。使用这种方式意味着元组的发送者可以指定下游的哪个任务可以接收这个元组。只有在数据流被声明为直接数据流时才能够使用直接分组方式。使用直接数据流发送元组需要使用 [OutputCollector](http://storm.apache.org/javadoc/apidocs/backtype/storm/task/OutputCollector.html) 的其中一个 [emitDirect](http://storm.apache.org/javadoc/apidocs/backtype/storm/task/OutputCollector.html#emitDirect-int-java.lang.String-java.util.List-) 方法。Bolt 可以通过 [TopologyContext](http://storm.apache.org/javadoc/apidocs/backtype/storm/task/TopologyContext.html) 来获取它的下游消费者的任务 id，也可以通过跟踪 [OutputCollector](http://storm.apache.org/javadoc/apidocs/backtype/storm/task/OutputCollector.html) 的 `emit` 方法（该方法会返回它所发送元组的目标任务的 id）的数据来获取任务 id。
8. 本地或随机分组（Local or shuffle grouping）：如果在源组件的 worker 进程里目标 Bolt 有一个或更多的任务线程，元组会被随机分配到那些同进程的任务中。换句话说，这与随机分组的方式具有相似的效果。

