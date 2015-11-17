---
layout: post
title: Linux下命令行删除符号链接
date: 2014-07-16 10:34:25 +0800
comments: true
tags: linux
categories: Backend
---

## 1. 问题

Linux下，如何从命令行删除符号链接(软链接)？

## 2. 命令：

删除符号链接，使用`rm`命令或者`unlink`命令，后接文件名或目录名，如果是目录，注意目录后，不要加/。

语法：

	rm linkname/dirname
	unlink linkname/dirname

示例：

	nkcoder@ubuntu:~$ cd ~
	nkcoder@ubuntu:~$ ln -s /usr/local/jetty/start.ini start.ini.bak
	nkcoder@ubuntu:~$ ll start.ini.bak
	lrwxrwxrwx 1 nkcoder nkcoder 26 Jul 16 10:28 start.ini.bak -> /usr/local/jetty/start.ini
	nkcoder@ubuntu:~$ unlink start.ini.bak

	nkcoder@ubuntu:~$ ln -s /usr/local/jetty jetty.bak
	nkcoder@ubuntu:~$ ll jetty.bak
	lrwxrwxrwx 1 nkcoder nkcoder 16 Jul 16 10:28 jetty.bak -> /usr/local/jetty/
	nkcoder@ubuntu:~$ rm jetty.bak

#### 参考

- [Linux Delete Symbolic Link ( Softlink )](http://www.cyberciti.biz/faq/linux-remove-delete-symbolic-softlink-command/)
