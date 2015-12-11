---
layout: post
title: elasticsearch查询api--match query
date: 2014-03-23 22:55:03 +0800
tags: elasticsearch
categories: Backend
---

## 1. match query

+ 与**term query**不同，**match query**的查询词是被分词处理的（analyzed），即首先分词，然后构造相应的查询，所以应该确保查询的分词器和索引的分词器是一致的；
+ 与**terms query**相似，提供的查询词之间默认是or的关系，可以通过`operator`属性指定；
+ **match query**有两种形式，一种是简单形式，一种是bool形式；

<!-- more -->

## 2. REST API

	# curl -XGET 'localhost:9200/video/video_info/_search?pretty' -d @match_query.json

简单形式的**match query**：

	{
		"query": {
			"match": {
				"tvName": "决战"
			}
		}
	}

bool形式的**match query**：

	{
		"query": {
			"match": {
				"tvName": {
					"query": "决战华岩寺",
					"operator": "or",
					"minimum_should_match": "2"
				}
			}
		}
	}

> 在REST API中，`tvName`表示要查询的目标字段，也可以是`_all`。bool形式的**match query**支持的属性有：

+ `operator`: 指定构造查询时的布尔操作，可以是`and`、`or`，默认是`or`；
+ `analyzer`：指定查询分词器，默认为默认的分词器；
+ `minimum_should_match`: 最小匹配个数

## 3. Java API

以下为**match query**的Java API示例：

	public static void matchQuery() {
		String queryWord = "天津电视台";
		QueryBuilder matchQuery = QueryBuilders.matchQuery(ConstantUtil.FIELD_TV_NAME, queryWord)
				.analyzer("ik").operator(MatchQueryBuilder.Operator.OR).minimumShouldMatch("1");
		SearchResponse response = searchRequestBuilder.setQuery(matchQuery)
				.setFrom(0).setSize(5).execute().actionGet();
		printResult(response);
	}

## 4. 看看Java源码

    public static MatchQueryBuilder matchQuery(String name, Object text) {
        return new MatchQueryBuilder(name, text).type(MatchQueryBuilder.Type.BOOLEAN);
    }

> **match query**对应的是`matchQuery()`函数，内部先调用`MatchQueryBuilder`的构造函数，然后将类型设置为*Boolean*，即先分词，然后构造布尔查询；

### 5. 参考：

+ [match query](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-match-query.html)
