---
layout: post
title: Quartz教程八--SchedulerListener
date: 2014-08-24 20:45:27 +0800
comments: true
tags: quartz
categories: Backend
---

SchedulerListener与TriggerListener、JobListener类似，但它仅接收来自Scheduler自身的消息，而不一定是某个具体的trigger或job的消息。

scheduler相关的消息包括：job/trigger的增加、job/trigger的删除、scheduler内部发生的严重错误以及scheduler关闭的消息等；

<!-- more -->

org.quartz.SchedulerListener接口：

	public interface SchedulerListener {

		public void jobScheduled(Trigger trigger);

		public void jobUnscheduled(String triggerName, String triggerGroup);

		public void triggerFinalized(Trigger trigger);

		public void triggersPaused(String triggerName, String triggerGroup);

		public void triggersResumed(String triggerName, String triggerGroup);

		public void jobsPaused(String jobName, String jobGroup);

		public void jobsResumed(String jobName, String jobGroup);

		public void schedulerError(String msg, SchedulerException cause);

		public void schedulerStarted();

		public void schedulerInStandbyMode();

		public void schedulerShutdown();

		public void schedulingDataCleared();
	}

SchedulerListener也是注册到scheduler的ListenerManager上的，任何实现了org.quartz.SchedulerListener接口的对象都可以是SchedulerListener(译者注：SchedulerListener与JobListener/TriggerListener一样，也可以继承SchedulerListenerSupport抽象类，重写感兴趣的方法即可)。

添加一个SchedulerListener：

	scheduler.getListenerManager().addSchedulerListener(mySchedListener);

删除一个SchedulerListener：

	scheduler.getListenerManager().removeSchedulerListener(mySchedListener);

- [原文链接](http://quartz-scheduler.org/documentation/quartz-2.2.x/tutorials/tutorial-lesson-08)
- 本系列教程由[quartz-2.2.x官方文档](http://quartz-scheduler.org/documentation/quartz-2.2.x/tutorials)翻译、整理而来，希望给同样对quartz感兴趣的朋友一些参考和帮助，有任何不当或错误之处，欢迎指正；有兴趣研究源码的同学，可以参考我对[quartz-core源码的注释(进行中)](https://github.com/nkcoder/quartz-explained)。
