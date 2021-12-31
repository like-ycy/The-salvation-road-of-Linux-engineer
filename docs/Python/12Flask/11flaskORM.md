# 数据库操作

数据库驱动（drivers）模块：pymysql、MySQLDB

## ORM

**ORM** 全拼`Object-Relation Mapping`，中文意为 **对象-关系映射**。主要实现模型对象到关系数据库数据的映射。

ORM提供了一种面向对象操作数据库的方式给开发者。不需要编写原生SQL语句也能操作数据库，实现了业务代码与底层数据的解耦。

优点：

- 只需要面向对象编程, 不需要面向数据库编写SQL.
  - 对数据库的操作都转化成对类/对象的属性和方法的操作.    表字段--->对象属性,   SQL关键字-> 操作方法
  - 不用编写各种数据库的`原生sql语句`，当然，也可以编写原生SQL语句。
- 实现了数据模型代码与数据库数据的解耦, 屏蔽了不同数据库操作上的差异
  - 不再需要关注当前项目使用的是哪种数据库。
  - 通过简单的配置就可以轻松更换数据库, 而不需要修改业务代码.

缺点：

- 相比较直接使用SQL语句操作数据库,ORM需要把操作转换成SQL语句，所以有性能损失.
- 根据对象的操作转换成SQL语句,根据查询的结果转化成模型实例对象, 在映射过程中有性能损失.
- 不同的ORM提供的操作不一样，增加了学习成本



## Flask-SQLAlchemy

flask默认提供模型操作，但是并没有提供ORM，所以一般开发的时候我们会采用flask-SQLAlchemy模块来实现ORM操作。

SQLAlchemy是一个python语言编写的高性能的关系型数据库ORM框架，它提供了高层的 ORM 和底层的原生数据库的操作。

我们使用sqlalchemy 不需要调用sqlalchemy 本身这个模块，而是采用flask-sqlalchemy ，这是一个简化了 SQLAlchemy 操作的flask扩展模块。（主要是简化了sqlalchemy初始化代码和分页操作等）

SQLAlchemy：https://docs.sqlalchemy.org/en/14/

flask-SQLAlchemy：https://flask-sqlalchemy.palletsprojects.com/en/2.x/quickstart/

安装 flask-sqlalchemy【清华源】

```bash
pip install flask-sqlalchemy -i https://pypi.tuna.tsinghua.edu.cn/simple
```

如果sqlalchemy连接的是 mysql /MariaDB数据库，需要安装 mysqldb 或pymysql**驱动**

```bash
conda install flask-mysqldb -c conda-forge
```

安装flask-mysqldb时，注意

```
使用pip install 安装 flask-mysqldb的时候，python底层依赖于一个底层的模块 mysqlclient 模块
如果没有这个模块，则会报错如下：

Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-install-21hysnd4/mysqlclient/
```

解决方案：

```bash
sudo apt-get install -y libmysqlclient-dev python3-dev

# 运行上面的安装命令如果再次报错如下：
#   dpkg 被中断，您必须手工运行 ‘sudo dpkg --configure -a’ 解决此问题。

# 则根据提示执行命令以下命令，再次安装mysqlclient
#    sudo dpkg --configure -a
#    apt-get install libmysqlclient-dev python3-dev

解决了mysqlclient问题以后，重新安装 flask-mysqldb即可。
pip install flask-mysqldb -i https://pypi.tuna.tsinghua.edu.cn/simple
```

原生SQLAlchemy连接配置操作

db.py，代码：

```python
from sqlalchemy import create_engine   # 驱动引擎
from sqlalchemy.ext.declarative import declarative_base # 数据库基类
from sqlalchemy import Column, Integer, String, Boolean, Numeric, Text # 字段、整型
from sqlalchemy.orm import sessionmaker  # 连接会话

engine = create_engine(
    # 连接数据库的URL
    # url="mysql+pymysql://root:123@127.0.0.1:3306/students?charset=utf8mb4",  # 如果底层驱动是pymysql
    url="mysql://root:123@127.0.0.1:3306/students?charset=utf8mb4",    # 如果底层驱动是MysqlDB
    echo=True, # 当设置为True时会将orm语句转化为sql语句打印，一般debug的时候可用
    pool_size=8, # 连接池的大小，默认为5个，设置为0时表示连接无限制
    pool_recycle=60*30 # 设置时间以限制数据库多久没连接自动断开
)

DbSession = sessionmaker(bind=engine)
session = DbSession()

# 创建数据基类
Model = declarative_base()
```

manage.py，代码：

```python
import db
class Student(db.Model):
    __tablename__ = "tb_student"
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(255))
    sex  = db.Column(db.Boolean)
    age  = db.Column(db.Integer)
    class_name = db.Column("class", db.String(255),)
    description = db.Column(db.Text)
    is_delete = db.Column(db.Boolean, nullable=True, default=False)

if __name__ == '__main__':
    # 如果没有提前声明模型中的数据表，则可以采用以下代码生成数据表，
    # 如果数据库中已经声明了有数据表，则不会继续生成
    db.Model.metadata.create_all(db.engine)

    # 根据ID查询一条数据
    # ret = db.session.query(Student).get(19)
    # print(ret)
    # print(ret.name)
    # print(ret.class_name)
    #
    # # 查询所有数据
    # data_list = db.session.query(Student).all()
    # print(data_list)
    # for data in data_list:
    #     print(data.name)

    # # 按条件查询
    # data_list = db.session.query(Student).filter(Student.sex==True, Student.class_name=='301').all()
    # print(data_list)
    # for data in data_list:
    #     print(data.sex, data.name)

    # 添加一条数据
    # student = Student(
    #     name="xiaohua",
    #     class_name="305",
    #     sex=False,
    #     age=18,
    #     description="美美的..",
    # )
    # db.session.add(student)
    # db.session.commit()
    #

    # # 添加多条数据
    # student_list = [
    #     Student(name="xiaohei", class_name="305", sex=False, age=18, description="美美的..",),
    #     Student(name="xiaobai", class_name="304", sex=True, age=18, description="美美的..",),
    #     Student(name="xiaohui", class_name="303", sex=False, age=18, description="美美的..",),
    # ]
    #
    # db.session.add_all(student_list)
    # db.session.commit()

    # 更新一条数据
    # student = db.session.query(Student).get(35)
    # student.name = "小会"
    # db.session.commit()

    # 更新多条数据
    # db.session.query(Student).filter(Student.class_name=="303").update({Student.age:Student.age+1})
    # db.session.commit()


    # 删除一条数据
    # student = db.session.query(Student).get(35)
    # db.session.delete(student)
    # db.session.commit()

    # 删除多条数据
    db.session.query(Student).filter(Student.class_name == "401").delete()

    # 原生SQL语句
    # 读
    cursor = db.session.execute('select * from tb_student')
    # 一条
    # data = cursor.fetchone()
    # print(data)

    # 多条
    # data_list = cursor.fetchall()
    # print(data_list)

    # 写[添加、删除、修改]
    cursor = db.session.execute(
        'insert into tb_student(name, class, age, sex, description) values(:name, :class, :age, :sex, :description)',
        params={
            "name": "xiaohong",
            "class": "307",
            "age": 19,
            "sex": 0,
            "description": ".....",
        })
    db.session.commit()
    print(cursor.lastrowid) # 获取最后添加的主键ID



```



### 数据库连接设置

- 在 Flask-SQLAlchemy 中，数据库的链接配置信息使用URL指定，而且程序使用的数据库必须保存到Flask的 **SQLALCHEMY_DATABASE_URI** 配置项中

manage.py，代码：

```python
# SQLAlchemy的链接配置："数据库名://账户名:密码@服务器地址:端口/数据库名称?配置参数选项"
app.config["SQLALCHEMY_DATABASE_URI"] = "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"


# 如果不使用mysqldb改用pymysql，则需要在连接时指定pymysql
# app.config["SQLALCHEMY_DATABASE_URI"] = "mysql+pymysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
```

其他设置项：

```python
# 动态追踪修改设置，如未设置只会提示警告
SQLALCHEMY_TRACK_MODIFICATIONS = True
#查询时会显示原始SQL语句
SQLALCHEMY_ECHO = True
```

- 配置完成需要去 MySQL 中创建项目所使用的数据库

```bash
mysql -uroot -p123
create database flaskdemo charset=utf8mb4;
```



### 常用的SQLAlchemy字段类型

| 模型字段类型名   | python中数据类型  | 说明                                                         |
| :--------------- | :---------------- | :----------------------------------------------------------- |
| **Integer**      | int               | 普通整数，一般是32位                                         |
| **SmallInteger** | int               | 取值范围小的整数，一般是16位                                 |
| BigInteger       | int               | 不限制精度的整数                                             |
| Float            | float             | 浮点数                                                       |
| **Numeric**      | decimal.Decimal   | 普通数值，一般是32位                                         |
| **String**       | str               | 变长字符串                                                   |
| **Text**         | str               | 变长字符串，对较长或不限长度的字符串做了优化                 |
| Unicode          | unicode           | 变长Unicode字符串                                            |
| UnicodeText      | unicode           | 变长Unicode字符串，对较长或不限长度的字符串做了优化          |
| **Boolean**      | bool              | 布尔值                                                       |
| **DateTime**     | datetime.datetime | 日期和时间                                                   |
| Date             | datetime.date     | 日期                                                         |
| Time             | datetime.time     | 时间                                                         |
| LargeBinary      | bytes             | 二进制文件内容                                               |
| **Enum**         | enum.Enum         | 枚举类型，相当于django的choices，但是功能没有choices那么强大 |



### 常用的SQLAlchemy列约束选项

| 选项名      | 说明                                                        |
| :---------- | :---------------------------------------------------------- |
| primary_key | 如果为True，代表当前数据表的主键                            |
| unique      | 如果为True，为这列创建唯一 索引，代表这列不允许出现重复的值 |
| index       | 如果为True，为这列创建普通索引，提高查询效率                |
| nullable    | 如果为True，允许有空值，如果为False，不允许有空值           |
| default     | 为这列定义默认值                                            |



## 数据库基本操作

- 在SQLAlchemy中，添加、修改、删除操作，均由数据库会话(sessionSM)管理。
  - 会话用 db.session 表示。在准备把数据写入数据库前，要先将数据添加到会话中然后调用 db.commit() 方法提交会话。
- 在SQLAlchemy 中，查询操作是通过 query 对象操作数据。
  - 最基本的查询是返回表中所有数据，也可以通过filter过滤器进行更精确的数据库查询。

## 模型类定义

我们后面会把模型创建到单独的文件中，但是现在我们先把模型类写在main.py文件中。

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
class Config(object):
    DEBUG = True
    # 数据库链接配置 = 数据库名称://登录账号:登录密码@数据库主机IP:数据库访问端口/数据库名称?charset=编码类型
    SQLALCHEMY_DATABASE_URI = "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
    # 动态追踪修改设置，如未设置只会提示警告
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    # ORM运行时会显示ORM生成的原始SQL语句[调试]
    SQLALCHEMY_ECHO = True

app.config.from_object(Config)


"""模型类定义"""
db = SQLAlchemy(app=app)
# 等同于
# db = SQLAlchemy()
# db.init_app(app) # 加载配置并完成初始化过程

