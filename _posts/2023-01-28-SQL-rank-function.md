---
title: 关于SQL语法中的排名相关开窗函数总结
date: 2023-01-28 12:23:11 +0800
categories: [数据分析,SQL学习]
tags: [SQL,开窗函数]
math: true
mermaid: true
---


今天在做Leetcode的SQL题目[**185.部门工资前三高的员工**](https://leetcode.cn/problems/department-top-three-salaries/solutions/)时，发现自己在排名（rank）相关的开窗函数上掌握的不是很好，特别是记不住每个函数的区别。

因此特此查询了相关的官方SQL文档和CSDN博客，共总结了四种和排名相关的开窗函数。

## 开窗函数基本介绍和用法
> 开窗函数只适用于SQL8.0及以上.
{: .prompt-warning }
开窗函数允许在查询中进行对分组数据进行计算、 同时保留原始行的详细信息，并且把分组查询的结果直接在每一行列出。相对于传统的分组函数+聚合的方式，开窗函数的聚合效果更好，简化了查询的难度。

具体的使用方法如下:

```
  开窗函数(计算字段名) OVER (PARTITION BY 分组字段名 ORDER BY 排序字段名)
```
接下来介绍四大排名函数

## row_number()

此函数是针对序号进行计数，序号连续不重复

例如有名为test的表

| id | number |
|:---:|:----:|
| 1  | 4 |
| 2  | 3 |
| 3  | 3 |
| 4  | 2 |
| 5  | 5 |
| 6  | 4 |

运行语句

```sql
select 
  *
  ,row_number() over (order by number) as row_number
from
  test
```

得到的结果为：

| id | number | row_number |
|:---:|:----:|:--:|
| 5  | 5 | 1 |
| 1  | 4 | 2 |
| 6  | 4 | 3 |
| 2  | 3 | 4 |
| 3  | 3 | 5 |
| 4  | 2 | 6 |

注意：若row_number()中over子句中的order by和SQL排序记录中的 order by不一致，会以SQL中的排序呈现，但是字句的需要是正确的

## rank()

相同的值认为是一个序号，但是下一个序号不连续。

运行语句

```sql
select 
  *
  ,rank() over (order by number) as rank
from
  test
```

得到的结果为：

| id | number | rank |
|:---:|:----:|:--:|
| 5  | 5 | 1 |
| 1  | 4 | 2 |
| 6  | 4 | 2 |
| 2  | 3 | 4 |
| 3  | 3 | 4 |
| 4  | 2 | 6 |

## dense_rank()

排序的序号是连续的，且相同的值会归为同一个序号，个人觉得这个函数最好用。

运行语句

```sql
select 
  *
  ,dense_rank() over (order by number) as dense_rank
from
  test
```

得到的结果为：

| id | number | dense_rank |
|:---:|:----:|:--:|
| 5  | 5 | 1 |
| 1  | 4 | 2 |
| 6  | 4 | 2 |
| 2  | 3 | 3 |
| 3  | 3 | 3 |
| 4  | 2 | 4 |

## ntile()

ntile这个函数与其他的三个函数不同，他主要用于分组，参数为ntile(group_num)，可以将所有记录分成group——num个组，每组序号一样。

运行语句

```sql
select 
  *
  ,ntile(2) over (order by number) as 'ntile(2)'
from
  test
```

得到的结果为：

| id | number | ntile(2) |
|:---:|:----:|:--:|
| 5  | 5 | 1 |
| 1  | 4 | 1 |
| 6  | 4 | 1 |
| 2  | 3 | 2 |
| 3  | 3 | 2 |
| 4  | 2 | 2 |

注意；如果行数不能被group_num整除，则最后一个组的行数是总体行数 % group_num

## 个人感悟

从数据分析和刷题的经验来说，我个人认为使用的频繁程度为

dense_rank > row_number > ntile = rank

dense_rank在查询中最为好用，且非常符合逻辑。
