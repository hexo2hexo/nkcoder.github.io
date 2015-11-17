---
layout: post
title: quartz教程--快速入门
date: 2014-06-22 10:11:23 +0800
comments: true
tags: quartz
categories: Backend
---

欢迎来到quartz快速入门教程。阅读本教程，你将会了解：

- quartz下载
- quartz安装
- 根据你的需要，配置Quartz
- 开始一个示例应用

<!-- more -->

当熟悉了quratz调度的基本功能后，可以尝试一些更高级的特性，比如[Where](http://terracotta.org/documentation/quartz-scheduler/quartz-scheduler-where)，这个一个企业级功能，可以让job和trigger运行在指定的，而不是随机的Terracotta客户端上。

## 下载和安装

首先，[下载最新的稳定版](http://quartz-scheduler.org/downloads/) - 不用注册。解压并安装。

### Quartz jar文件

quartz安装包根目录的lib/目录下有很多的jar包。其中，quartz-xxx.jar（其中xxx是版本号）是最主要的。为了使用quartz，必须将该jar包放在应用的classpath下；

下载后，解压，然后将quartz-xxx.jar放到你的应用中。

我主要是在应用服务器的环境中使用quartz，所以一般将quartz jar包放到应用中（.ear或.war）。当然，如果你希望在很多应用中使用quartz，将quartz的jar包放在应用服务器(appserver)的classpath下即可。如果你只是希望在独立的应用中使用quartz，将quartz的jar包和你的应用依赖的其它jar包放在一起即可。

quzrtz依赖一些第三方的库（以jar包的形式），这些库位于quartz安装包的`lib`目录下。要使用quartz的所有功能，必须将所有的第三方jar包都放到classpath下。如果你开发的是一个独立的quartz应用，建议将所有的jar包都放到classpath下；如果是在应用服务器环境下使用quartz，其中有些包可能已经存在于classpath中了，因此你需要自己选择。

>  在应用服务器环境下，如果同一个jar文件，存在两个不同的版本，要注意，可能会产生一些奇怪的结果；比如，WebLogic包含了一个J2EE的实现（在weblogic.jar中），该实现与servlet.jar的实现可能不一致。此时，应该从你的应用中排除掉servlet.jar，这样你就知道使用的是哪个类了。

### properties文件

quartz使用名为quartz.properties的配置文件。刚开始时该配置文件不是必须的，但是为了使用最基本的配置，该文件必须位于classpath下。

基于我的个人情况举个例子，我的应用是基于WebLogic Workshop开发的。我将所有的配置文件（包括quartz.properties）放到应用根目录下的一个项目中。当我将项目打包成.ear文件时，放置配置文件的项目会以jar包的形式进入最终的.ear包，所以quartz.properties文件就自动位于classpath中了。

如果你准备构建一个使用quartz的web应用（以.war包的形式），你应该将quartz.properties文件放到WEB-INF/classes目录下。

## 配置

这里包含很多内容。quartz是一个配置很灵活的应用。配置quartz最好的方式是，编辑quartz.properties文件，然后放到应用的classpath下。

quartz的安装包中包含了一些配置文件的示例，位于example/目录下。我建议你创建自己的quartz.properties文件，而不是简单地从示例中拷贝并删除不需要的部分。这样看起来更整洁，而且你也会了解到quartz的更多功能。

关于quartz配置文件的详细文档，请查阅[Quartz配置参考](http://quartz-scheduler.org/documentation/quartz-2.2.x/configuration)

为了使用quartz，一个基本的quartz.properties配置文件如下所示：

	org.quartz.scheduler.instanceName = MyScheduler
	org.quartz.threadPool.threadCount = 3
	org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore

上述配置的scheduler有如下特点：

- org.quartz.scheduler.instanceName - scheduler的名称为“MyScheduler”
- org.quartz.threadPool.threadCount - 线程池中有3个线程，即最多可以同时执行3个job；
- org.quartz.jobStore.class - quartz的所有数据，包括job和trigger的配置，都会存储在内存中（而不是数据库里）。如果你想使用quartz的数据库存储功能，我们建议在使用数据库存储之前，先使用内存存储（RamJobStore）。

## 示例应用

下载和安装完quartz后，是时候开发一个示例应用，并让它跑起来了。下面的示例代码，获取scheduler实例对象，启动，然后关闭。

QuartzTest.java

	import org.quartz.Scheduler;
	import org.quartz.SchedulerException;
	import org.quartz.impl.StdSchedulerFactory;
	import static org.quartz.JobBuilder.*;
	import static org.quartz.TriggerBuilder.*;
	import static org.quartz.SimpleScheduleBuilder.*;

	public class QuartzTest {

		public static void main(String[] args) {

			try {
				// Grab the Scheduler instance from the Factory
				Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();

				// and start it off
				scheduler.start();

				scheduler.shutdown();

			} catch (SchedulerException se) {
				se.printStackTrace();
			}
		}
	}

> 当你调用StdSchedulerFactory.getDefaultScheduler()获取scheduler实例对象后，在调用scheduler.shutdown()之前，scheduler不会终止，因为还有活跃的线程在执行。

注意示例代码中的静态导入(static import)，下面的代码中也会用到它们。

如果你没有配置日志输出，所有的日志会输出到控制台，比如：

	[INFO] 21 Jan 08:46:27.857 AM main [org.quartz.core.QuartzScheduler]
	Quartz Scheduler v.2.0.0-SNAPSHOT created.

	[INFO] 21 Jan 08:46:27.859 AM main [org.quartz.simpl.RAMJobStore]
	RAMJobStore initialized.

	[INFO] 21 Jan 08:46:27.865 AM main [org.quartz.core.QuartzScheduler]
	Scheduler meta-data: Quartz Scheduler (v2.0.0) 'Scheduler' with instanceId 'NON_CLUSTERED'
	  Scheduler class: 'org.quartz.core.QuartzScheduler' - running locally.
	  NOT STARTED.
	  Currently in standby mode.
	  Number of jobs executed: 0
	  Using thread pool 'org.quartz.simpl.SimpleThreadPool' - with 50 threads.
	  Using job-store 'org.quartz.simpl.RAMJobStore' - which does not support persistence. and is not clustered.


	[INFO] 21 Jan 08:46:27.865 AM main [org.quartz.impl.StdSchedulerFactory]
	Quartz scheduler 'Scheduler' initialized from default resource file in Quartz package: 'quartz.properties'

	[INFO] 21 Jan 08:46:27.866 AM main [org.quartz.impl.StdSchedulerFactory]
	Quartz scheduler version: 2.0.0

	[INFO] 21 Jan 08:46:27.866 AM main [org.quartz.core.QuartzScheduler]
	Scheduler Scheduler_$_NON_CLUSTERED started.

	[INFO] 21 Jan 08:46:27.866 AM main [org.quartz.core.QuartzScheduler]
	Scheduler Scheduler_$_NON_CLUSTERED shutting down.

	[INFO] 21 Jan 08:46:27.866 AM main [org.quartz.core.QuartzScheduler]
	Scheduler Scheduler_$_NON_CLUSTERED paused.

	[INFO] 21 Jan 08:46:27.867 AM main [org.quartz.core.QuartzScheduler]
	Scheduler Scheduler_$_NON_CLUSTERED shutdown complete.

你可以在start()和shutdown()之间做一些有趣的事情：

    // define the job and tie it to our HelloJob class
    JobDetail job = newJob(HelloJob.class)
        .withIdentity("job1", "group1")
        .build();

    // Trigger the job to run now, and then repeat every 40 seconds
    Trigger trigger = newTrigger()
        .withIdentity("trigger1", "group1")
        .startNow()
        .withSchedule(simpleSchedule()
                .withIntervalInSeconds(40)
                .repeatForever())            
        .build();

    // Tell quartz to schedule the job using our trigger
    scheduler.scheduleJob(job, trigger);

> 在调用shutdown()之前，你需要给job的触发和执行预留一些时间，比如，你可以调用Thread.sleep(60000)让线程睡眠一段时间。

好了，自己去探索吧！

- [原文链接](http://quartz-scheduler.org/documentation/quartz-2.2.x/quick-start)
- 本系列教程由[quartz-2.2.x官方文档](http://quartz-scheduler.org/documentation/quartz-2.2.x/tutorials)翻译、整理而来，希望给同样对quartz感兴趣的朋友一些参考和帮助，有任何不当或错误之处，欢迎指正；有兴趣研究源码的同学，可以参考[我对quartz-core源码的注释(进行中)](https://github.com/nkcoder/quartz-explained)。
