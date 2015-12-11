---
layout: post
title: Redis 3.0.0 RC1的发布和更新
keywords: redis, RC1
date: 2014-10-21 00:04:26 +0800
comments: true
tags: redis
categories: Backend
---

Redis 3.0.0 RC1在10月9日发布，这个版本具有重要的意义，说明Redis Cluster已经非常稳定了。

我们来简单梳理一下RC1中的重要更新。

<!-- more -->

## 1. Redis 3.0.0 RC1的[release notes](https://raw.githubusercontent.com/antirez/redis/3.0/00-RELEASENOTES)

### 1.1 常规更新

+ [修复] 对一些简单问题的修复汇总，修复列表可以参考[issue #1906](https://github.com/antirez/redis/pull/1906)，修复的主要内容有：
	- 将sds的size从2GB增加到4GB(即使用`unsigned int`替换`int`，其它不变)；
	- 如果redis在前台运行，Ctrl-C会将server优雅地`SHUTDOWN`，而不是仅仅将server进程kill掉；
	- Sentinel支持`INFO ALL`命令；
	- 使用`redis-benchmark`命令测试时，新增`-a`参数用于验证；
	- 对代码的优化、重构等。
+ [修复] `SAVE` 命令不会蔓延到AOF文件和Slave实例；
+ [修复] 当hash table处于异常状态时，限制`SCAN`操作的延迟(当数据库正在rehash时，可用的bucket数量很少)；
+ [新增] redis在启动时可以加载被截断的AOF文件，而不需要先执行`redis-check-aof`工具，该功能可以在`redis.conf`中配置：`aof-load-truncated yes`；
+ [新增] `INCR`：尽可能地在原地修改自增键，可以得到更好的性能；

### 1.2 Cluster更新

+ [修复] 即使连接断开，仍要求返回ping_sent时间；
+ [修复] 判断是否处于少数节点构成的网络时，修复判断逻辑；
+ [修复] 修复已加入集群的节点间的gossip通信逻辑；
+ [新增] Redis Cluster运行稳定且通过了详尽的测试，因此从beta版晋升为stable版；
+ [新增] 当slot没有被完全覆盖时，集群仍然可用；
+ [新增] 当failover过程停顿的时候，slave会打印更详细的日志，包括原因；

### 1.3 Sentinel更新

+ [修复] 修复严重bug：在计算大多数(majority)节点时，之前的实现方式是错误的，而specification上说的是对的。大多数节点是针对master的所有sentinel，而不管它当前的状态。
+ [修复] 修复hiredis库中的一个内存泄露问题：当一个被监控的实例或者另一个sentinel不可用的时候会导致内存泄露，每一次重连都会泄露少量的内存，但是如果持续累积，将会导致大量的内存泄露；

## 2. redis.conf更新

将redis-3.0.0-beta8中的redis.conf和redis-3.0.0-rc1中的redis.conf通过diff查看两者的区别。

	# diff redis-b8.conf redis-rc1.conf > diff.conf

对比后发现，大部分的更新都是针对表述上的，功能上的更新主要有三点：

+ `unixsocketperm 700`: unixsocketperm的权限由755修改为700；
+ `aof-load-truncated yes`: redis在启动的时候可以加载被截断的AOF文件，默认启用；
+ `cluster-require-full-coverage yes`: rc1之前，如果至少有一个slot没有被节点覆盖，则整个集群不可用；现在，启用该配置，即使有部分slot没有被覆盖，被覆盖到的那部分slot构成的子集群仍然可用。
