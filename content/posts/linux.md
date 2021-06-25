---
title: VirtualBox 安装 Ubuntu开发环境
date: 2020-07-12 16:57:02
tags: [ubuntu,VirtualBox]
categories: [机器学习]
---

## Linux和ubuntu

首先介绍一下Linux，大家想必都知道windows，和windows，Linux也是操作系统，相对于Windows而言， Linux是完全免费的，开放源码，为用户提供了最大限度的自由度。且Linux更加稳定更加安全，所以通常作为服务器的操作系统使用。

那么，ubuntu是什么呢？ubuntu和Linux的关系，打个比方，就像是一系列手机厂商的EMUI，MIUI和Android的关系，ubuntu也是Linux众多发行版之中较流行的一个（据说适合入门） ，而且网上的教程也多，出现了问题也比较容易找到解决方案。

该安装教程主要借鉴了[原帖](https://zhuanlan.zhihu.com/p/35619204)

## VirtualBox安装

相对于常用的VMware，VirtualBox可能更适合入门折腾。

[VirtualBox 官方下载](https://www.virtualbox.org/wiki/Downloads)

点进去windows用户直接下载第一个就好

## Ubuntu下载

两种渠道，第一种是[官网下载](https://ubuntu.com/download/desktop)（需要科学）

第二种就是阿里云，腾讯云，清华镜像等提供的镜像下载

我选择的使用[阿里云](http://mirrors.aliyun.com/ubuntu-releases/20.04/)，并且下载了20.04最新版

![](https://img.nc-77.top//20200712174229.png)

## 虚拟机创建

- 打开VirtualBox，点击右上角的新建

- 选择Linux类型和ubuntu（64bit)版本

  <img src="https://img.nc-77.top//20200712174643.png" style="zoom:70%;" />

- 点击下一步进行内存大小和硬盘设置（按自己需求即可）

- 硬盘文件类型和存储均选择默认即可

- 最后确定文件存放位置以及硬盘的大小

  <img src="https://img.nc-77.top//20200712175101.png" style="zoom:70%;" />

  

## Ubuntu20.04安装

在启动虚拟机前，还有一些设置要做

- 设置-存储-选择虚拟盘（刚刚下好的ubuntu桌面版64bit）

  ![](https://img.nc-77.top//20200712204507.png)

- 设置-网络-网卡1-连接方式选择桥接网卡（不然主机无法访问虚拟机）

  <img src="https://img.nc-77.top//20200712204649.png" style="zoom:70%;" />

- 启动虚拟机，安装过程中选择中文然后一路点next下来，设置下账号密码就可以了。

  <img src="https://img.nc-77.top//20200712204959.png" style="zoom:50%;" />

