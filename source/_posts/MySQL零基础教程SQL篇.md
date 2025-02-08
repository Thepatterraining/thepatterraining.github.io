---
title: MySQL零基础教程基础篇
date: 2023-04-26 10:12:47
tags: ['数据库','数据库原理','sql','mysql']
category: CMU15445
article: MySQL零基础教程基础篇
---

# MySQL零基础教程基础篇

大家好，我是大头，98年，职高毕业，做过上市公司架构师，做过大厂资深开发，管理过10人团队，我是如何做到的呢？

这离不开持续学习的能力，而其中最重要的当然是数据库技术了！

对于所有开发来说，都离不开数据库，因为所有的数据都是要存储的。

关注我一起学习！可获得系统性的学习教程、转码经验、技术交流、大厂内推等～

文末有惊喜哦！

基础篇的内容大致如下图所示。

![概念学习](../images/mysql/mysql1-1.png)

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

### DCL

DCL（Data Control Language）语句用于控制对数据库的访问权限，包括用户权限的授予和撤销。DCL语句主要涉及用户和角色的权限管理，确保数据库的安全性和数据的完整性。

- 授予权限（GRANT）
- 撤销权限（REVOKE）
- 设置用户密码（SET PASSWORD）
- 查看用户权限（SHOW GRANTS）

#### GRANT

- GRANT语句用于授予用户或角色特定的权限。
- 权限可以包括对表、视图、存储过程等的访问和操作权限。

语法
```sql
GRANT privilege_type ON object_name TO user_or_role;
```

示例：授予用户zhangsan对employees表的SELECT和INSERT权限
```sql
GRANT SELECT, INSERT ON employees TO 'zhangsan'@'localhost';
```

#### REVOKE

- REVOKE语句用于撤销用户或角色的特定权限。
- 撤销的权限可以是之前授予的任何权限。

语法如下：

```sql
REVOKE privilege_type ON object_name FROM user_or_role;
```

示例：撤销用户zhangsan对employees表的INSERT权限

```sql
REVOKE INSERT ON employees FROM 'zhangsan'@'localhost';
```

#### SET PASSWORD

- SET PASSWORD语句用于设置或更改用户的密码。

语法如下：

```sql
SET PASSWORD FOR user = 'new_password';
```

示例：设置用户zhangsan的新密码为new_password。

```sql
SET PASSWORD FOR 'zhangsan'@'localhost' = 'new_password';
```

#### SHOW GRANTS

- SHOW GRANTS语句用于查看用户的权限。

语法：
```sql
SHOW GRANTS FOR user;
```

示例如下：查看用户zhangsan的权限
```sql
SHOW GRANTS FOR 'zhangsan'@'localhost';
```

DCL主要是对于权限的控制，希望大家可以自己创建一个数据库a，在创建一个用户a,授予a用户a数据库的权限。进行练习。

### DDL

DDL（Data Definition Language，数据定义语言）是SQL语言的一部分，用于定义和修改数据库的结构。DDL语句主要涉及数据库、表、索引、视图等的创建、修改和删除操作。这些语句直接影响数据库的结构，但不会直接操作数据本身。

#### 数据库模式定义

##### 创建数据库

使用 `CREATE DATABASE` 语句，`IF NOT EXISTS`代表没有这个数据库，才会进行创建。如果已经有了，则不会再创建了。

`CHARACTER SET`是设置字符集，推荐设置为utf8mb4字符集，`COLLATE`则使用默认的就可以了。

这里说一下`utf8`和`utf8mb4`这两个字符集的区别。
- utf8字符集：在MySQL中实际上是一个有限的字符集，它只支持最多3字节的UTF-8字符。这意味着它不能存储所有可能的Unicode字符，特别是那些需要4字节表示的字符（如某些表情符号）。utf8字符集支持的Unicode范围是U+0000到U+FFFF，即基本多语言平面（BMP）。
- utf8mb4字符集支持完整的UTF-8字符集，包括4字节的字符。这意味着它可以存储所有可能的Unicode字符，包括表情符号和一些罕见的字符。utf8mb4字符集支持的Unicode范围是U+0000到U+10FFFF，即整个Unicode范围。

```SQL
CREATE {DATABASE} [IF NOT EXISTS] db_name
[DEFAULT] CHARACTER SET[=]charset_name
| [DEFAULT] COLLATE[=]collation_name
```

创建一个测试数据库

```sql
create database test;
```

##### 查看数据库

使用`show databases`命令可以查看所有的数据库。也包括一些MySQL自带的数据库。这些数据库存储了MySQL的元数据，具体的等到原理篇会讲到。

```SQL
SHOW {DATABASES}
[LIKE pattern | WHERE expr] 
```

##### 选择数据库

使用`use`指令➕数据库名称可以选择数据库，或者说进入数据库。只有先进入一个数据库，才能操作这个数据库里面的数据表等等。

