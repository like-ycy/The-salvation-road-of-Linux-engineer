# MySQL索引

MySQL官方对索引的定义为：索引(Index)是帮助MySQL高效获取数据的数据结构。索引的建立对于MySQL的高效运行是很重要的，索引可以大大提高MySQL的检索速度。

拿汉语字典的目录页（索引）打比方，我们可以按拼音、笔画、偏旁部首等排序的目录（索引）快速查找到需要的字。

索引分单列索引和组合索引。单列索引，即一个索引只包含单个列，一个表可以有多个单列索引，但这不是组合索引。组合索引，即一个索引包含多个列。

创建索引时，你需要确保该索引是应用在 SQL 查询语句的条件(一般作为 WHERE 子句的条件)。

实际上，索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录。

**MySQL中有四种索引**

+ 主键索引
+ 常规索引
+ 唯一索引
+ 全文索引

**索引目的**

为了增加表的查询效率

经过索引的层层筛选，快速定位数据的位置，避免了全表扫描，数据量越大，效率越明显

## 1、主键索引

+ 概述

  主键索引是关系型数据库中最常见的索引类型，主要的作用是确定数据表中数据的确定的位置

+ 设置

  primary  key

+ 注意事项

  + 最好为每张表去指定一个主键，但不是必须的
  + 一个表中只能指定一个主键索引，而且主键的值不能为空，通常配合 auto_increment 来使用
  + 字段类型一般设为 int 类型
  + 主键名称一般为id ，也有叫 表前缀_id

+ 创建表时，创建索引

  ```sql
  # 在末尾添加
  CREATE TABLE test (
    id int(10) unsigned NOT NULL AUTO_INCREMENT,
    username varchar(10) DEFAULT NULL,
    PRIMARY KEY (`id`)
  );
  
  # 或者在一个字段后添加
  CREATE TABLE test (
    id int(10) unsigned NOT NULL PRIMARY KEY (`id`) AUTO_INCREMENT,
    username varchar(10) DEFAULT NULL,
  );
  ```

+ 自增步长(了解)

  mysql的默认步长是基于session，默认为1

  + 查看步长

    show session variables like 'auto_inc%';

  + 设置步长，只针对当前的会话

    set session auto_increment_increment = 2; # 将步长设置为2

  + 设置全局步长

    set global session auto_increment_increment = 2;

  + 注意：

    当表中数据删除以后，自增位置并不会改变

    自增归位：（两种方式）

    + truncate 表名;  清空表并将自增归位
    + alter table 表明  auto_increment=1;

## 2、常规索引（普通索引）

+ 概述

  常规索引是关系型数据库中查询最重要的技术，如果想要提升数据库的效率，那么索引优化是首先要考虑。因为它可以使数据库的性能得到很大的提升

+ 缺点

  + 多占用磁盘空间
  + 会减慢插入、修改和删除的效率

+ 创建索引

  使用 index 或者  key  随同表一起创建

  创建常规索引，使用index创建常规索引，索引名称为u_index

  ```sql
  mysql> create table test4(
      -> id int unsigned primary key auto_increment,
      -> username varchar(10),
      -> index u_index(username)
      -> );
  ```

  使用key创建常规索引，不起别名

  ```sql
  mysql> create table test5(
      -> username varchar(10),
      -> key (username)
      -> );
  ```

  说明：如果创建索引的时候给定了索引名称，则索引名为你给定的名称，否则为字段名


## 3、唯一索引

+ 概述

  唯一索引与主键索引一样，都可以防止创建重复的值，但是不同之处在于，每个表中只能有一个主键索引，但是可以有多个唯一索引 

+ 定义

  unique 创建唯一索引

+ 注意

  如果你在给表中插入数据出现了报错，但是SQL语句还没有问题，那你就要查看一下，当前字段是否存在唯一索引

+ 创建

  创建唯一索引并起名称

  ```sql
  mysql> create table test(
      -> id int unsigned primary key auto_increment,
      -> username varchar(10),
      -> unique u_username(username)
      -> );
  ```

  创建唯一索引不起名称（默认名称为字段名）

  ```sql
  mysql> create table test(
      -> id int unsigned primary key auto_increment,
      -> username varchar(10),
      -> unique(username)
      -> );
  ```

