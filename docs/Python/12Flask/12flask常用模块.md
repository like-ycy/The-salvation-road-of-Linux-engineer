# 常用模块

## Faker

文档: https://faker.readthedocs.io/en/master/locales/zh_CN.html

批量生成测试数据: https://github.com/joke2k/faker

```bash
pip install faker -i https://pypi.douban.com/simple
```

代码：

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

class Config(object):
    DEBUG = True
    # 数据库连接配置
    # SQLALCHEMY_DATABASE_URI = "数据库类型://数据库账号:密码@数据库地址:端口/数据库名称?charset=utf8mb4"
    SQLALCHEMY_DATABASE_URI = "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4"
    # 动态追踪修改设置，如未设置只会提示警告
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    # 查询时会显示原始SQL语句
    SQLALCHEMY_ECHO = False

app.config.from_object(Config)

db = SQLAlchemy(app=app)

class Student(db.Model):
    """学生信息模型"""
    __tablename__ = "db_student"
    id = db.Column(db.Integer, primary_key=True,comment="主键")
    name = db.Column(db.String(15), comment="姓名")
    age = db.Column(db.SmallInteger, comment="年龄")
    sex = db.Column(db.Boolean, default=True, comment="性别")
    email = db.Column(db.String(128), comment="邮箱地址")
    money = db.Column(db.Numeric(10,2), default=0.0, comment="钱包")

    def __repr__(self):
        return f"{self.name}<Student>"

# 自定义批量生成学生
import random,click
from faker import Faker
# 自定义终端命令
@app.cli.command("faker_user")     # 指定终端命令的调用名称
@click.argument("num", default=5, type=int)  # 命令的选项
def faker_user_command(num):
    """生成测试学生信息"""
    faker = Faker(locale="ZH_CN")
    for i in range(num):
        sex = bool( random.randint(0,1) )
        student = Student(
            name= faker.name_male() if sex else faker.name_female(),
            age=random.randint(15,60),
            sex=sex,
            email=faker.free_email(),
            money= float( random.randint(100,100000) / 100 ),
        )
        db.session.add(student)
    # 在循环外面统一提交
    db.session.commit()

@app.route("/")
def index():
    return "ok"

if __name__ == '__main__':
    with app.app_context():
        db.create_all()
    app.run()


"""
export FLASK_APP=manage.py
flask faker-user 10
"""
```



## flask-session

flask-session，允许设置session到指定的存储空间中，例如：redis/mongoDB/mysql。

文档: https://flask-session.readthedocs.io/en/latest/

```
pip install Flask-Session
```

使用session之前,必须配置一下配置项:

```python
# session秘钥
app.config["SECRET_KEY"] = "*(%#4sxcz(^(#$#8423"
```



### SQLAlchemy存储session的基本配置

需要手动创建session表，在项目第一次启动的时候，使用`db.create_all()`来完成创建。

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
# 引入session存储驱动类
from flask_session import Session
# 引入sessio操作类，注意：引入路径不同，大小写不同的。
from flask import session

app = Flask(__name__, template_folder="templates", static_folder="static")

db = SQLAlchemy()

# 实例化session存储类
session_store = Session()

# 配置
app.config.update({
    "DEBUG": True,
    # 使用session之前,必须配置一下秘钥
    "SECRET_KEY": "*(%#4sxcz(^(#$#8423",

    # 要把存储到SQLAlchemy，必须配置数据库连接
    # "SQLALCHEMY_DATABASE_URI": "数据库类型://数据库账号:密码@数据库地址:端口/数据库名称?charset=utf8mb4"
    "SQLALCHEMY_DATABASE_URI": "mysql://root:123@127.0.0.1:3306/flaskdemo?charset=utf8mb4",
    # 动态追踪修改设置，如未设置只会提示警告
    "SQLALCHEMY_TRACK_MODIFICATIONS": False,
    # 查询时会显示原始SQL语句
    "SQLALCHEMY_ECHO": False,

    # 把session通过SQLAlchmey保存到mysql中
    "SESSION_TYPE": "sqlalchemy",  # session类型为sqlalchemy
    "SESSION_SQLALCHEMY": db,  # SQLAlchemy的数据库连接对象
    "SESSION_SQLALCHEMY_TABLE": 'db_session',  # session要保存的表名称
    "SESSION_PERMANENT": True,    # 如果设置为True，则关闭浏览器session就失效
    "SESSION_USE_SIGNER": True,  # 是否对发送到浏览器上session的cookie值进行添加签名，防止串改。
    "SESSION_KEY_PREFIX": "session:"  # session数据表中sessionID的前缀，默认就是 session:
})

db.init_app(app)

# 务必保证在数据库配置初始化以后才进行session存储类的初始化
session_store.init_app(app)


# 如果要把session保存到数据库中，则必须先执行db.create_all() 让数据库提前创建session表。否则使用session时报错。
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

@app.route("/set")
def set_session():
    session["uname"] = "xiaoming"
    session["age"] = 18
    return "ok"

@app.route("/get")
def get_session():
    print(session.get("uname"))
    print(session.get("age"))
    return "ok"

@app.route("/del")
def del_session():
    # 此处的删除，不是删除用户对应的session表记录，而是删除session值而已。
    print(session.pop("uname"))
    print(session.pop("age"))
    return "ok"


if __name__ == '__main__':
    app.run()

```



