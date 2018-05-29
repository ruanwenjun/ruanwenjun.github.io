---
layout: post
title: Spring JDBC学习
tags: [Spring]
---
目录：
- [Hello World](#helloworld)

> Spring 为我们提供了操作数据库的框架，对一些重复的代码使用模板方法进行了封装。同时Spring也可以很好的与其他的ORM框架整合

## Hello World

(1) 导包
```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.8-dmr</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
    <version>2.1.1</version>
</dependency>
 <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.0.2.RELEASE</version>
</dependency>
```

(2) 书写dao的配置文件

```
 <context:component-scan base-package="com.ruanwenjun.mvc.dao"/>
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mvc?characterEncoding=utf-8&amp;serverTimezone=UTC"/>
    <property name="username" value="root"/>
    <property name="password" value="root" />
    <property name="initialSize" value="5"/>
</bean>
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
   <constructor-arg name="dataSource" ref="dataSource"/>
</bean>
```

注意：这里如果在url后面不加上serverTimezone=UTC会报错，可能是后来版本的特殊要求。

(3) dao代码

```java
@Repository
public class SpittleDaoImpl implements SpittleDao {

    @Autowired
    private JdbcOperations jdbcOperations;

    private String INSERT_SPITTER = "INSERT INTO spitter " +
            "(username,password,firstname,lastname,picture) " +
            "VALUES (?,?,?,?,?)";

    @Override
    public void addSpitter(Spitter spitter) {
        jdbcOperations.update(INSERT_SPITTER,
                                spitter.getUsername(),
                                spitter.getPassword(),
                                spitter.getFirstName(),
                                spitter.getLastName(),
                                spitter.getPicture());
    }

}
```

使用spring之后极大的简少了代码量，并不需要去书写获得数据库连接，关闭数据库连接，以及开启事务关闭事务的重复代码。

这里JdbcOperations是JdbcTemplae的接口
