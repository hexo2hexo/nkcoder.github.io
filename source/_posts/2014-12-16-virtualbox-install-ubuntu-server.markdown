---
layout: post
title: VirtualBox安装Ubuntu Server
date: 2014-12-16 20:47:40 +0800
comments: true
tags: virtualbox
categories: Backend
---

有段时间没更新博客了，今天写一篇随手笔记吧。

公司给配置的电脑是Thinkpad， 装的是Windows，但手边没有个Linux环境，总感觉缺点啥，于是决定在虚拟机上装个Ubuntu。

<!--more-->

版本情况：

+ Windows：64位Windows 7
+ VirtualBox：VirtualBox-4.3.20-96997-Win
+ Ubuntu：ubuntu-14.04.1-server-amd64

在VirtualBox中新建虚拟机时，发现ubuntu的版本只有`Ubuntu(32bit)`一个选项，有点纳闷，不管了，先继续安装。选择完语言，开始安装后就报错了：

	This kernel requires an X86-64 CPU,but only detected an i686 CPU

Goolge一下，很常见的问题了，需要在BIOS中启用`Intel VT-x`的虚拟化配置，该配置默认是禁用的。

然后，重新启动虚拟机，还是同样的错误，原来还需要在settings中将version修改为`Ubuntu(64 bit)`，再重新启动就OK了。

以前在VirtualBox中安装CentOS时，也遇到相同的问题，只是过了一段时间后就忘了，好记性不如烂笔头。

### 参考：

+ [This kernel requires an x86-64 CPU](https://hereirestinremorse.wordpress.com/virtualbox/this-kernel-requires-an-x86-64-cpu-but-only-detected-an-i686-cpu-unable-to-boot-please-use-a-kernel-appropriate-for-your-cpu/)
