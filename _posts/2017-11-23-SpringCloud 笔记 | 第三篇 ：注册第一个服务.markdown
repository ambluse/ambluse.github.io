---
layout:     post
title:      "SpringCloud 笔记 | 第三篇"
subtitle:   "服务的注册和调用"
date:       2017-11-23
author:     "BENJAMIN"
header-img: ""
tags:
    - SpringCloud-learn 系列
   
---


前两篇我们介绍了注册中心eureka如何搭建以及它的高可用是如何实现的，接下来介绍一下如何使用eureka服务注册中心，搭建一个简单的服务端注册服务，客户端去调用服务使用的案例。

案例中有三个角色：服务注册中心、服务提供者、服务调用者，其中服务注册中心就是eureka，流程是首先启动注册中心，服务提供者生产服务并注册到服务中心中，调用者从服务中心中获取服务并执行。

## 服务提供者

服务提供者可以使用我们springboot教程中的模板，看[这里](https://ambluse.github.io/2017/10/13/SpringBoot-%E7%AC%94%E8%AE%B0-%E7%AC%AC%E4%BA%8C%E7%AF%87-%E5%9C%A8%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E4%BC%98%E9%9B%85%E7%9A%84%E4%BD%BF%E7%94%A8springboot-mybatis/)

### 准备工作
可以将模板项目做成骨架提交到nexus私有库上，后续生成就非常方便了。

在数据库建一个测试表t_user

```
CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(45) DEFAULT NULL,
  `age` int(3) DEFAULT NULL,
  `create_date` datetime DEFAULT NULL,
  `update_date` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4

```

修改pom.xml 和 applicaiton.properties中关于包名的配置，修改成你的报名，然后右键pom.xml
运行 mybatis-generator:generate 生成需要的mybatis对象  

建议依赖这个common项目，会有很多方便开发的封装：

```
		<dependency>
			<groupId>com.yonyou.cloud</groupId>
			<artifactId>common-elegance</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
```

代码和介绍在[这里](https://github.com/ambluse/common-elegance)  

引用了这个项目之后，可以简单通过配置就生成单个对象的rest接口和增删改查的实现  

上代码：

**写一个controller继承BaseController**

```
@RestController
@RequestMapping(value="/user")
public class UserController extends BaseController<UserService, TUser>{

}
```

**写一个service集成BaseService**

```
@Service
public class UserService extends BaseService<Mapper<TUser>, TUser>{

}

```
**好了，现在针对User对象的rest接口以及全部都实现好了，我一行业务代码都没写，是不是很方便简单**

如果增加过swagger的依赖，可以看到我们现在api如下：

![](https://github.com/ambluse/ambluse.github.io/blob/master/img/image.png)

可以通过页面点点增加一个用户用来等后续来测试测试  

### 注册到Eureka上

下面我们来将服务注册到eureka上来供其他服务调用

将我们这个boot的项目改造为一个cloud的项目：

1.引入springcloud

```
<properties>
	<spring-cloud.version>Dalston.SR4</spring-cloud.version>
</properties>	

<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```
2.增加eureka的starter

```
<!--eureka客户端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
```

3.修改启动类,增加EnableDiscoveryClient

```
@SpringBootApplication
@EnableSwagger2Doc
@EnableDiscoveryClient
public class BootMApplication {

	public static void main(String[] args) {
		SpringApplication.run(BootMApplication.class, args);
	}
	
	
}
```

4.修改配置文件

```
spring.application.name=service-provider
server.port:8080
eureka.client.service-url.defaultZone=http://***:8761/eureka

```
ok，启动后去eureka的控制台看下是不是注册上去了

## 服务调用者

和服务提供者一样建立一个工程，我这里使用了之前模板项目的骨架

删除掉不用的类，在调用者中做如下变化：

1.引入springcloud

```
<properties>
	<spring-cloud.version>Dalston.SR4</spring-cloud.version>
</properties>	

<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```
2.增加eureka的starter

```
<!--eureka客户端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		
		
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-feign</artifactId>
		</dependency>
		
```

3.修改启动类,增加EnableDiscoveryClient和EnableFeignClients

```
package com.yonyou.cloud.service.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.feign.EnableFeignClients;

import com.spring4all.swagger.EnableSwagger2Doc;

@SpringBootApplication
@EnableSwagger2Doc
@EnableDiscoveryClient
@EnableFeignClients
public class BootMApplication {

	public static void main(String[] args) {
		SpringApplication.run(BootMApplication.class, args);
	}
}

```

4.修改配置文件

```
spring.application.name=service-consumer
server.port:8081
eureka.client.service-url.defaultZone=http://***:8761/eureka

```
ok，启动后去eureka的控制台是不是提供者和调用者都有了

和提供者不同的是多依赖了一个Feign

>Feign是一个声明式Web Service客户端。使用Feign能让编写Web Service客户端更加简单, 它的使用>方法是定义一个接口，然后在上面添加注解，同时也支持JAX-RS标准的注解。Feign也支持可拔插式的编码>器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和>HttpMessageConverters。Feign可以与Eureka和Ribbon组合使用以支持负载均衡。


## 测试一下是否能够调用注册的服务

在调用者这里写一个feign来调用提供者

```
package com.yonyou.cloud.service.consumer.feign;

import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.yonyou.cloud.common.beans.RestResultResponse;
import com.yonyou.cloud.service.consumer.entity.TUser;

@FeignClient(name= "service-provider")
public interface UserFeign {
	
	@RequestMapping(value = "user/{id}",method=RequestMethod.GET)
    public RestResultResponse<TUser> getUserInfo(@PathVariable(value="id") int id);
	
}

```

写一个Controller来测试一下


```
package com.yonyou.cloud.service.consumer.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.yonyou.cloud.service.consumer.feign.UserFeign;

@RestController
@RequestMapping(value="/c-user")
public class UserController {

	@Autowired
	UserFeign client;
	
	
	@RequestMapping("/{id}")
	public String userName(@PathVariable int id){
		return client.getUserInfo(id).getData().getName();
	}
	
	
}

```

http://localhost:8081/c-user/1

浏览器敲一下，返回了用户名称，看下log就明白调用流程了



