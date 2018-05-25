---
layout: post
title: Mybatis的初步学习
---
* [入门案例](#入门案例)
* [SqlSessionFactory](#sqlsessionfactory)
* [sqlSession](#sqlsession)
* [SQL语句映射文件](#sql语句映射文件)
* [作用域和生命周期](#作用域和生命周期)

 Mybatis是一款优秀的持久层框架

要使用Mybatis需要一个mubatis-x.x.x.jar包，当然还有一些依赖包，例如数据库驱动，日志处理等
。

---
## 入门案例：

1.在Ecplise中创建一个JAVA工程，导入mybatis包和mysql连接驱动包。
2.编写jdbc.properties数据库连接配置文件

```java
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8
jdbc.username=root
jdbc.password=317287
```

3.数据库建表，建一个user表，同时在项目中创建一个User实体类
4.编写主配置文件

```java
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties resource="jdbc.properties"/>
	<!-- 和spring整合后 environments配置将废除    -->
	<environments default="development">
		<environment id="development">
			<!-- 使用jdbc事务管理 -->
			<transactionManager type="JDBC" />
			<!-- 数据库连接池 -->
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
	</environments>
	<!--加载sql映射文件-->
	<mappers>
		<mapper resource="cn/ruanwenjun/sqlmap/UserMapper.xml" />
	</mappers>
</configuration>
```

5.log4j日志记录文件 log4.properties

```java
# Global logging configuration
log4j.rootLogger=DEBUG, stdout
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n

```

6.sql语句映射文件

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 写Sql语句   -->
<mapper namespace="test">
	<select id="findUserById" parameterType="Integer" resultType="cn.ruanwenjun.domain.User">
		select * from user where id = #{v}
	</select>
	<select id="addUser" parameterType="cn.ruanwenjun.domain.User" >
		insert into user values(#{id},#{username},#{birthday},#{sex},#{address})
	</select>
	<select id="updateUser" parameterType="cn.ruanwenjun.domain.User" >
		update user set username=#{username},sex=#{sex},address=#{address},birthday=#{birthday} where id =#{id}
	</select>
	<select id="likeSelectUserByUsername" parameterType="String" resultType="cn.ruanwenjun.domain.User">
		select * from user where username like "%"#{v}"%"
	</select>
</mapper>
```

7.测试代码

```java
public class Demo {
	@Test
	//查找
	public void findUserById() {
		try {
			String resource = "SqlMapConfig.xml";
			InputStream in = Resources.getResourceAsStream(resource);
			SqlSessionFactoryBuilder ssfb = new SqlSessionFactoryBuilder();
			SqlSessionFactory ssf = ssfb.build(in);
			SqlSession session = ssf.openSession();
			User user = session.selectOne("test.findUserById", 1);
			System.out.println(user);
			
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
输出结果正常。
```

---

## SqlSessionFactory
SqlSessionFactory是Mybatis中十分重要的工厂，是一个接口，有两个实现类，每一个基于Mybatis的应用都要依靠SqlSessionFactory,可以从以下方式来构建SqlSessionFactory。

1.从XML配置文件中构建（文档上描述建议将该资源文件放在类路径下，即src下）

```java
String resource = "SqlMapConfig.xml";
InputStream inputStream= Resources.getResourceAsStream(resource);//Resources类是Mybatis自带的一个用于读取资源文件
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

2.通过创建Configuration来构建SqlSessionFactory（不推荐使用）

```java
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();//这里是得到一个数据库连接池，具体代码没有给出
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```
## sqlSession

也是一个接口，有两个实现类
```java
SqlSession session = sqlSessionFactory.openSession();  //这里得到的是sqlSession的两种实现类
```

SqlSession包含执行sql命令的方法，还有事务的回滚、提交。

里面还有一个重要的方法getMapper(Class clazz);
现在文档上面似乎是推荐使用mapper来使用

## SQL语句映射文件

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  
<mapper namespace="cn.ruanwenjun.dao.UserMapper">
    <select id="selectUserByUsername" resultType="cn.ruanwenjun.domain.User" parameterType="string">
		select * from user where username = #{v}
	</select>
</mapper>
```
有这个配置文件，那么就可以通过session来调用配置文件里的SQL语句
1. namespace:现在是必须的，建议将这写为mapper接口的类路径
2. id:写为接口中的方法
3. resultType:写为方法的返回值类型
4. parameterType:参数类型

## 作用域和生命周期
- SqlSessionFactoryBuilder 建议定义为局部方法变量
- SqlSessionFactory 建议一个应用只存在一个，使用单利模式或静态单例模式
- SqlSession 每次请求都应该打开一个新的session,因为sqlsession是线程不安全的，因此不能被共享。每次收到HTTP请求就应该打开一个Session,返回一个响应就应该关闭它。即可以放在类似ThredLocal作用域中。
- mapper 映射器是由sqlsession创建的，即任何映射器的最大作用域是和创建它的sqlsession相同的，建议放在方法内，即在方法内创建，用过之后就丢弃。
