---
layout: post
title: springboot(二)：Spring boot中Redis的使用
category: springboot
tags: [springboot]
---

本文介绍一下在springboot中如何使用redis来做缓存

## 说明

spring boot除了对于常用的数据库支持，还对nosql 数据库也进行了封装自动化。这里对redis的使用做一个介绍。

## redis介绍

Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。从2010年3月15日起，Redis的开发工作由VMware主持。从2013年5月开始，Redis的开发由Pivotal赞助。

redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。

Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。存盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。

可以说Redis兼具了缓存系统和数据库的一些特性，因此有着丰富的应用场景。本文介绍Redis在Spring Boot中两个典型的应用场景。

##  如何使用

1、引入 spring-boot-starter-redis依赖

``` xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2、添加配置文件

``` properties
# REDIS (RedisProperties)
# Redis数据库索引（默认为0）
spring.redis.database=0  
# Redis服务器地址
spring.redis.host=192.168.0.58
# Redis服务器连接端口
spring.redis.port=6379  
# Redis服务器连接密码（默认为空）
spring.redis.password=  
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8  
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1  
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8  
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0  
# 连接超时时间（毫秒）
spring.redis.timeout=0  
```

3、创建redis服务类

- redis提供了下列几种数据类型可供存取
  - string
  - hash
  - list
  - set
  - zset


- 此服务类中提供用户的增删改查操作
- 由于redis 没有表结构的概念，所以要实现mysql数据库中表的数据在redis中存取，需要做一个转换，将json格式的文本作为redis和java普通对象互相交换数据的存储格式，要取出数据时，再将json文本转换为java 对象。
- 保存redis使用key-value的方式存储数据，默认是永久存储的，也可以指定一个时限来定义数据的生命周期，超过指定时限的数据将被redis自动清除。

~~~java
@Repository
public class UserRedis {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    public void add(String key, Long time, User user) {
        redisTemplate.opsForValue().set(key, JSONObject.toJSONString(user), time, TimeUnit.SECONDS);
    }

    public void add(String key, Long time, List<User> users) {
        redisTemplate.opsForValue().set(key, JSONObject.toJSONString(users), time, TimeUnit.SECONDS);
    }

    public User get(String key) {
        String userJson = redisTemplate.opsForValue().get(key);
        User user = null;
        if (!StringUtils.isEmpty(userJson)) {
            user = JSONObject.parseObject(userJson, User.class);
        }
        return user;
    }

    public List<User> getList(String key) {
        String usersJson = redisTemplate.opsForValue().get(key);
        List<User> users = null;
        if (!StringUtils.isEmpty(usersJson)) {
            users = (List<User>)JSONObject.parseObject(usersJson, List.class);
        }
        return users;
    }

    public void delete(String key) {
        redisTemplate.opsForValue().getOperations().delete(key);
    }
}
~~~



添加cache的配置类，此处springboot1.x版本和2.x版本有一定的差别

- 1.x 版本的配置

``` java
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport{
	
	@Bean
	public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }

    @SuppressWarnings("rawtypes")
    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager rcm = new RedisCacheManager(redisTemplate);
        //设置缓存过期时间
        //rcm.setDefaultExpiration(60);//秒
        return rcm;
    }
    
    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate template = new StringRedisTemplate(factory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }

}
```

- 2.x版本的配置

~~~java
@Configuration
public class RedisConfig {

    @Bean
    public KeyGenerator keyGenerator() {
        return (target, method, params) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getName());
            sb.append(method.getName());
            for (Object obj : params) {
                sb.append(obj.toString());
            }
            return sb.toString();
        };
    }

    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
       RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(factory);
        return redisTemplate;
    }
}
~~~

  

3、配置完成后就可以开始测试redis

``` java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootRedisApplicationTests {

	@Autowired
	private UserRedis userRedis;

	@Test
	public void add() {
		User user = new User("aa", 10, "男");
		userRedis.add("1",60l, user);
		if (userRedis.get("1") != null) {
			System.out.println("保存成功，保存的数据为" + JSONObject.toJSONString(userRedis.get("1")));
		}
	}

	@Test
	public void addList() {
		User user1 = new User("aa", 10, "男");
		User user2 = new User("aa", 10, "男");
		userRedis.add("2",60l, Arrays.asList(user1, user2));
		if (userRedis.getList("2") != null) {
			System.out.println("保存成功，保存的数据为" + JSONObject.toJSONString(userRedis.getList("2")));
		}
	}

	@Test
	public void delete(){
		userRedis.delete("2");

	}

}
```




##  共享Session-spring-session-data-redis

分布式系统中，sessiong共享有很多的解决方案，其中托管到缓存中应该是最常用的方案之一， 

### Spring Session官方说明

Spring Session provides an API and implementations for managing a user’s session information.


### 如何使用

1、引入依赖

``` xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

2、Session配置：

``` java
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 86400*30)
public class SessionConfig {
}
```

> maxInactiveIntervalInSeconds: 设置Session失效时间，使用Redis Session之后，原Boot的server.session.timeout属性不再生效

好了，这样就配置好了，我们来测试一下


3、测试

添加测试方法获取sessionid

``` java
@RequestMapping("/uid")
    String uid(HttpSession session) {
        UUID uid = (UUID) session.getAttribute("uid");
        if (uid == null) {
            uid = UUID.randomUUID();
        }
        session.setAttribute("uid", uid);
        return session.getId();
    }
```

登录redis 输入 ``` keys '*sessions*' ``` 
``` xml
1) "spring:session:sessions:expires:d3388eda-c4e8-4833-ab99-7aa4f139916d"
2) "spring:session:sessions:ff0245b8-eefb-4b2a-9420-5ec9683ccab7"
```
其中 d3388eda-c4e8-4833-ab99-7aa4f139916d为失效时间，这里做了数据转换，意思是这个时间后session失效，``` ff0245b8-eefb-4b2a-9420-5ec9683ccab7 ``` 为sessionId,登录http://localhost:8080/uid 发现会一致，就说明session 已经在redis里面进行有效的管理了。


### 如何在两台或者多台中共享session

 其实就是按照上面的步骤在另一个项目中再次配置一次，启动后自动就进行了session共享。


**[示例代码-github](**https://github.com/ldmyown/springboot-learning**)**

-------------

**作者：telAngel**  
**出处：[https://ldmyown.github.io](https://ldmyown.github.io)**      
**版权所有，欢迎保留原文链接进行转载：)**