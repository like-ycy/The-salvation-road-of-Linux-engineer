# Python操作MySQL

### 1、前期准备

+ 安装

  pip install pymysql

+ 使用

  import pymysql

+ 前提

  开启mysql服务

### 2、使用

(1) 连接MySQL数据库

db = pymysql.connect('主机名','用户名','密码','数据库名')

(2) 设置字符编码

db.set_charset('utf8')

(3) 创建游标对象

cursor = db.cursor()

(4) 执行SQL语句

cursor.execute(sql语句)

(5) 获取结果集

获取所有

cursor.fetchall()

获取一条

cursor.fetchone()

(6) 获取受影响的行数

cursor.rowcount

(7) 事物

因为pymysql开启了事物 所以需要提交或回滚

commit

rollback

(8) 关闭数据库连接

db.close()

实例

```sql
import pymysql
# (1) 连接MySQL数据库
db = pymysql.connect('127.0.0.1', 'root', '123456', 'xxxx')
# (2) 设置字符编码
db.set_charset('utf8')
# (3) 创建游标对象
cursor = db.cursor()
# (4) 执行SQL语句
try:
    sql = 'insert into new_user values("张三", "w", 20)'
    cursor.execute(sql)
    db.commit()
except:
    db.rollback()
# (5) 获取结果集
# 获取所有
# data = cursor.fetchall()
# print(data)
# 获取一条
# data = cursor.fetchone()
# print(data)
# (6) 获取受影响的行数
print(cursor.rowcount)
# 关闭数据库连接
db.close()
```



