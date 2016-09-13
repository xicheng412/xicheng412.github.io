---
layout: post
title:  "我的kindle3出问题了以后的解决办法"
date:   2013-04-20 00:00:00
categories: kindle
tags: kindle
---

* content
{:toc}


## 现象：

kindle在安装了多看之后，再回到原系统升级卡死。

stage 1 of 4 卡住不能动






## 解决方法：

1. 越狱，安装jailbreak
2. 安装usbnetwork在kindle
3. 安装winscp和putty工具包到windows，以及使用方法
4. kindle上用;debugon ;dumpmessages ;debugoff得到自检文件找到ota_install的失败项目iptable
5. 用notepad++注意unix的回车是\n而win的回车是\r\n，修改为unix的回车（好压的md5不准确问题）,这是重点之一
6. winscp的黑色命令输入框输入mntroot rw修改写入权限
7. 上传iptable文件
8. 按顺序升级3.0.2 -> 3.1 -> 3.3 -> 3.4