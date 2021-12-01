# Postgresql

中文文档：http://www.postgres.cn/docs/13/

手册下载：https://github.com/postgres-cn/pgdoc-cn/releases

PostgreSQL是以加州大学伯克利分校计算机系开发的对象关系型数据库管理系统（ORDBMS），免费开源。

docker安装PostgreSQL

```bash
docker run -d --name=postgres --restart=always -e POSTGRES_PASSWORD='password' -e ALLOW_IP_RANGE=0.0.0.0/0 -v ~/postgres/data:/var/lib/postgresql -p 5432:5432 postgres:12.8

# 进入容器
docker exec -it postgres bash
```

## 终端操作

### 进入终端

```bash
# 使用其他操作系统用户登入 postgresql数据库交互终端
root@9ad0ef6c0ed5:/# psql -Upostgres
psql (12.8 (Debian 12.8-1.pgdg100+1))
Type "help" for help.

# ＃表示的当前用户是超级管理员，$表示其他普通管理员
postgres=#
```



### 元命令

文档：http://www.postgres.cn/docs/13/app-psql.html#APP-PSQL-META-COMMANDS

| 元命令                     | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| \?                         | 查看所有终端操作符                                           |
| \q                         | 退出终端                                                     |
| \l                         | 列出服务器中的数据库并且显示它们的名称、拥有者、字符集编码以及访问特权。\list也可以 |
| \c  <数据库名>             | 切换进入数据库                                               |
| \dt                        | 列出当前数据库中所有数据表                                   |
| \du                        | 列出系统中所有的用户角色                                     |
| \di                        | 列出当前数据库中所有索引                                     |
| \h                         | 查看命令的帮助信息                                           |
| \conninfo                  | 输出有关当前数据库连接的信息                                 |
| \encoding                  | 输出当前客户端连接的编码信息，\encoding utf8 表示设置编码    |
| \password [ *`username`* ] | 给指定用户设置新的密码口令                                   |



### 数据库管理

```sql
-- 创建数据库
-- CREATE DATABASE <数据库名>;
create database school;
-- 查看数据库列表，也可以直接使用 \l
select datname from pg_database;
--                                  List of databases
--    Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-- -----------+----------+----------+------------+------------+-----------------------
--  postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
--  school    | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
--  template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
--            |          |          |            |            | postgres=CTc/postgres
--  template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
--            |          |          |            |            | postgres=CTc/postgres
-- (4 rows)

-- 进入数据库school
\c school

-- 删除数据库
-- DROP DATABASE <数据库名>;
drop database school;
```



### 数据库模式

PostgreSQL 模式（SCHEMA）可以看着是一个表的集合。一个模式下可以包含多个视图、索引、数据类型、函数和操作符等。

相同的对象名称（表名、视图、索引等）可以被用于不同的模式中而不会出现冲突，例如 模式schema1 和 模式myschema 都可以包含名为 mytable 的表。

使用模式的优势：

-   允许多个用户使用一个数据库并且不会互相干扰。

-   将数据库对象组织成逻辑组以便更容易管理。
-   第三方应用的数据可以放在独立的模式中，这样它们就不会与其他对象的名称发生冲突。

总而言之，就是用户的数据可以保存在同一个数据库中，但是因为各自处于不同的模式下，所以看到的表结构可以一样，但是数据是不一样的。非常适合用于实现权限隔离、一般在开发中往往用于多租户系统。

模式类似于我们开发时使用的python虚拟环境。

#### 创建模式

基本语法：

```sql
-- 可以使用 CREATE SCHEMA 语句来创建模式
-- CREATE SCHEMA schema_name.mytable (
-- ...
-- );

-- 例子：schema_name 就是模式名称，但是实际开发，一般都会基于不同的组织、角色、群组来设置模式名称
create table schema_name.company(
   ID   INT              NOT NULL,
   NAME VARCHAR (20)     NOT NULL,
   AGE  INT              NOT NULL,
   ADDRESS  CHAR (25),
   SALARY   DECIMAL (18, 2),
   PRIMARY KEY (ID)
);
```

