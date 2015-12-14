title: 搭建Scrapy爬虫的开发环境
categories: Backend
date: 2015-11-17 21:33:22
tags: Scrapy
---

这一章主要介绍Scrapy的安装、安装过程中可能遇到的问题以及解决方式。由于我在Mac和Ubuntu环境下都尝试过，所以会将两个平台上遇到的问题都记下来以供参考。

在安装Scrapy之前，首先需要安装以下组件：

- python 2.7
- pip
- lxml
- openssl

接下来分别介绍。

<!-- more -->

## 1. 安装python 2.7

目前Scrapy 1.x仅支持python2.x（官方说以后会支持python 3.x，但目前不支持）。一般系统都预装了python，可以通过`-V`命令查看版本：

	GuoDaniel:~ nkcoder$ python -V
	Python 2.7.10

当然你可以直接使用系统的python，但更好地做法是通过`virtualenv`虚拟化一个python环境，与系统的python隔离，避免依赖冲突等问题。

安装`virtualenv`：

	$ [sudo] pip install virtualenv

创建一个python虚拟环境：

	GuoDaniel:start_scrapy nkcoder$ virtualenv startenv
	New python executable in /Users/nkcoder/Projects/python/start_scrapy/startenv/bin/python
	Installing setuptools, pip, wheel...done.
	GuoDaniel:start_scrapy nkcoder$ source startenv/bin/activate
	(startenv) GuoDaniel:start_scrapy nkcoder$ ls
	startenv
	(startenv) GuoDaniel:start_scrapy nkcoder$ which pip
	/Users/nkcoder/Projects/python/start_scrapy/startenv/bin/pip
	(startenv) GuoDaniel:start_scrapy nkcoder$ python -V
	Python 2.7.10

virtualenv也可以指定python解释器，默认使用PATH定义的python解释器。比如创建一个python 3的虚拟环境：

	GuoDaniel:start_scrapy nkcoder$ virtualenv -p /Library/Frameworks/Python.framework/Versions/3.5/bin/python3 python3env
	Running virtualenv with interpreter /Library/Frameworks/Python.framework/Versions/3.5/bin/python3
	Using base prefix '/Library/Frameworks/Python.framework/Versions/3.5'
	New python executable in /Users/nkcoder/Projects/python/start_scrapy/python3env/bin/python3
	Also creating executable in /Users/nkcoder/Projects/python/start_scrapy/python3env/bin/python
	Installing setuptools, pip, wheel...done.
	GuoDaniel:start_scrapy nkcoder$ source python3env/bin/activate
	(python3env) GuoDaniel:start_scrapy nkcoder$ which python
	/Users/nkcoder/Projects/python/start_scrapy/python3env/bin/python
	(python3env) GuoDaniel:start_scrapy nkcoder$ python -V
	Python 3.5.0

- 参考：[VirtualEnv Installation](https://virtualenv.readthedocs.org/en/latest/installation.html)

## 2. 安装pip

通过virtualenv安装的python，默认已经安装了对应版本的pip，查看pip版本：

	GuoDaniel:start_scrapy nkcoder$ source startenv/bin/activate
	(startenv) GuoDaniel:start_scrapy nkcoder$ which pip
	/Users/nkcoder/Projects/python/start_scrapy/startenv/bin/pip
	(startenv) GuoDaniel:start_scrapy nkcoder$ pip -V
	pip 7.1.2 from /Users/nkcoder/Projects/python/start_scrapy/startenv/lib/python2.7/site-packages (python 2.7)

如果需要自己安装pip，也很简单，首先[下载get-pip.py脚本](https://bootstrap.pypa.io/get-pip.py)，然后安装：

	$ python get-pip.py

- 参考：[Pip Installation](http://pip.readthedocs.org/en/stable/installing/)

## 3. 安装lxml

通过`pip`安装lxml：

	$ sudo pip install lxml

**在Mac环境下安装lxml可能会遇到以下错误**：

	In file included from src/lxml/lxml.etree.c:314:
	/private/tmp/pip_build_root/lxml/src/lxml/includes/etree_defs.h:9:10: fatal error: 'libxml/xmlversion.h' file not found
	#include "libxml/xmlversion.h"
				 ^
	1 error generated.
	error: command 'cc' failed with exit status 1

安装或更新xcode-select即可：

	$ xcode-select --install

- 参考：[Cannot install Lxml on Mac os x 10.9](http://stackoverflow.com/questions/19548011/cannot-install-lxml-on-mac-os-x-10-9)

**在Ubuntu环境下可能会遇到以下错误**：

	libxml/xmlversion.h: No such file or directory compilation terminated.

需要安装相应的dev包：

	$ sudo apt-get install libxml2-dev libxslt1-dev python-dev

- 参考：[how-to-install-lxml-on-ubuntu](http://stackoverflow.com/questions/6504810/how-to-install-lxml-on-ubuntu)

## 4. 安装openssl

系统一般都预装有openssl：

	$ openssl version
	OpenSSL 0.9.8zg 14 July 2015

## 5. 安装Scrapy

通过pip安装Scrapy：

	$ sudo pip install scrapy

**在Mac环境下，如果系统版本是`OS X EI Capitan`，并且使用的是系统的python，而不是virtualenv的虚拟环境，则可能会遇到如下问题**：

	OSError: [Errno 1] Operation not permitted: '/tmp/pip-nIfswi-uninstall/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/six-1.4.1-py2.7.egg-info'

原因是Scrapy的依赖库Six与系统搞得依赖库Six发生了冲突，一种解决方式是将系统的`System Integrity Protection`临时禁用，更好地解决方式当然是使用virtualenv创建一个隔离的python环境。

- 参考：[“OSError: [Errno 1] Operation not permitted”](http://stackoverflow.com/questions/31900008/oserror-errno-1-operation-not-permitted-when-installing-scrapy-in-osx-10-11)

**在ubuntu环境下，可能会遇到这个问题**：

	fatal error: openssl/aes.h: No such file or directory

安装`libssl-dev`即可：

	$ sudo apt-get install libssl-dev

- 参考：[How to fix “fatal error: openssl/aes.h: No such file or directory”](http://ask.xmodulo.com/fix-fatal-error-openssl.html)
