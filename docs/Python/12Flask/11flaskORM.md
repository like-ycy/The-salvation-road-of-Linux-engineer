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