class Student(db.Model):
    """学生信息模型"""
    # 声明与当前模型绑定的数据表名称
    __tablename__ = "tb_student"
    """
    # 企业中，往往大部分的公司会出现以下2种不同的数据库开发情况。
    # 1. 企业中有DBA，DBA会提前创建数据库和项目中具体业务的数据表。
         也就是说我们不需要自己手动建库建表，只需要根据数据库表结构，使用python声明对应的模型与之匹配，就可以操作数据库了。
    # 2. 企业没有DBA，比较坑爹：
    #    2.1 开发人员，自己手撸SQL语句，手动建库建表。
    #    2.2 开发人员，编写模型，使用数据迁移，手动建库和数据迁移建表。

    # 原生SQL语句
    create table db_student(
      id int primary key auto_increment comment "主键",
      name varchar(15) comment "姓名",
      age smallint comment "年龄",
      sex tinyint default 1 comment "性别",
      email varchar(128) comment "邮箱地址",
      money NUMERIC(10,2) default 0.0 comment "钱包",
      key (name),
      unique key (email)
    );
    # 字段根据SQL语句来声明
    """
    id = db.Column(db.Integer, primary_key=True,comment="主键")
    name = db.Column(db.String(15), index=True, comment="姓名")
    age = db.Column(db.SmallInteger, comment="年龄")
    sex = db.Column(db.Boolean, default=True, comment="性别")
    email = db.Column(db.String(128), unique=True, comment="邮箱地址")
    money = db.Column(db.Numeric(10,2), default=0.0, comment="钱包")

    def __repr__(self): # 相当于django的__str__
        return f"{self.name}<{self.__class__.__name__}>"


# 所有的模型必须直接或间接继承于db.Model
class Course(db.Model):
    """课程数据模型"""
    __tablename__ = "db_course"
    """
    # 原生SQL语句
    create table db_course (
        id int primary key auto_increment comment "主键",
        name varchar(64) comment "课程",
        price NUMERIC(7,2) comment "价格",
        unique (name)
    );
    # 字段根据SQL语句来声明
    """
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="课程")
    price = db.Column(db.Numeric(7, 2), comment="价格")
    # repr()方法类似于django的__str__，用于打印模型对象时显示的字符串信息
    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"

class Teacher(db.Model):
    """老师数据模型"""
    __tablename__ = "db_teacher"
    """
    # 原生SQL语句
    create table db_teacher (
        id int primary key auto_increment comment "主键",
        name varchar(64) comment "姓名",
        option enum("讲师", "助教", "班主任") comment "职位",
        unique (`name`)
    );
    # 字段根据SQL语句来声明
    """
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="姓名")
    option = db.Column(db.Enum("讲师", "助教", "班主任"), default="讲师")

    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"

@app.route("/")
def index():
    return "ok"

if __name__ == '__main__':
    app.run()
```



# 数据表操作

### 创建和删除表

创建表

```python
# 在视图内调用：
@app.route("/create")
def create_table():
    db.create_all() # 为项目中被识别的所有模型创建数据表
    return "ok"


# 在视图以外的地方调用：
	with app.app_context():
        # create_all()方法执行的时候，需要放在模型的后面
        # 检测数据库中是否存在和模型匹配的数据表。
        # 如果没有，则根据模型转换的建表语句进行建表。
        # 如果找到，则不会进行额外处理
        db.create_all()
```

删除表

```python
# 在视图内调用：
@app.route("/drop")
def drop_table():
    db.drop_all()   # 为项目中被识别的所有模型删除数据表
    return "ok"


# 在视图以外的地方调用：
    with app.app_context():
        db.drop_all()  # 慎用，很给力的！！这表示删除数据库中所有模型对应的表。
```

代码：

```python
from flask import Flask, render_template
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
app = Flask(__name__, template_folder="templates", static_folder="static")

# 配置
app.config.update({
    "DEBUG": True,
    "SQLALCHEMY_DATABASE_URI": "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4",
    # 如果使用pymysql，则需要在连接时指定pymysql
    # "SQLALCHEMY_DATABASE_URI": "mysql+pymysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
    # 动态追踪修改设置，如未设置只会提示警告，设置False即可
    "SQLALCHEMY_TRACK_MODIFICATIONS": False,
    # ORM执行SQL查询时是哦否显示原始SQL语句，debug模式下可以开启
    "SQLALCHEMY_ECHO": True,
})

db.init_app(app)



class Student(db.Model):
    """学生管理"""
    __tablename__ = "db_student" # 表名
    # __abstract__ = True        # 抽象模型，数据迁移/建表的时候，不会认为这是一个模型，也就不会建表，往往用于设置公共模型，保存公共字段
    """
        # 企业中，往往大部分的公司会出现以下2种不同的数据库开发情况。
        # 1. 企业中有DBA，DBA会提前创建数据库和项目中具体业务的数据表。
             也就是说我们不需要自己手动建库建表，只需要根据数据库表结构，使用python声明对应的模型与之匹配，就可以操作数据库了。
        # 2. 企业没有DBA，比较坑爹：
        #    2.1 开发人员，自己手撸SQL语句，手动建库建表。
        #    2.2 开发人员，编写模型，使用数据迁移或者ORM提供建表方法，手动建库和数据迁移建表。

        # 原生SQL语句
        create table db_student(
          id int primary key auto_increment comment "主键",
          name varchar(15) comment "姓名",
          age smallint comment "年龄",
          sex tinyint(1) default 1 comment "性别",
          email varchar(255) comment "邮箱地址",
          money NUMERIC(10,2) default 0.0 comment "钱包",
          key (name),
          unique key (email)
        );
        # 字段根据SQL语句来声明
    """
    # 属性名 = db.Column(字段类型, 字段列约束选项)
    # 如果SQL语句中的字段名在python中是关键字/保留字，则建议改写绑定字段名
    # 属性名 = db.Column("字段名", 字段类型, 字段列约束选项)
    id = db.Column("student_id", db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(15), index=True, comment="姓名")
    age = db.Column(db.SmallInteger, comment="年龄")
    sex = db.Column(db.SmallInteger, comment="性别")
    email = db.Column(db.String(255), unique=True, comment="邮箱地址")
    money = db.Column(db.Numeric(10,2), default=0.0, comment="钱包")

    # repr()方法类似于django的__str__，用于打印模型对象时显示的字符串信息
    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"

# 所有的模型必须直接或间接继承于db.Model
class Course(db.Model):
    """课程数据模型"""
    __tablename__ = "db_course"
    """
    # 原生SQL语句
    create table db_course (
        id int primary key auto_increment comment "主键",
        name varchar(64) comment "课程",
        price NUMERIC(7,2) comment "价格",
        unique (name)
    );
    # 字段根据SQL语句来声明
    """
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="课程")
    price = db.Column(db.Numeric(7, 2), comment="价格")
    # repr()方法类似于django的__str__，用于打印模型对象时显示的字符串信息
    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"

class Teacher(db.Model):
    """老师数据模型"""
    __tablename__ = "db_teacher"
    """
    # 原生SQL语句
    create table db_teacher (
        id int primary key auto_increment comment "主键",
        name varchar(64) comment "姓名",
        option enum("讲师", "助教", "班主任") comment "职位",
        unique (`name`)
    );
    # 字段根据SQL语句来声明
    """
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="姓名")
    option = db.Column(db.Enum("讲师", "助教", "班主任"), default="讲师")

    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"


@app.route("/")
def index():
    title = "网站首页"
    return render_template("index.html", **locals())


@app.route("/create")
def create_table():
    db.create_all() # 为项目中被识别的所有模型创建数据表
    return "ok"


@app.route("/drop")
def drop_table():
    db.drop_all()   # 为项目中被识别的所有模型删除数据表
    return "ok"

if __name__ == '__main__':
    app.run()

```



# 数据操作

添加一条数据

```python
# 添加一条数据
student = Student(name="小明", age=17, email="xiaoming@qq.com", money=100) # 实例化模型对象
db.session.add(student) # 把模型对象添加数据库session会话对象中。db.session是SQLAlchemy中内置的会话管理对象sessionSM的成员
db.session.commit()     # 提交会话

# 再次插入一条数据
student2 = Student(name='小红', sex=False, age=13, email="16565666@qq.com", money=600)
db.session.add(student2)
db.session.commit()     # 提交会话
```



一次插入多条数据

```python
# 1. 先实例化要创建的模型对象
student = Student(name="小红", age=17, email="xiaohong@qq.com", money=200)
# 2. 把实例对象添加到连接会话
db.session.add(student)
# 1. 先实例化要创建的模型对象
student = Student(name="小花", age=16, email="xiaohua@qq.com", money=200)
# 2. 把实例对象添加到连接会话
db.session.add(student)
# 3. 只需要在结束的时候提交事务即可
db.session.commit()
```

```python
# 1. 先创建的列表要添加的实例化模型对象列表
student_list = [
    Student(name='wang', email='wang@163.com', age=20),
    Student(name='zhang', email='zhang@189.com', age=21),
    Student(name='chen', email='chen@126.com', age=19),
    Student(name='zhou', email='zhou@163.com', age=18),
    Student(name='tang', email='tang@163.com', age=16),
    Student(name='wu', email='wu@gmail.com', age=20),
    Student(name='qian', email='qian@gmail.com', age=21),
    Student(name='liu', email='liu@163.com', age=21),
    Student(name='li', email='li@163.com', age=18),
    Student(name='sun', email='sun@163.com', age=17),
]

# 2. 一次性添加到连接会话中
db.session.add_all(student_list)
db.session.commit()
```



删除数据

```python
# 方法1[先查询后删除，2条语句完成删除操作]
# 先查询出来
student = Student.query.first()
print(student)
# 再进行删除
db.session.delete(student)
db.session.commit()

# 方法2【1条语句完成删除操作，性能更好更高效】
# 类似乐观锁，在数据改动时添加条件并判断条件成立以后才进行数据操作，这种用法就是乐观锁
Student.query.filter(Student.id == 5).delete()
db.session.commit()


"""
悲观锁，是属于数据库中的一种锁机制，但是乐观锁并非真正的数据库锁。
2种锁都是数据库在应对并发操作时，防止出现资源抢夺的，基于不同人生观所实现2种解决方案。
悲观锁的基本使用：
    >>> 数据库终端开始

    begin;  -- 开启事务
    select * from db_student where student_id = 5 for update; -- 添加一把更新锁【悲观锁】
    ....    -- 在事务提交之前，任何第三方连接都不能修改 student_id = 5这条数据
    commit; -- 提交事务

    <<< 数据库终端开始

悲观锁的问题：
1. 提前锁定数据，形成串行化，形成阻塞，不利于性能发挥，不适用高并发场景。
2. 悲观锁只能保证数据的一致性，不能保证脏数据的出现

乐观锁的出现就是为了解决悲观锁的问题。
举例：双11活动，商城里面id=5的商品的库存=10了，现在我们要基于乐观锁和悲观锁来解决下单过程中，出现的资源抢夺现象，避免出现超卖（商品数量不能为负数）。

乐观锁：
---> begin;  开启事务
---> 先查看库存，记录当前库存 num=10
---> 进行下单操作，买6件
---> 付款
---> 扣除库存 update goods set num=num-6 where num=10 and id=5;  # 增加更新条件，判断库存是否还是原来
---> 如果执行成功，则表示没有人抢，购买成功
     如果执行事变，则表示已经有人先抢购
---> commit;

悲观锁：
---> begin; 开启事务
---> 先给id=5的数据，加锁
     select * from goods where id=5 for update;
---> 进行下单操作，买6件
---> 付款
---> 扣除库存  update goods set num=num-6 where id=5
---> 执行成功解锁
---- commit;  提交事务
"""
```



更新数据

```python
from flask import Flask, render_template
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
app = Flask(__name__, template_folder="templates", static_folder="static")

