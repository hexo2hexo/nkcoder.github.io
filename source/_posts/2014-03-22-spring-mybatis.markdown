---
layout: post
title: Spring持久化之MyBatis
date: 2014-03-22 20:12:52 +0800
tags:
  - spring
  - mybatis
categories: Backend
---

MyBatis是一个优秀的轻量级持久化框架，本文主要介绍MyBatis与Spring集成的配置与用法。

## 1. Spring MyBatis配置

### 1.1 添加Maven依赖

在pom.xml文件里添加mybatis-spring和mybatis的依赖：

	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis-spring</artifactId>
		<version>${mybatis.spring.version}</version>
	</dependency>

	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>${mybatis.version}</version>
	</dependency>

> mybatis-spring当前最新版本为1.2.2，mybatis当前版本是3.2.5.

### 1.2 添加dao接口

这里的dao必须是接口，而不是具体的实现，如MyBatisDao.java内容为：

	public interface MyBatisTest {

		public String getUserNameById(int id);

		public List<Students> getStudentByNumAndCity(Map<String, Object> queryMap);
	}

> 接口中定义的每一个方法对应于mapper映射文件中定义的jdbc执行模块，如`<select/>`、`<update/>`、`<insert/>`等。

### 1.3 添加mybatis配置文件

该配置文件里主要配置类型别名`<typeAliases/>`、设置`<settings/>`，mapper映射文件路径`<mappers/>`也可以放在这里，但更建议将所有的mapper文件都放在一个目录下，在定义`sqlSessionFactory`时通过属性`mapperLocations`指定。如mybatis.xml配置文件可以如下定义：

	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

	<configuration>
		<typeAliases>
			<typeAlias type="com.sohu.tv.bean.Students" alias="Students"/>
		</typeAliases>

		<!--<mappers>-->
			<!--<mapper resource="com/sohu/tv/mapper/MybatisTest.xml"/>-->
		<!--</mappers>-->

	</configuration>

> 类型别名是用别名来代表全限定类名，如在需要用到com.sohu.tv.bean.Students的地方，都可以使用Students来代替。

### 1.4 添加mapper映射文件：

mapper映射文件可以定义数据库列与POJO类属性的映射，以及与dao接口类中的方法对应的JDBC执行模块，如MyBatisMapper.xml的内容为：

	<mapper namespace="com.sohu.tv.dao.MyBatisTest">
		<resultMap id="studentMap" type="Students">
			<result column="name" property="name"/>
			<result column="sex" property="sex"/>
			<result column="number" property="number"/>
			<result column="enable" property="enable"/>
			<result column="city" property="city"/>
		</resultMap>

		<select id="getUserNameById" parameterType="int" resultType="String">
			select name from students where id = #{id}
		</select>

		<select id="getStudentByNumAndCity" parameterType="map" resultMap="studentMap">
			select * from students where number = #{num} and city = #{city}
		</select>

	</mapper>

> `<resultMap/>`即定义列与属性的字段映射；`<select/>`中的参数和返回值的类型，既可以为基本类型，如string，int，long，也可以是map，返回类型还可以是`<resultMap/>`定义的映射map；如果参数类型是map，则sql中的参数名（如#{num})必须是map的key；如果返回类型为map，则sql语句中返回的列名为key；如果是基本类型，使用type，如parameterType，resultType，如果是自定义map，使用parameterMap，resultMap.

### 1.5 Spring配置文件的配置

首先需要配置sqlSessionFactory：

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="feedbackDataSource"/>
        <property name="configLocation" value="classpath:mybatis.xml"/>
		<property name="mapperLocations" value="classpath:com/sohu/tv/mapper/*.xml"/>
    </bean>

> 属性`dataSource`引用JDBC数据源；属性`configLocation`指定mybatis配置文件的位置，配置文件中定义别名`<typeAliases/>`，设置`<settings/>`等。`mapperLocations`指定mapper映射文件的路径。有一点需要注意的是，要确保mapper映射文件被打包进classpath中，默认情况下，maven会忽略源文件中的资源文件，可以通过在pom文件中配置，使得资源文件被一起打包进classpath中；如在pom配置文件中添加：

    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <excludes><exclude>**/*.java</exclude></excludes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
	</build>

其次，需要定义与dao接口相关联的mapperFactoryBean：

    <bean id="mybatisDaoImpl" class="org.mybatis.spring.mapper.MapperFactoryBean">
        <property name="mapperInterface" value="com.sohu.tv.dao.MyBatisTest"/>
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>

> `mapperInterface`属性的值为相关的dao接口，`sqlSessionFactory`属性引用了上述定义的sqlSessionFacotry；

### 1.6 service类中调用dao类实现业务逻辑

在MyBatisServiceImpl.java中使用dao接口中提供的方法：

	@Resource(name = "mybatisDaoImpl")
	MyBatisDao myBatisDaoImpl;

	String userName = mybatisDaoImpl.getUserNameById(2);
	System.out.println(userName);

	Map<String, Object> queryMap = new HashMap<String, Object>();
	queryMap.put("num", 333);
	queryMap.put("city", "beijing");
	List<Students> studentsList = mybatisDaoImpl.getStudentByNumAndCity(queryMap);
	for (Students students: studentsList) {
		System.out.println(students.getName());
	}

## 2. 启动自动扫描注解

我们可以在applicationContext.xml配置文件里为每个dao接口定义bean，但mybatis还提供了一种更简便的自动扫描注解的机制，即`<mybatis:scan/>`和`<MapperScannerConfigurer/>`。
配置`<mybatis:scan/>`，需要在applicationContext.xml配置文件里添加：

	<mybatis:scan base-package="com.sohu.tv.dao"/>

> `<mybatis:scan/>`与Spring的`<context:component-scan/>`非常相似，`base-package`指定要扫描的包，并将包下的所有接口注册为对应的bean。命名规则：和Spring一样，如果该接口没有被注解，则bean的名称为首字母小写的非限定类名，如接口为`com.sohu.tv.dao.MyBatisDao`，则bean的名字为`myBatisDao`；如果dao接口使用了Spring的注解，如@Component或@Named等注解，并提供了bean的名称，则mybatis使用该注解的名称作为bean的名称。如将MyBatisDao接口重定义如下：

	@Repository(value = "mybatisDao")
	public interface MyBatisDao {

		public String getUserNameById(int id);

		public List<Students> getStudentByNumAndCity(Map<String, Object> queryMap);
	}

> 测试MyBatisDao被自动注解后的bean的名称为mybatisDao。建议通过注解指定bean的名称，防止类类名的变化导致了bean名称的变化；

配置`<MapperScannerConfigurer/>`，需要在applicationContext.xml中添加：

	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.sohu.tv.dao"/>
        <property name="sqlSessionFactoryBeanName" value = "sqlSessionFactory"/>
    </bean>

>这里的`basePackage`与`<mybatis:scan/>`的`base-package`的含义一致，bean的命名规则也是一样的，所以这两种方式等价。

> 如果启动了自动扫描注解，则在spring配置文件中不再需要dao接口的bean定义了。

## 3. 总结-最佳实践

+ mapper映射文件放在单独的目录中，统一管理，在配置`sqlSessionFactory`时，通过属性`mapperLocations`指定；
+ mybatis配置文件中只定义`typeAliases`、`settings`等配置信息；
+ Spring配置文件中，通过`<mybatis:scan/>`或者`<MapperScannerConfigurer/>`启动自动注解，并通过Spring的注解对bean命名。

### 参考资料：

+ 1. [MyBatis-Spring](http://mybatis.github.io/spring/index.html)
