---
layout: post
title: mysql重连，连接丢失的问题
date: 2014-07-12 20:58:06 +0800
comments: true
tags: mysql
categories: Backend
---

## 1.1 错误信息：

	Caused by: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was 20,820,001 milliseconds ago.  The last packet sent successfully to the server was 20,820,002 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.
			at sun.reflect.GeneratedConstructorAccessor29.newInstance(Unknown Source) ~[na:na]
			at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45) ~[na:1.7.0_51]
			at java.lang.reflect.Constructor.newInstance(Constructor.java:526) ~[na:1.7.0_51]
			at com.mysql.jdbc.Util.handleNewInstance(Util.java:411) ~[mysql-connector-java-5.1.29.jar:na]
			at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:1129) ~[mysql-connector-java-5.1.29.jar:na]
			at com.mysql.jdbc.MysqlIO.send(MysqlIO.java:3988) ~[mysql-connector-java-5.1.29.jar:na]
			at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2598) ~[mysql-connector-java-5.1.29.jar:na]
			at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2778) ~[mysql-connector-java-5.1.29.jar:na]
			at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2828) ~[mysql-connector-java-5.1.29.jar:na]
			at com.mysql.jdbc.ConnectionImpl.setAutoCommit(ConnectionImpl.java:5372) ~[mysql-connector-java-5.1.29.jar:na]
			at com.mchange.v2.c3p0.impl.NewProxyConnection.setAutoCommit(NewProxyConnection.java:881) ~[c3p0-0.9.1.1.jar:0.9.1.1]
			at org.quartz.impl.jdbcjobstore.AttributeRestoringConnectionInvocationHandler.setAutoCommit(AttributeRestoringConnectionInvocationHandler.java:98) ~[quartz-2.2.1.jar:na]

## 1.2 解决方法

### - 如果使用的是JDBC，在JDBC URL上添加`?autoReconnect=true`，如：

	jdbc:mysql://10.10.10.10:3306/mydb?autoReconnect=true

### - 如果是在Spring中使用DBCP连接池，在定义datasource增加属性`validationQuery`和`testOnBorrow`，如：

	<bean id="vrsRankDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}" />
        <property name="url" value="${countNew.jdbc.url}" />
        <property name="username" value="${countNew.jdbc.user}" />
        <property name="password" value="${countNew.jdbc.pwd}" />
        <property name="validationQuery" value="SELECT 1" />
        <property name="testOnBorrow" value="true"/>
    </bean>

### - 如果是在Spring中使用c3p0连接池，则在定义datasource的时候，添加属性`testConnectionOnCheckin`和`testConnectionOnCheckout`，如：

    <bean name="cacheCloudDB" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${cache.url}"/>
        <property name="user" value="${cache.user}"/>
        <property name="password" value="${cache.password}"/>
        <property name="initialPoolSize" value="10"/>
        <property name="maxPoolSize" value="${cache.maxPoolSize}"/>
        <property name="testConnectionOnCheckin" value="false"/>
        <property name="testConnectionOnCheckout" value="true"/>
        <property name="preferredTestQuery" value="SELECT 1"/>
    </bean>

### 参考

- [MySQL reconnect issues](http://itellity.wordpress.com/2013/07/18/mysql-reconnect-issues-or-the-last-packet-successfully-received-from-the-server-xx-milliseconds-ago-errors/)
