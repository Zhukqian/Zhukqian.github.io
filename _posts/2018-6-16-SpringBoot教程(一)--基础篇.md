---
layout:     post
title:      "SpringBoot教程(一)--基础篇"
subtitle:   SpringBoot
date:       2018-6-16
author:     朱坤乾
header-img: ![](https://imgchr.com/i/kQTFtH)
catalog: true
tags:
    - SpringBoot
---
##  1.简介
官方简介:

SpringBoot在建立生产中的独立程序上非常简便、只需要一些简便的配置就能运行起来。大致有如下特点：

    创建独立的Spring applications
	
    能够使用内嵌的Tomcat, Jetty or Undertow，不需要部署war
	
    提供starter pom来简化maven配置
	
    自动配置Spring
	
	
    提供一些生产环境的特性，比如metrics, health checks and externalized configuration
	
    绝对没有代码生成和XML配置要求
	
##  2. 快速开始

###  2.1maven构建项目

1、访问http://start.spring.io/

2、选择构建工具Maven Project、Spring Boot版本1.3.6以及一些工程基本信息，点击“Switch to the full version.”java版本选择1.7，可参考下图所示：

3、点击Generate Project下载项目压缩包

4、解压后，使用IDEA，导入即可

Spring Boot的基础结构共三个文件:

    src/main/java 程序开发以及主程序入口

    src/main/resources 配置文件

    src/test/java 测试程序


另外，spingboot建议的目录结果如下：

root package结构：com.example.myproject


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


    Application.java 建议放到根目录下面,主要用于做一些框架配置

    domain目录主要用于实体（Entity）与数据访问层（Repository）

    service 层主要是业务类代码

    controller 负责页面访问控制
	
采用默认配置可以省去很多配置，当然也可以根据自己的喜欢来进行更改

最后，启动Application main方法，至此一个java项目搭建好了！

###  2.2引入web模块


1、pom.xml中添加支持web的模块：

```
<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.3.3.RELEASE</version>
	</parent>
	<dependencies>
	  <!—SpringBoot web 组件 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>

```

在pom.xml中引入spring-boot-start-parent,spring官方的解释叫什么stater poms,它可以提供dependency management,也就是说依赖管理，引入以后在申明其它dependency的时候就不需要version了，后面可以看到。

spring-boot-starter-web作用  :  springweb 核心组件

spring-boot-maven-plugin作用: 如果我们要直接Main启动spring，那么以下plugin必须要添加，否则是无法启动的。如果使用maven 的spring-boot:run的话是不需要此配置的。（我在测试的时候，如果不配置下面的plugin也是直接在Main中运行的。）


###  2.3编写HelloWorld服务

创建HelloController类，内容如下

```
@RestController
@EnableAutoConfiguration
public class HelloController {
	@RequestMapping("/hello")
	public String index() {
		return "Hello World";
	}	
public static void main(String[] args) {
		SpringApplication.run(HelloController.class, args);
	}
}

```

####  @RestController解释:

在上加上RestController 表示修饰该Controller所有的方法返回JSON格式,直接可以编写Restful接口

####  @EnableAutoConfiguration

注解:作用在于让 Spring Boot   根据应用所声明的依赖来对 Spring 框架进行自动配置
        这个注解告诉Spring Boot根据添加的jar依赖猜测你想如何配置Spring。由于spring-boot-starter-web添加了Tomcat和Spring MVC，所以auto-configuration将假定你正在开发一个web应用并相应地对Spring进行设置。

####  SpringApplication.run(HelloController.class, args);

   标识为启动类
   
###  3.启动方式

####  3.1:SpringBoot启动方式1
   
   Springboot默认端口号为8080
   
```
@RestController
@EnableAutoConfiguration
public class HelloController {
	@RequestMapping("/hello")
	public String index() {
	return "Hello World";
	}	
public static void main(String[] args) {
		SpringApplication.run(HelloController.class, args);
	}
}

```
   启动主程序，打开浏览器访问http://localhost:8080/index，可以看到页面输出Hello World
   
####  3.2:SpringBoot启动方式2

@ComponentScan(basePackages = "com.itmayiedu.controller")---控制器扫包范围

```
@ComponentScan(basePackages = "com.itmayiedu.controller")
@EnableAutoConfiguration
public class App {
	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
}

```
添加一个注解,进行包扫描,然后启动

####  3.3:SpringBoot启动方式3

打成jar包,然后java -jar  包名



##  总结

这样就简单搭建了一个SpringBoot项目,后期会逐步的添加各种应用.











