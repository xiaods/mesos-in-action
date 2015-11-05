# Mesos搭建大数据平台实战




#1.  Mesos 搭建大数据平台项目概述



# 2. 搭建环境简介
  本次搭建的环境以及下一节的Mesos集群的部署都是参考我之前在Dockerone上写的一篇文章，因为这个环境是现成的且结构比较全（3个maste节点和3个slave节点)，而且那篇部署教程在写完之后我自己还进行过重新部署，是写的比较全面且坑比较少的部署教程。以下是部署环境的简介：
  ![](83dadc53a396208fa96de2f448e3859e.png)

  如图所示其中master节点都需要运行ZooKeeper、Mesos-master、Marathon，在slave节点上只需要运行master-slave就可以了，但是需要修改ZooKeeper的内容来保证slave能够被master发现和管理。为了节约时间和搞错掉，我在公司内部云平台上开一个虚拟机把所有的软件都安装上去，做成快照进行批量的创建，这样只需要在slave节点上关闭ZooKeeper、Mesos-master服务器就可以了，在文中我是通过制定系统启动规则来实现的。希望我交代清楚了，现在开始部署。

# 3. Mesos集群部署

##  一、准备部署环境

* 在Ubuntu 14.04的虚拟机上安装所有用到软件，并保证虚拟机可以上互联网。

* 安装Python依赖
* ```apt-get install curl python-setuptools python-pip python-dev python-protobuf```
* 安装配置zookeeper
* ```apt-get install ZooKeeperd &
echo 1 | sudo dd of=/var/lib/ZooKeeper/myid```
* 安装配置Mesos－master和Mesos-slave
* ```curl-fL http://downloads.Mesosphere.io/master/ubuntu/14.04/Mesos_0.19.0~ubuntu14.04%2B1_amd64.deb -o /tmp/Mesos.deb ```
* ```dpkg -i /tmp/Mesos.deb```
* ```mkdir -p /etc/Mesos-master```
* ```echo in_memory | sudo dd of=/etc/Mesos-master/registry```
* 安装配置Mesos的Python框架
* ```curl -fL http://downloads.Mesosphere.io/master/ubuntu/14.04/Mesos-0.19.0_rc2-py2.7-linux-x86_64.egg -o /tmp/Mesos.egg```
* ```easy_install /tmp/Mesos.egg```

   至此在一个虚拟机上就完成了所有组件的安装部署，下面就是对虚拟机打快照，然后快速的复制出6个一样的虚拟机，按照上图的ip配置进行配置之后就可以进入下个阶段，当然为了保险你可以测试一下上处组件是否安装成功和配置正确。如果你没有使用云平台，或者不具备快照功能，那就只能在6个虚拟机上重复6遍上处过程了。
  

## 二、在所有的节点上配置ZooKeeper

  在配置maser节点和slave节点之前，需要先在所有的6个节点上配置一下ZooKeeper，配置步骤如下：
  * 修改zk的内容
  * ```sudo vi /etc/Mesos/zk```
  * 将zk的内容修改为如下：
  * ```zk://10.162.2.91:2181,10.162.2.92:2181,10.162.2.93:2181/Mesos```
 
## 三、配置集群中的三个master节点

在所有的master节点上都要进行如下操作：
* 修改ZooKeeper的myid的内容
* ```sudo vi /etc/ZooKeeper/conf/myid```
* 将三个master节点的myid按照顺序修改为1，2，3。
* 修改ZooKeeper的zoo.cfg
* ```sudo vi/etc/ZooKeeper/conf/zoo.cfg```
* 配置内容如下：
* ```server.1=10.162.2.91:2888:3888```
* ```server.2=10.162.2.92:2888:3888```
* ```server.3=10.162.2.93:2888:3888```
* 修改Mesos的quorum
* ```sudo vi /etc/Mesos-master/quorum```
* 将值修改为2。
* 配置master节点的Mesos 识别ip和和hostname(以在master1上的配置为例）
* ```echo 10.162.2.91 | sudo tee /etc/Mesos-master/ip ```
* ```sudo cp /etc/Mesos-master/ip /etc/Mesos-master/hostname```
* 配置master节点服务启动规则（重启不启动slave服务）
* ```sudo stop Mesos-slave
echo manual | sudo tee /etc/init/Mesos-slave.override```


## 四、配置集群中的的slave节点
* 配置slave节点的服务启动规则（重启不启动zookeeper和slave服务）
* ```sudo stop ZooKeeper```
* ```echo manual | sudo tee /etc/init/ZooKeeper.override```
* ```echo manual | sudo tee /etc/init/Mesos-master.override```
* ```sudo stop Mesos-master```
* 配置slave节点的识别ip和hostname（以slave1节点为例）
* ```echo 192.168.2.94 | sudo tee /etc/Mesos-slave/ip```
* ```sudo cp /etc/Mesos-slave/ip /etc/Mesos-slave/hostname```

## 五、在集群的所有节点上启动相应的服务
* 启动master节点的服务（zookeeper和mesos-master服务）
* ```initctl reload-configuration```
* ```service Zookeeper start```
* ```service Mesos-master start```
* 启动slave节点上的相应服务（mesos-slave服务）
* ```sudo start mesos-slave```

## 六、Troubleshooting
   由于有的网络情况和设备情况不一样，所以选举的过程有的快有的慢，但刷新几次就可以完成选举。当发现slave节点有些正常有些不正常时，可以通过reboot来促使自己被master发现。













# Hadoop在Mesos集群上部署




# Spark计算框架的部署和应用







# 部署基于Azkaban的工作流管理平台












