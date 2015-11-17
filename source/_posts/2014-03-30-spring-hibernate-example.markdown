---
layout: post
title: Spring持久化之Hibernate
date: 2014-03-30 18:37:27 +0800
tags:
  - spring
  - hibernate
categories: Backend
---

本文简要介绍Spring通过Hibernate实现持久化需要的基本配置，分别通过注解和xml配置来实现。

## 1. 通过注解实现

### 1.1 添加pom依赖

在pom的xml里添加*Hibernate*相关的依赖，如：

	<dependency>
		<groupId>org.hibernate</groupId>
		<artifactId>hibernate-core</artifactId>
		<version>3.6.10.Final</version>
	</dependency>

	<dependency>
		<groupId>javax.persistence</groupId>
		<artifactId>persistence-api</artifactId>
		<version>${javax.persist}</version>
	</dependency>

	<dependency>
		<groupId>javassist</groupId>
		<artifactId>javassist</artifactId>
		<version>3.12.1.GA</version>
	</dependency>

### 1.2 实体类的注解定义

先定义一个实体类StudentEntity.java，映射到数据库中的表students：

	@Entity
	@Table(name = "students")
	public class StudentEntity {
		@Id
		@Column(name = "id", nullable = false)
		@GeneratedValue
		private int id;

		@Column(name = "name", nullable = false)
		private String name;

		@Column(name = "sex", nullable = true)
		private String sex;

		public int getId() {
			return id;
		}

		public void setId(int id) {
			this.id = id;
		}

		public String getName() {
			return name;
		}

		public void setName(String name) {
			this.name = name;
		}

		public String getSex() {
			return sex;
		}

		public void setSex(String sex) {
			this.sex = sex;
		}

		public StudentEntity() {}

		public StudentEntity(String name, String sex) {
			this.name = name;
			this.sex = sex;
		}
	}

> **@Entity**表示这是一个实体类，**@Table(name = "students")**表示映射到数据源指定的数据库中的一个名为*students*的表；**@Id**表示该字段是主键，**@Column(name = "id", nullable = false)**表示该属性映射到表中名为*id*的属性，且不能为空；**@GeneratedValue**表示该字段是自增的；这样定义之后，该实体类便与表*students*完成了一一对应。

### 1.3 定义dao实现类

我们的dao实现类为HibernateDaoImpl.java，依赖一个属性，即*SessionFactory*的实例，我们通过注解实现注入：

	@Repository (value = "hibernateDaoImpl")
	public class HibernateDaoImpl implements StudentDao {

		@Resource(name = "sessionFactory")
		SessionFactory sessionFactory;

		@Override
		@Transactional
		public StudentEntity getUserNameById(int id) {
			StudentEntity studentEntity = (StudentEntity) sessionFactory.getCurrentSession().get(StudentEntity.class, id);
			return studentEntity;
		}

		@Override
		@Transactional
		public void saveStudents(StudentEntity studentEntity) {
			sessionFactory.getCurrentSession().persist(studentEntity);
		}
	}

> **Repository**注解表示该类会被Spring的注解扫描器自动定义为bean，bean的名字为*hibernateDaoImpl*，同时实现异常转换，将Hibernate的检查型异常，转换为Spring的非检查型异常，这是通过在Spring配置文件里配置**PersistenceExceptionTranslationPostProcessor**的ban实现的。通过*sessionFactory*的*getCurrentSession*方法获取当前的*session*，实现对数据库的持久化操作。Hibernate还要求这些操作是与事务绑定的，因此需要在dao层或者service层添加事务的注解，即*@Transactional*。

### 1.4 添加Spring配置

在Spring配置文件applicationContext.xml主要是添加**SessionFactory**的配置，以及配置事务，与*Hibernate*相关的配置如下：

    <bean name="sessionFactory" class="org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean">
        <property name="dataSource" ref="feedbackDataSource"/>
        <property name="packagesToScan" value="com.sohu.tv.bean"/>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
                <prop key="hibernate.show_sql">true</prop>
            </props>
        </property>
    </bean>

    <bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>

    <bean id="tx" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

    <tx:annotation-driven transaction-manager="tx"/>

> 在定义*sessionFactory*的bean时，因为是基于注解的，所以引用的class为：**AnnotationSessionFactoryBean**；属性*dataSource*即配置的数据源；属性*packagesToScan*即要扫描的包名，在这个包下的所有类，如果使用了*@Entity*、*@Column*等注解，都会被扫描器处理；*@hibernateProperties*属性定义了Hibernate相关的属性，如使用的dialect，是否显示输出sql语句等。**PersistenceExceptionTranslationPostProcessor**定义的bean是进行异常转换的；*HibernateTransactionManager*用于定义事务；*<tx:annotation-driven />*类似于*<context:component-scan />*，扫描事务类型的注解。

## 2. 通过xml映射文件实现

通过xml映射文件与通过注解实现的区别就在于实体类与数据库之间的映射的定义，所以其它相同的部分就不重复了。

### 2.1 定义映射文件

这个映射文件定义实体类与数据库的表之间的映射，即属性与字段的映射，如hibernate.xml文件内容可以如下定义：

	<?xml version="1.0"?>
	<!DOCTYPE hibernate-mapping PUBLIC
			"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
			"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
	<hibernate-mapping>
		<class name="com.sohu.tv.bean.Students" table="students">

			<id name="id" type="int">
				<column name="id" />
				<generator class="identity" />
			</id>
			<property name="name" type="string">
				<column name="name" length="255" not-null="true" />
			</property>
			<property name="sex" type="string">
				<column name="sex" not-null="false" />
			</property>
		</class>
	</hibernate-mapping>

### 2.2 修改Spring配置文件

需要修改的是*sessionFactory*的bean的定义，如：

    <bean name="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource" ref="feedbackDataSource"/>
        <property name="mappingLocations">
            <list>
                <value>classpath*:hibernate.xml</value>
            </list>
        </property>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
                <prop key="hibernate.show_sql">true</prop>
            </props>
        </property>
    </bean>

> 将**AnnotationSessionFactoryBean**修改为**LocalSessionFactoryBean**，同时，需要将属性*packagesToScan*修改为*mappingLocations*，即xml映射文件的位置。

### 参考：

+ [Spring 实战](http://book.douban.com/subject/24714203/)
+ [Struts + Spring + Hibernate Integration Example](http://www.mkyong.com/struts/struts-spring-hibernate-integration-example/)
