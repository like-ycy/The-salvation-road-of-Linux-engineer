# MySQL权限管理

MySQL 是一个多用户数据库，具有功能强大的访问控制系统，可以为不同用户指定不同权限。在前面的章节中我们使用的是 root 用户，该用户是超级管理员，拥有所有权限，包括创建用户、删除用户和修改用户密码等管理权限。

## 一、MySQL权限简介

关于mysql的权限简单的理解就是mysql允许你做你全力以内的事情，不可以越界。比如只允许你执行select操作，那么你就不能执行update操作。只允许你从某台机器上连接mysql，那么你就不能从除那台机器以外的其他机器连接mysql。

那么Mysql的权限是如何实现的呢？这就要说到mysql的两阶段验证，下面详细介绍：第一阶段：服务器首先会检查你是否允许连接。因为创建用户的时候会加上主机限制，可以限制成本地、某个IP、某个IP段、以及任何地方等，只允许你从配置的指定地方登陆。第二阶段：如果你能连接，Mysql会检查你发出的每个请求，看你是否有足够的权限实施它。比如你要更新某个表、或者查询某个表，Mysql会查看你对哪个表或者某个列是否有权限。再比如，你要运行某个存储过程，Mysql会检查你对存储过程是否有执行权限等。

MYSQL到底都有哪些权限呢？从官网复制一个表来看看：

| **权限**                | **权限级别**           | **权限说明**                                                 |
| ----------------------- | ---------------------- | ------------------------------------------------------------ |
| CREATE                  | 数据库、表或索引       | 创建数据库、表或索引权限                                     |
| DROP                    | 数据库或表             | 删除数据库或表权限                                           |
| GRANT OPTION            | 数据库、表或保存的程序 | 赋予权限选项                                                 |
| REFERENCES              | 数据库或表             |                                                              |
| ALTER                   | 表                     | 更改表，比如添加字段、索引等                                 |
| DELETE                  | 表                     | 删除数据权限                                                 |
| INDEX                   | 表                     | 索引权限                                                     |
| INSERT                  | 表                     | 插入权限                                                     |
| SELECT                  | 表                     | 查询权限                                                     |
| UPDATE                  | 表                     | 更新权限                                                     |
| CREATE VIEW             | 视图                   | 创建视图权限                                                 |
| SHOW VIEW               | 视图                   | 查看视图权限                                                 |
| ALTER ROUTINE           | 存储过程               | 更改存储过程权限                                             |
| CREATE ROUTINE          | 存储过程               | 创建存储过程权限                                             |
| EXECUTE                 | 存储过程               | 执行存储过程权限                                             |
| FILE                    | 服务器主机上的文件访问 | 文件访问权限                                                 |
| CREATE TEMPORARY TABLES | 服务器管理             | 创建临时表权限                                               |
| LOCK TABLES             | 服务器管理             | 锁表权限                                                     |
| CREATE USER             | 服务器管理             | 创建用户权限                                                 |
| PROCESS                 | 服务器管理             | 查看进程权限                                                 |
| RELOAD                  | 服务器管理             | 执行flush-hosts, flush-logs, flush-privileges, flush-status, flush-tables, flush-threads, refresh, reload等命令的权限 |
| REPLICATION CLIENT      | 服务器管理             | 复制权限                                                     |
| REPLICATION SLAVE       | 服务器管理             | 复制权限                                                     |
| SHOW DATABASES          | 服务器管理             | 查看数据库权限                                               |
| SHUTDOWN                | 服务器管理             | 关闭数据库权限                                               |
| SUPER                   | 服务器管理             | 执行kill线程权限                                             |

 MYSQL的权限如何分布，就是针对表可以设置什么权限，针对列可以设置什么权限等等，这个可以从官方文档中的一个表来说明：

| 权限分布 | 可能的设置的权限                                             |
| -------- | ------------------------------------------------------------ |
| 表权限   | 'Select', 'Insert', 'Update', 'Delete', 'Create', 'Drop', 'Grant', 'References', 'Index', 'Alter' |
| 列权限   | 'Select', 'Insert', 'Update', 'References'                   |
| 过程权限 | 'Execute', 'Alter Routine', 'Grant'                          |

##  二、MySQL权限经验原则

权限控制主要是出于安全因素，因此需要遵循一下几个经验原则：

1、只授予能满足需要的最小权限，防止用户干坏事。比如用户只是需要查询，那就只给select权限就可以了，不要给用户赋予update、insert或者delete权限。

2、创建用户的时候限制用户的登录主机，一般是限制成指定IP或者内网IP段。

3、初始化数据库的时候删除没有密码的用户。安装完数据库的时候会自动创建一些用户，这些用户默认没有密码。

4、为每个用户设置满足密码复杂度的密码。

5、定期清理不需要的用户。回收权限或者删除用户。

