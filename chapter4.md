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
* 













# Hadoop在Mesos集群上部署

# Spark计算框架的部署和应用

# 部署基于Azkaban的工作流管理平台












