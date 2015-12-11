---
layout: post
title: ElasticSearch+LogStash+Kibana+Redis日志服务的高可用方案
date: 2014-11-06 21:35:33 +0800
comments: true
tags:
  - elasticsearch
  - kibana
  - logstash
categories: Backend
---

### 1. 高可用方案的架构

在上一篇文章[使用ElasticSearch+LogStash+Kibana+Redis搭建日志管理服务](http://nkcoder.github.io/blog/20141031/elkr-log-platform-deploy/)中介绍了日志服务的整体框架以及各组件的搭建部署，本篇文章主要讨论一下日志服务框架的高可用方案，主要从以下三个方面考虑：

<!-- more -->

+ 作为broker的Redis，可以使用redis cluster或者主备结构代替单实例，提高broker组件的可用性；
+ 作为indexer的LogStash，可以部署多个LogStash实例，协作处理日志信息，提高indexer组件的可用性；
+ 作为search&storage的ElasticSearch，采用ElasticSearch Cluster，提高search&storage组件的性能和可用性；

<!-- more -->

日志服务的高可用方案的示意图为：

![ELKR-log-platform-ha](/images/post/ELKR-log-platform-ha.jpg)

下面分别予以介绍。

### 2. Redis Cluster和主备结构

#### 2.1 Redis Cluster

首先需要部署一个redis cluster，为了方便，我在本机上部署了一个三主三从的cluster，端口分别为：7000, 7001, 7002, 7003, 7004, 7005，以端口7000为例，配置文件为：

	include ../redis.conf
	daemonize yes
	pidfile /var/run/redis_7000.pid
	port 7000
	logfile /opt/logs/redis/7000.log
	appendonly yes
	cluster-enabled yes
	cluster-config-file node-7000.conf

对于redis来说，远程Logstash和中心LogStash都是redis cluster的client，因此只需要与cluster中的任何一个节点连接即可；远程LogStash和中心LogStash的redis配置部分为：

shipper.conf:

	output {
		redis {
			host => "20.8.40.49:7000"
			data_type => "list"
			key => "key_count"
		}   
	}

central.conf:

	input {
		redis {
			host => "20.8.40.49"
			port => 7000
			type => "redis-cluster-input"
			data_type => "list"
			key => "key_count"
		}
	}

使用redis cluster的优劣分析：

+ 可以提高broker组件的可用性：当每一个master节点都有slave节点的时候，任何一个节点挂掉，都不影响集群的正常工作；如果启用`cluster-require-full-coverage no`，则有效节点构成的子集群仍然可用。
+ 当作为broker组件的redis成为瓶颈的时候，redis cluster提供良好的扩展性。但是redis cluster有一个比较头疼的问题，就是在伸缩(增删节点)时，需要手动做sharding，官方提供了`redis-trib.rb`工具，我自己实现了一个java版本，可以作为参考[redis-toolkit](https://github.com/nkcoder/redis-toolkit)。
+ 当前的redis cluster还是RC1版，稳定版还需要等待一段时间。

#### 2.2 redis主备结构

注意，主备不是主从，备用的redis实例只是作为冗余节点，当主节点挂掉时，备用的节点会顶上，任何时刻仅有一个节点在提供服务。无论是远程LogStash还是中心LogStash，都需要明确配置所有主备redis节点的信息，LogStash会轮询节点列表，选择一个可用的节点。比如，配置两个redis实例，6379作为主，6380作为从，则远程LogStash和中心LogStash的配置分别为：

shipper.conf:

	output {
		redis {
			host => ["20.8.40.49:6379", "20.8.40.49:6380"]
			data_type => "list"
			key => "key_count"
		}   
	}

central.conf:

	input {
		redis {
			host => "20.8.40.49"
			port => 6379
			type => "redis-cluster-input"
			data_type => "list"
			key => "key_count"
		}

		redis {
			host => "20.8.40.49"
			port => 6380
			type => "redis-cluster-input"
			data_type => "list"
			key => "key_count"
		}
	}

redis主备结构的优劣分析：

+ 可以提高broker组件的可用性：只要主备节点中有一个节点可用，broker组件服务就是可用的；
+ 主备结构无法解决redis成为瓶颈的情况；


### 3. 部署多个中心LogStash

当日志信息的量很大时，作为indexer的LogStash很可能成为瓶颈，此时，可以部署多个中心LogStash，它们之间的关系是对等的，共同从broker处提取消息，中心LogStash节点之间是相互独立的。每一个中心LogStash节点的配置是完全一样的，比如当broker是redis cluster时，中心LogStash的配置为：

central.conf:

	input {
		redis {
			host => "20.8.40.49"
			port => 7000
			type => "redis-cluster-input"
			data_type => "list"
			key => "key_count"
		}
	}

部署多个中心LogStash的优劣分析：

+ 可以提高indexer组件的可用性：多个中心LogStash节点之间是相互独立的，任何一个节点的失效不会影响其它节点，更不会影响整个indexer组件；当broker是redis时，各个中心LogStash都是redis的client，它们都是执行`BLPOP`命令从redis中提取消息，而redis的任何单个命令都是原子的，因此多中心LogStash不仅可以提高indexer组件的可用性，也可以提高indexer组件的处理能力和效率；
+ 多中心LogStash节点的部署和维护成本，可以借助配置管理工具如[Puppet](http://puppetlabs.com/)、[SaltStack](http://www.saltstack.com/)等


### 4. ElasticSearch Cluster

ElasticSearch原生支持Cluster模式，节点之间通过单播或多播进行通信；ElasticSearch Cluster能自动检测节点的增加、失效和恢复，并重新组织索引。

比如，我们启动两个ElasticSearch实例构成集群，使用默认配置即可，如：

	$ bin/elasticsearch -d
	$ bin/elasticsearch -d

使用默认配置，两个实例的HTTP监听端口分别为9200和9201，它们的节点间通信端口分别为9300和9301，默认使用多播构成集群，集群的名称为elasticsearch；

中心LogStash只需要配置ElasticSearch的cluster name即可(如果不能发现ES集群，可以指定host: `host => "20.8.40.49"`)，如：

	output {
		elasticsearch {
			cluster => "elasticsearch"
			codec => "json"
			protocol => "http"
		}   
	}

使用ElasticSearch Cluster的优劣分析：

+ 提高组件的可用性：cluster中任何一个节点挂掉，索引及副本都会自动重新分配；
+ 极佳的水平扩展性：向cluster中增加新的节点即可，索引会自动重新组织；


### 5. 后续工作

关于ELKR日志服务，下一步的工作方向有：

+ 学习grok正则表达式，匹配自定义的日志输出格式；
+ 研究ElasticSearch的查询功能及原理；
+ 熟悉Kibana丰富的图标展示功能；

Ok，高可用方案的介绍告一段落，如果用到实际的生产环境中，应该会遇到很多意想不到的问题，后续会继续总结和分享。

####  参考：

+ [The LogStash Book](http://www.logstashbook.com/)
+ [ElasticSearch doc](http://www.elasticsearch.org/guide/)
+ [Redis Documentation](http://redis.io/documentation)