除此之外，也可以在操作数据表的前面加上数据库名称，但是那样比较麻烦。

```SQL
use db_name
```

##### 修改数据库

使用`ALTER DATABASE`指令可以修改数据库。

```SQL
ALTER DATABASE db_name
DEFAULT CHARACTER SET gb2312
DEFAULT COLLATE gb2312_chinese_ci;
```

##### 删除数据库

当这个数据库不再使用的时候，可以通过`DROP DATABSE`指令来删除掉这个数据库。

`IF EXISTS`代表存在则删除，不存在就不会删除。和创建的时候那个指令正好相反。都是可选指令。

```SQL
DROP {DATABASE} [IF EXISTS] db_name;
```


#### 表定义

数据表被定义为字段的集合
按 `行` 和`列`的格式存储
每一行代表一条记录
每一列代表记录中一个字段的取值

##### 创建表

使用`create table`指令可以创建数据表，后面跟的是表名称和字段。

`temporary`表示临时表，临时表存放在内存中。

字段类型常用的如下：
- int类型，占11位，也可以设置为int(5)等，但是这个只影响展示，并不影响实际的存储。
- varchar/char类型，相当于字符串类型，varchar是可变长度的字符串，char是不可变长度的字符串。
- text类型，很不推荐使用该类型，会导致查询速度变慢，尽量使用varchar代替。
- timestamp时间戳类型，不推荐使用，因为该类型表示1970年到现在的秒数，最大只能到2038-01-19号，而现在已经2025年了。
- datetime类型，推荐使用这个代替时间戳，直接存储时间类型，并且表里的updated_time字段可以使用`DEFAULT CURRENT_TIMESTAMP`作为默认值，还可以使用`ON UPDATE CURRENT_TIMESTAMP`来实现自动更新。
- decimal类型,用来存储小数，使用定点小数来存储，可以防止精度丢失。请避免使用`float`和`double`来存储小数。
- json类型，可以存储json字符串。

```SQL
create [temporary] table [if not exists] tbl_name
(
    字段名1 数据类型 [列完整性约束条件] [默认值]
)
```

##### 修改表

对于创建的表结构不满意，可以通过`ALTER TABLE`指令来修改表结构。

```SQL
ALTER TABLE table_name
```

下面介绍一些子句，配合alter table命令来执行。
- ADD [COLUMN] 子句：给表结构增加字段。
- change [COLUMN] 子句：修改表结构的字段类型、字段名称等。
    - CHANGE COLUMN name new_name VARCHAR(200);
- alter [column] 子句 修改或删除表中指定列的默认值。
    - alter colum city set default 'bj'
- modify [column] 子句 只修改指定列的数据类型，不会干涉它的列名
    - modify column city char(50);
- drop [column] 子句 删除指定列
    - drop column city;
- rename [to] 子句 修改表名
    - rename table table_name to new_table_name
- add index index_name(column_name) 创建索引
- drop index index_name


##### 删除表

当一个表不再使用的时候，也可以使用`drop table`将它删除。

```SQL
drop [temporary] table [if exists]

```

##### 查看表

可以通过`SHOW CREATE TABLE`来查看表结构。

```SQL
SHOW CREATE TABLE tablename;
```

#### 索引定义

索引是提高数据文件访问效率的有效方法，比如MySQL中的B+树索引、hash索引、全文索引等。

缺点
- 索引是以文件的形式存储的，如果有大量的索引，索引文件可能比数据文件更快达到最大的文件尺寸
- 索引在提高查询速度的同时，会降低更新表的速度

##### 索引物理结构

- b+树索引
- hash索引
- 倒排索引

##### 索引逻辑结构

- index 或 key： 普通索引
- unique ：唯一性索引 候选码
- primary key： 主键

##### 索引逻辑概念

- 聚簇索引：比如主键索引，也就是b树的叶子节点存储数据的索引。
- 联合索引：由多个字段共同组成的索引。
- 覆盖索引：查询的字段和索引的字段一致，从而避免了再次去主键索引获取数据。

关于索引的具体讲解将放在原理篇讲解，这里以介绍DDL语句为主，有个概念就可以了。

##### 创建索引

想要创建一个索引可以使用`create index`命令，`unique`表示创建唯一索引。

`index_col_name`表示要将索引创建在哪个字段上面，也可以是多个字段。

```SQL
create [unique] index index_name
on table_name(index_col_name)
```

示例，在user表上的`email`字段上创建一个索引，索引名称是`email_idx`。

```SQL
create index email_idx
on user(email);
```

> 索引并不是越多越好，太多的索引会导致维护成本升高，尽量少且有用即可。尤其是后续增加索引的时候，如果数据表中数据过多，建立索引的过程会较慢，会对业务产生影响，这个时候需要慎重。

##### 索引删除

当索引不再使用的时候，可以删除索引，使用命令`drop index`可以删除索引。