+ 建表以后添加索引 ( 常规或唯一索引 )

  create index 索引名称 on 表名(字段名)

  create index username on test(username);

+ 删除索引

  drop index username on test;

  drop index 索引名称 on 表名

+ 注意

  在给mysql创建唯一或者常规索引的时候，最好单占一行

  一个表中可以存在多个索引，但是要根据具体需求设置索引，前提是某个字段有大量查询的时候

## 4、常规索引和唯一索引的区别

这俩个索引在查询能力上是没有区别的，如果考虑对于更新性能的影响，建议使用常规索引

**常规索引：**将数据页从磁盘读出到内存更新数据页

**唯一索引：**将数据页从磁盘读出到内存，判断是否唯一，再更新数据页

## 5、查看索引执行效率

+ 创建表

  ```sql
  mysql> create table test(
      -> id int unsigned primary key auto_increment,
      -> username varchar(10),
      -> age tinyint default 18
      -> );
  ```

+ 写入数据

  insert into test8 values(null, 'jack', 18);

  insert into test8(username, age) select username,age from test8;

+ 开启mysql运行时间监控

  set profiling = 1;   

+ 开启查询

  select * from test8 where name='zhangsan'

+ 删除索引

  drop index username on test8;

+ 查看时间

  show profiles;

## 6、外键（FOREIGN KEY）

MySQL外键约束（FOREIGN KEY）是表的一个特殊字段，经常与主键约束一起使用。对于两个具有关联关系的表而言，相关联字段中主键所在的表就是主表（父表），外键所在的表就是从表（子表）。

外键用来建立主表与从表的关联关系，为两个表的数据建立连接，约束两个表中数据的一致性和完整性。比如，一个水果摊，只有苹果、桃子、李子、西瓜等 4 种水果，那么，你来到水果摊要买水果就只能选择苹果、桃子、李子和西瓜，其它的水果都是不能购买的。

主表删除某条记录时，从表中与之对应的记录也必须有相应的改变。一个表可以有一个或多个外键，外键可以为空值，若不为空值，则每一个外键的值必须等于主表中主键的某个值。

定义外键时，需要遵守下列规则：

- 主表必须已经存在于数据库中，或者是当前正在创建的表。如果是后一种情况，则主表与从表是同一个表，这样的表称为自参照表，这种结构称为自参照完整性。
- 必须为主表定义主键。
- 主键不能包含空值，但允许在外键中出现空值。也就是说，只要外键的每个非空值出现在指定的主键中，这个外键的内容就是正确的。
- 在主表的表名后面指定列名或列名的组合。这个列或列的组合必须是主表的主键或候选键。
- 外键中列的数目必须和主表的主键中列的数目相同。
- 外键中列的数据类型必须和主表主键中对应列的数据类型相同。

**外键的定义语法：**

```sql
[CONSTRAINT symbol] FOREIGN KEY [id] (index_col_name, ...)
REFERENCES tbl_name (index_col_name, ...)
[ON DELETE {RESTRICT | CASCADE | SET NULL | NO ACTION | SET DEFAULT}]
[ON UPDATE {RESTRICT | CASCADE | SET NULL | NO ACTION | SET DEFAULT}]
```

该语法可以在 CREATE TABLE 和 ALTER TABLE 时使用，如果不指定CONSTRAINT symbol，MYSQL会自动生成一个名字。

- ON DELETE、ON UPDATE表示事件触发限制，可设参数：
  - RESTRICT（限制外表中的外键改动，默认值）
  - CASCADE（跟随外键改动）
  - SET NULL（设空值）
  - SET DEFAULT（设默认值）
  - NO ACTION（无动作，默认的）

### 6.1、创建表时添加外键

**例** 1

为了展现表与表之间的外键关系，本例在 test_db 数据库中创建一个部门表 tb_dept1，表结构如下表所示。

