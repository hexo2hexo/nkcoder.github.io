---
layout: post
title: Spring依赖注入之Java注解
date: 2014-03-10 23:29:44 +0800
tags: spring
categories: Backend
---

> 基于Java的注解，是在类级别上定义beans，在方法级别上定义bean。和自动注解一样，可以减少xml的配置。

## 1. xml配置

在xml配置文件中添加自动扫描的配置：

	<context:component-scan base-package="org.yousharp"/>

>该配置不仅扫描构造型注解(@Controller, @Service, @Repository, @Component)，也会扫描@Configuration注解。

## 2. 定义dao类和service类

dao类是service类的一个属性，是需要注入的对象。

	public class VideoService {
		SaveVideoInfoDao saveVideoInfoDao;

		public VideoService(){}

		public VideoService(SaveVideoInfoDao saveVideoInfoDao) {
			this.saveVideoInfoDao = saveVideoInfoDao;
		}

		public void saveVideoInfo(String info) {
			saveVideoInfoDao.printMesg(info);
		}
	}

	public class SaveVideoInfoDao {

		public void printMesg(String message) {
			System.out.println("saving video info....");
		}
	}

> dao类和service类就是两个很普通的java类，没有任何注解依附。

## 3. 定义一个配置类

单独定义一个配置类（在自动扫描包下），在类上使用@Configuration注解，在方法上使用@Bean注解：

	@Configuration
	public class AppConfig {

		@Bean
		public SaveVideoInfoDao saveVideoInfoDao() {
			return new SaveVideoInfoDao();
		}

		@Bean
		public VideoService videoService() {
			VideoService videoService = new VideoService(saveVideoInfoDao());
			return videoService;
		}
	}

> @Configuration注解等价于xml配置中的`beans`，该注解告诉Spring该类中包含一个或多个bean的定义；@Bean注解等价于xml配置中的`bean`，将方法返回的对象定义为bean，方法名为bean的id；上面第一个bean的定义等价于xml配置：

	<bean id="saveInfoDao" class="org.yousharp.AppConfig"/>

第二个bean的定义通过构造函数实现依赖注入，其含义等价于xml配置：

	<bean id="videoService" class="org.yousharp.AppConfig">
		<constructor-arg name="videoService" ref="saveVideoInfoDao">
	</bean>

这里是通过构造函数实现注入，当然也可以通过setter方法实现注入。在VideoInfoService中去掉带参数的构造函数，同时对属性saveVideoInfoDao添加setter方法，修改AppConfig类中的第二个bean为：

	@Bean
	public VideoService videoService() {
		VideoService videoService = new VideoService();
		videoService.setSaveVideoInfoDao(saveVideoInfoDao());
		return videoService;
	}

## 4. Controller中引用

Controller里和使用xml配置bean和自动注解一样，通过bean的id获取对bean的引用。这里VideoService的bean的id为videoService。

	ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

	public void execute() {
		VideoService videoService = (VideoService) context.getBean("videoService");
		String videoInfo = "vrs video info";
		videoService.saveVideoInfo(videoInfo);
	}

### 参考资料

+ [Spring实战(第3版)](http://www.amazon.cn/Spring%E5%AE%9E%E6%88%98-%E6%B2%83%E5%B0%94%E6%96%AF/dp/B00CY6UD2I/ref=sr_1_1?ie=UTF8&qid=1394943496&sr=8-1&keywords=spring+in+action)
