# MySQL基础操作SQL语句

### 对于库和表操作的命令

+ 创建

  create

+ 删除

  drop

+ 查看

  show

+ 修改

  alter

## 注意事项

1. windwos下不区分大小写
2. SQL命令不区分大小写
3. 进入数据库以后是没有退出的，只有数据库的切换（use database）
4. MySQL命令以英文分号作为结尾
5. 如果使用数据库默认编码创建了库和表，那么插入中文则乱码，则需要更改字段编码、表编码、库编码
6. MySQL数据库、表、字段 具有唯一性。但是不同的库中表、字段，可以重名
7. 当命令输入完并且以分号作为了结尾，发现命令不能执行，那要查看一下，当前是否有未闭合的引号

## 小命令

+ 命令撤销  \c

  在一条sql未执行时，不想执行了，末尾添加\c取消sql

+ 以竖状形式查看 \G

  show create database xxxx\G

+ 数据库的退出

  + \q
  + exit
  + quit

## 库的基本操作

```sql
1）查看所有数据库
mysql>: show databases;

2）选择某个数据库
mysql>: use 数据库名

3）查看当前所在数据库
mysql>: select database();

4）创建数据库
mysql>: create database 数据库名 [charset=编码格式];
eg>: create database owen;
eg>: create database zero charset=utf8;
eg>: create database test character set utf8;
    
5）查看创建数据库的详细内容
mysql>: show create database 数据库名;
eg>: show create database owen;

6) 修改数据库编码集
mysql>: alter database 数据库名 charset="字符集";
eg>: alter database db1 charset="utf8";
eg>: alter database db1 character set utf8;

7) 删除库
mysql>: drop database 数据库名；
eg>: drop database db1;
```



## 表的基本操作

```sql
1）查看某个库下的所有表，先进入数据库
mysql>: show tables;

2) 查看表的详细信息
mysql>: show create table 表名;

3) 查看表结构
mysql>: desc 表名;

4）增，创建表
mysql>: create table 表名(字段 字段属性, 字段2 字段属性2);
eg>: create table student(name varchar(16), age int);

5)修改表名
mysql>: alter table 表名 rename 新表名;
eg>: alter table t1 rename t2;

6) 删除表
mysql>: drop table 表名;
eg>: drop table test;

7) 修改表字符编码
mysql>: alter table 表名 charset="utf8";
或者
mysql>: alter table 表名 character set utf8;
```

- 索引的操作

  ```sql
  1、添加索引
  1) 添加索引名
  mysql>: alter table 表名 add 索引类型 索引名称(字段名) 
  eg1>: alter table test9 add key k_username(username);
  eg2>: alter table test9 add unique u_age(age);
  2) 不添加索引名
  mysql>: alter table 表名 add 索引类型(字段名) 
  eg1>: alter table test9 add key(username);
  eg2>: alter table test9 add unique(age);
  
  2、删除索引
  mysql>: alter table drop key 索引名称;
  eg>: alter table test9 drop key u_age;
  
  3、修改表和字段字符编码(了解)
  mysql>: alter table 表名 character set 字符编码;
  eg>: alter table 表名 modify 字段名 字段类型[ 约束条件] [ 说明 ] character set utf8;
  ```

## 字段的基本操作

```sql
1）查,查看某个数据库中的某个表的所有记录，如果在对应数据库中，可以直接查找表
mysql>: select * from [数据库名.]表名;
注：*代表查询所有字段

模糊查询：like，_代表任意一个字符，%0个或多个任意字符
mysql>: select * from [数据库名.]表名 where 字段名 like "模糊查询的内容";
eg:  select * from t2 where name like "xc%";


2）增，给表的所有字段插入数据
mysql>: insert [into] [数据库名.]表名 values (值1,...,值n);
eg：如果给有name和age两个字段的student表插入数据
1条>：insert into student values ('Bob', 18);
多条>：insert into student values ('张三', 18), ('李四', 20);
指定库>：insert owen.student values ('张三', 18), ('李四', 20);

3）改，根据条件修改指定内容
mysql>: update [数据库名.]表名 set 字段1=新值1, 字段n=新值n where 字段=旧值;
eg:> update student set name='王五', age='100' where name='张三';
注：i) 可以只修改部分字段 ii) 没有条件下，所有记录都会被更新
eg:> update student set name='呵呵';

4）删，根据条件删除记录
mysql>: delete from [数据库名.]表名 where 条件;
eg:> delete from student where age<30;

5)修改字段属性
mysql>: alter table 表名 modify 字段名 字段属性;
eg>: alter table t1 modify name char(20);

6)修改字段名
mysql>: alter table 表名 change 旧字段名 新字段名 新字段属性;
eg>: alter table t1 change name usr char(16);

7) 表添加新字段
mysql>: alter table 表名 add  字段名 字段类型[ 约束条件] [ 说明 ]
eg>: alter table test9 add username varchar(10) not null default 'xxxx' comment '用户名';

8) 表末尾添加字段
mysql>: alter table 表名 add 字段名 类型[(长度) 约束];  # 末尾
eg>: alter table tf1 add z int unsigned;

9) 表首位添加字段
mysql>: alter table 表名 add 字段名 类型[(宽度) 约束] first;  # 首位
eg>: alter table tf1 add a int unsigned first;

10) 指定字段后添加字段
mysql:> alter table 表名 modify 字段名 字段类型[ 约束条件] [ 说明 ] after 字段名
eg>: alter table test9 modify username varchar(10) default 'xxx' after age;

11) 删除字段
mysql:> alter table 表名 drop 字段名;
eg>: alter table test9 drop a;
```
