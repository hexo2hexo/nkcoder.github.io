---
layout: post
title: Spring依赖注入之注解注入
date: 2014-03-09 18:19:11 +0800
tags: spring
categories: Backend
---

## 1. bean的自动装配和自动检测的区别

启用注解装配，在xml配置文件里添加：

	<context:annotation-config/>

启用自动扫描注解，在xml配置文件里添加：

	<context:component-scan base-package="org.yousharp.base"/>

> 二者的区别：
自动装配表示通过@Autowired, @Inject, @Resource等实现对属性或构造函数的自动注入；仍然需要在配置文件里定义bean，只是通过自动装配省去了`property`和`constructor-arg`的配置。
自动检测是扫描特定的注解（包括：@Component, @Controller, @Service, @Repository)，将注解过的类自动定义为bean，自动检测是自动装配的超集，通过自动检测，可以省去在xml配置文件里定义`bean`了。

## 2. 通过自动装配和注解实现注入

在xml配置文件里启动注解装配：

	<context:annotation-config/>

定义bean：

	<bean id="saveVideoInfoDao" class="org.dao.SaveVideoInfoDao"/>

然后在需要注入的属性上或者其setter方法上，或者构造函数上，添加@Autowired, @Inject, @Resource等注解实现自动注入：

	//@Autowired
	//@Inject
	@Resource
	public void setSaveVideoInfoDao(SaveVideoInfoDao videoInfoDao) {
		this.saveVideoInfoDao = videoInfoDao;
	}

> @Autowired, @Inject, @Resource的区别：@Autowired是Spring 3.0的注解，是通过byType形式实现注解，可以通过@Qualifier根据bean的id进行限定；使用@Autowired注解即引入了对Spring的依赖。@Inject是JSR 330的注解，使用该注解需要导入包javax.inject，@Named(value="")可以根据bean的id进行限定；@Resource是JSR 250的注解，可以通过value限定bean的id，如@Resource(value="");

## 3. 使用自动检测注解实现注入

在xml配置文件里增加自动检测的配置：

	<context:component-scan base-package="org.yousharp.base"/>

将需要被自动检测而注册为bean的类使用对应的构造型注解：

	@Repository
	public class SaveVideoInfoDao {

		public void printMesg(String message) {
			System.out.println("saving video info....");
		}
	}

使用注解对依赖的属性进行输入：

	@Service
	public class VideoService {
		@Resource
		SaveVideoInfoDao saveVideoInfoDao;

		public void saveVideoInfo(String info) {
			saveVideoInfoDao.printMesg(info);
		}
	}

> @Component是通用的注解，@Controller表示将该类定义为Spring MVC的控制器，@Service定义服务层，@Repository定义数据仓库；这些构造型注解默认以无限定类名作为bean的id，也可以显式指定id名称，如@Service("videoInfoService")或者@Service(name="videoInfoService");

## 4. 自动检测注解的过滤

可以为扫描行为定义过滤器，如`context:include-filter`, `context:exclude-filter`

    <context:component-scan base-package="org.yousharp">
        <context:include-filter type="assignable" expression="org.yousharp.dao.SaveVideoInfoDao"/>
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Component"/>
    </context:component-scan>

> `context:include-filter`表示需要扫描并注解的类，`context:exclude-filter`表示扫描时需要排除的包；`type`一种有5中，`assignable`表示继承自`expression`所指定的包，`annotation`表示所有expression所指定的注解。

### 参考资料

+ [Spring实战(第3版)](http://www.amazon.cn/Spring%E5%AE%9E%E6%88%98-%E6%B2%83%E5%B0%94%E6%96%AF/dp/B00CY6UD2I/ref=sr_1_1?ie=UTF8&qid=1394943496&sr=8-1&keywords=spring+in+action)
