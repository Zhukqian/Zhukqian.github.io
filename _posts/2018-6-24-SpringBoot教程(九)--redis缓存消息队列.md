---
layout:     post
title:      "SpringBoot教程(九)--redis缓存消息队列"
subtitle:   SpringBoot
date:       2018-6-24
author:     朱坤乾
header-img: ![](https://imgchr.com/i/kQTFtH)
catalog: true
tags:
    - SpringBoot
---

这篇文章主要讲述如何在springboot中用reids实现消息队列。

## 准备阶段

    安装redis
    java 1.8
    maven 3.0
    idea

##  环境依赖

创建一个新的springboot工程，在其pom文件,加入spring-boot-starter-data-redis依赖：

```

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>

```

##  创建一个消息接收者

REcevier类，它是一个普通的类，需要注入到springboot中。

```

public class Receiver {
    private static final Logger LOGGER = LoggerFactory.getLogger(Receiver.class);

    private CountDownLatch latch;

    @Autowired
    public Receiver(CountDownLatch latch) {
        this.latch = latch;
    }

    public void receiveMessage(String message) {
        LOGGER.info("Received <" + message + ">");
        latch.countDown();
    }
}

```

##  注入消息接收者

```
@Bean
	Receiver receiver(CountDownLatch latch) {
		return new Receiver(latch);
	}

	@Bean
	CountDownLatch latch() {
		return new CountDownLatch(1);
	}

	@Bean
	StringRedisTemplate template(RedisConnectionFactory connectionFactory) {
		return new StringRedisTemplate(connectionFactory);
	}

```

##  注入消息监听容器

在spring data redis中，利用redis发送一条消息和接受一条消息，需要三样东西：


    一个连接工厂
    一个消息监听容器
    Redis template

上述1、3步已经完成，所以只需注入消息监听容器即可：

```

@Bean
	RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory,
											MessageListenerAdapter listenerAdapter) {

		RedisMessageListenerContainer container = new RedisMessageListenerContainer();
		container.setConnectionFactory(connectionFactory);
		container.addMessageListener(listenerAdapter, new PatternTopic("chat"));

		return container;
	}

	@Bean
	MessageListenerAdapter listenerAdapter(Receiver receiver) {
		return new MessageListenerAdapter(receiver, "receiveMessage");
	}

```

##  测试

在springboot入口的main方法：

```

public static void main(String[] args) throws Exception{
		ApplicationContext ctx =  SpringApplication.run(SpringbootRedisApplication.class, args);

		StringRedisTemplate template = ctx.getBean(StringRedisTemplate.class);
		CountDownLatch latch = ctx.getBean(CountDownLatch.class);

		LOGGER.info("Sending message...");
		template.convertAndSend("chat", "Hello from Redis!");

		latch.await();

		System.exit(0);
	}


```

先用redisTemplate发送一条消息，接收者接收到后，打印出来。启动springboot程序，控制台打印：

    2017-04-20 17:25:15.536 INFO 39148 — [ main] com.forezp.SpringbootRedisApplication : Sending message… 2017-04-20 17:25:15.544 INFO 39148 — [ container-2] com.forezp.message.Receiver : 》Received <Hello from Redis!>

测试通过，接收者确实接收到了发送者的消息。