### redis保存session的基本配置

这个功能必须确保，服务器必须已经安装了redis而且当前项目虚拟环境中已经安装了redis扩展库

```bash
pip install flask-redis -i https://pypi.douban.com/simple
```

flask-redis是第三方开发者为了方便我们在flask框架中集成redis数据库操作所封装一个redis操作库、

在flask中要基于flask-redis进行数据库则可以完成以下3个步骤即可：

```bash
from flask import Flask
from flask_redis import FlaskRedis

# 实例化
app = Flask(__name__)
session_redis = FlaskRedis(config_prefix="SESSION")
user_redis = FlaskRedis(config_prefix="USER")
order_redis = FlaskRedis(config_prefix="ORDER")

# 初始化 flask_redis
session_redis.init_app(app)
user_redis.init_app(app)
order_redis.init_app(app)

@app.route("/")
def q2():
    user_redis.setnx("doing", 100)
    return "ok"

if __name__ == '__main__':

    app.run(host="0.0.0.0", port=5000, debug=True)
```



在redis中保存session，代码：

```python
from flask import Flask
from flask_redis import FlaskRedis
from flask_session import Session
from flask import session

app = Flask(__name__, template_folder="templates", static_folder="static")

redis0 = FlaskRedis(config_prefix="REDIS")
redis1 = FlaskRedis(config_prefix="SESSION_REDIS")
session_store = Session()
# 配置
app.config.update({
    "DEBUG": True,
    # 使用session之前,必须配置一下秘钥
    "SECRET_KEY": "*(%#4sxcz(^(#$#8423",
    "REDIS_URL": "redis://:123456@127.0.0.1:6379/0",
    "SESSION_REDIS_URL": "redis://:123456@127.0.0.1:6379/1",

    # 把session保存到redis中
    "SESSION_TYPE": "redis",  # session类型为sqlalchemy
    "SESSION_PERMANENT": True,  # 如果设置为True，则关闭浏览器session就失效
    "SESSION_USE_SIGNER": True,  # 是否对发送到浏览器上session的cookie值进行添加签名，防止串改。
    "SESSION_KEY_PREFIX": "session:",  # session数据表中sessionID的前缀，默认就是 session:
    # session保存数据到redis时启用的链接对象
    "SESSION_REDIS": redis1,      # 用于连接redis的配置
})


redis0.init_app(app)
redis1.init_app(app)

# 务必保证session存储类初始化之前，redis已经完成初始化了。
session_store.init_app(app)

@app.route("/")
def index():
    return "ok"


@app.route("/set")
def set_session():
    session["uname"] = "xiaoming"
    session["age"] = 18
    return "ok"


@app.route("/get")
def get_session():
    print(session.get("uname"))
    print(session.get("age"))
    return "ok"


@app.route("/del")
def del_session():
    # 此处的删除，是直接删除保存在redis中的数据，当所有session被删除，则key也会消失了。
    print(session.pop("uname"))
    print(session.pop("age"))
    return "ok"


if __name__ == '__main__':
    app.run()

```

