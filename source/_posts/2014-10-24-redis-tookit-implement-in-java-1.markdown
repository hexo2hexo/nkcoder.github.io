---
layout: post
title: redis-toolkit--Java实现的redis工具(一)
date: 2014-10-24 11:21:49 +0800
comments: true
tags: redis
categories: Backend
---

redis作者提供了redis-trib.rb工具，用来与redis cluster进行交互，该工具使用ruby实现；对该工具提供的主要功能，我在redis的java客户端jedis的基础上进行了对应的实现，希望给同样使用java与redis交互的朋友一些参考。

<!--more-->

## 1. 创建集群

使用作者在[cluster-tutorial](http://redis.io/topics/cluster-tutorial)上的示例，即在localhost的7000、7001、7002、7003、7004、7005启动6个redis实例，且都启用cluster模式，使用redis-trib创建集群的命令为：

	./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005

上述命令表示使用这6个实例创建集群，3主3从；

实际上，这一条命令，如果拆分为原生的redis命令来实现，则主要有以下4个过程：

	1. 使用`CLUSTER MEET`命令将所有节点构建成一个集群；
	2. 使用`CLUSTER REPLICATE`命令设置master/slave结构；
	3. 使用`CLUSTER SETSLOTS`命令将16384个slot分配到集群中的master中；
	4. 等待集群的状态变为OK；

对于以上原生命令，jedis都是支持的，所以根据以上4个步骤，使用jedis创建集群就不难了；
在redis-tookit中整个创建的过程被封装到Create#create()方法中了，示例代码：

	/* 构建一个3主3从的集群 */
	ArrayListMultimap<HostAndPort, HostAndPort> clusterNodes = ArrayListMultimap.create();
	clusterNodes.put(HostAndPort.fromString("127.0.0.1:7000"), HostAndPort.fromString("127.0.0.1:7001"));
	clusterNodes.put(HostAndPort.fromString("127.0.0.1:7002"), HostAndPort.fromString("127.0.0.1:7003"));
	clusterNodes.put(HostAndPort.fromString("127.0.0.1:7004"), HostAndPort.fromString("127.0.0.1:7005"));
	Create.create(clusterNodes);

## 2. 数据迁移(reshard)

在beta8之前，使用redis-trib.rb做reshard时，仅支持交互时模式，beta8之后，开始支持非交互式模式。比如，从源节点38807bd0262d99f205ebd0eb3e483cc09e927731上迁移100个slot到目标节点47ef6c293bb3f9763d421f56c63f00cf06ef5b3f的非交互式命令为：

	redis-trib.rb reshard --from 38807bd0262d99f205ebd0eb3e483cc09e927731 --to 47ef6c293bb3f9763d421f56c63f00cf06ef5b3f --slots 100 --yes 127.0.0.1:7000

那么，使用redis的原生命令如何完成数据的reshard呢？其实，在数据reshard时，slot是逐个迁移的，redis迁移一个slot的步骤为：

	1. 对目标节点发送`CLUSTER SETSLOT <slot> IMPORTING <source_node_id>`命令，表示目标节点将从源节点迁移slot；
	2. 对源节点发送`CLUSTER SETSLOT <slot> MIGRATING <target_node_id>`命令，表示源节点将向目标节点迁移slot；
	3. 对源节点发送`CLUSTER GETKEYSINSLOT <slot> <count>`命令，表示从slot中取出count个key/value对的key；
	4. 对源节点发送`CLUSTER MIGRATE <target_ip> <target_port> <key_name> 0 <timeout>`命令，表示将key迁移到目标节点，对步骤3中的每个key都会执行该命令；
	5. 重复执行步骤3和4，直到该slot中所有的key都被迁移完毕；
	6. 向集群中的任一节点发送`CLUSTER SETSLOT <slot> NODE <target_node_id>`，表示告诉集群，将该slot分配给目标节点；
	7. 等待集群的状态变为OK；

使用java实现时，也是通过jedis调用redis原生命令，根据以上步骤来实现的。在redis-toolkit中，封装了两个方法，Reshard#migrateSlots()表示迁移一个slot，Reshard#migrate()表示批量迁移，批量迁移时，从源节点的slot索引，从低到高依次迁移，使用示例：

	/** 将slot 9189从节点7002迁移到节点7006 **/
	HostAndPort srcNodeInfo = HostAndPort.fromString("10.7.40.49:7002");
	HostAndPort destNodeInfo = HostAndPort.fromString("10.7.40.49:7006");
	Reshard.migrateSlots(srcNodeInfo, destNodeInfo, 9189);

	/** 从节点7000迁移100个slot到节点7004 **/
	HostAndPort srcNodeInfo = HostAndPort.fromString("10.7.40.49:7000");
	HostAndPort destNodeInfo = HostAndPort.fromString("10.7.40.49:7004");
	Reshard.migrate(srcNodeInfo, destNodeInfo, 100);

## 3. 节点的增加/删除

### 3.1 增加节点

无论要添加的是master节点还是slave节点，首先都需要将该节点加入到集群中，然后，如果是master节点，则迁移一部分数据到该节点上，如果是slave节点，则使该节点成为其master节点的slave。

通过redis-trib.rb工具，增加一个master节点命令为：

	./redis-trib.rb add-node 127.0.0.1:7006 127.0.0.1:7000

然后通过`redis-trib.rb reshard`命令迁移数据到新节点7006上；

增加一个slave节点，redis-trib.rb提供了两种方式：

	./redis-trib.rb add-node --slave 127.0.0.1:7006 127.0.0.1:7000

	./redis-trib.rb add-node --slave --master-id 3c3a0c74aae0b56170ccb03a76b60cfe7dc1912e 127.0.0.1:7006

第一种方式将该slave节点随机分配给集群中具有最少slave的master节点，第二种方式是将slave节点分配给指定的master节点；更通用的方式，则是先将该节点作为空的master节点加入集群，然后通过`CLUSTER REPLICATE`命令使其成为一个master节点的slave。

对redis集群进行扩容时，可分为垂直扩容和水平扩容，垂直扩容即通过`CONFIG SET`命令在线修改redis实例的`maxmemory`的大小，水平扩容即增加新的master节点，然后迁移一部分数据到新的节点上。

向集群中添加一个master节点，则相当于水平扩容，给集群中的一个节点增加一个slave节点，则不是扩容。

所以，增加节点的步骤为：

	- 使用`CLUSTER MEET`命令将节点加入到当前集群；
	- 如果要加入的是master节点，则迁移slot到新节点上(参考第2步的“数据迁移”)；
	- 如果要加入的是slave节点，则使用`CLUSTER REPLICATE`命令建立master/slave关系；

redis-tookit中提供的增加节点的方法为Manage#addNewNode()，支持增加master节点和slave节点；如果增加的是master节点，则会迁移数据到新的master节点，为了迁移操作更均衡，根据每个master节点上的slot数等比例地迁移，即从slot更多的节点上迁移更多的slot，从slot少的节点上迁移更少的slot，使用示例：

	/** 向集群中增加一个master节点7006 **/
	HostAndPort clusterNodeInfo = HostAndPort.fromString("10.7.40.49:7000");
	HostAndPort newMaster = HostAndPort.fromString("10.7.40.49:7006");
	Manage.addNewNode(clusterNodeInfo, newMaster, null);

	/** 给集群中增加一个slave节点8001，作为master节点7002的slave **/
	HostAndPort clusterNodeInfo = HostAndPort.fromString("10.7.40.49:7000");
	HostAndPort master = HostAndPort.fromString("10.7.40.49:7002");
	HostAndPort newSlave = HostAndPort.fromString("10.7.40.49:8001");
	Manage.addNewNode(clusterNodeInfo, newSlave, master);

## 3.2 删除节点

从集群中删除一个节点，如果要删除的是slave节点，直接将节点从集群中剔除即可，如果要删除的是master节点，则需要先将所有的slot迁移走，然后删除节点。

redis-trib.rb工具提供的删除一个节点的命令为：

	./redis-trib del-node 127.0.0.1:7000 3c3a0c74aae0b56170ccb03a76b60cfe7dc1912e

删除节点，就是调用`CLUSTER FORGET`命令，将要删除的节点从当前集群中踢掉，需要注意的是：

	- 需要对集群中的每一个节点(无论是master还是slave)都调用`CLUSTER FORGET`来“忘记”该节点，因为如果集群中有一个节点还“认识”待删除的节点，则通过消息，不久集群中的所有节点都”认识“待删除的节点了；
	- 一个slave节点不能”忘记“它的master节点；
	- 一个节点不能”忘记“它自己；

所以，删除一个节点的步骤为：

	- 如果节点是master节点，先将节点上的slot迁移，然后从集群中删除该master节点以及它的所有slave节点；
	- 如果节点是slave节点，则直接从集群中删除即可；

redis-tookit中提供了删除master节点和slave节点的方法，删除master节点时，需要迁移数据，当前的实现是将待删除的master节点上的所有slot平均分配给集群中的其它master；使用示例：

	/* 从集群中删除一个节点，删除时，集群会判断待删除的节点是master还是slave */
	HostAndPort oneNode = HostAndPort.fromString("10.7.40.49:7000");
	HostAndPort nodeToDelete = HostAndPort.fromString("10.7.40.49:7006");
	Manage.removeNode(oneNode, nodeToDelete);

redis-tookit代码在https://github.com/nkcoder/redis-toolkit

## 参考

+ [Redis cluster tutorial](http://redis.io/topics/cluster-tutorial)
+ [Redis设计与实现](http://book.douban.com/subject/25900156/)