# 配置
app.config.update({
    "DEBUG": True,
    "SQLALCHEMY_DATABASE_URI": "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4",
    # 如果使用pymysql，则需要在连接时指定pymysql
    # "SQLALCHEMY_DATABASE_URI": "mysql+pymysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
    # 动态追踪修改设置，如未设置只会提示警告，设置False即可
    "SQLALCHEMY_TRACK_MODIFICATIONS": False,
    # ORM执行SQL查询时是哦否显示原始SQL语句，debug模式下可以开启
    "SQLALCHEMY_ECHO": True,
})

db.init_app(app)



class Student(db.Model):
    """学生管理"""
    __tablename__ = "db_student" # 表名
    # 属性名 = db.Column(字段类型, 字段列约束选项)
    # 如果SQL语句中的字段名在python中是关键字/保留字，则建议改写绑定字段名
    # 属性名 = db.Column("字段名", 字段类型, 字段列约束选项)
    id = db.Column("student_id", db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(15), index=True, comment="姓名")
    age = db.Column(db.SmallInteger, comment="年龄")
    sex = db.Column(db.SmallInteger, comment="性别")
    email = db.Column(db.String(255), unique=True, comment="邮箱地址")
    money = db.Column(db.Numeric(10,2), default=0.0, comment="钱包")

    # repr()方法类似于django的__str__，用于打印模型对象时显示的字符串信息
    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"


# 所有的模型必须直接或间接继承于db.Model
class Course(db.Model):
    """课程数据模型"""
    __tablename__ = "db_course"
    """
    # 原生SQL语句
    create table db_course (
        id int primary key auto_increment comment "主键",
        name varchar(64) comment "课程",
        price NUMERIC(7,2) comment "价格",
        unique (name)
    );
    # 字段根据SQL语句来声明
    """
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="课程")
    price = db.Column(db.Numeric(7, 2), comment="价格")
    # repr()方法类似于django的__str__，用于打印模型对象时显示的字符串信息
    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"


class Teacher(db.Model):
    """老师数据模型"""
    __tablename__ = "db_teacher"
    """
    # 原生SQL语句
    create table db_teacher (
        id int primary key auto_increment comment "主键",
        name varchar(64) comment "姓名",
        option enum("讲师", "助教", "班主任") comment "职位",
        unique (`name`)
    );
    # 字段根据SQL语句来声明
    """
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="姓名")
    option = db.Column(db.Enum("讲师", "助教", "班主任"), default="讲师")

    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"


@app.route("/")
def index():
    return "ok"


@app.route("/create")
def create_table():
    db.create_all() # 为项目中被识别的所有模型创建数据表
    return "ok"


@app.route("/drop")
def drop_table():
    db.drop_all()   # 为项目中被识别的所有模型删除数据表
    return "ok"


"""一次添加一条数据"""
@app.route("/add")
def add_student():
    # 1. 先实例化要创建的模型对象
    student = Student(name="小明", age=17, email="xiaoming@qq.com", money=100)  # 实例化模型对象
    # 2. 把实例对象添加到连接会话
    db.session.add(student)
    # 3. 提交事务
    db.session.commit()
    return "ok"


"""一次添加多条数据"""
@app.route("/madd")
def multi_add():
    # 1. 先实例化要创建的模型对象
    student = Student(name="小红", age=17, email="xiaohong@qq.com", money=200)
    # 2. 把实例对象添加到连接会话
    db.session.add(student)
    # 1. 先实例化要创建的模型对象
    student = Student(name="小花", age=16, email="xiaohua@qq.com", money=200)
    # 2. 把实例对象添加到连接会话
    db.session.add(student)
    # 3. 只需要在结束的时候提交事务即可
    db.session.commit()
    return "ok"


@app.route("/madd2")
def multi_add2():
    # 1. 先创建的列表要添加的实例化模型对象列表
    student_list = [
        Student(name='wang', email='wang@163.com', age=20),
        Student(name='zhang', email='zhang@189.com', age=21),
        Student(name='chen', email='chen@126.com', age=19),
        Student(name='zhou', email='zhou@163.com', age=18),
        Student(name='tang', email='tang@163.com', age=16),
        Student(name='wu', email='wu@gmail.com', age=20),
        Student(name='qian', email='qian@gmail.com', age=21),
        Student(name='liu', email='liu@163.com', age=21),
        Student(name='li', email='li@163.com', age=18),
        Student(name='sun', email='sun@163.com', age=17),
    ]

    # 2. 一次性添加到连接会话中
    db.session.add_all(student_list)
    db.session.commit()

    return "ok"



@app.route("/del")
def delete_student():
    """删除一条数据"""
    # 先查询出来
    student = Student.query.first()
    # student = db.session.query(Student).first()
    # 再进行删除
    db.session.delete(student)
    db.session.commit()

    return "ok"


@app.route("/mdel")
def multi_delete_student():
    """按条件删除多条数据"""
    Student.query.filter(Student.id > 5).delete()
    # db.session.query(Student).filter(Student.id > 5).delete()
    db.session.commit()

    return "ok"


@app.route("/update")
def update():
    """更新一条"""
    # 先查询出来
    student = Student.query.filter(Student.id == 4).first()
    student.name = "小白"
    db.session.commit()
    return "ok"


@app.route("/update2")
def update2():
    """直接根据条件更新一条或多条数据"""
    Student.query.filter(Student.name == 'zhang', Student.money == -99.00).update({'money': 1998})
    db.session.commit()
    return "ok"


@app.route("/update3")
def update3():
    # 字段引用[利用当前一条数据的字典值进行辅助操作,实现类似django里面F函数的效果]
    # 每次自增100
    Student.query.filter(Student.name == "小花").update({"money": Student.money + 100})
    db.session.commit()
    return "ok"


@app.route("/update4")
def update4():
    # 字段引用[利用当前一条数据的字典值进行辅助操作,实现类似django里面F函数的效果]
    # 在原有money的基础上按age补贴1000*age
    Student.query.filter(Student.name == "zhang").update({"money": Student.money + 1000 * Student.age})
    db.session.commit()
    return "ok"

if __name__ == '__main__':
    app.run()
```



## 基本查询

### SQLAlchemy常用的查询过滤器

| 过滤器         | 说明                                                         |
| :------------- | :----------------------------------------------------------- |
| **filter()**   | 把过滤器添加到原查询上，返回一个新查询                       |
| filter_by()    | 把等值过滤器添加到原查询上，返回一个新查询                   |
| **limit()**    | 使用指定的值限定原查询返回的**结果数量**                     |
| **offset()**   | 设置结果范围的**开始位置**，偏移原查询返回的结果，返回一个新查询 |
| **order_by()** | 根据指定条件对原查询结果进行**排序**，返回一个新查询         |
| **group_by()** | 根据指定条件对原查询结果进行**分组**，返回一个新查询         |

### SQLAlchemy常用的查询结果方法

| 方法           | 说明                                                         |
| :------------- | :----------------------------------------------------------- |
| **all()**      | 以**列表形式**返回查询的所有结果                             |
| **first()**    | 返回查询的第一个结果，**模型对象**，如果未查到，返回**None** |
| first_or_404() | 返回查询的第一个结果，**模型对象**，如果未查到，通过abort 返回404异常 |
| **get()**      | 返回**指定主键**对应的**模型对象**，如不存在，返回None       |
| get_or_404()   | 返回指定主键对应的行，如不存在，abort 返回404                |
| **count()**    | 返回查询结果的**数量**                                       |
| **paginate()** | 返回一个Paginate**分页器对象**，它包含指定范围内的结果       |
| **having()**   | 返回结果中符合条件的数据，**必须跟在group by后面**，其他地方无法使用。 |



get():参数为主键，表示根据主键查询数据，如果主键不存在返回None

```python
@app.route("/get")
def get():
    """按主键获取一条"""
    # student = Student.query.get({"id": 5})
    # student = Student.query.get((5,))
    # student = db.session.query(Student).get(5)
    student = Student.query.get(5)
    print(student)
    return "ok"

```

代码：

```python
# 前面代码省略


@app.route("/")
def index():
    return "ok"


@app.route("/create")
def create_table():
    db.create_all() # 为项目中被识别的所有模型创建数据表
    return "ok"


@app.route("/drop")
def drop_table():
    db.drop_all()   # 为项目中被识别的所有模型删除数据表
    return "ok"

@app.route("/query")
def query():
    query1 = Student.query   # 简写操作
    query2 = db.session.query(Student)
    print(type(query1), query1)
    print(type(query2), query2)
    return "ok"


@app.route("/get")
def get():
    """get根据主键获取数据"""
    # student1 = Student.query.get({"id":10})
    student1 = Student.query.get(30)
    # student2 = db.session.query(Student).get({"id":10})
    student2 = db.session.query(Student).get(30)
    # 结果是模型对象
    print(type(student1), student1)
    print(type(student2), student2)
    # 查询不到结果，则返回值为None
    if student1:
        print(student1.name, student1.age)
    if student2:
        print(student2.name, student2.age)
    return "ok"

if __name__ == '__main__':
    app.run()

```



all()返回查询到的所有对象

```python
模型类.query.all()

"""获取多个数据"""
student = Student.query.all()
print(student) # [dong<Student>, 小红<Student>, wang<Student>, chen<Student>, zhou<Student>, tang<Student>, wu<Student>, qian<Student>, liu<Student>, li<Student>, sun<Student>]

student = Student.query.filter(Student.id<5).all()  # 没有结果返回空列表[]
print(student) # [dong<Student>, 小红<Student>, wang<Student>]

# all()的返回值是一个python列表，可以直接使用切片，与django的QuerySet完全不是一回事。
student = Student.query.filter(Student.id < 5).all()[:-1]  # 没有结果返回空列表[]
print(student) # [dong<Student>, 小红<Student>]
```



count 返回结果的数量

```python
# 返回结果的数量
ret = Student.query.filter(Student.id < 5).count()
print(f"ret={ret}")
```



first()返回查询到的第一个对象【first获取一条数据,all获取多条数据】

```python
模型类.query.first()

"""获取第一个数据"""
student = Student.query.first()
print(student)

student = Student.query.filter(Student.id==5).first() # 没有结果返回None
print(student)
```



filter条件查询，支持各种运算符和查询方法或者模糊查询方法。

返回名字结尾字符为g的所有数据。

```python
# 模糊查询
# 使用163邮箱的所有用户
student_list = Student.query.filter(Student.email.endswith("@163.com")).all()
print(student_list)

