---
layout: post
title:  "手机问题"
date:   2017-03-13 00:00:00
categories: android
tags: android bootloader unlock
---

* content
{:toc}

## 问题描述

这几天由于手机升级了7.1.2系统，同时又由于受到国内各个软件相互唤醒等问题折磨，手机卡顿，耗电巨大。准备把手机root掉。之前一直在使用的是没有root的手机。

手机型号nexus 5X。在我进入了系统以后，进行root，但是无论如何都不成功。现在新出了systemless的root方式，我就是按照这个方式想进行root。但是不能成功。后来仔细查看，发现应该可能是硬件问题。

网络上描述的问题是，可能是nand的内存读写坏了，这个硬件问题是软件不能解决的。

## 过程

### 下载platform-tools和usb驱动

首先要root需要进行的是输入fastboot等命令。

第一个坑就是，如何获得fastboot adb等工具。在寻找工具的过程重，可以到谷歌官方下载。

[platform-tools 谷歌官方下载地址](https://developer.android.com/studio/releases/platform-tools.html)

[usb 驱动谷歌官方下载地址](https://developer.android.com/studio/run/win-usb.html)

这个地址在访问的时候，最开始点进去以后是中文帮助页面，中文页面重的下载地址是错误的，会定位到自身页面上。所以一定需要在英文页面下载。

下载usb驱动以后直接安装就好。

platform-tools需要先解压到电脑目录里面，然后再增加电脑的path的环境变量，增加解压的目录。这样就可以直接使用adb和fastboot命令，而不需要在指定目录使用。

### 进行unlock

解锁的方式就是

1.先关机
2.关机以后按住音量减和电源键
3.等待进入bootloader的位置

然后就在电脑上输入

```bash
fastboot oem unlock
::或者
fastboot flashing unlock
```

通过这个操作，就可以解锁手机了。

但是注意！！！！这个操作会 **消除手机上的所有文件，并且将手机重置** ，包括用户数据。所以一定要先备份所有数据以后再进行解锁！！

### 遇到问题

然后我遇到的问题就是，当我进行了这个操作以后，只需要再次重启手机，手机的解锁状态又会从unlock变回locked。

这个状态无法解除，只能维持当次的状态，也就是说依然可以刷文件，但是不能重启手机。所以不能够进行root操作。

根据我后面搜索了很多资料，看了XDA上的很多文章之后，基本确定应该是NAND错误引起的。

## 附上其他各种操作

在进行测试的过程中，进行了各种操作，现在大概记录一下。

### fastboot命令

查看所有的已经连接的设备

> fastboot devices


下载了官方的包以后，可以先解压，然后通过以下命令手工刷系统的各个部分。

```bash
fastboot erase system
fastboot erase boot
fastboot erase cache
fastboot erase userdata
fastboot flash boot boot.img
fastboot flash system system.img
fastboot flash userdata data.img
fastboot reboot
```

擦除分区: fastboot erase boot或fastboot erase system等。

>fastboot erase {partition}
 

烧写指定分区：fastboot flash boot boot.img或fastboot flash system system.img等。

> fastboot flash {partition} {*.img} 

wipe掉data和cache分区的数据

> fastboot -w

重启手机

> fastboot reboot

### adb命令

* 查看设备

> adb devices

这个命令是查看当前连接的设备, 连接到计算机的android设备或者模拟器将会列出显示

* 安装软件

> adb install

adb install <apk文件路径> :这个命令将指定的apk文件安装到设备上

* 卸载软件

> adb uninstall <软件名>
> adb uninstall -k <软件名>

如果加 -k 参数,为卸载软件但是保留配置和缓存文件.

* 进入设备或模拟器的shell：

> adb shell

通过上面的命令，就可以进入设备或模拟器的shell环境中，在这个Linux Shell中，你可以执行各种Linux的命令，另外如果只想执行一条shell命令，可以采用以下的方式：

> adb shell [command]

如：adb shell dmesg会打印出内核的调试信息。

* 发布端口

可以设置任意的端口号，做为主机向模拟器或设备的请求端口。如：

> adb forward tcp:5555 tcp:8000

* 从电脑上发送文件到设备

> adb push <本地路径> <远程路径>

用push命令可以把本机电脑上的文件或者文件夹复制到设备(手机)

* 从设备上下载文件到电脑

> adb pull <远程路径> <本地路径>

用pull命令可以把设备(手机)上的文件或者文件夹复制到本机电脑

* 查看bug报告

> adb bugreport

* 记录无线通讯日志

一般来说，无线通讯的日志非常多，在运行时没必要去记录，但我们还是可以通过命令，设置记录：

> adb shell

> logcat -b radio

* 获取设备的ID和序列号

> adb get-product

> adb get-serialno

> adb shell

> sqlite3

### 刷twrp 过程

TWRP Recovery 是一个recovery分区的软件，根据之前的说明可以知道，使用命令：

> fastboot flash recovery twrp.img

其中twrp.img根据实际情况确定。下载了新的twrp之后，就直接使用这个命令写入。

### 刷 root 的过程

在写入了twrp之后，在superSU官网下载新的superSU二进制文件

将下载到的zip文件直接存入手机，在开机后通过bootloader进入recovery。

在twrp中选择superSU的二进制文件进行写入。

然后在系统中安装SuperSU的apk进行root相关操作。

### 刷其他系统

在有twrp以后，将其他系统的zip文件下载到手机，用twrp进行写入。

有时候还需要官方的镜像中的vendor.img文件。这个需要注意。


### 写入官方镜像并回归到初始状态

[官方镜像下载地址](https://developers.google.com/android/images?hl=en)

下载以后解压到任意目录，例如 *C:\bullhead\images\* 然后进行后续操作。

连接数据线，并重启手机按住电源和音量减两个键进入bootloader

```bash
fastboot flash bootloader C:\bullhead\images\bootloader-bullhead-bullhead-xx.xx.img
fastboot reboot-bootloader
fastboot flash radio C:\bullhead\images\radio-bullhead-bullhead-xx.xx.img
fastboot reboot-bootloader
fastboot flash boot C:\bullhead\images\boot.img
fastboot erase cache
fastboot flash cache C:\bullhead\images\cache.img
fastboot flash recovery C:\bullhead\images\recovery.img
fastboot flash system C:\bullhead\images\system.img
fastboot flash vendor C:\bullhead\images\vendor.img
```

如果你想恢复锁定状态，可以输入

> fastboot oem lock

重启手机

> fastboot reboot

### 更新系统

这个只需要解锁bootloader就好，在已有的官方系统基础上，用完整的包更新系统。操作也是和写入官方镜像类似，官方的更新镜像下载地址也相同。

```bash
fastboot flash bootloader C:\bullhead\images\bootloader-bullhead-bullhead-xx.xx.img
fastboot reboot-bootloader
fastboot flash radio C:\bullhead\images\radio-bullhead-bullhead-xx.xx.img
fastboot reboot-bootloader
fastboot flash boot C:\bullhead\images\boot.img
fastboot erase cache
fastboot flash cache C:\bullhead\images\cache.img
fastboot flash recovery C:\bullhead\images\recovery.img
fastboot flash system C:\bullhead\images\system.img
fastboot flash vendor C:\bullhead\images\vendor.img
```

这个过程可以选择是否写入recovery的文件，也可以不写，以便于保留之前的twrp。
