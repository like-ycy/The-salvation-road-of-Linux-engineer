# MySQL多表查询

首先两张表

+ user表

  ```sql
  CREATE TABLE user (
  `id`  int(10) UNSIGNED NOT NULL AUTO_INCREMENT ,
  `username`  varchar(10) DEFAULT 'xxx' ,
  `sex`  enum('w','m') DEFAULT 'w' ,
  `age`  tinyint(4) NULL DEFAULT 18 ,
  `grade`  varchar(10) DEFAULT 'python36' ,
  PRIMARY KEY (`id`)
  );
  ```

+ address表

  ```sql
  CREATE TABLE address (
  `id`  int(10) UNSIGNED NOT NULL AUTO_INCREMENT ,
  `uid`  int(10) UNSIGNED NOT NULL ,
  `address`  varchar(255) DEFAULT NULL ,
  PRIMARY KEY (`id`)
  );
  ```

## 隐式内连接

+ 主体结构

  ```sql
  select * from 表名1,表名2... where 表名1.字段=表名2.字段  and ...
  ```

+ 实例

  ```sql
  # 不起别名查询所有
  SELECT * FROM USER,address WHERE user.id = address.uid AND user.id = 1;
  
  # 不起别名查询某个字段
  SELECT user.id, user.username, address.address FROM USER, address WHERE user.id = address.uid AND user.id = 1;
  
  # 起别名查询某个字段
  SELECT u.id,u.username,a.address FROM USER u,address a WHERE u.id = a.uid AND u.id = 1;
  ```


## 显示内连接  inner join

+ 主体结构

  ```sql
  select * from 表1 inner join 表2 on 条件；
  ```

- 实例

  ```sql
  ### 查询id为1的用户的   用户信息以及地址信息
  
  # 不起别名
  SELECT * FROM USER INNER JOIN address ON `user`.id = address.uid AND `user`.id = 1;
  
  # 起别名
  select * from user u INNER JOIN address a on u.id=a.uid and u.id=1;
  ```

## 左连接  left join

+ 主体

  ```sql
  select * from 表1 left join 表2 on  条件...
  ```

+ 实例

  ```sql
  ### 查询id为1的用户的   用户信息以及地址信息
  
  # 不起别名
  select * from user LEFT JOIN address on `user`.id=address.uid and `user`.id=1;
  
  # 起别名
  select * from user u LEFT JOIN address a on u.id=a.uid and u.id=1;
  ```

  

## 右连接  right join

+ 主体

  ```sql
  select * from 表1 rightjoin 表2 on  条件...
  ```

+ 实例

  ```sql
  ### 查询id为1的用户的   用户信息以及地址信息
  
  # 不起别名
  select * from user RIGHT JOIN address on `user`.id=address.uid and `user`.id=1;
  
  # 起别名
  select * from user u RIGHT JOIN address a on u.id=a.uid and u.id=1;
  ```

## 注意

+ 显示内连接和隐式内连接是相同的，只把关联的数据查询出来
+ 左连接：会以左表为主表，右表为辅表，把主表的数据全部查询出来，辅表中如果没有关联的数据，则使用null来占位
+ 右连接：会以右表为主表，左表为辅表，把主表的数据全部查询出来，辅表中如果没有关联的数据，则使用null来占位
+ 建议使用隐式内连接

## 子查询

**子查询：将一条查询sql的结果作为另一条sql的条件**

- 增：insert into 表 **select子查询结果**
- 删：delete from 表 where条件 **select子查询(查询的表不能与delete表相同)**
- 查：select 字段 from 表 where条件 **select子查询**
- 改：update 表 set 字段=值 where **select子查询(查询的表不能与update表相同)**

```sql
# 子查询的sql
select dep, max(salary) from emp group by dep;

# 将子查询转换为一张表
create table t1(dep_name varchar(64), max_salary decimal(5,2));

## 子查询 - 增
insert into t1 select dep, max(salary) from emp group by dep;

## 子查询 - 删(查询的表不能与delete表相同)
delete from t1 where dep_name in (select distinct dep from emp);

## 子查询 - 改(查询的表不能与delete表相同)
update t1 set max_salary=max_salary+1 where dep_name in (select distinct dep from emp);

## 子查询 - 查
select * from emp where (dep, salary) in (select dep, max(salary) from emp group by dep);
```