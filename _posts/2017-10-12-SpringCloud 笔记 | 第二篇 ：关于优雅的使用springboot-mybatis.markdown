---
layout:     post
title:      "SpringCloud 笔记 | 第二篇"
subtitle:   "优雅使用springboot"
date:       2017-10-12
author:     "BENJAMIN"
header-img: ""
tags:
    - SpringCloud-learn 系列
   
---


> Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。用我的话来理解，就是spring boot其实不是什么新的框架，它默认配置了很多框架的使用方式，就像maven整合了所有的jar包，spring boot整合了所有的框架

---

为了准备springcloud后续的教程我们需要先搭建一个spring boot的脚手架项目

## 选型

涉及到选型的地方只有数据库的部分，orm框架的本质是简化编程中操作数据库的编码，发展到现在基本上就剩两家了，一个是号称不需要些SQL的JPA派，一个是灵活方便的mybatis，各有特点，由于我们的系统比较灵活多变(设计很烂)，目前我们选择mybatis来作为我们的数据库访问组件。

网上搜一下关于mybatis和springboot的整合各种各样，都不是我心目中最简单的方式，看了心累，现在我们来看看应该如何优雅的使用springboot。
 
## mybatis-spring-boot-starter

mybatis初期使用比较麻烦，需要各种配置文件、实体类、dao层映射关联、还有一大推其它配置。当然mybatis也发现了这种弊端，初期开发了generator可以根据表结果自动生产实体类、配置文件和dao层代码，可以减轻一部分开发量；后期也进行了大量的优化可以使用注解了，自动管理dao层和配置文件等，发展到最顶端就是今天要讲的这种模式了，mybatis-spring-boot-starter就是springboot+mybatis可以完全注解不用配置文件，也可以简单配置轻松上手。

### 开始
同样从start.spring.io 生成项目，下载下来导入到STS中。

在pom.xml中增加mybatis starter的依赖

```
		<dependency>
  			<groupId>org.mybatis.spring.boot</groupId>
  			<artifactId>mybatis-spring-boot-starter</artifactId>
  			<version>1.3.1</version>
		</dependency>
```
通过如下链接可以找到最新的mybatis starter最新版本是1.3.1，如果有新的就用新的好了

http://maven.aliyun.com/nexus/#nexus-search;quick~mybatis-spring-boot-starter

为了测试再增加web的依赖(即springmvc)

```
		<dependency>
  			<groupId>org.springframework.boot</groupId>
  			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
```
org.springframework.boot下的组件的版本不需要关心，跟着springboot的parent依赖就好了

配置文件最终如下

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>org.lxd.learn</groupId>
	<artifactId>springboot-demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>springboot-demo</name>
	<description>Demo project for Spring Boot</description>

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
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
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
		
		
		<dependency>
  			<groupId>org.mybatis.spring.boot</groupId>
  			<artifactId>mybatis-spring-boot-starter</artifactId>
  			<version>1.3.1</version>
		</dependency>
		
		
		<dependency>
  			<groupId>org.springframework.boot</groupId>
  			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>


```

ok依赖的类库就这么解决了，springboot就是这么优雅

### 配置application.properties

```

mybatis.type-aliases-package=org.lxd.learn.springbootdemo.entity

spring.datasource.driverClassName = com.mysql.jdbc.Driver
spring.datasource.url = jdbc:mysql://10.180.8.205:3306/dmc_coupon?useUnicode=true&characterEncoding=utf-8
spring.datasource.username = root
spring.datasource.password = Pass1q2w

```

mybatis.type-aliases-package是数据库映射的po对应的位置

springboot会自动加载spring.datasource.*相关配置，数据源就会自动注入到sqlSessionFactory中，sqlSessionFactory会自动注入到Mapper中，对了你一切都不用管了，直接拿起来使用就行了。

### 配置mybatis
为了配置清晰，在建立一个config包，新建mybatis配置类，只要如下即可

```

package org.lxd.learn.springbootdemo.config;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan("org.lxd.learn.springbootdemo.mapper")
public class MybatisMapperConfig {

}
```

指定MapperScan就好了，是不是很简单。
以上就完成了所有的配置了


### 开发Mapper

这个就是业务开发了，在mapper的包下面新增对应的dao就好了

```
import java.util.List;

import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
import org.lxd.learn.springbootdemo.entity.TrPackageBatch;

public interface CouponMapper {
	
	@Select("SELECT * FROM tr_package_batch")
	List<TrPackageBatch> getAll();
	
