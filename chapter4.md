# Mesos搭建大数据平台实战




#一.  Mesos 搭建大数据平台项目概述





# 二. 搭建环境简介
  本次搭建的环境以及下一节的Mesos集群的部署都是参考我之前在Dockerone上写的一篇文章，因为这个环境是现成的且结构比较全（3个maste节点和3个slave节点)，而且那篇部署教程在写完之后我自己还进行过重新部署，是写的比较全面且坑比较少的部署教程。以下是部署环境的简介：
  ![](83dadc53a396208fa96de2f448e3859e.png)

  如图所示其中master节点都需要运行ZooKeeper、Mesos-master、Marathon，在slave节点上只需要运行master-slave就可以了，但是需要修改ZooKeeper的内容来保证slave能够被master发现和管理。为了节约时间和搞错掉，我在公司内部云平台上开一个虚拟机把所有的软件都安装上去，做成快照进行批量的创建，这样只需要在slave节点上关闭ZooKeeper、Mesos-master服务器就可以了，在文中我是通过制定系统启动规则来实现的。希望我交代清楚了，现在开始部署。

# 三. Mesos集群部署

##  3.1、准备部署环境

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
  

## 3.2、在所有的节点上配置ZooKeeper

  在配置maser节点和slave节点之前，需要先在所有的6个节点上配置一下ZooKeeper，配置步骤如下：
  * 修改zk的内容
  * ```sudo vi /etc/Mesos/zk```
  * 将zk的内容修改为如下：
  * ```zk://10.162.2.91:2181,10.162.2.92:2181,10.162.2.93:2181/Mesos```
 

## 3.3、配置集群中的三个master节点

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


## 3.4、配置集群中的的slave节点
* 配置slave节点的服务启动规则（重启不启动zookeeper和slave服务）
* ```sudo stop ZooKeeper```
* ```echo manual | sudo tee /etc/init/ZooKeeper.override```
* ```echo manual | sudo tee /etc/init/Mesos-master.override```
* ```sudo stop Mesos-master```
* 配置slave节点的识别ip和hostname（以slave1节点为例）
* ```echo 192.168.2.94 | sudo tee /etc/Mesos-slave/ip```
* ```sudo cp /etc/Mesos-slave/ip /etc/Mesos-slave/hostname```

## 3.5、在集群的所有节点上启动相应的服务
* 启动master节点的服务（zookeeper和mesos-master服务）
* ```initctl reload-configuration```
* ```service Zookeeper start```
* ```service Mesos-master start```
* 启动slave节点上的相应服务（mesos-slave服务）
* ```sudo start mesos-slave```

## 3.6、Troubleshooting
   由于有的网络情况和设备情况不一样，所以选举的过程有的快有的慢，但刷新几次就可以完成选举。当发现slave节点有些正常有些不正常时，可以通过reboot来促使自己被master发现。



# 四、Hadoop在Mesos集群上部署

