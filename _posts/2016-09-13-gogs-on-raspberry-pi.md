---
layout: post
title:  "在树莓派上搭建Gogs"
date:   2016-09-13 00:00:00
categories: RaspberryPi
tags: RaspberryPi Git Gogs
---

* content
{:toc}

## 概述

搭建平台： 树莓派3
使用工具： Go Git Service
完成功能： 快速方便的实现了自助的 Git 服务，可以在网页上管理，有用户、分组等概念，和Github的感觉非常接近。







## Gogs 简介

Gogs (Go Git Service) 是一款极易搭建的自助 Git 服务。

Gogs 的目标是打造一个最简单、最快速和最轻松的方式搭建自助 Git 服务。使用 Go 语言开发使得 Gogs 能够通过独立的二进制分发，并且支持 Go 语言支持的 所有平台，包括 Linux、Mac OS X、Windows 以及 ARM 平台。

* 这里是：  [官方网址](https://gogs.io)
* 但是官网速度特别慢，具体的内容可以到Github上查看：  [Github地址](https://github.com/gogits/gogs)

## 树莓派介绍

树莓派（英语：Raspberry Pi），是一款基于Linux的单板机电脑。它由英国的树莓派基金会所开发，目的是以低价硬件及自由软件促进学校的基本计算机科学教育。

树莓派3代B型于2016年2月29日正式发布。3代控制器针对2代控制器增加了WiFi和蓝牙功能，而且电源也增加到2.5A /5V，但是外观未做更改，与其上一代相比，速度更快，功能更强大。凭借其内置的无线和蓝牙连接，它将成为支持物联网的理想解决方案。Raspberry Pi 3搭载了64位四核1.2GHz处理器，1GB LPDDR2内存，兼容现已发布的应用程序。网络方面，Raspberry Pi 3还直接搭载了激动人心的802.11n Wi-Fi和蓝牙4.1支持。更重要的是价格合理。

本次升级除了 Wi-Fi 和蓝牙外，Raspberry Pi 3 最重要的更新莫过于性能了。Raspberry Pi 2 B 拥有一块四核高通 900MHz 处理器、1GB RAM 和 VideoCore IV GPU。新版本则拥有博通 BCM2837 1.2GHz 四核处理器、1GB RAM 和 VideoCore IV GPU。另外，GPU 的规格也从 250MHz 上升至 400MHz，RAM 从 450MHz 升至 900MHz。其进步可以说非常明显。从跑分结果来看，Raspberry Pi 3 相比上一代高出了 700 多分，这应该说是目前比较热门的迷你电脑了。 

[维基百科的介绍](https://zh.wikipedia.org/zh-cn/%E6%A0%91%E8%8E%93%E6%B4%BE)

## 安装步骤

下面就是比较完整的安装步骤，发现很多地方官网和现有资料讲的不太仔细。我主要采用了sqlite3的数据库，所以不需要额外配置数据库。其他的教程大部分都用的Mysql，所以有一些地方不太一样。

### 准备工具

先安装一些可能会用到的工具，如果有了就跳过。

* wget --用于下载安装程序
* supervisor  --用于维持gogs运行，但是要依赖python2,所以要安装Python
* python
* pip --用于安装supervisor，包管理软件

```bash
sudo apt install wget nginx git sqlite
```

完成下载工具的安装和nginx的安装，还有就是Git和sqlite是gogs依赖的必须要的工具。

然后开始安装python,和supervisor。


```bash
sudo apt install python
sudo easy_install pip
sudo pip install supervisor
```

到此，基本的准备工作完成。


### 新建用户

由于gogs需要运行在git用户的目录下面，所以需要创建一个名字叫做git的新用户。（在gogs里面不用git用户应该也是可以的，但是需要很多额外的配置，所以暂时还是继续用git用户）

运行


```bash
sudo adduser git
su git
mkdir ~/.ssh
```

如果需要修改git用户的密码，可以在 `su git` 命令之后运行 `passwd` 来设置

### 安装gogs

安装本身是极为简单方便的，我下载的二进制包，正如官网所说，这个是可以达到下载-解压-运行就可以安装好。

下载地址可以到Github上面去看上面的release放出来的下载地址，官网也有对应的下载地址

* [官网二进制文件下载地址列表](https://gogs.io/docs/installation/install_from_binary)
* [Github的release列表](https://github.com/gogits/gogs/releases)

我下载的时候是在官网下载的 Raspberry Pi v2，这个版本是可以用sqlite的。
我一开始选的是ARM版本，发现ARM版本不能选择sqlite，所以要注意，最好不用ARM版本。

用 `wget + 下载地址` 下载安装程序到 `/home/git/gogs`
### 配置nginx代理

nginx的配置是比较简单快速加暴力的，

### 配置supervisor

切换到pi的用户，将配置文件输入supervisor.conf

'''
echo_supervisord_conf > /etc/supervisord.conf
'''

如果遇到权限问题，就先在home生成一个，然后copy到etc目录。

