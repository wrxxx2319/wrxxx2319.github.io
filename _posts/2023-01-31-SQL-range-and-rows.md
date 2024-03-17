---
title: 开窗函数的滑动窗口语法
date: 2023-01-31 08:11:01 +0800
categories: [数据分析,SQL学习]
tags: [SQL,开窗函数,滑动窗口]
math: true
mermaid: true
---

此前一直对SQL语言的某些功能“缺陷”十分不满意，特别是在一张表的前几行或着后几行进行聚合运算的时候，会非常麻烦，lead/lag语句也不能满足所有的操作，只能费劲的使用自交或子查询。顿感麻烦，甚至不如Excel表格的功能强大。

今天在做Leetcode的[**1321.参观营业额变化增长**](https://leetcode.cn/problems/restaurant-growth/description/)题目时更是感觉非常麻烦，感觉SQL语言不可能不提供类似滑动窗口的机制来解决这种窗口均值的问题，且这种问题在数据分析的场景下十分普遍。所以我就自己查了一下，发现是自己孤陋寡闻了，SQL语言在8.0版本后支持了滑动窗口子句，功能十分强大，特此记录。

## Range/Rows基本介绍
> 开窗函数只适用于SQL8.0及以上.
{: .prompt-warning }

Range 和 Rows滑动窗口字句主要用于定义当前的聚合函数的作用范围，Range和Rows的作用依据不同，具体如下

- Range
  + Range表示滑动窗口将按照<span style="color:red;">值</span>的范围进行定义，只跟值相关，
  <span style="color:red;">行数不固定</span>，
  只要数值在范围内就会包含在窗口中(<span style="color:red;">与当前行的行号无关，只与当前行的值有关</span>)。
- Rows
  + Rows表示滑动窗口将按照<span style="color:red;">行</span>的范围进行定义，只跟行号相关，若排序(order by)后，则会按照排序的前后行顺序进行计算(<span style="color:red;">与当前行的值无关，只与排序后的行号相关</span>)。


rows/range子句的使用方法如下:

```
  <窗口函数> over ( partition by <用于分组的列名> order by <用于排序的列名> 
                      rows/range子句<用于定义窗口大小> )
```

## Range/Rows子句的写法

rows/range子句的具体书写规范如下

```
  <rows or range> between <Start expr> and <End expr>
```

Start expr 和 End expr 的可取值为
- rurrent row
  + 当前行
- N preceding
  + 前N行(rows)，前N个数值(range)
- unbounded preceding
  + 不限制前向行数，即从开头开始
- N following
  + 后N行(rows)，后N个数值(range)
- unbounded following
  + 不限制后向行数，即从结尾开始

注意：其中，and语句可以省略，且N的数值可以是日期，例如：
```
  range 6 preceding	
  -- 当前数值至(数值-6)

  range interval 6 day preceding	
  -- 最近7天的值
```
## Range/Rows子句的使用例子

例如名为Customer的表

(customer_id, visited_on) 是该表的主键。
该表包含一家餐馆的顾客交易数据。
visited_on 表示 (customer_id) 的顾客在 visited_on 那天访问了餐馆。
amount 是一个顾客某一天的消费总额。

| customer_id | name         | visited_on   | amount      |
|:-----------:|:------------:|:------------:|:-----------:|
| 1           | Jhon         | 2019-01-01   | 100         |
| 2           | Daniel       | 2019-01-02   | 110         |
| 3           | Jade         | 2019-01-03   | 120         |
| 4           | Khaled       | 2019-01-04   | 130         |
| 5           | Winston      | 2019-01-05   | 110         | 
| 6           | Elvis        | 2019-01-06   | 140         | 
| 7           | Anna         | 2019-01-07   | 150         |
| 8           | Maria        | 2019-01-08   | 80          |
| 9           | Jaze         | 2019-01-09   | 110         | 

运行语句

```sql
select
  *,
  -- 计算最近七天(包含当前天)的营业额总和
  sum(amount) over ( order by visited_on range interval 6 day preceding) as '7days_sum',
  -- 计算前后三天(包含当前天)的营业额总和
  sum(amount) over ( order by visited_on range between interval 3 day preceding 
                        and interval 3 day following ) as '+-3days_sum'
from 
  Customer
```

得到的结果为：

| customer_id | name    | visited_on | amount | 7days_sum | +-3days_sum |
|:-----------:|:-------:|:----------:|:------:|:---------:|:-----------:|
| 1           | Jhon    | 2019-01-01 | 100    | 100       | 460         |
| 2           | Daniel  | 2019-01-02 | 110    | 210       | 570         |
| 3           | Jade    | 2019-01-03 | 120    | 330       | 710         |
| 4           | Khaled  | 2019-01-04 | 130    | 460       | 860         |
| 5           | Winston | 2019-01-05 | 110    | 570       | 840         |
| 6           | Elvis   | 2019-01-06 | 140    | 710       | 840         |
| 7           | Anna    | 2019-01-07 | 150    | 860       | 1000        |
| 8           | Maria   | 2019-01-08 | 80     | 840       | 870         |
| 9           | Jaze    | 2019-01-09 | 110    | 840       | 760         |

注意：这里的滑动窗口是全闭区间，都会包含边缘的行

## 个人感悟

窗口子语句功能十分强大，可以满足多种操作。感觉自己之前还是在SQL语句方面关注的太少，应该进一步加强SQL语句的学习和新语言特性的关注