# 姓名以"zh"开头的
student_list = Student.query.filter(Student.name.startswith("zh")).all()
print(student_list)

# 名字中带有"a"字母的数据
student_list = Student.query.filter(Student.name.contains("a")).all()
print(student_list)

"""单条件比较"""
# 则需要指定条件格式为: filter(模型.字段 比较运算符 值)。
# 运算符可以是: ==表示相等, !=不相等，> 表示大于  < 表示小于，>=大于等于，<=小于等于
# student_list = Student.query.filter(Student.age > 18).all()
# print(student_list) # [wang<Student>, chen<Student>, zhou<Student>,...]

"""多条件比较"""
# 要求多个条件都要满足，相当于逻辑查询中的 并且(and)！！
student_list = Student.query.filter(Student.age > 18, Student.sex == True).all()
print(student_list) # [wang<Student>, chen<Student>, qian<Student>, liu<Student>]
```



filter_by精确条件查询

filter_by 只支持字段的**值是否相等**的情况，对于大于、等于、等等其他条件是不支持的。

例如：返回age等于22的学生

```python
# 单条件
student_list = Student.query.filter_by(age=22).all()  # 字段添加不需要附带模型类
print(student_list)

# 多条件
student_list = Student.query.filter_by(age=22,sex=True).all()
print(student_list)
```



练习：

```python
查询所有男生[Student.sex==True]数据

查询id为4的学生[3种方式]

查询年龄等于22的所有学生数据

查询name为小白的学生数据

查询20岁的男生
```



代码：

```python
from flask import Flask, render_template
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
app = Flask(__name__, template_folder="templates", static_folder="static")

# 配置
app.config.update({
    "DEBUG": True,
    "SQLALCHEMY_DATABASE_URI": "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4",
    # 如果使用pymysql，则需要在连接时指定pymysql
    # "SQLALCHEMY_DATABASE_URI": "mysql+pymysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
    # 动态追踪修改设置，如未设置只会提示警告，设置False即可
    "SQLALCHEMY_TRACK_MODIFICATIONS": False,
    # ORM执行SQL查询时是哦否显示原始SQL语句，debug模式下可以开启
    "SQLALCHEMY_ECHO": True,
})

db.init_app(app)


class Student(db.Model):
    """学生管理"""
    __tablename__ = "db_student" # 表名
    # 属性名 = db.Column(字段类型, 字段列约束选项)
    # 如果SQL语句中的字段名在python中是关键字/保留字，则建议改写绑定字段名
    # 属性名 = db.Column("字段名", 字段类型, 字段列约束选项)
    id = db.Column("student_id", db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(15), index=True, comment="姓名")
    age = db.Column(db.SmallInteger, comment="年龄")
    sex = db.Column(db.SmallInteger, comment="性别")
    email = db.Column(db.String(255), unique=True, comment="邮箱地址")
    money = db.Column(db.Numeric(10,2), default=0.0, comment="钱包")

    # repr()方法类似于django的__str__，用于打印模型对象时显示的字符串信息
    def __repr__(self):
        return f"<{self.__class__.__name__} {self.name}>"


# 所有的模型必须直接或间接继承于db.Model
class Course(db.Model):
    """课程数据模型"""
    __tablename__ = "db_course"
    """
    # 原生SQL语句
    create table db_course (
        id int primary key auto_increment comment "主键",
        name varchar(64) comment "课程",
        price NUMERIC(7,2) comment "价格",
        unique (name)
    );
    # 字段根据SQL语句来声明
    """
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="课程")
    price = db.Column(db.Numeric(7, 2), comment="价格")
    # repr()方法类似于django的__str__，用于打印模型对象时显示的字符串信息
    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"


class Teacher(db.Model):
    """老师数据模型"""
    __tablename__ = "db_teacher"
    """
    # 原生SQL语句
    create table db_teacher (
        id int primary key auto_increment comment "主键",
        name varchar(64) comment "姓名",
        option enum("讲师", "助教", "班主任") comment "职位",
        unique (`name`)
    );
    # 字段根据SQL语句来声明
    """
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="姓名")
    option = db.Column(db.Enum("讲师", "助教", "班主任"), default="讲师")

    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"


@app.route("/")
def index():
    return "ok"


@app.route("/create")
def create_table():
    db.create_all() # 为项目中被识别的所有模型创建数据表
    return "ok"


@app.route("/drop")
def drop_table():
    db.drop_all()   # 为项目中被识别的所有模型删除数据表
    return "ok"

@app.route("/exam")
def exam():
    # 查询所有男生[Student.sex == True]数据
    # student_list = Student.query.filter(Student.sex == True).all()
    # print(student_list)

    # 查询id为4的学生[3种方式]
    # 1. get
    # student = Student.query.get(4)
    # print(student)

    # 2. filter+first
    # student = Student.query.filter(Student.id == 4).first()
    # print(student)

    # 3. filter_by + first
    # student = Student.query.filter_by(id=4).first()
    # print(student)

    # 查询年龄等于20的所有学生数据
    # student_list = Student.query.filter(Student.age == 20).all()
    # print(student_list)

    # 查询name为小白的学生数据
    # student = Student.query.filter(Student.name == "小白").first()
    # print(student)

    # 查询20岁的男生
    student_list = Student.query.filter(Student.age == 20, Student.sex == True).all()
    print(student_list)

    return "ok"

if __name__ == '__main__':
    app.run()

```



### 多条件查询

逻辑与，需要导入`and_`，返回`and_()`条件满足的所有数据

```python
from sqlalchemy import and_
Student.query.filter(and_(Student.name!='wang',Student.email.endswith('163.com'))).all()


# # and_(条件1,条件2,....)  等价于  filter(条件1,条件2,.....)
# # age > 18 and email like "%163.com"
# # student_list = Student.query.filter(Student.age > 18, Student.email.endswith("163.com")).all()
#
# student_list = Student.query.filter(
#     and_(
#         Student.age > 18,
#         Student.email.endswith("163.com")
#     )
# ).all()
```



逻辑或，需要导入or_

```python
from sqlalchemy import or_
Student.query.filter(or_(Student.name!='wang',Student.email.endswith('163.com'))).all()

# 查询年龄在20岁，使用的邮箱是qq或者163邮箱的
student_list = Student.query.filter(
    Student.age == 20,
    or_(
        Student.email.endswith("qq.com"),
        Student.email.endswith("163.com")
    )
).all()


# 复合条件的查询情况
# 查询年龄>17岁的女生或者年龄>18岁的男生
student_list = Student.query.filter(
    or_(
        and_(Student.age > 17, Student.sex == False),
        and_(Student.age > 18, Student.sex == True),
    )
).all()
print(student_list)

print(student_list)
```



逻辑非，返回名字不等于"小白"的所有数据

```python
Student.query.filter(Student.name!='小白').all()
```

not_ 相当于取反

```python
from sqlalchemy import not_
Student.query.filter(not_(Student.name=='小白')).all()

# # 查询年龄不等于22
# student_list = Student.query.filter(Student.age != 22).all()
# print(student_list)
# student_list = Student.query.filter(not_(Student.age==22)).all()
# print(student_list)
```



in_范围查询

```python
# 查询id是 5, 7, 10 的学生信息
student_list = Student.query.filter(Student.id.in_([5, 7, 10])).all()
print(student_list)

# 查询id不是 1 3 5 的学生信息
student_list = Student.query.filter(not_(Student.id.in_([1, 3, 5]))).all()
print( student_list )
```

is_判断值查询

```python
    """判断值查询"""
    student_list = Student.query.filter(Student.email.is_(None)).all()
    print(student_list)

    student_list = Student.query.filter(Student.email == None).all()
    print(student_list)
```



order_by 排序

```python
# 倒序[值从大到小]
student_list = Student.query.order_by(Student.id.desc()).all()
# 升序[值从小到大]
student_list = Student.query.order_by(Student.id.asc()).all()

# 多字段排序[第一个字段值一样时，比较第二个字段，进行排序]
student_list = Student.query.order_by(Student.money.asc(), Student.age.asc(), Student.id.asc()).all()
print(student_list)
```



count统计

```python
# 查询age>=19的男生的数量
from sqlalchemy import and_
# ret = Student.query.filter( and_(Student.age>=19,Student.sex==True) ).count()
ret = Student.query.filter( Student.age>=19, Student.sex==True ).count()
```



对结果进行偏移量和数量的限制

```python
# 查询年龄最大的3个学生
student_list = Student.query.order_by(Student.age.desc()).limit(3).all()
print(student_list)

# 查询年龄排第4到第7名的学生
student_list = Student.query.order_by(Student.age.desc()).offset(3).limit(4).all()
print(student_list)

# 查询年龄最小的3个人
student_list = Student.query.order_by(Student.age.asc()).limit(3).all()
print(student_list)
```

SQL

```sql
# 查询年龄最大的3个学生
# select * from db_student order by age desc limit 3;

# 查询年龄排第4到第7名的学生
select * from db_student order by age desc limit 3, 4;
# select * from db_student order by age desc limit 4 offset 3;

# 查询年龄最小的3个人
# select * from db_student order by age asc limit 3;
```



练习

```python
# 查询age是18 或者 使用163邮箱的所有学生
Student.query.filter(or_(Student.age==18,Student.email.endswith("163.com"))).all()
# 查询id为 [1, 3, 5, 7, 9] 的学生列表
student_list = Student.query.filter(Student.id.in_([1, 3, 5, 7, 9])).all()
print(student_list)
```



### 分页器

manage.py，代码：

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
class Config(object):
    DEBUG = True
    # 数据库链接配置 = 数据库名称://登录账号:登录密码@数据库主机IP:数据库访问端口/数据库名称?charset=编码类型
    SQLALCHEMY_DATABASE_URI = "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
    # 动态追踪修改设置，如未设置只会提示警告
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    # ORM运行时会显示ORM生成的原始SQL语句[调试]
    SQLALCHEMY_ECHO = True

app.config.from_object(Config)


"""模型类定义"""
db = SQLAlchemy(app=app)

class Student(db.Model):
    """学生信息模型"""
    # 声明与当前模型绑定的数据表名称
    __tablename__ = "db_student"
    id = db.Column(db.Integer, primary_key=True,comment="主键")
    name = db.Column(db.String(15), index=True, comment="姓名")
    age = db.Column(db.SmallInteger, comment="年龄")
    sex = db.Column(db.Boolean, default=True, comment="性别")
    email = db.Column(db.String(128), unique=True, comment="邮箱地址")
    money = db.Column(db.Numeric(10,2), default=0.0, comment="钱包")

    def __repr__(self): # 相当于django的__str__
        return f"{self.name}<{self.__class__.__name__}>"


# 所有的模型必须直接或间接继承于db.Model
class Course(db.Model):
    """课程数据模型"""
    __tablename__ = "db_course"
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="课程")
    price = db.Column(db.Numeric(7, 2), comment="价格")
    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"

class Teacher(db.Model):
    """老师数据模型"""
    __tablename__ = "db_teacher"
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="姓名")
    option = db.Column(db.Enum("讲师", "助教", "班主任"), default="讲师")

    def __repr__(self):
        return f"{self.name}<{self.__class__.__name__}>"

from flask import request,render_template

@app.route("/")
def index():
    # 分页器
    page  = int(request.args.get("page", 1))
    size = int(request.args.get("size", 3))
    pagination = Student.query.paginate(page, size)
    print(pagination)
    """
    from flask_sqlalchemy import Pagination
    # 常用属性
    total  总数据量
    items  每一页数据项列表
    pages  总页码===> math.ceil( total/per_page )

    # 常用方法
    prev      上一页分页对象
    prev_num  上一页页码
    has_prev  是否有上一页

    next      下一页分页对象
    next_num  下一页页码
    has_next  是否有下一页
    """
    print(pagination.items)  # 当前页的数据项列表
    print(pagination.total)  # 总数据量
    print(pagination.pages)  # 总页码数量
    print(pagination.prev_num)  # 上一页页码
    print(pagination.next_num)  # 下一页页码
    print(pagination.has_prev)  # 是否有上一页
    print(pagination.has_next)  # 是否有下一页
    print(pagination.prev())    # 上一页的分页对象
    print(pagination.next())    # 下一页的分页对象

    # """前后端分离"""
    # data = {
    #     "page": pagination.page, # 当前页码
    #     "pages": pagination.pages, # 总页码
    #     "has_prev": pagination.has_prev, # 是否有上一页
    #     "prev_num": pagination.prev_num, # 上一页页码
    #     "has_next": pagination.has_next, # 是否有下一页
    #     "next_num": pagination.next_num, # 下一页页码
    #     "items": [{
    #         "id": item.id,
    #         "name": item.name,
    #         "age": item.age,
    #         "sex": item.sex,
    #         "money": item.money,
    #     } for item in pagination.items]
    # }
    # return data


    """前后端不分离"""
    return render_template("list.html",**locals())





if __name__ == '__main__':
    with app.app_context():
        db.create_all()
    app.run()
```

