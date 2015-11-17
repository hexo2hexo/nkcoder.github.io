---
layout: post
title: 将Jetty配置为Linux服务
date: 2014-04-11 08:04:08 +0800
tags: jetty
categories: Backend
---

最近将项目都升级到了Jetty 9 和JDK 7，记录一下配置的过程。以下配置基于的版本为：

- Jetty：9.1.1.v20140108
- JDK：1.7.0_51
- 官方文档版本：2014-03-17T10:14:41-05:00

## 1. 作为root用户配置

如果你希望以root用户启动jetty。并将jetty配置为服务，以下为快速通道：

### 1.1 将脚本jetty.sh拷贝init.d目录下

	# cp bin/jetty.sh /etc/init.d/jetty

> jetty.sh是jetty的启动脚本。

### 1.2 配置/etc/default/jetty

	# vim /etc/default/jetty
	---------------------------------------------------
	PATH=$PATH:/usr/local/jdk1.7.0_51/bin
	JETTY_HOME=/usr/local/jetty9.1
	TMPDIR=/usr/local/jetty9.1/tmp
	---------------------------------------------------

> jetty.sh脚本会将/etc/default/下同名的文件作为其配置文件。需要配置几个环境变量：主要是java和jetty的根目录，以及指定一个目录作为jetty解压war包时的临时目录，默认是/tmp（不够安全，因为系统服务可能会不定时删除/tmp目录）。

### 1.3 启动与关闭

	# service jetty start
	# service jetty stop

