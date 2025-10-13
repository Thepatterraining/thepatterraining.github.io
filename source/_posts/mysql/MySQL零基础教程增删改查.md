---
title: MySQL零基础教程增删改查实战
date: 2025-07-01 10:12:47
tags: ['数据库','数据库原理','sql','mysql']
category: mysql
article: MySQL零基础教程增删改查实战
---

> 大家好，我是大头，职高毕业，现在大厂资深开发，前上市公司架构师，管理过10人团队！
> 我将持续分享成体系的知识以及我自身的转码经验、面试经验、架构技术分享、AI技术分享等！
> 愿景是带领更多人完成破局、打破信息差！我自身知道走到现在是如何艰难，因此让以后的人少走弯路！
> 无论你是统本CS专业出身、专科出身、还是我和一样职高毕业等。都可以跟着我学习，一起成长！一起涨工资挣钱！
> 关注我一起挣大钱！文末有惊喜哦！

> 关注我发送“MySQL知识图谱”领取完整的MySQL学习路线。
> 发送“电子书”即可领取价值上千的电子书资源。
> 发送“大厂内推”即可获取京东、美团等大厂内推信息，祝你获得高薪职位。
> 发送“AI”即可领取AI学习资料。

# MySQL零基础教程

本教程为零基础教程，零基础小白也可以直接学习，有基础的可以跳到后面的原理篇学习。
基础概念和SQL已经更新完成。

接下来是应用篇，应用篇的内容大致如下图所示。