list.html，代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
    .page a,.page span{
        padding: 2px 6px;
        color: #fff;
        background: #6666ff;
        text-decoration: none;
    }
    .page span{
        color: #fff;
        background: orange;
    }

    </style>
</head>
<body>
    <table border="1" align="center" width="600">
        <tr>
           <th>ID</th>
           <th>age</th>
           <th>name</th>
           <th>sex</th>
           <th>money</th>
        </tr>
        {% for student in pagination.items %}
        <tr>
           <td>{{ student.id }}</td>
           <td>{{ student.age }}</td>
           <td>{{ student.name }}</td>
           <td>{{ "男" if student.sex else "女" }}</td>
           <td>{{ student.money }}</td>
        </tr>
        {% endfor %}
        <tr align="center">
            <td colspan="5" class="page">
                {% if pagination.has_prev %}
                <a href="?page=1">首  页</a>
                <a href="?page={{ pagination.page-1 }}">上一页</a>
                <a href="?page={{ pagination.page-1 }}">{{ pagination.page-1 }}</a>
                {% endif %}
                <span>{{ pagination.page }}</span>
                {% if pagination.has_next %}
                <a href="?page={{ pagination.page+1 }}">{{ pagination.page+1 }}</a>
                <a href="?page={{ pagination.page+1 }}">下一页</a>
                <a href="?page={{ pagination.pages }}">尾  页</a>
                {% endif %}
            </td>
        </tr>
    </table>
</body>
</html>
```



### 聚合分组

分组查询和分组查询结果过滤

一般分组都会结合**聚合函数**来一起使用。SQLAlchemy中所有的聚合函数都在`func`模块中声明的。

```python
from sqlalchemy import func
```

| 函数名     | 说明     |      |
| ---------- | -------- | ---- |
| func.count | 统计总数 |      |
| func.avg   | 平均值   |      |
| func.min   | 最小值   |      |
| func.max   | 最大值   |      |
| func.sum   | 求和     |      |

代码：

```python
"""聚合函数"""
from sqlalchemy import func
# 获取所有学生的money总数
# SELECT sum(db_student.money) AS sum_1 FROM db_student LIMIT %s
# ret = db.session.query(func.sum(Student.money)).first()[0]
# print(ret) # 3998.0
# # 查询女生的数量
# ret = db.session.query(func.count(Student.id)).filter(Student.sex==False).first()[0]
# print(ret) # 7
# # 查询所有学生的平均年龄
# ret = db.session.query(func.avg(Student.age)).first()[0]
# print(ret) # 18.2727

"""
聚合分组
在聚合分组的情况下，db.session.query中的参数只能是被分组的字段或者是被聚合的数据
"""
# # 查询当前所有男生女生的数量
# ret = db.session.query(Student.sex,func.count(Student.id)).group_by(Student.sex).all()
# print(ret) # [(False, 7), (True, 4)]

# # 查询各个年龄段的学生数量
# ret = db.session.query(Student.age, func.count(Student.id)).group_by(Student.age).all()
# print(ret) # [(15, 2), (13, 1), (22, 4), (19, 1), (18, 1), (16, 1), (17, 1)]
#
# # 查看当前男生女生的平均年龄
# ret = db.session.query(Student.sex, func.avg(Student.age)).group_by(Student.sex).all()
# ret = [{"sex":"男" if item[0] else "女","age":float(item[1])} for item in ret]
# print(ret) # [{'sex': '女', 'age': 18.0}, {'sex': '男', 'age': 18.75}]

# # 分组后的过滤操作 having
# # 在所有学生中，找出各个年龄中拥有最多钱的同学，并在这些同学里面筛选出money > 500的数据
# subquery = func.max(Student.money)
# print(subquery) # max(db_student.money)
# ret = db.session.query(Student.age, subquery).group_by(Student.age).having(subquery > 500).all()
# print(ret)  # [(18, Decimal('1000.00')), (22, Decimal('26000.00')), (23, Decimal('1998.00'))]

"""
多字段分组
    字段1   字段2
    1      3
    2      4

    分组如下：
    13
    14
    23
    24
"""
# 各个年龄里，男生和女生的money总数
subquery = func.sum(Student.money)
ret = db.session.query(Student.sex, Student.age, subquery).group_by(Student.sex, Student.age).all()
print(ret) # [(False, 15, 1000.0), (False, 13, 600.0), (True, 15, 0.0), (True, 22, 1998.0), (False, 19, 0.0), (False, 22, 400.0), (False, 18, 0.0), (True, 16, 0.0), (False, 17, 0.0)]
```



SQL方法中的关键字顺序：

```python
模型.query.   // db.session.query.
filter/ filter_by
group by
having
order_by
limit  offset
all / get / first / count / paginate
```



### 执行原生SQL语句

```python
"""执行原生SQL语句"""
# # 查询多条数据
# ret = db.session.execute("select * from db_student").fetchall()
# print(ret)
# # 查询一条数据
# ret = db.session.execute("select * from db_student").fetchone()
# print(ret)

"""
    name  age  achievement
               80
    小明   17   81
               83

    group_concat 逗号合并
    小明   17   80,81,83

    concat  字符串拼接
    小明   17   808183
    """

# # 添加数据
# db.session.execute("insert db_student (name,age,sex,email,money) select name,age,sex,concat(now(),email),money from db_student")
# db.session.commit()

# # # 更新/删除
# db.session.execute("UPDATE db_student SET money=(db_student.money + %s) WHERE db_student.age = %s" % (200, 22))
# db.session.commit()


"""分组合并"""
# 统计各个年龄段的学生人数，并记录对应年龄段的学生ID
ret = db.session.execute("select age,count(id),group_concat(id) from db_student group by age").fetchall()
print(ret)
return "ok"
```

## 关联查询

### 常用的SQLAlchemy关系选项

| 选项名         | 说明                                                         |
| :------------- | :----------------------------------------------------------- |
| backref        | 在关系的另一模型中添加**反向引用**,用于设置外键名称,在1查多的 |
| primary join   | 明确指定两个模型之间使用的连表条件, 用于1对1 或者1对多连表中 |
| lazy           | 指定如何加载关联模型数据的方式，用于1对1或1对多链表中。参数值:<br>select（立即加载，查询所有相关数据显示，相当于lazy=True）<br>subquery（立即加载，但使用子查询）<br>dynamic（不立即加载，但提供加载记录的查询对象） |
| uselist        | 指定1对1或1对多连表时，返回的数据结果是模型对象还是模型列表，如果为False，不使用列表，而使用模型对象。<br>1对1或多对1关系中，需要设置relationship中的uselist=Flase，1对多或多对多关系中，需要设置relationshio中的uselist=True。 |
| secondary      | 指定多对多关系中关系表的名字。<br>多对多关系中，需建立关系表，设置 secondary=关系表 |
| secondary join | 在SQLAlchemy中无法自行决定时，指定多对多关系中的二级连表条件，绑定主外键。 |

范式理论：一套提供给数据库开发者设置标准、规范的数据库的理论。

```
1NF. 数据不可再分，必须保证原子性。数据的值保证可以方便存储，不可再分。
2NF. 数据不能重复，必须保证唯一性。必须使用主键来进行区分每一行数据。
3NF. 数据不能冗余，必须保证关联性。冗余的数据必须使用另外的数据表存放，并与当前表进行关联。

基于实际业务的角度出发，设计出违背了范式理论的表结构。
逆范式：以空间换时间

class table:
    id   name      teacher      
    1    301班      李老师       
    2    302班      王老师      

student table:
    id   name        class_id
    1    xiaoming    1
    2    xiaohong    2
    3    xiaohui     3
    
    

```



三范式+逆范式

第三范式：数据不能冗余，把关联性不强的数据可以移除到另一个表中。使用外键进行管理。

```python
1对1：把主表的主键放到附加表中作为外键存在。

      商品信息表
      id   name        price   image    描述    售后   配置    包装
      1    PC-1-gr      100    1.png    
      2    PC-1-re      100    1.png
      3    PC-1-he      100    1.png
      4    PC-1-bu      100    1.png

1对多：把主表(1) 的主键放到附加表(多)作为外键存在。
     订单信息表        订单详情表
     1个订单     ----> 多个商品

多对多：把主表(多)的主键和附加表的(多)主键，放到第三方表(关系表)中作为外键。
     用户表            课程表
      1  xiaoming      1   python
      2  xiaohong      2   django
      3  xiaolong      3   flask
                
     用户与课程的购买关系表
     user_id   course
     1          1
     1          2
     1          3
```



### 模型之间的关联

#### 一对一

常见的业务：主表和详情表（用户、会员、学生、商品、文章、主机）

```python
class Student(db.Model):
    """个人信息主表"""
	....
    # 关联属性，这个不会被视作表字段，只是模型对象的属性。
    # 因为StudentInfo和Student是一对一的关系，所以uselist=False表示关联一个数据
    info = db.relationship("StudentInfo", uselist=False, backref="own")


class StudentInfo(db.Model):
    """个人信息附加表"""

    # 外键，
    # 如果是一对一，则外键放在附加表对应的模型中
    # 如果是一对多，则外键放在多的表对象的模型中
    # sid = db.Column(db.Integer, db.ForeignKey(Student.id),comment="外键")
    sid = db.Column(db.Integer, db.ForeignKey("student表名.主键"),comment="外键")
```



##### 关联属性声明在主模型中【最常用】

代码：

```python
import json
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy import func

