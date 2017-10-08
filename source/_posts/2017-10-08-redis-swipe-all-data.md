title: 【问答系列】redis中如何清空所有数据？
categories: Backend
date: 2017-10-08 17:37:38
tags: [redis, 问答系列]
---

redis中与清空数据有关的命令有3个，分别是：[FLUSHALL](https://redis.io/commands/flushall), [FLUSHDB](https://redis.io/commands/flushdb), [SCRIPT FLUSH](https://redis.io/commands/script-flush)。

## FLUSHALL [ASYNC]

表示删除**所有DB**中的**所有数据**。默认是同步操作，选项`ASYNC`表示异步，即清空操作在一个新的线程中进行，不会阻塞主线程。

```redis
  $ redis-cli -h 127.0.0.1 -p 6379 FLUSHALL ASYNC
```

<!-- more -->

## FLUSHDB [ASYNC]

表示删除**当前DB**中的**所有数据**。默认是同步操作，和`FLUSHall`一样，支持选项`ASYNC`，表示异步。要删除指定DB中的所有数据，可以使用`SELECT`命令先选中DB，然后使用`FLUSHDB`命令清空数据：

```redis
  $ redis-cli -h 127.0.0.1 -p 6379 SELECT 0
  $ redis-cli -h 127.0.0.1 -p 6379 FLUSHDB
```

## SCRIPT FLUSH

表示删除**所有**的LUA脚本缓存。所有执行过的LUA脚本都会放在脚本缓存中，该命令可以强制清空所有的LUA脚本缓存。关于LUA脚本的更多内容，请参考[EVAL](https://redis.io/commands/eval)命令。

```redis
  $ redis-cli -h 127.0.0.1 -p 6379 SCRIPT FLUSH
```


## 参考

+ [FLUSHALL](https://redis.io/commands/flushall)
+ [FLUSHDB](https://redis.io/commands/flushdb)
+ [SCRIPT FLUSH](https://redis.io/commands/script-flush)
+ [EVAL](https://redis.io/commands/eval)
