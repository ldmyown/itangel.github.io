---
layout: post
title: springboot(四)：Spring boot中mybatis的多数据源配置
category: springboot
tags: [springboot]
---

# 说明

​	在一些复杂的项目中，为了提高服务器数据库的并发性能，数据库会使用主从模式，数据的读取在从库中操作，数据的增删改在主库中操作，由于数据库设置了主从备份，所有主库的数据变更会实时的同步到从库中，所以能够保证数据的一致性，这样就减轻了单个数据库的压力，从而能够提升服务器的整体性能。也有一些项目，由于业务逻辑很复杂，需要使用多个库来支持业务模型，因此多数据源的使用就显得很重要了。

​	那么在springboot中如何去使用多数据源呢，下面提供一个简单的实现多数据源配置的方法。



# springboot 使用mybatis配置多数据源

## 1. 在配置文件中配置多数据源

在`application.properties` 添加相关配置

 ```
spring.datasource.url=jdbc:mysql://localhost:3306/test1?useUnicode=true&characterEncoding=utf-8
spring.datasource.username=root
spring.datasource.password=root123
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
mybatis.type-aliases-package=com.telangel.entity
 ```

配置完spring.datasource 相关配置后，在系统系统时，数据源会自动注入到sqlSessionFactory中，sqlSessionFactory会自动注入到mapper中，所有的配置操作都自动帮我们完成了，我们只需要直接使用就可以了。

### 添加mapper包扫描

  mapper包扫描有两种方式添加

- 1. 在启动类中添加对mapper包扫描`@MapperScan`注解，建议采用这一种，只需要配置一次就可以了。

     ~~~java
        @SpringBootApplication
        @MapperScan("com.telangel.springbootmybatis.mapper")
        public class SpringbootMybatisApplication {

        	public static void main(String[] args) {
        		SpringApplication.run(SpringbootMybatisApplication.class, args);
        	}

        }
     ~~~

- ​ 2. 直接在Mapper类上面添加注解`@Mapper`,这种方式，在每一个mapper上都要加上这个注解，相对来说比较麻烦。

  ~~~java
      @Mapper
      public interface UserMapper {
          
          User getUserById(Integer id);
      }
  ~~~

      ​

### 开发mapper
这一步是最重要的一步，所有的curd操作都放在这里

    ​~~~java
    public interface UserMapper {
    
        @Select("select * from t_user where user_id = #{id}")
        @Results({
                @Result(property = "userName",  column = "user_name"),
                @Result(property = "userPassword", column = "user_password"),
                @Result(property = "userId", column = "user_id")
        })
        User getUserById(Long id);
    
        @Insert("insert into t_user(user_name, user_password, sex, age) values(#{userName}, #{userPassword}, #{sex}, #{age})")
        void insert(User user);
    
        @Update("update t_user set user_name = #{userName}, user_password = #{userPassword}, sex = #{sex}, age = #{age} where user_id = #{userId}")
        void update(User user);
    
        @Delete("delete from t_user where user_id = #{userId}")
        void delete(Long id);
    }
    ​~~~
    
    > - @Select 是查询类的注解，所有的查询均使用这个
    > - @Result 修饰返回的结果集，关联实体类属性和数据库字段一一对应，如果实体类属性和数据库属性名保持一致，就不需要这个属性来修饰。
    > - @Insert 插入数据库使用，直接传入实体类会自动解析属性到对应的值
    > - @Update 负责修改，也可以直接传入对象
    > - @delete 负责删除
    
    ​
    
    > **注意，使用#符号和$符号的不同**



```java
// This example creates a prepared statement, something like select * from teacher where name = ?;
@Select("Select * from teacher where name = #{name}")
Teacher selectTeachForGivenName(@Param("name") String name);

// This example creates n inlined statement, something like select * from teacher where name = 'someName';
@Select("Select * from teacher where name = '${name}'")
Teacher selectTeachForGivenName(@Param("name") String name);
```


### 编写controller测试curd

```java
  @RestController
  @RequestMapping("/user")
  public class UserController {

      @Autowired
      protected UserService userService;

      // http://127.0.0.1:8080/user/getUser?id=1
      @RequestMapping("/getUser")
      public User getUser(Long id){
          return userService.getUserById(id);
      }

      // http://127.0.0.1:8080/user/addUser?userName=aa&userPassword=123456&sex=1&age=10
      @RequestMapping("/addUser")
      public String addUser(User user){
          userService.insertUser(user);
          return "添加用户成功";
      }

      // http://127.0.0.1:8080/user/updateUser?userId=1&userName=aabd&userPassword=123456&sex=1&age=10
      @RequestMapping("/updateUser")
      public String updateUser(User user) {
          userService.updateUser(user);
          return "更新用户成功";
      }
      // http://127.0.0.1:8080/user/delUser/1
      @RequestMapping("/delUser/{id}")
      public String delUser(@PathVariable("id") Long id) {
          userService.deleteUserById(id);
          return "删除用户成功";
      }
  }
```



