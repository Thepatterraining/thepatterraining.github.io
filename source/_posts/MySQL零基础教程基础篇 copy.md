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

```

### DML

#### 插入数据

insert  values 
```SQL
insert into table_name[(col_name)] values ();
```

insert  set
```SQL
insert into table_name
set col_name = '值', col_name = '值';
```

insert  select
```SQL
insert into table_name
select * from table_name;
```

#### 删除数据

```SQL
delete from table_name where id = 1
```

#### 修改数据

```SQL
update table_name set col_name = '值' where id = 1

```

### 数据查询

#### select 语句

select * from table_name

#### 列的选择与指定

select col_name from table_name

#### 定义别名

select col_name as alias from table_name

#### 替换查询结果集中的数据

```SQL
case 
when 条件1 then 表达式1
when 条件2 then 表达式2
else 表达式
end as alias
```

#### 计算列值

select col_name + 100 from table_name

#### from 子句与多表连接查询

##### 交叉连接，笛卡尔积

select * from table_namme1 cross join table_name2

简写：
select * from table_name1,table_name2;

##### 内连接

select col_name from table_name inner join table_name2 on table_name.id = table_name2.t_id;

##### 外连接

left join

right join

### 数据查询

#### where 子句和条件查询

between 2 and 4 包含2,4

in (1,2,4)

is null

is not null

##### 子查询

表子查询
行子查询
列子查询
标量子查询

比较运算符包括
- ALL
- SOME
- ANY

结合exists

###### group

group by id asc|desc with rollup

##### having

group by id having count(*) < 3

##### order

order by id asc|desc


##### group 和 order的差别

|group | order|
|:----|:-------|
|分组行，但输出可能不是分组的排序|排序产生的输出|
|只能使用选择列或表达式列| 任意列都可以使用|
|若与聚合函数一起使用列或表达式, 则必须使用group| 不一定需要|


##### limit

limit 1,10



### DDL

#### 数据库模式定义

创建数据库
使用 `CREATE DATABASE` 或 `CREATE SCHEMA`语句

```SQL
CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name
[DEFAULT] CHARACTER SET[=]charset_name
| [DEFAULT] COLLATE[=]collation_name
```

查看数据库

```SQL
SHOW {DATABASES | SCHEMAS}
[LIKE pattern | WHERE expr] 
```

选择数据库

```SQL
use db_name
```

修改数据库

```SQL
ALTER DATABASE db_name
DEFAULT CHARACTER SET gb2312
DEFAULT COLLATE gb2312_chinese_ci;
```

删除数据库

```SQL
DROP {DATABASE|SCHEMA} [IF EXISTS] db_name;
```

#### 连接数据库

mysql -u root -p 

#### 创建数据库

create database my_test;

#### 查看数据库

show databases;

#### 删除数据库

drop my_test;

#### 进入数据库

use my_test;

#### 表定义

创建表

数据表被定义为字段的集合
按 `行` 和`列`的格式存储
每一行代表一条记录
每一列代表记录中一个字段的取值

```SQL
create [temporary] table tbl_name
(
    字段名1 数据类型 [列完整性约束条件] [默认值]
)
```


修改表

```SQL
ALTER TABLE table_name
```

字句
- ADD [COLUMN] 子句
- change [COLUMN] 子句
- alter [column] 子句 修改或删除表中指定列的默认值
    - alter colum city set default 'bj'
- modify [column] 子句 只修改指定列的数据类型，不会干涉它的列名
    - modify column city char(50);
- drop [column] 子句 删除指定列
    - drop column city;
- rename [to] 子句 修改表名
    - rename to table_name
- add index index_name(column_name) 创建索引
- drop index index_name


修改表名：
rename table table_name to new_table_name


删除表

```SQL
drop [temporary] table [if exists]

```

查看表

```SQL
show colums from table_name;
```

#### 索引定义

索引时提高数据文件访问效率的有效方法

缺点
- 索引是以文件的形式存储的，如果有大量的索引，索引文件可能比数据文件更快达到最大的文件尺寸
- 索引在提高查询速度的同时，会降低更新表的速度


- index 或 key 普通索引
- unique 唯一性索引 候选码
- primary key 主键

创建索引

```SQL
create [unique] index index_name
on table_name(index_col_name)
```

索引删除

drop index index_name on table_name



