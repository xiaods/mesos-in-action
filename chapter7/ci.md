# 持续集成介绍

持续集成的概念已经在概述中进行了简单介绍，鉴于本书以实战为主，这里对持续集成的概念不再进一步讨论，感兴趣的朋友可以网上搜索几篇博客看下。

本章主要结合实践经验讨论下持续集成的下一步－－**持续交付**(Continuous delivery)。持续交付指的是，频繁地将软件的新版本，交付给质量团队或者用户，以供评审。如果评审通过，代码就进入生产阶段。

下面的图片是我在[atlassian wiki](https://chrisshayan.atlassian.net/wiki/display/my/2013/07/23/Continuous+Delivery+Matrix)下载的持续交付成熟度矩阵

![持续交付成熟度矩阵](ContinuousDeliveryMatrix.png)

通过这张图片我们可以看出，持续交付本身也是一个逐步迭代，逐步完善的过程。从初期的 nightly build 到最终的 镜像交付； 从 merges are rare 到自动生成 ReleaseNote；从简单的将构建结果通知committer到向客户分享构建报告与统计结果，等等。我们可以发现，一个成熟的继续交付系统可以极大的提高生产率。