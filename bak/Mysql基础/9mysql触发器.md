# MySQL触发器

MySQL 的触发器和存储过程一样，都是嵌入到 MySQL 中的一段程序，是 MySQL 中管理数据的有力工具。不同的是执行存储过程要使用 CALL 语句来调用，而触发器的执行不需要使用 CALL 语句来调用，也不需要手工启动，而是通过对数据表的相关操作来触发、激活从而实现执行。比如当对 student 表进行操作（INSERT，DELETE 或 UPDATE）时就会激活它执行。

触发器与数据表关系密切，主要用于保护表中的数据。特别是当有多个表具有一定的相互联系的时候，触发器能够让不同的表保持数据的一致性。

在 MySQL 中，只有执行 INSERT、UPDATE 和 DELETE 操作时才能激活触发器，其它 SQL 语句则不会激活触发器。

那么为什么要使用触发器呢？比如，在实际开发项目时，我们经常会遇到以下情况：

- 在学生表中添加一条关于学生的记录时，学生的总数就必须同时改变。
- 增加一条学生记录时，需要检查年龄是否符合范围要求。
- 删除一条学生信息时，需要删除其成绩表上的对应记录。
- 删除一条数据时，需要在数据库存档表中保留一个备份副本。


虽然上述情况实现的业务逻辑不同，但是它们都需要在数据表发生更改时，自动进行一些处理。这时就可以使用触发器处理。例如，对于第一种情况，可以创建一个触发器对象，每当添加一条学生记录时，就执行一次计算学生总数的操作，这样就可以保证每次添加一条学生记录后，学生总数和学生记录数是一致的。

## 触发器的优缺点

触发器的优点如下：

- 触发器的执行是自动的，当对触发器相关表的数据做出相应的修改后立即执行。
- 触发器可以实施比 FOREIGN KEY 约束、CHECK 约束更为复杂的检查和操作。
- 触发器可以实现表数据的级联更改，在一定程度上保证了数据的完整性。


触发器的缺点如下：

- 使用触发器实现的业务逻辑在出现问题时很难进行定位，特别是涉及到多个触发器的情况下，会使后期维护变得困难。
- 大量使用触发器容易导致代码结构被打乱，增加了程序的复杂性，
- 如果需要变动的数据量较大时，触发器的执行效率会非常低。

## MySQL 支持的触发器

在实际使用中，MySQL 所支持的触发器有三种：INSERT 触发器、UPDATE 触发器和 DELETE 触发器。

#### 1) INSERT 触发器

在 INSERT 语句执行之前或之后响应的触发器。

使用 INSERT 触发器需要注意以下几点：

- 在 INSERT 触发器代码内，可引用一个名为 NEW（不区分大小写）的虚拟表来访问被插入的行。
- 在 BEFORE INSERT 触发器中，NEW 中的值也可以被更新，即允许更改被插入的值（只要具有对应的操作权限）。
- 对于 AUTO_INCREMENT 列，NEW 在 INSERT 执行之前包含的值是 0，在 INSERT 执行之后将包含新的自动生成值。

#### 2) UPDATE 触发器

在 UPDATE 语句执行之前或之后响应的触发器。

使用 UPDATE 触发器需要注意以下几点：

- 在 UPDATE 触发器代码内，可引用一个名为 NEW（不区分大小写）的虚拟表来访问更新的值。
- 在 UPDATE 触发器代码内，可引用一个名为 OLD（不区分大小写）的虚拟表来访问 UPDATE 语句执行前的值。
- 在 BEFORE UPDATE 触发器中，NEW 中的值可能也被更新，即允许更改将要用于 UPDATE 语句中的值（只要具有对应的操作权限）。
- OLD 中的值全部是只读的，不能被更新。


注意：当触发器设计对触发表自身的更新操作时，只能使用 BEFORE 类型的触发器，AFTER 类型的触发器将不被允许。

#### 3) DELETE 触发器

在 DELETE 语句执行之前或之后响应的触发器。

使用 DELETE 触发器需要注意以下几点：

- 在 DELETE 触发器代码内，可以引用一个名为 OLD（不区分大小写）的虚拟表来访问被删除的行。
- OLD 中的值全部是只读的，不能被更新。


总体来说，触发器使用的过程中，MySQL 会按照以下方式来处理错误。

对于事务性表，如果触发程序失败，以及由此导致的整个语句失败，那么该语句所执行的所有更改将回滚；对于非事务性表，则不能执行此类回滚，即使语句失败，失败之前所做的任何更改依然有效。

