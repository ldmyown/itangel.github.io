---
layout: post
title: springboot(七)：Spring boot中集成protobuf
category: springboot
tags: [springboot]
---



protobuf 是一种数据格式，只不过在序列化后的大小、序列化、反序列化方面更优秀一些。

由于它是一种二进制的格式，比使用 xml进行数据交换快许多。可以把它用于分布式应用之间的数据通信或者异构环境下的数据交换。作为一种效率和兼容性都很优秀的二进制数据传输格式，可以用于诸如网络传输、配置文件、数据存储等诸多领域。

这里总结一下如何实现springboot集成protobuf数据格式。

## 项目搭建

### 创建一个简单的springboot项目

在pom文件中引入邮件**protobuf**的依赖

~~~xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<!-- protobuf依赖-->
		<dependency>
			<groupId>com.google.protobuf</groupId>
			<artifactId>protobuf-java</artifactId>
			<version>3.6.0</version>
		</dependency>
		<dependency>
			<groupId>com.googlecode.protobuf-java-format</groupId>
			<artifactId>protobuf-java-format</artifactId>
			<version>1.2</version>
		</dependency>

		<!-- 网络请求依赖 -->
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
			<version>4.5.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpcore</artifactId>
			<version>4.4</version>
		</dependency>
~~~

### 配置protobuf

配置protobuf的序列化与反序列化bean

~~~java
@Configuration
public class ProtobufConfig {
    /**
     * protobuf 序列化
     */
    @Bean
    ProtobufHttpMessageConverter protobufHttpMessageConverter() {
        return new ProtobufHttpMessageConverter();
    }

    /**
     * protobuf 反序列化
     */
    @Bean
    RestTemplate restTemplate(ProtobufHttpMessageConverter protobufHttpMessageConverter) {
        return new RestTemplate(Collections.singletonList(protobufHttpMessageConverter));
    }
}
~~~



### 定义protobuf

​	首先我们需要建立一个**proto**文件，在该文件定义我们需要传输的文件。例如我们需要定义一个用户的信息，包含的字段主要有姓名、年龄，部门，工作。那么该**protobuf**文件的格式如下:

```java
syntax = "proto3";
// 生成的包名
option java_package = "com.telangel.protobuf";
//生成的java名
option java_outer_classname = "User";

message UserRequest {
    string username = 1;
}

message UserResponse {
    string username = 1;
    int64 age = 2;
    string dept = 3;
    string job = 4;
}
```

### 生成java文件

​	创建好proto文件之后，通过生成java文件的软件**protoc.exe**生成java文件，具体命令为```protoc.exe --java_out=输出路径 proto文件路径```

例如:

~~~
protoc.exe --java_out=D:\protobuf User.proto
~~~

生成后，我们将java文件放到项目的指定路径下就可以使用了。

​	如果有很多proto文件需要批量生成Java文件，可以通过脚本生成，将所有的protobuf文件都放在一个文件夹下，循环遍历文件将所有的protobuf生成Java文件

```java
public static void main(String[] args) {
    String protoFile = "user.proto";
    File file1 = new File("./src/main/resources/protobuf/protoc.exe");
    File file2 = new File("./src/main/resources/protobuf"+ protoFile);
    file1.exists();
    file2.exists();
    String strCmd = "./src/main/resources/protobuf/protoc.exe --java_out=./src/main/java ./src/main/resources/protobuf/"+ protoFile;
    try {
        Runtime.getRuntime().exec(strCmd);
    } catch (IOException e) {
        e.printStackTrace();
    }//通过执行cmd命令调用protoc.exe程序
}
```

### 书写测试接口

~~~java
@RestController
public class UserController {

    @Autowired
    UserService userService;

    @PostMapping(value = "getUser", produces = "application/x-protobuf")
    public User.UserResponse getUser(@RequestBody User.UserRequest request) {
        String username = request.getUsername();
        User.UserResponse userResponse = userService.findUserByName(username);
        return userResponse;
    }
}
~~~

### 书写测试用例测试

~~~java
	@Test
	public void contextLoads() {
		try {
			URI uri = new URI("http", null, "127.0.0.1", 8080, "/getUser", "", null);
			HttpPost request = new HttpPost(uri);
			User.UserRequest.Builder builder = User.UserRequest.newBuilder();
			builder.setUsername("aa");
			HttpResponse response = HttpUtils.doPost(request, builder.build());
			User.UserResponse userResponse = 				User.UserResponse.parseFrom(response.getEntity().getContent());
			System.out.println(userResponse.getDept());
			System.out.println(userResponse.toString());
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}
~~~

### protobuf序列化与反序列化

序列化与反系列化的代码示例

~~~java
	@Test
	public void test() throws IOException {
		// 按照定义的数据结构，创建一个对象
		User.UserResponse.Builder userInfo = User.UserResponse.newBuilder();
		userInfo.setUsername("张三");
		userInfo.setAge(10);
		userInfo.setDept("人事部");
		userInfo.setJob("招聘");

		// 将数据写到输出流
		ByteArrayOutputStream output = new ByteArrayOutputStream();
		User.UserResponse userResponse = userInfo.build();
		userResponse.writeTo(output);
		// 将数据序列化后发送
		byte[] bytes = output.toByteArray();
		// 接收到流并读取
		ByteArrayInputStream input = new ByteArrayInputStream(bytes);
		// 反序列化  
		User.UserResponse userResponse2 = User.UserResponse.parseFrom(input);

		System.out.println("name:" + userResponse2.getUsername());
		System.out.println("age:" + userResponse2.getAge());
		System.out.println("dept:" + userResponse2.getDept());
		System.out.println("job:" + userResponse2.getJob());
	}
~~~






**[示例代码-github](**https://github.com/ldmyown/springboot-learning**)**

-------------

**作者：telAngel**  
**出处：[https://ldmyown.github.io](https://ldmyown.github.io)**      
**版权所有，欢迎保留原文链接进行转载：)**