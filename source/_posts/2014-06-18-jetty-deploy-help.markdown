---
layout: post
title: jetty9部署指南
date: 2014-06-18 00:18:15 +0800
comments: true
tags: jetty
categories: Backend
---

## 1. 简单有效的方式

把要部署的工程（war包、工程目录或者xml描述文件）放到${JETTY_HOME}的webapps目录下即可；

需要注意的是：
> jetty会对webapps目录下的几乎所有war包、目录、xml文件（有一些例外，如隐藏文件和.d结尾的目录等会被忽略）进行自动部署。
> 如果war包、目录和xml文件**同名**，则部署的顺序为xml文件 > war包 > 目录 。比如，webapps目录下有：rank.war，rank目录以及rank.xml，其中rank目录为rank.war解压后的目录，rank.xml中引用的是rank.war包或者rank目录，则此时，仅有xml文件被部署，这里成立的前提是**同名**，如果不同名，但它们是同一个工程，则会导致工程被重复部署，切记！（关于重复部署，参考前一篇博文：[Jetty9避免重复部署](http://nkcoder.github.io/blog/20140522/jetty-avoid-repeat-deploy/)）

我建议的做法是：将war包或解压后的目录放在webapps目录下，或者将xml描述文件放在webapps目录下，将war包或目录放在单独的目录里。

## 2. 配置context path

默认，jetty将webapps目录下的工程名作为context path，如果工程名称为ROOT，则context path为/；比如，将rank.war（或rank目录）放在webapps目录下，则context path为/rank，如果将rank.war重命名为ROOT.war，则context path为/；

如果通过文件名来配置context path无法满足要求，则可以通过xml文件来配置，如将rank.xml放在webapps目录下，添加如下内容：

	<?xml version="1.0"  encoding="UTF-8"?>
	<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure_9_0.dtd">

	<Configure class="org.eclipse.jetty.webapp.WebAppContext">

	  <Set name="contextPath">/</Set>
	  <Set name="war">/opt/www/ugc-base/webapps/RankByElasticSearch-1.0.war</Set>

	</Configure>

*contextPath*配置context path，*war*指定工程war包或目录的路径；

## 3. jetty启动/停止

建议通过${JETTY_HOME}下bin/jetty.sh脚本来启动/停止jetty，如：

	$ bin/jetty start
	$ bin/jetty stop

当然，也可以通过start.jar来启动，如：

	$ java -jar start.jar

如果希望通过start.jar停止，则在启动的时候需要指定STOP.PORT和STOP.KEY两个参数，且启动和停止时，两个参数的值必须匹配，如：

	$ java -jar start.jar STOP.PORT=8181 STOP.KEY=ugcKey
	$ java -jar start.jar STOP.PORT=8181 STOP.KEY=ugcKey --stop

所以，一般通过bin/jetty.sh控制jetty的运行，使用start.jar查看jetty的配置和状态。

## 4. 配置jetty环境变量和jvm参数

通过bin/jetty.sh来控制jetty的运行，所以编辑bin/jetty.sh文件，可以配置的变量主要有：

	JAVA: 设置java命令的绝对路径，即jdk的bin目录下的java命令的路径，如果没设置，则从PATH环境变量中查找；
	JAVA_OPTIONS：设置jvm参数；
	JETTY_HOME：jetty的安装目录，如果没有设置，则从调用该脚本的上下文环境中猜测；
	JETTY_BASE：jetty的base目录，即当前工程使用的jetty环境的根目录，如果没有设置，则与JETTY_HOME相同；
	JETTY_RUN：配置保存jetty pid文件的路径，如果没有配置，根据以下顺序查找第一个可用目录：/var/run, /usr/var/run, JETTY_BASE, /tmp；
	JETTY_PID：pid文件路径，默认为：$JETTY_RUN/$NAME.pid（NAME变量表示启动jetty时，去掉扩展名的脚本名称）；
	JETTY_ARGS：jetty参数，如配置端口号等：JETTY_ARGS=8080 jetty.spdy.port=8443
	JETTY_USER：配置启动用户，如以nkcoder用户启动：JETTY_USER=nkcoder

> 注意：以上这些变量，虽然在jetty的运行环境下都具有默认值，但是在设置时，这些参数还是空的，即不能互相引用，比如，没有显式配置JETTY_BASE，直接配置JETTY_RUN=JETTY_BASE，则此时JETTY_RUN使用的还是默认值，因为JETTY_BASE此时为空。

这里提供一个简单的配置供参考：

	JETTY_HOME=/usr/local/jetty9.1
	JETTY_BASE=$JETTY_HOME
	JETTY_RUN=$JETTY_BASE
	JETTY_USER=www
	JETTY_ARGS=jetty.port=8989
	JAVA=/usr/local/jdk7/bin/java
	JAVA_OPTIONS="-Xloggc:/opt/logs/vrsRank/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=10M -XX:+UseG1GC -XX:+UnlockExperimentalVMOptions -XX:G1MaxNewSizePercent=50 -XX:PermSize=256m -XX:MaxPermSize=256m -Xss256k -server -Xms4G -Xmx4G -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.port=18787 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -XX:+UnlockCommercialFeatures -XX:+FlightRecorder -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/opt/logs/vrsRank/oom.log"

## 5. 查看jetty配置

通过start.jar查看帮助和配置：

	$ java -jar start.jar --help

主要的查看配置的参数有：

	--list-config: 查看启动jetty使用的配置：java环境，jetty环境，JVM参数，属性，服务器classpath，服务器的xml配置等；
	--list-modules: 查看系统使用的模块
	--list-classpath: 查看系统使用的classpath
	--version：查看版本信息
	--module=<model-name>：临时启用一个模块

## 6. 一台机器上同时部署多个jetty

以在一台服务器上同时部署两个工程为例，需要两份jetty和一份jdk。和单独部署的唯一区别就是，只要确保pid和port是不同的即可。

第一种方式：修改jetty.sh脚本的名称，因为pid文件的名称就是脚本的名称，如：

工程1使用jetty1，将bin/jetty.sh重命名为bin/jetty1.sh，同时修改其配置如下（注意不用配置JETTY_RUN变量）：

	JETTY_HOME=/usr/local/jetty1
	JETTY_BASE=$JETTY_HOME
	JETTY_USER=www
	JETTY_ARGS=jetty.port=8181
	JAVA=/usr/local/jdk7/bin/java
	JAVA_OPTIONS="-Xloggc:/opt/logs/ugcRank/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"

工程2使用jetty2，将bin/jetty.sh重命名为bin/jetty2.sh，并修改配置：

	JETTY_HOME=/usr/local/jetty2
	JETTY_BASE=$JETTY_HOME
	JETTY_USER=www
	JETTY_ARGS=jetty.port=8282
	JAVA=/usr/local/jdk7/bin/java
	JAVA_OPTIONS="-Xloggc:/opt/logs/vrsRank/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"

此时，两个jetty的pid均位于默认的目录下，即/var/run，路径分别为/var/run/jetty1.pid，/var/run/jetty2.pid。

第二种方式：修改pid文件保存的目录，该目录由JETTY_RUN配置，默认都在/var/run下，如：

工程1使用jetty1，修改bin/jetty.sh如下：

	JETTY_HOME=/usr/local/jetty1
	JETTY_BASE=$JETTY_HOME
	JETTY_RUN=$JETTY_BASE
	JETTY_USER=www
	JETTY_ARGS=jetty.port=8181
	JAVA=/usr/local/jdk7/bin/java
	JAVA_OPTIONS="-Xloggc:/opt/logs/ugcRank/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"

工程2使用jetty2，修改bin/jetty.sh如下：

	JETTY_HOME=/usr/local/jetty1
	JETTY_BASE=$JETTY_HOME
	JETTY_RUN=$JETTY_BASE
	JETTY_USER=www
	JETTY_ARGS=jetty.port=8181
	JAVA=/usr/local/jdk7/bin/java
	JAVA_OPTIONS="-Xloggc:/opt/logs/ugcRank/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"

此时，pid文件位于各自jetty的安装目录下，虽然都为jetty.pid，但是互不影响。

## 7. 将jetty配置为系统服务

首先，将bin/jetty.sh拷贝到/etc/init.d中：

	$ cp bin/jetty.sh /etc/init.d/jetty

然后，新建文件/etc/default/jetty，在其中设置环境变量JETTY_HOME：

	$ vim /etc/default/jetty
	JETTY_HOME=/usr/local/jetty9.1

启动和停止：

	$ service jetty start
	$ service jetty stop

说明：bin/jetty.sh默认将/etc/default/{pid}作为其配置文件，此时pid名称即为jetty，所以/etc/default/jetty会作为jetty的配置文件，可以在其中配置JETTY_HOME, JAVA, JAVA_OPTIONS等环境变量。

### 参考：

1. [Jetty : The Definitive Reference](http://www.eclipse.org/jetty/documentation/current/index.html)
