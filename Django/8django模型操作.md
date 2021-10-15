# Django 模型操作

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
  
  class MyManager(models.Manager):
      # 重写查询方法
      def get_queryset(self):
          # 重写了父类中的get_queryset方法  并增加了过滤功能 过滤掉isdelete为True的数据
          return super().get_queryset().filter(isDelete=False)
          # return super().get_queryset()
         
  class User(models.Model):
      # 自定义模型管理器对象
      objects = models.Manager()
      o = models.Manager()   # 可以有多个模型管理器
      my_obj = MyManager()  # 自定义添加过滤功能的模型管理器
  
      # 注册时间
      registerTime = models.DateTimeField(auto_now_add=True)
      def __str__(self):
          return self.usernam
      class Meta:
          db_table = 'user'  # 更改表名称为user
          # ordering = ['-age', 'id']
          # ordering = ['-age']  # 按照年龄降序
  ```
  
  > 如果自定义了管理器类，那么将没有objects属性
  
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

  views.py

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
  u = User(3, '张三','123456') # 第一位的数字为数据库表的主键，必须给，否则报错
  
  u = User(username='李四', userpass='123456')
  u.save()
  return HttpResponse('数据添加')
  ```

  !> 注意：使用位置参数添加数据时，首位必须添加主键id，否则报错


### 3、create方法

+ 使用create方法

  ```python
  # 第三种数据添加方法
  User.objects.create(username='王五', userpass='123456')
  return HttpResponse('数据添加')
  ```

  > 1、create方法不需要save方法保存数据
  >
  > 2、create 方法只能传入关键字参数，如果需要传入位置参数，可以重写create方法，见下例

