---
layout: post
title: Quartz教程一--使用quartz
date: 2014-06-23 21:01:32 +0800
comments: true
tags: quartz
categories: Backend
---

Scheduler在使用之前需要实例化。一般通过SchedulerFactory来创建一个实例。有些用户将factory的实例保存在JNDI中，但直接初始化，然后使用该实例也许更简单（见下面的示例）。

scheduler实例化后，可以启动(start)、暂停(stand-by)、停止(shutdown)。注意：scheduler被停止后，除非重新实例化，否则不能重新启动；只有当scheduler启动后，即使处于暂停状态也不行，trigger才会被触发（job才会被执行）。

<!-- more -->

下面的代码片段，实例化并启动一个scheduler，调度执行一个job：

	 SchedulerFactory schedFact = new org.quartz.impl.StdSchedulerFactory();

	  Scheduler sched = schedFact.getScheduler();

	  sched.start();

	  // define the job and tie it to our HelloJob class
	  JobDetail job = newJob(HelloJob.class)
		  .withIdentity("myJob", "group1")
		  .build();

	  // Trigger the job to run now, and then every 40 seconds
	  Trigger trigger = newTrigger()
		  .withIdentity("myTrigger", "group1")
		  .startNow()
		  .withSchedule(simpleSchedule()
			  .withIntervalInSeconds(40)
			  .repeatForever())
		  .build();

	  // Tell quartz to schedule the job using our trigger
	  sched.scheduleJob(job, trigger);

你看到了，quartz的使用并不难。[教程二](http://nkcoder.github.io/blog/20140624/quartz-tutorial-api-job-trigger/)会简要地介绍job和trigger，以及quartz的API，然后你会更好地理解上面的示例。

- [原文链接](http://quartz-scheduler.org/documentation/quartz-2.2.x/tutorials/tutorial-lesson-01)
- 本系列教程由[quartz-2.2.x官方文档](http://quartz-scheduler.org/documentation/quartz-2.2.x/tutorials)翻译、整理而来，希望给同样对quartz感兴趣的朋友一些参考和帮助，有任何不当或错误之处，欢迎指正；有兴趣研究源码的同学，可以参考[我对quartz-core源码的注释(进行中)](https://github.com/nkcoder/quartz-explained)。