## 3 xml版本

### 配置

    pom文件和上个版本一样，只是`application.properties`新增以下配置

~~~properties
    mybatis.config-location=classpath:mybatis/mybatis-config.xml
    mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
~~~


    这里的配置是指定mybatis的基础配置文件和实体类映射文件的地址

   ​

### 创建mybatis的基础配置文件

~~~xml
    <configuration>
    	<typeAliases>
    		<typeAlias alias="Integer" type="java.lang.Integer" />
    		<typeAlias alias="Long" type="java.lang.Long" />
    		<typeAlias alias="HashMap" type="java.util.HashMap" />
    		<typeAlias alias="LinkedHashMap" type="java.util.LinkedHashMap" />
    		<typeAlias alias="ArrayList" type="java.util.ArrayList" />
    		<typeAlias alias="LinkedList" type="java.util.LinkedList" />
    	</typeAliases>
    </configuration>
~~~

   ​

### 添加User的映射文件

这里实际上就是把注解版的sql转移到了xml文件中

   ~~~
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
   <mapper namespace="com.telangel.springbootmybatis.mapper.UserMapper" >
       <resultMap id="BaseResultMap" type="com.telangel.springbootmybatis.entity.User" >
           <id column="user_id" property="userId" jdbcType="BIGINT" />
           <result column="user_name" property="userName" jdbcType="VARCHAR" />
           <result column="user_password" property="userPassword" jdbcType="VARCHAR" />
           <result column="sex" property="sex"/>
           <result column="age" property="age" jdbcType="VARCHAR" />
       </resultMap>

       <sql id="Base_Column_List" >
           user_id, user_name, user_password, sex, age
       </sql>

       <select id="getUserById" parameterType="java.lang.Long" resultMap="BaseResultMap" >
           SELECT 
          <include refid="Base_Column_List" />
   	   FROM t_user
   	   WHERE user_id = #{userId}
       </select>

       <insert id="insert" parameterType="com.telangel.springbootmybatis.entity.User" >
          INSERT INTO 
          		t_user
          		(user_name,user_password, sex, age)
          	VALUES
          		(#{userName}, #{userPassword}, #{sex}, #{age})
       </insert>
       
       <update id="update" parameterType="com.telangel.springbootmybatis.entity.User" >
          UPDATE 
          		t_user
           <set>
               <if test="userName != null">user_name = #{userName}, </if>
               <if test="userPassword != null">user_password = #{userPassword},</if>
               <if test="sex != null">sex = #{sex},</if>
               <if test="age != null">age = #{age}</if>
           </set>
          WHERE
          		user_id = #{userId}
       </update>
       
       <delete id="delete" parameterType="java.lang.Long" >
          DELETE FROM
          		 t_user
          WHERE 
          		 user_id =#{id}
       </delete>

   </mapper>
   ~~~

   ​

### 创建dao层代码

~~~java
   public interface UserMapper {

       User getUserById(Long id);

       void insert(User user);

       void update(User user);

       void delete(Long id);
   }

~~~

   ​

### 创建接口方法

   此处和注解版的创建方法一样，不在赘述

### 测试使用

   此处和注解版的创建方法一样，不在赘述

   ​


## 总结

​	springboot对于mybatis的这两种实现都可以完美运行，我们只需要把对应的start导入到项目中，然后做简单的配置就可以使用了，mybatis的这两种模式各有各的特点，下面说一下两种方式的使用场景。

​	注解版：注解版适合简单快速的开发，如果是现在流行的微服务模式，那么一个微服务就对应一个自己的数据库，涉及到的多表连接查询的需求很少，这种场景下，我们可以采用注解版的模式使mybatis。

​	xml版：在一些传统集成的大型项目中，涉及到的查询会很复杂，会有很多的多表连接查询，这时可以使用xml版的mybatis，能够非常方便灵活的动态生成sql，灵活调整。

​	


**[示例代码-github](**https://github.com/ldmyown/springboot-learning**)**  



-------------

**作者：telAngel**  
**出处：[https://ldmyown.github.io](https://ldmyown.github.io)**      
**版权所有，欢迎保留原文链接进行转载：)**