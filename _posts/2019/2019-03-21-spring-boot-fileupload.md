---
layout: post
title: springboot(六)：Spring boot实现基于web文件上传下载功能
category: springboot
tags: [springboot]
---



## 说明

​	最近部门的实施人员需要一个基于web的传输大文件工具，于是寻找了一下资料，集成springboot书写了一个文件上传的小工具。

​	web中上传大文件不像桌面软件那么容易，在web端上传文件常用的做法就是表单上传，但是当文件较大时，如果网络速度比较慢，那么就只能等着文件上传完成，期间不能做刷新页面的操作，并且如果出现网络波动，那么就会上传失败，前面上传的操作就白费了，需要重新上传。

​	该项目是基于百度开源的一款前端上传控件WebUploader来实现的，WebUploader是由Baidu WebFE(FEX)团队开发的一个简单的以HTML5为主，FLASH为辅的现代文件上传组件。

​	对于大文件的上传，我们需要实现哪些必须要的功能呢？

- **断点续传** 文件上传最主要的功能，在断网或者暂停的情况下， 能够接着上次传输的进度继续上传
- **分块上传** 通过前端将文件分块，后台将分块的文件组装起来成为新的文件，这是实现断点续传的基础
- **文件秒传** 如果服务端已经存在该文件，那么再次上传此文件的时候，就能实现秒传到服务端
- **基于WebUploader实现的功能** 这些功能是通过[WebUploader](http://fex.baidu.com/webuploader) 来实现的，使用方法很简单
  - **多线程上传** 实现多个线程同时上传不同块的文件
  - **文件进度显示** 显示文件上传的进度情况
  - **多文件同时上传** 可以选择多个文件同时上传

  ​

## 项目搭建

### 创建一个简单的springboot项目

在pom文件中引入文件上传的基础依赖包

~~~xml
	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- 公共工具包 -->
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
		</dependency>
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
		</dependency>
~~~






**[示例代码-github](**https://github.com/ldmyown/springboot-learning**)**

-------------

**作者：telAngel**  
**出处：[https://ldmyown.github.io](https://ldmyown.github.io)**      
**版权所有，欢迎保留原文链接进行转载：)**