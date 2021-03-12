# MySQL

数据库用以减少数据访问的复杂性，DBMS 是管理数据库的软件系统，数据库系统的核心。DBMS 用以限制用户直接对数据文件的访问，作为一中间层，用户以 client 通过网络与之交互。

### 关系型数据库

- 关系：关系就是二维表，其中表的行列次序并不重要
- 行 row：表中的每一行，又称为一条记录
- 列 column：表中的每一列，称为属性、字段 field
- 主键 primary key（pk）：用于唯一确定一个记录的字段；是一属性一特征，一旦作用到某个字段上则不许相同；或关联到多个（复合主键）；
- 域 domain：属性的取值范围
- 外键 foreign key（fk）：表示一个表中的字段被另一个表中的字段引用。

目前关系型数据库的**六种范式**，一般到三范式即可。

- 1NF：无重复的列，每一列都是不可分割的基本数据项，同一列不能有多个值，即实体中的某个属性不能有多个值或者不能有重复的属性，除去同类型的字段，就是无重复的列；说明：第一范式是对关系模型的基本要求，不满足第一范式的数据库就不是关系数据库；
- 2NF：属性完全依赖于主键，第二范式必须先满足第一范式，要求表中的每个行必须可以被唯一地区分。通常为表加上一个列，以存储各个实例的唯一表示 pk，非 pk 的字段需要与整个 pk 有直接相关性。
- 3NF：属性不依赖于其它非主属性，满足第三范式必须先满足第二范式。第三范式要求一个数据库表中不包含在其他表中已包含的非主关键字信息，非 pk 的字段间不能有从属关系。

### SQL 概念

SQL： Structure Query Language 结构化查询语言

数据存储协议：应用层协议，C/S；server 监听于套接字接受并处理客户端的应用请求。

客户端工具：`which mysql`

约束 constraint，表中的数据要遵守的限制

- 主键：一个或多个字段的组合，填入的数据必须能在本表中唯一标识本行；必须提供数据，即 NOT NULL，一个表只能有一个；
- 唯一键 uk：一个或多个字段的组合，填入的数据必须能在本表中唯一标识本行；允许为 NULL，一个表可以存在多个
- 外键：一个表中的某个字段可填入的数据取决于另一个表的主键或唯一键已有的数据
- 检查：字段值在一定范围内

基本概念

- 索引：将表中的一个或多个字段中的数据复制一份另存，并且按特定的次序排序存储
- 选择：挑选出符合条件的行
- 投影：挑选出需要的字段
- 连接：表间字段的关联

数据抽象：

- 物理层：数据存储格式，即 RDBMS 在磁盘上如何组织文件
- 逻辑层：DBA 角度，描述存储什么数据，以及数据间存在什么样的关系
- 视图层：用户角度，描述 DB 中的部分数据

官方文档：

- dev.mysql.com/doc/
- mariadb.com/kb/en

### 首次连接

```bash
systemctl start mariadb
# systemctl enable mariadb
```

特性：

- 插件式存储引擎：也称为“表类型”，MySQL 默认为 InnoDB
- 单进程，多线程

```bash
~]# mysql
MariaDB [(none)]> show engines;
MariaDB [(none)]> status
MariaDB [(none)]> SELECT user();
```

从状态信息中可观察到当前用户为`root@localhost`即用户名加主机的格式，本机当中通过 socket 文件 `/var/lib/mysql/mysql.sock `进行连接：

```bash
[root@localhost ~]# lsof /var/lib/mysql/mysql.sock 
COMMAND  PID  USER   FD   TYPE             DEVICE SIZE/OFF  NODE NAME
mysqld  1931 mysql   15u  unix 0xffff8c4ab9607000      0t0 25760 /var/lib/mysql/mysql.sock
mysqld  1931 mysql   34u  unix 0xffff8c49b757a400      0t0 19168 /var/lib/mysql/mysql.sock
[root@localhost ~]# ps aux | grep mariadb
mysql     1931  0.1  2.2 970940 88768 ?        Sl   15:40   0:00 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock
```

### 安全配置

MySQL 中由 user 和 host 共同组成完整的用户名，从而指定用户能通过哪台主机登陆：

```bash
SHOW databases;
USE mysql;
DESC user;
SELECT user, password, host from user;
```

接着可使用安全配置脚本来设置 root 密码、禁止远程登陆以及禁止匿名用户登陆：

```bash
/usr/bin/mysql_secure_installation
```