+ 重写create方法，以可以使用位置参数进行传参

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

  查看符合条件的数据

  ```python
  # 查询没有删除的数据和性别为False的数据，以下两条代码结果没有任何区别
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

### 3、返回单个值的过滤器

- get()

  返回单个满足条件的对象

  ```python
  u = User.objects.get(age=20)
  u = User.objects.get(age=18)
  u = User.objects.get(age=180)
  return HttpResponse('show')
  ```

  !>  注意： 只能返回一个值，如果大于小于一个值，都会抛出异常（比如要查询24岁的，但是24岁的数据有多条，则会报错）

- count()

  返回当前查询集的数据条数

  ```
  u = User.objects.all().count()
  u = User.objects.filter(age=18).count()
  return HttpResponse('show')
  ```

- first()

  返回查询集中的第一个对象

  ```python
  u = User.objects.order_by('-age').first()
  return HttpResponse('show')
  ```

- last()

  ```python
  u = User.objects.order_by('-age').first()
  return HttpResponse('show')
  ```

- exists()

  判断筛选后的数据是否存在

  ```python
  u = User.objects.filter(age=18).exists()
  u = User.objects.filter(age=180).exists()
  ```

- values_list()

  将模型对象转换成列表，达到减少内存损耗 ，提高性能

  列表中的每一条数据为元组类型

  ```python
  u = User.objects.values_list()
  ```


### 4、字段查询

+ 概述

  实现where子句，filter、exclude等参数

+ 语法

  属性名\_\_比较运算符=值

+ 更改表结构，查询区分大小写

  alter table user modify username varchar(20) collate utf8_bin not null;

   alter table user modify username varchar(20) character set utf8 not null;

+ exact()

  表示判等，不区分大小写

  ```python
  u = User.objects.filter(username__exact='lucky')
  u = User.objects.filter(username='lucky')
  ```

+ contains

  包含  (区分大小写)

  ```python
  u = User.objects.filter(username__contains='l')
  u = User.objects.filter(username__contains='L')
  ```

  不区分大小写

  ```python
  u = User.objects.filter(username__icontains='l')
  ```

+ startswith

  以...开头  (区分大小写)

  ```python
  u = User.objects.filter(username__startswith='L')
  ```

  不区分大小写

  ```python
  u = User.objects.filter(username__istartswith='L')
  ```

+ endswith

  以...结尾  (区分大小写)

  ```python
  u = User.objects.filter(username__endswith='Y')
  ```

  不区分大小写

  ```python
  u = User.objects.filter(username__iendswith='Y')
  ```

+ null

  查询数据为null

  ```python
  # 查询为null的数据
  u = User.objects.filter(username__isnull=True)
  u = User.objects.filter(username=None)
  # 查询不为空的数据
  u = User.objects.filter(username__isnull=False)
  u = User.objects.exclude(username__isnull=True)
  ```

+ in

  是否包含范围内

  ```python
  # 包含范围内
  u = User.objects.filter(id__in=[1,2,3,4,5])
  u = User.objects.filter(pk__in=[1,2,3,4,5])
  # 不包含范围内
  u = User.objects.exclude(pk__in=[1,2,3,4,5])
  ```

+ 比较运算符

  + gt  大于
  + gte  大于等于
  + lt 小于
  + lte  小于等于

  ```python
  u = User.objects.filter(id__gt=10)
  u = User.objects.filter(id__gte=10)
  u = User.objects.filter(id__lt=10)
  u = User.objects.filter(id__lte=10)
  ```

### 5、聚合函数

+ 导入

  ```python
  from django.db.models import Avg,Max,Min,Sum,Count
  ```

+ 方法

  aggregate()

+ 实例

  ```python
  u = User.objects.aggregate(Avg('age'))
  u = User.objects.aggregate(Max('age'))
  u = User.objects.aggregate(Min('age'))
  u = User.objects.aggregate(Sum('age'))
  u = User.objects.aggregate(Count('age'))
  ```

### 6、限制结果数量all()

+ 和列表截取一样的操作

```python
u = User.objects.all()
print(u[0])
print(u[100])
print(u[0:10])
print(u[0:10:2])
```

## 五、修改与删除

### 1、修改

+ save()

  ```python
  def update(req):
      # 获取要修改数据的对象
      u1 = User.objects.get(pk=1)
      print(u1)
      u1.sex = False
      u1.age = 17
      u1.save()
      return HttpResponse('update')
  ```

+ update() 值的修改

  ```python
  # 修改多个
  u = User.objects.filter(id__in=[1,2,3])
  # print(u)
  u.update(age=17, sex=False)
  ```

+ 区别

  save适用于修改单个对象

  update适用于修改多个对象

### 2、删除

+ delete

  ```python
  u = User.objects.filter(id__in=[1,2,3])
  u.delete()
  ```

## 六、高级查询

### 1、Q对象

+ 导入

  ```
  from django.db.models import Q
  ```

+ 作用

  逻辑或

  ```python
  # 逻辑或
  u = User.objects.filter(Q(age=18)|Q(sex=True))
  # 逻辑与
  u = User.objects.filter(age=18, sex=True)
  u = User.objects.filter(age=18).filter(sex=True)
  ```

+ 逻辑非

  ```python
  u = User.objects.filter(~Q(age=18))
  u = User.objects.exclude(Q(age=18))
  ```

### 2、F对象

+ 导入

  ```python
  from django.db.models import F
  ```

+ 概述

  模型类中A与B字段之间的比较

  ```python
  # 比较俩个字段相等的值
  u = User.objects.filter(id=F('age'))
  # 比较id大于age字段的值
  u = User.objects.filter(id__gt=F('age'))
  ```

### 3、原生SQL操作

+ 格式

  类名.objects.raw(sql)

+ 实例

  ```python
  u = User.objects.raw('select * from user')
  u = User.objects.raw('select id, u_age, sex from user')
  ```

+ 注意

  查询某个字段时，包含主键

### 4、获取原生SQL

+ 格式

  对象.query

+ 示例

  ```python
  u = User.objects.filter(id__gt=F('age'))
  print(u)
  print(u.query)
  ```



### 5、关联查询

+ 作用

  俩个表关联

+ 语法

  关联模型类名小写`__属性名__` 比较运算符=值

+ 实例

  ```python
  s = Grade.objects.filter(students__sname__contains='lucky')
  ```


## 七、模型对应关系

### 1、概述

+ 主从表

  没有声明关系的表为主表

  声明关系的表为从表

+ 对应关系

  + 一对一
  + 一对多
  + 多对多

### 2、一对一模型关系 OneToOneField

+ 说明

  使用OneToOneField声明模型关系为一对一关系将OneToOneField放与从表中

+ 使用场景

  表的字段太多，进行拆分

+ 关系位置

  哪张表都可以

+ 创建模型类

  用户模型类

  ```python
  from django.db import models
  
  class User(models.Model):
      # 用户名
      username = models.CharField(max_length=12)
      # 密码
      userpass = models.CharField(max_length=64)
      # 性别
      sex = models.BooleanField(default=True)
      # 年龄
      age = models.SmallIntegerField(default=18, db_column='u_age')
      # 个人简介
      info = models.CharField(max_length=200, default='个人简介')
      # 头像
      icon = models.CharField(max_length=60, default='default.jpg')
      # 是否删除
      isDelete = models.BooleanField(default=False)
      # 注册时间
      registerTime = models.DateTimeField(auto_now_add=True)
  
      # 类方法用于添加数据
      def __str__(self):
          return self.username
      class Meta:
           db_table = 'user'  # 更改表名称为user
  ```

  身份证模型类

  ```python
  # 身份证
  class IdCard(models.Model):
      # 身份证号码
      num = models.CharField(max_length=18)
      name = models.CharField(max_length=10)
      sex = models.BooleanField(default=True)
      birth = models.DateTimeField(auto_now_add=True)
      address = models.CharField(max_length=50, default='地址')
      # 创建了模型一对第一的关系 从表随着主表的删除而删除
      # user = models.OneToOneField(User, on_delete=models.CASCADE)
      # 受保护 模式  如果从表有关联数据 则不能删除主表关联的数据
      # user = models.OneToOneField(User, on_delete=models.PROTECT)
      # 置空模式  设置为null
      user = models.OneToOneField(User, on_delete=models.SET_NULL, null=True)
      def __str__(self):
          return self.name
      class Meta:
          db_table = 'idcard'  # 更改表名称为user
  ```

+ 添加一对一模型数据

  添加主表

  ```python
  def addUser(req):
      User(username='张三').save()
      User(username='李四').save()
      User(username='王五').save()
      return HttpResponse('一对一模型关系添加User数据')
  ```

  添加从表

  ```python
  def addIdCard(req):
      import random
      u = User.objects.first()
      IdCard(num=random.randrange(100000000,99999999999), name=u.username,user=u).save()
      return HttpResponse('一对一模型关系添加IdCard数据')
  ```

+ 查询

  通过主表查从表

  主表对象.从表类名小写

  ```python
  u = User.objects.first()
  # print(u)
  print(u.idcard)
  print(u.idcard.num)
  ```

  通过从表查主表

  从表对象.外键属性名

  ```python
  idcard = IdCard.objects.first()
  # print(idcard)
  print(idcard.user)
  print(idcard.user.sex)
  ```

### 3、一对多模型关系

- 说明

  使用ForeignKey创建一对多模型关系

- 位置

  放在多的一方

- 创建模型

  models.py

  班级模型类

  ```python
  from django.db import models

  # 创建班级模型
  class Grade(models.Model):
      gname = models.CharField(max_length=10)
      gnum = models.SmallIntegerField(default=30)
      gboynum = models.SmallIntegerField(default=15)
      ggirlnum = models.SmallIntegerField(default=15)

      def __str__(self):
          return self.gname
      class Meta:
          db_table = 'grade'
  ```

  学生模型类

  ```python
  from django.db import models

  # 创建学生模型类
  class Student(models.Model):
      sname = models.CharField(max_length=10)
      sage = models.SmallIntegerField(default=18)
      ssex = models.BooleanField(default=True)

      # 一对多关系模型，从表随着主表的删除而删除
      sgrade = models.ForeignKey(Grade, on_delete=models.CASCADE)
      # sgrade = models.ForeignKey(Grade, on_delete=models.SET_NULL, null=True)
      # sgrade = models.ForeignKey(Grade, on_delete=models.PROTECT)
      # sgrade = models.ForeignKey(Grade, on_delete=models.SET_DEFAULT, default=1)

      def __str__(self):
          return self.sname

      class Meta:
          db_table = 'student'
  ```

- 路由

  ```python
  # 一对多模型关系路由
      path('addgrade/', onetomany.addGrade),
      path('addstudent/', onetomany.addStudent),
      path('showstudent/', onetomany.showStudent),
      path('showgrade/', onetomany.showGrade),
  ```

- 视图函数

  添加主表grade数据

  ```python
  from django.http import HttpResponse
  from App.models import Grade, Student

  # 添加班级数据
  def addGrade(request):
      Grade(gname='python35', gnum=25, gboynum=20, ggirlnum=5).save()
      Grade(gname='python36', gnum=21, gboynum=18, ggirlnum=3).save()
      Grade(gname='python37', gnum=10, gboynum=10, ggirlnum=0).save()
      return HttpResponse('添加班级数据')
  ```

  添加从表student数据

  ```python
  from django.http import HttpResponse
  from App.models import Grade, Student

  # 添加学生数据
  def addStudent(request):
      # g36 = Grade.objects.filter(gname='python36')[0]
      # print(g36)
      # Student(sname='张三', sage=20, ssex=True, sgrade=g36).save()
      # Student(sname='李四', sage=37, ssex=False, sgrade=g36).save()
      # Student(sname='李俊', sage=38, ssex=False, sgrade=g36).save()
      # Student(sname='王五', sage=16, ssex=True, sgrade=g36).save()
      g35 = Grade.objects.filter(gname='python35')[0]
      Student(sname='张龙', sage=20, ssex=True, sgrade=g35).save()
      Student(sname='赵虎', sage=37, ssex=False, sgrade=g35).save()
      Student(sname='王朝', sage=38, ssex=False, sgrade=g35).save()
      Student(sname='马汉', sage=16, ssex=True, sgrade=g35).save()
      return HttpResponse('添加学生数据')
  ```

- 查询数据

  主查从（班级查所含学生）

  ```python
  from App.models import Grade, Student

  # 查询班级里所包含的学生（主查从）
  def showStudent(request):
      # g36 = Grade.objects.filter(gname='python36')[0]
      # student = g36.student_set.all()
      # print(student)
      g35 = Grade.objects.filter(gname='python35')[0]
      student = g35.student_set.all()
      print(student)
      return HttpResponse('主查从，查看班级里的学生')
  ```

  从查主（学生查所在班级）

  ```python
  from App.models import Grade, Student

  # 查询学生所在的班级（从查主）
  def showGrade(request):
      # stu_obj = Student.objects.filter(sname='李俊')[0]
      stu_obj = Student.objects.filter(sname='张龙')[0]
      grade_name = stu_obj.sgrade.gname
      print(grade_name)
      return HttpResponse('从查主，查看学生所在班级')
  ```

- 删除

  删除从表student数据，主表不受影响

  ```python
  # 删除student表的数据
  def delStudent(request):
      s = Student.objects.filter(sname='张龙')
      s.delete()
      return HttpResponse('删除从表student的数据')
  ```

  删除主表grade数据（从表根据on_delete策略的变化而变化）

  ```python
  # 删除grade表的数据
  def delGrade(request):
      g = Grade.objects.filter(gname='python35')[0]
      g.delete()
      return HttpResponse('删除主表grade的数据')
  ```

### 4、多对多

- 说明

  使用ManyToManyField创建多对多模型关系

- 原理

  底层是用两个外键实现的，会生成中间表，存储id

- 关系

  放在哪张表都可以

- 创建模型类

  User模型类

  ```python
  from django.db import models

  # 用户模型
  class User(models.Model):
      # 用户名
      username = models.CharField(max_length=12)
      # 密码
      userpass = models.CharField(max_length=64, default='123456')
      # 性别
      sex = models.BooleanField(default=True)
      # 年龄
      age = models.SmallIntegerField(default=18, db_column='u_age')
      # 个人简介
      info = models.CharField(max_length=200, default='个人简介')
      # 头像
      icon = models.CharField(max_length=60, default='default.jpg')
      # 是否删除
      isDelete = models.BooleanField(default=False)
      # 注册时间
      registerTime = models.DateTimeField(auto_now_add=True)

      # 类方法用于添加数据
      def __str__(self):
          return self.username
      class Meta:
          db_table = 'user'  # 更改表名称为user

  ```

  博客Posts模型类

  ```python
  # 博客模型
  class Posts(models.Model):
      title = models.CharField(max_length=20)
      article = models.CharField(max_length=200)
      createTime = models.DateTimeField(auto_now_add=True)

      # 外键，多对多
      users = models.ManyToManyField(User)

      def __str__(self):
          return self.title
      class Meta:
          db_table = 'posts'
  ```

 - 路由

    ```python
    from .views import manytomany

    # 多对多模型关系路由
        path('addmanyuser/', manytomany.addUSer),
        path('addpost/', manytomany.addPost),
        path('addonecollect/', manytomany.addOneCollect),
        path('addmanycollect/', manytomany.addManyCollect),
        path('show_userposts/', manytomany.show_Users_Posts),
        path('show_postusers/', manytomany.show_Posts_Users),
        path('delonecollect/', manytomany.delOneCollect),
        path('delmanycollect/', manytomany.delManyCollect),
    ```

- 添加数据

  添加用户数据User

  ```python
  from django.http import HttpResponse
  from App.models.manytomany import PostUser, Posts

  # 添加用户数据
  def addUSer(request):
      PostUser(username='张龙', userpass=123456, sex=False).save()
      PostUser(username='赵虎', userpass='zhaohu1123@', sex=True).save()
      PostUser(username='李俊', userpass='blackboy', sex=False, age=38, info='I am a blackboy').save()
      return HttpResponse('多对多模型添加数据')
  ```

  添加博客数据Posts

  ```python
  # 添加博客数据
  def addPost(request):
      Posts(title='使用jenkins共享库对流水线进程扩展', article='共享库这并不是一个全新的概念，其实具有编程能力的同学应该清楚一些。例如在编程语言Python中，我们可以将Python代码写到一个文件中，当代码数量增加，我们可以将代码打包成模块然后再以import的方式使用此模块中的方法。').save()
      Posts(title='Nginx 动态DNS解析方案: resolver', article='nginx resolver 存在一定解析失败的概率问题').save()
      Posts(title='Dockerfile文件全面详解', article='Docker 可以通过读取 Dockerfile 中的指令自动构建镜像。Dockerfile 是一个文本文档，其中包含了用户创建镜像的所有命令和说明。').save()
      Posts(title='PLEG is not healthy？幕后黑手居然是它！', article='PLEG 全称叫 Pod Lifecycle Event Generator，即 Pod 生命周期事件生成器。实际上它只是 Kubelet 中的一个模块，主要职责就是通过每个匹配的 Pod 级别事件来调整容器运行时的状态，并将调整的结果写入缓存，使 Pod 的缓存保持最新状态。先来聊聊 PLEG 的出现背景。在 Kubernetes 中，每个节点上都运行着一个守护进程 Kubelet 来管理节点上的容器，调整容器的实际状态以匹配 spec 中定义的状态。').save()
      return HttpResponse('多对多模型关系，添加博客数据')
  ```

- 收藏功能（将数据添加到中间表中）

  添加单个收藏

  ```python
  # 单个收藏
  def addOneCollect(request):
      u1 = PostUser.objects.filter(username='张龙')[0]
      p1 = Posts.objects.first()
      p1.users.add(u1)
      return HttpResponse('单用户添加单个收藏')
  ```

  添加多个收藏

  ```python
  # 多个收藏
  def addManyCollect(request):
      u1 = PostUser.objects.filter(username='赵虎')[0]
      u2 = PostUser.objects.filter(username='李俊')[0]
      p1 = Posts.objects.filter(title='使用jenkins共享库对流水线进程扩展')[0]
      p2 = Posts.objects.filter(title='PLEG is not healthy？幕后黑手居然是它！')[0]
      p1.users.add(u1, u2)
      p2.users.add(u1, u2)
      return HttpResponse('多个用户收藏多个博客')
  ```

- 多对多数据查询

  查询用户收藏了哪些博客

  ```python
  # 查询用户收藏了哪些博客
  def show_Users_Posts(request):
      user = PostUser.objects.first()
      # user = PostUser.objects.filter(username='赵虎')[0]
      # user = PostUser.objects.filter(username='李俊')[0]
      post = user.posts_set.all()
      print(post)
      return HttpResponse('查询用户收藏了哪些博客')
  ```

  查询博客被哪些用户收藏了

  ```python
  # 查询博客被哪些用户收藏了
  def show_Posts_Users(request):
      # 标题太长，不全打了，使用contains包含查询
      post = Posts.objects.filter(title__contains='PLEG is not healthy')[0]
      # post = Posts.objects.filter(title__contains='jenkins')[0]
      user = post.users.all()
      print(user)
      return HttpResponse('查询博客被哪些用户收藏了')
  ```

- 取消收藏

  删除单条数据

  ```python
  # 取消单条
  def delOneCollect(request):
      u1 = PostUser.objects.filter(username='李俊')[0]
      p1 = Posts.objects.filter(title__contains='jenkins')[0]
      p1.users.remove(u1)
      return HttpResponse('删除单条数据 / 取消单个收藏 ')
  ```

  删除多条数据

  ```python
  # 取消多条
  def delManyCollect(request):
      u1 = PostUser.objects.filter(username='张龙')[0]
      u2 = PostUser.objects.filter(username='赵虎')[0]
      p1 = Posts.objects.filter(title__contains='jenkins')[0]
      p1.users.remove(u1, u2)
      return HttpResponse('删除多条数据 / 取消多个收藏')
  ```




## 八、模型类继承

### 1、普通继承

```python
from django.db import models


