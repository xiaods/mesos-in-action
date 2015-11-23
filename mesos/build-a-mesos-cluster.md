# 搭建一个 Mesos 集群

实践出真知，学习 Mesos 也不例外，这里将介绍怎样搭建一个最小生产集群。
由于 Mesos 对 Linux 支持最好，所以这里将介绍怎样在 Linux 环境下搭建 Mesos 集群；
另外，考虑到在生产环境中主要以 RHEL/CentOS 为主，所以本节内容都基于 CentOS 7。

## 准备工作

在开始搭建 Mesos 集群之前，需要准备以下环境。

  - 3 台 Intel x86/x86_64 架构的计算机
  - 计算机中安装了 CentOS 7.1 操作系统
  - 计算机至少配备了 1core, 1GB 内存

如果准备 3 台物理机有难度，虚拟机也是可以的，介绍怎样准备虚拟机，
怎样安装 CentOS 7.1 操作系统超出了本书的范围，不再赘述。

假设 3 台计算机分别为：

  - A, IP 192.168.1.101
  - B, IP 192.168.1.102
  - C, IP 192.168.1.103

下面我们将在这三台计算机中搭建一个 Mesos 集群。

在开始之前，我们选定 A 作为操作机，大部分的操作都将在 A 上完成，在 A 上通过 SSH
连接到 B 和 C 来完成部署。

### 设置 SSH 环境

在 CentOS 7 上搭建 SSH 非常简单，只需要安装上 SSH 服务器和客户端即可，
并且系统的默认安装已经自带了 SSH 客户端。

由于这里使用 A 作为控制机，所以需要在 B 和 C 上安装 SSH
服务器端，并且配置好账号以便能够从 A 上登录。

下面的操作需要在 B 和 C 上完成，这里以 B 为例。

由于要执行一些特权操作，所以这里直接切换到 root 用户，所以，一定要小心操作，
以免损坏系统。

```
$ su - root
FIXME: output
```

在输入 root 用户密码后，即获得了 root 用户权限。下面来安装 SSH 服务器，
输入如下命令。

```
# yum clean all
# yum makecache
# yum install -y openssh-server
FIXME: output
```

安装完成后，设置 SSH 服务器开机启动，并且启动 SSH 服务器。

```
# systemctl enable sshd.service
FIXME: output
# ssytemctl start sshd.service
FIXME: output
```

`systemctl` 是 systemd 的控制命令，上面两条命令分别为设置 sshd.service
为开机启动和立即启动。

启动后，同样可以通过 `systemctl` 命令来查看 sshd.service 的状态。

```
# systemctl status sshd.service
FIXME: output
```

开启了 SSH 服务器后，该机器上的用户就可以从其它任何可以访问该机器 SSH
服务的地方登陆系统了。

下面从计算机 A 登陆 B，在 A 上执行下面命令：

```
$ ssh -l root 192.168.1.102
FIXME: output
```

在输入 B 机器中 root 账号的密码后，登陆成功了！接下来在这个会话中执行的任何命令都和在计算机
B 上执行的效果一样（当然，除了登出）。

在配置好了计算机 B 的 SSH 登陆后，以同样的方式配置好 C。

## 搭建 ZooKeeper 集群

在搭建 Mesos 集群之前，首先需要搭建 ZooKeeper 集群，因为 Mesos 依赖 ZooKeeper
提供的服务，缺少它，Mesos 将无法工作。

和 Mesos 一样，ZooKeeper 也是 Apache 软件基金会下属的顶级开源项目，
它的目的是为分布式软件开发中涉及到的技术提供通用的实现，以简化分布式软件的开发，
降低复杂度。

包括 Mesos 在内，本章后面将要介绍的 Marathon, Chronos, Storm 等也都使用了
ZooKeeper。

首先，我们需要到 ZooKeeper 项目主页（http://zookeeper.apache.org/）上下载
ZooKeeper，如下图所示：

![zookeeper project](assets/zookeeper-project.png)

这里我们下载目前最新的稳定版本：3.4.6，如下图所示：

![zookeeper download](assets/zookeeper-download.png)

```
$ cd ~/Downloads
$ wget http://apache.fayea.com/zookeeper/current/zookeeper-3.4.6.tar.gz
FIXME: output
```

要提供稳定可靠的 ZooKeeper 服务，至少需要 3 个 ZooKeeper 服务实例，这里将在 A, B
和 C 上搭建一个由 3 个 ZooKeeper 服务实例组成的服务。

**同样，这里的操作如非特别说明，都在计算机 A 上完成。**

首先，解压下载好的 ZooKeeper 压缩包。

```
$ cd ~/Downloads
$ tar -xzf zookeeper-3.4.6.tar.gz
```


