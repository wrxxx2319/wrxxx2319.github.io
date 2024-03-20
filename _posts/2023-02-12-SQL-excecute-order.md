---
title: SQL查询语句书写与执行顺序
date: 2023-02-12 09:12:32 +0800
categories: [数据分析,SQL学习]
tags: [SQL,书写顺序,执行顺序]
math: true
mermaid: true
---

其他的编程语言基本都是按照书写顺序执行的，但SQL语言具有一定的特殊性，其书写顺序和执行顺序并不相同。若按照执行顺序进行书写会报错。因此我特此整理出查询语句的书写顺序和执行顺序，在需要时可以查询。

## SQL查询语句的书写顺序

SQL共存在十种语句，分别为
select, from, join, on, where, group by, having, union, order by, limit。
其中，select必须存在

其书写顺序为：
```
SELECT <字段名>
FROM <表名>
JOIN <表名> 
ON <连接条件>
WHERE <筛选条件>
GROUP BY <字段名>
HAVING <筛选条件>
UNION
ORDER BY <字段名>
LIMIT <限制行数>
```

## SQL查询语句的执行顺序

在真正执行时，SQL语句执行引擎会在每一个语句执行后生成一个虚拟表(virtual_table)，并把这个虚拟表当作下一个语句的输入。使用者并不能操作这些虚拟表，语句结束后才能被使用者使用。

具体的顺序如下

1. FORM：选择FROM后跟的表，产生虚拟表1，<span style="color:red;">若存在两个表，则会直接取笛卡尔积</span>。

2. ON：ON是JOIN的连接条件，符合连接条件的行会被记录在虚拟表2中。

3. JOIN：如果指定了LEFT JOIN，那么保留表中未匹配的行就会作为外部行添加到虚拟表2中，产生虚拟表3。如果有多个JOIN链接，会重复执行步骤1~3，直到处理完所有表。

4. WHERE：对虚拟表3进行WHERE条件过滤，符合条件的记录会被插入到虚拟表4中。

5. GROUP BY：根据GROUP BY子句中的列，对虚拟表2中的记录进行分组操作，产生虚拟表5。

6. HAVING：对虚拟表5进行HAVING过滤，符合条件的记录会被插入到虚拟表6中。

7. SELECT：SELECT到一步才执行，选择指定的列，插入到虚拟表7中。

8. UNION：UNION连接的两个SELECT查询语句，会重复执行步骤1~7，产生两个虚拟表7，UNION会将这些记录合并到虚拟表8中。

9. ORDER BY: 将虚拟表8中的记录进行排序，虚拟表9。

10. LIMIT：取出指定行的记录，返回结果集。

## 使用注意事项

1. SELECT语句总是写在最前面，但在大部分语句之后才执行。所以在SQL语句中，我们不能在WHERE、GROUP BY、 HAVING语句中使用在 SELECT 中设定的别名。

2. MySQL数据库可以在GROUP BY、 HAVING语句中使用在 SELECT 中设定的别名。<span style="color:red;">不是因为MYSQL中会提前执行SELECT，而是因为在GROUP BY这一步返回了游标，导致可以使用，但是SELECT语句执行顺序还是排在后面的。</span>这是MySQL数据库的特性，其他数据库引擎不一定支持这样。

3. 无论是书写顺序，还是执行顺序，UNION 都是排在 ORDER BY 前面的。SQL语句会将所有UNION 段合并后，再进行排序。