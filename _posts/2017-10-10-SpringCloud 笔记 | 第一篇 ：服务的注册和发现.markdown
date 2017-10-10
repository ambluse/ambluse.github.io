---
layout:     post
title:      "SpringCloud 笔记 | 第一篇"
subtitle:   "简单的注册中心"
date:       2017-10-10
author:     "BENJAMIN"
header-img: ""
tags:
    - SpringCloud-learn 系列
   
---


> Eureka是Netflix开源的一款提供服务注册和发现的产品，它提供了完整的Service Registry和
> Service Discovery实现。也是SpringCloud体系中最重要最核心的组件之一。

---

## 创建一个eureka server

创建一个maven项目

或者通过start.spring.io来自动创建：

pom.xml 配置关键信息如下:

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.7.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spring-cloud.version>Dalston.SR4</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

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

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
	
可以看到使用的是spring boot 1.5.7.RELEASE 和 spring cloud Dalston.SR4