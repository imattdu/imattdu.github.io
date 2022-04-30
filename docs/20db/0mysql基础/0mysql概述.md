## 概述

### 持久化

持久化(persistence): **把数据保存到可掉电式存储设备中以供之后使用**

掉电设备：数据库、文件、其他

### 数据库概念

#### 数据库 db

存储数据

#### DBMS**:数据库管理系统(**Database Management System**)**

对数据库进行统一管理和控制

#### SQL**:结构化查询语言(**Structured Query Language**)**

专门用来与数据库通信的语言

### 数据库分类

#### 关系型数据库

##### 概述

这种类型的数据库是 最古老 的数据库类型，关系型数据库模型是把复杂的数据结构归结为简单的

二元关系 (即二维表格形式)。

##### 优缺点

**复杂查询** 可以用SQL语句方便的在一个表以及多个表之间做非常复杂的数据查询。

**事务支持** 使得对于安全性能很高的数据访问要求得以实现。

#### 非关系型数据库

**非关系型数据库**，可看成传统关系型数据库的功能 阉割版本 ，基于键值对存储数据，不需要经过SQL层

的解析， 性能非常高 。同时，通过减少不常用的功能，进一步提高性能。

键值数据库

文档数据库

搜索引擎数据库

列式数据库

## 安装

### mac

#### 安装

官网：https://dev.mysql.com/downloads/mysql/

![](https://raw.githubusercontent.com/imattdu/img/main/img/202202120638517.png)

**一步一步安装即可**

**这里选择5.x加密**

![](https://raw.githubusercontent.com/imattdu/img/main/img/202202120151271.png)

设置root密码

![](https://raw.githubusercontent.com/imattdu/img/main/img/202202120152377.png)

#### 启动

在系统偏好设置里启动或者关闭服务

![](https://raw.githubusercontent.com/imattdu/img/main/img/202202120154920.png)

#### 配置

```sh
vim .zshrc
```

```sh
# mysql

PATH=$PATH:/usr/local/mysql/bin
export PATH
```

```sh
mysql -h 主机名 -P 端口号 -u 用户名 -p密码
```

```sh
mysql -h localhost -P 3306 -u root -prootroot


mysql -u root -prootroot


exit
```







“information_schema”是 MySQL 系统自带的数据库，主要保存 MySQL 数据库服务器的系统信息，比如数据库的名称、数据表的名称、字段名称、存取权限、数据文件 所在的文件夹和系统使用的文件夹，等等



“performance_schema”是 MySQL 系统自带的数据库，可以用来监控 MySQL 的各类性能指标。 



“sys”数据库是 MySQL 系统自带的数据库，主要作用是以一种更容易被理解的方式展示 MySQL 数据库服务器的各类性能指标，帮助系统管理员和开发人员监控 MySQL 的技术性能。



“mysql”数据库保存了 MySQL 数据库服务器运行时需要的系统信息，比如数据文件夹、当前使用的字符集、约束检查信息，等等





### 编码

```mysql

 show variables like 'character_%';
 
 show variables like 'collation_%';
```



mysql8.0以上默认会采用utf8





![](https://raw.githubusercontent.com/imattdu/img/main/img/202204231419559.png)





如果不是8.0

可以参考下面文章进行修改

[Mac MySql - 查看以及修改编码格式 - 掘金](https://juejin.cn/post/6877051802447577096)





```mysql
登录 - MySQL内部
mysql -u root -p

MySQL内部 - 查看数据库编码格式 （推荐用这个，我用的这个！）
show variables like 'character%';

MySQL内部 - 查看数据库编码格式 （这两个都可以查看数据库编码格式）
show variables like ‘collation%’;


```





- `character_set_database` 和 `character_set_server` 是 `latin1` 的字符集，也就是说 `mysql` 后续创建的表都是 `latin1` 字符集的，不是 `utf8`，会造成一些麻烦。

- 从以上信息可知数据库的编码为 `latin1`，需要修改为 `gbk` 或者是 `utf8` ；
  
  - `character_set_client`：为客户端编码方式；
  - `character_set_connection`：为建立连接使用的编码；
  - `character_set_database`：数据库的编码；
  - `character_set_results`：结果集的编码；
  - `character_set_server`：数据库服务器的编码；
  - 只要保证以上五个采用的编码方式一样，就不会出现乱码问题。
  
  想要修改编码, 就需要修改 `mysql` 的配置文件 `my.cnf`。



**重要问题：在修改 `my.cnf` 之前一定要关闭 `mysql` 进程，关闭 `mysql`，不然会遇到 `mysql` 的 `sock` 不能连接的问题!**

找到下面路径

```
/usr/local/mysql/support-files
```

在文件夹里面找到 `my-default.cnf` 或者 `my-default` 的文件。

将其复制到桌面上，改名为 `my.cnf`，如果没有，直接桌面新建一个文件，文件改为 `my.cnf`

```
[mysqld]
default-storage-engine=INNODB
character-set-server=utf8
port = 3306

[client]
default-character-set=utf8

```



保存之后，我们需要找到 `/etc` 路径, 将 `my.cnf` 复制贴贴到 `/etc` 这个目录下。可以直接通过快捷键 `command + shift + G` 前面文件:





重启电脑查看mysql编码格式









command + shift + G：移动文件
