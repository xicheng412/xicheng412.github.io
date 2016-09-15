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
sudo apt install wget unzip nginx git sqlite
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

在git用户下操作

```bash
su git
cd /home/git
wget https://dl.gogs.io/gogs_v0.9.97_raspi2.zip
```

用 `wget + 下载地址` 下载安装程序到 `/home/git`

然后得到下载文件之后解压

```bash
unzip gogs*.zip
```

得到文件夹gogs，然后进入gogs并执行文件gogs。

```bash
cd gogs
./gogs web
```

然后可以看到一堆信息，可以看到运行在3000端口上。如果3000端口被占用，需要更换端口的话可以使用 `./gogs web -p 3001` 来指定一个不一样的端口，如3001之类的。

### 对网页端配置

在网页浏览器里面打开ip地址和端口如

* 本机的话就打开 127.0.0.1:3000
* 如果是其他地方访问的话就打开树莓派对应地址，如 192.168.0.123:3000

选择数据库是 sqlite

其他的按照自己的理解选择就可以了，基本上其他设置就默认就好

### 配置nginx代理

nginx的配置是比较简单快速的，安装好nginx以后，默认的nginx配置文件为 `/etc/nginx/nginx.conf`

用 vim 或者 nano 之类的工具打开，并找到

```
##
# Virtual Host Configs
##

include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```

这样的地方，这个是配置server对应的文档将 `include /etc/nginx/sites-enabled/*;` 修改成 `include /etc/nginx/sites-enabled/*.conf;` ，或者前面加 `#` 注释掉。
因为这个地方会加载默认的nginx的80端口配置，会覆盖掉前面的一些配置。我之前就是在这个地方卡了好一会，最后发现是设置被覆盖的问题。

然后在 nginx 的加载位置里面增加自己的配置文件

```bash
cd /etc/nginx/conf.d/
nano default.conf
```

打开了文件以后，粘贴以下的配置到这个文件


```
#gogs port 3000
server{
        server_name 127.0.0.1;
        listen 80;

        location /{
                proxy_pass http://127.0.0.1:3000/;
        }
}
```

这个配置文件就是设置代理访问，这样每次只需要输入ip就可以访问了，不需要输入端口号。它将代理的3000端口，转发向80端口。

如果是在服务器上配置，nginx也会非常方便，也可以反向代理固定的ip地址或是域名。

### 配置supervisor

这个是配置的重中之重，只有supervisor设置稳定了，才能够比较好的访问。最关键的是，我一开始想自己写脚本让gogs运行遇到了一个没有见过的问题，我通过 'sudo git -c /home/git/gogs/gogs web'，发现一个很奇怪的问题，就是这样是不能够进入到这个目录执行的，如果执行的时候工作目录不是 `/home/git/gogs/` 就会使得数据库不能存储。所以用supiervisor也能够解决这个问题当然就是最为方便的，而且能够保持程序在一直运行。

切换到pi的用户，将配置文件输入supervisor.conf，然后再建立我们需要的目录，过一会用来保存supervisor需要的单个的程序配置（我们现在就只有gogs,以后可能会还有，到时候就直接添加配置的文件就好）。

```bash
su pi
sudo echo_supervisord_conf > /etc/supervisord.conf
sudo mkdir -p /etc/supervisor/conf.d/
```

如果遇到权限问题，就先在home生成一个，然后copy到etc目录。

然后来设置supervisor的配置，这个配置文件就是刚才生成的 `/etc/supervisord.conf`，找到里面的`;[include]`的地方，去掉分号注释，修改这个配置为

```
[include]
files = /etc/supervisor/conf.d/*.conf
```

这样以后配置supervisor的时候就只需要写一个配置program的文件到 `/etc/supervisor/conf.d/` 中，然后名字设置为 `程序名.conf` 就可以自动保持运行了。

然后找到supervisor的gogs配置文件，在gogs目录的一个叫scrpit的目录中。

拷贝文件到 `/etc/supervisor/conf.d/` 中去，并且修改名字。

```bash
cp /home/git/gogs/scripts/supervisor/gogs /etc/supervisor/conf.d/gogs.conf
```

修改conf.d中的配置文件gogs.conf为以下设置。可以按照自己的实际情况修改，主要是目录要注意，目录设置决定了sqlite数据库的位置。


```
[program:gogs]
directory=/home/git/gogs/
command=/home/git/gogs/gogs web
autostart=true
autorestart=true
startsecs=10
stdout_logfile=/var/log/gogs/stdout.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/var/log/gogs/stderr.log
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
user = git
environment = HOME="/home/git", USER="git"
```

配置好以后运行命令，打开supervisor的守护进程，通过supervisorctl控制。


```
supervisord
supervisorctl start gogs
```

然后等程序执行了之后，只要状态不是error,就可以通过访问树莓派的ip地址来访问网页控制界面了。

后面的申请账号之类的就和github差不多了。但是要注意第一个账号是管理员账号，如果前面没有设置的话，新申请的第一个账号是有管理员权限的。

## 后记

这个主要是想给实验室的小伙伴们备份代码用，但是现在的方式是备份到树莓派上，不是特别安全，TF 卡不可靠。等后面有时间，应该再做到经常进行异地备份，再备份到云服务器上之类的才好。

主要参考了官方网站的资料，然后还有一些网络上搜索到的东西。

网络上还能够搜索到有一些做备份的资料参考，以后再补全。

还想到了一个问题，就是要做一个内网穿透，等有时间会专门做一个搭建内网穿透的文章。

* [用Gogs搭建自己的Git服务器](https://libhappy.com/2016/01/build-gogs-service/index.html)
* [搭建gogs代码托管服务器](http://blog.just4fun.site/gogs-install.html)