![应用学习](https://thepatterraining.github.io/images/mysql/mysql1-2.png)
 
## 增删改查实战

MySQL是最常用的关系型数据库之一，掌握基本的增、删、改、查（CRUD）操作是学习MySQL的基础。本篇文章将通过详细示例，帮助零基础的读者快速上手MySQL的增、删、改、查操作。无论是新手还是有一定数据库基础的开发者，都能从中受益。

## 1. MySQL环境准备

在进行任何操作之前，首先需要确保你已经成功安装了MySQL数据库，并能顺利连接到数据库。如果没有安装MySQL，请参考MySQL官方网站或相关教程进行安装。

### 连接到MySQL

在终端中使用以下命令登录MySQL：

```bash
mysql -u root -p
```

输入密码后，成功登录后会进入MySQL命令行界面。

### 创建数据库

在MySQL中，所有的操作都发生在数据库中，因此我们首先需要创建一个数据库：

```sql
CREATE DATABASE mydb;
```

然后进入到mydb数据库：

```sql
USE mydb;
```

### 创建表

在进行增删改查操作之前，先创建一个简单的表。假设我们要管理一个学生信息表，表结构如下：


```sql
CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    age INT,
    grade VARCHAR(10)
);
```

表结构说明：
- id：学生的唯一标识，使用AUTO_INCREMENT自增。
- name：学生的姓名，VARCHAR(50)表示最多存储50个字符。
- age：学生的年龄，使用INT表示整数类型。
- grade：学生的年级，VARCHAR(10)表示最多存储10个字符。

## 2. 增：插入数据（INSERT）

在MySQL中，使用INSERT INTO语句来插入数据。我们可以插入一条或多条记录。

### 插入一条记录
插入一条记录到students表：

```sql
INSERT INTO students (name, age, grade) 
VALUES ('John Doe', 20, 'Sophomore');

```

- `INSERT INTO students (name, age, grade)`：指定插入的数据列。
- `VALUES ('John Doe', 20, 'Sophomore')`：插入的数据值，注意顺序要和列名匹配。

### 插入多条记录
我们也可以一次性插入多条记录：

```sql
INSERT INTO students (name, age, grade) 
VALUES 
    ('Alice', 19, 'Freshman'),
    ('Bob', 21, 'Junior'),
    ('Charlie', 22, 'Senior');

```

- 通过多个VALUES列表插入多条记录，减少了插入的语句数量，提升了效率。

### 插入数据并返回自动生成的ID

如果表中有`AUTO_INCREMENT`字段（如id），可以使用`LAST_INSERT_ID()`函数获取插入后生成的ID：

```sql
INSERT INTO students (name, age, grade) 
VALUES ('David', 23, 'Graduate');
SELECT LAST_INSERT_ID();
```

## 3. 查：查询数据（SELECT）

MySQL使用`SELECT`语句从表中查询数据。可以查询全部列，也可以选择特定列，还可以进行条件过滤、排序等。

### 查询所有记录

查询`students`表中所有的数据：

```sql
SELECT * FROM students;
```

- *表示查询所有列的数据。

### 查询特定列

查询`name`和`age`列的数据：

```sql
SELECT name, age FROM students;
```

### 使用WHERE条件筛选数据

查询年纪大于20的学生：

```sql
SELECT * FROM students WHERE age > 20;
```

- WHERE用于添加条件，过滤符合条件的记录。

### 排序查询结果

按年龄升序排列学生：

```sql
SELECT * FROM students ORDER BY age ASC;
```

- `ASC`表示升序（默认），如果要降序排列，可以使用`DESC`：

```sql
SELECT * FROM students ORDER BY age DESC;
```

### 使用LIMIT限制返回结果数量

如果只想查看前两条记录：

```sql
SELECT * FROM students LIMIT 2;
```

### 模糊查询（LIKE）

使用`LIKE`进行模糊查询，查找名字中包含“a”的学生：

```sql
SELECT * FROM students WHERE name LIKE '%a%';
```

- %是通配符，表示任意字符。

### 聚合函数

MySQL提供了多种聚合函数，如`COUNT()`、`AVG()`、`SUM()`等。

查询学生人数：

```sql
SELECT COUNT(*) FROM students;

```

查询平均年龄：

```sql
SELECT AVG(age) FROM students;

```

## 4. 改：更新数据（UPDATE）

使用UPDATE语句修改表中已有的数据。可以根据条件更新特定记录，也可以批量更新。

### 更新单条记录

更新`id=1`的学生姓名为“John Smith”：

```sql
UPDATE students 
SET name = 'John Smith' 
WHERE id = 1;

```

- SET用于指定需要更新的列和新的值。
- WHERE用于指定要更新的记录，确保不修改所有记录。

### 更新多条记录

更新所有年龄大于20的学生年级为“Graduate”：

```sql
UPDATE students 
SET grade = 'Graduate' 
WHERE age > 20;

```

### 更新多个字段

同时更新多个字段：

```sql
UPDATE students 
SET age = 23, grade = 'Postgraduate' 
WHERE id = 2;

```

## 5. 删：删除数据（DELETE）

使用`DELETE`语句删除表中的记录，可以删除特定记录或删除所有记录。

### 删除单条记录

删除`id=1`的学生：

```sql
DELETE FROM students WHERE id = 1;
```

- WHERE子句用于指定删除条件，确保只删除满足条件的记录。

### 删除多条记录

删除所有年龄大于20的学生：

```sql
DELETE FROM students WHERE age > 20;
```

### 删除所有记录

删除`students`表中的所有记录（注意：表结构不变）：

```sql
DELETE FROM students;
```

### 删除表中的记录（TRUNCATE）

`TRUNCATE`语句与`DELETE`类似，但其删除的是所有数据并且不记录日志，因此效率更高：

```sql
TRUNCATE TABLE students;
```

- TRUNCATE会立即清空表中的所有数据，并且不能恢复。

### 删除表（DROP）

如果需要删除整个表，包括表结构：

```sql
DROP TABLE students;
```

- DROP会删除表以及表中所有的数据，无法恢复，因此要小心使用。

## 总结

在MySQL中，增、删、改、查（CRUD）操作是数据库管理的基础。通过上述操作，我们能够对数据库表中的数据进行增删改查，并且可以通过各种条件、排序和聚合功能来灵活查询数据。

这里，要特别注意`更新`和`删除`操作。一定要加条件，不然就会更新全表数据，或者删除全表数据。大家有没有误操作过呢？一起分享下。

## 文末福利

> 关注我发送“MySQL知识图谱”领取完整的MySQL学习路线。
> 发送“电子书”即可领取价值上千的电子书资源。
> 发送“大厂内推”即可获取京东、美团等大厂内推信息，祝你获得高薪职位。
> 发送“AI”即可领取AI学习资料。
> 部分电子书如图所示。

![概念学习](https://thepatterraining.github.io/images/bottom1.png)

![概念学习](https://thepatterraining.github.io/images/bottom2.png)

![概念学习](https://thepatterraining.github.io/images/bottom3.png)

![概念学习](https://thepatterraining.github.io/images/bottom4.png)