db = SQLAlchemy()
app = Flask(__name__, template_folder="templates", static_folder="static")

# 配置
app.config.update({
    "DEBUG": True,
    "SQLALCHEMY_DATABASE_URI": "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4",
    # 如果使用pymysql，则需要在连接时指定pymysql
    # "SQLALCHEMY_DATABASE_URI": "mysql+pymysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
    # 动态追踪修改设置，如未设置只会提示警告，设置False即可
    "SQLALCHEMY_TRACK_MODIFICATIONS": False,
    # ORM执行SQL查询时是哦否显示原始SQL语句，debug模式下可以开启
    "SQLALCHEMY_ECHO": True,
})

db.init_app(app)


class Student(db.Model):
    """学生管理"""
    __tablename__ = "tb_student"  # 表名
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(15), index=True, comment="姓名")
    age = db.Column(db.SmallInteger, comment="年龄")
    sex = db.Column(db.SmallInteger, comment="性别")
    email = db.Column(db.String(255), unique=True, comment="邮箱地址")
    money = db.Column(db.Numeric(10,2), default=0.0, comment="钱包")
    # 模型的关联属性，不会在数据表创建字段
    # 因为StudentInfo和Student是一对一的关系，所以uselist=False表示关联一个数据
    info = db.relationship("StudentInfo", uselist=False, backref="student")

    def __repr__(self):
        return json.dumps(self.__to_dict__, ensure_ascii=False)

    @property
    def __to_dict__(self):
        return {
            "id": self.id,
            "name": self.name,
            "age": self.age,
            "sex": self.sex,
            "email": self.email,
            "money": float(self.money),
        }


class StudentInfo(db.Model):
    __tablename__ = "tb_student_info"  # 表名
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    # 附加表的外键，就是主表的主键
    # 如果是一对一，则外键放在附加表对应的模型中
    # 如果是一对多，则外键放在多的附加表对应的模型中
    sid = db.Column(db.Integer, db.ForeignKey("tb_student.id"), comment="外键")  # mysql字段
    # student = db.relationship("Student", uselist=False, backref="info")     # ORM关联属性
    mobile = db.Column(db.String(15), index=True, comment="手机号码")
    address = db.Column(db.String(255), nullable=True, comment="家庭地址")

    def __repr__(self):
        return json.dumps(self.__to_dict__, ensure_ascii=False)

    @property
    def __to_dict__(self):
        return {
            "id": self.id,
            "sid": self.sid,
            "mobile": self.mobile,
            "address": self.address,
        }


@app.route("/create")
def create_table():
    db.create_all() # 为项目中被识别的所有模型创建数据表
    return "ok"


@app.route("/drop")
def drop_table():
    db.drop_all()   # 为项目中被识别的所有模型删除数据表
    return "ok"


@app.route("/a1")
def a1():
    """添加操作"""
    # 添加主表信息的时候通过关联属性db.relationship同步添加附件表信息
    student = Student(
        name="xiaolan01",
        age=16,
        sex=False,
        money=10000,
        email="xiaolan01@qq.com",
        info=StudentInfo(
            address="北京市昌平区百沙路203号",
            mobile="13312345672"
        )
    )

    db.session.add(student)
    db.session.commit()

    return "ok"


@app.route("/a2")
def a2():
    """添加操作"""
    # 假设已经存在主表信息，后面再补充附加表信息
    # student = Student(
    #     name="xiaolan03",
    #     age=16,
    #     sex=False,
    #     money=10000,
    #     email="xiaolan03@qq.com",
    # )
    #
    # db.session.add(student)
    # db.session.commit()

    # 上面代码先执行了，现有1个学生没有附加信息的，我们可以在后续代码通过查询主表的主键，补充附加表数据
    student = Student.query.get(3)
    if not student.info:
        """添加附加表数据"""
        """方式1"""
        # student.info = StudentInfo(mobile=13300010002, address="北京市昌平区百沙路205号",)
        # db.session.commit()

        """方式2"""
        # info = StudentInfo(
        #     student=student,  # 关联属性
        #     mobile=13300010002,
        #     address="北京市昌平区百沙路205号",
        # )
        #
        # db.session.add(info)
        # db.session.commit()

        """方式3"""
        info = StudentInfo(
            sid=student.id,  # 外键
            mobile=13300010002,
            address="北京市昌平区百沙路205号",
        )

        db.session.add(info)
        db.session.commit()

    return "ok"


@app.route("/a3")
def a3():
    """添加操作"""
    # 添加附加模型数据的同时，把主模型也进行添加
    info = StudentInfo(
        mobile=13300010003,
        address="北京市昌平区百沙路206号",
        student=Student(
            name="xiaolan04",
            age=17,
            sex=False,
            money=10000,
            email="xiaolan04@qq.com",
        ),
    )

    db.session.add(info)
    db.session.commit()

    return "ok"


@app.route("/g1")
def g1():
    """查询操作"""
    # # 正向关联----> 从主模型查询外键模型
    # student = Student.query.get(2)
    # print(student.name)          # xiaolan01
    # # 如果主模型没有调用关联属性查询附加模型对象，则ORM不会执行查询关联模型的SQL语句
    # print(student.info)          # {"id": 2, "sid": 2, "mobile": "13312345672", "address": "北京市昌平区百沙路203号"}
    # print(student.info.address)  # 北京市昌平区百沙路203号

    # 反向关联----> 从外键模型查询主模型
    student_info = StudentInfo.query.filter(StudentInfo.mobile == "15012345678").first()
    print(student_info.address)       # 北京市昌平区百沙路206号
    # 如果附加模型没有调用关联属性查询主模型对象，则ORM不会执行查询主模型的SQL语句
    print(student_info.student)       # {"id": 4, "name": "xiaolan04", "age": 17, "sex": 0, "email": "xiaolan04@qq.com", "money": 10000.0}
    print(student_info.student.name)  # xiaolan04
    print(student_info.sid)           # 4

    return "ok"


@app.route("/u1")
def u1():
    """修改数据"""
    # # 通过主模型使用关联属性修改附加模型的数据
    # student = Student.query.get(2)
    # student.info.address = "广州市天河区天河东路103号"
    # db.session.commit()

    # 也可以通过附加模型直接修改主模型的数据
    student_info = StudentInfo.query.filter(StudentInfo.mobile == "13312345678").first()
    # 如果要修改的数据，与数据表的没有改动，则不会执行更新的SQL语句
    student_info.student.age = 23
    db.session.commit()

    return "ok"


@app.route("/d1")
def d1():
    """删除数据"""

    # # 1. 如果删除主模型数据，则会先将附加模型的外键设置为null，然后才会删除主模型对象
    # student = Student.query.get(4)
    # db.session.delete(student)
    # db.session.commit()

    # 2. 如果删除附加模型数据，则直接删除，不会修改主模型数据
    StudentInfo.query.filter(StudentInfo.mobile == "13312345678").delete()
    db.session.commit()
    return "ok"


if __name__ == '__main__':
    app.run()
```



##### 在外键模型中声明关联属性

代码：

```python
import json
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.orm import backref

db = SQLAlchemy()
app = Flask(__name__, template_folder="templates", static_folder="static")

# 配置
app.config.update({
    "DEBUG": True,
    "SQLALCHEMY_DATABASE_URI": "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4",
    # 如果使用pymysql，则需要在连接时指定pymysql
    # "SQLALCHEMY_DATABASE_URI": "mysql+pymysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
    # 动态追踪修改设置，如未设置只会提示警告，设置False即可
    "SQLALCHEMY_TRACK_MODIFICATIONS": False,
    # ORM执行SQL查询时是哦否显示原始SQL语句，debug模式下可以开启
    "SQLALCHEMY_ECHO": True,
})

db.init_app(app)


class Student(db.Model):
    """学生管理"""
    __tablename__ = "tb_student"  # 表名
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(15), index=True, comment="姓名")
    age = db.Column(db.SmallInteger, comment="年龄")
    sex = db.Column(db.SmallInteger, comment="性别")
    email = db.Column(db.String(255), unique=True, comment="邮箱地址")
    money = db.Column(db.Numeric(10, 2), default=0.0, comment="钱包")

    def __repr__(self):
        return json.dumps(self.__to_dict__, ensure_ascii=False)

    @property
    def __to_dict__(self):
        return {
            "id": self.id,
            "name": self.name,
            "age": self.age,
            "sex": self.sex,
            "email": self.email,
            "money": float(self.money),
        }


class StudentInfo(db.Model):
    __tablename__ = "tb_student_info"  # 表名
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    # 附加表的外键，就是主表的主键
    # 如果是一对一，则外键放在附加表对应的模型中
    # 如果是一对多，则外键放在多的附加表对应的模型中
    sid = db.Column(db.Integer, db.ForeignKey("tb_student.id"), comment="外键")  # mysql字段
    # 模型的关联属性，不会在数据表创建字段
    # 因为StudentInfo和Student是一对一的关系，所以uselist=False表示关联一个数据
    student = db.relationship("Student", uselist=False, backref=backref("info", uselist=False))  # ORM关联属性
    mobile = db.Column(db.String(15), index=True, comment="手机号码")
    address = db.Column(db.String(255), nullable=True, comment="家庭地址")

    def __repr__(self):
        return json.dumps(self.__to_dict__, ensure_ascii=False)

    @property
    def __to_dict__(self):
        return {
            "id": self.id,
            "sid": self.sid,
            "mobile": self.mobile,
            "address": self.address,
        }


@app.route("/create")
def create_table():
    db.create_all()  # 为项目中被识别的所有模型创建数据表
    return "ok"


@app.route("/drop")
def drop_table():
    db.drop_all()  # 为项目中被识别的所有模型删除数据表
    return "ok"


@app.route("/a1")
def a1():
    """添加操作"""
    # 添加主表信息的时候通过关联属性db.relationship同步添加附件表信息
    student = Student(
        name="xiaolan01",
        age=16,
        sex=False,
        money=10000,
        email="xiaolan01@qq.com",
        info=StudentInfo(
            address="北京市昌平区百沙路203号",
            mobile="13312345672"
        )
    )

    db.session.add(student)
    db.session.commit()

    return "ok"


@app.route("/a2")
def a2():
    """添加操作"""
    # 假设已经存在主表信息，后面再补充附加表信息
    # student = Student(
    #     name="xiaolan03",
    #     age=16,
    #     sex=False,
    #     money=10000,
    #     email="xiaolan03@qq.com",
    # )
    #
    # db.session.add(student)
    # db.session.commit()

    # 上面代码先执行了，现有1个学生没有附加信息的，我们可以在后续代码通过查询主表的主键，补充附加表数据
    student = Student.query.get(2)
    if student and not student.info:
        """添加附加表数据"""
        """方式1"""
        # student.info = StudentInfo(mobile=13300010002, address="北京市昌平区百沙路205号",)
        # db.session.commit()

        """方式2"""
        # info = StudentInfo(
        #     student=student,  # 关联属性
        #     mobile=13300010002,
        #     address="北京市昌平区百沙路205号",
        # )
        #
        # db.session.add(info)
        # db.session.commit()

        """方式3"""
        info = StudentInfo(
            sid=student.id,  # 外键
            mobile=13300010002,
            address="北京市昌平区百沙路205号",
        )

        db.session.add(info)
        db.session.commit()

    return "ok"


