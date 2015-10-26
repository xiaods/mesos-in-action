# Mesos搭建持续集成系统实战

**持续集成(CI)**是一种软件开发实践，使用得当，它会极大的提高软件开发效率并保障软件开发质量；**Jenkins**是一个开源项目，它提供了一种易于使用的持续集成系统；**Mesos**是Apache下的一个开源的统一资源管理与调度平台，它被称为是分布式系统的内核；**Marathon**是注册到Apache Mesos上的管理长时应用(long-running applications)的framework，如果把Mesos比作数据中心kernel的话，那么Marathon就是init或者upstart的daemon。

  本文旨在探讨如何利用Jenkins，Apache Mesos和Marathon搭建一套**弹性**的，**高可用**的持续集成环境。


###为什么要把Jenkins运行到Apache Mesos上

  把Jenkins运行到Apache Mesos上，或者说利用Apache Mesos向Jenkins提供slave资源，最主要的目的是利用Mesos的弹性资源分配来提高资源利用率。通过配置**Jenkins-on-Mesos**插件，Jenkins Master可以在作业构建时根据实际需要动态的向Mesos申请slave节点，并在构建完成的一段时间后将节点归还给mesos。

  同时，Marathon会对发布到它之上的应用程序进行健康检查，从而在应用程序由于某些原因意外崩溃后自动重启该应用。这样，选择利用Marathon管理Jenkins Master保证了该构建系统的全局高可用。而且，Jenkins Master本身也通过Marathon部署运行在Mesos资源池内，进一步实现了资源共享，提高了资源利用率。

 下面两张图形象的说明了Marathon将Jenkins Master部署到Mesos资源池，以及Jenkins Master使用Mesos资源池进行作业构建的整个过程。 

  <img src="/assets/how-marathon-run-jenkins-on-mesos.png" style="width: 630px; height: 200px;" alt="Jenkins Master实例信息"/>
  <img src="/assets/how-jenkins-master-run-on-mesos.png" style="width: 600px; height: 210px;" alt="Jenkins Master运行在Mesos上"/>

