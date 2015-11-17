---
layout: post
title: Spring AOP之入门实例
date: 2014-03-13 22:58:06 +0800
tag: spring
categories: Backend
---

依赖注入（DI）有助于应用对象之间的解耦，而面向切面编程（AOP）有助于横切关注点与所影响的对象之间的解耦。所谓横切关注点，即影响应用多处的功能，这些功能各个应用模块都需要，但又不是其主要关注点，常见的横切关注点有日志、事务和安全等。

将横切关注点抽离形成独立的类，即形成了切面。切面主要由切点和通知构成，通知定义了切面是什么，以及何时执行何种操作；切点定义了在何处执行通知定义的操作。

下面以简单的日志作为切面，分别介绍通过xml配置和注解实现AOP。

## 0. 添加依赖

因为Spring AOP用到AspectJ的相关对象和注解，需要添加AspectJ的依赖：

	<dependency>
		<groupId>org.aspectj</groupId>
		<artifactId>aspectjrt</artifactId>
		<version>1.7.4</version>
	</dependency>
	<dependency>
		<groupId>org.aspectj</groupId>
		<artifactId>aspectjweaver</artifactId>
		<version>1.7.4</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-aop</artifactId>
		<version>${spring.version}</version>
	</dependency>

## 1. 通过XML配置实现

1.1 将日志类作为切面，定义为bean即可：

	@Service("logService")
	public class LogService {
		private Logger logger = LoggerFactory.getLogger(LogService.class);

		public void printLog(ProceedingJoinPoint joinPoint) throws Throwable{
			logger.info("before service is called...");

			String methodModifier = joinPoint.getSignature().getDeclaringTypeName();
			String methodName = joinPoint.getSignature().getName();
			Object[] argsArr = joinPoint.getArgs();
			logger.info("class: {}; method: {}; args: {}", methodModifier, methodName, Arrays.toString(argsArr));
			joinPoint.proceed();

			logger.info("after service is called....");
		}
	}

> 首先，通过@Service注解将类定义为bean；方法printLog的参数`ProceedingJoinPoint`对象属于`AspectJ`，通过这个对象可以获取被通知的对象和方法的信息，并通过`proceed`方法执行被通知的对象。

1.2 在xml配置文件里定义切面和切点

    <aop:config>
        <aop:aspect ref="logService">
            <aop:pointcut id="serviceLog" expression="execution(* org.yousharp.service.VideoService..*(..))"/>
            <aop:around method="printLog" pointcut-ref="serviceLog"/>
        </aop:aspect>
    </aop:config>

> `aop:aspect`通过ref引用`logService` bean，将`LogService`定义为切面。`aop:pointcut`定义一个切点，被通知的对象通过`execution`表达式指定，即当`VideoService`下的任意方法被执行时，触发切面。`aop:aroud`定义一个环绕通知，引用切点*serviceLog`，当切面被触发时，调用`method`指定的方法。

## 2. 通过注解实现

通过注解，可以直接在类上定义切面、通知和切点，无需任何xml配置。

	@Service("logService")
	@Aspect
	public class LogService {
		private Logger logger = LoggerFactory.getLogger(LogService.class);

		@Pointcut(value = "execution(* org.yousharp.service.VideoService.saveVideoInfo(..))")
		public void saveVideo() { }

		@Around(value = "saveVideo()")
		public void printLog(ProceedingJoinPoint joinPoint) throws Throwable{
			logger.info("before service is called...");
			String methodModifier = joinPoint.getSignature().getDeclaringTypeName();
			String methodName = joinPoint.getSignature().getName();
			Object[] argsArr = joinPoint.getArgs();
			logger.info("class: {}; method: {}; args: {}", methodModifier, methodName, Arrays.toString(argsArr));
			joinPoint.proceed();

			logger.info("after service is called....");
		}
	}

> `@Aspect`注解表示将该类定义为切面，`@Pointcut`定义切点，execution表达式与xml配置一样；切点的id即为空方法的方法名（这里的方法内容不重要，主要是供切点依附），这里即为saveVideo；`@Around`表示定义环绕通知，需要引用切点。

### 参考资料

+ [Spring实战(第3版)](http://www.amazon.cn/Spring%E5%AE%9E%E6%88%98-%E6%B2%83%E5%B0%94%E6%96%AF/dp/B00CY6UD2I/ref=sr_1_1?ie=UTF8&qid=1394943496&sr=8-1&keywords=spring+in+action)
