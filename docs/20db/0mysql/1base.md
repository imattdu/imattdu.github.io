## sql介绍

### sql分类

SQL语言在功能上主要分为如下3大类:

DDL**(**Data Definition Languages**、数据定义语言)**，这些语句定义了不同的数据库、表、视图、索 引等数据库对象，还可以用来创建、删除、修改数据库和数据表的结构。

主要的语句关键字包括 CREATE 、 DROP 、 ALTER 等。



 DML**(**Data Manipulation Language**、数据操作语言)**，用于添加、删除、更新和查询数据库记 录，并检查数据完整性。

主要的语句关键字包括 INSERT 、 DELETE 、 UPDATE 、 SELECT 等。

SELECT**是**SQL**语言的基础，最为重要。
** DCL**(**Data Control Language**、数据控制语言)**，用于定义数据库、表、字段、用户的访问权限和 安全级别。

主要的语句关键字包括 GRANT 、 REVOKE 、 COMMIT 、 ROLLBACK 、 SAVEPOINT 等。







因为查询语句使用的非常的频繁，所以很多人把查询语句单拎出来一类:DQL(数据查询语言)。

还有单独将 COMMIT 、 ROLLBACK 取出来称为TCL (Transaction Control Language，事务控制语言)。





### 规范



必须使用英文状态下的半角输入方式 

字符串型和日期时间类型的数据可以使用单引号(' ')表示 

列的别名，尽量使用双引号(" ")，而且不建议省略as





数据库名、表名、表别名、字段名、字段别名等都小写 

SQL 关键字、函数名、绑定变量等都大写





单行注释: #注释文字(MySQL特有的方式) 

单行注释:-- 注释文字(--后面必须包含一个空格。)

 多行注释:/* 注释文字 */







## 基础操作





### 导入





```mysql
source xxx.sql
```



### 基础查询



```sql
SELECT * FROM `user`;

SELECT `name` 
FROM `user`;

# 别名
SELECT id, `name` as "姓名"
FROM user;


# 去重
select DISTINCT id
from `user`;
```

### 去重

**DISTINCT 其实是对后面所有列名的组合进行去重，你能看到最后的结果是 74 条，因为这 74 个a不同，都有 b 这个属性值。如果你想要看都有哪些不同的a，只需要写 a即可，后面不需要再加其他的列名了。**



### 空值 null



在 MySQL 里面， 空值不等于空字符串。一个空字符串的长度是 0，而一个空值的长度是空。而且，在 MySQL 里面，空值是占用空间的。

所有运算符或列值遇到null值，运算的结果都为null



### 反引号

我们需要保证表中的字段、表名等没有和保留字、数据库系统或常用方法冲突。如果真的相同，请在

SQL语句中使用一对``(着重号)引起来。

### 常数列



```sql
SELECT id, `name`, 'matt'
FROM `user`
```





### desc



```sql
desc user;
```