###环境设置

  为了便于理解，这里我简化了Mesos/Marathon集群的架构，不再考虑集群本身的高可用性。至于如何利用zookeeper配置高可用的mesos/marathon集群，可以参考[Mesosphere的官方文档](https://mesos.apache.org/documentation/latest/mesos-architecture/)，这里不再展开。

  我搭建了一个包含40个节点 ``192.168.3.4-192.168.3.43`` 的Mesos集群，其中一个节点用作运行Marthon及Mesos-master，其它39个节点作为mesos的slave，如下所示。

    192.168.3.4  marathon/mesos-master
    192.168.3.5  mesos-slave
    192.168.3.6  mesos-slave
    ......
    192.168.3.43  mesos-slave

参照[http://get.dataman.io](http://get.dataman.io)的文档配置启动Marathon，Mesos-Master和Mesos-Slave，下面的整个操作都将在这个集群上完成。
  

###在Marathon上部署Jenkins的master实例

  Marathon支持web页面或者RESTapi两种方式发布应用，在``192.168.3.＊``内网执行下面的bash命令，就会通过Marathon的RESTapi在mesos slave上启动一个Jenkins master实例。

    git clone git@github.com:Dataman-Cloud/jenkins-on-mesos.git && cd jenkins-on-mesos && curl -v -X POST \
    -H 'Accept: application/json' \
    -H 'Accept-Encoding: gzip, deflate' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'User-Agent: HTTPie/0.8.0' \
    -d@marathon.json \
    http://192.168.3.4:8080/v2/apps

  *这里我在github上fork了[mesosphere的jenkins-on-mesos的repo](https://github.com/mesosphere/jenkins-on-mesos)到[DataMan-Cloud/jenkins-on-mesos](https://github.com/Dataman-Cloud/jenkins-on-mesos)，并进行了一些[改进](https://github.com/Dataman-Cloud/jenkins-on-mesos/commits?author=vitan)。*
  
  如果Jenkins master实例被成功部署，通过浏览器访问``http://192.168.3.4:8080``(**请确定你的浏览器能够访问内网，譬如可以利用设置浏览器代理等方式来搞定**)可以在running tasks列表中找到jenkins，点击进入详细信息页面，我们会看到下图：

  <img src="/assets/jenkins-master-on-marathon.png" style="width: 750px; height: 450px;" alt="Jenkins Master实例信息"/>

 访问``http://192.168.3.4:5050/#/frameworks``并在**Active Frameworks**中找到Marathon，点击进入详细信息页面，可以在该页面找到Jenkins Master具体运行到Mesos哪一台Slave上，如下图所示：

  <img src="/assets/jenkins-master-on-mesos-slave.png" style="width: 750px; height: 450px;" alt="Jenkins Master运行在mesos slave上"/>

  点击sandbox

  <img src="/assets/jenkins-master-on-mesos-slave-2.png" style="width: 750px; height: 300px;" alt="Jenkins Master运行在mesos slave上"/>


###配置Jenkins Master实现弹性伸缩

  接下来是配置Jenkins注册成为Mesos的Framework，需要通过浏览器访问``http://192.168.3.25:31052/``来到Jenkins Master的UI页面。下面的截图是我逐步配置的全过程。

  <img src="/assets/jenkins-configure.png" style="width: 750px; height: 400px;" alt="Jenkins Master配置页面"/>
  <img src="/assets/jenkins-mesos-configure.png" style="width: 750px; height: 400px;" alt="Jenkins Master配置Mesos"/>

  如果Jenkins在Mesos上注册成功，访问``http://192.168.3.4:5050/#/frameworks``，我们可以找到jenkins Framework，如下图所示：

  <img src="/assets/jenkins-framework-on-mesos.png" style="width: 750px; height: 300px;" alt="Jenkins Framework on Mesos"/>

  现在我们可以同时启动多个构建作业来看一下Jenkins在Mesos上的弹性伸缩，在``http://192.168.3.25:31052/``上新建一个名为``test``的工程，配置其构建过程为运行一个shell命令``top``，如下图所示：

  <img src="/assets/test-job-config.png" style="width: 750px; height: 450px;" alt="配置构建作业"/>

  把该工程复制3份``test2``、``test3``和``test4``，并同时启动这4个工程的构建作业，Jenkins Master会向Mesos申请资源，如果资源分配成功，Jenkins Master就在获得的slave节点上进行作业构建，如下图所示：

  <img src="/assets/building-jobs.png" style="width: 750px; height: 340px;" alt="构建作业列表"/>

因为在前面的系统配置里我们设置了*执行者数量*为2（即最多有两个作业同时进行构建），所以在上图中我们看到两个正在进行构建的作业，而另外两个作业在排队等待。

  下图展示了当前的Jenkins作业构建共使用了0.6CPU和1.4G内存,

  <img src="/assets/jenkins-utilization.png" style="width: 750px; height: 450px;" alt="Jenkins资源使用"/>

  正在使用的slave节点的详细信息

  <img src="/assets/jenkins-slave-detail.png" style="width: 750px; height: 400px;" alt="Jenkins slave on mesos"/>
  <img src="/assets/jenkins-slave.png" style="width: 750px; height: 300px;" alt="Jenkins slave详细信息"/>
  
####配置Jenkins Slave参数(可选)

  在使用Jenkins进行项目构建时，我们经常会面临这样一种情形，不同的作业会有不同的资源需求，有些作业需要在配置很高的slave机器上运行，但是有些则不需要。为了提高资源利用率，显然，我们需要一种手段来向不同的作业分配不同的资源。通过设置Jenkins Mesos Cloud插件的slave info，我们可以很容易的满足上述要求。 具体的配置如下图所示：

  <img src="/assets/jenkins-config-slave.png" style="width: 750px; height: 450px;" alt="Jenkins 配置 slave"/>


###总结

  利用mesos为jenkins弹性的提供资源，同时配置Jenkins Slave的参数来满足不同作业的资源需求，这些都大大提高了集群的资源利用率。另外，由于 Marathon 会自动检查运行在它之上的app的健康状态， 并重新发布崩溃掉的应用程序。