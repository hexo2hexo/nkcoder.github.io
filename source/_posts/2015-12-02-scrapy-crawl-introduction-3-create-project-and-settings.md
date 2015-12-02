title: Scrapy爬虫入门（三）-创建项目
categories: Backend
date: 2015-12-02 19:50:05
tags: Scrapy
---

本节介绍Scrapy项目的搭建，以及配置选项。主要内容包括：

	- 创建一个新的Scrapy项目；
	- 介绍基本的设置；
	- 定义Item，即为数据建模；
	- 创建一个spider；

<!-- more -->

### 1. 创建一个新的Scrapy项目

首先创建一个项目目录，然后创建一个`virtualenv`环境，最后通过`scrapy startproject`命令创建一个新的项目：

	GuoDaniel:python nkcoder$ mkdir scrapy_start
	GuoDaniel:python nkcoder$ cd scrapy_start/
	GuoDaniel:scrapy_start nkcoder$ virtualenv start_env
	New python executable in /Users/nkcoder/Projects/python/scrapy_start/start_env/bin/python
	Installing setuptools, pip, wheel...sourdone.
	GuoDaniel:scrapy_start nkcoder$ source start_env/bin/activate
	(start_env) GuoDaniel:scrapy_start nkcoder$ pip install scrapy
	...
	(start_env) GuoDaniel:scrapy_start nkcoder$ scrapy startproject scrapy_start
	2015-12-02 07:39:12 [scrapy] INFO: Scrapy 1.0.3 started (bot: scrapybot)
	2015-12-02 07:39:12 [scrapy] INFO: Optional features available: ssl, http11
	2015-12-02 07:39:12 [scrapy] INFO: Overridden settings: {}
	New Scrapy project 'scrapy_start' created in:
	    /Users/nkcoder/Projects/python/scrapy_start/scrapy_start

	You can start your first spider with:
	    cd scrapy_start
	    scrapy genspider example example.com
	(start_env) GuoDaniel:scrapy_start nkcoder$ ls
	scrapy_start	start_env

`pip install scrapy`命令的输出太多，所以被省略了。项目创建完毕，结构为：

	scrapy_start/
	├── scrapy.cfg
	└── scrapy_start
	    ├── __init__.py
	    ├── items.py
	    ├── pipelines.py
	    ├── settings.py
	    └── spiders
	        └── __init__.py

其中，`scrapy_start`是项目根目录；`scrapy.cfg`是项目部署的配置文件，后面讲到部署时再介绍；子目录`scrapy_start`是项目的模块根目录；`items.py`定义item，即将爬取的数据映射到对象上；`pipelines.py`定义pipeline，处理item的，比如将爬取的数据保存到json文件，或者保存到MySQL数据库；`settings.py`是项目配置文件；`spiders`是spider的根目录，即我们定义的所有spider都应该放在这个目录下（或者其子目录下）。

### 2. 设置

Scrapy支持多种配置方式，它们的优先级不同，优先级从高到低依次为：

	1. 命令行选项
	2. spider级的设置
	3. project级的设置
	4. 基于命令的默认设置
	5. 全局默认设置

我们主要介绍第3种，即`project级的设置`，对应于`settings.py`配置文件：

	BOT_NAME：可以被认为是项目名，主要用于构造默认的`useragent`值以及日志输出，默认值为项目名；

	CONCURRENT_ITEMS：对于一个response，pipeline能并发处理的item的最大值，默认值为100；

	CONCURRENT_REQUESTS：scrapy的下载器能并发发起的request请求的最大值，默认值为16；

	CONCURRENT_REQUESTS_PER_DOMAIN：每一个域名的并发request阀值，默认值为8；

	CONCURRENT_REQUESTS_PER_IP：每一个IP的并发request阀值，该配置会覆盖`CONCURRENT_REQUESTS_PER_DOMAIN`，默认值为0，即未设置；

	DOWNLOAD_DELAY：下载延时，即下载同一个网站的连续网页之间的延时，是为了控制爬取的频率。该配置通常与`RANDOMIZE_DOWNLOAD_DELAY`一起使用，如果`RANDOMIZE_DOWNLOAD_DELAY`的值为`True`（默认为True），则scrapy会使用[0.5*DOWNLOAD_DELAY, 1.5*DOWNLOAD_DELAY]的随机延时。如果`CONCURRENT_REQUESTS_PER_IP`的值非0，则该延时是基于IP的，而不是基于domain的。默认值为0，表示禁用。

	DOWNLOAD_TIMEOUT：下载超时，单位是秒；可以将`download_timeout`作为`Request.meta`的key，对每一个request设置下载延时。

	ITEM_PIPELINES：所有定义的pipeline都需要在这里配置，类似于声明；

	SPIDER_MODULES：spider的根目录，默认为项目的spiders目录；可以配置多个，比如一个用于测试环境，一个用于生产环境；

	NEWSPIDER_MODULE：通过`genspider`命令新创建的spider的目录；

	COOKIES_ENABLED：是否启用cookies，默认为True；