#### 删除模式

```sql
-- 删除一个为空的模式
DROP SCHEMA myschema;
-- 删除一个非空的模式
DROP SCHEMA myschema CASCADE;
```



### 数据表管理

```sql
-- 创建数据表
-- CREATE TABLE <数据表名> (
--    字段名 数据类型 类型约束 注释,
--    字段名 数据类型 类型约束 注释,
--    ...
--    unique name(name)
-- );
create table student (
   id serial not null primary key,
   name char(15),
   age smallint,
   sex boolean default true,
   mobile char(20) unique,
   money real
);

-- 列出当前数据库所有数据表，也可以使用 \d
SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';
--                List of relations
--  Schema |      Name      |   Type   |  Owner
-- --------+----------------+----------+----------
--  public | student        | table    | postgres
--  public | student_id_seq | sequence | postgres
-- (2 rows)

-- 查看表结构，也可以使用-- \d <数据表名>
SELECT column_name,data_type,is_nullable,column_default FROM information_schema.columns WHERE table_name ='student';
--                                Table "public.student"
--  Column |     Type      | Collation | Nullable |               Default
-- --------+---------------+-----------+----------+-------------------------------------
--  id     | integer       |           | not null | nextval('student_id_seq'::regclass)
--  name   | character(15) |           |          |
--  age    | smallint      |           |          |
--  sex    | boolean       |           |          | true
--  mobile | character(20) |           |          |
--  money  | real          |           |          |
-- Indexes:
--     "student_pkey" PRIMARY KEY, btree (id)
--     "student_mobile_key" UNIQUE CONSTRAINT, btree (mobile)

-- 删除表
-- DROP TABLE <数据表名>;
drop table student;
```



### 数据管理

#### 添加数据

```sql
-- 添加1条数据
INSERT INTO <数据表名> (字段1,字段2,字段3,...) VALUES (值1, 值2, 值3,...);
-- 添加多条数据
INSERT INTO <数据表名> (字段1,字段2,字段3,...) VALUES (值1, 值2, 值3,...),(值1, 值2, 值3,...),(值1, 值2, 值3,...);
```

```sql
INSERT INTO student (name,age,sex,mobile,money) VALUES ('张小娟', 15, false,'15354929233',1300.00);
INSERT INTO student (name,age,sex,mobile,money) VALUES 
('张小明', 16, true, '15350939483',1100.00),
('张小雨', 15, false,'13312349633',1150.00),
('张娟',   17, true, '15397494555',2350.00),
('张成雨', 17, true, '15512392542',1850.00),
('张潇雨', 14, false,'13529258339',1450.00),
('张成风', 15, true, '15059229494',1540.00),
('江小白', 16, false,'13669338500',1339.00),
('江浩宇', 16, true, '13939529299',2015.00),
('江浩天', 19, true, '18252242002',1996.00),
('江茗雨', 18, false,'18253293345',1560.00),
('江建宇', 17, true, '15522120421',1702.00),
('江小雨', 17, false,'15368285757',1888.00),
('江小雨', 17, false,'10086111111',1888.00);
```

#### 基本查询

```sql
-- 查询所有数据
SELECT * FROM <数据表名>;
-- 查询所有数据的指定字段信息
SELECT <字段1>,<字段2>,.... FROM <数据表名>;
-- 查询符合条件的数据的指定字段信息
SELECT <字段1>,<字段2>,.... FROM <数据表名> WHERE 条件;
```

```sql
-- 查询所有学生的所有信息
select * from student;
-- 查询出id为偶数的学生信息
select id,name,mobile from student where id%2=0;
-- 查询出所有女生的学生信息
select id,name,mobile from student where sex=false;
-- 查询出所有15岁以上女生的学生信息
select id,name,mobile from student where sex=false and age > 15;
```

#### 删除数据

```sql
-- 删除符合条件的数据
DELETE FROM <数据表名> WHERE 条件;
-- 清空数据
TRUNCATE TABLE <数据表名>;
```

