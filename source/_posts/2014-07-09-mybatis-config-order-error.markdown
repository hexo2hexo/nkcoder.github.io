---
layout: post
title: 【MyBatis】元素类型为"configuration"的内容必须匹配
date: 2014-07-09 22:23:31 +0800
comments: true
tags: mybatis
categories: Backend
---

> 错误信息：

	org.apache.ibatis.exceptions.PersistenceException:
	### Error building SqlSession.
	### Cause: org.apache.ibatis.builder.BuilderException: Error creating document instance.  Cause: org.xml.sax.SAXParseException; lineNumber: 45; columnNumber: 17; 元素类型为 "configuration" 的内容必须匹配 "(properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,plugins?,environments?,databaseIdProvider?,mappers?)"。

> 原因：在配置mybatis-config.xml时，其中的节点是有顺序的，配置顺序依次为：

	properties/settings/typeAliases/typeHandlers/objectFactory/objectWrapperFactory/plugins/environments/databaseIdProvider/mappers

> 我出现该错是因为将settings节点放到了environments后面。