## 三、MySQL权限实战：

### 1、GRANT命令使用说明：

先来看一个例子，创建一个只允许从任意地址登录的超级用户jack，并允许将权限赋予别的用户，密码为：123.com

```sql
mysql> grant all privileges on *.* to jack@'%' identified by "jack" with grant option;
```

GRANT命令说明：
**ALL PRIVILEGES** 是表示所有权限，你也可以使用select、update等权限。

**ON** 用来指定权限针对哪些库和表。

 `.`中前面的\*号用来指定数据库名，后面的*号用来指定表名。

**TO** 表示将权限赋予某个用户。

**jack@'localhost'** 表示jack用户，@后面接限制的主机，可以是IP、IP段、域名以及%，%表示任何地方。

注意：这里%有的版本不包括本地，以前碰到过给某个用户设置了%允许任何地方登录，但是在本地登录不了，这个和版本有关系，遇到这个问题再加一个localhost的用户就可以了。

**IDENTIFIED BY** 指定用户的登录密码。

W**ITH GRANT OPTION** 这个选项表示该用户可以将自己拥有的权限授权给别人。注意：经常有人在创建操作用户的时候不指定WITH GRANT OPTION选项导致后来该用户不能使用GRANT命令创建用户或者给其它用户授权。

备注：可以使用GRANT重复给用户添加权限，权限叠加，比如你先给用户添加一个select权限，然后又给用户添加一个insert权限，那么该用户就同时拥有了select和insert权限。

### 2、刷新权限

使用这个命令使权限生效，尤其是你对那些权限表user、db、host等做了update或者delete更新的时候。以前遇到过使用grant后权限没有更新的情况，只要对权限做了更改就使用FLUSH PRIVILEGES命令来刷新权限。

```sql
mysql> flush privileges;
```

### 3、查看权限

1) 、查看当前用户的权限：

```sql
mysql> show grants;
+---------------------------------------------------------------------+
| Grants for root@localhost                                           |
+---------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION        |
+---------------------------------------------------------------------+
2 rows in set (0.00 sec)
```

2) 、查看某个用户的权限：

```sql
mysql> show grants for 'jack'@'%';
+-----------------------------------------------------------------------------------------------------+
| Grants for jack@%                                                                                   |
+-----------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'jack'@'%' IDENTIFIED BY PASSWORD '*9BCDC990E611B8D852EFAF1E3919AB6AC8C8A9F0' |
+-----------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### 4、回收权限

```sql
mysql> revoke delete on *.* from 'jack'@'%';
Query OK, 0 rows affected (0.01 sec)
```

###   5、删除用户


```sql
mysql> select host,user,password from user;
+-----------+------+-------------------------------------------+
| host      | user | password                                  |
+-----------+------+-------------------------------------------+
| localhost | root |                                           |
| rhel5.4   | root |                                           |
| 127.0.0.1 | root |                                           |
| ::1       | root |                                           |
| localhost |      |                                           |
| rhel5.4   |      |                                           |
| localhost | jack | *9BCDC990E611B8D852EFAF1E3919AB6AC8C8A9F0 |
+-----------+------+-------------------------------------------+
7 rows in set (0.00 sec)

mysql> drop user 'jack'@'localhost';
Query OK, 0 rows affected (0.01 sec)
```

###   6、对账户重命名

```sql
mysql> rename user 'jack'@'%' to 'jim'@'%';
Query OK, 0 rows affected (0.00 sec)
```

###   7、修改密码

```sql
# 1、用set password命令
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123456');
Query OK, 0 rows affected (0.00 sec)

# 2、用mysqladmin
[root@rhel5 ~]# mysqladmin -uroot -p123456 password 1234abcd
备注：
格式：mysqladmin -u用户名 -p旧密码 password 新密码

# 3、用update直接编辑user表
mysql> use mysql

mysql> update user set PASSWORD = PASSWORD('1234abcd') where user = 'root';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

## 四、其他sql语句

```
1）查看当前登录的用户
mysql>: select user();

2）root权限下可以查看所有用户信息
mysql>: select * from mysql.user;
mysql>: select * from mysql.user \G
mysql>: select user,password,host from mysql.user;

3）root登录下，删除游客（操作后要重启mysql服务）
mysql>: delete from mysql.user where user='';

4）root登录下，修改密码（操作后要重启mysql服务）
mysql>: update mysql.user set password=password('12345678') where host='localhost';

5）没有登录，修改密码（操作后要重启mysql服务）
>: mysqladmin -u用户名 -p旧密码 -h域名 password "新密码"
eg>: mysqladmin -uroot -p12345678 -hlocalhost password "root"
eg>: mysqladmin -uroot -p12345678 -hlocalhost password root
注：上面格式必须是这样不能空,password后面不能是单引号，只能是双引号，或者不加引号
```

