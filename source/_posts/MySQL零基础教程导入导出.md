---
title: MySQL零基础教程高阶应用实战
date: 2025-07-26 10:12:47
tags: ['数据库','数据库原理','sql','mysql']
category: mysql
article: MySQL零基础教程高阶应用实战
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
 
## 高阶应用实战

除了常用的增删改查以外，MySQL还有很多的功能。比如`CTE`、`窗口函数`这种。再比如一些特性比如`视图`，`触发器`等。

这次我们就来看一下对于这些如何使用。

有些功能用好了可以完成很多实际业务需求的。


### CTE(common table expressions)

在MySQL中，**CTE（Common Table Expressions，公共表表达式）**是一种临时的结果集，可以在查询中被引用。CTE通常用于简化复杂的查询，使查询更易于理解和维护。CTE在MySQL 8.0及更高版本中得到了支持。

#### CTE的主要特点
临时结果集：
- CTE是一个临时的结果集，可以在查询中被引用。它类似于子查询，但更易于阅读和维护。
简化复杂查询：
- CTE可以将复杂的查询分解为多个简单的部分，使查询更易于理解和维护。
可重用性：
- CTE可以被多次引用，减少了代码重复，提高了开发效率。
递归查询：
- CTE支持递归查询，可以用于处理层次结构或递归数据。

使用`CTE`实现获取每个课程中分数最高的学生信息。

通过`WITH`语句来声明一个临时表。表名`cteSource`，表的内容就是最的sid，通过`SELECT MAX(sid) FROM enrolled`查询出来的结果。字段名叫`maxId`。

然后在查询语句里面就可以连接`cteSource`表，然后通过sid = cteSource.maxId 来获取到sid最大的用户信息。

```SQL
WITH cteSource (maxId) AS (
    SELECT MAX(sid) FROM enrolled
)
SELECT name FROM student, cteSource
WHERE student.sid = cteSource.maxId
```

还有一些其他的用法，比如:

```SQL
WITH cte1 (col1) AS (
SELECT 1
),
cte2 (col2) AS (
SELECT 2
)
SELECT * FROM cte1, cte2;
```

#### CTE（公共表表达式）使用场景

1. 递归查询（如树形结构）

CTE最常见的场景之一是递归查询，比如组织架构、分类目录、菜单结构等树状数据。例如，查询某个员工的所有下属员工。

```sql
WITH RECURSIVE subordinates AS (
  SELECT id, name, manager_id
  FROM employees
  WHERE manager_id IS NULL -- 顶层经理

  UNION ALL

  SELECT e.id, e.name, e.manager_id
  FROM employees e
  INNER JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates;
```

2. 简化复杂查询逻辑，提高可读性

当查询逻辑复杂时，可以将中间结果用CTE分段处理，让SQL更易读。例如，先筛选出某类用户，再基于这些用户做进一步统计。

```sql
WITH active_users AS (
  SELECT user_id
  FROM user_logins
  WHERE login_date > CURRENT_DATE - INTERVAL '30 days'
)
SELECT COUNT(*) FROM orders WHERE user_id IN (SELECT user_id FROM active_users);
```

3. 多次引用中间结果，避免重复计算

如果某个子查询需要被多次引用，CTE可以定义一次，后续多次调用，避免重复书写和计算，提高效率。

```sql
WITH sales_summary AS (
  SELECT product_id, SUM(amount) AS total_sales
  FROM sales
  GROUP BY product_id
)
SELECT * FROM sales_summary WHERE total_sales > 1000;
-- 也可以在后续多个查询中引用 sales_summary
```

4. 分步处理数据，逐步细化结果

例如，先计算每个部门的总销售额，再筛选出销售额最高的部门。

```sql
WITH dept_sales AS (
  SELECT department_id, SUM(sales) AS total_sales
  FROM employees
  GROUP BY department_id
),
top_dept AS (
  SELECT department_id
  FROM dept_sales
  ORDER BY total_sales DESC
  LIMIT 1
)
SELECT * FROM employees WHERE department_id IN (SELECT department_id FROM top_dept);
```

5. 数据去重和排名

CTE可以配合窗口函数实现数据去重、分组排名等需求。

```sql
WITH ranked_orders AS (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
  FROM orders
)
SELECT * FROM ranked_orders WHERE rn = 1;
```

#### 实战案例

比如我之前的业务。有一个公司表，一个公司合同表，一个公司合同申请表。在后台的公司合同列表里面，要展示最新的申请记录和申请状态。

