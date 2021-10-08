# Django 模型

## 前言

我们在使用Django框架开发web应用的过程中，不可避免地会涉及到数据的管理操作（如增、删、改、查），而一旦谈到数据的管理操作，就需要用到数据库管理软件，例如mysql、oracle、Microsoft SQL Server等。

 如果应用程序需要操作数据（比如将用户注册信息永久存放起来），那么我们需要在应用程序中编写原生sql语句，然后使用pymysql模块远程操作mysql数据库，详见图1

但是直接编写原生sql语句会存在两方面的问题，严重影响开发效率，如下：

1. sql语句的执行效率：应用开发程序员需要耗费一大部分精力去优化sql语句
2. 数据库迁移：针对mysql开发的sql语句无法直接应用到oracle数据库上，一旦需要迁移数据库，便需要考虑跨平台问题

## ORM

为了解决上述问题，django引入了ORM的概念，ORM全称Object Relational Mapping，即对象关系映射，是在pymysq之上又进行了一层封装，对于数据的操作，我们无需再去编写原生sql，取代代之的是基于面向对象的思想去编写类、对象、调用相应的方法等，ORM会将其转换/映射成原生SQL然后交给pymysql执行。

如此，开发人员既不用再去考虑原生SQL的优化问题，也不用考虑数据库迁移的问题，ORM都帮我们做了优化且支持多种数据库，这极大地提升了我们的开发效率，下面就让我们来详细学习ORM的使用吧

映射关系：

| 类           | 数据库中的表           |
| :----------- | :--------------------- |
| 对象         | 表单记录               |
| 对象获取属性 | 记录的某个字段对应的值 |

**优点:** 能够让一个不会数据库操作的人 也能够简单快捷去使用数据库

**缺点:** 由于封装程度太高 可能会导致程序的执行效率偏低，有时候 结合项目需求 可能需要你手写sql语句

!> 注意事项：django的orm不会自动帮你创建库，库需要你自己手动创建，表会自动帮你创建，你只需要书写符合django orm语法的代码即可，去应用下所在的models.py中书写类



## ORM常用字段

### 1、字段类型

| 字段名称         | 字段说明                                                     | 参数                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| AutoField        | 一个根据实际id进行自动增长的inter field（默认创建的）        | primary_key = True                                           |
| CharField        | varchar类型字段                                              | max_length 存储值最大长度                                    |
| TextField        | longtext 长文本                                              |                                                              |
| IntegerField     | int类型字段 存储整形                                         |                                                              |
| SmallInterField  | 小整数                                                       | -32768-32767                                                 |
| DecimalField     | 存储浮点型 更加精准                                          | max_digits=None 位数长度decimal_places=None 小数的位数       |
| FloatField       | 浮点型                                                       |                                                              |
| BooleanField     | 存储bool值 True/False                                        |                                                              |
| NullBooleanField | 存储Null/True/False                                          |                                                              |
| DateField        | date字段                                                     | auto_now=False 如果对数据进行修改则会自动保存修改时间 <br/>auto_now_add=False 会自动添加第一次保存的时间  俩个不同同时设定 |
| TimeField        | time字段                                                     | 同上                                                         |
| DateTimeField    | datetimefield字段                                            | 同上                                                         |
| FileField        | 文件上传                                                     | upload_to = '' 文件上传保存的路径                            |
| ImageField       | 图片上传字段（继承了FileField 对图像进行了验证 确保是一个有效的图片） | 同上                                                         |

### 2、字段选项

通过字段选项，可以实现对字段约束，在字段对象使用关键字参数进行指定

| 选项        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| null        | 如果设置为True  则当前字段可以为null                         |
| blank       | 如果设置为True 则当前字段可以什么值都没有 设置在字符串类型的字段上 |
| db_column   | 设置字段名称  如果不设置 默认为属性名                        |
| db_index    | 设置为True  常规索引                                         |
| unique      | 设置为True  唯一索引                                         |
| primary_key | 设置为True   主键索引                                        |
| default     | 默认值                                                       |

## 一、创建模型

### 1、准备工作：数据库配置

+ 默认配置

  ```python
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.sqlite3',
          'NAME': BASE_DIR / 'db.sqlite3',
      }
  }
  ```