@app.route("/a3")
def a3():
    """添加操作"""
    # 添加附加模型数据的同时，把主模型也进行添加
    info = StudentInfo(
        mobile=13300010003,
        address="北京市昌平区百沙路206号",
        student=Student(
            name="xiaolan04",
            age=17,
            sex=False,
            money=10000,
            email="xiaolan04@qq.com",
        ),
    )

    db.session.add(info)
    db.session.commit()

    return "ok"


@app.route("/g1")
def g1():
    """查询操作"""
    # # 正向关联----> 从主模型查询外键模型
    # student = Student.query.get(2)
    # print(student.name)          # xiaolan01
    # # 如果主模型没有调用关联属性查询附加模型对象，则ORM不会执行查询关联模型的SQL语句
    # print(student.info)          # {"id": 2, "sid": 2, "mobile": "13312345672", "address": "北京市昌平区百沙路203号"}
    # print(student.info.address)  # 北京市昌平区百沙路203号

    # 反向关联----> 从外键模型查询主模型
    student_info = StudentInfo.query.filter(StudentInfo.mobile == "13300010003").first()
    print(student_info.address)  # 北京市昌平区百沙路206号
    # 如果附加模型没有调用关联属性查询主模型对象，则ORM不会执行查询主模型的SQL语句
    print(
        student_info.student)  # {"id": 3, "name": "xiaolan04", "age": 17, "sex": 0, "email": "xiaolan04@qq.com", "money": 10000.0}
    print(student_info.student.name)  # xiaolan04
    print(student_info.sid)  # 3

    return "ok"


@app.route("/u1")
def u1():
    """修改数据"""
    # 通过主模型使用关联属性修改附加模型的数据
    student = Student.query.get(2)
    student.info.address = "广州市天河区天河东路103号"
    db.session.commit()

    # # 也可以通过附加模型直接修改主模型的数据
    # student_info = StudentInfo.query.filter(StudentInfo.mobile == "13312345672").first()
    # # 如果要修改的数据，与数据表的没有改动，则不会执行更新的SQL语句
    # student_info.student.age = 23
    # db.session.commit()

    return "ok"


@app.route("/d1")
def d1():
    """删除数据"""

    # # 1. 如果删除主模型数据，则会先将附加模型的外键设置为null，然后才会删除主模型对象
    student = Student.query.get(2)
    db.session.delete(student)
    db.session.commit()

    # 2. 如果删除附加模型数据，则直接删除，不会修改主模型数据
    # StudentInfo.query.filter(StudentInfo.mobile == "13312345672").delete()
    # db.session.commit()
    return "ok"


if __name__ == '__main__':
    app.run()

```



#### 一对多

常见业务：商品分类和商品、文章分类和商品、班级与学生、部分与员工、角色与会员、订单与订单详情、用户与收货地址。。。

```python
class User(db.Model):
	...
    # 关联属性，一的一方添加模型关联属性
    address_list = db.relationship("UserAddress", uselist=True, backref="user", lazy='dynamic')
   
class UsertAddress(db.Model):
	...
    # 外键，多的一方模型中添加外间
    user_id = db.Column(db.ForeignKey(User.id))
```

- 其中realtionship描述了Student和StudentAddress的关系。第1个参数为对应参照的类"StudentAddress"
- 第3个参数backref为类StudentAddress声明关联属性
- 第4个参数lazy决定了什么时候SQLALchemy什么时候执行读取关联模型的SQL语句
  - lazy='subquery'，查询当前数据模型时，采用子查询(subquery)，把外键模型的属性也同时查询出来了。
  - lazy=True或lazy='select'，查询当前数据模型时，不会把外键模型的数据查询出来，只有操作到外键关联属性时，才进行连表查询数据[执行SQL]
  - lazy='dynamic'，查询当前数据模型时，不会把外键模型的数据立刻查询出来，只有操作到外键关联属性并操作外键模型具体字段时，才进行连表查询数据[执行SQL]
- 常用的lazy选项：dynamic和select

课堂代码：

manage.py，代码：

```python
import json
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.orm import backref

db = SQLAlchemy()
app = Flask(__name__, template_folder="templates", static_folder="static")

# 配置
app.config.update({
    "DEBUG": True,
    "SQLALCHEMY_DATABASE_URI": "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4",
    # 如果使用pymysql，则需要在连接时指定pymysql
    # "SQLALCHEMY_DATABASE_URI": "mysql+pymysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
    # 动态追踪修改设置，如未设置只会提示警告，设置False即可
    "SQLALCHEMY_TRACK_MODIFICATIONS": False,
    # ORM执行SQL查询时是哦否显示原始SQL语句，debug模式下可以开启
    "SQLALCHEMY_ECHO": True,
})

db.init_app(app)


class User(db.Model):
    __tablename__ = "tb_user"
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    username = db.Column(db.String(50), unique=True, comment="用户名")
    nickname = db.Column(db.String(50), index=True, comment="昵称")
    sex = db.Column(db.Boolean, default=True, comment="性别")
    money = db.Column(db.Numeric(8,2), default=0.0, comment="钱包余额")
    address_list = db.relationship("UserAddress", uselist=True, backref="user", lazy="dynamic")

    def __repr__(self):
        return json.dumps(self.__to_dict__, ensure_ascii=False)

    @property
    def __to_dict__(self):
        return {
            "id": self.id,
            "username": self.username,
            "nickname": self.nickname,
            "sex": self.sex,
            "money": float(self.money),
        }


class UserAddress(db.Model):
    __tablename__ = "tb_user_address"
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(50), default="默认", comment="名称")
    province = db.Column(db.String(50), comment="省份")
    city = db.Column(db.String(50), comment="城市")
    area = db.Column(db.String(50), comment="地区")
    address = db.Column(db.String(500), comment="详细地址")
    mobile = db.Column(db.String(15), comment="收货人电话")
    user_id = db.Column(db.Integer, db.ForeignKey("tb_user.id"), comment="外键")

    def __repr__(self):
        return json.dumps(self.__to_dict__, ensure_ascii=False)

    @property
    def __to_dict__(self):
        return {
            "id": self.id,
            "name": self.name,
            "province": self.province,
            "city": self.city,
            "area": self.area,
            "address": self.address,
            "mobile": self.mobile,
            "user_id": self.user_id,
        }


@app.route("/create")
def create_table():
    db.create_all()  # 为项目中被识别的所有模型创建数据表
    return "ok"


@app.route("/drop")
def drop_table():
    db.drop_all()  # 为项目中被识别的所有模型删除数据表
    return "ok"


@app.route("/")
def index():
    return "ok"


@app.route("/a1")
def a1():
    """添加数据"""
    # # 添加主模型的同时也会给附加模型添加数据，这种情况，附加模型的值可以以列表形式进行添加，一次性添加多个附加模型数据进去。
    # user = User(
    #     username="xiaolan001",
    #     nickname="xiaolan001",
    #     sex=False,
    #     money=10000,
    #     address_list=[
    #         UserAddress(name="公司", province="北京市", city="北京市", area="昌平区", address="百沙路201", mobile="13012345678"),
    #         UserAddress(name="门口小卖部", province="北京市", city="北京市", area="昌平区", address="百沙路202", mobile="13012345677"),
    #         UserAddress(name="小区门口", province="北京市", city="北京市", area="昌平区", address="百沙路203", mobile="13012345676"),
    #     ]
    # )
    # db.session.add(user)
    # db.session.commit()

    # 添加外键模型数据的同时，添加主模型数据
    address = UserAddress(
        province="天津市",
        city="天津市",
        area="静海区",
        address="静安路1103号",
        user=User(
            username="xiaolan02",
            nickname="xiaolan02",
            money=10000,
            sex=False,
        )
    )
    db.session.add(address)
    db.session.commit()

    return "ok"


@app.route("/q1")
def q1():
    # 正向关联----> 从主模型查询外键模型
    student = User.query.filter(User.username == "xiaolan001").first()


    # 观察连表SQL语句的执行
    """ lazy="select" || lazy=True """
    # print(student) # 没有查询附加表
    # print(student.address_list) # 查询了附加表

    """ lazy="subquery" """
    # print(student)  # 已经查询了附加表，如果没有使用到外键模型的数据，则本次查询存在资源浪费！！

    """ lazy="dynamic" """
    # print(student)  # 没有查询附加表
    # student.address_list  # 没有查询附加模型的SQL语句
    # print( student.address_list ) # 只要访问关联模型的具体字段才真正执行，如果海量数量查询，则本次查询会影响返回数据给客户端的时间


    print( student.address_list[0] ) # 获取返回列表的第1个成员


    return "ok"

if __name__ == '__main__':
    app.run()
```



#### 多对多

```python
# 关系表[这种表，无法提供给python进行操作的，仅仅用于在数据库中记录两个模型之间的关系]
# 关系表[这种表，无法提供给python进行操作的，仅仅用于在数据库中记录两个模型之间的关系]
student_and_course = db.Table(
    "table_student_course",
    db.Column("id", db.Integer, primary_key=True, comment="主键ID"),
    db.Column("sid", db.Integer, db.ForeignKey("table_student.id"), comment="学生"),
    db.Column("cid", db.Integer, db.ForeignKey("table_course.id"), comment="课程"),
    db.Column("created_time", db.DateTime, default=datetime.now, comment="购买时间"), # 当前字段无法操作
)

class Student(db.Model):
    id = db.Column(db.Integer, primary_key=True,comment="主键")
    ...
    course_list = db.relationship("Course", secondary=student_and_course, backref="student_list", lazy="dynamic")

class Course(db.Model):
    ...
    
# 关系模型，[关系模型和关系表，任选其一]
class Achievement(db.Model):
    ...
```



##### 基于第三方关系表构建多对多

代码：

```python
import json
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.orm import backref
from datetime import datetime

db = SQLAlchemy()
app = Flask(__name__, template_folder="templates", static_folder="static")

# 配置
app.config.update({
    "DEBUG": True,
    "SQLALCHEMY_DATABASE_URI": "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4",
    # 如果使用pymysql，则需要在连接时指定pymysql
    # "SQLALCHEMY_DATABASE_URI": "mysql+pymysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
    # 动态追踪修改设置，如未设置只会提示警告，设置False即可
    "SQLALCHEMY_TRACK_MODIFICATIONS": False,
    # ORM执行SQL查询时是哦否显示原始SQL语句，debug模式下可以开启
    "SQLALCHEMY_ECHO": True,
})

db.init_app(app)

# 关系表[这种表，无法提供给python进行操作的，仅仅用于在数据库中记录两个模型之间的关系]
student_and_course = db.Table(
    "table_student_course",
    db.Column("id", db.Integer, primary_key=True, comment="主键ID"),
    db.Column("sid", db.Integer, db.ForeignKey("table_student.id"), comment="学生"),
    db.Column("cid", db.Integer, db.ForeignKey("table_course.id"), comment="课程"),
    db.Column("created_time", db.DateTime, default=datetime.now, comment="购买时间"), # 当前字段无法操作
)