	@Select("SELECT * FROM tr_package_batch WHERE rel_id = #{id}")
//	@Results({
//		@Result(property = "type",  column = "user_sex", javaType = UserSexEnum.class),
//		@Result(property = "name", column = "nick_name")
//	})
	TrPackageBatch getOne(Long id);

	@Insert("INSERT INTO tr_package_batch (package_id,batch_id,grand_count) VALUES(#{packageId}, #{batchId}, #{grandCount})")
	void insert(TrPackageBatch user);

	@Update("UPDATE tr_package_batch SET grand_count=#{grandCount} WHERE rel_id =#{id}")
	void update(TrPackageBatch user);

	@Delete("DELETE FROM tr_package_batch WHERE rel_id =#{id}")
	void delete(Integer id);

}
```

### 开测
上面三步就基本完成了相关dao层开发，使用的时候当作普通的类注入进入就可以了

写个测试类测试一下

```
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.lxd.learn.springbootdemo.entity.TrPackageBatch;
import org.lxd.learn.springbootdemo.mapper.CouponMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootDemoApplicationTests {

//	@Test
//	public void contextLoads() {
//	}
//	
	
	@Autowired
	private CouponMapper couponMapper;

	
	@Test
	public void testInsert() throws Exception {
		couponMapper.insert(new TrPackageBatch(1, 2, 3));
		couponMapper.insert(new TrPackageBatch(10, 20, 30));
		couponMapper.insert(new TrPackageBatch(11, 12, 13));

		Assert.assertEquals(3, couponMapper.getAll().size());
	}
}
```

执行测试 OK


## 简单的XML的配置方式
极简xml版本保持映射文件的老传统，优化主要体现在不需要实现dao的是实现层，系统会自动根据方法名在映射文件中找对应的sql.

### 配置
在application增加xml的配置：

```
mybatis.config-locations=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml

```
resource下增加couponMapper.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="org.lxd.learn.springbootdemoxml.mapper.CouponMapper" >
    <resultMap id="BaseResultMap" type="org.lxd.learn.springbootdemoxml.entity.TrPackageBatch" >
        <result column="REL_ID" property="relId" jdbcType="INTEGER" />
        <result column="BATCH_ID" property="batchId" jdbcType="INTEGER" />
        <result column="PACKAGE_ID" property="packageId" javaType="INTEGER"/>
        <result column="GRANT_COUNT" property="grantCount" jdbcType="INTEGER" />
    </resultMap>
    
    <sql id="Base_Column_List" >
        REL_ID, BATCH_ID, PACKAGE_ID, GRANT_COUNT
    </sql>

    <select id="getAll" resultMap="BaseResultMap"  >
       SELECT 
       <include refid="Base_Column_List" />
	   FROM TR_PACKAGE_BATCH
    </select>

    <select id="getOne" parameterType="java.lang.Integer" resultMap="BaseResultMap" >
        SELECT 
       <include refid="Base_Column_List" />
	   FROM TR_PACKAGE_BATCH
	   WHERE REL_ID = #{rel_id}
    </select>

    <insert id="insert" parameterType="org.lxd.learn.springbootdemoxml.entity.TrPackageBatch" >
      INSERT INTO TR_PACKAGE_BATCH (PACKAGE_ID,BATCH_ID,GRANT_COUNT) VALUES (#{packageId}, #{batchId}, #{grantCount})
    </insert>
    
    <update id="update" parameterType="org.lxd.learn.springbootdemoxml.entity.TrPackageBatch" >
       UPDATE TR_PACKAGE_BATCH SET GRAND_COUNT=#{grantCount} WHERE REL_ID =#{id}
    </update>
    
    <delete id="delete" parameterType="java.lang.Integer" >
      DELETE FROM TR_PACKAGE_BATCH WHERE REL_ID =#{id}
    </delete>
</mapper>
```


原来的mapper类，就简单了：

```
package org.lxd.learn.springbootdemoxml.mapper;

import java.util.List;

import org.lxd.learn.springbootdemoxml.entity.TrPackageBatch;

public interface CouponMapper {
	
	List<TrPackageBatch> getAll();
	
	TrPackageBatch getOne(Long id);

	void insert(TrPackageBatch user);

	void update(TrPackageBatch user);

	void delete(Integer id);

}

```

其他一致，执行单元测试。


后面再写一篇配置稍微复杂但是建议在生产上使用的配法