+ 配置连接MySQL数据库

  + 安装mysql扩展库

    ```
    pip install pymysql
    ```

  - project/init.py

    ```
    import pymysql
    pymysql.install_as_MySQLdb()
    ```

  settings.py

  ```python
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': 'day40django',
          'HOST': '127.0.0.1',
          'PORT': '3306',
          'USER': 'root',
          'PASSWORD': '123456',

          'OPTIONS': {
              "init_command": "SET storage_engine=INNODB", #设置创建表的存储引擎为INNODB
          }
      }

  }
  ```

##### 如果想打印orm转换过程中的sql，需要在settings中进行配置日志：

```python
CopyLOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console':{
            'level':'DEBUG',
            'class':'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'propagate': True,
            'level':'DEBUG',
        },
    }
}
```

### 2、表分析

+ 需求

  创建用户表

+ 所需字段

  id  用户名   密码    性别  年龄 个人简介 头像 是否删除  注册时间

### 3、创建模型类

models.py

```python
from django.db import models

# Create your models here.
class User(models.Model): # 必须继承models.Model的子类
    # 用户名
    username = models.CharField(max_length=12)
    # 密码
    userpass = models.CharField(max_length=64)
    # 性别
    sex = models.BooleanField(default=True)
    # 年龄
    age = models.SmallIntegerField(default=18)
    # 个人简介
    info = models.CharField(max_length=200, default='个人简介')
    # 头像
    icon = models.CharField(max_length=60, default='default.jpg')
    # 是否删除
    isDelete = models.BooleanField(default=False)
    # 注册时间
    registerTime = models.DateTimeField(auto_now_add=True)
```

### 4、迁移到数据库中

+ 生成迁移文件

  python manage.py makemigrations

+ 迁移到数据库中

  python manage.py migrate

!> 数据库中生成的表名默认为：应用名\_类名 （全部小写）

> 注意：
> 1、makemigrations只是生成一个数据库迁移记录的文件，而migrate才是将更改真正提交到数据库执行
> 2、数据库迁移记录的文件存放于app下的migrations文件夹里
> 3、了解：使用命令python manage.py showmigrations可以查看没有执行migrate的文件

### 5、元选项

+ 作用

  在模型类中使用Meta类设置一些信息

+ 实例

  ```python
  class User(models.Model):
  	...
      registerTime = models.DateTimeField(auto_now_add=True)
      class Meta:
          db_table = 'user'  # 更改表名称为user
          # ordering = ['-age', 'id']  # 按照年龄降序 如果年龄相同 id升序
          ordering = ['-age']  # 按照年龄降序
  ```

### 6、objects

+ 概念

  + 是Manager类的对象，用于与数据库进行交互
  + 如果定义模型时没有指定objects，则会默认创建objects

+ 问题

  在查询的时候，好多地方都使用了同一个过滤器，可不可以把过滤器放到objects中，对原始查询集进行过滤

+ 自定义管理器对象

  models.py

  ```python
  from django.db import models


  # 继承模型管理器类
  class MyManager(models.Manager):
      # 重写查询方法
      def get_queryset(self):
          # 重写了父类中的get_queryset方法  并增加了过滤功能 过滤掉isdelete为True的数据
          return super().get_queryset().filter(isDelete=False)
          # return super().get_queryset()


  # Create your models here.
  class User(models.Model):
      # 自定义模型管理器对象
      objects = models.Manager()
      o = models.Manager()   # 可以有多个模型管理器
      my_obj = MyManager()  # 自定义添加过滤功能的模型管理器
      ...
      # 注册时间
      registerTime = models.DateTimeField(auto_now_add=True)
      def __str__(self):
          return self.username
      class Meta:
          db_table = 'user'  # 更改表名称为user
          # ordering = ['-age', 'id']
          # ordering = ['-age']  # 按照年龄降序
  ```

  views.py

  ```python
  def select(req):
      # 使用自定义模型管理器
      # u = User.objects.all()
      # u = User.o.all()

      # 使用自定义模型管理器并重写了查询方法
      # u = User.my_obj.all()  # 等同于  u = User.objects.filter(isDelete=False)

      u = User.objects.filter(isDelete=False)
      print(u.query)
      for o in u:
          # print(o.username,'-------',o.userpass)
          print(o.isDelete)
      # u = User.objects
      # print(u)
      # print(type(u))
      return HttpResponse('数据查询')
  ```



## 二、数据添加

### 1、创建对象

+ 概述

  当创建对象以后，使用属性进行赋值；当调用save以后，才与数据库进行交互