class Student(db.Model):
    """学生信息模型"""
    # 声明与当前模型绑定的数据表名称
    __tablename__ = "table_student"
    id = db.Column(db.Integer, primary_key=True,comment="主键")
    name = db.Column(db.String(15), comment="姓名")
    age = db.Column(db.SmallInteger, comment="年龄")
    sex = db.Column(db.Boolean, default=True, comment="性别")
    email = db.Column(db.String(128), comment="邮箱地址")
    money = db.Column(db.Numeric(10, 2), default=0.0, comment="钱包")
    course_list = db.relationship("Course", secondary=student_and_course, backref="student_list", lazy="dynamic")

    def __repr__(self):
        return json.dumps(self.__to_dict__, ensure_ascii=False)

    @property
    def __to_dict__(self):
        return {
            "id": self.id,
            "name": self.name,
            "age": self.age,
            "sex": self.sex,
            "email": self.email,
            "money": float(self.money),
        }


class Course(db.Model):
    """课程数据模型"""
    __tablename__ = "table_course"
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="课程")
    price = db.Column(db.Numeric(7, 2), default=0.0, comment="价格")

    def __repr__(self):
        return json.dumps(self.__to_dict__, ensure_ascii=False)

    @property
    def __to_dict__(self):
        return {
            "id": self.id,
            "name": self.name,
            "price": self.price,
        }


@app.route("/create")
def create_table():
    db.create_all()  # 为项目中被识别的所有模型创建数据表
    return "ok"


@app.route("/drop")
def drop_table():
    db.drop_all()  # 为项目中被识别的所有模型删除数据表
    return "ok"


@app.route("/")
def index():
    return "ok"


@app.route("/a1")
def a1():
    """添加数据"""
    """添加其中一个主模型数据时，同时绑定添加另外一个主模型的数据，这个过程中，关系表会自动写入2者的关系数据，绑定2个模型之间的关系"""
    # student = Student(
    #     name="xiaozhao",
    #     age=13,
    #     sex=False,
    #     money=30000,
    #     email="xiaozhao@qq.com",
    #     course_list=[
    #         Course(name="python入门", price=99.99),
    #         Course(name="python初级", price=199.99),
    #         Course(name="python进阶", price=299.99),
    #     ]
    # )
    # db.session.add(student)
    # db.session.commit()

    """在学生报读课程的基础上，新增报读课程。"""
    # student = Student(
    #     name="xiaohong",
    #     age=14,
    #     sex=False,
    #     money=30000,
    #     email="300000@qq.com",
    # )
    # db.session.add(student)
    # db.session.commit()

    # student = Student.query.filter(Student.name=="xiaohong").first()
    # # 直接采用python内置的list方法操作
    # student.course_list.append(Course.query.get(3)) # 新增已经存在的课程
    # student.course_list.append(Course(name="python顶级", price=399.99))  # 已有课程，并让当前学生报读该课程
    # db.session.commit()

    """添加学生报读课程的测试数据"""
    # student1 = Student.query.get(1)
    # course_list = Course.query.filter(Course.id.in_([1,2])).all()
    # student1.course_list.extend(course_list)
    # db.session.commit()

    student2 = Student.query.get(2)
    course_list = Course.query.filter(Course.id.in_([3,2])).all()
    student2.course_list.extend(course_list)
    db.session.commit()

    return "ok"


@app.route("/q1")
def q1():
    """查询操作"""
    # 查询ID=4的学生报读的课程列表
    # student = Student.query.get(2)
    # course_list = [{"name":item.name,"price":float(item.price)} for item in student.course_list]
    # print(course_list)

    # 查询出2号课程，都有那些学生在读？
    course = Course.query.get(2)
    student_list = [{"name":item.name,"age":item.age} for item in course.student_list]
    print(student_list)

    return "ok"

@app.route("/u1")
def u1():
    """更新数据"""
    # # 给报读了3号课程的同学，返现红包200块钱
    # course = Course.query.get(3)
    # for student in course.student_list:
    #     student.money+=200
    # db.session.commit()

    # db.Table的缺陷: 无法通过主模型直接操作db.Table中的外键之外的其他字段，例如：报读课程的时间
    course = Course.query.get(3)
    print(course.student_list)

    # 解决：在声明2个模型是多对多的关联关系时，把关联关系使用第三个模型来创建声明，
    # 就是不要使用db.Table改成模型来绑定关系，把模型的对多对拆分成2个1对多

    return "ok"



if __name__ == '__main__':
    app.run()

```

多对多，也可以拆解成3个模型（2个主模型，1个关系模型，关系模型保存了2个主模型的外键），其中tb_achievement作为单独模型存在。



##### 基于第三方关系模型构建多对多

在SQLAlchemy中，基于db.Table创建的关系表，如果需要新增除了外键以外其他字段，无法操作。所以将来实现多对多的时候，除了上面db.Table方案以外，还可以把关系表声明成模型的方法，如果声明成模型，则原来课程和学生之间的多对多的关系，就会变成远程的1对多了。

代码：

```python
import json
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.orm import backref
from datetime import datetime

db = SQLAlchemy()
app = Flask(__name__, template_folder="templates", static_folder="static")

# 配置
app.config.update({
    "DEBUG": True,
    "SQLALCHEMY_DATABASE_URI": "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4",
    # 如果使用pymysql，则需要在连接时指定pymysql
    # "SQLALCHEMY_DATABASE_URI": "mysql+pymysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
    # 动态追踪修改设置，如未设置只会提示警告，设置False即可
    "SQLALCHEMY_TRACK_MODIFICATIONS": False,
    # ORM执行SQL查询时是哦否显示原始SQL语句，debug模式下可以开启
    "SQLALCHEMY_ECHO": True,
})

db.init_app(app)

# 关系表[这种表，无法提供给python进行操作的，仅仅用于在数据库中记录两个模型之间的关系]
# student_and_course = db.Table(
#     "demo_student_course",
#     db.Column("id", db.Integer, primary_key=True, comment="主键ID"),
#     db.Column("sid", db.Integer, db.ForeignKey("demo_student.id"), comment="学生"),
#     db.Column("cid", db.Integer, db.ForeignKey("demo_course.id"), comment="课程"),
#     db.Column("created_time", db.DateTime, default=datetime.now, comment="购买时间"), # 当前字段无法操作
# )


class StudentCourse(db.Model):
    __tablename__ = "demo_student_course"
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    sid = db.Column(db.Integer, db.ForeignKey("demo_student.id"))
    cid = db.Column(db.Integer, db.ForeignKey("demo_course.id"))
    created_time = db.Column(db.DateTime, default=datetime.now, comment="购买时间")

    # 关联属性
    student = db.relationship("Student", uselist=False, backref=backref("to_relation", uselist=True))
    course = db.relationship("Course", uselist=False, backref=backref("to_relation", uselist=True))


class Student(db.Model):
    """学生信息模型"""
    # 声明与当前模型绑定的数据表名称
    __tablename__ = "demo_student"
    id = db.Column(db.Integer, primary_key=True,comment="主键")
    name = db.Column(db.String(15), comment="姓名")
    age = db.Column(db.SmallInteger, comment="年龄")
    sex = db.Column(db.Boolean, default=True, comment="性别")
    email = db.Column(db.String(128), comment="邮箱地址")
    money = db.Column(db.Numeric(10, 2), default=0.0, comment="钱包")

    def __repr__(self):
        return json.dumps(self.__to_dict__, ensure_ascii=False)

    @property
    def __to_dict__(self):
        return {
            "id": self.id,
            "name": self.name,
            "age": self.age,
            "sex": self.sex,
            "email": self.email,
            "money": float(self.money),
        }


class Course(db.Model):
    """课程数据模型"""
    __tablename__ = "demo_course"
    id = db.Column(db.Integer, primary_key=True, comment="主键")
    name = db.Column(db.String(64), unique=True, comment="课程")
    price = db.Column(db.Numeric(7, 2), default=0.0, comment="价格")

    def __repr__(self):
        return json.dumps(self.__to_dict__, ensure_ascii=False)

    @property
    def __to_dict__(self):
        return {
            "id": self.id,
            "name": self.name,
            "price": self.price,
        }


@app.route("/create")
def create_table():
    db.create_all()  # 为项目中被识别的所有模型创建数据表
    return "ok"


@app.route("/drop")
def drop_table():
    db.drop_all()  # 为项目中被识别的所有模型删除数据表
    return "ok"


@app.route("/")
def index():
    return "ok"


@app.route("/a1")
def a1():
    """添加数据"""
    """添加其中一个主模型数据时，同时绑定添加另外一个主模型的数据"""
    # student = Student(
    #     name="xiaozhao",
    #     age=13,
    #     sex=False,
    #     money=30000,
    #     email="xiaozhao@qq.com",
    #     to_relation=[
    #         StudentCourse(
    #             course=Course(name="python基础", price=99.9)
    #         )
    #     ]
    # )
    # db.session.add(student)
    # db.session.commit()

    """在学生报读课程的基础上，新增报读课程。"""
    # student = Student(
    #     name="xiaohong",
    #     age=14,
    #     sex=False,
    #     money=30000,
    #     email="300000@qq.com",
    # )
    # db.session.add(student)
    # db.session.commit()

    # student = Student.query.filter(Student.name == "xiaohong").first()
    # # 直接采用python内置的list方法操作
    # student.to_relation.extend([
    #     StudentCourse(
    #         course=Course.query.get(1) # 已经存在的课程，给学生报读
    #     ),
    #     StudentCourse(
    #         course=Course(name="python进阶", price=399.99)  # 新增课程，并让当前学生报读该课程
    #     )
    # ])
    #
    # db.session.commit()


    """添加学生报读课程的测试数据"""
    student1 = Student.query.get(1)
    course_list = Course.query.filter(Course.id.in_([1,2])).all()
    student1.to_relation.extend([StudentCourse(course=course) for course in course_list])
    db.session.commit()

    return "ok"


@app.route("/q1")
def q1():
    """查询操作"""
    # 查询ID=2的学生报读的课程列表
    # student = Student.query.get(2)
    # course_list = [{"name":item.course.name,"price":float(item.course.price)} for item in student.to_relation]
    # print(course_list)

    # 查询出2号课程，都有那些学生在读？
    course = Course.query.get(2)
    student_list = [{"name":item.student.name,"age":item.student.age} for item in course.to_relation]
    print(student_list)

    return "ok"

@app.route("/u1")
def u1():
    """更新数据"""
    # # 给报读了2号课程的同学，返现红包200块钱
    # course = Course.query.get(2)
    # for relation in course.to_relation:
    #     relation.student.money += 200
    # db.session.commit()

    # 获取中间的关系模型的字段
    course = Course.query.get(2)
    for relation in course.to_relation:
        print(relation.created_time)

    return "ok"



if __name__ == '__main__':
    app.run()

```

relationship还有一个设置外键级联的属性：cascade="all, delete, delete-orphan"

```
作业：
1. 我们现在学习的flask框架集成SQLAlchemy操作数据库使用的是flask-SQLAlchemy模块。如果原生的python下面应该如何使用SQLAlchemy进行初始化数据库连接和声明模型并实现模型的基本操作[增删查改，关联查询]？

2. flask中的SQLAlchemy如何进行自关联查询？ 这里自己写一个关于行政区划的自关联操作。
```