| 字段名称 | 数据类型    | 备注     |
| -------- | ----------- | -------- |
| id       | INT(11)     | 部门编号 |
| name     | VARCHAR(22) | 部门名称 |
| location | VARCHAR(22) | 部门位置 |

创建 tb_dept1 的 SQL 语句和运行结果如下所示。

```sql
mysql> CREATE TABLE tb_dept1
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22) NOT NULL,
    -> location VARCHAR(50)
    -> );
```

创建数据表 tb_emp6，并在表 tb_emp6 上创建外键约束，让它的键 deptId 作为外键关联到表 tb_dept1 的主键 id，SQL 语句和运行结果如下所示。

```sql
mysql> CREATE TABLE tb_emp6
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> CONSTRAINT fk_emp_dept1
    -> FOREIGN KEY(deptId) REFERENCES tb_dept1(id)
    -> );
Query OK, 0 rows affected (0.37 sec)

mysql> DESC tb_emp6;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  | MUL | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (1.33 sec)
```

以上语句执行成功之后，在表 tb_emp6 上添加了名称为 fk_emp_dept1 的外键约束，外键名称为 deptId，其依赖于表 tb_dept1 的主键 id。

> 注意：从表的外键关联的必须是主表的主键，且主键和外键的数据类型必须一致。例如，两者都是 INT 类型，或者都是 CHAR 类型。如果不满足这样的要求，在创建从表时，就会出现“ERROR 1005(HY000): Can't create table”错误。

### 6.2、在修改表时添加外键约束

外键约束也可以在修改表时添加，但是添加外键约束的前提是：从表中外键列中的数据必须与主表中主键列中的数据一致或者是没有数据。

在修改数据表时添加外键约束的语法格式如下：

ALTER TABLE <数据表名> ADD CONSTRAINT <外键名>
FOREIGN KEY(<列名>) REFERENCES <主表名> (<列名>);

**例 2**

修改数据表 tb_emp2，将字段 deptId 设置为外键，与数据表 tb_dept1 的主键 id 进行关联，SQL 语句和运行结果如下所示。

```sql
mysql> ALTER TABLE tb_emp2
    -> ADD CONSTRAINT fk_tb_dept1
    -> FOREIGN KEY(deptId)
    -> REFERENCES tb_dept1(id);
Query OK, 0 rows affected (1.38 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE tb_emp2\G
*************************** 1. row ***************************
       Table: tb_emp2
Create Table: CREATE TABLE `tb_emp2` (
  `id` int(11) NOT NULL,
  `name` varchar(30) DEFAULT NULL,
  `deptId` int(11) DEFAULT NULL,
  `salary` float DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_tb_dept1` (`deptId`),
  CONSTRAINT `fk_tb_dept1` FOREIGN KEY (`deptId`) REFERENCES `tb_dept1` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.12 sec)
```

注意：在为已经创建好的数据表添加外键约束时，要确保添加外键约束的列的值全部来源于主键列，并且外键列不能为空。

### 6.3、删除外键约束

当一个表中不需要外键约束时，就需要从表中将其删除。外键一旦删除，就会解除主表和从表间的关联关系。

删除外键约束的语法格式如下所示：

ALTER TABLE <表名> DROP FOREIGN KEY <外键约束名>;

**例 3**

删除数据表 tb_emp2 中的外键约束 fk_tb_dept1，SQL 语句和运行结果如下所示。

```sql
mysql> ALTER TABLE tb_emp2
    -> DROP FOREIGN KEY fk_tb_dept1;
Query OK, 0 rows affected (0.19 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE tb_emp2\G
*************************** 1. row ***************************
       Table: tb_emp2
Create Table: CREATE TABLE `tb_emp2` (
  `id` int(11) NOT NULL,
  `name` varchar(30) DEFAULT NULL,
  `deptId` int(11) DEFAULT NULL,
  `salary` float DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_tb_dept1` (`deptId`)
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)
```

可以看到，tb_emp2 中已经不存在 FOREIGN KEY，原有的名称为 fk_emp_dept 的外键约束删除成功。