```sql
-- 删除mobile = '10086111111'的数据
delete from student where mobile = '10086111111';
-- 清空数据表数据记录，并重置表状态
truncate table student;
```

#### 更新数据

```sql
-- 全表更新数据
UPDATE <数据表名> SET 字段=值,字段=值,...;
-- 更新符合条件的数据
UPDATE <数据表名> SET 字段=值 WHERE 条件;
```

```sql
-- 设置所有学生的money为100
update student set money=1000 where money < 0;
-- 给所有学生在原基础上增加2000块以内的随机金额
update student set money=money+ceil(random()*2000);
-- 给所有学生在原基础上扣除2000块以内的随机金额
update student set money=money-ceil(random()*2000);
select id,name,money from student;
-- 给id=1的学生添加100余额
update student set money=money+100 where id=1;
```

#### 查询进阶

```sql
-- 多表查询
SELECT * FROM <数据表名1>,<数据表名2>,...;
SELECT <字段1>,<字段2>,... FROM <数据表名1>,<数据表名2>,...;

-- 排序显示数据
SELECT <字段1>,<字段2>,... FROM <数据表名> ORDER BY <字段> DESC,<字段> ASC,...;

-- 查询n条数据
SELECT <字段1>,<字段2>,... FROM <数据表名> WHERE 条件 LIMIT <n>;

-- 查询下标s之后的n条数据
SELECT <字段1>,<字段2>,... FROM <数据表名> WHERE 条件 OFFSET s LIMIT <n>;

-- 查询统计
SELECT 聚合函数(字段) FROM <数据表名>;

-- 单字段分组统计
SELECT <分组字段>,聚合函数(<字段>) FROM <数据表名> group by <分组字段>;
-- 多字段分组统计
SELECT <分组字段1>,<分组字段2>,...,聚合函数(<字段>) FROM <数据表名> group by <分组字段1>,<分组字段2>,...;

-- 联合查询
-- 注意，UNION的每个SELECT语句必须是相同数量的列。列必须有相似的数据类型。同时每个SELECT语句中的列的顺序也必须相同。
-- 数据不重复
SELECT <字段1>,<字段2>,... FROM <数据表名> UNION SELECT <字段1>,<字段2>,... FROM <数据表名>;
-- 数据重复
SELECT <字段1>,<字段2>,... FROM <数据表名> UNION ALL SELECT <字段1>,<字段2>,... FROM <数据表名>;

-- 子查询
-- 注意：虽名为子查询，但不管select，还是delete、update、insert都可用子查询，原因是数据库操作中，一切操作都是查询。
-- SELECT <字段1>,<字段2>,.... FROM (SELECT <字段1>,<字段2>,....) <临时表别名>;
```

```sql
-- 查询所有学生的所有信息，并以age做倒序排列
select * from student order by age desc;
-- 查询所有学生的所有信息，并以age第一倒序，money第二升序
select * from student order by age desc,money asc;
-- 查询5个学生信息
select * from student limit 5;
-- 查询钱最多的6个学生
select * from student order by money desc limit 6;
-- 按钱包余额，查出排名在4-6名的3个学生信息
select * from student order by money desc offset 3 limit 3;
-- 查询出手机号以153或者150开头的学生信息，并以手机号升序排序
select id,name,mobile from student where mobile like '153%' or mobile like '150%' order by mobile asc;

-- 统计所有学生人数
select count(id) from student;
-- 统计所有学生人数，并给返回的结果字段设置字段别名
select count(id) total from student;

-- 分组统计男女人数
select sex, count(id) from student group by sex;
-- 字段别名【as可以省略】
select case when sex then '男' else '女' end as sex, count(id) from student group by sex;
-- 统计各个年龄段男女人数
select age,sex, count(id) from student group by age,sex;
-- 统计15岁以上各个年龄段男女人数
select age,sex, count(id) from student  where age > 15 group by age,sex;
-- 统计15岁以上各个年龄段钱包余额最少的男女生，并剔除money>1500的信息
select age,sex, min(money) from student  where age > 15 group by age,sex having min(money) <= 1500;

-- 单表操作基本不需要子查询，这里仅仅为了演示子查询的使用。
-- 假设每个学生一个月平均消费800，查询出15岁以上学生在本月，1个月后，2个月后的余额。
select money,money-800, money-1600 from (select id,age, money from student where age > 15) table1;
```



