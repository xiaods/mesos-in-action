# 搭建Chronos服务

在了解了 Chronos 的工作原理和特性后，这里将介绍怎样搭建一个生产环境可用的 Chronos 服务，生产环境可用意味着要实现服务的高可用性。

我们知道 Chronos 将所有需要持久化的数据都存储到了 ZooKeeper 服务中，所以 Chronos 数据的可靠性由 ZooKeeper 服务来保障，由于 Chronos 是单一 Leader 模型，所以本身不存在数据一致性的问题，但是由于 ZooKeeper 服务是分布式的，所以需要保障数据的一致性。

本节首先介绍 Chronos 高可用性模型，然后动手搭建一个高可用的 Chronos 服务，更进一步，我们将 Chronos 作为一个 Marathon App 运行在 Marathon 之上，从而进一步减少对 Chronos 的运维。

## Chronos 高可用性

得益于将所有数据持久化在 ZooKeeper 服务中，Chronos 的高可用性模型非常简单。Chronos 使用 ZooKeeper 来实现服务内部实例之间的 Leader 选举，任意时刻只有一个服务实例作为 Leader 提供服务，其它实例均作为跟随者，虽然可以接受请求，但是并不对请求进行处理，而是简单的转发，Chronos 会监听 Leader 改变事件，实时知道当前的 Leader 实例。

下图是一个高可用性的 Chronos 服务在生产环境中运行的示例：

![FIXME: chronos in production](assets/chronos-in-production.png)

Chronos 的高可用性非常简单，在任意时刻只需要有一个 Chronos 实例在运行即可提供服务，所以，由两个实例组成的 Chronos 服务即可提供高可用服务。

## 搭建 Chronos 服务

在编写本书之时，Chronos 最新发布的稳定版本是 2.4.0，并且支持 Mesos 0.24.0，而当前 Mesos 最新的稳定版本是 0.25.0，所以这里我们将搭建 Chronos-2.4.0_mesos-0.25.0，所以需要重新基于 Mesos 0.25.0 编译 Chronos。

### 编译 Chronos


### 运行单个 Chronos 服务


### 搭建高可用 Chronos 服务


### Chronos 配置参数


## 将 Chronos 运行在 Marathon 上


## 小结

本节介绍了 Chronos 高可用模型，怎样搭建高可用的 Chronos 服务，以及将 Chronos 运行在 Marathon 之上，进一步提高可用性，降低对 Chronos 的维护成本。