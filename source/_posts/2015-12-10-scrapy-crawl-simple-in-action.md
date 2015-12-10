title: Scrapy爬虫入门实例
categories: Backend
date: 2015-12-10 16:20:48
tags: Scrapy
---

在搭建好了Scrapy的开发环境后（如果配置过程中遇到问题，请参考上一篇文章[搭建Scrapy爬虫的开发环境](http://nkcoder.github.io/2015/11/17/Scrapy-crawl-intro-install-and-config/)，或者在博客里留言），我们开始演示爬取实例。

我们试图爬取[论坛-东京版](http://www.xiaochuncnjp.com/forum-19-1.html)的主题贴。该网站需要登录后才能查看帖子附带的大图，适合演示登录过程。

<!-- more -->

### 1. 定义item

我们需要保存标题、帖子详情、帖子详情的url、图片列表，所以定义item如下：

	class RentItem(scrapy.Item):
	    '''
	    item类
	    '''
	    title = scrapy.Field()          # 标题
	    rent_desc = scrapy.Field()      # 详情
	    url = scrapy.Field()            # 详情的url
	    pic_list = scrapy.Field()       # 图片列表

### 2. 使用FormRequest模拟登录

首先我们需要分析页面，找到登录的form，以及需要提交的数据（用Fiddler或Firebug分析请求即可），然后使用Scrapy提供`FormRequest.from_response()`模拟页面的登录过程，主要代码如下：

	if 'id="lsform"' in response.body:
        logging.info('in parse, need to login, url: {0}'.format(response.url))
        form_data = {'handlekey': 'ls', 'quickforward': 'yes', 'username': 'user123', 'password': 'passwd123'}
        request = FormRequest.from_response(response=response,
                                            formxpath='//form[contains(@id, "lsform")]',
                                            formdata=form_data,
                                            callback=self.parse_list)
    else:
        logging.info('in parse, NOT need to login, url: {0}'.format(response.url))
        request = Request(response.url, callback=self.parse_list)

如果请求的页面需要登录，则通过xpath定位到对应的form，将登录需要的数据作为参数，提交登录，在callback对应的回调方法里，处理登录成功后的爬取逻辑。

### 3. 使用XPath提取页面数据

Scrapy使用XPath或CSS表达式分析页面结构，由基于lxml的Selector提取数据。关于XPath，请参考[zvon-XPath 1.0 Tutorial](http://zvon.org/comp/r/tut-XPath_1.html)，示例丰富且易懂，看完这个入门教程，常见的爬取需求基本都能满足。我这里简单解释一下几个重要的点：

- /表示绝对路径，即匹配从根节点开始，./表示当前路径，//表示匹配任意开始节点；

- *是通配符，可以匹配任意节点；

- 在一个节点上使用[]，如果是数字n表示匹配第n个element，如果是@表示匹配属性，还可以使用函数，比如常用的contains()表示包含，starts-with()表示字符串起始匹配等。

- 在取节点的值时，text()只是取该节点下的值，而不会取该节点的子节点的值，而.则会取包括子节点在内的所有值，比如：


	<div>Welcome to <strong>Chengdu</strong></div>

	sel.xpath('div/text()') 	// Welcome to 
	sel.xpath('div').xpath('string(.)')		// Welcome to Chengdu

### 4. 不同的spider使用不同的pipeline

我们可能有很多的spider，不同的spider爬取的数据的结构不一样，对应的存储格式也不尽相同，因此我们会定义多个pipeline，让不同的spider使用不同的pipeline。

首先我们需要定义一个decorator，表示如果spider的`pipeline`属性中包含了添加该注解的pipeline，则执行该pipeline，否则跳过该pipeline：

	def check_spider_pipeline(process_item_method):

	    @functools.wraps(process_item_method)
	    def wrapper(self, item, spider):

	        # message template for debugging
	        msg = '%%s %s pipeline step' % (self.__class__.__name__,)

	        # if class is in the spider's pipeline, then use the
	        # process_item method normally.
	        if self.__class__ in spider.pipeline:
	            logging.info(msg % 'executing')
	            return process_item_method(self, item, spider)

	        # otherwise, just return the untouched item (skip this step in
	        # the pipeline)
	        else:
	            logging.info(msg % 'skipping')
	            return item

	    return wrapper

然后，我们还需要在所有pipeline类的回调方法`process_item()`上添加该decrator注解：

   @check_spider_pipeline
   def process_item(self, item, spider):

最后，在spider类中添加一个数组属性`pipeline`，里面是所有与该spider对应的pipeline，比如：

	# 应该交给哪个pipeline去处理
    pipeline = set([
        pipelines.RentMySQLPipeline,
    ])

### 5. 将爬取的数据保存到mysql

数据存储的逻辑在pipeline中实现，可以使用`twisted adbapi`以线程池的方式与数据库交互。首先从setttings中加载mysql配置：

	@classmethod
    def from_settings(cls, settings):
        '''加载mysql配置'''
        dbargs = dict(
            host=settings['MYSQL_HOST'],
            db=settings['MYSQL_DBNAME'],
            user=settings['MYSQL_USER'],
            passwd=settings['MYSQL_PASSWD'],
            charset='utf8',
            use_unicode=True
        )

        dbpool = adbapi.ConnectionPool('MySQLdb', **dbargs)
        return cls(dbpool)

然后在回调方法`process_item`中使用dbpool保存数据到mysql：

	@check_spider_pipeline
    def process_item(self, item, spider):
        '''
        pipeline的回调.
        注解用于pipeline与spider之间的对应，只有spider注册了该pipeline，pipeline才会被执行
        '''

        # run db query in the thread pool，在独立的线程中执行
        deferred = self.dbpool.runInteraction(self._do_upsert, item, spider)
        deferred.addErrback(self._handle_error, item, spider)
        # 当_do_upsert方法执行完毕，执行以下回调
        deferred.addCallback(self._get_id_by_guid)

        time.sleep(10)
        return deferred

### 6. 将图片保存到七牛云

查看七牛的python接口即可，这里要说明的是，上传图片的时候，不要使用BucketManager的`bucket.fetch()`接口，因为经常上传失败，建议使用`put_data()`接口，比如：

	def upload(self, file_data, key):
        try:
            token = self.auth.upload_token(QINIU_DEFAULT_BUCKET)
            ret, info = put_data(token, key, file_data)
        except Exception as e:
            logging.error('upload error, key: {0}, exception: {1}'.format(key, e))

        if info.status_code == 200:
            logging.info('upload data to qiniu ok, key: {0}'.format(key))
            return True
        else:
            logging.error('upload data to qiniu error, key: {0}'.format(key))
            return False

ok，这篇入门实例的重点就这么多，项目的源码在github上，[戳这里]。

### 7. 项目部署

部署可以使用[scrapyd](http://scrapyd.readthedocs.org/en/latest/api.html)和[scrapyd-client](https://github.com/scrapy/scrapyd-client/tree/master/scrapyd-client)。
首先安装：

	$ pip install scrapyd
	$ pip install scrapyd-client

启动scrapyd:

	$ sudo scrapyd &

修改部署的配置文件scrapy.cfg:

	[settings]
	default = timediff_crawler.settings

	[deploy:dev]
	url = http://localhost:6800/
	project = scrapy_start

其中dev表示target，scrapy_start表示project，部署即可：

	$ scrapyd-deploy dev -p scrapy_start

### 参考

- [Scrapy 1.0 documentation](http://doc.scrapy.org/en/latest/index.html)
- [XPath 1.0 Tutorial](http://zvon.org/comp/r/tut-XPath_1.html#intro)
- [How can I use different pipelines for different spiders in a single Scrapy project](http://stackoverflow.com/questions/8372703/how-can-i-use-different-pipelines-for-different-spiders-in-a-single-scrapy-proje)
- 

