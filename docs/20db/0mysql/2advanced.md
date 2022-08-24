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











## 数据类型













### 整数



#### base

整数类型一共有 5 种，包括 TINYINT、SMALLINT、MEDIUMINT、INT（INTEGER）和 BIGINT。



![](https://raw.githubusercontent.com/imattdu/img/main/img/202207230912167.png)





#### 宽度



```sql
CREATE TABLE t31(
	id int(8) UNSIGNED ZEROFILL 
)
```

宽度是8,当输入的字符不足8位时会使用0填充，超过8位也不影响

二者存储空间和宽度无关



**如果没有负数可以使用**

**UNSIGNED**







### 小数



不推荐使用浮点数



#### 浮点数

##### base



 MySQL支持的浮点数类型，分别是 FLOAT、DOUBLE、REAL。



![](https://raw.githubusercontent.com/imattdu/img/main/img/202207230924054.png)



REAL默认就是 DOUBLE。如果你把 SQL 模式设定为启用“ REAL_AS_FLOAT ”，那 么，MySQL 就认为 REAL 是 FLOAT。



```sql
SET sql_mode = “REAL_AS_FLOAT”;
```



MySQL 存储浮点数的格式为： 符号(S) 、 尾数(M) 和 阶码(E) 。因此，无论有没有符号，MySQL 的浮 点数都会存储表示符号的部分。因此， 所谓的无符号数取值范围，其实就是有符号数取值范围大于等于 零的部分。



##### 精度说明



FLOAT(M,D) 或 DOUBLE(M,D) 。这里，M称为 精度 ，D称为 标度 。(M,D)中 M=整数位+小数 位，D=小数位。 D<=M<=255，0<=D<=30。



1.四舍五入后整数超过给定的范围则报错

2.小数超过则不报错，删除多余的小数位并保存



##### 为什么浮点数表示的不精准

**如 果尾数不是 0 或 5（比如 9.624），你就无法用一个二进制数来精确表达。进而，就只好在取值允许的范 围内进行四舍五入**





#### 定点数



##### base

**DECIMAL(M,D),DEC,NUMERIC      M+2字节       有效范围由M和D决定**

DECIMAL(M,D) ,M被称为精度，D被称为标度。0<=M<=65， 0<=D<=30，如果不指定默认为DECIMAL(10,0)。

定点数在MySQL内部是以 字符串 的形式进行存储，这就决定了它一定是精准的。



如果插入的数据超过了定点数指定的范围也四舍五入处理



##### 定点数 浮点数 区别

浮点数：表示的范围大，但是不精准

定点数：范围小但是精准



### 日期时间



#### base



- YEAR 类型通常用来表示年
-  DATE 类型通常用来表示年、月、日
-  TIME 类型通常用来表示时、分、秒
-  DATETIME 类型通常用来表示年、月、日、时、分、秒 
- TIMESTAMP 类型通常用来表示带时区的年、月、日、时、分、秒



#### YEAR



YEAR类型用来表示年份，在所有的日期时间类型中所占用的存储空间最小，只需要 1个字节 的存储空间。





以4位字符串或数字格式表示YEAR类型，其格式为YYYY，最小值为1901，最大值为2155。 

以2位字符串格式表示YEAR类型，最小值为00，最大值为99。 当取值为01到69时，表示2001到2069； 当取值为70到99时，表示1970到1999； 当取值整数的0或00添加的话，那么是0000年； 当取值是日期/字符串的'0'添加的话，是2000年。

从MySQL5.5.27开始，2位格式的YEAR已经不推荐使用。YEAR默认格式就是“YYYY”，没必要写成YEAR(4)， 从MySQL 8.0.19开始，不推荐使用指定显示宽度的YEAR(4)数据类型。





#### DATE



格式为 YYYY-MM-DD ，其中，YYYY表示年份，MM表示月份，DD表示 日期。需要 3个字节 的存储空间。





1.以 YYYY-MM-DD 格式或者 YYYYMMDD 格式表示的字符串日期，其最小取值为1000-01-01，最大取值为 9999-12-03。YYYYMMDD格式会被转化为YYYY-MM-DD格式。 

2.以 YY-MM-DD 格式或者 YYMMDD 格式表示的字符串日期，此格式中，年份为两位数值或字符串满足 YEAR类型的格式条件为：当年份取值为00到69时，会被转化为2000到2069；当年份取值为70到99 时，会被转化为1970到1999。 

3.使用 CURRENT_DATE() 或者 NOW() 函数，会插入当前系统的日期。



#### TIME



3个字节存储



（1）可以使用带有冒号的 字符串，比如' D HH:MM:SS' 、' HH:MM:SS '、' HH:MM '、' D HH:MM '、' D HH '或' SS '格式，都能被正 确地插入TIME类型的字段中。其中D表示天，其最小值为0，最大值为34。如果使用带有D格式的字符串 插入TIME类型的字段时，D会被转化为小时，计算格式为D*24+HH。当使用带有冒号并且不带D的字符串 表示时间时，表示当天的时间，比如12:10表示12:10:00，而不是00:12:10。 

（2）可以使用不带有冒号的 字符串或者数字，格式为' HHMMSS '或者 HHMMSS 。如果插入一个不合法的字符串或者数字，MySQL在存 储数据时，会将其自动转化为00:00:00进行存储。比如1210，MySQL会将最右边的两位解析成秒，表示 00:12:10，而不是12:10:00。 

（3）使用 CURRENT_TIME() 或者 NOW() ，会插入当前系统的时间。







#### DATETIME

8字节

在格式上 为DATE类型和TIME类型的组合，可以表示为 YYYY-MM-DD HH:MM:SS ，其中YYYY表示年份，MM表示月 份，DD表示日期，HH表示小时，MM表示分钟，SS表示秒。



以 YYYY-MM-DD HH:MM:SS 格式或者 YYYYMMDDHHMMSS 格式的字符串插入DATETIME类型的字段时， 最小值为1000-01-01 00:00:00，最大值为9999-12-03 23:59:59。以YYYYMMDDHHMMSS格式的数字插入DATETIME类型的字段时，会被转化为YYYY-MM-DD HH:MM:SS格式。

使用函数 CURRENT_TIMESTAMP() 和 NOW() ，可以向DATETIME类型的字段插入系统的当前日期和 时间。





#### TIMESTAMP



是 YYYY-MM-DD HH:MM:SS ，需要4个字节的存储空间。但是TIMESTAMP存储的时间范围比DATETIME要小很多，只能存储 “1970-01-01 00:00:01 UTC”到“2038-01-19 03:14:07 UTC”之间的时间。其中，UTC表示世界统一时间，也叫 作世界标准时间。



存储数据的时候需要对当前时间所在的时区进行转换，查询数据的时候再将时间转换回当前的时 区。因此，使用TIMESTAMP存储的同一个时间值，在不同的时区查询时会显示不同的时间。



向TIMESTAMP类型的字段插入数据时，当插入的数据格式满足YY-MM-DD HH:MM:SS和YYMMDDHHMMSS 时，两位数值的年份同样符合YEAR类型的规则条件，只不过表示的时间范围要小很多。



如果向TIMESTAMP类型的字段插入的时间超出了TIMESTAMP类型的范围，则MySQL会抛出错误信息。





#### 注意

推荐使用DATETIME和时间戳



### 字符类型



#### base

MySQL中，文本字符串总体上分为 CHAR 、 VARCHAR 、 TINYTEXT 、 TEXT 、 MEDIUMTEXT 、 LONGTEXT 、 ENUM 、 SET 等类型。



![](https://raw.githubusercontent.com/imattdu/img/main/img/202207220023207.png)







#### CHAR VARCHAR

##### 范围

![](https://raw.githubusercontent.com/imattdu/img/main/img/202207220024734.png)



65535 - 需要1-2个字节记录它的长度

65533/3 = 21843





##### what

CHAR类型： CHAR(M) 类型一般需要预先定义字符串长度。如果不指定(M)，则表示长度默认是1个字符。 



如果保存时，数据的实际长度比CHAR类型声明的长度小，则会在 右侧填充空格以达到指定的长度。

当MySQL检索CHAR类型的数据时，CHAR类型的字段会去除尾部的空格。 

定义CHAR类型字段时，声明的字段长度即为CHAR类型字段所占的存储空间的字节数。





VARCHAR(M) 定义时， 必须指定 长度M，否则报错。 MySQL4.0版本以下，varchar(20)：指的是20字节，

MySQL5.0版本以上，varchar(20)：指的是20字符。 

检索VARCHAR类型的字段数据时，会保留数据尾部的空格。**VARCHAR类型的字段所占用的存储空间 为字符串实际长度加1个字节。**





##### 比较



| 类型       | 特点     | 空间上       | 时间上 | 适用场景             |
| ---------- | -------- | ------------ | ------ | -------------------- |
| CHAR(M)    | 固定长度 | 浪费存储空间 | 效率高 | 存储不大，速度要求高 |
| VARCHAR(M) | 可变长度 | 节省存储空间 | 效率低 | 非CHAR的情况         |



固定长度推荐使用CHAR

频繁变更这个列，使用CHAR，因为VARCHAR每次存储都需要额外计算





MyISAM 数据存储引擎和数据列：MyISAM数据表，最好使用固定长度(CHAR)的数据列代替可变长 度(VARCHAR)的数据列。这样使得整个表静态化，从而使 数据检索更快 ，用空间换时间。



 InnoDB 存储引擎，建议使用VARCHAR类型。因为对于InnoDB数据表，内部的行存储格式并没有区 分固定长度和可变长度列（所有数据行都使用指向数据列值的头指针），而且主要影响性能的因素 是数据行使用的存储总量，由于char平均占用的空间多于varchar，所以除了简短并且固定长度的， 其他考虑varchar。这样节省空间，对磁盘I/O和数据存储总量比较好。





#### TEXT



![](https://raw.githubusercontent.com/imattdu/img/main/img/202207220034723.png)





text不允许默认值

而且text和blob类型的数据删除后容易导致 “空洞”，使得文件碎片比较多，所以频繁使用的表不建议包含TEXT类型字段，建议单独分出去，单独用 一个表。

















```sql
CREATE TABLE t1(
	id int NOT NULL,
	`name` VARCHAR(255)
)

ALTER TABLE t1 
MODIFY id INT;

INSERT INTO t1(`name`)
VALUES ('matt')

SELECT * FROM t1;
```











```sql
SHOW TABLES

DROP TABLE t2;

CREATE TABLE t2(
	id int UNIQUE,
	`name` VARCHAR(255)
)

CREATE TABLE t2(
	id int UNIQUE KEY,
	`name` VARCHAR(255)
)

CREATE TABLE t2(
	id int,
	`name` VARCHAR(255)
)

ALTER TABLE t2
ADD UNIQUE(id)

ALTER TABLE t2 
MODIFY id INT;

INSERT INTO t2(id, `name`)
VALUES 
	(2, 'matt'),
	(3, 'matt')
SELECT * FROM t2;

SHOW INDEX FROM t2;

ALTER TABLE t2
DROP INDEX id;

```