当再次查看 user 表时，其 password 字段已有加密信息，并且尝试使用 eth0 网卡登陆时将不被允许：

```bash
~]# mysql -uroot -h192.168.122.91 -p
Enter password: 
ERROR 1130 (HY000): Host '192.168.122.91' is not allowed to connect to this MariaDB server
```

## 数据库操作

```bash
help CREATE DATABASE;
```

###  数据类型

整数型：tinyint，smallint，mediumint，int，bigint
小数型：float，decimal
字符型：char（定长），varchar（变长），text
enum 枚举

修饰符：NULL，NOT NULL，DEFAULT, UNIQUE KEY, AUTO_INCREMENT 自动递增适用于整数, UNSIGNED 加载类型之后取值范围翻倍

#### 创建表

```bash
CREATE TABLE [IF NOT EXISTS] 'tbl_name' (col1 type1 修饰符, col2 type2 修饰符, ..)

MariaDB [(none)]> create database studentdb;
MariaDB [(none)]> use studentdb;
MariaDB [studentdb]> create table student (id int unsigned auto_increment primary key, name varchar(10) not null, sex enum('f', 'm') default 'm', age tinyint unsigned, mobile char(11), address varchar(50));
MariaDB [(none)]> desc student;
MariaDB [(none)]> show columns from student;
MariaDB [(none)]> show create table student;
MariaDB [(none)]> show table status like 'student'\G
MariaDB [(none)]> show table status from mysql\G
```

#### 表操作

```bash
DROP TABLE [IF EXISTS] 'tbl_name';
HELP ALTER TABLE 'tbl_name'
SHOW INDEXES FROM [db_name.]tbl_name; # 查看表上地索引
```

注意：很少修改表结构！

#### DML 语句：Data Manipulation Language

DML: INSERT, DELETE, UPDATE

- INSERT：

```bash
MariaDB [(none)]> help insert;
MariaDB [studentdb]> insert student (name, age, mobile, address) values  ('Tom', 18, '999999', 'Beijing');

MariaDB [studentdb]> alter table student character set = utf8mb4;
MariaDB [studentdb]> status
# 注意服务端与客户端的字符集差别！还要注意字段字符集！


vim /etc/my.cnf
[mysqld]
character-set-server=utf8mb4
vim /etc/my.cnf.d/mysql-clients.cnf
[mysql]
default-character-set=utf8mb4
systemctl restart mysqld
```

- DELETE：一定要记得加 where 限定！

- UPDATE：

```bash
mysql --help
	 -U, --safe-updates  Only allow UPDATE and DELETE that uses keys.
```

#### DQL 语句：Data Query Language

SELECT：

```bash
MariaDB [hellodb]> select gender, count(*) as num from students group by gender;
MariaDB [hellodb]> select gender, avg(age) from students group by gender;
# 注意聚合函数在 select 后面的逻辑！
MariaDB [hellodb]> select gender, avg(age) from students group by gender having gender = 'm';
MariaDB [hellodb]> select * from students order by age limit 3;
MariaDB [hellodb]> select * from students order by classid, age desc;
MariaDB [hellodb]> select classid, avg(age) from students group by classid having avg(age) > 30;
MariaDB [hellodb]> select * from students where age between 20 and 25;
```

#### TCL 语句：Transaction Control Language

#### SQL JOINS



#### 视图

视图本身并不存放数据，内容全部来自于表，因此不修改仅用于查询。

```sql
help view
```

#### 函数

MariaDB [hellodb]> help create function

```sql
MariaDB [hellodb]> create function HelloFun() returns varchar(20) return "Hello World!";
Query OK, 0 rows affected (0.01 sec)

# 其必须嵌入到 select 中使用
MariaDB [hellodb]> select hellofun(); 
+--------------+
| simplefun()  |
+--------------+
| Hello World! |
+--------------+
1 row in set (0.00 sec)

MariaDB [hellodb]> show function status\G
```

#### 存储过程

存储过程把经常使用的 SQL 语句或业务逻辑封装起来，预编译保存在数据库中。查看当下存储过程：

```sql
show procedure status;
```

存储过程保存在`mysql.proc`表中，

```bash
delimiter //
create procedure selectbyid(in uid smallint unsigned) begin select * from students where stuid=uid; end//
delimiter ;

call selectbyid(6);
```





## 配置和优化

select 语句的执行过程：

