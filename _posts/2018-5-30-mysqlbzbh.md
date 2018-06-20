---
layout: post
title: Mysql必知必会总结
tags: [Mysql]
---
> 数据库是保存有组织的数据的容器

我们并不直接访问数据库，而是通过DBMS，它代替我们访问数据库

**检索数据**

- SELECT product_name FROM product
- SELECT product_name,product_price FROM product
- SELECT * FROM product

检索不同的行：
- SELECT DISTINCT product_price FROM product

注意：不能部分使用DISTINCE,例如:

SELECT DISTINCT product_name,product_price FROM product

查询到的是这两个组合的唯一而不是每一列或者product_name列的唯一

分页查询：
- SELECT product_name FROM pruduct LIMIT 5

将返回前5行，默认从第0行开始，如果想返回下一个5行

- SELECT product_name FROM product LIMIT 5,5

第一个数为开始位置，第二个数为检索行数

**排序**：

- SELECT product_name FROM product ORDER BY product_name

可以使用非检索的列来排序

按多个列排序：
- SELECT product_id,product_price,product_name FROM product ORDER BY product_price,product_name

排序按price name依次

指定排序的方向：
- 降序：DESC
- 升序：ASC

默认是升序，如果想在多个列上都使用降序，必须在每个列后面都加DESC

**过滤**：
使用WHERE子句

- 等于： =
- 不等于：！=
- 不等于：<>
- 小于：<
- 小于等于：<=
- 范围：BETWEEN * AND *
- 空值：IS NULL
- 非空：IS NOT NULL
- AND 操作符
- OR 操作符

通常有多个条件需要使用AND OR 时，带上括号

- IN 操作符

- SELECT product_name FROM product WHERE product_id IN(1002,1003)

- NOT 操作符
-  SELECT product_name FROM product WHERE product_id NOT IN(1002,1003)

**通配符：**

- LIKE操作符
- %
- _
- ==SELECT product_name FROM product WHERE product_name LIKE 'mac%'==
- SELECT product_name FROM product WHERE product_name LIKE 'macboo_'

_下划线只能匹配一个字符，%可以匹配任意字符

**创建计算字段**
- 拼接：Concat
- SELECT Concat(product_name,'(',product_id,')')

**聚集函数**：
- AVG() : 返回某列的平均值
- COUNT() : 返回某列的行数。如果指定列名，忽略NULL值；如果使用*则不忽略
- MAX() : 返回某列的最大值
- MIN() : 返回某列的最小值
- SUM() : 返回某列值之和

**分组数据**：
- SELECT product_name,COUNT(*) AS num FROM product GROUP BY product_name
- SELECT product_name,COUNT(*) AS num FROM product GROUP BY product_name HAVING COUNT( * ) >=2

**使用子查询**：
- SELECT cust_id FROM order WHERE order_id IN (SELECT order_id FROM orderitem WHERE product_id = 'mac')

**联结表**：
- 内联结：INNER JOIN ... ON
- 左外：LEFT JOIN ... ON
- 右外:RIGHT JOIN ... ON

**组合查询**:
- UNION

**全文本搜索**

**插入数据**
- INSERT INTO customer(cust_name,cust_address) VALUES ('wenjun','changsha')

**更新数据**
- UPDATE customer SET cust_address = 'wuhan' WHERE cust_name = 'wenjun'

**删除数据**
- DELECT FROM customer WHERE cust_name = 'wenjun'
