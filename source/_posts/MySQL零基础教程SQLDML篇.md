---
title: MySQL零基础教程基础篇
date: 2023-04-26 10:12:47
tags: ['数据库','数据库原理','sql','mysql']
category: CMU15445
article: MySQL零基础教程基础篇
---

# MySQL零基础教程基础篇

大家好，我是大头，98年，职高毕业，上市公司架构师，大厂资深开发，管理过10人团队，我是如何做到的呢？

这离不开持续学习的能力，而其中最重要的当然是数据库技术了！

对于所有开发来说，都离不开数据库，因为所有的数据都是要存储的。

关注我一起学习！文末有惊喜哦！

基础篇的内容大致如下图所示。

![概念学习](https://thepatterraining.github.io/images/mysql/mysql1-1.png)
 
## SQL语句

介绍完概念以后，我们可以来看看SQL语句了。SQL语句通常由三类组成。

数据定义 DDL
- CREATE 创建数据库或数据库对象
- ALTER 对数据库或数据库对象进行修改
- DROP 删除数据库或数据库对象
数据操纵 DML
- SELECT 从表或视图中检索数据
- INSERT 将数据插入到表或视图中
- UPDATE 修改表或视图中的数据
- DELETE 从表或视图中删除数据
数据控制 DCL
- GRANT 用于授予权限
- REVOKE 用于收回权限

### DML

MySQL 中的 DML（数据操纵语言，Data Manipulation Language） 语句用于对数据库中的数据进行操作，主要包括数据的插入、更新、删除和查询。DML 语句是数据库操作中最常用的语句类型。

#### 插入数据

插入数据使用`insert`指令，可以往创建好的一张表里面插入数据，支持多种插入方式。

最常用的方式是`insert  values`，这种方式也支持批量插入数据。

```SQL
insert into table_name[(col_name)] values ();
```

示例：

```SQL
INSERT INTO employees (id, name, department, salary)
VALUES (1, 'John Doe', 'IT', 50000);
```

还有一种方式是 `insert  set`。这种方式不支持批量插入。这种方式以键值对的形式插入数据，适用于插入单行数据。这种方式在插入单行数据时更加直观，尤其是当列名较多时。
```SQL
insert into table_name
set col_name = '值', col_name = '值';
```

还是刚才的插入示例：
```SQL
INSERT INTO employees (id, name, department, salary)
set id = 1，name = 'John Doe'，department='IT'，salary=50000；
```

还有一种方式是`insert  select`方式，这种方式适合快速复制表数据，将查询出来的数据插入到另外一个表里面。

```SQL
insert into table_name
select * from table_name;
```

#### 删除数据

可以使用`delete from`指令来删除已经插入的数据。如果不加where条件的话，就是删除全表数据。

删除数据操作一定要慎重！！！

```SQL
delete from table_name where id = 1
```

示例：删除刚才插入到表`employees`的`ID=1`的数据。
```SQL
delete from employees where id = 1
```

除了使用delete指令以外，还可以使用`TRUNCATE`指令，这个指令可以删除全表的数据，并且新的数据id自增从1开始。删除全表数据的话，该指令通常更快速。

```SQL
delete TABLE table_name
```

#### 修改数据

当插入的数据内容需要修改或者说更新的时候，则可以使用`update set`指令进行修改。修改操作可以使用where条件来选择要修改的数据，不加where条件则会更新所有数据。

```SQL
update table_name set col_name = '值' where id = 1
```

示例：将刚才插入的数据部门修改一下。

```SQL
update employees set department = 'Market' where id = 1
```

#### 数据查询

数据查询语句是最复杂的语句，这里只是介绍，想要完全用明白，需要大量的实践。

##### select 语句

select用于查询数据表里面插入的数据。

`*`代表查询所有字段。

```SQL
select * from table_name
```

##### 列的选择与指定

如果查询指定字段，则使用字段名称代替`*`。

实际开发中不推荐查询所有字段，推荐查询需要的字段，可以提升查询速度。
- 如果查询的字段正好是索引，那么可以触发覆盖索引
- 如果查询的字段过多，会增加网络传输消耗

```SQL
select col_name1,col_name2... from table_name
```

##### 定义别名

可以给字段和表定义别名，通过`as`指令实现。别名可以解决一些字段名冲突或者字段名过长的问题。

```SQL
select col_name as alias from table_name
```

示例：department字段给一个别名是depart
```SQL
select id，department as depart from employees
```

##### where条件

通过where关键字来进行条件筛选，可以选择出符合条件的数据。

比如当前用户表user有数据如下：以下数据均为随机生成，非真实数据。

| ID   | Name      | Gender | Mobile          | Email                     |
|------|-----------|--------|------------------|---------------------------|
| 001  | 张三      | 男     | 13800001234      | zhangsan@example.com      |
| 002  | 李四      | 女     | 13900005678      | lisi@example.com          |
| 003  | 王五      | 男     | 13700009012      | wangwu@example.com        |
| 004  | 赵六      | 女     | 13600003456      | zhaoliu@example.com       |
| 005  | 孙七      | 男     | 13500007890      | sunqi@example.com         |
| 006  | 周八      | 女     | 13400001234      | zhouba@example.com        |
| 007  | 吴九      | 男     | 13300005678      | wujiu@example.com         |
| 008  | 郑十      | 女     | 13200009012      | zhengshi@example.com      |
| 009  | 钱伯      | 男     | 13100003456      | qianbo@example.com        |
| 010  | 孔仲      | 女     | 13000007890      | kongzhong@example.com     |

这个时候我们需要查询出张三的用户信息，而不是将这10个数据都查询出来到程序里在筛选出张三的数据。

可以使用如下sql完成。
```sql
select * from user where name = '张三'; 
```

这个sql会把name字段中等于‘张三’的数据查询出来。

| ID   | Name      | Gender | Mobile          | Email                     |
|------|-----------|--------|------------------|---------------------------|
| 001  | 张三      | 男     | 13800001234      | zhangsan@example.com      |

where条件支持的类型如下：

- 比较操作符

| 操作符 | 描述 |
|--------|------|
| `=`    | 等于 |
| `<>`   | 不等于（也可用`!=`） |
| `>`    | 大于 |
| `<`    | 小于 |
| `>=`   | 大于等于 |
| `<=`   | 小于等于 |

- 逻辑操作符
| 操作符 | 描述 |
|--------|------|
| `AND`  | 逻辑与（两个条件都满足） |
| `OR`   | 逻辑或（至少一个条件满足） |
| `NOT`  | 逻辑非（对条件取反） |

- 范围操作符
| 操作符 | 描述 |
|--------|------|
| `BETWEEN...AND...` | 在指定范围内（包括边界值） |
| `NOT BETWEEN` | 不在指定范围内 |

- 列表操作符
| 操作符 | 描述 |
|--------|------|
| `IN`   | 在指定的列表中 |
| `NOT IN` | 不在指定的列表中 |

- 模糊匹配操作符
| 操作符 | 描述 |
|--------|------|
| `LIKE` | 模糊匹配（使用`%`和`_`作为通配符） |
| `NOT LIKE` | 不匹配指定模式 |

- 空值操作符
| 操作符 | 描述 |
|--------|------|
| `IS NULL` | 判断是否为`NULL` |
| `IS NOT NULL` | 判断是否不为`NULL` |

- 其他操作符
| 操作符 | 描述 |
|--------|------|
| `EXISTS` | 检查子查询是否存在结果 |
| `NOT EXISTS` | 检查子查询是否不存在结果 |

##### 替换查询结果集中的数据

可以使用if条件来进行结果的判定，比如性别，数据库里面存的可能是1代表男，2代表女。如果要查询出来男和女的话，就可以直接通过sql处理。

```SQL
case 
when 条件1 then 表达式1
when 条件2 then 表达式2
else 表达式
end as alias
```

示例：性别转换。
```SQL
SELECT 
    CASE 
        WHEN gender = 1 THEN '男'
        WHEN gender = 2 THEN '女'
        ELSE '未知'
    END AS 性别
FROM user;
```

#### 计算列值

可以直接计算将字段的值进行加减乘除运算。

```SQL
select col_name + 100 from table_name
```

#### from 子句与多表连接查询

##### 交叉连接，笛卡尔积

交叉连接可以连接两个表，产生两个表的笛卡尔积作为结果。

语法如下：
```SQL
select * from table_namme1 cross join table_name2
```
可以直接简写：
```SQL
select * from table_name1,table_name2;
```

示例：获取两个表的交叉连接。

假设有表user如下：

| ID   | Name      | Gender | Mobile          | Email                     |
|------|-----------|--------|------------------|---------------------------|
| 001  | 张三      | 男     | 13800001234      | zhangsan@example.com      |
| 002  | 李四      | 女     | 13900005678      | lisi@example.com          |

还有表user_account存储用户的账户余额信息如下：

| ID   | user_id      | balance |
|------|-----------|--------|
| 001  | 001      | 10     | 
| 002  | 002      | 20     | 

使用如下sql语句获取交叉连接：
```SQL
select * from user,user_account;
```

结果如下：也就是用户表数据001和用户账户001产生一条数据，和用户账户002产生一条数据，用户数据002同样。

| ID   | Name      | Gender | Mobile          | Email                     | ID2   | user_id      | balance |
|------|-----------|--------|------------------|---------------------------|------|-----------|--------|
| 001  | 张三      | 男     | 13800001234      | zhangsan@example.com      |001  | 001      | 10     | 
| 001  | 张三      | 男     | 13800001234      | zhangsan@example.com      |002  | 002      | 20     | 
| 002  | 李四      | 女     | 13900005678      | lisi@example.com          |001  | 001      | 10     | 
| 002  | 李四      | 女     | 13900005678      | lisi@example.com          |002  | 002      | 20     | 


##### 内连接

内连接返回两个表中匹配的记录。只有当两个表中的记录满足连接条件时，才会出现在结果集中。可以理解为两个表的交集。

连接的时候，`on`就类似于where条件，只不过仅仅在连接表数据的时候生效。内连接返回两个表都满足这个条件的交集。

语法：
```SQL
select col_name from table_name inner join table_name2 on table_name.id = table_name2.t_id;
```

示例：获取用户数据和用户账户数据的内连接。

假设有表user如下：

| ID   | Name      | Gender | Mobile          | Email                     |
|------|-----------|--------|------------------|---------------------------|
| 001  | 张三      | 男     | 13800001234      | zhangsan@example.com      |
| 002  | 李四      | 女     | 13900005678      | lisi@example.com          |
| 003  | 王五      | 男     | 13700009012      | wangwu@example.com        |
| 004  | 赵六      | 女     | 13600003456      | zhaoliu@example.com       |
| 005  | 孙七      | 男     | 13500007890      | sunqi@example.com         |
| 006  | 周八      | 女     | 13400001234      | zhouba@example.com        |
| 007  | 吴九      | 男     | 13300005678      | wujiu@example.com         |
| 008  | 郑十      | 女     | 13200009012      | zhengshi@example.com      |
| 009  | 钱伯      | 男     | 13100003456      | qianbo@example.com        |
| 010  | 孔仲      | 女     | 13000007890      | kongzhong@example.com     |

还有表user_account存储用户的账户余额信息如下：

| ID   | user_id      | balance |
|------|-----------|--------|
| 001  | 001      | 10     | 
| 002  | 002      | 20     | 
| 003  | 011      | 20     | 

可以看到这两个表的交集就只有两条数据，也就是001和002，

使用如下sql语句获取交叉连接：

`on user.id = user_account.user_id`这个条件代表只有当user表的id字段和user_account表的user_id字段相等的时候，才会有结果;
```SQL
select * from user inner join user_account on user.id = user_account.user_id;
```

结果如下：

| ID   | Name      | Gender | Mobile          | Email                     | ID2   | user_id      | balance |
|------|-----------|--------|------------------|---------------------------|------|-----------|--------|
| 001  | 张三      | 男     | 13800001234      | zhangsan@example.com      |001  | 001      | 10     | 
| 002  | 李四      | 女     | 13900005678      | lisi@example.com          |002  | 002      | 20     | 


##### 外连接

外连接分为左连接和右连接，左连接返回内连接的结果+左表剩余的数据，右连接返回内连接的结果+右表剩余的数据。

左表就是 `join`左边的表，右表就是`join`右边的表。

左连接使用`left join`指令。

```SQL
select col_name from table_name left join table_name2 on table_name.id = table_name2.t_id;
```

示例：获取用户表和用户账户表的左连接结果。

| ID   | Name      | Gender | Mobile          | Email                     |
|------|-----------|--------|------------------|---------------------------|
| 001  | 张三      | 男     | 13800001234      | zhangsan@example.com      |
| 002  | 李四      | 女     | 13900005678      | lisi@example.com          |
| 003  | 王五      | 男     | 13700009012      | wangwu@example.com        |
| 004  | 赵六      | 女     | 13600003456      | zhaoliu@example.com       |
| 005  | 孙七      | 男     | 13500007890      | sunqi@example.com         |
| 006  | 周八      | 女     | 13400001234      | zhouba@example.com        |
| 007  | 吴九      | 男     | 13300005678      | wujiu@example.com         |
| 008  | 郑十      | 女     | 13200009012      | zhengshi@example.com      |
| 009  | 钱伯      | 男     | 13100003456      | qianbo@example.com        |
| 010  | 孔仲      | 女     | 13000007890      | kongzhong@example.com     |

还有表user_account存储用户的账户余额信息如下：

| ID   | user_id      | balance |
|------|-----------|--------|
| 001  | 001      | 10     | 
| 002  | 002      | 20     | 
| 003  | 011      | 20     | 

使用如下sql语句，可以看到，仅仅是`inner join`换成了`left join`
```SQL
select * from user left join user_account on user.id = user_account.user_id;
```

结果如下：在内连接的结果基础上，增加了`左表user表`剩下的8条数据，右表的字段内容则是`null`，代表没有对应字段的数据。

| ID   | Name      | Gender | Mobile          | Email                     | ID2   | user_id      | balance |
|------|-----------|--------|------------------|---------------------------|------|-----------|--------|
| 001  | 张三      | 男     | 13800001234      | zhangsan@example.com      |001  | 001      | 10     | 
| 002  | 李四      | 女     | 13900005678      | lisi@example.com          |002  | 002      | 20     | 
| 003  | 王五      | 男     | 13700009012      | wangwu@example.com        |null | null      | null     | 
| 004  | 赵六      | 女     | 13600003456      | zhaoliu@example.com       |null | null      | null     | 
| 005  | 孙七      | 男     | 13500007890      | sunqi@example.com         |null | null      | null     | 
| 006  | 周八      | 女     | 13400001234      | zhouba@example.com        |null | null      | null     | 
| 007  | 吴九      | 男     | 13300005678      | wujiu@example.com         |null | null      | null     | 
| 008  | 郑十      | 女     | 13200009012      | zhengshi@example.com      |null | null      | null     | 
| 009  | 钱伯      | 男     | 13100003456      | qianbo@example.com        |null | null      | null     | 
| 010  | 孔仲      | 女     | 13000007890      | kongzhong@example.com     |null | null      | null     | 


右连接使用`right join`指令。
语法如下：
```SQL
select col_name from table_name right join table_name2 on table_name.id = table_name2.t_id;
```

示例：获取用户表和用户账户表的右连接结果。

| ID   | Name      | Gender | Mobile          | Email                     |
|------|-----------|--------|------------------|---------------------------|
| 001  | 张三      | 男     | 13800001234      | zhangsan@example.com      |
| 002  | 李四      | 女     | 13900005678      | lisi@example.com          |
| 003  | 王五      | 男     | 13700009012      | wangwu@example.com        |
| 004  | 赵六      | 女     | 13600003456      | zhaoliu@example.com       |
| 005  | 孙七      | 男     | 13500007890      | sunqi@example.com         |
| 006  | 周八      | 女     | 13400001234      | zhouba@example.com        |
| 007  | 吴九      | 男     | 13300005678      | wujiu@example.com         |
| 008  | 郑十      | 女     | 13200009012      | zhengshi@example.com      |
| 009  | 钱伯      | 男     | 13100003456      | qianbo@example.com        |
| 010  | 孔仲      | 女     | 13000007890      | kongzhong@example.com     |

还有表user_account存储用户的账户余额信息如下：

| ID   | user_id      | balance |
|------|-----------|--------|
| 001  | 001      | 10     | 
| 002  | 002      | 20     | 
| 003  | 011      | 20     | 

使用如下sql语句，可以看到，仅仅是`left join`换成了`right join`
```SQL
select * from user right join user_account on user.id = user_account.user_id;
```

结果如下：在内连接的结果基础上，增加了`右表user_account表`剩下的1条数据，左表的字段内容则是`null`，代表没有对应字段的数据。

| ID   | Name      | Gender | Mobile          | Email                     | ID2   | user_id      | balance |
|------|-----------|--------|------------------|---------------------------|------|-----------|--------|
| 001  | 张三      | 男     | 13800001234      | zhangsan@example.com      |001  | 001      | 10     | 
| 002  | 李四      | 女     | 13900005678      | lisi@example.com          |002  | 002      | 20     | 
| null  | null      | null     | null      | null        |003  | 011      | 20     | 

##### 子查询

在MySQL中，子查询是一种强大的功能，允许在一个查询中嵌套另一个查询。根据子查询返回的结果类型，可以将其分为以下几种：
- 表子查询
- 行子查询
- 列子查询
- 标量子查询

> 注意：所有的子查询应该慎重使用，因为子查询会导致查询速度降低。

| 子查询类型 | 定义 | 特点 | 示例 |
|------------|------|------|------|
| **表子查询** | 返回一个完整的表（多行多列） | 通常用于`FROM`子句或`JOIN`操作中，结果是一个表结构 | ```sql <br>SELECT * FROM (SELECT id, name FROM users) AS subquery;<br> ``` |
| **行子查询** | 返回一行数据（多列） | 通常用于`WHERE`子句中，结果是一行数据，可以与多列比较 | ```sql <br>SELECT * FROM users WHERE (id, name) = (SELECT id, name FROM users WHERE age = 25);<br>``` |
| **列子查询** | 返回一列数据（多行） | 通常用于`WHERE`子句中，结果是一列数据，可以与`IN`、`ANY`、`ALL`等操作符配合使用 | ```sql<br>SELECT * FROM users WHERE id IN (SELECT id FROM orders);<br>``` |
| **标量子查询** | 返回单个值（一行一列） | 通常用于`WHERE`子句中，结果是一个单一值，可以与比较操作符配合使用 | ```sql<br>SELECT * FROM users WHERE age = (SELECT MAX(age) FROM users);<br>``` |

###### 表子查询

- 定义：返回一个完整的表（多行多列）。
- 特点：可以作为虚拟表使用，通常用于FROM子句或JOIN操作中。
- 示例：SELECT id, name FROM users 这就是一个子查询，该子查询返回的结果是一张表的数据，将该子查询的结果作为一张表，供外部的查询使用。

```sql 
SELECT * 
FROM (SELECT id, name FROM users) AS subquery;
``` 

###### 行子查询

- 定义：返回一行数据（多列）。
- 特点：结果是一行数据，可以与多列比较，通常用于WHERE子句中。
- 示例：SELECT id, name FROM users WHERE mobile = "13012345678" 是一个子查询，该子查询返回了mobile字段等于13012345678的一行数据，并且只查询了id和name字段。将这两个字段作为外部查询的where条件。
```sql 
SELECT * 
FROM users 
WHERE (id, name) = (SELECT id, name FROM users WHERE mobile = "13012345678");
```

###### 列子查询

- 定义：返回一列数据（多行）。
- 特点：结果是一列数据，可以与IN、ANY、ALL等操作符配合使用，通常用于WHERE子句中。
- 示例：SELECT user_id FROM orders 是一个子查询，该子查询返回了orders表的所有用户id信息，并将这些用户id作为外部查询的where条件。
```sql 
SELECT * 
FROM users 
WHERE id IN (SELECT user_id FROM orders);
```

###### 标量子查询

- 定义：返回单个值（一行一列）。
- 特点：结果是一个单一值，可以与比较操作符配合使用，通常用于WHERE子句中。
- 示例：SELECT MAX(age) FROM users 是一个子查询，该子查询返回了users表的最大的年龄信息，并将最大的用户年龄作为外部查询的where条件。
```sql 
SELECT * 
FROM users 
WHERE age = (SELECT MAX(age) FROM users);
```

##### group by

- group语句可以实现分组的效果，什么是分组？

假设该数据表中存储了10条订单信息，有3条是张三的，3条是李四的，剩下4条是王五的。

group分组以后就可以分成3组，一组是张三的3条数据，一组是李四的3条数据，一组是王五的4条数据。

- 分组能干什么？

分组以后可以统计每个分组中的订单数量、订单总额、订单平均金额等。

语法

```sql 
SELECT * 
FROM table_name 
group by col_name
```

支持的聚合函数：
- count(col_name): 计算每个分组中该字段的数量，比如订单数量
- sum(col_name): 计算每个分组中该字段的总额，比如订单总金额
- avg(col_name): 计算每个分组中该字段的平均值，比如订单平均金额
- min(col_name): 获取每个分组中该字段的最小值
- max(col_name): 获取每个分组中该字段的最大值

有人要问了？那我不使用`group by`的情况下，可以使用上面的聚合函数吗？

当然可以了，没有分组，其实相当于所有数据是一个大分组，所以计算的是所有数据数量、总额等。

示例：下表是订单表，记录了3个用户的订单信息，现在需要查询这3个用户的订单数量、订单总金额、订单平均金额、最小金额以及最大金额。

| OrderID | UserID | OrderDate   | OrderAmount | OrderStatus |
|---------|--------|-------------|-------------|-------------|
| 1001    | 1      | 2025-02-01  | 120.00      | Completed   |
| 1002    | 1      | 2025-02-02  | 85.00       | Pending     |
| 1003    | 1      | 2025-02-03  | 230.00      | Shipped     |
| 1004    | 2      | 2025-02-04  | 150.00      | Completed   |
| 1005    | 2      | 2025-02-05  | 90.25       | Pending     |
| 1006    | 2      | 2025-02-06  | 110.00      | Shipped     |
| 1007    | 3      | 2025-02-07  | 100.00      | Completed   |
| 1008    | 3      | 2025-02-08  | 200.00      | Pending     |
| 1009    | 3      | 2025-02-09  | 130.75      | Shipped     |
| 1010    | 3      | 2025-02-10  | 160.00      | Completed   |

使用如下sql：对userID进行分组，就可以分成三组数据了，在对每个分组使用聚合函数。

```sql 
SELECT UserID, count(OrderID), sum(OrderAmount), avg(OrderAmount), min(OrderAmount), max(OrderAmount)
FROM orders 
group by UserID
```

结果如下：

| UserID | count(OrderID) | sum(OrderAmount)   | avg(OrderAmount) | min(OrderAmount) | max(OrderAmount) |
|---------|--------|-------------|-------------|-------------|-------------|
| 1    | 3      | 435.00  | 145.00      | 85.00   | 230.00 |
| 2    | 3      | 350.25  | 116.75       | 90.25      |150.00      | 
| 3    | 4      | 590.75  | 147.68      | 100.00     |200.00 | 

##### having

`having`语句用来过滤`group by`分组以后的数据。

简单点说，就是相当于where条件，只不过`where条件`的执行顺序在`group by`之前，having条件的执行顺序在group by之后。

语法如下：
```sql 
SELECT * 
FROM table_name 
group by col_name [having col_name = 任何数]
```

示例：还是上面group by的表，这次我们只需要总金额大于400的数据，从上面的结果来看，我们知道，只需要userId为1和3的数据。

但是注意，`where条件`是在`group by`之前执行，这个时候还没有总金额这个字段呢。所以，就需要使用`having`了。

使用的sql如下：可以看到，仅仅是在后面增加了`having sum(OrderAmount) > 400`这一条。
```sql 
SELECT UserID, count(OrderID), sum(OrderAmount), avg(OrderAmount), min(OrderAmount), max(OrderAmount)
FROM orders 
group by UserID having sum(OrderAmount) > 400
```

结果如下：

| UserID | count(OrderID) | sum(OrderAmount)   | avg(OrderAmount) | min(OrderAmount) | max(OrderAmount) |
|---------|--------|-------------|-------------|-------------|-------------|
| 1    | 3      | 435.00  | 145.00      | 85.00   | 230.00 |
| 3    | 4      | 590.75  | 147.68      | 100.00     |200.00 | 

##### order by

如果想对查询出来的结果集进行排序，可以使用`order by`语句。

语法如下：asc代表升序，即1，2，3这种排序，desc代表降序，即3，2，1这种。默认是asc。
```sql 
SELECT * 
FROM table_name 
order by col_name [asc｜desc]
```

order by排序作用在group by分组之后，这意味着可以使用分组之后的聚合函数的结果进行排序，同时也意味着可以影响group by之后的数据。

示例：对上面group by之后的数据按照总金额进行降序排序。

```sql 
SELECT UserID, count(OrderID), sum(OrderAmount), avg(OrderAmount), min(OrderAmount), max(OrderAmount)
FROM orders 
group by UserID having sum(OrderAmount) > 400
order by sum(OrderAmount) desc
```

结果如下：

| UserID | count(OrderID) | sum(OrderAmount)   | avg(OrderAmount) | min(OrderAmount) | max(OrderAmount) |
|---------|--------|-------------|-------------|-------------|-------------|
| 3    | 4      | 590.75  | 147.68      | 100.00     |200.00 | 
| 1    | 3      | 435.00  | 145.00      | 85.00   | 230.00 |


##### group 和 order的差别

|group | order|
|:----|:-------|
|分组行，但输出可能不是分组的排序|排序产生的输出|
|只能使用选择列或表达式列| 任意列都可以使用|
|若与聚合函数一起使用列或表达式, 则必须使用group| 不一定需要|


##### limit

如果不想每次都查询数据表的全部数据，只想获取几条数据呢？比如分页功能，一页10条数据这种，就可以使用`limit`命令来实现。

语法如下：start代表开始的位置，end代表结束的位置。
```sql 
SELECT *
FROM orders 
limit [start, end]
```

比如表中有100条数据。获取1-10条数据就是`limit 1,10`，获取11-20条数据就是`limit 11,20`。

limit最好是配合order by使用。性能更佳，另外，如果只获取1条数据，也建议使用`limit 1`代表获取1条数据。

具体的原因在后面原理篇会讲到。




