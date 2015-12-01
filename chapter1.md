# Mesos概述

云计算时代，由主机构成的集群系统成为主要的计算平台，由它支撑的企业的核心计算服务和科学计算应用。为了管理这些应用，开发者已经开发出来一系列多样的计算框架来简化管理集群的难度。典型的案例包括MapReduce、Kafka、Spark等等。很显然的事情是不断有新的框架在产生，但是我们不得不说，现在没有一个框架能完全满足全部的应用需求。所以，人们希望能不能让多个框架运行在同一个集群里，通过混布框架，我们可以提高资源的综合利用率，并且对于大数据的获取带来了意想不到的便利。


## 1.1 Mesos简介
Apache Mesos是由加州大学伯克利分校的AMPLab首先开发的一款开源群集管理软件，支持 Hadoop、ElasticSearch、Spark、Storm 和 Kafka 等应用架构。Mesos特性如下：

- 可扩展到10000个节点
- 使用 ZooKeeper 实现 Master 和 Slave 的容错
- 支持 Docker 容器
- 使用 Linux 容器实现本地任务隔离
- 基于多资源（内存，CPU、磁盘、端口）调度
- 提供 Java，Python，C++等多种语言 APIs
- 通过 Web 界面查看集群状态
- 新版本将支持更多功能

Mesos架构图是这样的：
![Mesos](http://mesos.apache.org/assets/img/documentation/architecture3.jpg "Title")


* Mesos本身包含两个组件:Master Daemon和Slave Daemon。
    * Master Daemon
        * 管理所有的 Slave Daemon。
        * 用[Resource Offers](https://github.com/Dataman-Cloud/Mesos-CN/blob/master/OverView/Mesos-of-ResourceOffer.md)实现跨应用细粒度资源共享，如 cpu、内存、磁盘、网络等。
        * 限制和提供资源给应用框架使用。
        * 使用可拔插的模块化的架构，方便增加新的策略控制机制。
    * Slave Daemon
        * 负责接收和管理 Master 发来的需求 Task
        * 支持使用各种环境运行各种 Task，如Docker、VM、进程调度(纯硬件)。
        
* Mesos上的task由2个组件管理:调度器(Scheduler)和执行进程(Executor Process)
    * 调度器(Scheduler)
        * 调度器通过注册 Mesos Master获得集群资源调度权限
        * 调度器可通过 MesosSchedule Driver 接口和 Mesos Master 交互
    * 执行进程(Executor Process)
        * 用于启动框架内部的 Task
        * 不同的调度器使用不同的 Executor

* Mesos 集群为了避免单点故障，所以使用 Zookeeper 进行集群交互。


### 1.1.1 Mesos的运行方式

下图描述了一个 Framework 如何通过调度来运行一个 Task
![Resource offer](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)

事件流程:
1. Slave1 向 Master 报告，有4个CPU和4 GB内存可用
2. Master 发送一个 Resource Offer 给 Framework1 来描述 Slave1 有多少可用资源
3. FrameWork1 中的 FW Scheduler会答复 Master，我有两个 Task 需要运行在 Slave1，一个 Task 需要<2个CPU，1 GB内存>，另外一个Task需要<1个CPU，2 GB内存>
4. 最后，Master 发送这些 Tasks 给 Slave1。然后，Slave1还有1个CPU和1 GB内存没有使用，所以分配模块可以把这些资源提供给 Framework2

> 注意：当 Tasks 完成和有新的空闲资源时，Resource Offer会不断重复这一个过程。


### 1.1.2 Mesos与虚拟机、容器技术对比


### 1.1.3 Meoss的应用场景


## 1.2 Meos安装指南


### 1.2.1 Mesos集群组件介绍

### 1.2.2 Mesos生产环境配置介绍



## 1.3 安装Mesos和Zookeeper

### 1.3.1 标准包安装

### 1.3.2 源码包安装


## 1.4 Docker安装配置指南




### 1.4.1 安装方法




### 1.4.2 配置方法




### 1.4.3 配置Mesos Slave与Docker




## 1.5 Mesos升级




### 1.5.1 升级Mesos Master和Slave方法




## 1.6 总结















