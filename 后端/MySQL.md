### 数据库引擎

```sql
--关于数据库引擎
/*
INNODB	默认使用
MYISAM	早些年使用的
*/
```

|            | MYISAM | INNODB        |
| ---------- | ------ | ------------- |
| 事务支持   | 不支持 | 支持          |
| 数据行锁定 | 不支持 | 支持          |
| 外键约束   | 不支持 | 支持          |
| 全文索引   | 支持   | 不支持        |
| 表空间大小 | 较小   | 较大，约为2倍 |

常规使用操作：

- MYISAM	节约空间，速度较快
- INNODB    安全性高，事务的处理，多表多用户操作

所有数据库文件都存在data目录下

本质还是文件的存储

### sql语法

![image-20210305131434602](C:\Users\mujou\AppData\Roaming\Typora\typora-user-images\image-20210305131434602.png)

### delete和truncate

- 相同点：都能删除数据，都不会删除表结构

- 不同点

  - truncate：重新设置自增列，计数器会归零
  - truncate：不会影响事务
  - delete：计数器不归零

  了解即可：`deltete删除的问题`，重启数据库，现象

  - InnoDB	自增列会重新从1开始（存在内存中，断电即失）
  - MyISAM   继续从上一个自增量开始（存在文件中，不会丢失）

### distinct：发现重复数据，去重

| 运算符      | 语法              | 描述                                       |
| ----------- | ----------------- | ------------------------------------------ |
| IS NULL     | a is null         | 如果操作符为NULL，结果为真                 |
| IS NOT NULL | a is not null     | 如果操作符不为NULL，结果为真               |
| BETWEEN     | a between b and c | 若a在b和c之间，结果为真                    |
| **Like**    | a like b          | SQL匹配，如果a匹配b，结果为真              |
| **In**      | a in(a1,a2,a3...) | 假设a在a1，a2...其中的某一个值中，结果为真 |

```sql
--模糊查询
--查询刘性同学
--like结合	%（代表0到任意个字符）	_（一个字符）
select `studentno`,`studentname` from `student` where studentname like '刘%'

--查询刘性同学，名字后面只有一个字
select `studentno`,`studentname` from `student` where studentname like '刘_'

--查询刘性同学，名字后面有两个字
select `studentno`,`studentname` from `student` where studentname like '刘__'

--查询名字有嘉字的同学
select `studentno`,`studentname` from `student` where studentname like '%嘉%'

--in（具体的一个或者多个值）
--查询1001、1002、1003号学员
select `studentno`,`studentname` from `student` where studentno in (1001,1002,1003)
```

### 联表查询

on和where的区别：https://blog.csdn.net/xc008/article/details/2872310?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=1328602.10553.16149193092276583&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019061718553496.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Fzc2Fzc2luaGFuYw==,size_16,color_FFFFFF,t_70)

```sql
--联表查询
--查询参加考试的同学（学号，姓名，科目编号，分数）
/*思路
1、分析需求，查询的字段来自哪些表（连接查询）
2、确定使用哪种连接查询
确定交叉点（这两个表中哪两个数据是相同的）
判断的条件
*/
select s.studentno,studentname,subjectno,studentresult from 
`student` as s
inner join result as r
on s.studentno=r.studentno

--right join
select s.studentno,studentname,subjectno,studentresult from 
`student` as s
right join result as r
on s.studentno=r.studentno

--left join
select s.studentno,studentname,subjectno,studentresult from 
`student` as s
left join result as r
on s.studentno=r.studentno
```

| 操作       | 描述                                         |
| ---------- | -------------------------------------------- |
| inner join | 如果表中至少有一个匹配，就返回行             |
| left join  | 即使右表中没有匹配，也会从左表中返回所有的值 |
| right join | 即使左表中没有匹配，也会从右表中返回所有的值 |

### 自连接

自己的表和自己的表连接，核心：**一张表拆为两张一样的表即可**

category:

![image-20210305130532463](C:\Users\mujou\AppData\Roaming\Typora\typora-user-images\image-20210305130532463.png)

父类

| categoryid | categoryName |
| ---------- | ------------ |
| 2          | 信息技术     |
| 3          | 软件开发     |
| 5          | 美术设计     |

子类

| pid  | categoryid | categoryName |
| ---- | ---------- | ------------ |
| 3    | 4          | 数据库       |
| 2    | 8          | 办公信息     |
| 3    | 6          | web开发      |
| 5    | 7          | 美术设计     |

操作：查询父类对应的子类关系

| 父类     | 子类     |
| -------- | -------- |
| 信息技术 | 办公信息 |
| 软件开发 | 数据库   |
| 软件开发 | web开发  |
| 美术设计 | ps技术   |

```sql
--查询父子信息	把一张表看成两张一样的表
select a.categoryName as '父栏目',b.categoryName as '子栏目' from category as a,category as b
where a.categoryid=b.pid
```

### 分页和排序

![image-20210305131652140](C:\Users\mujou\AppData\Roaming\Typora\typora-user-images\image-20210305131652140.png)

```sql
--100w条数据
/*
为什么分页？
1、缓解数据库压力 
2、给人的体验更好
*/
--瀑布流
--分页，每个页面只显示五条数据
--语法：limit 起始值，页面大小
--limit 0，5 1~5
--limit 1，5 2~6
```

### 子查询

在where语句中嵌套一个子查询语句

where（select * from）

相当于一个in子句，不过这个范围集要自己select出来

