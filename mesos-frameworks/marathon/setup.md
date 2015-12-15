# 搭建Marathon服务

在编写本书时，Marathon 最新的稳定版本是 0.13.0，所以这里将以 Marathon 0.13.0
版本为例来搭建 Marathon 服务。

## 准备环境

Marathon 是运行在 Mesos 之上的长时任务处理框架，并且依赖 ZooKeeper
服务来持久化数据，所以在开始搭建 Marathon 服务之前，首先需要有可用的 Mesos
集群以及 ZooKeeper 服务。

这里我们将使用在前一章中搭建的 Mesos 集群以及 ZooKeeper 服务，虽然 Mesos
也是基于此 ZooKeeper 服务，但是这里纯属巧合，Marathon 可以使用任何其它可用的
ZooKeeper 服务。

所以，Mesos 集群的地址为：
zk://192.168.1.101:2181,192.168.1.102:2181,192.168.1.103:2181/mesos；
ZooKeeper 服务地址为：
zk://192.168.1.101:2181,192.168.1.102:2181,192.168.1.103:2181。

这里，我们选择在 192.168.1.101 上搭建 Marathon 服务，也就是 Mesos
集群中的其中一台，我们在上一章中称为 A 的机器，这里我们继续称之为 A。

之所以选择 A，而不是在另外一台没有安装 Mesos 的机器上运行 Marathon，是因为
Marathon 依赖于 Mesos 库，所以，如果读者要在另外一台机器上安装 Marathon，
请参考上一章的内容首先安装 Mesos，当然，只需要安装即可，不用启动 mesos-master
或者 mesos-slave 进程，相反，你应该禁止它们。

## 下载 Marathon

首先，到 mesosphere 网站下载 Marathon，如下图所示：

![FIXME: download marathon]()

假设将 Marathon 下载到了 ~/Downloads 目录下，或者执行下面的命令进行下载：

```
$ cd ~/Downloads
$ curl -O http://downloads.mesosphere.com/marathon/v0.13.0/marathon-0.13.0.tgz
```

使用下面的命令解压下载好的压缩包

```
$ tar xzf marathon-0.13.0.tgz
$ tree
FIXME: output
```

执行解压后的 `bin/start` 脚本即可启动 marathon，如下所示：

```
$ ./bin/start --master zk:///mesos --zk zk://192.168.1.101:2181,192.168.1.102:2181,192.168.1.103:2181/mesos \
> --zk zk://192.168.1.101:2181,192.168.1.102:2181,192.168.1.103:2181/marathon
```

上面命令中有两个参数：

  - `--master`, 指定 Mesos 集群控制结点服务地址
  - `--zk`, 指定 ZooKeeper 服务地址

Marathon 将会把自己注册到 Mesos 集群中，并且将数据持久化到 `--zk` 指定的
ZooKeeper 服务中的指定地址。

现在，打开 Mesos 服务的 Web UI，就可以看到刚刚注册的 Marathon
服务了，如下图所示：

![FIXME marathon registered]()

另外，Marathon 会默认在 `8080` 端口启动 Web UI，所以，将浏览器指向
http://192.168.1.101:8080 将能够看到 Marathon Web UI，如下图所示：

![FIXME: marathon web ui]()

到现在为止，一个可用的 Marathon 服务就搭建起来了，非常简单，Marathon
还有一些可配置的参数，这里介绍一些读者可能会用到的参数。

## Marathon 参数

marathon 只有一个必须的参数，即 `--master`，以便知道 Mesos 的服务地址。
其它一些比较常用的可选参数如下表所示：

参数                     |默认值                    |示例                     |含义
------------------------|-------------------------|-------------------------|-----
`--zk` | 无 | `--zk=zk://host1:port1,host2:port2,host3:port3/path` | 指定 ZooKeeper 服务地址，指定后 marathon 将会使用 ZooKeeper 作为持久化存储后端
`--[disable_]checkpoint`|`--checkpoint`           |`--checkpoint `          |是否开启任务 checkpoint， 开启 checkpoint 后，在 mesos-slave 重启或者 marathon failover 期间，任务会继续运行；注意：开启 checkpoint 必须在 mesos-slave 相应地开启 checkpoint，如果关闭，则任务会在 mesos-slave 重启或者 marathon failover 期间失败
`--failover_timeout` | 604800 | `--failover_timeout=86400` | 设置 mesos-master 允许 marathon failover 的时间，如果 marathon 没有在此时间内恢复，mesos 将删除 marathon 的任务
`--hostname` |主机的 hostname | `--hostname=192.168.1.101` | 如果你的机器主机名没有被正确配置，很可能需要手动指定，否则可能导致 marathon 不能和 mesos 通信，或者不能和其它 marathon 服务实例通信
`--mesos_role` | 无 | `--mesos_role=marathon` | 设置其在 mesos 中的 role，mesos 的资源预留和共享机制建立在 role 之上，默认不指定 role 的框架具有的 role 为 `*`，表示使用共享资源，注意：这里指定的 role 必须是在 mesos-master 中指定的 `roles` 中的值
`--default_accepted_resource_roles` | 所有资源 | `--default_accepted_resource_roles=marathon` | 接受具有指定 role 类型的资源，注意，所有资源类型都必须具有指定 role，例如：cpus(marathon):20;mem(*):20480; 将不被接受，因为内存只有 `*` 资源，而没有 `marathon` 资源
`--task_launch_timeout` |300000, 5 分钟| `--task_launch_timeout=1800000` |设置任务启动时间，也就是从提交任务到任务进入 RUNNING 的时间，通常来说，如果需要进行比较长的准备时间，需要将该值增大，例如：从 docker-registry 下载镜像
`--event_subscriber` | 无 | `--event_subscriber=http_callback` | 设置开启的事件订阅模块，目前只支持 `http_callback` 一种类型，开启后，marathon 将接受用户的事件订阅，并且相应地在发生事件时，回调注册的 http_callback，并且将事件内容以 JSON 的方式传递给 http_callbak
`--http_address` | 所有网络地址 | `--http_address=192.168.1.101` | 监听的网络地址，通常来说，Linux 系统中都会有一个本地回环 IP: 127.0.0.1，往往映射到主机名 localhost，只能通过本机访问，另外，还有至少一张配置好的网卡和其它主机通信，例如：192.168.1.101
`--http_credentials` | 无 | `--http_credentials=admin:adminpass` | marathon basic auth 的用户名和密码
`--http_port` | 8080 | `--http_port=80` | marathon 服务监听的端口
`--http_max_concurrent_requests` | 无 | `--http_max_concurrent_requests=100` | 最大并发请求数，当请求队列超过该值时，直接返回 503 错误代码，如果 marathon 服务并发很大，那么为了避免服务不稳定或出现故障，最好设置该值，以便客户端收到失败返回后重试

Marathon 的参数还可以通过环境变量来配置，这和 Mesos 类似，只需要将参数全部大写，并且加上 MARATHON_ 前缀，例如：

- `--zk` 参数可以通过环境变量 `MARATHON_ZK` 来指定
- `--mesos_role` 可以通过环境变量 `MARATHON_MESOS_ROLE` 来指定

需要注意的是，如果一个参数同时通过环境变量指定，又通过命令行参数指定，那么命令行参数将会覆盖环境变量。
