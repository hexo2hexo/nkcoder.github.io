---
layout: post
title: 使用ElasticSearch+LogStash+Kibana+Redis搭建日志管理服务
date: 2014-10-31 09:37:56 +0800
comments: true
tags:
  - elasticsearch
  - kibana
  - logstash
categories: Backend

---

### 1. 日志平台的结构示意图

![ELKR-log-platform](/images/post/ELKR-log-platform.jpg)

说明：

- 多个独立的agent(Shipper)负责收集不同来源的数据，一个中心agent(Indexer)负责汇总和分析数据，在中心agent前的Broker(使用redis实现)作为缓冲区，中心agent后的ElasticSearch用于存储和搜索数据，前端的Kibana提供丰富的图表展示。
- Shipper表示日志收集，使用LogStash收集各种来源的日志数据，可以是系统日志、文件、redis、mq等等；
- Broker作为远程agent与中心agent之间的缓冲区，使用redis实现，一是可以提高系统的性能，二是可以提高系统的可靠性，当中心agent提取数据失败时，数据保存在redis中，而不至于丢失；
- 中心agent也是LogStash，从Broker中提取数据，可以执行相关的分析和处理(Filter)；
- ElasticSearch用于存储最终的数据，并提供搜索功能；
- Kibana提供一个简单、丰富的web界面，数据来自于ElasticSearch，支持各种查询、统计和展示；

<!-- more -->

### 2. 搭建部署

环境：

