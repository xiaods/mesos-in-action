# 搭建Spark服务

Spark 可以以多种方式运行，最简单的为本地运行方式，适合于开发测试；还可以以 Spark
集群的方式运行；运行在 Mesos 或者 YARN 上。

这里将以两种方式搭建 Spark 服务：集群方式和 Mesos 方式。

## Spark 集群

在生产环境中搭建 Spark 服务，最简单的应该就是以独立 Spark 集群的方式提供 Spark
服务，因为这种方式更简单，不用处理与其它框架整合，例如：Mesos 和 YARN，
然后不可避免的还需要和其它计算框架共享集群。

在开始之前，首先下载 Spark，在编写本书之际，Spark 最新的稳定版本是 1.5.2，可以从
Spark 官网上下载，如下图所示：

![download spark](assets/download-spark.png)

点击页面中的 `Download Spark` 后，跳转到下载页面，如下图所示：

![download spark hadoop](assets/download-spark-hadoop.png)

由于 Spark 使用了 Hadoop client 来访问 HDFS，所以 Spark 需要基于 Hadoop 来构建，
Spark 下载页面提供了多种下载选择：

  - 源代码
  - 不包含 Hadoop 的可执行文件
  - 针对各个特性 Hadoop 版本的可执行文件，例如：Hadoop 2.6 及更新版本

这里下载 `Pre-built for Hadoop 2.6 and later`，下载的文件名为：`spark-1.5.2-bin-hadoop2.6.tgz`，
假设下载到本地的 ~/Downloads 目录下，下载完成后，解压。

```
$ cd ~/Downloads
$ tar xzf spark-1.5.2-bin-hadoop2.6.tgz
$ cd spark-1.5.2-bin-hadoop2.6
$ ls
FIXME: output
```

为了启动集群，需要将 Spark 部署到各个结点中，Spark 集群中只有一个控制结点，
其余均为计算结点。

这里依然假设有 3 台服务器 A, B, C 用来搭建 Spark 集群，其中 A 将作为 Spark
控制结点，运行 Spark master 进程，而 B 和 C 将作为计算节点，运行 Spark worker
进程。

将下载好的 spark-1.5.2-bin-hadoop2.6.tgz 复制到 3 台服务器的 /home/spark
目录下，并且解压。

### Spark master

在服务器 A 上启动 Spark master 进程，如下：

```
$ cd /home/spark/spark-1.5.2-bin-hadoop2.6
$ ./sbin/start-master.sh
FIXME: output
```

Spark master 进程默认会监听本地地址的以下端口：

  - `8080`, Spark master web 用户界面，可以用浏览器查看
  - `7077`, Spark master 服务端口，worker 进程将和 master
    进程通过此端口建立连接

现在，打开浏览器，指向本地地址的 8080 端口，将可以看到 Spark
集群的运行情况，如下图所示：

![FIXME: spark master]()

从 Spark master web 用户界面，可以看到以下几方面的信息：

  - FIXME
  - FIXME

可以看到，目前集群中还没有任何可用的计算结点，所以也不能执行任何任务。
现在，让我们向集群中添加两个计算结点。

### Spark worker

分别在服务器 B 和 C 上执行下面的操作来启动 Spark worker 进程。

```
$ cd /home/spark/spark-1.5.2-bin-hadoop2.6
$ ./sbin/start-slave.sh spark://192.168.1.101:7077
FIXME: output
```

`start-slave.sh` 接收一个必需的参数，指定 Spark master 运行的地址。Spark worker
在启动之后，会向 Spark master 注册自己，并且汇报可用的计算资源，默认情况下，
将汇报所有可用的 CPU 资源以及所有内存减去 1GB，例如：如果计算结点配备了 8GB
的内存，那么 Spark worker 将汇报有 7GB 可用内存，目的是为计算节点自身稳定运行预留
1GB 内存。

Spark worker 默认会监听在本地地址的 8081 端口，提供 web 用户界面，所以，
用浏览器打开 B 结点的 8081 地址，如下图所示：

![FIXME: spark worker]()

从 Spark worker web 用户界面，可以了解到以下几方面信息：

  - FIXME
  - FIXME

### 启动参数配置

在上面启动 Spark master 和 worker 进程时，对于可选参数，都使用了默认值，
在大多数情况下，这都工作得非常好，但是有些时候可能需要调整一些默认参数。
下面分别是 start-master.sh 和 start-slave.sh 接受的可选参数。

#### 公共参数

start-master.sh 和 start-slave.sh 接受以下公共参数。

参数 | 默认值 | 含义
-----| ------ | ----
`-h HOST, --host HOST` | 本地所有地址 | 监听的地址，例如：127.0.0.1, 192.168.1.101
`-i HOST, --ip HOST` | 本地所有地址 | 不再建议使用，建议使用 `-h 或者 --host`
`-p PORT, --port PORT` | 对于 master 来说是 7077，slave 为随机端口 | master 或 slave 和对方通信的端口
`--webui-port PORT` | 对于 master 来说是 8080, slave 是 8081 | master 或 slave web 用户界面监听的端口
`--properties-file FILE` | conf/spark-defaults.conf | 属性配置文件

#### start-master.sh 参数

除了公共参数外，start-master.sh 并没有其它特有的参数。

#### start-slave.sh 参数

