---
layout: post
title: Spring持久化之JDBC
date: 2014-03-16 18:01:26 +0800
tag: spring
categories: Backend
---

这个小系列介绍Spring的持久化策略，主要关注当前应用最广泛的三种方式：Spring JDBC， MyBatis以及Hibernate。
使用JDBC的优劣如下：

+ 优势：相对于持久化框架，使用JDBC，不需要掌握其它框架的查询语言，允许我们在更低的层次上处理数据，能够访问数据库中单独的列，而且能够更好地对数据访问的性能进行调优。
+ 劣势：随着项目的规模和复杂度的提升，使用JDBC会非常繁琐，同时不易于处理复杂的问题。
+ Spring JDBC：提供数据访问模板，简化JDBC编程，同时提供了平台无关的持久化异常体系。

<!-- more -->

## 1. 配置数据源
在生产环境，考虑到性能，应该使用数据库连接池。以commons dbcp配置为例，配置如下：

	<context:property-placeholder location="classpath:jdbc.properties"/>

    <bean id="vrsDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${jdbc.drvier}"/>
        <property name="url" value="${jdbc.videodb.url}"/>
        <property name="username" value="${jdbc.videodb.username}"/>
        <property name="password" value="${jdbc.videodb.password}"/>
        <property name="maxActive" value="${jdbc.videodb.maxActive}"/>
        <property name="initialSize" value="${jdbc.videodb.initialSize}"/>
    </bean>

> 主要属性参数说明：

+ driverClassName: JDBC驱动的全限定类名，如mysql就是：com.mysql.jdbc.Driver；
+ url：JDBC的url，如使用mysql的url：jdbc:mysql://10.11.132.193:3306/vrs；
+ username, password: 连接该数据源的用户名和密码；
+ initialSize：表示初始大小，即连接池启动时创建的连接数量；
+ maxActive：表示同一时间可从池中分配的最大连接数，0表示无限制；
+ maxIdle：池里不会被释放的最大连接数，0表示无限制；
+ minIdle：在不创建新连接的情况下，池中保持空闲的最小连接数；
+ maxWait：没有可用连接时，在抛出异常之前，池等待连接回收的最大时间；-1表示无线等待；
+ validationQuery：验证连接的sql查询，至少返回一行；
更多属性参考[wiki页](http://commons.apache.org/proper/commons-dbcp/configuration.html)

> 其它推荐的连接池有：

+ [c3p0](http://www.mchange.com/projects/c3p0/): 适合多线程环境；
+ [druid](https://github.com/alibaba/druid)：完善的监控功能；

## 2. 使用JdbcTemplate

使用JdbcTemplate类，在sql语句中，以?作为参数的占位符，传入的参数的顺序与sql语句中?的顺序必须是一一对应的。

首先在xml配置文件里添加jdbcTemplate的bean，其参数`dataSource`引用之前定义的数据源：

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="vrsDataSource"/>
    </bean>

然后，在dao类中引用JdbcTemplate类，并注入：

	@Resource(name = "jdbcTemplate")
	JdbcTemplate jdbcTemplate;

查询
返回基本类型：

	String getUserById = "select name from students where id = ?";
	String userName = jdbcTemplate.queryForObject(getUserById, new Object[]{3}, String.class);

返回`Map<String, Object>`，key为列名，value为对应列的值，此时返回值只能有一行，否则报错：

	String getStudentByNumber = "select * from students where number = ?";
	Map<String, Object> studentMap = jdbcTemplate.queryForMap(getStudentByNumber, 111);

返回`List<Map<String, Object>>`, 可以返回多列：

	String getStudentsByCity = "select * from students where city = ?";
	List<Map<String, Object>> studentList = jdbcTemplate.queryForList(getStudentsByCity, "tianjin");

返回自定义class的对象，需要实现RowMapper接口，定义列名和属性的映射：
首先实现RowMapper接口：

	public class StudentRowMapper implements RowMapper<Students> {
		@Override
		public Students mapRow(ResultSet rs, int rowNum) throws SQLException {
			Students students = new Students();
			students.setId(rs.getInt("id"));
			students.setName(rs.getString("name"));
			students.setSex(rs.getString("sex"));
			students.setNumber(rs.getInt("number"));
			students.setEnable(rs.getInt("enable"));
			students.setCity(rs.getString("city"));
			return students;
		}
	}

然后，使用query或者queryForObject查询多行或一行：

	String getStudentsByCity = "select * from students where city = ?";
	List<Students> studentsList = jdbcTemplate.query(getStudentsByCity, new StudentRowMapper(), "tianjin");

## 2. 使用NamedParameterJdbcTemplate

使用NamedParameterJdbcTemplate类，sql语句中的参数是以命名的变量来表示，传入参数时，只要参数名一致即可，索引位置不必一一对应。
首先在xml配置文件里定义NamedParameterJdbcTemplate的bean：

	<bean id="namedParameterJdbcTemplate" class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
        <constructor-arg name="dataSource" ref="feedbackDataSource"/>
    </bean>

然后将namedParameterJdbcTemplate bean注入到dao中，

	@Resource(name = "namedParameterJdbcTemplate")
	NamedParameterJdbcTemplate namedParameterJdbcTemplate;

定义sql，参数以(:变量名)形式给出即可，如：

	String getStudentByNumber = "select * from students where number = (:number) and city = (:city)";
	NamedParameterJdbcTemplate namedParameterJdbcTemplate = (NamedParameterJdbcTemplate) context.getBean("namedParameterJdbcTemplate");
	Map<String, Object> queryMap = new HashMap<String, Object>();
	queryMap.put("number", 333);
	queryMap.put("city", "tianjin");
	List<Map<String, Object>> mapList = namedParameterJdbcTemplate.queryForList(getStudentByNumber, queryMap);

> 除了使用命名参数外，NamedParameterJdbcTemplate与JdbcTempate的主要用法都是一致的。

## 3. 使用JdbcDaoSupport

JdbcDaoSupport是一个父类，如果有多个dao类，通过继承JdbcDaoSupport，可以更方便地获取jdbcTemplate。
首先让dao类继承JdbcDaoSupport类：

	public class InfoJdbcImpl extends JdbcDaoSupport implements VideoInfoDao {

然后在定义dao类的bean时，注入一个`jdbcTemplate`属性，或者直接注入一个`dataSource`属性（这两个属性来自于JdbcDaoSupport）：

    <bean id="infoJdbcImpl" class="com.sohu.tv.dao.impl.InfoJdbcImpl">
        <property name="dataSource" ref="feedbackDataSource"/>
    </bean>

然后通过getJdbcTemplate()获取jdbcTemplate，实现dao层的逻辑：

	String getStudentByNumber = "select * from students where number = ?";
	Map<String, Object> studentMap = getJdbcTemplate().queryForMap(getStudentByNumber, 111);

> `NamedParameterJdbcDaoSupport`与`JdbcDaoSupport`的用法类似。

### 参考资料

+ [Spring实战(第3版)](http://www.amazon.cn/Spring%E5%AE%9E%E6%88%98-%E6%B2%83%E5%B0%94%E6%96%AF/dp/B00CY6UD2I/ref=sr_1_1?ie=UTF8&qid=1394943496&sr=8-1&keywords=spring+in+action)
+ [spring-javadoc-api](http://docs.spring.io/spring/docs/3.2.8.RELEASE/javadoc-api/)