若 BEFORE 触发程序失败，则 MySQL 将不执行相应行上的操作。

若在 BEFORE 或 AFTER 触发程序的执行过程中出现错误，则将导致调用触发程序的整个语句失败。

仅当 BEFORE 触发程序和行操作均已被成功执行，MySQL 才会执行 AFTER 触发程序。

## NEW与OLD详解

MySQL 中定义了 NEW 和 OLD，用来表示触发器的所在表中，触发了触发器的那一行数据，来引用触发器中发生变化的记录内容，具体地：

　　①在INSERT型触发器中，NEW用来表示将要（BEFORE）或已经（AFTER）插入的新数据；

　　②在UPDATE型触发器中，OLD用来表示将要或已经被修改的原数据，NEW用来表示将要或已经修改为的新数据；

　　③在DELETE型触发器中，OLD用来表示将要或已经被删除的原数据；

**使用方法：**

NEW.columnName （columnName为相应数据表某一列名）

另外，OLD是只读的，而NEW则可以在触发器中使用 SET 赋值，这样不会再次触发触发器，造成循环调用（如每插入一个学生前，都在其学号前加“2013”）。

## 创建触发器

触发器是与 MySQL数据表有关的数据库对象，在满足定义条件时触发，并执行触发器中定义的语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性。

### 基本语法

在 MySQL 5.7 中，可以使用 CREATE TRIGGER 语句创建触发器。

语法格式如下：

```sql
CREATE TRIGGER <触发器名> 
< BEFORE | AFTER >
<INSERT | UPDATE | DELETE >
ON <表名> FOR EACH ROW
BEGIN
sql语句（触发的语句可以是一条也可以是多条）
END;
```

语法说明如下：

#### 1) 触发器名

触发器的名称，触发器在当前数据库中必须具有唯一的名称。如果要在某个特定数据库中创建，名称前面应该加上数据库的名称。

#### 2) INSERT | UPDATE | DELETE

触发事件，用于指定激活触发器的语句的种类。

注意：三种触发器的执行时间如下。

- INSERT：将新行插入表时激活触发器。例如，INSERT 的 BEFORE 触发器不仅能被 MySQL 的 INSERT 语句激活，也能被 LOAD DATA 语句激活。
- DELETE： 从表中删除某一行数据时激活触发器，例如 DELETE 和 REPLACE 语句。
- UPDATE：更改表中某一行数据时激活触发器，例如 UPDATE 语句。

#### 3) BEFORE | AFTER

BEFORE 和 AFTER，触发器被触发的时刻，表示触发器是在激活它的语句之前或之后触发。若希望验证新数据是否满足条件，则使用 BEFORE 选项；若希望在激活触发器的语句执行之后完成几个或更多的改变，则通常使用 AFTER 选项。

#### 4) 表名

与触发器相关联的表名，此表必须是永久性表，不能将触发器与临时表或视图关联起来。在该表上触发事件发生时才会激活触发器。同一个表不能拥有两个具有相同触发时刻和事件的触发器。例如，对于一张数据表，不能同时有两个 BEFORE UPDATE 触发器，但可以有一个 BEFORE UPDATE 触发器和一个 BEFORE INSERT 触发器，或一个 BEFORE UPDATE 触发器和一个 AFTER UPDATE 触发器。

#### 5) FOR EACH ROW

一般是指行级触发，对于受触发事件影响的每一行都要激活触发器的动作。例如，使用 INSERT 语句向某个表中插入多行数据时，触发器会对每一行数据的插入都执行相应的触发器动作。

> 注意：每个表都支持 INSERT、UPDATE 和 DELETE 的 BEFORE 与 AFTER，因此每个表最多支持 6 个触发器。每个表的每个事件每次只允许有一个触发器。单一触发器不能与多个事件或多个表关联。

### 创建 BEFORE 类型触发器

在 test_db 数据库中，数据表 tb_emp8 为员工信息表，包含 id、name、deptId 和 salary 字段，数据表 tb_emp8 的表结构如下所示。

```sql
mysql> SELECT * FROM tb_emp8;
Empty set (0.07 sec)
mysql> DESC tb_emp8;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(22) | YES  | UNI | NULL    |       |
| deptId | int(11)     | NO   | MUL | NULL    |       |
| salary | float       | YES  |     | 0       |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.05 sec)
```

【实例 1】创建一个名为 SumOfSalary 的触发器，触发的条件是向数据表 tb_emp8 中插入数据之前，对新插入的 salary 字段值进行求和计算。输入的 SQL 语句和执行过程如下所示。