```sql
select `studentno`,`subjectno`,`studentresult` from `result` where subjectno = (select subjectno from subject where subjectname='数据库结构-1')
```

### 分组

聚集函数：AVG()、MAX()、MIN()

```sql
select subjectname,AVG(studentresult),MAX(studentresult),MIN(studentresult) from result r inner join subject on r.subjectno=sub.subjectno
group by r.subjectno --通过什么字段来分组
```

 where不能使用聚集函数，要在having中使用

```sql
--error
select subjectname,AVG(studentresult),MAX(studentresult),MIN(studentresult) from result r inner join subject on r.subjectno=sub.subjectno
group by r.subjectno --通过什么字段来分组
where AVG(studentresult)>=80
-- success
select subjectname,AVG(studentresult),MAX(studentresult),MIN(studentresult) from result r inner join subject on r.subjectno=sub.subjectno
group by r.subjectno --通过什么字段来分组
having AVG(studentresult)>=80 --过滤分组记录需要满足的次要条件
```

### 数据库级别的MD5加密

什么是MD5？

主要增强算法复杂度和**不可逆**性

MD5破解网站的原理，背后有一个字典，MD5加密后的值  加密前的值

如何校验：前端的密码加密传到后端匹配数据库里面加密后的MD5值

### 事务

什么是事务？

https://blog.csdn.net/dengjili/article/details/82468576

将一组SQL放在一个批次中去执行

> 事务原则：ACID原则	原子性 一致性 隔离性 持久性

#### 事务管理

**原子性（Atomicity）**：原子性表示所有步骤要么一起成功，要么一起失败，不能只发生其中的一个动作。
原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
**一致性（Consistency）**：针对一个事务操作前后状态一致，事务完成后要符合逻辑运算。
事务前后数据的完整性必须保持一致。
**隔离性（Isolation）**：针对多个用户同时操作，主要是排除其他事务对本次事务的影响。
事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。
**持久性（Durability）**：事务结束后数据不随外界原因导致数据丢失。事务一旦提交要持久化到数据库中
持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响

#### 事务的隔离级别

**脏读：**指一个事务读取了另外一个事务未提交的数据。

![image-20210305152951961](C:\Users\mujou\AppData\Roaming\Typora\typora-user-images\image-20210305152951961.png)

**不可重复读：**在一个事务内读取表中的某一行数据，多次读取结果不同。（这个不一定是错误，只是某些场合不对）

>  页面统计查询值
>
> | A    | 100  |
> | ---- | ---- |
> | B    | 200  |
> | C    | 500  |
>
> 点击生成报表的时候，B有人转账进来300（事务已经提交）
>
> | A    | 100  |
> | ---- | ---- |
> | B    | 500  |
> | C    | 500  |

**(虚读)幻读：**是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。（一般是行影响，多了一行）

> | A    | 100  |
> | ---- | ---- |
> | B    | 500  |
> | C    | 500  |
>
> 
>
> | A    | 100  |
> | ---- | ---- |
> | B    | 500  |
> | C    | 500  |
> | D    | 500  |

```sql
--mysql是默认开启事务提交的
set autocommit=0 --关闭
set autocommit=1 --开启
--事务基本都在业务层

--手动处理事务
set autocommit=0 --关闭自动提交
--事务开启
START TRANSACTION --标记一个事务的开始
INSERT XX
INSERT XX
--提交：持久化
COMMIT
--回滚
ROLLBACK
--事务结束
set autocommit=1 --打开自动提交

SAVEPOINT 保存点名 --设置一个事务的保存点
ROLLBACK TO SAVEPOINT --回滚到保存点
RELEASE SAVEPOINT --撤销保存点
```

转装场景模拟

```sql
set autocommit=0 --关闭自动提交
--事务开启
START TRANSACTION --标记一个事务的开始
UPDATE account SET money=money-500 WHERE name='A' --A减500
UPDATE account SET money=money+500 WHERE name='B' --B加500

COMMIT --提交：持久化
ROLLBACK --回滚
--事务结束
set autocommit=1 --打开自动提交

```

事务一旦提交成功就无法回滚

### 索引

- 主键索引（PRIMARY KEY）
  - 唯一的标识，主键不可重复，只能有一个列作为主键
- 唯一索引（UNIQUE KEY）
  - 避免重复列的出现，唯一索引可以重复，多个列都可以标识为唯一索引
- 常规索引（KEY/INDEX）
  - 默认的，index，key关键字来设置
- 全文索引（FullText）
  - 在特定的车数据库引擎下才有，MyISAM
  - 快速的定位数据

```sql
--显示所有的索引信息
SHOW INDEX FROM student
--增加一个全文索引 （索引名）列名
ALTER TABLE school.student ADD FULLTEXT INDEX `studentname`(`studentname`)
--EXPLAIN 分析sql执行状况
EXPLAIN SELECT * FROM student --非全文索引
EXPLAIN SELECT * FROM student WHERE MATCH(studentname) AGAINST('刘')
```

数据库插入百万数据？

索引在小数据量的时候，用处不大，但是在大数据的时候，区别十分明显

#### 索引原则

- 索引不是越多越好
- 不要对经常变动数据加索引、
- 小数据量的表不需要加索引
- 索引一般加在常用来查询的字段上

#### 索引的数据结构

Hash类型的索引

B树：InnoDB默认的数据结构

**查资料这里**



### 数据库备份



### 权限管理



### 数据库归约，三大范式



--业务级别的MySQL学习

--运维级别的MySQL学习