### 备份还原

```bash
# 二进制备份
# pg_dump -h IP地址 -U账户 -d数据库 -Fc > <文件名.dmp>
pg_dump -Upostgres -dschool -Fc > school.dmp

# 导出sql表结构, 不包含数据
# pg_dump --verbose --schema-only --db=数据库名 --file=文件名.sql
pg_dump --verbose --schema-only --db=school --file=./school.sql

# 导出sql表结构与表数据
# pg_dump -U账户名 --column-inserts 数据库名 > 文件名.sql
pg_dump -U postgres --column-inserts school > ./school.sql

# 二进制还原
# pg_restore -h IP地址 -U账户名 -d数据库 -Fc <文件名>
pg_restore -Upostgres -dtemplate1 -Fc school.dmp

# SQL还原
# psql -d数据库名 -U账户名 -f 文件名.sql
psql -dschool -Upostgres -f ./school.sql
```



### 权限管理

文档：https://www.shulanxt.com/database/postgresql/postgresql-privileges

#### 创建用户

```bash
CREATE USER <用户名> WITH PASSWORD '<登陆密码>';
```

```bash
CREATE USER shulanxt WITH PASSWORD 'password';
```

#### 分配权限

```python
grant 

voker
```



## Python操作

windows下不能直接通过conda或者pip 的install来安装postgresql模块。

windows系统下载扩展包：https://pypi.org/project/psycopg2/#files

```bash
pip install psycopg2
```

```python
# pip install psycopg
import psycopg2,json
from psycopg2.extras import RealDictCursor

if __name__ == '__main__':
    # 连接数据库
    conn = psycopg2.connect(database="school", user="postgres", password="password", host="127.0.0.1", port="5432")
    print(conn)
    # 获取游标
    cursor = conn.cursor(cursor_factory=RealDictCursor)
    # print(cursor)

    # # """读取数据"""
    # # 读取多条数据
    # cursor.execute('select * from student limit 5')
    # rows = cursor.fetchall()
    # for row in rows:
    #     row = dict(row)
    #     print(row["name"], row)
    #
    #     data_str = json.dumps(row,ensure_ascii=False)
    #     print(data_str)

    # # 读取1条数据
    # cursor.execute("""select * from student order by age desc""")
    # row = cursor.fetchone()
    # row = dict(row)
    # print(row["name"], row)

    """添加数据"""
    # try:
    #     # 添加1条数据
    #     # cursor.execute("""INSERT INTO student (name,age,sex,mobile,money) VALUES ('小明', 15, false,'17835492923',1300.00)""")
    #     # conn.commit()
    #     # conn.close()
    #
    #     data = [
    #         {"name": "小白", "age": 15, "sex": False, "mobile": "15912341234", "money": 880},
    #         {"name": "小黑", "age": 16, "sex": True,  "mobile": "15992929332", "money": 1220},
    #     ]
    #     cursor.executemany("""INSERT INTO student (name,age,sex,mobile,money) VALUES (%(name)s, %(age)s, %(sex)s, %(mobile)s,%(money)s)""", data)
    #     conn.commit()
    #     conn.close()
    #
    # except Exception as e:
    #     print(f"添加数据失败：{e}")


    """更新/删除数据"""
    # cursor.execute("""update student set money=money+1000 where name = '小明'""")
    # conn.commit()
```

## Django连接postgresql

模块安装：

```bash
pip install psycopg2-binary
```

settings.dev，代码：

```python
DATABASES = {
 'default': {
     'ENGINE': 'django.db.backends.postgresql_psycopg2',
     'NAME': 'cps',
     'USER': 'cps_user',
     'PASSWORD': 'password',
     'HOST': '127.0.0.1',
     'PORT': '5432',
 }
}
```

