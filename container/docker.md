# docker基础知识介绍

什么是docker，引用官方的一句话

    Develop, Ship and Run Any Application, Anywhere
    
docker就是一个这样的工具。它可以帮助开发者很方便的去开发，部署，运行自己的程序。它可以让你非常迅速的测试和部署你的项目到生产环境中。

对于docker的具体实现和原理我们不多讲，让我们直接来做一个简单的例子来体验一下docker的魅力。

首先你需要在你自己的机器上安装docker，安装的文档你可以在这里找到.[install](https://docs.docker.com/installation)

我们这里以Ubuntu14.04系统为例，我们在这个系统上安装docker.

    curl -sSL https://get.docker.com | sh
    
一段美妙的小脚本就被安装到了你的机器上，他完成了你安装docker需要的所有内容。下面我们就开始使用它吧。

如果我们以一个简单的小应用来演示肯定激发不了你的兴趣，那么我们以安装一个wordpress为例，看看docker是如何快速安装一个wordpress 的。

以前安装wordpress,你可能需要去了解PHP，mysql，然后还有你的服务器的系统，最后才是去安装wordpress。非常的麻烦，但是如果我们换一个角度，使用docker来安装呢。



