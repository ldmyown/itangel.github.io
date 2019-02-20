---
layout: post
title: springboot(三)：Spring boot中task的使用
category: springboot
tags: [springboot]
---

本文介绍一下springboot中定时任务的使用方法

## 说明

在项目开发过程中，我们经常会用到定时任务，在spring中，可以直接使用task来创建定时任务，那么在springboot中如何使用定时任务呢？

## springboot 创建定时任务

### 1.创建一个简单的springboot项目

对应pom文件为

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.telangel</groupId>
	<artifactId>springboot-task</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>springboot-task</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

~~~



### 2. 使用注解的方式开始任务调度功能

在启动类上添加```@EnableScheduling```注解，开启spring的定时任务功能

~~~java
@SpringBootApplication
@EnableScheduling
public class SpringbootTaskApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootTaskApplication.class, args);
	}

}

~~~



### 3. 配置定时任务

创建一个类来配置定时任务，此类使用```@Component```注解让其能够被spring管理

~~~java
@Component
public class MyTask {
    private static final Logger LOGGER = LoggerFactory.getLogger(MyTask.class);

    private static final long SECOND = 1000;

    @Autowired
    UserService userService;

    /**
     * 固定频率执行任务
     * 固定间隔3秒，可以引用变量
     * fixedRate：以每次开始时间作为测量，间隔固定时间
     */
    @Scheduled(fixedRate = 3 * SECOND)
    public void task1() {
        LOGGER.info("当前时间：{}\t\t任务：fixedRate task，每3秒执行一次", System.currentTimeMillis());
        userService.test();
    }

    /**
     *  固定延迟3秒，从前一次任务结束开始计算，延迟3秒执行
     */
    @Scheduled(fixedDelay = 3 * SECOND)
    public void task2(){
        LOGGER.info("当前时间：{}\t\t任务：fixedDelay task，每延迟3秒执行一次", System.currentTimeMillis());
    }

    /**
     * cron表达式，每5秒执行
     */
    @Scheduled(cron = "*/5 * * * * ?")
    public void task3() {
        LOGGER.info("当前时间：{}\t\t任务：cron task，每5秒执行一次", System.currentTimeMillis());
    }

}
~~~



### 4. 测试

~~~
2019-02-20 10:46:36.025  INFO 11704 --- [           main] c.t.s.SpringbootTaskApplication          : Started SpringbootTaskApplication in 1.518 seconds (JVM running for 3.304)
2019-02-20 10:46:39.001  INFO 11704 --- [   scheduling-1] com.telangel.springboottask.task.MyTask  : 当前时间：1550630799001		任务：fixedRate task，每3秒执行一次
userservice中的test方法
2019-02-20 10:46:39.003  INFO 11704 --- [   scheduling-1] com.telangel.springboottask.task.MyTask  : 当前时间：1550630799003		任务：fixedDelay task，每延迟3秒执行一次
2019-02-20 10:46:40.001  INFO 11704 --- [   scheduling-1] com.telangel.springboottask.task.MyTask  : 当前时间：1550630800001		任务：cron task，每5秒执行一次
2019-02-20 10:46:42.001  INFO 11704 --- [   scheduling-1] com.telangel.springboottask.task.MyTask  : 当前时间：1550630802001		任务：fixedRate task，每3秒执行一次
~~~



### 5. 总结一个小坑

- **默认这个schedule只使用一个线程。**如果有多个函数上使用了```@Scheduled```，那么只有当前一个执行完毕后，才能执行下一个任务，这种往往不是我们需要的效果。

- 那么要怎么样才能并发执行呢，这里我们可以创建一个线程池，然后让定时任务在线程池中执行，互相不受影响

### 6 多线程执行任务 

在SpringBoot项目中一般使用config配置类的方式添加配置

~~~java
@Configuration
@EnableAsync
public class SchedulingConfiguration {
    //线程池配置参数（可通过配置文件读取）
    private int corePoolSize = 10;
    private int maxPoolSize = 200;
    private int queueCapacity = 10;
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(corePoolSize);
        executor.setMaxPoolSize(maxPoolSize);
        executor.setQueueCapacity(queueCapacity);
        executor.initialize();
        return executor;
    }

}
~~~



@Configuration：表明该类是一个配置类 
@EnableAsync：开启异步事件的支持

然后在定时任务的类或者方法上添加@Async 便支持每一个任务都是在不同的线程中

这样就可以完成不同的任务在不同的线程中执行了



**[示例代码-github](**https://github.com/ldmyown/springboot-learning**)**  



-------------

**作者：telAngel**  
**出处：[https://ldmyown.github.io](https://ldmyown.github.io)**      
**版权所有，欢迎保留原文链接进行转载：)**