如果没有CTE的话，我们就需要使用`子查询`来实现。这对于一个列表来说，性能是很低的。

表结构假设:
- company：公司表，字段如 company_id, company_name 等。
- company_contract：公司合同表，字段如 contract_id, company_id, contract_no 等。
- company_contract_application：公司合同申请表，字段如 application_id, contract_id, apply_time, status 等。

```sql
SELECT
    c.company_id,
    c.company_name,
    cc.contract_id,
    cc.contract_no,
    cca.application_id,
    cca.apply_time,
    cca.status
FROM company c
JOIN company_contract cc ON c.company_id = cc.company_id
LEFT JOIN company_contract_application cca
    ON cc.contract_id = cca.contract_id
    AND cca.apply_time = (
        SELECT MAX(apply_time)
        FROM company_contract_application
        WHERE contract_id = cc.contract_id
    );
```

而有了CTE以后，我们可以通过CTE来实现，这样可以提升性能。

```sql
WITH latest_applications AS (
    SELECT
        application_id,
        contract_id,
        apply_time,
        status,
        ROW_NUMBER() OVER (PARTITION BY contract_id ORDER BY apply_time DESC) AS rn
    FROM company_contract_application
)
SELECT
    c.company_id,
    c.company_name,
    cc.contract_id,
    cc.contract_no,
    la.application_id,
    la.apply_time,
    la.status
FROM company c
JOIN company_contract cc ON c.company_id = cc.company_id
LEFT JOIN latest_applications la ON cc.contract_id = la.contract_id AND la.rn = 1;
```

### 窗口函数

窗口函数（Window Function）是SQL标准中的一种功能强大的工具，它允许在查询中对一组行进行计算，而不会像聚合函数那样消除行的个数。窗口函数在MySQL 8.0及更高版本中得到了支持，它们可以用于计算移动平均值、累积和、排名等复杂的分析任务。

#### 窗口函数的主要特点
行级计算：
- 窗口函数在每一行上执行计算，同时可以访问同一组中的其他行。
- 这与聚合函数不同，聚合函数会将多行数据合并为一行。
分区和排序：
- 窗口函数可以使用PARTITION BY子句将数据分成多个分区，每个分区独立计算。
- 可以使用ORDER BY子句在每个分区内对数据进行排序。
灵活的范围定义：
- 窗口函数可以定义计算的范围，如当前行的前几行或后几行。
- 使用ROWS或RANGE子句可以指定计算的范围。
多种功能：
- 窗口函数提供了多种功能，如ROW_NUMBER()、RANK()、DENSE_RANK()、NTILE()、SUM()、AVG()、LEAD()、LAG()等。

`ROW_NUMBER`和`RANK`都需要和`OVER`一起使用。

- ROW_NUMBER(): 显示当前行号
- RANK() : 显示排序后的排名，如果没有排序，都是1
- OVER()
    - PARTITION BY 进行分组
    - GROUP BY 进行分组
    - ORDER BY 排序