除了公共参数外，start-slave.sh 还有一些特有的参数：

参数 | 默认值 | 含义
-----| ------ | ----
`-c CORES, --cores CORES` | 本机所有的 CPU | 向 master 注册的可用 CPU 核数
`-m MEM, --memory MEM` | 本机所有的内存减去 1GB | 向 master 注册的可用内存
`-d DIR, --work-dir DIR` | SPARK_HOME/work | 本地工作目录

例如，我们可以修改 `--cores`, `--memory` 为本机预留更多的资源，以便计算结点自身更加稳定。
只需要在启动 `start-slave.sh` 时指定即可，例如：

```
$ ./sbin/start-slave.sh --cores 7 --memory 14G spark://192.168.1.101:7077
```

### 集群辅助脚本

除了以 `start-master.sh`, `start-slave.sh` 分别启动 master 和 slave
进程外，Spark 还提供了一次性启动或停止集群的方式，包括以下脚本，
它们都位于 Spark 的 `sbin` 目录下。

  - start-slaves.sh, 在所有 `conf/slaves` 文件中指定的计算节点上启动
    slave 进程
  - start-all.sh, 启动 master 和所有 slave 进程
  - stop-slaves.sh - 在所有 `conf/slaves` 文件中指定的计算节点上停止 slave 进程
  - stop-all.sh - 停止整个集群，包括 master 和 slave 进程

这些脚本都通过 SSH 的方式来工作，所以需要事先在其它结点上配置好基于公钥认证的 SSH 登陆配置。
详细配置方法可以参考本章开始。

### 高可用性

Spark 集群的高可用性包含两方面：Spark master 的高可用性和 Spark worker
的高可用性。

Spark worker 的高可用性通过将运行在其上的任务转移到其它结点来实现，所以 Spark
本身即实现了 worker 的高可用性。

而 Spark master 在整个集群中只有一个，所以是一个单点故障，
从而需要借助其它方式来实现 master 的高可用性。Spark 提供了两种方式：

  - 基于 ZooKeeper
  - 基于本地文件

#### 基于 ZooKeeper

在前面的章节已经介绍过 ZooKeeper 的基础知识，以及怎样搭建 ZooKeeper 服务。
而且包括：Mesos, Marathon, Chronos 都无一例外的使用了 ZooKeeper，Spark
也不例外，使用 ZooKeeper 作为其 master 高可用性方案也是首选。

和 Marathon, Chronos 类似，Spark master 也可以将集群元数据存储在 ZooKeeper
之中，以便在重新启动，或者其它 master 被选举为 Leader 时重新加载数据，
从而继续提供服务。

所以，基于 ZooKeeper 实现 Spark master 的高可用性非常容易理解：
在不同机器上以相同的配置启动多个 Spark master 进程，都注册到同一个 ZooKeeper
服务中即可。相关配置如下：

  - spark.deploy.recoveryMode, 设置为 `ZOOKEEPER`
  - spark.deploy.zookeeper.url, 设置 ZooKeeper 集群地址
  - spark.deploy.zookeeper.dir, ZooKeeper 中用来存储数据的目录，默认为 `/spark`

这些参数都通过 SPARK_DAEMON_JAVA_OPTS 来设置，这个变量可以在 conf/spark-env.sh
中设置。例如：

```
$ cd /home/spark/spark-1.5.2-bin-hadoop2.6
$ cp conf/spark-env.sh.template conf/spark-env.sh
```

默认这个文件中没有开启任何配置，所以在这个文件中添加如下一行来配置上面的参数。

```
SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=192.168.1.101:2181,192.168.1.102:2181,192.168.1.103:2181
```

注意，需要修改所有 Spark 控制结点上的配置。同时，由于现在启动了多个 Spark
master 进程，当当前作为 Leader 的 master 故障时，为了能够让 worker
能够自动注册到新的 Leader 结点，worker 在启动时需要同时指定所有 Spark
master。如下：

```
./sbin/start-slave.sh spark://host1:port1,host2:port2
```

如果我们在服务器 A 和 B 上启动了 Spark master，那么可以以下面的方式启动 worker

```
./sbin/start-slave.sh spark://192.168.1.101:7077,192.168.1.102:7077
```

#### 基于本地文件

Spark master 除了能够将集群元数据存储在 ZooKeeper
之外，还可以将数据存储在本地文件，只需要配置以下两个参数：

  - spark.deploy.recoveryMode, 设置为 FILESYSTEM
  - spark.deploy.recoveryDirectory, 设置为一个本地目录

和基于 ZooKeeper 方式一样，通过设置 conf/spark-env.sh 文件中的
SPARK_DAEMON_JAVA_OPTS 变量即可。

基于本地文件方式的高可用性假设本地文件系统足够可靠，并且能够及时重新启动 Spark
master 进程。所以，这种方式本质上不能称作高可用性，也不推荐在生产环境中使用。
因为首先本地文件并不是可靠的，硬件可能损坏，文件系统可能损坏，操作系统也可能损坏，
所以很可能需要花很长一段时间来恢复服务。

当然，为了避免本地文件系统的不可用性，读者也可以使用网络文件系统，
但这或许会引入性能问题或者其它复杂度的问题。所以，总是推荐在生产环境中使用基于
ZooKeeper 的高可用性实现方式。

## Spark on Mesos