+ 实例

  ```python
  def save(req):
      u = User()
      u.username = 'lucky'
      u.userpass = '123456'
      u.sex = True
      u.age = 18
      u.info = 'lucky的个人简介'
      u.icon = './lucky.jpg'
      u.isDelete = False
      u.save()
      return HttpResponse('数据添加')
  ```

### 2、实例化对象并传参

+ 概述

  当模型类实例化以后进行参数传递

+ 实例

  ```python
  #数据添加第二种方法
  u = User(3, '张三','123456')
  u = User(username='李四', userpass='123456')
  u.save()
  return HttpResponse('数据添加')
  ```

+ 注意

  如果不指定参数，id也需要传参

### 3、create方法

+ 使用create方法

  ```python
  # 第三种数据添加方法
  User.objects.create(username='王五', userpass='123456')
  return HttpResponse('数据添加')
  ```

+ 重写create方法

  models.py

  ```python
  class MyManager(models.Manager):
      def get_queryset(self):
          # 重写了父类中的get_queryset方法  并增加了过滤功能 过滤掉isdelete为True的数据
          return super().get_queryset().filter(isDelete=False)
          # return super().get_queryset()

      # 重写create方法
      def create(self, username='', userpass='', sex=True,age=18,info='lucky的个人简介', icon='./lucky.jpg', isDelete=False):
          u = self.model()
          u.username = username
          u.userpass = userpass
          u.sex = sex
          u.age = age
          u.info = info
          u.icon = icon
          u.isDelete = isDelete
          return u


  # Create your models here.
  class User(models.Model):
      # 自定义模型管理器对象
      objects = models.Manager()
      # o = models.Manager()
      my_obj = MyManager()
      # 用户名
      ...
  ```

  views.py

  ```python
  u = User.my_obj.create('赵六', '123456')
  u.save()
  ```

### 4、添加多条数据 bulk_create()

+ 概述

  批量添加数据

  ```python
  u1 = User(username='秦七', userpass='123456')
  u2 = User(username='王老八', userpass='123456')
  User.objects.bulk_create([u1, u2])
  ```

### 5、类方法

+ 概述

  通过classmethod方法进行数据添加

+ 实例

  models.py

  ```python
  # Create your models here.
  class User(models.Model):
      # 自定义模型管理器对象
      objects = models.Manager()
      # o = models.Manager()
      my_obj = MyManager()
  	...
      # 类方法用于添加数据
      @classmethod
      def create(cls, username='', userpass='', sex=True,age=18,info='lucky的个人简介', icon='./lucky.jpg', isDelete=False):
          return cls(username=username, userpass=userpass, sex=sex,age=age,info=info, icon=icon, isDelete=isDelete)
  ```

  views.py

  ```python
  # 第五种 使用类方法
  User.create('婷婷', '123456').save()
  return HttpResponse('数据添加')
  ```

## 三、查询

### 1、数据查询解析

+ 查询集

  从数据库中获取数据查询的集合

+ 原始查询集

  调用get_queryset() 方法查询获取到的数据为原始查询集，不作为数据的提取

+ 数据查询集

  通过原始查询集配合过滤器等最终得到的数据

+ 过滤器

  对数据进行筛选

### 2、返回查询集的过滤器

+ all()

  概述： 返回所有

  ```python
  def show(req):
      u = User.objects.all()
      print(u)
      return render(req, 'show.html', {'data': u})
  ```

+ filter()

  过滤掉不符合条件的数据

  ```python
  # 查询没有删除的数据和性别为False的数据
  u = User.objects.filter(isDelete=False).filter(sex=False)
  u = User.objects.filter(isDelete=False, sex=False)
  ```

+ exclude()

  过滤掉符合条件的数据

  ```python
  # 过滤掉没有删除的数据和性别为False的数据
  u = User.objects.exclude(isDelete=False, sex=False)
  ```

+ order_by()

  排序

  + '字段'  升序
  + '-字段'  降序

  ```python
  u = User.objects.order_by('-age')
  u = User.objects.order_by('age' ,'-id')
  ```

+ values()

  返回一个列表，每条数据是一个字典

  + values()  返回所有
  + values(field1,[field2....])  返回指定字段的值

  ```
  u = User.objects.values()
  u = User.objects.values('username')
  ```

+ extra()

  起别名

  extra(select={'new': 'old'})

  ```python
  u = User.objects.all().extra(select={'name': 'username'})
  print(u[0].name)
  ```

