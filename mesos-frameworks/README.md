# Mesos 框架

在 Mesos 出现之前，各个分布式计算框架都是以独占的方式使用集群资源，
Mesos 的出现，使计算框架之间共享集群计算资源成为了可能，以便更好的利用集群资源，
降低部署、运维成本。

目前有许多计算框架都能完美的运行在 Mesos 之上，比如：Hadoop, Spark, Storm,
Chronos, Marathon, Cassandra 等等。

下图是一个可以运行在 Mesos 之上的计算框架元素周期表，可见，Mesos 生态非常庞大。

![FIXME: mesos frameworks](assets/mesos-frameworks.png)

本章我们将介绍几个典型的计算框架：

  - Hadoop, 大数据处理框架
  - Marathon, 长时任务处理框架
  - Chronos, 批处理任务处理框架
  - Cassandra, 数据存储框架

更多的内容等待读者去挖掘！
