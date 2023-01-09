---
title: "记一次在Oracle中关联其他表更新数据的操作"
date: 2023-01-09T14:15:37+08:00
draft: true
tags: [DB,Oracle]
categories: [DB]
---

## 一、场景

> 在Oracle中从关联表根据关联条件取满足条件的其中一条更新本表的部分字段

### 场景解读

表1：学生Student表

|id|name|topSubject|topRank|
|:---:|:---:|:---:|:---:|
|学号|学生姓名|最高排名科目|最高排名|
|1|张三|-|-|
|2|李四|-|-|
|3|王五|-|-|

表2：成绩Score表

> 记录每个学生在每科的排名

|id|sid|subject|rank|score|
|:---:|:---:|:---:|:---:|:---:|
|记录id|学号|科目|排名|成绩|
|1|1|语文|32|52|
|2|1|数学|25|97|
|4|2|语文|2|62|
|5|2|数学|65|72|
|7|3|语文|6|83|
|8|3|数学|5|95|

目标操作：从成绩表中找到每个人排名最高的科目的记录，并更新学生表中对应的topSubject字段和topRank字段

## 二、基本方式

逐行更新，从成绩表中找到对应学生成绩记录中，排名最高的那条记录，并更新当前记录的topSubject字段和topRank字段

```sql
update Student t
set (t.topSubject,t.topRank)=(
    select s.subject,rank 
    from Score s 
    where t.id=s.sid 
        and s.rank=(select min(s.rank) from Score where sid=s.sid));
```

### 基本方式解读解读

如果未对任何字段建立索引的情况下，该语句的执行会进行三次整表扫描，

分别为`update Student`、子查询`select s.subject,rank`、子查询`select min(rank)`。

> 针对该方式的优化空间为：针对成绩表Score的`sid`和`rank`字段建立索引`sid, rank desc`
> 从而降低`select s.subject,rank`和`select min(rank)`两个查询的查询复杂度;

## 三、update-with方式

建立中间表temp，中间表记录每个学生的高排名和对应的科目，执行update操作的时候从中间表更新

```sql
update Student t
set (t.topSubject,t.topRank)=(
    with temp as (
        select s.sid,s.subject,rank,ROW_NUMBER() OVER(PARTITION BY sid ORDER BY rank ASC) rn
        from Score s
        where rn=1)
    select temp.subject,temp.rank 
    from temp 
    where temp.sid=t.id);
```

### update-with方式解读

该语句的执行会进行两次整表扫描

分别为`update Student`、with子句`with temp as(select s.sid,....)`

## 四、merge-into方式

Oracle在9i中引入了`merge`命令，能够在一个SQL语句中同时执行inserts和updates操作。

```sql
MERGE INTO [target-table] A 
USING [source-table sql] B 
ON([conditional expression] and [...]...)
WHEN MATCHED THEN
    [UPDATE sql]
WHEN NOT MATCHED THEN
    [INSERT sql]
```

通过merge操作对匹配到的记录进行更新操作

```sql
merge into Student t
USING (
    select * from (
        select ROW_NUMBER() OVER(PARTITION BY sid ORDER BY rank ASC) rn, s.* 
        from Score 
        where rn=1)) temp
ON temp.sid = t.id
WHEN MATCHED THEN
    update set t.topSubject=temp.subject, t.topRank=temp.rank;
```

### merge-into方式解读

该语句执会执行两次整表扫描，分别为`merge into Student`、临时表建立`USING(select * ...)temp`，

> - 相比update-with方式，merge-into提供了其他的条件下的更多操作，
> - 但merge-into在其他数据库(MySQL、PostgreSQL等)中不被支持。
