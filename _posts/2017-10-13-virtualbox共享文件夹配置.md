---
layout: post
title: virtualbox共享文件夹配置
categories: 配置 
tags:  配置 virtualbox
---

* content
{:toc}

> 我的技术博客，主要是记载看过的书以及对写的好的博客文章的搬运整理，方便自己他人查看，也方便别人指出我文章中的错误，达到一起学习的目的。
> 技术永无止境

本文主要是对各个软件配置的归纳整理



#virtualbox 共享文件夹

1. Devices -> Install Guest Additions...(it's like insert a cd to cdrom drive)
2. Open a terminal to login the guest machine
3. Type below

```sh
sudo mkdir --p /media/cdrom
sudo mount -t auto /dev/cdrom /media/cdrom/
cd /media/cdrom/
sudo sh VBoxLinuxAdditions.run
```

You now can do full screen, shared folder, clipboard sharing, etc

> 其中安装过程，我们需要安装编译器，否则插件无法安装
> `$ sudo yum install -y kernel-devel gcc`
> 另外，需要更新内核，否则会导致kernel与kernel-devel版本不一致的问题
> `yum update kernel`


[终端挂载ios](https://askubuntu.com/questions/80341/unable-to-mount-virtualbox-guest-additions-as-a-guest-win7-host)
[虚拟机安装VBoxAdditions增强功能](http://www.cnblogs.com/xqzt/p/4937954.html)
[VirtualBox虚拟机CentOS安装增强功能Guest Additions](http://www.jianshu.com/p/7c556c783bb2)


