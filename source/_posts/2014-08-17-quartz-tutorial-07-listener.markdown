---
layout: post
title: Quartz教程七--TriggerListener和JobListener
date: 2014-08-17 20:46:50 +0800
comments: true
tags: quartz
categories: Backend
---

listener是一个对象，用于监听scheduler中发生的事件，然后执行相应的操作；你可能已经猜到了，*TriggerListeners*接受与trigger相关的事件，*JobListeners*接受与jobs相关的事件。

trigger相关的事件包括：trigger的触发、trigger错过触发(mis-fire)以及trigger的完成(即trigger触发的job执行完成)。

<!-- more -->

org.quartz.TriggerListener接口

	public interface TriggerListener {

		public String getName();

		public void triggerFired(Trigger trigger, JobExecutionContext context);

		public boolean vetoJobExecution(Trigger trigger, JobExecutionContext context);

		public void triggerMisfired(Trigger trigger);

		public void triggerComplete(Trigger trigger, JobExecutionContext context,
				int triggerInstructionCode);
	}

job相关的事件包括：job即将执行的通知以及job执行完毕的通知。

org.quartz.JobListener接口：

	public interface JobListener {

		public String getName();

		public void jobToBeExecuted(JobExecutionContext context);

		public void jobExecutionVetoed(JobExecutionContext context);

		public void jobWasExecuted(JobExecutionContext context,
				JobExecutionException jobException);

	}

## 使用自定义的listener

创建一个listener，只需要实现*org.quartz.TriggerListener*接口或者*org.quartz.JobListener *接口即可；然后在运行时将listener注册到scheduler上，并且需要给listener取个名称(因为listener需要通过其getName()方法广播它的名称)。

我们可以实现上面提到的接口，但更方便的方式是继承*JobListenerSupport*类或者*TriggerListenerSupport*类，只需重写需要的方法即可。

listener是注册到scheduler的*ListenerManager*上的，与listener一同注册的还有一个*Matcher*对象，该对象用于描述listener期望接收事件的job或trigger。

> listener是在运行的时候注册到scheduler上的，而且**不会**与job和trigger一样保存在JobStore中。因为listener一般是应用的一个集成点(integration point)，因此，应用每次运行的时候，listener都应该重新注册到scheduler上。

给一个job添加JobListener：

	scheduler.getListenerManager().addJobListener(myJobListener, KeyMatcher.jobKeyEquals(new JobKey("myJobName", "myJobGroup")));

可以对matcher和key下的类进行静态导入，这样使得matcher的定义更加清晰：

	import static org.quartz.JobKey.*;
	import static org.quartz.impl.matchers.KeyMatcher.*;
	import static org.quartz.impl.matchers.GroupMatcher.*;
	import static org.quartz.impl.matchers.AndMatcher.*;
	import static org.quartz.impl.matchers.OrMatcher.*;
	import static org.quartz.impl.matchers.EverythingMatcher.*;
	...etc.

静态导入后，上面的实例可以写成：

	scheduler.getListenerManager().addJobListener(myJobListener, jobKeyEquals(jobKey("myJobName", "myJobGroup")));

给一个group下的所有job添加一个JobListener：

	scheduler.getListenerManager().addJobListener(myJobListener, jobGroupEquals("myJobGroup"));

给两个group下的所有job添加一个JobListener：

	scheduler.getListenerManager().addJobListener(myJobListener, or(jobGroupEquals("myJobGroup"), jobGroupEquals("yourGroup")));

给所有的job添加一个JobListener：

	scheduler.getListenerManager().addJobListener(myJobListener, allJobs());

TriggerListener的注册过程与JobListener类似。

Quartz的大部分用户都不使用listener，但如果应用需要对发生的事件感兴趣，当然可以让job显示通知应用，但显然listener更方便。

- [原文链接](http://quartz-scheduler.org/documentation/quartz-2.2.x/tutorials/tutorial-lesson-07)
- 本系列教程由[quartz-2.2.x官方文档](http://quartz-scheduler.org/documentation/quartz-2.2.x/tutorials)翻译、整理而来，希望给同样对quartz感兴趣的朋友一些参考和帮助，有任何不当或错误之处，欢迎指正；有兴趣研究源码的同学，可以参考[我对quartz-core源码的注释(进行中)](https://github.com/nkcoder/quartz-explained)。