![001](https://thepatterraining.github.io/images/15445001.png)

![002](https://thepatterraining.github.io/images/15445002.png)

![003](https://thepatterraining.github.io/images/15445003.png)

#### 实战：获取每个课程中分数最高的学生信息

下面的SQL，在postgresql中执行成功，mysql8执行报错。

首先查询所有课程信息，并按照课程分组，按照分数排序。

```SQL
SELECT *,
RANK() OVER (PARTITION BY cid ORDER BY grade ASC)
AS rank FROM enrolled
```

![004](https://thepatterraining.github.io/images/15445004.png)

接着搜索上表中分数为1，也就是分数最高的学生。也就是每个课分数最高的学生信息。

```SQL
SELECT * FROM (
    SELECT *,
    RANK() OVER (PARTITION BY cid
    ORDER BY grade ASC)
    AS rank FROM enrolled) AS ranking
WHERE ranking.rank = 1
```

![005](https://thepatterraining.github.io/images/15445005.png)

### 视图

在MySQL中，**视图（View）**是一种虚拟表，其内容由SQL查询定义。视图并不存储实际的数据，而是根据定义的查询动态生成数据。视图可以简化复杂的SQL操作，提供数据的逻辑抽象，并且可以限制对某些数据的访问，从而增强数据的安全性。

MySQL中的视图是`虚拟视图`，说白了就是一条SQL语句，当查询视图的时候执行SQL语句而已。

除此之外，还有一个东西叫做`物化视图`，MySQL并没有实现这个东西，物化视图就是一张真的表，而不是一个SQL语句，因此查询效率更好。

#### 视图的主要特点

虚拟表：
- 视图是一个虚拟表，其内容由SQL查询定义。视图本身并不存储数据，而是根据定义的查询动态生成数据。

简化复杂查询：
- 视图可以简化复杂的SQL操作，将复杂的查询逻辑封装起来，使用户可以像查询普通表一样查询视图。

数据抽象：
- 视图提供数据的逻辑抽象，隐藏了底层表的复杂性，使用户可以更直观地访问数据。

安全性：
- 视图可以限制对某些数据的访问，增强数据的安全性。通过视图，用户只能访问视图定义的特定数据，而不能访问底层表的全部数据。

更新限制：
- 视图可以是可更新的，也可以是不可更新的。可更新视图允许用户通过视图插入、更新或删除数据，但需要满足一定的条件。不可更新视图则不允许用户通过视图修改数据。

创建一个视图。下面的语句，创建一个视图，视图名称是`sales_employees`，内容就是后面的Select语句的结果。当原始表`employees`变化以后，视图的内容也会跟着变化。
```sql
CREATE VIEW sales_employees AS
SELECT name, salary
FROM employees
WHERE department = 'Sales';
```

#### 实战：交易聚合

我之前有一个业务，存在两个表，一个是买单表，代表买方要买一个产品出的价格。一个是卖单表，代表卖方要卖一个产品出的价格。

现在有一个需求是：在C端的一个页面上，要求混合展示买单信息和卖单信息，按照价格从低到高排序

数据库假设：
- buy: 买单表
- sell: 卖单表
- order: 交易视图

```sql
CREATE VIEW order AS
SELECT *
FROM buy, sell
order by price asc;
```

### 触发器

在MySQL中，**触发器（Trigger）**是一种特殊的存储过程，它在特定的数据库操作（如INSERT、UPDATE、DELETE）发生时自动执行。触发器可以用于实现复杂的业务逻辑，确保数据的完整性和一致性，以及自动维护数据的同步。

#### 触发器的主要特点

自动执行：
- 触发器在特定的数据库操作发生时自动执行，无需显式调用。这使得触发器可以用于实现自动化的数据处理和维护。

数据完整性：
- 触发器可以用于确保数据的完整性和一致性。例如，可以在插入或更新数据时自动检查数据的有效性，或者在删除数据时自动清理相关数据。

业务逻辑：
- 触发器可以用于实现复杂的业务逻辑。例如，可以在插入或更新数据时自动计算某些字段的值，或者在删除数据时自动更新相关表的数据。

数据同步：
- 触发器可以用于自动维护数据的同步。例如，可以在插入或更新数据时自动更新相关表的数据，或者在删除数据时自动清理相关表的数据。

#### 触发器的类型

- BEFORE INSERT：
  - 在插入数据之前执行触发器逻辑。
- AFTER INSERT：
  - 在插入数据之后执行触发器逻辑。
- BEFORE UPDATE：
  - 在更新数据之前执行触发器逻辑。
- AFTER UPDATE：
  - 在更新数据之后执行触发器逻辑。
- BEFORE DELETE：
  - 在删除数据之前执行触发器逻辑。
- AFTER DELETE：
  - 在删除数据之后执行触发器逻辑。

#### 触发器的限制

性能影响：
- 触发器的执行会增加数据库操作的开销，可能会影响性能。因此，应谨慎使用触发器，避免在高频操作的表上定义过多的触发器。
复杂性：
- 触发器的逻辑可以非常复杂，但过多的复杂逻辑可能导致触发器难以维护和调试。因此，应尽量保持触发器的逻辑简单明了。
调试困难：
- 触发器的调试相对困难，因为它们在特定的操作发生时自动执行，难以直接观察和调试。因此，建议在开发和测试阶段充分测试触发器的逻辑。

因此，实际开发中基本不使用触发器。

### 存储过程

在MySQL中，**存储过程（Stored Procedure）**是一种预编译的SQL语句集合，它存储在数据库中，可以通过调用其名称并传递参数来执行。存储过程可以包含复杂的逻辑和多个SQL语句，用于完成特定的任务。它们类似于其他编程语言中的函数或方法。

可以把存储过程想成一个函数。只不过是在MySQL中的函数，这个函数可以实现各种功能。可以实现一些复杂的SQL处理，这样可以简化调用。

#### 存储过程的主要特点

预编译：
- 存储过程在创建时被预编译并存储在数据库中，这使得它们的执行速度比单独的SQL语句更快。
代码重用：
- 存储过程可以被多次调用，减少了代码重复，提高了开发效率。
减少网络流量：
- 存储过程在服务器端执行，减少了客户端和服务器之间的网络流量，因为只需要发送存储过程的名称和参数，而不是大量的SQL语句。
安全性：
- 存储过程可以限制用户对底层数据的直接访问，只允许通过存储过程进行数据操作，从而增强数据的安全性。
事务管理：
- 存储过程可以包含事务控制语句，如COMMIT和ROLLBACK，确保数据操作的完整性和一致性。

#### 存储过程的限制

性能影响：
- 存储过程的执行会增加数据库操作的开销，可能会影响性能。因此，应谨慎使用存储过程，避免在高频操作的表上定义过多的存储过程。
复杂性：
- 存储过程的逻辑可以非常复杂，但过多的复杂逻辑可能导致存储过程难以维护和调试。因此，应尽量保持存储过程的逻辑简单明了。
调试困难：
-   存储过程的调试相对困难，因为它们在服务器端执行，难以直接观察和调试。因此，建议在开发和测试阶段充分测试存储过程的逻辑。

#### 实战案例

存储过程的实际应用场景有限。

我们之前有一个场景，是需要批量处理数据，有几十行SQL语句。因此将这些SQL封装到了一个存储过程中，这样只需要执行这个存储过程就可以了。

这里给出一个简单的实战案例：

创建一个存储过程get_employee_details，用于根据员工ID获取员工的详细信息：

DELIMITER用来设置结束符，比如正常的句子结束符是句号。代码结束符是分号；

IN代表输入参数，也就是这个函数有一个输入参数emp_id，是int类型。
还有out代表输出参数，用于返回结果。
INOUT代表既可以输入参数也可以是输出参数。

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

上面这个存储过程的函数体，就是一段select查询语句。

### 存储函数

在MySQL中，**存储函数（Stored Function）**是一种预编译的SQL语句集合，类似于存储过程，但它返回一个值。存储函数可以被SQL语句直接调用，就像调用普通的函数一样。存储函数通常用于封装复杂的逻辑，并在查询中重用这些逻辑。

存储函数同样是一个函数，和上面的存储过程差不多。

#### 存储过程和存储函数的区别

| 区别类别   | 存储过程     | 存储函数     |
|------------|------------|------------|
| 返回值     | 1. 可以返回多个值，通过OUT或INOUT参数返回。<br>2. 可执行多个SQL语句，但不直接返回单一值。| 1. 必须返回一个单一的值。<br>2. 可以被SQL语句直接调用，就像普通函数一样。 |
| 调用方式   | 1. 通过CALL语句调用。<br>2. 可执行复杂逻辑，包括多个SQL语句和事务控制。 | 1. 可直接在SQL语句中调用，如普通函数。<br>2. 通常用于封装复杂逻辑，并在查询中重用。       |

#### 存储函数的主要特点

| 主要特点     | 说明                                                                                                   |
|--------------|--------------------------------------------------------------------------------------------------------|
| 返回值       | 存储函数必须返回一个值，这使得它们可以被SQL语句直接调用。                                               |
| 代码重用     | 存储函数可以被多次调用，减少了代码重复，提高了开发效率。                                                 |
| 减少网络流量 | 存储函数在服务器端执行，减少了客户端和服务器之间的网络流量，因为只需要发送函数的名称和参数，而不是大量的SQL语句。 |
| 安全性       | 存储函数可以限制用户对底层数据的直接访问，只允许通过函数进行数据操作，从而增强数据的安全性。             |
| 事务管理     | 存储函数可以包含事务控制语句，如COMMIT和ROLLBACK，确保数据操作的完整性和一致性。                        |

#### 实战案例

创建一个存储函数get_employee_salary，用于根据员工ID获取员工的薪资：

可以看到和上面存储过程的区别，声明了一个返回值，类型是DECIMAL,最后通过return返回了，并且声明了一个变量。参数也没有IN、OUT这种了。
```sql
DELIMITER //

CREATE FUNCTION get_employee_salary(emp_id INT)
RETURNS DECIMAL(10, 2)
BEGIN
    DECLARE emp_salary DECIMAL(10, 2);
    SELECT salary INTO emp_salary
    FROM employees
    WHERE id = emp_id;
    RETURN emp_salary;
END //

DELIMITER ;
```

## 总结

本次讲解了MySQL的高级功能和实战应用的一些场景。

只有知道了场景才能活学活用。这里面最常用的其实还是CTE和窗口函数。

可以多多练习，希望遇到这些场景的时候，大家能想起来使用这些功能。

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