## 4.1、部署前准备
* 在部署和HDFS初始化过程中都需要跨节点的操作和SSH，因此首先在Mesos集群的所有节点上关闭防火墙。如下：
* ```chkconfig iptables off ```（在root用户下操作）
* 关闭selinux
* ```setenforce 0```(在root用户下操作）
* 在HDFS部署中，或者说在分布式架构的部署过程中节点之间的通信都是以主机名为地址标识，因此要保证每个节点主机名的唯一。
* 在Master1节点上配置hostname:
* ```vi /etc/hostname```
* ```master1```
* 在Masrer1节点上配置hosts
* ```vi /etc/hosts```
* ```127.0.0.1 localhost```
* ```10.162.2.91 master1```
* ```10.162.2.92 master2```
* ```10.162.2.93 master3```
* ```10.162.2.94 slave1```
* ```10.162.2.95 slave2```
* ```10.162.2.96 slave3```
* 在其他节点上都参照上述配置进行修改hostname和hosts.
* 在部署HDFS和Hadoop时，需要使用非root用户来进行操作，虽然大部分情况下也可以在root用户下操作，但以往的经验是非root用户下操作部署更顺畅。
* 添加新的用户
* ```addusr hadoop```
* 重置hadoop用户的密码
*```passwd hadoop```

## 4.2、在Mesos集群中部署HDFS
* 本次案例的集群中master有三个节点，slave有三个节点，在master节点中使用zookeeper进行服务选举，在部署HDFS时也会用到zookeeper进行namenode节点的选举。

### 4.2.1、 在master节点上部署namenode
* 创建一个文件目录用于
* ```mkdir -p /mnt/cloudera-hdfs/1/dfs/nn /nfsmount/dfs/nn```
* 修改文件目录的用户权限，给上一步创建的文件目录添加用户hadoop操作权限
* ```chown -R hadoop:hadoop /mnt/cloudera-hdfs/1/dfs/nn /nfsmount/dfs/nn```
* 修改文件目录的操作权限
* ```chmod 700 /mnt/cloudera-hdfs/1/dfs/nn /nfsmount/dfs/nn```
* 使用apt-get 安装hadoop-hdfs-namenode,这样可以避免使用二进制文件在编译过程中遇到的坑。
* ```wget http://archive.cloudera.com/cdh5/one-click-install/precise/amd64/cdh5-repository_1.0_all.deb```
* ```sudo dpkg -i cdh5-repository_1.0_all.deb```
* ```sudo apt-get update; sudo apt-get install hadoop-hdfs-namenode```
*```cp /etc/hadoop/conf.empty/log4j.properties/etc/hadoop/conf.name/log4j.properties```

### 4.2.2、在slave节点上部署datanode
* 创建挂载目录
* ```mkdir -p /mnt/cloudera-hdfs/1/dfs/dn /mnt/cloudera-hdfs/2/dfs/dn /mnt/cloudera-hdfs/3/dfs/dn /mnt/cloudera-hdfs/4/dfs/dn```
* 修改挂在目录所属的用户组
* ```chown -R hadoop:hadoop /mnt/cloudera-hdfs/1/dfs/dn/mnt/cloudera-hdfs/2/dfs/dn /mnt/cloudera-hdfs/3/dfs/dn /mnt/cloudera-hdfs/4/dfs/dn```
* 使用apt-get 安装hadoop-hdfs-datanode
* ```wget http://archive.cloudera.com/cdh5/one-click-install/precise/amd64/cdh5-repository_1.0_all.deb```
* ```dpkg -i cdh5-repository_1.0_all.deb```
* ```sudo apt-get update; sudo apt-get install hadoop-hdfs-datanode```
* ```sudo apt-get install hadoop-client```


### 4.2.3、 格式化并启动namenode节点
* ```sudo -u hadoop hadoop namenode -format```
* ```service hadoop-hdfs-namenode start```

### 4.2.4、启动slave节点
* ```service hadoop-hdfs-datanode start```

### 4.2.5 配置服务自启动
* 在namenode节点上
* ```update-rc.d hadoop-hdfs-namenode defaults```
* ```update-rc.d zookeeper-server defaults```
*在slave节点上
* ```update-rc.d hadoop-hdfs-datanode defaults```

## 4.3、在Mesos集群中部署Hadoop

### 4.3.1、Hadoop的基本安装
* 下载Hadoop安装文件包
* ```wget http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.3.0-cdh5.1.2.tar.gz```
* 将安装文件包解压缩
* ```tar zxf hadoop-2.3.0-cdh5.1.2.tar.gz```
* 下载hadooponmesos二进制文件
* ```git clone https://github.com/mesos/hadoop.git hadoopOnMesos```
* 使用mvn编译hadooponmesos文件
* ```mvn package```
* 注意：mvn package要在下载的hadoopOnMesos文件夹中执行,编译好后可以在target文件夹中找到一个编译好的jar文件。

* 将编译好的jar文件拷贝到hadoo-hdfs安装文件夹和下载的hadoop安装文件夹中
* ```cp hadoopOnMesos/target/hadoop-mesos-0.1.0.jar /usr/lib/hadoop-0.20-mapreduce/lib/```
* ```cp hadoopOnMesos/target/hadoop-mesos-0.1.0.jar hadoop-2.3.0-cdh5.1.2/share/hadoop/common/lib/```
* 配置CDH5使用MRv1，因为在MRv2中hadoop的任务调度是使用yarn的
* ```cd hadoop-2.3.0-cdh5.1.2```
* ```mv bin bin-mapreduce2```
* ```mv examples examples-mapreduce2```
* ```ln -s bin-mapreduce1 bin```
* ```ln -s examples-mapreduce1 examples```
* ```pushd etc```
* ```mv hadoop hadoop-mapreduce2```
* ```ln -s hadoop-mapreduce1 hadoop```
* ```popd```
* ```pushd share/hadoop```
* ```rm mapreduce```
* ```ln -s mapreduce1 mapreduce```
* ```popd```
* 配置Hadoop运行所需的环境和配置文件：
* ```cp target/hadoop-mesos-0.1.0.jar /usr/lib/hadoop-0.20-mapreduce/lib```
* */上一步是将hadoopOnmesos编译好的jar包放到hadoop调用库文件夹中，以便hadoop可以使用mesos调度资源运行job任务。
* ```vim /etc/profile.d/hadoop.sh```
* ```export HADOOP_MAPRED_HOME=/usr/lib/hadoop-0.20-mapreduce```
* ```export MESOS_NATIVE_JAVA_LIBRARY=/usr/local/lib/libmesos.so```
* */上一步是在hadoop的运行脚本中配置HOME路径和Mesos的原生库。
* ```chmod +x /etc/profile.d/hadoop.sh```
* ```/etc/profile.d/hadoop.sh```
* ```cd ..```
* ```rm hadoop-2.3.0-cdh5.1.2-mesos-0.20.tar.gz```
* */删除下载的cdh5原始文件
* ```tar czf hadoop-2.3.0-cdh5.1.2-mesos-0.20.tar.gz hadoop-2.3.0-cdh5.1.2/```
* */上一步是将配置好的hadoop安装文件重新打包
* ```hadoop dfs -put hadoop-2.3.0-cdh5.1.2-mesos-0.20.tar.gz /```
* */上一步是将打包好的hadoop安装包上传到hdfs上

### 4.3.2 Hadoop配置文件配置

* 配置mapred-site.xml
* ```vi /etc/hadoop/conf.cluster-name/mapred-site.xml ```
* mapred-site.xml需要配置文件内容
* ```<name>mapred.jobtracker.taskScheduler</name>```
* ```<value>org.apache.hadoop.mapred.MesosScheduler</value>```
* */上一步是通过配置mapred的jobtrackerd.taskSchedule来告诉Hadoop使用Mesos来调度管理任务。
* ```<name>mapred.mesos.taskScheduler</name>```
* ```<value>org.apache.hadoop.mapred.JobQueueTaskScheduler</value>```
* ```<name>mapred.mesos.master</name>```
* ```<value>zk:10.162.2.91:2181，10.162.2.92:2181，10.162.2.93:2181/mesos</value>```
* */上一步配置是保证mapred能够准确的找到mesos master节点通过zookeeper选举出来的的主节点。
* ```<name>mapred.mesos.executor.uri</name>```
* ```<value>hdfs:/10.162.2.92:9000/hadoop-2.3.0-cdh5.1.2-mesos.0.20.tar.gz</value>```
* */上一步是配置hadoop的路径，这样mapred可以知道到那里调用hadoop代码执行task，这里10.162.2.92是本地主机的IP地址，在10.162.2.91主机上时就改成10.162.2.91.
* ```<name>mapred.job.tracker</name>```
* ```<value>10.162.2.92:9001</value>```
* */上一步是配置jobtracker的主机IP地址，和上一步一样这要配置本机的IP地址
* 配置本地的Mesos原生库
* ```vim /usr/lib/hadoop-0.20-mapreduce/bin/hadoop-daemon.sh```
* ```export MESOS_NATIVE_JAVA_LIBRARY=/usr/local/lib/libmesos.so```
*完成上述配置后尝试启动jobtracker，验证是否安装部署成功。
* ```service hadoop-0.20-mapreduce-jobtracker start```
* 可以通过jps查看jobtracker进程是否在运行
* ```jps```
* 




































#5、Spark计算框架的部署和应用


## 一、部署前需要具备的环境

* HDFS已经部署在Mesos集群环境中
* Mesos集群已经部署成功且正常运行
* 在集群系统中已经具备了Spark安装部署所需的环境，具体可参考（http://spark.apache.org/docs/latest/index.html）

## 二、在Mesos集群上部署Spark

* 在所有集群的节点上使用wget下载Spark安装包
* ```wget http://mirror.bit.edu.cn/apache/spark/spark-1.5.1/spark-1.5.1.tgz```
* 注意：由于在Sparkr任务在Mesos集群的节点上运行时需要在各个节点上进行初始化，因此建议在每个节点上都下载Spark的安装包。















# 6、部署基于Azkaban的工作流管理平台