```SQL
drop [unique] index index_name
on table_name
```

示例：删除刚才建立的`email_idx`索引。

```SQL
drop index email_idx
on user;
```

#### 视图定义

什么是视图
- 视图是一个对象，他是数据库提供给用户的以多种角度观察数据库中数据的一种重要机制
- 视图不是数据库中真实的表，而是一张虚拟表，其自身并不存储数据

视图的优点
- 集中分散数据
- 简化查询语句
- 重用SQL语句
- 保护数据安全
- 共享所需数据
- 更改数据格式

##### 创建视图

想要创建视图，可以使用`create view`指令。

`or replace` 防止报错，存在替换，不存在创建。
`with check option` 增删改查的时候检查视图条件。
`select_statement` 是一段select查询语句。视图的本质就是这一段select查询语句。

```SQL
create [OR REPLACE] view view_name [(col_list)]
as select_statement
[with check option]
```

示例：创建一个`zhangsan`用户的登录记录的视图。

```SQL
create view zhangsan
as select * from login_log where user = ‘zhangsan’
```

##### 修改视图

想要修改视图，可以使用`alter view`指令。

修改视图其实就是修改这个查询语句。当然了，也可以修改其他的属性等。

```SQL
alter view view_name [(col_list)]
as select_statement
[with check option]
```

##### 删除视图

当视图不需要了，可以使用`drop view`指令删除视图。

```SQL
DROP VIEW [IF EXISTS] view_name
```

示例：删除刚才创建的视图zhangsan

```SQL
DROP view zhangsan
```

##### 查看视图定义

和上面说的查看表的定义一样，也可以查看视图的定义。

```SQL
show create view view_name
```

#### 存储过程定义

`存储过程` 是一组为了完成某项特定功能的 `SQL语句集`
- 可增强SQL语言的功能和灵活性
- 良好的封装性
- 高性能
- 可减少网络流量
- 可作为一种安全机制来确保数据库的安全性和数据的完整性
其实质就是一段存储在数据库中的 `代码`
它可以由声明式的sql语句和过程式sql语句组成

##### 创建存储过程

`DELIMITER $$`是用户定义的MYSQL 结束符

参数：in|out|inout 参数名 参数类型

```SQL
DELIMITER $$
create procedure sp_name(参数)
BEGIN
body //存储过程代码
END $$
```

示例：查询员工表的名称、部门和薪资。

```sql
DELIMITER //

CREATE PROCEDURE get_employee_details(IN emp_id INT)
BEGIN
    SELECT name, department, salary
    FROM employees
    WHERE id = emp_id;
END //

DELIMITER ;
```

##### 调用存储过程

调用需要使用`call`指令来调用。

```sql
call sp_name(参数)
```

示例：调用刚才的存储过程。 
```sql
call get_employee_details(1);
```

##### 删除存储过程

如果存储过程不再需要了，则可以通过`drop procedure`指令来删除它。

```sql
drop procedure sp_name
```

示例：删除刚才的存储过程。

```sql
drop procedure get_employee_details;
```

#### 存储函数定义

存储函数由SQL语句和过程式语句组成。

##### 创建存储函数

使用 `create function`指令可以创建一个存储函数。

```SQL
create function sp_name(参数)
    returns type
    routine_body //主体
```

示例：给定id号返回性别。

```SQL
use test; //进入数据库test
delimiter $$ //声明结束符号
create function fn_search(cid int) //创建函数fn search，参数为cid，int类型
    returns char(20) //声明返回值类型char20
    deterministic
begin //开始
    declare sex char(20) //声明一个变量sex 类型char20
    select cust_sex into sex from customers where id = cid; //select语句，把查询出来的cust_sex字段内容放入变量sex中
    if sex is null then //if判断，如果sex变量是null，则返回'没有该客户'
        return(select '没有该客户');
    else //如果sex变量不是null
        if sex = 'F' then //则判断是F的话，返回'女'
            return(select '女');
        else // 不然的话就返回'男'
            return(select '男');
        end if;
    end if;
end $$
```

##### 调用存储函数

使用 select 调用存储函数。

```SQL
select sp_name(参数)；
```

示例：调用刚才的存储函数。

```SQL
select fn_search(1)$$
```

##### 删除存储函数

当存储函数不再使用的时候，可以使用`drop function`将它删除。

```SQL
drop function fun_name
```

示例：删除刚才的存储函数。
```SQL
drop function fn_search
```

## 文末福利

以上就是整体的MySQL学习路线了。

关注我发送“MySQL知识图谱”领取完整的MySQL学习路线。

发送“电子书”即可领取价值上千的电子书资源。

部分电子书如图所示。

![概念学习](../images/bottom1.png)

![概念学习](../images/bottom2.png)

![概念学习](../images/bottom3.png)

![概念学习](../images/bottom4.png)