1. FROM：指明标；
2. WHERE：指明列；
3. GROUP BY：按列中相同的值进行行分组；
4. HAVING：从分组中挑选；
5. ORDER BY：将行进行排序；
6. SELECT：挑选列；
7. LIMIT：限定行数。

#### 用户帐号和权限管理

忘记 root 用户密码时，在`/etc/my.cnf`文件`[mysqld]`下添加：

```bash
sikp-grant-tables
```

之后再重启服务即可。

grant 命令用于创建用户及授权，

### MySQL 体系结构

连接管理模块，线程管理模块，用户管理模块，命令分发器，命令解析器，存储引擎接口等等。

#### 存储引擎

InnoDB：

- 支持事务；
- 行级锁；
- MVCC（多本版并发控制机制）高并发；
- 可缓存数据及索引；

何为为每张表自动添加记录时间 的 create，delete 字段？

## 游标
[ref](https://blog.csdn.net/qq_27825451/article/details/81477618)
游标是 SQL 的一种数据访问机制，是一种处理数据的方法。
SQL 的 select 查询操作返回的结果是一个包含一行或者多行的数据集，此时无法对结果再进行简单的查询。
我们可以将游标看作是“结果集”上的指针，可根据需要在结果集上查询。

## 索引
[https://blog.csdn.net/weiliangliang111/article/details/51333169](https://blog.csdn.net/weiliangliang111/article/details/51333169)
如果没有索引，单次的查找会触发全表扫描。
使用索引，就是为了通过缩小一张表中需要查询的记录/行的数目来加快查询速度。

**索引是一种数据结构：B-Tree** 是最常用的数据结构。[wiki](https://zh.wikipedia.org/wiki/B%E6%A0%91)
B-tree 是一种自平衡的树，能够保持数据有序。这种数据结构能够让查找数据、顺序访问、插入和删除都能在对数时间内完成。
数据库索引里并不存储表中的其他字段，而存储了指向表中某一行的指针。就像一本书的所以一样，可以通过索引页找到有关信息并且还包含页码。
**使用索引的代价：**索引占用额外的空间，表越大索引越大；性能损失，当你在对表进行增删或更新操作时，在索引中也会有相同的操作，建立在某列的索引需要保存最新数据。当某列在查询时使用非常频繁时，那就应该建立索引。

## 事务
[wiki](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1)
事务是数据库管理系统执行过程中的一个逻辑单位，由一个有限的数据库操作序列构成。
[liaoxuefeng](https://www.liaoxuefeng.com/wiki/1177760294764384/1179611198786848)某些时候应业务要求，一系列操作必须全部执行，而不能进执行一部分，例如一个转账操作。如果一成功而第二条语句失败，则需要全部撤销。

ACID 性质：
- A，Atomic 原子性：所有 SQL 要么全部执行，要么全部不执行；
- C，Consistent 一致性：事务完成后所有数据的状态是一致的，即转账 A 账户进去 10 元，则 B 必须增加 10 元；
- I，Isolation 隔离性：如果多个事务并发执行，每个事物做出的修改必须与其他事务隔离；
- D，Duration 持久性：即事务完成后，对数据口数据的修改被持久化存储。

### 隔离级别
对于并发执行的事务，如果涉及到操作同一条记录的时候，可能会因为并发操作带来数据的不一致性，包括脏读、不可重复读、幻读。
数据库提供了 4 中隔离级别，分别对应的可能出现的数据不一致的情况。

## 三范式
[ref 多理解其中的举例](https://blog.csdn.net/hanxueyu666/article/details/81587199)
Normal Form，NF：
- 1NF：强调的是列的原子性，即列不能够再分成其他几列；
- 2NF：首先满足 1NF 后，表必须有一个主键，同时没有包含在主键中的列必须完全依赖于主键，而不能只依赖于主键（可以是多个列）的一部分；
- 3NF：首先满足 2NF 后，非主键列必须直接依赖于主键，不能存在传递依赖（即不能存在非主键 A 依赖于非主键 B，而 B 才依赖于主键）；

## MySQL 对比 Redis
[https://zhuanlan.zhihu.com/p/141385393](https://zhuanlan.zhihu.com/p/141385393)
mysql 是关系型数据库，主要在硬盘中存放持久化数据，读取速度较慢；
redis 是 nosql 非关系型数据库，也是缓存数据库，将数据存放在主存当中，运行效率高但保存时间有限。

大多数策略是 MySQL + Redis，redis 被用作主存的缓存加快访问速度。











