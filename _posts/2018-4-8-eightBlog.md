---
layout: post
title: Mybatis-generator生成器初步学习!
---
mybatis-generator是Mybatis的一款生成代码的工具，利用他可以自动将数据库中的表生成对应的实体model,接口mapper和sqlmapper,免去开发过程中的一些重复而繁杂的工作。
# 在Ecplise中使用
1. 在Ecplise的Marketplace中搜索mybatis generator即可找到该插件，点击下载即可。
2. 安装完成后，新建项目，选择other,然后在弹出框中选择Mybatis文件夹里面的MyBatis Generator Configuration File
![](https://ruanwenjun.github.io/images/01.png)

3. 点击next之后Location即为生成的配置文件所在的项目
4. 书写generatorConfig.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration 
PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" 
"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
	<!--targetRuntime是 指定值，具体看文档，id随意指定，不重复即可 -->
  <context id="DB2Tables" targetRuntime="MyBatis3">
  
  	<commentGenerator>
		<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
		<property name="suppressAllComments" value="true" />
	</commentGenerator>
	<!--数据库连接的一些信息  -->
    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
        connectionURL="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8"
        userId="root"
        password="317287">
    </jdbcConnection>
	<!--是否将数据库一些大数类型用java中的bigDecimal表示  -->
    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>
	<!--实体生成，targetPackage是包名，targetProject是路径  -->
    <javaModelGenerator targetPackage="cn.ruanwenjun.domain" targetProject="mybatis-generator\src">
      <property name="enableSubPackages" value="true" />
      <!--是否去掉前面后面的空格  -->
      <property name="trimStrings" value="true" />
    </javaModelGenerator>

    <sqlMapGenerator targetPackage="cn.ruanwenjun.mapper"  targetProject="mybatis-generator\src">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>
	<!--这里type也是指定的名称，具体看文档  -->
    <javaClientGenerator type="XMLMAPPER" targetPackage="cn.ruanwenjun.mapper"  targetProject="mybatis-generator\src">
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>
	<!--需要转换的数据库中的表  -->
    <table schema="" tableName="user"  > </table>
    <table schema="" tableName="orders"  > </table>
  </context>
</generatorConfiguration>
```

5. 书写完毕之后右键该配置文件，Run as ->点击小鸟，然后控制台出现运行成功，项目中多了两个包（注意：如果没有导入数据库驱动包，那么就会报错）
6. 书写SqlMapConfig.xml

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties resource="jdbc.properties"/>
	<typeAliases>
		<!-- <package name="cn.ruanwenjun.domain"/> -->
		<typeAlias type="cn.ruanwenjun.domain.User" alias="user"/>
	</typeAliases>
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
	<mappers>
		<package name="cn.ruanwenjun.mapper"/>
	</mappers>
</configuration>

```
还有书写jdbc.properties数据库连接信息的文件
7. 书写测试代码

```java
package cn.ruanwenjun.junit;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;
import cn.ruanwenjun.domain.User;
import cn.ruanwenjun.domain.UserExample;
import cn.ruanwenjun.mapper.UserMapper;

/**
 * @author ruanwenjun E-mail:861923274@qq.com
 * @date 2018年4月8日 上午11:00:20
*/
public class Demo {
	@Test
	public void testGenerator() {
		
		String source = "SqlMapConfig.xml";
		InputStream in = null;
		try {
			in = Resources.getResourceAsStream("SqlMapConfig.xml");
		} catch (IOException e) {
			e.printStackTrace();
		}
		SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
		SqlSessionFactory sqlSessionFactory = builder.build(in);
		SqlSession session = sqlSessionFactory.openSession();
		UserMapper mapper = session.getMapper(UserMapper.class);
		UserExample example = new UserExample();
		example.createCriteria().andIdBetween(1, 20);
		List<User> list = mapper.selectByExample(example);
		System.out.println(list.toString());
	}
}

```
运行成功

---
关于Example还有许多用法，涵盖了基本的根据ID进行CURD还有添加条件等，功能十分强大，不过如果需要关联查询可能还是需要取mapper里面自己书写SQL语句
