![](https://raw.githubusercontent.com/imattdu/img/main/img/202204231614478.png)



- Field:表示字段名称。
- Type:表示字段类型，这里 barcode、goodsname 是文本型的，price 是整数类型的。 
- Null:表示该列是否可以存储NULL值。 
- Key:表示该列是否已编制索引。PRI表示该列是表主键的一部分;UNI表示该列是UNIQUE索引的一部分;MUL表示在列中某个给定值允许出现多次。 
- Default:表示该列是否有默认值，如果有，那么值是多少。
- Extra:表示可以获取的与给定列有关的附加信息，例如AUTO_INCREMENT等。









## 运算符



### 算数运算符

```sql
+ - * / % 
```



**在Java中，+的左右两边如果有字符串，那么表示字符串的拼接。但是在MySQL中+只表示数 值相加。如果遇到非数值类型，先尝试转成数值，如果转失败，就按0计算。(补充:MySQL 中字符串拼接要使用字符串函数CONCAT()实现)**





一个数除以整数后，不管是否能除尽，结果都为一个浮点数; 一个数除以另一个数，除不尽时，结果为一个浮点数，并保留到小数点后4位;

在数学运算中，0不能用作除数，在MySQL中，一个数除以0为NULL。





### 比较运算符



```sql
>
>=
<
<=
!=

=

# 不等于
<>
# 安全等于
<=>
```



#### 等号

如果等号两边的值、字符串或表达式都为字符串，则MySQL会按照字符串进行比较，其比较的 是每个字符串中字符的ANSI编码是否相等。

如果等号两边的值都是整数，则MySQL会按照整数来比较两个值的大小。

如果等号两边的值一个是整数，另一个是字符串，则MySQL会将字符串转化为数字进行比较。 

如果等号两边的值、字符串或表达式中有一个为NULL，则比较结果为NULL。





#### null

```sql
SELECT 1 is NULL, 1 is NOT NULL
```



#### least greatest

最小值 最大值

```SQL
SELECT LEAST(1,2), GREATEST(1, 2)
```

参数是整数或者浮点数时，LEAST将返回其中最小的值;当参数为字符串时，返回字 母表中顺序最靠前的字符;当比较值列表中有NULL时，不能判断大小，返回值为NULL。





#### between and



```SQL
# 1 <= id <= 3
SELECT *
FROM `user`
WHERE id BETWEEN 1 AND 3
```



#### in not in

in:在

not in:不在



```sql
SELECT *
FROM `user`
WHERE id in(2, 4)
```



#### like

模糊匹配



```sql
SELECT *
FROM `user`
WHERE `name` like '%m%'
```

“%”:匹配0个或多个字符。

 “_”:只能匹配一个字符。



#### ESCAPE



```sql
SELECT job_id
FROM jobs
WHERE job_id LIKE ‘IT\_%‘;
```

如果使用\表示转义，要省略ESCAPE。如果不是\，则要加上ESCAPE。

```sql
SELECT job_id
FROM jobs
WHERE job_id LIKE ‘IT$_%‘ escape ‘$‘;
```





#### REGEXP运算符



（1）‘^’匹配以该字符后面的字符开头的字符串。 

（2）‘$’匹配以该字符前面的字符结尾的字符串。 

（3）‘.’匹配任何一个单字符。 

（4）“[...]”匹配在方括号内的任何字符。例如，“[abc]”匹配“a”或“b”或“c”。为了命名字符的范围，使用一 个‘-’。“[a-z]”匹配任何字母，而“[0-9]”匹配任何数字。 

（5）‘*’匹配零个或多个在它前面的字符。例如，“x*”匹配任何数量的‘x’字符，“[0-9]*”匹配任何数量的数字， 而“*”匹配任何数量的任何字符。





```sql
 SELECT 'shkstart' REGEXP '^s', 'shkstart' REGEXP 't$', 'shkstart' REGEXP 'hk';
```



```sql
SELECT *
FROM `user`
WHERE `name` REGEXP 'Lo'
```



### 逻辑运算符



and 与

or 或

not 非

xor 异或





### 位运算



| 运算符 |   作用   |
| :----: | :------: |
|   &    |  按位与  |
|   \|   |  按位或  |
|   ^    | 按位异或 |
|   ~    | 按位取反 |
|   >>   |   右移   |
|   <<   |   左移   |









## 排序 分页





### 排序



```sql
select *
from `user`
order by id desc, name desc
```



### 分页

**LIMIT [位置偏移量,] 行数**



从0开始



10：10条记录

0 9

```SQL
select *
from `user`
order by id desc, name desc
LIMIT 0, 10

```



## 多表查询



### 基础

#### 等值连接



```sql
SELECT a.a1, b.b1, b.a1
FROM ABC a, BCD b
WHERE a.a1 = b.a1
```

**多个表中有相同列时，必须在列名之前加上表名前缀。**



#### 非等值连接





```sql
SELECT a.a1, b.b1, b.a1
FROM ABC a, BCD b
WHERE a.a1 > b.a1
```



#### 自连接 非自连接

一张表



俩张及以上的表进行连接



#### 内连接 外连接





**1.内连接就是取交集的部分。**
**2.左连接就是左表全部的数据加上交集的数据。**
**3.右连接就是右表全部的数据加上交集的数据。**
**4.交叉连接就是全都要**



左外连接：左表是主表





### c99





```sql
SELECT a.*
FROM A a INNER JOIN B b
ON a.id = b.id
INNER JOIN C c
ON c.id = b.id
```



```sql
SELECT a.*
FROM A a LEFT OUTER JOIN B b
ON a.id = b.id
```





```SQL
SELECT a.*
FROM A a RIGHT OUTER JOIN B b
ON a.id = b.id
```



**需要注意的是，LEFT JOIN 和 RIGHT JOIN 只存在于 SQL99 及以后的标准中，在 SQL92 中不存在， 只能用 (+) 表示。**







满外连接的结果 = 左右表匹配的数据 + 左表没有匹配到的数据 + 右表没有匹配到的数据。 

SQL99是支持满外连接的。使用FULL JOIN 或 FULL OUTER JOIN来实现。 

需要注意的是，MySQL不支持FULL JOIN，但是可以用 LEFT JOIN UNION RIGHT join代替。









#### union



union:去重 (a, b, ab)

union all:不去重 (a, b, ab, ab)



```sql
select * from a
union
select * from b
```









![](https://raw.githubusercontent.com/imattdu/img/main/img/202204250001072.png)





左中图

```sql
#实现A - A∩B
select 字段列表
from A表 left join B表
on 关联条件
where 从表关联字段 is null and 等其他子句;
```





右中图

```sql
#实现B - A∩B
select 字段列表
from A表 right join B表
on 关联条件
where 从表关联字段 is null and 等其他子句;
```







左下图



```sql
实现查询结果是A∪B
#用左外的A，union 右外的B
select 字段列表
from A表 left join B表
on 关联条件
where 等其他子句
union
select 字段列表
from A表 right join B表
on 关联条件
where 等其他子句;
```









```sql
#实现A∪B - A∩B 或 (A - A∩B) ∪ （B - A∩B）
#使用左外的 (A - A∩B) union 右外的（B - A∩B）
select 字段列表
from A表 left join B表
on 关联条件
where 从表关联字段 is null and 等其他子句
union
select 字段列表
from A表 right join B表
on 关联条件
where 从表关联字段 is null and 等其他子句
```









## 聚合函数







### 函数介绍



#### AVG SUM

可以对数值型数据使用AVG 和 SUM 函数





#### MIN MAX

可以对任意数据类型的数据使用 MIN 和 MAX 函数。



#### COUNT

COUNT(*)返回表中记录总数，适用于任意数据类型。

COUNT(expr) 返回expr不为空的记录总数。





问题：用count(*)，count(1)，count(列名)谁好呢?

对于MyISAM引擎的表是没有区别的。这种引擎内部有一计数器在维护着行数。 Innodb引擎的表用count(*),count(1)直接读行数，复杂度是O(n)，因为innodb真的要去数一遍。但好 于具体的count(列名)。 



问题：能不能使用count(列名)替换count(*)? *

不要使用 count(列名)来替代 count(*) ， count(*) 是 SQL92 定义的标准统计行数的语法，跟数 据库无关，跟 NULL 和非 NULL 无关。 说明：count(*)会统计值为 NULL 的行，而 count(列名)不会统计此列为 NULL 值的行。







### GROUP BY





```sql
SELECT id, `name`, SUM(id)
FROM user
where age > 18
GROUP BY id, name
```

**在SELECT列表中所有未包含在组函数中的列都应该包含在 GROUP BY子句中**



```sql
SELECT id, `name`, SUM(id)
FROM user
where age > 18
GROUP BY id, name WITH ROLLUP
```

WITH ROLLUP:所有记录求聚合,

当使用ROLLUP时，不能同时使用ORDER BY子句进行结果排序，即ROLLUP和ORDER BY是互相排斥 的。





### HAVING



having 需要和group by搭配使用



```sql
SELECT `name` ,max(id)
FROM `user`
GROUP BY name
HAVING max(ID) > 1
```





区别1：WHERE 可以直接使用表中的字段作为筛选条件，但不能使用分组中的计算函数作为筛选

区别2：如果需要通过连接从关联表中获取需要的数据，WHERE 是先筛选后连接，而 HAVING 是先连接 后筛选。













## 子查询





### 单行子查询



#### operator



| op   | meaning  |
| ---- | -------- |
| =    | 等于     |
| >    | 大于     |
| >=   | 大于等于 |
| <    | 小于     |
| <=   | 小于等于 |
| <>   | 不等于   |







#### 案例

##### 返回公司工资最少的员工的last_name,job_id和salary

```sql
SELECT last_name, job_ib, salary
FROM employees
WHERE salary = (
	SELECT MIN(salary)
  FROM employees
)
AND employee_id NOT IN (174, 141)
```

##### 查询与141号或174号员工的manager_id和department_id相同的其他员工的employee_id， manager_id，department_id



不成对比较

```sql
SELECT employee_id, manager_id, department_id
FROM employees
WHERE manager_id in (
	SELECT manager_id
  FROM employees
  WHERE employee_id IN (174, 141)
)
AND department_id IN (
	SELECT department_id 
  FROM employees
  WHERE employee_id IN (174, 141)
)
AND employee_id NOT IN (174, 141)
```





#### case 子查询

显式员工的employee_id,last_name和location。其中，若员工department_id与location_id为1800 的department_id相同，则location为’Canada’，其余则为’USA’。



```sql
SELECT employee_id, last_name, (
	CASE department_id
  WHEN (
  	SELECT department_id
    FROM departments
    WHERE location_id = 1800
    THEN 'Canada' 
    ELSE 'USA' 
    END
  ) 
) location
FROM employees
```









#### 注意事项

如果子查询返回空则主查询也没数据

多行子查询使用单行比较符









### 多行子查询





#### 操作符

| op   | meaning 含义                                             |
| ---- | -------------------------------------------------------- |
| IN   | 任意一个                                                 |
| ANY  | 需要和单行比较操作符一起使用，和子查询返回的某一个值比较 |
| ALL  | 需要和单行比较操作符一起使用，和子查询返回的所有值比较   |
| SOME | 实际上是ANY的别名，作用相同，一般常使用ANY               |







#### 案例

##### 返回其它job_id中比job_id为'IT_PROG'部门**任一**工资低的员工的员工号、姓名、job_id 以及salary



```sql
SELECT employee_id, last_name, job_id, salary
FROM employee
WHERE salary < ANY (
	SELECT salary
  FROM employees
  WHERE job_id = 'IT_PROG'
)
```





返回其它job_id中比job_id为'IT_PROG'部门**所有**工资低的员工的员工号、姓名、job_id 以及salary



```sql
SELECT employee_id, last_name, job_id, salary
FROM employee
WHERE salary < ALL (
	SELECT salary
  FROM employees
  WHERE job_id = 'IT_PROG'
)
```





##### 查询平均工资最低的部门id





方式一



```sql
SELECT department_id
FROM employee
GROUP BY department_id
HAVING AVG(salary) = (
	SELECT MIN(avg_sal)
  FROM (
  	SELECT department_id, AVG(salary) avg_sal
    FROM employees
    GROUP BY department_id
  ) dept_avg_sal
)
```



##### 方式二 <= ALL



```sql
SELECT department_id
FROM employee
GROUP BY department_id
HAVING AVG(salary) <= ALL (
	SELECT avg_sal
  FROM (
  	SELECT department_id, AVG(salary) avg_sal
    FROM employees
    GROUP BY department_id
  ) dept_avg_sal
)
```









### 关联查询





#### definition/define 定义



关联子查询：每执行一次外部查询，子查询需要重新计算一次





#### 基础案例



##### 查询员工中工资大于本部门平均工资的员工的last_name,salary和其department_id





相关子查询

```sql
SELECT last_name, salary, department_id
FROM employees emp_out
WHERE salary > (
	SELECT AVG(salary)
  FROM employee
  WHERE department_id = emp_out.deparment_id
)
```

from子查询

```sql
SELECT e1.last_name, e1.salary, e1.department_id
FROM employees e1, (
  SELECT deaparment_id, AVG(salary) avg_sal_dept
  FROM employeees
  GROUP BY deparment_id
) e2
WHERE e1.department_id = e2.department_id
AND e1.salary > e2.avg_sal_dept
```





order by 子查询

查询员工的id,salary,按照department_name 排序



```sql
SELECT employee_id, salary
FROM employee e
ORDER BY (
	SELECT department_name
  FROM departments d
  where e.department_id = d.department_id
)
```







#### exists

##### what

EXISTS

存在则不继续查找了，并返回TRUE



NOT EXISTS

存在则不继续查找了，并返回TRUE





##### how

查询公司管理者的employee_id，last_name，job_id，department_id信息



```sql
SELECT employee_id, last_name, department_id
FROM employees e1
WHERE EXISTS (
  SELECT *
  FROM employee e2
  WHERE e1.manager_id = e2.employee_id
)

```