这里选择的都是一些比较基础的配置，更多的配置含义请参考[settings](http://doc.scrapy.org/en/1.0/topics/settings.html)以及[Downloader Middleware](http://doc.scrapy.org/en/1.0/topics/downloader-middleware.html)。

### 3. 创建Item

Scrapy提供了Item类，用来保存从页面爬取的数据。有点类似于Java中的反序列化，只不过反序列化是将字节流转化为Java对象，而Item是一个通用的类，通过key/value的形式存取数据。

Item类中的所有字段通过`scrapy.Field()`来声明，声明的字段可以是任意类型，比如整数、字符串、列表等。

以我们在本系列教程中要爬取的网站为例：http://www.xiaochuncnjp.com/forum.php?mod=forumdisplay&fid=69&filter=typeid&typeid=62（因为这个网站需要登录才能爬取到大图，刚好方便后续登录验证章节的演示）。我们要爬取正文中的帖子列表，需要保存的数据主要有：标题、详情的url、详情、图片列表，在`items.py`中加入：

	class RentItem(scrapy.Item):
	    '''
	    item类
	    '''
	    title = scrapy.Field()          # 标题
	    rent_desc = scrapy.Field()      # 详情
	    url = scrapy.Field()            # 详情的url
	    pic_list = scrapy.Field()       # 图片列表

> 提示：如果python文件中包含中文注释，需要在文件的第一行指定utf-8编码：
	# -*- coding: utf-8 -*-

定义了item后，可以创建item对象，并赋值、取值，比如：

	rent_item = RentItem()
	rent_item['title'] = u'求张领收書'
	pic_list = ['images/1.jpg, images/2.jpg']
	rent_item['pic_List'] = pic_list

### 4. 创建一个spider

spider就是请求并解析页面，返回解析后的数据的执行过程。定义spider需要继承`scrapy.Spider`类，并重写其中的几个重要的属性和方法。

	name: spider的名称，全局唯一，必须的
	allowed_domains：域名白名单，即spider只会爬取该域名列表中的网页，可选的
	start_urls：爬虫的起始URL列表，必须的
	parse(response)：默认的回调方法，spider请求start_urls中的url后返回的response作为parse()的参数，返回值必须是Request()对象，或者Item对象，或者是一个dict

在scrapy_start/scrapy_start/spiders目录下新建文件`demo_spider.py`，打印一行日志：

	import scrapy
	from scrapy_start.items import RentItem

	class DemoSpider(scrapy.Spider):
	    name = 'xiaochuncnjp'
	    allowed_domains = ['xiaochuncnjp.com']
	    start_urls = [
	        'http://www.xiaochuncnjp.com/forum.php?mod=forumdisplay&fid=69&filter=typeid&typeid=62'
	        ]

	    def parse(self, response):
	        rent_item = RentItem()

	        print("in self， response is: {0}".format(response))

	        yield rent_item

运行刚定义的spider，可以看到打印的日志：

	(start_env) GuoDaniel:scrapy_start nkcoder$ scrapy list
	xiaochuncnjp
	(start_env) GuoDaniel:scrapy_start nkcoder$ scrapy crawl xiaochuncnjp
	...

### 参考：

- [Scrapy Tutorial](http://doc.scrapy.org/en/1.0/intro/tutorial.html)
- [Spiders](http://doc.scrapy.org/en/1.0/topics/spiders.html)
- [Items](http://doc.scrapy.org/en/1.0/topics/items.html)

