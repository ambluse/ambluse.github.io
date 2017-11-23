---
layout:     post
title:      "SpringCloud 笔记 | 第二篇"
subtitle:   "高可用的注册中心"
date:       2017-11-23
author:     "BENJAMIN"
header-img: ""
tags:
    - SpringCloud-learn 系列
   
---


> Eureka是Netflix开源的一款提供服务注册和发现的产品，它提供了完整的Service Registry和
> Service Discovery实现。也是SpringCloud体系中最重要最核心的组件之一。

---

先按照[上一篇](https://ambluse.github.io/2017/10/12/SpringCloud-%E7%AC%94%E8%AE%B0-%E7%AC%AC%E4%B8%80%E7%AF%87-%E6%9C%8D%E5%8A%A1%E7%9A%84%E6%B3%A8%E5%86%8C%E5%92%8C%E5%8F%91%E7%8E%B0/)的指南建立好工程


## 集群
注册中心这么关键的服务，如果是单点话，遇到故障就是毁灭性的。在一个分布式系统中，服务注册中心是最重要的基础部分，理应随时处于可以提供服务的状态。为了维持其可用性，使用集群是很好的解决方案。Eureka通过互相注册的方式来实现高可用的部署，所以我们只需要将Eureke Server配置其他可用的serviceUrl就能实现高可用部署。

### eureka集群使用

在生产中我们可能需要三台或者大于三台的注册中心来保证服务的稳定性，配置的原理其实都一样，将注册中心分别指向其它的注册中心。这里只介绍三台集群的配置情况，其实和双节点的注册中心类似，每台注册中心分别又指向其它两个节点即可。

1、创建application-peer1.properties，作为peer1服务中心的配置，并将serviceUrl指向peer2,peer3

```
spring.application.name=spring-cloud-eureka

server.port=8761
eureka.instance.hostname=peer1

eureka.client.serviceUrl.defaultZone=http://peer2:8762/eureka/,http://peer3:8763/eureka/

```

2、创建application-peer2.properties，作为peer2服务中心的配置，并将serviceUrl指向peer1,peer3


```
spring.application.name=spring-cloud-eureka

server.port=8762
eureka.instance.hostname=peer2

eureka.client.serviceUrl.defaultZone=http://peer1:8761/eureka/,http://peer3:8763/eureka/

```

3、创建application-peer3.properties，作为peer2服务中心的配置，并将serviceUrl指向peer1,peer2


```
spring.application.name=spring-cloud-eureka

server.port=8763
eureka.instance.hostname=peer3

eureka.client.serviceUrl.defaultZone=http://peer1:8761/eureka/,http://peer2:8762/eureka/

```

### 打包

运行 mvn clean package 打包

使用命令运行

```
java -jar demo-eureka-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer1
java -jar demo-eureka-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer2
java -jar demo-eureka-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer3
```

记得在hosts中加入机器名,否则相互不认识  

vi /etc/hosts

```
127.0.0.1 peer1
127.0.0.1 peer2
127.0.0.1 peer3
```

打开网页 http://121.196.193.149:8761/  
如果看到DS Replicas 中有其他两个节点，那么恭喜你，成功了

