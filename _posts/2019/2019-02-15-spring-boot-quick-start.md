---
layout: post
title: springboot(一)：入门篇
category: springboot
tags: [springboot]
---

## 什么是spring boot

要想使用spring boot，需要先了解什么是spring boot。

Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。

通俗的说就是，springboot将很多框架的使用方式做了默认的配置，在使用的过程中，只需要将对应的依赖引用进来，不需要我们去做额外的配置操作，从而简化了开发。



## 使用spring boot 有什么好处

  使用三个词来形容spring boot 的好处 ：简单、快速、方便。

  之前搭建一个spring web项目，我们需要做的操作有：

- 配置web.xml，加载spring和spring mvc

- 配置数据库连接、配置spring事务


- 配置加载配置文件的读取，开启注解


- 配置日志文件

- ...

- 配置完成后部署到tomcat 中调试运行

- ...

在现在微服务流行的时代，如果我们的项目仅需要实现一个很简单的功能，但是搭建一个项目需要把这整个流程都走一遍，是不是觉得会很折腾。

 那么springboot诞生后，我们搭建一个这样的项目需要做哪些操作呢？

其实很简单，我们只需要做非常少的配置就可以迅速的搭建一套web项目或者构建一个微服务。

下面我们来试一试使用springboot到底有多爽。



## spring boot 搭建web项目

我在开发中习惯使用idea，感觉比较好用，所以这里采用idea来搭建项目，当然也可以使用eclipse等工具来搭建项目。

开发工具：maven + idea



### 新建项目

- 在idea中新建项目，然后选择Spring Initializr

  - idea中内置了spring boot项目的快速建立，只需要新建一个Spring Initializr项目，然后就会自动去[https://start.spring.io/](https://start.spring.io/) 中自动下载需要的jar包到本地，springboot支持的jdk版本为1.7以上，我使用的是1.8。

![springboot-01](..\..\assets\images\2019\springboot\springboot-01.png)



- 点击next后，输入项目的基本信息。


 ![springboot-02](..\..\assets\images\2019\springboot\springboot-02.png)



- 点击next，可以选择需要导入的依赖，这里提供了很多可以导入的依赖，包括数据库，web，Security等等，当然，这些依赖也可以创建项目后，手动导入。我们这个只是简单的web项目，所以只选择的web模块。

   ![springboot-03](..\..\assets\images\2019\springboot\springboot-03.png)

  ​


- 点击next，输入项目的名称和路径。

   ![springboot-04](..\..\assets\images\2019\springboot\springboot-04.png)



- 点击Finish后，idea会自动去maven仓库中下载需要的jar包，等待导包完成后，第一个web项目就创建好了。

### 书写controller

~~~java
@RestController
public class HelloWorldController {

    @GetMapping("hello")
    public String helloWorld(){
        return "Hello World";
    }
}
~~~

```@RestController``` 的意思就是controller里面的方法都以json格式输出，不用再写什么jackjson配置的了！



### 访问程序

- 启动主程序，打开浏览器访问http://localhost:8080/hello，就可以看到效果了，有木有很简单！



### 项目结构介绍

 ![springboot-05](\assets\images\2019\springboot\springboot-05.png)

 从图中可以看出，Spring boot 的基础结构共三个文件

- src/main/java: 程序开发以及主程序的入口
- src/main/resources: 配置文件
- src/test/java: 单元测试程序



另外，spingboot建议的目录结果如下：  
root package结构：```com.example.myproject```

``` java
com
  +- example
    +- myproject
      +- Application.java
      |
      +- domain
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
      +- controller
      |  +- CustomerController.java
      |
```


- 1、Application.java 建议放到根目录下面,主要用于做一些框架配置
- 2、domain目录主要用于实体（Entity）与数据访问层（Repository）
- 3、service 层主要是业务类代码
- 4、controller 负责页面访问控制

采用默认配置可以省去很多配置，当然也可以根据自己的喜欢来进行更改

### pom.xml 文件解析

~~~xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
~~~

```spring-boot-starter-web``` : spring boot web支持模块，会自动引入web容器等。

```spring-boot-devtools``` : spring boot开发工具，在开发过程中一些很实用的工具，最常用的就是热启动。

```spring-boot-starter-test``` : 测试模块，包括JUnit、Hamcrest、Mockito。



### 单元测试

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootHelloworldApplicationTests {

	private MockMvc mvc;

	@Before
	public void setUp() throws Exception {
		mvc = MockMvcBuilders.standaloneSetup(new HelloWorldController()).build();
	}

	@Test
	public void getHello() throws Exception {
		mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
				.andExpect(status().isOk());
	}

}
~~~

spring boot 提供了自己的一种简化的单元测试的方式。

使用注解```@RunWith``` 加上 ```@SpringBootTest``` 即可完成单元测试的准备，然后书写自己的单元测试用例即可。

编写简单的http请求来测试；使用mockmvc进行，利用MockMvcResultHandlers.print()打印出执行结果。



### 总结	

使用spring boot可以非常方便、快速搭建项目，使我们不用关心框架之间的兼容性，适用版本等各种问题，我们想使用任何东西，仅仅添加一个配置就可以，所以使用sping boot非常适合构建微服务。

[示例代码](https://github.com/ldmyown/springboot-learning)