class A(models.Model):
    name = models.CharField(max_length=10, default='name')
    age = models.SmallIntegerField(default=18)


class B(A):
    class Meta:
        db_table = 'b'


class C(A):
    class Meta:
        db_table = 'c'
```

> 如果此刻进行数据亏迁移：则A类也会创建出模型
>
> 但是这并不是我们的本意，我们只是把A类当做基类，并不需要去创建A表

### 2、将父类进行抽象化

使用 abstract = True 参数将父类进行抽象化，不会创建A表

```python
from django.db import models


class A(models.Model):
    name = models.CharField(max_length=10, default='name')
    age = models.SmallIntegerField(default=18)
    class Meta:
        # 抽象类
        abstract = True


class B(A):
    class Meta:
        db_table = 'b'


class C(A):
    class Meta:
        db_table = 'c'
```

## 八、模型自关联

+ 概述

  自关联就是1张数据表    主键和外键都在一张表上  一般用于 菜单、商品类别、三级联动

+ 创建

  | ｉＤ | 类别名称 | parent_id |
  | ---- | -------- | --------- |
  | 1    | 服装     |           |
  | 2    | 食品     |           |
  | 3    | 数码产品 |           |
  | ４   | 男装     | １        |
  | ５   | 女装     | １        |

  拆分

  父类别表

  | ｉＤ | 类别名称 |
  | ---- | -------- |
  | 1    | 服装     |
  | 2    | 食品     |
  | 3    | 数码产品 |

  子类别表

  | ｉＤ | 类别名称 | parent_id |
  | ---- | -------- | --------- |
  | １   | 男装     | １        |
  | ２   | 女装     | １        |

+ 操作

  ```python
  from .models import GoodsType
  # 自关联查询
  def show(req):
      # 查询子类别
      g_obj = GoodsType.objects.filter(parent_id__isnull=True)
      # fz = g_obj.first()
      # print(fz)
      # 通过服装拿到子类别
      # son_list = fz.son_list.all()
      # print(son_list)
      # 删除类别
      sm = g_obj.last()
      sm.delete()
      return HttpResponse('自关联查询')
  ```

+ 注意

  如果删除的父类没有子类别 则正常删除

  如果父类存在子类别 在删除父类的时候 会抛出异常
