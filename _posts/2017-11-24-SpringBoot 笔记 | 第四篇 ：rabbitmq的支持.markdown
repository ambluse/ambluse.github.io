---
layout:     post
title:      "SpringBoot 笔记 | 第四篇"
subtitle:   "rabbitmq"
date:       2017-11-24
author:     "BENJAMIN"
header-img: ""
tags:
    - SpringBoot 系列
---

## 介绍

RabbitMQ 即一个消息队列，主要是用来实现应用程序的异步和解耦，同时也能起到消息缓冲，消息分发的作用。  

消息中间件最主要的作用是解耦，中间件最标准的用法是生产者生产消息传送到队列，消费者从队列中拿取消息并处理，生产者不用关心是谁来消费，消费者不用关心谁在生产消息，从而达到解耦的目的。在分布式的系统中，消息队列也会被用在很多其它的方面，比如：分布式事务的支持，RPC的调用等等。

我们通过rabbitmq也实现了消息必达的一个组件，用来解决分布式事务的问题，地址在[这里](https://github.com/yonyou-auto-dev/microservice-mom)，当然这不是这篇的重点。  

下面我们来看看springboot怎么简单集成rabbitmq

## 准备工作

还是之前的的模板项目，增加依赖

```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-amqp</artifactId>
		</dependency>
```

增加配置文件

```


spring.rabbitmq.host=**
spring.rabbitmq.port=5672
spring.rabbitmq.username=test
spring.rabbitmq.password=test


```

写一个sender

```
package com.yonyou.cloud.boot.amqp.sender;

import java.util.Date;

import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class DemoSender {
	
	@Autowired
	AmqpTemplate client;
	
	
	public void send() {
		String context = "hello " + new Date();
		System.out.println("Sender : " + context);
		this.client.convertAndSend("hello-exchange", "",context);
	}

}

```

写一个listener

```
package com.yonyou.cloud.boot.amqp.listener;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.core.ChannelAwareMessageListener;
import org.springframework.amqp.support.converter.SimpleMessageConverter;
import org.springframework.stereotype.Component;

import com.rabbitmq.client.Channel;


@Component
public class DemoListener implements ChannelAwareMessageListener{

	SimpleMessageConverter convert =  new SimpleMessageConverter();
	
	@Override
	public void onMessage(Message message, Channel channel) throws Exception {
		// TODO Auto-generated method stub
		System.out.println(convert.fromMessage(message));
	}

}

```

>这个里用的是 SimpleMessageConverter 来转换的数据，如果入队的是json格式就应该用对应的convert来转换了



最后配置到一起写个config来绑定队列与exchange

```
package com.yonyou.cloud.boot.amqp.config;

import org.springframework.amqp.core.AcknowledgeMode;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.yonyou.cloud.boot.amqp.listener.DemoListener;

@Configuration
public class RabbitMqConfig {

    @Bean
    public Queue mqOps() {
        return new Queue("hello-queue");
    }

    @Bean
    FanoutExchange fanoutExchange() {
        return new FanoutExchange("hello-exchange");
    }

    @Bean
    Binding bindingMqMsgExchange(Queue mqOps,FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(mqOps).to(fanoutExchange);
    }
    
    @Bean
	public SimpleMessageListenerContainer messageContainer1(ConnectionFactory connectionFactory,
			DemoListener mqMsgListener) {
		SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
		container.setQueues(mqOps());
		container.setExposeListenerChannel(true);
		container.setConcurrentConsumers(10);
		container.setAcknowledgeMode(AcknowledgeMode.AUTO); // 设置确认模式手工确认
		container.setMessageListener(mqMsgListener);
		container.setMaxConcurrentConsumers(10);//设置最大消费者数量 防止大批量涌入
		return container;
	}
	
}
```

好了 写个controller测试下

```
package com.yonyou.cloud.boot.amqp.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.yonyou.cloud.boot.amqp.sender.DemoSender;

@RestController
public class DemoController {

	@Autowired
	DemoSender sender;
	
	@RequestMapping("/demo")
	public String send(){
		sender.send();
		return "ok";
	}
}


```

http://ip:8080/demo，看下控制台

```
[2m2017-11-24 11:44:52.660[0;39m [32m INFO[0;39m [35m31287[0;39m [2m---[0;39m [2m[nio-8080-exec-1][0;39m [36mo.a.c.c.C.[Tomcat].[localhost].[/]      [0;39m [2m:[0;39m Initializing Spring FrameworkServlet 'dispatcherServlet'
[2m2017-11-24 11:44:52.660[0;39m [32m INFO[0;39m [35m31287[0;39m [2m---[0;39m [2m[nio-8080-exec-1][0;39m [36mo.s.web.servlet.DispatcherServlet       [0;39m [2m:[0;39m FrameworkServlet 'dispatcherServlet': initialization started
[2m2017-11-24 11:44:52.700[0;39m [32m INFO[0;39m [35m31287[0;39m [2m---[0;39m [2m[nio-8080-exec-1][0;39m [36mo.s.web.servlet.DispatcherServlet       [0;39m [2m:[0;39m FrameworkServlet 'dispatcherServlet': initialization completed in 40 ms
Sender : hello Fri Nov 24 11:44:52 CST 2017
hello Fri Nov 24 11:44:52 CST 2017
```

搞定，代码在[这里](https://github.com/ambluse/SpringCloud-learn/tree/master/springcloud%20chapter4/boot-amqp)



