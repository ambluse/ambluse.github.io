---
layout:     post
title:      "SpringBoot 笔记 | 第二篇"
subtitle:   "在生产环境优雅的使用springboot-mybatis"
date:       2017-10-13
author:     "BENJAMIN"
header-img: ""
tags:
    - SpringBoot 系列
---


上一篇写了最简单的方式来进行springboot与mybatis的整合，算是一个脚手架方便我们自己按自己的需要增加其他的组件来提高效率，在真正项目中像之前这样虽然简单但是还是太费劲了，我们今天来看看如何集成一些工具用简单优雅的配置方式提高项目中的生产力

## 需求

* 首先还是要配置简单
* 使用连接池来提高执行效率
* 提供通用的单表增删改查的实现(不需要再单独开发了)
* 提供通用的翻页的实现

让开发人员专注与业务sql实现就好了
 
## 选型

1. 与mybatis的整合mybatis-spring-boot-starter还是少不了的
2. 连接池使用alibaba的[druid](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter) ，可以通过官方的starter来简化使用
3. 通用的增删改查使用[通用mapper](https://github.com/abel533/Mapper)，集成很简单
4. 通用翻页使用[pagehelper](https://github.com/pagehelper/Mybatis-PageHelper)

### 开始
同样从start.spring.io 生成项目，下载下来导入到STS中。

直接给出依赖pom的结果，以后拷贝过去直接用

```

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>org.lxd.learn</groupId>
	<artifactId>boot-m</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>boot-m</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.7.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		
		
		<!--  MyBatis Generator 的配置 -->
        <!--  Java接口和实体类  -->
        <targetJavaProject>${basedir}/src/main/java</targetJavaProject>
        <targetMapperPackage>org.lxd.learn.bootm.mapper</targetMapperPackage>
        <targetModelPackage>org.lxd.learn.bootm.entity</targetModelPackage>
        <!--  XML生成路径  -->
        <targetResourcesProject>${basedir}/src/main/resources</targetResourcesProject>
        <targetXMLPackage>mybatis/mapper</targetXMLPackage>
        
        <mapper.version>3.3.6</mapper.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- 连接池 -->
		<!--通过starter简化配置-->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid-spring-boot-starter</artifactId>
			<version>1.1.4</version>
		</dependency>
		
		<!-- 
		通过starter简化配置 这个就不用了
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.1.4</version>
		</dependency>
 		-->

		<!-- 通用mapper -->
		<dependency>
			<groupId>tk.mybatis</groupId>
			<artifactId>mapper-spring-boot-starter</artifactId>
			<version>1.1.3</version>
		</dependency>

		<!-- 通用翻页 -->
		<dependency>
			<groupId>com.github.pagehelper</groupId>
			<artifactId>pagehelper-spring-boot-starter</artifactId>
			<version>1.2.3</version>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			
			<plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
                <configuration>
                    <configurationFile>${basedir}/src/main/resources/generator/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>${mysql.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>tk.mybatis</groupId>
                        <artifactId>mapper</artifactId>
                        <version>${mapper.version}</version>
                    </dependency>
                </dependencies>
            </plugin>
		</plugins>
	</build>


</project>


```

看上面的配置，比上一篇增加了 mapper的依赖，page的依赖，自动生成mybatis代码的generator插件，
druid的依赖。

修改application.properties

```
mybatis.type-aliases-package=org.lxd.learn.bootm.entity
#mybatis.config-location=classpath:mybatis/mybatis-config.xml 需要配置再用
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml



spring.datasource.driverClassName = com.mysql.jdbc.Driver
spring.datasource.url = jdbc:mysql://10.180.8.205:3306/dmc_coupon?useUnicode=true&characterEncoding=utf-8
spring.datasource.username = root
spring.datasource.password = rcs

#更多配置查看 https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter


#通用mapper的配置 https://mapperhelper.github.io/docs/1.integration/
mapper.mappers=tk.mybatis.mapper.common.Mapper
mapper.not-empty=false
mapper.identity=MYSQL

#pagehelper的配置  https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md
pagehelper.helperDialect=mysql
pagehelper.reasonable=true
pagehelper.supportMethodsArguments=true
pagehelper.params=count=countSql

mapper.plugin = tk.mybatis.mapper.generator.MapperPlugin


```

配置的上面都还一样 ，下面增加mapper的配置和pagehelper的配置以及generator的配置，还有更多配置用到的话查看对应的文档。  
>**如果是生产上，这个配置应该会再拆部分到 application-pro.properties 或者 application-test.properties 中，通过启动脚本来区分读取哪个配置，application.properties中应该放一些和环境无关的配置**


配置代码生成文件generatorConfig.xml  
放到resources/generator/下

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
	<properties resource="application.properties"/>
	
	<context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
		<property name="beginningDelimiter" value="`"/>
		<property name="endingDelimiter" value="`"/>
		
		<plugin type="${mapper.plugin}">
		  <property name="mappers" value="${mapper.mappers}"/>
		</plugin>
		
		<jdbcConnection driverClass="${spring.datasource.driverClassName}"
		                connectionURL="${spring.datasource.url}"
		                userId="${spring.datasource.username}"
		                password="${spring.datasource.password}">
		</jdbcConnection>
		
		<javaModelGenerator targetPackage="${targetModelPackage}" targetProject="${targetJavaProject}"/>
		
		<sqlMapGenerator targetPackage="${targetXMLPackage}"  targetProject="${targetResourcesProject}"/>
		
		<javaClientGenerator targetPackage="${targetMapperPackage}" targetProject="${targetJavaProject}" type="XMLMAPPER" />
		
		<table tableName="TR_PACKAGE_BATCH" >
		  <generatedKey column="REL_ID" sqlStatement="Mysql" identity="true"/>
		</table>
	</context>
</generatorConfiguration>

```

## 提升生产力

下面我们来一生成dao层，满足开篇需要的需求  
右键pom.xml runAs mybatis-generator:generate 回车执行
  
**检查一下对应的entity mapper  mapper.xml是不是都生成了，如果找不到检查看generatorConfig.xml的配置 。**   

下面就可以直接利用好这些工具进行开发了，所有增删改成的方法你的mapper就已经集成了。