> init.d目录下的jetty就是服务名.
参考：[Startup a Unix Service using jetty.sh](http://www.eclipse.org/jetty/documentation/current/startup-unix-service.html)


## 2. 作为普通用户配置

如果你希望以普通用户运行jetty，并且可以灵活扩展到多工程的部署，同时jetty升级方便，可以参考一下步骤。

### 2.1 配置工程目录

新建工程目录vrs-base

	[root@localhost www]# mkdir vrs-base
	[root@localhost www]# cd vrs-base/
	[root@localhost www]# mkdir tmp
	[root@localhost vrs-base]# pwd
	/opt/www/vrs-base

配置jetty环境

	[root@localhost vrs-base]# cp $JETTY_HOME/start.ini /opt/www/vrs-base
	[root@localhost vrs-base]# java -jar $JETTY_HOME/start.jar --add-to-start=http,deploy,logging

修改端口

	[root@localhost vrs-base]# grep port start.ini
	jetty.port=8080

### 2.2 配置服务脚本和属性

	[root@localhost www]# cp $JETTY_HOME/bin/jetty.sh /etc/init.d/jetty-vrs
	[root@localhost www]# vim /etc/default/jetty-vrs
	---------------------------------------------------
	PATH=$PATH:/usr/local/jdk1.7.0_51/bin
	JETTY_HOME=/usr/local/jetty9.1
	JETTY_BASE=/opt/www/vrs-base
	TMPDIR=/opt/www/vrs-base/tmp
	---------------------------------------------------

> JETTY_HOME环境变量表示jetty的安装根目录，JETTY_BASE环境变量表示当前应用的根目录，TEPDIR表示应用被jetty解压后的临时目录，不建议使用默认的/tmp目录。

### 2.3 应用部署

	[root@localhost vrs-base]# cp /opt/www/vrs.xml /opt/www/vrs-base/webapps/
	[root@localhost vrs-base]# cp /opt/www/vrs.war /opt/www/vrs-base/webapps/

> 将应用的xml配置文件和war包放到$JETTY_BASE/webapps目录中，jetty对该目录下的内容进行自动部署和扫描。

### 2.4 新建用户/修改权限

	# useradd -U www
	# chown -R www $JETTY_HOME
	# chown -R www /opt/www
	# chown -R www /opt/logs

> **注意**：所有jetty会访问的目录都需要修改权限，如：排行榜系统在/opt/rank目录下生成文件，则该目录也需要修改为www用户权限。

### 2.5 启动与停止

	[root@localhost vrs-base]# su - www service jetty-vrs start
	Starting Jetty: 2014-04-04 10:17:58.452:INFO::main: Redirecting stderr/stdout to /opt/www/vrs-base/logs/2014_04_04.stderrout.log
	. . . . OK Fri Apr  4 10:18:17 CST 2014

	[root@localhost vrs-base]# ps -ef | grep jetty
	www       2821     1 65 10:19 pts/1    00:00:15 /usr/local/jdk1.7.0_51/bin/java -Djetty.state=/opt/www/vrs-base/jetty-vrs.state -Djetty.logs=/opt/www/vrs-base/logs -Djetty.home=/usr/local/jetty9.1 -Djetty.base=/opt/www/vrs-base -Djava.io.tmpdir=/opt/www/vrs-base/tmp -jar /usr/local/jetty9.1/start.jar jetty-logging.xml jetty-started.xml
	root      2887  1929  0 10:19 pts/1    00:00:00 grep jetty

	[root@localhost vrs-base]# su - www service jetty-vrs stop
	Stopping Jetty: OK

### 2.6 多应用部署

可以使用同一个jetty，启动多个实例，每个实例部署一个应用，并作为服务运行。
重复2.1~2.5的步骤，只需要再定义一个JETTY_BASE，在/etc/init.d目录下添加一个对应的服务名即可。
即：将2.1中的vrs-base修改为ugc-base，将2.2中的jetty-vrs修改为jetty-ugc，启动和停止：

	$ su - www service jetty-ugc start
	$ su - www service jetty-ugc stop

### 附录

xml配置文件示例(feedback.xml):

	<?xml version="1.0"  encoding="UTF-8"?>
	<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure_9_0.dtd">

	<Configure class="org.eclipse.jetty.webapp.WebAppContext">

	  <Set name="contextPath">/</Set>
	  <Set name="war">/opt/www/feedback-base/webapps/feedback.war</Set>
	  <Set name="tempDirectory">/opt/www/feedback-base/tmp</Set>
	  <Set name="persistTempDirectory">true</Set>

	   <Set name="handler">
		<New id="RequestLog" class="org.eclipse.jetty.server.handler.RequestLogHandler">
		  <Set name="requestLog">
			<New id="RequestLogImpl" class="org.eclipse.jetty.server.NCSARequestLog">
			  <Set name="filename"><Property name="jetty.logs" default="./logs"/>/feedback-yyyy_mm_dd.request.log</Set>
			  <Set name="filenameDateFormat">yyyy_MM_dd</Set>
			  <Set name="LogTimeZone">Asia/Shanghai</Set>
			  <Set name="retainDays">60</Set>
			  <Set name="append">true</Set>
			</New>
		  </Set>
		</New>
	  </Set>

	</Configure>

/etc/default目录下的配置文件示例(jetty-feedback)：

	JAVA=/usr/local/jdk7/bin/java
	JETTY_HOME=/usr/local/jetty9
	JETTY_BASE=/opt/www/feedback-base
	TMPDIR=/opt/www/feedback-base/tmp

启动/停止脚本示例(jettyctl.sh)

	# usage
	usage()
	{
		echo "Usage: ${0##*/} {start|stop|status|restart}"
		exit 1
	}

	# need one param
	if [ $# -lt 1 ]; then
		usage
	fi

	action=$1

	# control jetty
	case "$action" in
		start)
			su - www service jetty-vrs start
		;;
		stop)
			su - www service jetty-vrs stop
		;;
		status)
			su - www service jetty-vrs status
		;;
		restart)
			su - www service jetty-vrs restart
		;;
		*)  
			usage
		;;
	esac

### 参考

+ [Startup a Unix Service using jetty.sh](http://www.eclipse.org/jetty/documentation/current/startup-unix-service.html)
