---
layout:     post
title:      "SpringBoot 笔记 | 第三篇"
subtitle:   "多数据源"
date:       2017-10-31
author:     "BENJAMIN"
header-img: ""
tags:
    - SpringBoot 系列
---


上一篇我们搭建了一个可以在生产环境使用的整合好mybatis的框架，应该可以应付大多数的需求了。但是有时候比较复杂的业务需要访问多个库，或者主从分离的情况下需要做读写分离，一个方案使用数据库中间件类似mycat这样的，但是又要搭建一个中间件系统太复杂了，如果只是简单的使用，我建议使用如下的办法。

## 配置文件

**pom配置和上一篇一样:**  
主要变化在properties部分，需要将两个数据源的mapper和entity生成到不同的目录中

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>org.lxd.learn</groupId>
	<artifactId>boot-multi</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>boot-multi</name>
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
		<!-- MyBatis Generator 的配置 -->
		
		<!-- Java接口和实体类 -->
		<targetJavaProject>${basedir}/src/main/java</targetJavaProject>
		<!-- datasource1 -->
		<targetMapperPackage1>org.lxd.learn.boot.multi.datasource1.mapper</targetMapperPackage1>
		<targetModelPackage1>org.lxd.learn.boot.multi.datasource1.entity</targetModelPackage1>
		
		<!-- datasource2 -->
		<targetMapperPackage2>org.lxd.learn.boot.multi.datasource2.mapper</targetMapperPackage2>
		<targetModelPackage2>org.lxd.learn.boot.multi.datasource2.entity</targetModelPackage2>
		
		<!-- XML生成路径 -->
		<targetResourcesProject>${basedir}/src/main/resources</targetResourcesProject>
		<!-- datasource1 -->
		<targetXMLPackage1>mybatis/mapper/datasource1</targetXMLPackage1>
		
		<!-- datasource2 -->
		<targetXMLPackage2>mybatis/mapper/datasource2</targetXMLPackage2>
		
		
		

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
		<!--通过starter简化配置 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid-spring-boot-starter</artifactId>
			<version>1.1.4</version>
		</dependency>

		<!-- 通过starter简化配置 这个就不用了 <dependency> <groupId>com.alibaba</groupId> <artifactId>druid</artifactId> 
			<version>1.1.4</version> </dependency> -->

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
**application.properties**  

配置两个数据源：datasource1 ，datasource2 指向不同连接

```
mybatis.type-aliases-package=org.lxd.learn.boot.multi.entity
#mybatis.config-location=classpath:mybatis/mybatis-config.xml 需要配置再用
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml



spring.datasource.datasource1.driverClassName = com.mysql.jdbc.Driver
spring.datasource.datasource1.url = jdbc:mysql://10.180.8.205:3306/test?useUnicode=true&characterEncoding=utf-8
spring.datasource.datasource1.username = root
spring.datasource.datasource1.password = rcs


spring.datasource.datasource2.driverClassName = com.mysql.jdbc.Driver
spring.datasource.datasource2.url = jdbc:mysql://10.180.8.173:8306/test?useUnicode=true&characterEncoding=utf-8
spring.datasource.datasource2.username = root
spring.datasource.datasource2.password = abcd123

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

**新建两个config文件用来配置两个不同的数据源**

记得把之前的MybatisMapperConfig删了，现在用新的config来指定扫的目录

**DataSource1Config1：**

```
package org.lxd.learn.boot.multi.config;

import javax.sql.DataSource;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

@Configuration
@MapperScan(basePackages="org.lxd.learn.boot.multi.datasource1.mapper",sqlSessionTemplateRef="datasource1SqlSessionTemplate")
public class DataSource1Config {
	
	 	@Bean(name = "datasource1DataSource")
	    @ConfigurationProperties(prefix = "spring.datasource.datasource1")
	    @Primary
	    public DataSource testDataSource() {
	        return DataSourceBuilder.create().build();
	    }

