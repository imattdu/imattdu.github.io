## 管理数据库和表





### base

#### 标识符命名规则



- 数据库名、表名不得超过30个字符，变量名限制为29个 
- 必须只能包含 A–Z, a–z, 0–9, _共63个字符 
- 数据库名、表名、字段名等对象名中间不要包含空格 
- 同一个MySQL软件中，数据库不能同名；同一个库中，表不能重名；同一个表中，字段不能重名
- 必须保证你的字段没有和保留字、数据库系统或常用方法冲突。如果坚持使用，请在SQL语句中使 用`（着重号）引起来 
- 保持字段名和类型的一致性：在命名字段并为其指定数据类型的时候一定要保证一致性，假如数据 类型在一个表里是整数，那在另一个表里可就别变成字符型了





### 管理数据库





创建数据库

```sql
CREATE DATABASE IF NOT EXISTS stu_go_1 CHARACTER SET utf8 COLLATE utf8_general_ci
```

查看库表



```sql
# 查看当前服务器有哪些数据库
SHOW DATABASES

# 使用哪个库
USE stu_go

# 查看当前使用的数据库
SHOW DATABASE()

# 查看当前使用数据库有哪些表
SHOW TABLES;

SHOW TABLES FROM stu_go_1
```



数据库修改



```sql
DROP DATABASE IF EXISTS stu_go_1


ALTER DATABASE s1 CHARACTER SET utf8 COLLATE utf8_general_ci;
```











```sql
CREATE TABLE IF NOT EXISTS t2 (
	id INT(11) AUTO_INCREMENT COMMENT 'id',
	`name` VARCHAR(255) NOT NULL DEFAULT '' COMMENT '姓名',
	`age` INT(11) NOT NULL DEFAULT 0 COMMENT '年龄',	
	PRIMARY KEY(id)
)

```







```sql
CREATE TABLE IF NOT EXISTS t3
AS 
SELECT * FROM `user
```



### 创建表





```sql
# 创建表
CREATE TABLE IF NOT EXISTS t2 (
	id INT(11) AUTO_INCREMENT COMMENT 'id',
	`name` VARCHAR(255) NOT NULL DEFAULT '' COMMENT '姓名',
	`age` INT(11) NOT NULL DEFAULT 0 COMMENT '年龄',	
	PRIMARY KEY(id)
)



# 创建表 + 插入表 合二为一
CREATE TABLE IF NOT EXISTS t3
AS 
SELECT * FROM `user`


# 查看表的结构
DESC user
```







### 修改表





```sql
# 添加列
ALTER TABLE t3 
ADD COLUMN f4 VARCHAR(255) NOT NULL DEFAULT '' COMMENT '测试添加列'


# 修改列的类型
ALTER TABLE t3 
MODIFY COLUMN f4 VARCHAR(255) NOT NULL DEFAULT '' COMMENT '测试添加列'




# 不能把string 修改为int
# 修改列的名字
ALTER TABLE t3
CHANGE COLUMN f4 f5 VARCHAR(11)


# 删除列
ALTER TABLE t3
DROP COLUMN f5

# 重命名表1
RENAME TABLE t3
TO t4


# 重命名表2
ALTER TABLE t4
RENAME TO t3


# 删除表
DROP TABLE IF EXISTS t3

# 清空表
TRUNCATE TABLE `user`

```