- 本机(20.8.40.49)上部署：redis, 中心agent(LogStash), ElasticSearch以及Kibana
- 远程测试机(20.20.79.75)上部署：独立agent(LogStash)
- Redis版本：[3.0.0-rc1](https://github.com/antirez/redis/archive/3.0.0-rc1.tar.gz)
- LogStash版本；[logstash-1.4.2](https://download.elasticsearch.org/logstash/logstash/logstash-1.4.2.tar.gz)
- ElasticSearch版本：[elasticsearch-1.3.4](https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.3.4.tar.gz)
- Kibana版本：[kibana-3.1.1](https://download.elasticsearch.org/kibana/kibana/kibana-3.1.1.tar.gz)

#### 2.1 部署redis

部署一个redis单机实例:

	$ wget https://github.com/antirez/redis/archive/3.0.0-rc1.tar.gz
	$ tar zxvf 3.0.0-rc1.tar.gz -C /usr/local

redis.conf配置文件为：

	include ../redis.conf
	daemonize yes
	pidfile /var/run/redis_6379.pid
	port 6379
	logfile /opt/logs/redis/6379.log
	appendonly yes

启动：

	$ redis.server redis.conf

> ip为10.7.40.40， 端口为6379

#### 2.2 部署中心LogStash

下载并解压：

	$ wget https://download.elasticsearch.org/logstash/logstash/logstash-1.4.2.tar.gz
	$ tar zxvf logstash-1.4.2.tar.gz -C /usr/local/
	$ cd /usr/local/logstash-1.4.2
	$ mkdir conf logs

配置文件conf/central.conf:

	input {
		redis {
			host => "127.0.0.1"
			port => 6379
			type => "redis-input"
			data_type => "list"
			key => "key_count"
		}   
	}

	output {
		stdout {}
		elasticsearch {
			cluster => "elasticsearch"
			codec => "json"
			protocol => "http"
		}   
	}

启动：

	$ bin/logstash agent --verbose --config conf/central.conf --log logs/stdout.log

> 配置文件表示输入来自于redis，使用redis的list类型存储数据，key为"key_count"；输出到elasticsearch，cluster的名称为"elasticsearch";

#### 2.3 部署ElasticSearch

下载并解压：

	$ wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.3.4.tar.gz
	$ tar zxvf elasticsearch-1.3.4.tar.gz -C /usr/local

elasticsearch使用默认配置即可，默认的cluster name为：elasticsearch；

启动：

	$ bin/elasticsearch -d

#### 2.4 部署远程LogStash

与部署中心LogStash的步骤是类似的，只是配置文件不一样，使用新的配置文件启动即可；

配置文件conf/shipper.conf的内容为：

	input {
		file {
			type => "type_count"
			path => ["/data/logs/count/stdout.log", "/data/logs/count/stderr.log"]
			exclude => ["*.gz", "access.log"]
		}   
	}

	output {
		stdout {}
		redis {
			host => "20.8.40.49"
			port => 6379
			data_type => "list"
			key => "key_count"
		}   
	}

> 配置文件表示输入来自于目录/data/logs/count/下的stdout.log和stderr.log两个文件，且排除该目录下所有.gz文件和access.log；(这里因为path没有使用通配符，所以exclude是没有效果的)；输出表示将监听到的event发送到redis服务器，使用redis的list保存，key为"key_count"，这里的`data_type`属性和`key`属性应该与中心agent的配置一致；

#### 2.5 部署Kibana

下载并安装：

	$ wget https://download.elasticsearch.org/kibana/kibana/kibana-3.1.1.tar.gz
	$ tar zxvf kibana-3.1.1.tar.gz

修改配置文件config.js，仅需要配置elasticsearch的地址即可：

	elasticsearch: "http://20.8.40.49:9200"

将目录kibana-3.1.1拷贝到jetty的webapp目录下，并启动jetty：

	$ mv kibana-3.1.1 /usr/local/jetty-distribution-9.2.1.v20140609/webapps/
	$ bin/jetty start

浏览器访问：

	http://20.8.40.49:8080/kibana-3.1.1/

### 3. 简单测试

打开LogStash的远程agent和中心agent的日志：

	$ tail -f logs/stdout.log

远程agent的数据是以`rpush`操作将event推送到redis的list中，中心agent通过`blpop`命令从redis的list中提取数据，因此，测试时由于数据量小，通过命令`llen key_count`的返回结果很可能为空，因此为了观察redis中数据流的变化，可以使用`monitor`命令：

	$ redis-cli -p 6379 monitor

现在，我们向/data/logs/count目录下的stdout.log和stderr.log各发送一条数据：

	$ echo "stdout: just a test message" >> stdout.log
	$ echo "stderr: just a test message" >> stderr.log

远程agent和中心agent都会收到event消息，如远程agent的日志为：

	{:timestamp=>"2014-10-31T09:30:40.323000+0800", :message=>"Received line", :path=>"/data/logs/count/stdout.log", :text=>"stdout: just a test message", :level=>:debug, :file=>"logstash/inputs/file.rb", :line=>"134"}
	{:timestamp=>"2014-10-31T09:30:40.325000+0800", :message=>"writing sincedb (delta since last write = 52)", :level=>:debug, :file=>"filewatch/tail.rb", :line=>"177"}
	......
	{:timestamp=>"2014-10-31T09:30:49.350000+0800", :message=>"Received line", :path=>"/data/logs/count/stderr.log", :text=>"stderr: just a test message", :level=>:debug, :file=>"logstash/inputs/file.rb", :line=>"134"}
	{:timestamp=>"2014-10-31T09:30:49.352000+0800", :message=>"output received", :event=>{"message"=>"stderr: just a test message", "@version"=>"1", "@timestamp"=>"2014-10-31T01:30:49.350Z", "type"=>"type_count", "host"=>"dn1", "path"=>"/data/logs/count/stderr.log"}, :level=>:debug, :file=>"(eval)", :line=>"19"}

我们可以观察到redis的输出：

	1414714174.936642 [0 20.20.79.75:54010] "rpush" "key_count" "{\"message\":\"stdout: just a test message\",\"@version\":\"1\",\"@timestamp\":\"2014-10-31T00:10:04.530Z\",\"type\":\"type_count\",\"host\":\"dn1\",\"path\":\"/data/logs/count/stdout.log\"}"
	1414714174.939517 [0 127.0.0.1:56094] "blpop" "key_count" "0"
	1414714198.991452 [0 20.20.79.75:54010] "rpush" "key_count" "{\"message\":\"stderr: just a test message\",\"@version\":\"1\",\"@timestamp\":\"2014-10-31T00:10:28.586Z\",\"type\":\"type_count\",\"host\":\"dn1\",\"path\":\"/data/logs/count/stderr.log\"}"
	1414714198.993590 [0 127.0.0.1:56094] "blpop" "key_count" "0"

从elasticsearch中执行如下的简单查询：

	$ curl 'localhost:9200/_search?q=type:type_count&pretty'
	{
	  "took" : 3,
	  "timed_out" : false,
	  "_shards" : {
		"total" : 6,
		"successful" : 6,
		"failed" : 0
	  },
	  "hits" : {
		"total" : 2,
		"max_score" : 0.5945348,
		"hits" : [ {
		  "_index" : "logstash-2014.10.31",
		  "_type" : "type_count",
		  "_id" : "w87bRn8MToaYm_kfnygGGw",
		  "_score" : 0.5945348,
		  "_source":{"message":"stdout: just a test message","@version":"1","@timestamp":"2014-10-31T08:10:04.530+08:00","type":"type_count","host":"dn1","path":"/data/logs/count/stdout.log"}
		}, {
		  "_index" : "logstash-2014.10.31",
		  "_type" : "type_count",
		  "_id" : "wwmA2BD6SAGeNsuYz5ax-Q",
		  "_score" : 0.5945348,
		  "_source":{"message":"stderr: just a test message","@version":"1","@timestamp":"2014-10-31T08:10:28.586+08:00","type":"type_count","host":"dn1","path":"/data/logs/count/stderr.log"}
		} ]
	  }
	}

再切换到Kibana的web界面：http://20.8.40.49:8080/kibana-3.1.1

![kibana-count-log](/images/post/kibana-count-log.png)

### 4. 后续工作
-----------------------------------------------------------------

+ 使用LogStash的Filter对日志数据进行过滤和分析；
+ 使用Redis的Cluster模式替换单机模式；
+ 在elasticsearch中对数据进行高级的分析和查询；
+ 熟悉Kibana的展示组件以及查询语法；


#### 参考：

+ [logstash-doc-1.4.2](http://logstash.net/docs/1.4.2/)
+ [elasticsearch](http://www.elasticsearch.org/overview/)
+ [redis-blpop](http://redis.io/commands/blpop)
