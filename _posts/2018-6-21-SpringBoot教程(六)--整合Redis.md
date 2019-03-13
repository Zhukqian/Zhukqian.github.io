---
layout:     post
title:      "SpringBoot教程(六)--整合Redis"
subtitle:   SpringBoot
date:       2018-6-21
author:     朱坤乾
header-img: ![](https://imgchr.com/i/kQTFtH)
catalog: true
tags:
    - SpringBoot
---
##  Redis简介

Redis就是Nosql,非关系数据库

Redis的出色之处不仅仅是性能，Redis最大的魅力是支持保存多种数据结构，此外单个value的最大限制是1GB，不像 memcached只能保存1MB的数据，因此Redis可以用来实现很多有用的功能。

支持丰富数据类型，支持string，list，set，sorted set，hash 

支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行 

丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除,自动增加删除

##  引入依赖：

```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>


```

##  配置数据源

```
spring.redis.host=localhost
spring.redis.port=6379
#spring.redis.password=
spring.redis.database=1
spring.redis.pool.max-active=8
spring.redis.pool.max-wait=-1
spring.redis.pool.max-idle=500
spring.redis.pool.min-idle=0
spring.redis.timeout=0

```

如果你的redis有密码，配置下即可。经过上述两步的操作，你可以访问redis数据了。

##  数据访问层dao

通过redisTemplate来访问redis.

```
@Repository
public class RedisDao {

    @Autowired
    private StringRedisTemplate template;

    public  void setKey(String key,String value){
        ValueOperations<String, String> ops = template.opsForValue();
        ops.set(key,value,1, TimeUnit.MINUTES);//1分钟过期
    }

    public String getValue(String key){
        ValueOperations<String, String> ops = this.template.opsForValue();
        return ops.get(key);
    }
}

```

##  单元测试

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootRedisApplicationTests {

	public static Logger logger= LoggerFactory.getLogger(SpringbootRedisApplicationTests.class);
	@Test
	public void contextLoads() {
	}

	@Autowired
	RedisDao redisDao;
	@Test
	public void testRedis(){
		redisDao.setKey("name","zkq");
		redisDao.setKey("age","11");
		logger.info(redisDao.getValue("name"));
		logger.info(redisDao.getValue("age"));
	}
}

```


启动单元测试，你发现控制台打印出,1分钟之后就会过期






