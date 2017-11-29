---
layout:     post
title:      "SpringCloud 笔记 | 第四篇"
subtitle:   "熔断器"
date:       2017-11-29
author:     "BENJAMIN"
header-img: ""
tags:
    - SpringCloud-learn 系列
   
---

## 介绍

服务可以相互调用了，但是我们没法保证每个服务在所有时间都能正常运行，一旦某个服务出现异常，  
将会影响整个调用链上的其他服务，造成雪崩效应，这与我们之前提到微服务舱壁的优势是相悖的，  
今天我们就来看看，SpringCloud如何缓解服务雪崩的问题

## 熔断器（CircuitBreaker）

熔断器的原理很简单，如同电力过载保护器。它可以实现快速失败，如果它在一段时间内侦测到许多类似的错误，会强迫其以后的多个调用快速失败，不再访问远程服务器，从而防止应用程序不断地尝试执行可能会失败的操作，使得应用程序继续执行而不用等待修正错误，或者浪费CPU时间去等到长时间的超时产生。熔断器也可以使应用程序能够诊断错误是否已经修正，如果已经修正，应用程序会再次尝试调用操作。

熔断器模式就像是那些容易导致错误的操作的一种代理。这种代理能够记录最近调用发生错误的次数，然后决定使用允许操作继续，或者立即返回错误

## SpringCloud Hystrix

还是使用上一篇我们已经实现的可以相互调用的两个项目，一个consumer，一个producer   

熔断器生效是在消费者一方，我们来修改下消费者的配置

application.properties中增加

```
feign.hystrix.enabled=true
```

实现一个fallback的处理类，实现原来的userFeign,
当失败的时候会由这里处理返回，限制设置返回一个默认用户

```
package com.yonyou.cloud.service.consumer.feign;

import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.yonyou.cloud.common.beans.RestResultResponse;
import com.yonyou.cloud.service.consumer.entity.TUser;

@Component
public class UserFeignHystrix implements UserFeign{

    @Override
    @RequestMapping(value = "user/{id}",method=RequestMethod.GET)
    public RestResultResponse<TUser> getUserInfo(@PathVariable(value="id") int id){
    	TUser user = new TUser();
    	user.setId(0);
    	user.setName("默认用户");
    	return new RestResultResponse().data(user).success(true);
    }
}

```

修改原来的feignclient，增加fallback的配置

```
package com.yonyou.cloud.service.consumer.feign;


import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.yonyou.cloud.common.beans.RestResultResponse;
import com.yonyou.cloud.service.consumer.entity.TUser;

@FeignClient(name= "service-provider",fallback=UserFeignHystrix.class)
public interface UserFeign {
	
	@RequestMapping(value = "user/{id}",method=RequestMethod.GET)
    public RestResultResponse<TUser> getUserInfo(@PathVariable(value="id") int id);
	
}
```

完成
调用 [http://localhost:8081/c-user/1](http://localhost:8081/c-user/1) 看看效果

返回：默认用户 成功


演示的代码在[这](https://github.com/ambluse/SpringCloud-learn/tree/master/springcloud%20chapter4)