```sql
mysql> CREATE TRIGGER SumOfSalary
    -> BEFORE INSERT ON tb_emp8
    -> FOR EACH ROW
    -> SET @sum=@sum+NEW.salary;
Query OK, 0 rows affected (0.35 sec)
```

触发器 SumOfSalary 创建完成之后，向表 tb_emp8 中插入记录时，定义的 sum 值由 0 变成了 1500，即插入值 1000 和 500 的和，如下所示。

```sql
SET @sum=0;
Query OK, 0 rows affected (0.05 sec)
mysql> INSERT INTO tb_emp8
    -> VALUES(1,'A',1,1000),(2,'B',1,500);
Query OK, 2 rows affected (0.09 sec)
Records: 2  Duplicates: 0  Warnings: 0
mysql> SELECT @sum;
+------+
| @sum |
+------+
| 1500 |
+------+
1 row in set (0.03 sec)
```

### 创建 AFTER 类型触发器

在 test_db 数据库中，数据表 tb_emp6 和 tb_emp7 都为员工信息表，包含 id、name、deptId 和 salary 字段，数据表 tb_emp6 和 tb_emp7 的表结构如下所示。

```sql
mysql> SELECT * FROM tb_emp6;
Empty set (0.07 sec)
mysql> SELECT * FROM tb_emp7;
Empty set (0.03 sec)
mysql> DESC tb_emp6;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  | MUL | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)
mysql> DESC tb_emp7;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | 0       |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.04 sec)
```

【实例 2】创建一个名为 double_salary 的触发器，触发的条件是向数据表 tb_emp6 中插入数据之后，再向数据表 tb_emp7 中插入相同的数据，并且 salary 为 tb_emp6 中新插入的 salary 字段值的 2 倍。输入的 SQL 语句和执行过程如下所示。

```sql
mysql> CREATE TRIGGER double_salary
    -> AFTER INSERT ON tb_emp6
    -> FOR EACH ROW
    -> INSERT INTO tb_emp7
    -> VALUES (NEW.id,NEW.name,deptId,2*NEW.salary);
Query OK, 0 rows affected (0.25 sec)
```

触发器 double_salary 创建完成之后，向表 tb_emp6 中插入记录时，同时向表 tb_emp7 中插入相同的记录，并且 salary 字段为 tb_emp6 中 salary 字段值的 2 倍，如下所示。

```sql
mysql> INSERT INTO tb_emp6
    -> VALUES (1,'A',1,1000),(2,'B',1,500);
Query OK, 2 rows affected (0.09 sec)
Records: 2  Duplicates: 0  Warnings: 0
mysql> SELECT * FROM tb_emp6;
+----+------+--------+--------+
| id | name | deptId | salary |
+----+------+--------+--------+
|  1 | A    |      1 |   1000 |
|  2 | B    |      1 |    500 |
+----+------+--------+--------+
3 rows in set (0.04 sec)
mysql> SELECT * FROM tb_emp7;
+----+------+--------+--------+
| id | name | deptId | salary |
+----+------+--------+--------+
|  1 | A    |      1 |   2000 |
|  2 | B    |      1 |   1000 |
+----+------+--------+--------+
2 rows in set (0.06 sec)
```

## 查看触发器

查看触发器是指查看数据库中已经存在的触发器的定义、状态和语法信息等。MySQL 中查看触发器的方法包括 SHOW TRIGGERS 语句和查询 information_schema 数据库下的 triggers 数据表等。本节将详细介绍这两种查看触发器的方法。

在 MySQL 中，可以通过 SHOW TRIGGERS 语句来查看触发器的基本信息，语法格式如下：

```sql
SHOW TRIGGERS;
```

#### 示例 1

首先创建一个数据表 account，表中有两个字段，分别是 INT 类型的 accnum 和 DECIMAL 类型的 amount。SQL 语句和运行结果如下：

```sql
mysql> CREATE TABLE account(
    -> accnum INT(4),
    -> amount DECIMAL(10,2));
Query OK, 0 rows affected (0.49 sec)
```


创建一个名为 trigupdate 的触发器，每次 account 表更新数据之后都向 myevent 数据表中插入一条数据。创建数据表 myevent 的 SQL 语句和运行结果如下：

```sql
mysql> CREATE TABLE myevent(
    -> id INT(11) DEFAULT NULL,
    -> evtname CHAR(20) DEFAULT NULL);
Query OK, 0 rows affected (0.26 sec)
```


创建 trigupdate 触发器的 SQL 代码如下：

