---
layout: post
title: elasticsearch查询api--terms query
date: 2014-03-20 08:24:42 +0800
tags: elasticsearch
categories: Backend
---

## 1. terms query查询

+ 提供多个查询词，以数组形式出现，查询指定的字段是否包含其中一个或者多个查询词；
+ 功能与：**bool query**通过对多个term进行`should`操作的功能是一致的，是更简单的一种语法形式；
+ 提供的查询词也是不分词的，只有完全包含才算匹配；可以通过`minimum_should_match`属性指定最少匹配的个数；

<!-- more -->

## 2. REST API

**terms query**的REST API示例如下：

	{
		"query": {
			"terms": {
				"tvName": ["日本", "停电"],
				"minimum_should_match": 1
			}
		}
	}

> 在REST API中，**terms query**使用`terms`关键字，其中*tvName*表示要查询的字段，`minimum_should_match`表示至少要匹配多少个查询词。

## 3. Java API

**terms query**的Java API示例如下：

	public static void termsQuery() {
		String[] queryWords = new String[] {"日本", "停电"};
		QueryBuilder queryBuilder = QueryBuilders.termsQuery(ConstantUtil.FIELD_TV_NAME, queryWords)
				.minimumShouldMatch("2");
		SearchResponse response = searchRequestBuilder.setQuery(queryBuilder)
				.setFrom(0).setSize(5).execute().actionGet();
		printResult(response);
	}

> 在Java API中，**terms query**使用**termsQuery()**函数，除了**minimumShouldMatch()**属性，还有**boost()**和**minimumMatch()**属性。

## 4. 看看Java源码

    public static TermsQueryBuilder termsQuery(String name, String... values) {
        return new TermsQueryBuilder(name, values);
    }

    public static TermsQueryBuilder termsQuery(String name, Object... values) {
        return new TermsQueryBuilder(name, values);
    }

    public static TermsQueryBuilder termsQuery(String name, Collection<?> values) {
        return new TermsQueryBuilder(name, values);
    }

    public TermsQueryBuilder(String name, Collection values) {
        this(name, values.toArray());
    }

    public TermsQueryBuilder(String name, Object... values) {
        this.name = name;
        this.values = values;
    }

> **termsQuery()**函数的第二个参数是一个查询词数组，可以是基本类型的数组，可以是Object数组，也可以是一个集合类型；其实，内部都是调用**TermsQueryBuilder**的一个构造函数，即上述的最后一个函数，第二个参数是Object数组；

### 参考

+ [terms-query](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-terms-query.html)
