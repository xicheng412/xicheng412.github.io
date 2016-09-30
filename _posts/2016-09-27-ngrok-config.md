---
layout: post
title:  "内网穿透 ngrok 服务器和客户端配置"
date:   2016-09-27 00:00:00
categories: ngrok
tags: ngrok supervisor
---

* content
{:toc}


## 前言

实验室有很多电脑，想要把这些电脑的服务比较方便的配置到网上访问，所以需要内网穿透。

在网上搜索了很多资料，自己摸索了几天以后才配置好，主要的问题就是要特别注意整个过程，每一个过程基本上都是有先后顺序的，不能随意变更，否则容易导致最终结果不成功。







## ngrok 简介及作用


[简介转自互联网](http://blog.lzp.name/archives/24)

ngrok 是一款用 go 语言开发的开源软件，它是一个反向代理，通过在公共的端点和本地运行的 Web 服务器之间建立一个安全的通道。下图简述了 ngrok 的原理。


### 完美代替“花生壳”软件。

“花生壳”是一款老牌的内网穿透软件，一年的内网版服务要两百多块钱，都快可以买一年垃圾点的 VPS 服务器了。而免费版的“花生壳”稳定性较差，隔三差五的不能访问，每个月只有 1G 流量，以前做项目没少被坑。ngrok 是一款免费开源的软件，稳定性极强，我曾做过测试，将 ngrok 客户端所在计算机的网络断开一阵子，再连接另一个网络，ngrok 很快会自动重连，几乎不受影响。

### 用于对处在内网环境中，无外网 IP 的计算机的远程连接。

ngrok 可以做 TCP 端口转发，对于 Linux 可以将其映射到 22 端口进行 SSH 连接。Windows 的远程桌面可以将其映射到 3389 端口来实现。同理，如果要做 MySQL 的远程连接，只需映射 3306 端口即可。

### 用作临时搭建网站并分配二级域名，可用作微信二次开发的本地调试。

微信公众平台二次开发时，服务器必须要能通过外网访问，而且必须是 80 接口。我们一般会在自己的电脑上写代码，但是由于电信运营商将 80 端口屏蔽了，甚至很多人通过无线路由器上网，根本就没有公网 ip。在这种情况下，我们每次都要上传代码到服务器对微信公众平台进行接口调试，十分的不方便。而 ngro 可以将内网映射到一个公网地址，这样就完美的解决了我们的问题。

ngrok 官方为我们免费提供了一个服务器，我们只需要下载 ngrok 客户端即可正常使用，但是后来官方的服务越来越慢，直到 ngrok 官网被完全屏蔽。现在我们已经无法使用 ngrok 官方的服务器了。所以，接下来我们自行搭建属于自己的 ngrok 服务器，为自己提供方便快捷又稳定的服务，一劳永逸。


## 过程梗概

这个配置的过程比较复杂，我的配置环境是：

* server 端是在阿里云，阿里云的系统版本是 CentOS 7 amd64
* client 端是对多的，有 ubuntu、windows、Raspberry Pi，基本上要覆盖到所有的平台 linux 、 windows 和 arm 。

所以配置的过程分为了以下步骤

1. server 环境配置
2. git 上下载
3. 生成证书
4. 生成服务器版本 server 上安装的 ngrokd
5. 生成各个平台上的 client 上安装的 ngrok
6. 调试

有一些要点要特别注意，我就是在这些东西上面绕了很久，配置的时候需要特别注意的地方：

1. 防火墙设置：服务器和客户端都要关闭对应端口的防火墙，否则不能链接会一直显示 connecting 。
2. 证书一定要设置正确： 证书会被编译到可执行文件中去，所以设置的时候需要正确设置地址，如果设置错误，最好是重新 git clone 一份代码来配置，make clean 在这里面似乎不能清除原有的配置。
3. 交叉编译的时候要注意平台是 32 位系统（386）、64 位系统（amd64）或者 arm ，设置错了不能运行

### server 配置

首先要安装 server 必须要的软件, 因为现在的阿里云自己的镜像带了 rhel 的源，所以可以直接偷懒用 yum 安装所有的东西。


```bash
yum install mercurial git gcc golang
```

因为以后还要保持运行，所以我加上了 supervisor 的设置。这样可以保持服务运行，重启电脑之后之类的都可以保持正常运行。supervisor 需要依赖 python，所以在安装的时候会要求安装 python 等相关的依赖。


```bash
yum install supervisor
```

### git 下载

我安装的时候用的 root, 所以比较偷懒的在目录 /root/ 下配置

```bash
cd /root

#官方地址，可能会报错，最近应该已经修复
git clone https://github.com/inconshreveable/ngrok.git

#修复地址，不会报错，感谢 tutumcloud
#git clone https://github.com/tutumcloud/ngrok.git
```

因为之前官方的源有一些问题，貌似是所使用的 Log4go 的组件在 github 上的地址失效了，所以编译的时候会出错。

现在似乎已经修正了问题，但是由于访问 github 不太顺畅，有可能下载的时候链接会中断，所以如果出错了就再运行一遍。

```
如果还有问题，修改源代码中库引用的错误方法：

由于 google code 的关闭，所以要把作者代码中的库引用地址修改一下
修改 src/ngrok/log/logger.go 文件
log "code.google.com/p/log4go" 改为 log "github.com/alecthomas/log4go"
```

### 证书生成

在使用官方服务的时候，我们使用的是官方的 SSL 证书，所以如果直接编译的话，默认的链接地址会到官方的 ngrok.com 去，所以我们需要自己生成证书。
** 证书非常关键，所有编译的文件都会携带证书文件在程序内部 **
所以证书生成的时候要保证所有的地方都是对应的。

首先我们要确定我们要使用的地址，这是受到域名解析服务的影响的。最简单的办法就是 ping 一下这个地址看是不是能够获得自己的服务器 ip 。这个服务一定是架设在当前的服务器上的，所以 dns 一定是 ping 了之后指向当前的服务器。

我使用的域名是 www.aiesst.com ， ip 地址就是我自己的服务器。最好先 ping 一下确定一下，有一些 dns 没有绑定，会导致不能访问；如果更换域名，又需要重新来一遍全过程，非常繁琐。所以一定要先确定好。

当域名为 www.aiesst.com 的时候，最终效果是远程机器的设置子域名为 abc 时，访问 `abc.www.aiesst.com: 端口 ` 就可以。端口是由运行的时候配置的，并不需要确定某一个端口，都是可以设置的。

大概的原理和效果讲清楚之后就开始写怎么做了：

```bash
cd /root/ngrok

#这里修改为自己的域名
NGROK_DOMAIN="www.aiesst.com"

openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=$NGROK_DOMAIN" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000
```

首先进入到 ngrok 的目录，然后设定域名地址，这里需要修改为自己的。openssl 就是生成 SSL 证书文件的过程，之后会在 ngrok 目录下生成 root*,device* 等六个文件。
然后需要拷贝到配置的目录中，在编译的时候会使用这些文件。

```bash
\cp rootCA.pem assets/client/tls/ngrokroot.crt -f
\cp device.crt assets/server/tls/snakeoil.crt  -f
\cp device.key assets/server/tls/snakeoil.key -f
```

这里用的 ` \cp ` 命令，因为系统默认 cp 是有别名的，实际执行的是 cp -i，需要每一个覆盖的时候都确认一遍，加上 `\cp` 就可强制覆盖了。

到这个地方，证书生成已经复制的准备工作就已经完成了。

### 生成服务器的 ngrokd

前面的配置工作已经完成，后面的就比较简单轻松了。由于是 go 语言写的，所以使用 golang 的 make 选项就好。

```bash
cd /root/ngrok
make release-server
```

等待下载和构建，如果下载失败什么的，估计是因为链接国外的服务器会断线的问题，重新运行一遍 `make release-server` 就好。

构建完成以后可以在 bin 目录下看到 `ngrokd` 这个文件，这个就是我们后面要开启的服务器端，现在先不要运行。

### 编译客户端的 ngrok

这里我们需要交叉编译，使用不同的编译选项来选择编译以后生成的平台

这里我主要是生成了 windows 、 arm 和 Linux 的版本。继续在原先的目录下:

```bash
GOOS=linux GOARCH=amd64 make release-client
GOOS=windows GOARCH=amd64 make release-client
GOOS=linux GOARCH=arm make release-client
```

不同平台使用不同的 GOOS 和 GOARCH，前面的编译选项就是指 go os , go 编译出来的操作系统 (windows,linux,darwin) ；go arch, 对应的构架 (386,amd64,arm)

* Linux 平台 32 位系统：GOOS=linux  GOARCH=386
* Linux 平台 64 位系统：GOOS=linux  GOARCH=amd64

* Windows 平台 32 位系统：GOOS=windows  GOARCH=386
* Windows 平台 64 位系统：GOOS=windows  GOARCH=amd64

* MAC 平台 32 位系统：GOOS=darwin  GOARCH=386
* MAC 平台 64 位系统：GOOS=darwin  GOARCH=amd64

* ARM 平台：GOOS=linux  GOARCH=arm 

通过前面的步骤，就会在bin目录里面生成所有的客户端文件，客户端平台是文件夹的名字，客户端放在对应的目录下，当前Linux平台客户端在bin目录下。然后我们就可以打个包，把所有文件下载到自己的本机了。

```bash
cd /root/ngrok
zip -r bin/
```

然后生成了Bin.zip的文件，通过scp之类的工具下载。

这个里面的注意

* 服务端叫 ngrokd
* 客户端叫 ngrok

所以以后要放到对应的平台，就只需要对应平台里面的ngrok文件就可以了。

### 配置服务器

在server端直接执行就可以了，其中NGROK_DOMAIN对应的就是一开始设置过的域名地址。

但是有一点要重点注意，就是httpAddr等端口的设置。

* httpAddr 是访问普通的http使用的端口号，用后面用 `子域名.www.aiesst.com:6060` 来访问服务
* httpsAddr 是访问的https使用的端口号,同上，只不过是需要https的服务访问才用这个端口
* tunnelAddr 是通道的端口号，这个端口是Ngrok用来通信的，所以这个端口在服务器上和客户端上设置必须要对应才可以正常的链接，默认不填写好像是4433

```bash
cd /root/ngrok
NGROK_DOMAIN="www.aiesst.com"
#http
bin/ngrokd -domain="$NGROK_DOMAIN" -httpAddr=":6060" -httpsAddr=":6061" -tunnelAddr=":6062" 
#https设置了tls
#bin/ngrokd -domain="www.aiesst.com" -httpAddr=":6060" -httpsAddr=":6061" -tunnelAddr=":6062" -tlsKey=./device.key -tlsCrt=./device.crt
```

#### 验证端口是否打开的问题

当打开了服务端程序之后，如果有疑问可以测试一下，因为有时候可能会遇到这个问题。我在一开始的时候并没有注意到防火墙问题，所以一直导致不能成功。除了这个问题之外，使用阿里云还有一些问题，就是我使用的系统是CentOS，默认是没有打开对应的端口的，如果遇到这个问题需要先验证一下端口是否打开。我使用了工具nc来测试。

使用的命令是:

```bash
nc -v -w 10 -z 127.0.0.1 6060-6062
```

如果显示的3个端口都有响应（都显示了 succeeded 就是正常），那么就可以继续开始后面的步骤。

#### 打开防火墙

如果是centOS的系统，防火墙应该是 `firewall-cmd` 来控制。对应的命令就是，其中端口号要写自己的：

```bash
firewall-cmd --permanent --zone=public --add-port=6060-6062/tcp  //永久
#firewall-cmd  --zone=public --add-port=6060-6062/tcp   //临时
```

如果是ubuntu之类的系统，防火墙一般是iptables来控制，对应的命令就是，也要修改自己的端口号才可以：

```bash
iptables -A INPUT -p tcp --dport 6060-6062 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 6060-6062 -j ACCEPT
```

打开了服务器的端口，然后就可以继续配置客户端。

### 配置windows客户端

这里的客户端就是指另外一台电脑，这一台电脑可以链接到服务器的电脑上，客户端常常是我们内网的电脑，这个就是ngrok实现的最大意义，可以和在公网的服务器通信之后，再直接访问内网电脑提供的服务，实现内网穿透。

这里我先使用了windows来测试，因为平时用的系统，有各种工具，相对会比较方便。

先找到一开始通过 scp 下载的 Bin.zip 文件，解压到任意目录。然后找到window版本的客户端 ngrok.exe。复制到任意目录，自己方便操作就好。我使用了 `D:\tmp\` 文件夹

先在这个目录下建立一个文件 `ngrok.cfg` ,然后写入内容:

* 地址和服务器设置的 NGROK_DOMAIN="www.aiesst.com" 中的地址保持一致
* 端口和服务器设置的通道端口 tunnelAddr=":6062" 保持一致

```
server_addr: "www.aiesst.com:6062"
trust_host_root_certs: false
```

然后 ctrl-r 打开运行，运行 cmd，输入：

* 日志： -log=ngrok_log.txt 是记录ngrok的日志，如果前期调试的时候加上这个参数，如果不能访问就可以查看到底是什么问题
* 子域名： -subdomain=abc 是定义访问的时候的子域名，现在访问 abc.www.aiesst.com:6060 就可以访问到这一台机器上80端口的服务

```bash
D:
cd tmp
ngrok.exe -log=ngrok_log.txt -subdomain=abc -config="ngrok.cfg" 80
```

稍微等待5秒钟，如果看到显示有 `tunnel status: online` ,那么后面的 forwarding 对应的内容就是访问本机的地址。

当然在windows上面，你需要再跑一个其他的服务在80端口可以看到访问的效果。

总之就是如果显示了 `tunnel status: online` 就是服务器和客户端是正常链接的，通过浏览器访问 abc.www.aiesst.com:6060 就可以链接到现在的内网主机上的服务。

### 配置 raspberry pi 客户端

配置流程和在windows下大同小异

先在需要的地方建立一个目录，然后建立一个和window下一样的ngrok.cfg的文件，内容也相同。

将 `Bin.zip` 里面的arm版本的ngrok解压出来，并上传到树莓派上对应的目录。

进入这个目录以后，通过 `chmod +x ngrok` 将ngrok设置成可执行文件。

执行命令 `./ngrok -subdomain=rbp -config="ngrok.cfg" 80`

这样就可以通过 `abc.www.aiesst.com:6060` 访问树莓派上的80端口对应的服务。（树莓派可以直接安装一个nginx, `apt-get install nginx` 然后默认80端口就可以显示nginx默认的页面）