```sql
mysql> CREATE TRIGGER trigupdate AFTER UPDATE ON account
    -> FOR EACH ROW INSERT INTO myevent VALUES(1,'after update');
Query OK, 0 rows affected (0.15 sec)
```


使用 SHOW TRIGGERS 语句查看触发器（在 SHOW TRIGGERS 命令后添加`\G`，这样显示信息会比较有条理），SQL 语句和运行结果如下：

```sql
mysql> SHOW TRIGGERS \G
*************************** 1. row ***************************
             Trigger: trigupdate
               Event: UPDATE
               Table: account
           Statement: INSERT INTO myevent VALUES(1,'after update')
              Timing: AFTER
             Created: 2020-02-24 14:07:15.08
            sql_mode: STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
             Definer: root@localhost
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.09 sec)
```

由运行结果可以看到触发器的基本信息。对以上显示信息的说明如下：

- Trigger 表示触发器的名称，在这里触发器的名称为 trigupdate；
- Event 表示激活触发器的事件，这里的触发事件为更新操作 UPDATE；
- Table 表示激活触发器的操作对象表，这里为 account 表；
- Statement 表示触发器执行的操作，这里是向 myevent 数据表中插入一条数据；
- Timing 表示触发器触发的时间，这里为更新操作之后（AFTER）；
- 还有一些其他信息，比如触发器的创建时间、SQL 的模式、触发器的定义账户和字符集等，这里不再一一介绍。


SHOW TRIGGERS 语句用来查看当前创建的所有触发器的信息。因为该语句无法查询指定的触发器，所以在触发器较少的情况下，使用该语句会很方便。如果要查看特定触发器的信息或者数据库中触发器较多时，可以直接从 information_schema 数据库中的 triggers 数据表中查找。

### 在triggers表中查看触发器信息

在 MySQL 中，所有触发器的信息都存在 information_schema 数据库的 triggers 表中，可以通过查询命令 SELECT 来查看，具体的语法如下：

SELECT * FROM information_schema.triggers WHERE trigger_name= '触发器名';

其中，`'触发器名'`用来指定要查看的触发器的名称，需要用单引号引起来。这种方式可以查询指定的触发器，使用起来更加方便、灵活。

#### 示例 2

下面使用 SELECT 命令查看 trigupdate 触发器，SQL 语句如下：

SELECT * FROM information_schema.triggers WHERE TRIGGER_NAME= 'trigupdate'\G

上述命令通过 WHERE 来指定需要查看的触发器的名称，运行结果如下：

```sql
mysql> SELECT * FROM information_schema.triggers WHERE TRIGGER_NAME= 'trigupdate'\G
*************************** 1. row ***************************
           TRIGGER_CATALOG: def
            TRIGGER_SCHEMA: test
              TRIGGER_NAME: trigupdate
        EVENT_MANIPULATION: UPDATE
      EVENT_OBJECT_CATALOG: def
       EVENT_OBJECT_SCHEMA: test
        EVENT_OBJECT_TABLE: account
              ACTION_ORDER: 1
          ACTION_CONDITION: NULL
          ACTION_STATEMENT: INSERT INTO myevent VALUES(1,'after update')
        ACTION_ORIENTATION: ROW
             ACTION_TIMING: AFTER
ACTION_REFERENCE_OLD_TABLE: NULL
ACTION_REFERENCE_NEW_TABLE: NULL
  ACTION_REFERENCE_OLD_ROW: OLD
  ACTION_REFERENCE_NEW_ROW: NEW
                   CREATED: 2020-02-24 16:07:15.08
                  SQL_MODE: STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
                   DEFINER: root@localhost
      CHARACTER_SET_CLIENT: gbk
      COLLATION_CONNECTION: gbk_chinese_ci
        DATABASE_COLLATION: latin1_swedish_ci
1 row in set (0.22 sec)
```

由运行结果可以看到触发器的详细信息。对以上显示信息的说明如下：

- TRIGGER_SCHEMA 表示触发器所在的数据库；
- TRIGGER_NAME 表示触发器的名称；
- EVENT_OBJECT_TABLE 表示在哪个数据表上触发；
- ACTION_STATEMENT 表示触发器触发的时候执行的具体操作；
- ACTION_ORIENTATION 的值为 ROW，表示在每条记录上都触发；
- ACTION_TIMING 表示触发的时刻是 AFTER；
- 还有一些其他信息，比如触发器的创建时间、SQL 的模式、触发器的定义账户和字符集等，这里不再一一介绍。


上述 SQL 语句也可以不指定触发器名称，这样将查看所有的触发器，SQL 语句如下：

SELECT * FROM information_schema.triggers \G

