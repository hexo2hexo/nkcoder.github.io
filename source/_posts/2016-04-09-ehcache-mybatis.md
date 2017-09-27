title: Ehcache以及与MyBatis的集成
categories: Backend
date: 2016-04-09 22:23:36
tags: [缓存, Ehcache, MyBatis]
---

Ehcache是目前使用很广泛的Java系的开源cache。可以把它当作通用的cache，或者作为Hibernate/MyBatis等的二级缓存。本文简要介绍在MyBatis中集成Ehcache，其中Ehcache的版本是**2.10.1**。

## 1. Ehcache的配置

Ehcache默认使用**CLASSPATH**根目录下的`ehcache.xml`作为配置文件，如果没找到，则使用Jar包下的`ehcache-failsafe.xml`作为配置文件，该配置文件提供了默认的简单配置：

<!--more-->

		```xml
		<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
			<diskStore path="java.io.tmpdir"/>
			<defaultCache
						maxElementsInMemory="10000"
						eternal="false"
						timeToIdleSeconds="120"
						timeToLiveSeconds="120"
						maxElementsOnDisk="10000000"
						diskExpiryThreadIntervalSeconds="120"
						memoryStoreEvictionPolicy="LRU">
					<persistence strategy="localTempSwap"/>
			</defaultCache>
		</ehcache>
		```

`<ehcache>`节点对应一个CacheManager，一个CacheManager可以管理多个cache实例。`ehcache`节点可配置参数主要有：

	- updateCheck: CacheManager是否自动在线检测Ehcache的新版本，默认为true；
	- dynamicConfig: 是否允许cache动态配置，即运行时动态调整内存、磁盘等的容量，默认为true；

以下三个配置由CacheManager管理的所有cache共享，单位可以时B/M/G：

	- maxBytesLocalHeap: CacheManager可用的最大heap内存;
	- maxBytesLocalOffHeap: CacheManager可用的最大的off-heap内存；
	- maxBytesLocalDisk: CacheManager可用的最大的本地disk空间；

`<diskStore>`节点表示启用磁盘cache时的文件路径，如果使用了磁盘作为cache，但是没有指定`<diskStore>`，则Ehcache默认使用`java.io.tmpdir`作为cache的磁盘路径；

`<cache>`节点和`<defaultCache>`都用于表示一个cache实例，不同的是`<cache>`需要配置`name`属性，而`<defaultCache>`不需要，因为它的name默认为`default`，`<defaultCache>`用于未显式配置的cache。

`<cache>`节点中的主要参数有：

- name：唯一标识cache实例；
- maxEntriesLocalHeap：Memory中可保存的对象的最大数量，默认为0表示不限；
- maxEntriesLocalDisk：Disk中可保存的对象的最大数量，默认为0表示不限；
- eternal：表示cache是否过期，如果eternal为true，则对象永不过期；
- maxBytesLocalHeap：该实例的最大可用Heap，不能超过`<ehcache>`中配置到CacheManager的最大Heap容量，如果使用了maxBytesLocalHeap，则不能使用maxBytesLocalHeap；
- maxBytesLocalDisk：该实例的最大可用磁盘容量；
- timeToIdleSeconds：表示对象最后一次访问到过期的时间，默认为0，表示不过期，该参数仅当eternal为false时有效；
- timeToLiveSeconds：表示对象从创建到过期的时间，默认为0，表示不过期，该参数仅当eternal为false时有效；
- memoryStoreEvictionPolicy：当cache的对象达到maxEntriesLocalHeap限制时使用的剔除策略，默认为`LRU`，可用值有：LRU, FIFO, LFU

`<persistence>`节点的参数:

	- <strategy>表示持久化方式，值为localTempSwap表示当heap/off-heap满的时候，将缓存的对象持久化到disk，none表示不持久化chache；

## 2. MyBatis中配置Ehcache

首先添加`mybatis-ehcache`依赖，当前版本是**1.0.3**，然后配置`ehcache.xml`并放在CLASSPATH的根目录下，最后在mapper文件中定义`<cache>`节点，如：

	<mapper namespace="xxx">
	    <cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
	    <select id="xxx" parameterType="map" resultType="xxx">
	        ...
	    </select>
	</mapper>

也可以在`<cache>`节点中配置ecache，就不需要额外的`.ecache.xml`配置了，如：

		```
		<cache type="org.mybatis.caches.ehcache.EhcacheCache">
        <property name="timeToIdleSeconds" value="3600"/>
        <property name="timeToLiveSeconds" value="3600"/>
        <property name="maxEntriesLocalHeap" value="1000"/>
        <property name="maxEntriesLocalDisk" value="100000"/>
        <property name="memoryStoreEvictionPolicy" value="LRU"/>
    </cache>
		```

在`mybatis-ehcache`的1.1.0-SNAPSHOT中，cache的type，除了`EhcacheCache`，还可以是`EhBlockingCache`。`EhBlockingCache`的主要应用场景是要缓存的数据是动态变化的，而并发访问数据的请求非常高，此时使用阻塞cache，让第一个线程去cache，其它等待的限制只需要直接去cache中取数据即可。

### 参考

1. [Configuring Cache](http://www.ehcache.org/generated/2.10.1/html/ehc-all/#page/Ehcache_Documentation_Set%2Fto-cfgbasics_configuring_cache.html%23)
2. [MyBatis Ehcache Adapter - Reference Documentation](http://www.mybatis.org/ehcache-cache/index.html)
3. [ehcache.xml](http://www.ehcache.org/ehcache.xml)

