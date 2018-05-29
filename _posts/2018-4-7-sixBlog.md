---
layout: post
title: Spring简单整合Mybatis
tags: [Spring]
---
* [导包](#导包)
* [spring配置文件](#spring配置文件)
* [mybatis配置文件](#mybatis配置文件)
* [书写sqlmapper和接口](#书写sqlmapper和接口)
* [测试](#测试)

# 整合步骤
- 导包
- 书写spring配置文件----applicationContext.xml
- 书写mybatis配置文件----SqlMapperConfig.xml
- 书写SQL映射文件----xxxMapper.xml

  书写接口
- 测试

---
# 导包
# spring配置文件
applicationContext.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context-4.0.xsd
	http://www.springframework.org/schema/aop
	http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
	http://www.springframework.org/schema/tx
	http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	http://www.springframework.org/schema/util
	http://www.springframework.org/schema/util/spring-util-4.0.xsd">

	<!--读取配置jdbc连接的信息文件-->
	<context:property-placeholder location="classpath:dbcp.properties"/>
	<!--采用dbcp连接池-->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
	<!--这里的close指的是当bean被销毁的时候调用连接池的close方法-->
	destroy-method="close">
		<property name="driverClassName" value="${jdbc.driver}"/>
		<property name="url" value="${jdbc.url}"/>
		<property name="username" value="${jdbc.username}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>
	<!--配置sqlSessionFactory,该类在org.mybatis.spring包下，并不在之前的那个mybatis包-->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="configLocation" value="classpath:SqlMapperConfig.xml"/>
	</bean>
	<!--这里是一个一个的配置代理接口，使用比较繁琐-->
	<!-- <bean id="mapperFactoryBean" class="org.mybatis.spring.mapper.MapperFactoryBean">
		<property name="sqlSessionFactory" ref="sqlSessionFactory"/>
		<property name="mapperInterface" value="cn.ruanwenjun.mapper.UserMapper"/>
	</bean> -->
	<!--使用扫描的方式来扫描包下的所有接口-->
	<bean id="mapperScannerConfigure"
		class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	    <!--这里不要配置sqlSessionFactory，会报错，
	    	因为在该类下的注入sqlSessionFactory的方法已经过期了，可以配置一个
	    	sqlSessionFactoryBeanname的属性，但是不配置的话spring也会自己寻找-->
		<property name="basePackage" value="cn.ruanwenjun.mapper"/>
	</bean>
</beans>
```
dbcp.properties

```java
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8
jdbc.username=root
jdbc.password=317287

```

# mybatis配置文件
sqlMapperConfiger.xml

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--给实体类配置别名，那么在mapper文件中可以直接使用别名来代替繁琐的类全路
    径名，这里配置包别名，那么该包下的所有类可以使用第一个字母为小写的简单类名
    代替-->
	<typeAliases>
		<package name="cn.ruanwenjun.domain"/>
	</typeAliases>
	<!--映射接口的包，将会将该包下的接口都映射-->
	<mappers>
		<package name="cn.ruanwenjun.mapper"/>
	</mappers>
</configuration>
```
# 书写sqlmapper和接口
userMapper.xml
```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.ruanwenjun.mapper.UserMapper">
	<select id="selectUserById" resultType="user" parameterType="Integer">
		SELECT * from user where id =#{id}
	</select>
</mapper>
```
UserMapper.java

```java
package cn.ruanwenjun.mapper;
import cn.ruanwenjun.domain.User;
/**
 * @author ruanwenjun E-mail:861923274@qq.com
 * @date 2018年4月7日 上午10:23:18
*/
public interface UserMapper {
	public User selectUserById(Integer id);
}

```
User.java,对应数据库中的user表

```java
package cn.ruanwenjun.domain;
import java.io.Serializable;
import java.util.Date;

public class User implements Serializable {
	/**
	 *
	 */
	private static final long serialVersionUID = 1L;
	private Integer id;
	private String username;// 用户姓名
	private String sex;// 性别
	private Date birthday;// 生日
	private String address;// 地址


	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getUsername() {
		return username;
	}
	public void setUsername(String username) {
		this.username = username;
	}
	public String getSex() {
		return sex;
	}
	public void setSex(String sex) {
		this.sex = sex;
	}
	public Date getBirthday() {
		return birthday;
	}
	public void setBirthday(Date birthday) {
		this.birthday = birthday;
	}
	public String getAddress() {
		return address;
	}
	public void setAddress(String address) {
		this.address = address;
	}
	@Override
	public String toString() {
		return "User [id=" + id + ", username=" + username + ", sex=" + sex
				+ ", birthday=" + birthday + ", address=" + address + "]";
	}
}

```
# 测试
```java
package cn.ruanwenjun.junit;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import cn.ruanwenjun.domain.User;
import cn.ruanwenjun.mapper.UserMapper;

/**
 * @author ruanwenjun E-mail:861923274@qq.com
 * @date 2018年4月7日 上午10:36:55
*/
public class Demo {
	@Test
	@SuppressWarnings("all")
	public void test() {
		ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
		UserMapper userMapper = ac.getBean(UserMapper.class);
		User user = userMapper.selectUserById(1);
		System.out.println(user);
	}
}

```
输出正常

---