这个语句会显示 triggers 数据表中所有的触发器信息。

## 修改和删除触发器

修改触发器可以通过删除原触发器，再以相同的名称创建新的触发器。

### 基本语法

与其他 MySQL 数据库对象一样，可以使用 DROP 语句将触发器从数据库中删除。

语法格式如下：

```sql
DROP TRIGGER [ IF EXISTS ] [数据库名] <触发器名>
```

语法说明如下：

##### 1) 触发器名

要删除的触发器名称。

##### 2) 数据库名

可选项。指定触发器所在的数据库的名称。若没有指定，则为当前默认的数据库。

##### 3) 权限

执行 DROP TRIGGER 语句需要 SUPER 权限。

##### 4) IF EXISTS

可选项。避免在没有触发器的情况下删除触发器。

> 注意：删除一个表的同时，也会自动删除该表上的触发器。另外，触发器不能更新或覆盖，为了修改一个触发器，必须先删除它，再重新创建。

### 删除触发器

使用 DROP TRIGGER 语句可以删除 MySQL 中已经定义的触发器。

- 实例

删除 double_salary 触发器，输入的 SQL 语句和执行过程如下所示。

```sql
mysql> DROP TRIGGER double_salary;
Query OK, 0 rows affected (0.03 sec)
```

删除 double_salary 触发器后，再次向数据表 tb_emp6 中插入记录时，数据表 tb_emp7 的数据不再发生变化，如下所示。

```sql
mysql> INSERT INTO tb_emp6
    -> VALUES (3,'C',1,200);
Query OK, 1 row affected (0.09 sec)
mysql> SELECT * FROM tb_emp6;
+----+------+--------+--------+
| id | name | deptId | salary |
+----+------+--------+--------+
|  1 | A    |      1 |   1000 |
|  2 | B    |      1 |    500 |
|  3 | C    |      1 |    200 |
+----+------+--------+--------+
3 rows in set (0.00 sec)
mysql> SELECT * FROM tb_emp7;
+----+------+--------+--------+
| id | name | deptId | salary |
+----+------+--------+--------+
|  1 | A    |      1 |   2000 |
|  2 | B    |      1 |   1000 |
+----+------+--------+--------+
2 rows in set (0.00 sec)
```

## 实例

商品列表和订单列表，订单列表数量的操作会触发商品列表数量的改变

#### 1、创建表 

+ goods 商品表

  ```sql
  create table goods(
  	goods_id int unsigned primary key auto_increment,
      goods_name varchar(20),
      goods_num int unsigned
  )
  ```

+ order 订单表

  ```sql
  create table `order`(
  	order_id int unsigned primary key auto_increment,
      goods_id int unsigned,
      order_num tinyint
  )
  ```

+ 添加商品数据

  insert into goods values(null,'iPhone13', 100),(null,'watch',20),(null,'平板', 30);

#### 2、下订单商品对应减少

+ 分析

  监视地点   order表

  监视事件    insert

  触发时间    insert之后

  触发事件   update操作

+ 实例

  ```sql
  create trigger t1
  after insert on `order`
  for each row
  begin
  update goods set goods_num=goods_num-new.order_num where goods_id=new.goods_id;
  end;
  ```

+ new的含义

  new代表新增的一条数据  行中每一列的值使用new.列名来代替

+ 插入语句

  ```sql
  insert into `order` values(null, 1, 5);
  ```


#### 3、撤销订单

+ 分析

  监视地点   order表

  监视事件    delete

  触发时间    delete之后

  触发事件   update操作

+ 实例

  ```sql
  create trigger t2
  after delete on `order`
  for each row
  begin
  update goods set goods_num=goods_num+old.order_num where goods_id=old.goods_id;
  end;
  ```
  
  删除触发器：drop trigger if exists t1;  
  
+ 删除刚才所下的订单

  ```sql
  delete from `order` where order_id=1;
  ```

#### 4、修改订单

+ 步骤

  + 撤销订单
  + 新增订单

+ 实例

  ```sql
  create trigger t3
  after update on `order`
  for each row
  begin
  --撤销订单
  update goods set goods_num=goods_num+old.order_num where goods_id=old.goods_id;
  --新增订单
  update goods set goods_num=goods_num-new.order_num where goods_id=new.goods_id;
  end;
  ```
  
+ 修改订单数量

  ```sql
  update `order` set order_num=10 where order_id=2;
  ```

+ 商品表执行了两次动作

  + 先将商品的数量和订单的数量累加在一起
  + 减掉订单的数量  则为当前商品的剩余数量