	    @Bean(name = "datasource1SqlSessionFactory")
	    @Primary
	    public SqlSessionFactory testSqlSessionFactory(@Qualifier("datasource1DataSource") DataSource dataSource) throws Exception {
	        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
	        bean.setDataSource(dataSource);
	        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/datasource1/*.xml"));
	        return bean.getObject();
	    }

	    @Bean(name = "datasource1TransactionManager")
	    @Primary
	    public DataSourceTransactionManager testTransactionManager(@Qualifier("datasource1DataSource") DataSource dataSource) {
	        return new DataSourceTransactionManager(dataSource);
	    }

	    @Bean(name = "datasource1SqlSessionTemplate")
	    @Primary
	    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("datasource1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
	        return new SqlSessionTemplate(sqlSessionFactory);
	    }
	
}

```

**DataSource1Config2：**

```
package org.lxd.learn.boot.multi.config;

import javax.sql.DataSource;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

@Configuration
@MapperScan(basePackages="org.lxd.learn.boot.multi.datasource2.mapper",sqlSessionTemplateRef="datasource2SqlSessionTemplate")
public class DataSource2Config {
	
	 	@Bean(name = "datasource2DataSource")
	    @ConfigurationProperties(prefix = "spring.datasource.datasource2")
	    public DataSource testDataSource() {
	        return DataSourceBuilder.create().build();
	    }

	    @Bean(name = "datasource2SqlSessionFactory")
	    public SqlSessionFactory testSqlSessionFactory(@Qualifier("datasource2DataSource") DataSource dataSource) throws Exception {
	        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
	        bean.setDataSource(dataSource);
	        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/datasource2/*.xml"));
	        return bean.getObject();
	    }

	    @Bean(name = "datasource2TransactionManager")
	    public DataSourceTransactionManager testTransactionManager(@Qualifier("datasource2DataSource") DataSource dataSource) {
	        return new DataSourceTransactionManager(dataSource);
	    }

	    @Bean(name = "datasource2SqlSessionTemplate")
	    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("datasource2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
	        return new SqlSessionTemplate(sqlSessionFactory);
	    }
	
}

```

以上配置完成

## 测试

同样使用generatorConfig来生成我们dao层的代码，对xml稍作调整  
**generatorConfig.xml：**

需要生成连接那个就解开哪个的注释

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
		
		<!-- source 1 -->
		<jdbcConnection driverClass="${spring.datasource.test1.driverClassName}"
		                connectionURL="${spring.datasource.test1.url}"
		                userId="${spring.datasource.test1.username}"
		                password="${spring.datasource.test1.password}">
		</jdbcConnection>
		
		
		
		
		<javaModelGenerator targetPackage="${targetModelPackage1}" targetProject="${targetJavaProject}"/>
		
		<sqlMapGenerator targetPackage="${targetXMLPackage1}"  targetProject="${targetResourcesProject}"/>
		
		<javaClientGenerator targetPackage="${targetMapperPackage1}" targetProject="${targetJavaProject}" type="XMLMAPPER" />
		
		<table tableName="t_dep" >
		  <generatedKey column="id" sqlStatement="Mysql" identity="true"/>
		</table>
		
		
		<!-- source 2 
		<jdbcConnection driverClass="${spring.datasource.test2.driverClassName}"
		                connectionURL="${spring.datasource.test2.url}"
		                userId="${spring.datasource.test2.username}"
		                password="${spring.datasource.test2.password}">
		</jdbcConnection>
		
		<javaModelGenerator targetPackage="${targetModelPackage2}" targetProject="${targetJavaProject}"/>
		
		<sqlMapGenerator targetPackage="${targetXMLPackage2}"  targetProject="${targetResourcesProject}"/>
		
		<javaClientGenerator targetPackage="${targetMapperPackage2}" targetProject="${targetJavaProject}" type="XMLMAPPER" />
		
		<table tableName="t_user" >
		  <generatedKey column="id" sqlStatement="Mysql" identity="true"/>
		</table>
		-->
		
	</context>
</generatorConfiguration>
```

点pom.xml 右键run as mybatis-generator:generate 回车执行  
注释掉一个数据源再执行一次，这样两个数据源的dao层生成完毕

写一个单元测试

```
package org.lxd.learn.boot.multi;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.lxd.learn.boot.multi.datasource1.entity.TDep;
import org.lxd.learn.boot.multi.datasource1.mapper.TDepMapper;
import org.lxd.learn.boot.multi.datasource2.entity.TUser;
import org.lxd.learn.boot.multi.datasource2.mapper.TUserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class BootMApplicationTests {

	@Test
	public void contextLoads() {
	}

	
//	@Autowired
//	TrPackageBatchMapper trPackageBatchMapper;
//	
//	@Test
//	public void test1(){
//		System.out.println(trPackageBatchMapper.selectAll().size());
//	}
	
	@Autowired
	TDepMapper tDepMapper;
	
	@Autowired
	TUserMapper tUserMapper;
	
	@Test
	public void test1(){
		TDep dep = new TDep();
		dep.setDepName("tt");
		tDepMapper.insert(dep);
		
		TUser user = new TUser();
		user.setName("ben");
		tUserMapper.insert(user);
		
		System.out.println(tDepMapper.selectAll().size());
		System.out.println(tUserMapper.selectAll().size());
	}
	
}

```

测试完毕！

demo再此：[https://github.com/ambluse/SpringCloud-learn/tree/master/springboot%20chapter3/boot-multi](https://github.com/ambluse/SpringCloud-learn/tree/master/springboot%20chapter3/boot-multi)