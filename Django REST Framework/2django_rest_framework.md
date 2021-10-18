# Django RESTful framework

核心思想: 大量缩减编写api接口的代码

Django REST framework是一个建立在Django基础之上的Web 应用开发框架，可以快速的开发REST API接口应用。在REST framework中，提供了序列化器Serialzier的定义，可以帮助我们简化序列化与反序列化的过程，不仅如此，还提供丰富的类视图、扩展类、视图集来简化视图的编写工作。REST framework还提供了认证、权限、限流、过滤、分页、接口文档等功能支持。REST framework还提供了一个调试API接口 的Web可视化界面来方便查看测试接口。

中文文档：https://q1mi.github.io/Django-REST-framework-documentation/#django-rest-framework

github: https://github.com/encode/django-rest-framework/tree/master

### 特点

- 提供了定义序列化器Serializer的方法，可以快速根据 Django ORM 或者其它库自动序列化/反序列化；
- 提供了丰富的类视图、Mixin扩展类，简化视图的编写；
- 丰富的定制层级：函数视图、类视图、视图集合到自动生成 API，满足各种需要；
- 多种身份认证和权限认证方式的支持；[jwt]
- 内置了限流系统；
- 直观的 API web 界面；【方便我们调试开发api接口】
- 可扩展性，插件丰富

## 1、环境安装与配置

DRF需要以下依赖：

- Python (3.5 以上)
- Django (2.2 以上)

**DRF是以Django子应用的方式提供的，所以我们可以直接利用已有的Django环境而无需从新创建。（若没有Django环境，需要先创建环境安装Django）**

### 1.1 安装DRF

anaconda (内置180多个python模块，)/ miniconda（内置仅仅是30多个常用模块）

前提是已经安装了django，建议安装在虚拟环境

```python
# conda create -n drfdemo python=3.8
# pip install django==3.2.4  -i https://pypi.douban.com/simple

pip install djangorestframework -i https://pypi.douban.com/simple

# 因为我们需要接下来，需要开发api接口肯定要操作数据，所以安装pymysql
pip install pymysql -i https://pypi.douban.com/simple
```

### 1.2 创建django项目

```
cd ~/Desktop
django-admin startproject drfdemo
```

使用pycharm打开项目，设置虚拟环境的解析器，并修改manage.py中的后缀参数。

### 1.3 添加rest_framework应用

在**settings.py**的**INSTALLED_APPS**中添加'rest_framework'。

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

接下来就可以使用DRF提供的功能进行api接口开发了。在项目中如果使用rest_framework框架实现API接口，主要有以下三个步骤：

- 将请求的数据（如JSON格式）转换为模型类对象
- 操作数据库
- 将模型类对象转换为响应的数据（如JSON格式）



接下来，我们快速体验下四天后我们学习完成drf以后的开发代码。接下来代码不需要理解，看步骤。

### 1.4 体验drf完全简写代码的过程

```bash
5个接口:
    添加1条数据
    获取所有数据
    获取1条数据
    更新1条数据
    删除1条数据

# 终端下执行：
django-admin startapp stuapi    # 提供原生的django代码实现的API接口
django-admin startapp students  # 提供drf代码实现的API接口
```

#### 1.4.1. 创建模型操作类

stuapi/models.py，代码：

```python
from django.db import models


# Create your models here.
class Student(models.Model):
    """学生信息"""
    name = models.CharField(max_length=255, verbose_name="姓名")
    sex = models.BooleanField(default=1, verbose_name="性别")
    age = models.IntegerField(verbose_name="年龄")
    classmate = models.CharField(max_length=5, verbose_name="班级编号")
    description = models.TextField(max_length=1000, verbose_name="个性签名")

    class Meta:
        db_table = "tb_student"
        verbose_name = "学生"
        verbose_name_plural = verbose_name

```

为了方便测试，所以我们可以先创建一个数据库。

```
mysql -uroot -p123
create database students charset=utf8mb4;
```

#### 1.4.2 执行数据迁移

把students子应用添加到INSTALL_APPS中

初始化数据库连接

```
安装pymysql
pip install pymysql
```

主引用中`__init__.py`设置使用pymysql作为数据库驱动

```python
import pymysql
pymysql.install_as_MySQLdb()
```

settings.py配置文件中设置mysql的账号密码

```python
DATABASES = {
    # 'default': {
    #     'ENGINE': 'django.db.backends.sqlite3',
    #     'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    # },
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': "students",
        "HOST": "127.0.0.1",
        "PORT": 3306,
        "USER": "root",
        "PASSWORD":"123",
    },
}
```

终端下，执行数据迁移。

```
python manage.py makemigrations
python manage.py migrate
```



#### 1.4.3 使用原生的Django代码提供基本API接口

```python
import json

from django.views import View
from django.http.response import JsonResponse
from .models import Student

# Create your views here.
"""
POST /students/   添加一个学生信息
GET  /students/   获取所有学生信息

GET /students/<pk>/  获取一个学生信息
PUT /students/<pk>/  更新一个学生信息
DELETE /students/<pk>/  删除一个学生信息
 
一个路由对应一个视图类，所以我们可以把5个API分成2个类来完成
"""


class StudentView(View):
    """学生视图"""

    def post(self, request):
        """添加一个学生信息"""
        # 1. 接收客户单提交的数据，验证客户端的数据
        data = json.loads(request.body)
        name = data.get("name")
        sex = data.get("sex")
        age = data.get("age")
        classmate = data.get("classmate")
        description = data.get("description")

        # 2. 操作数据库，保存数据
        instance = Student.objects.create(
            name=name,
            sex=sex,
            age=age,
            classmate=classmate,
            description=description,
        )

        # 3. 返回结果
        return JsonResponse(data={
            "id": instance.pk,
            "name": instance.name,
            "sex": instance.sex,
            "age": instance.age,
            "classmate": instance.classmate,
            "description": instance.description,
        }, status=201)

    def get(self, request):
        """获取多个学生信息"""
        # 1. 读取数据库
        students_list = list(Student.objects.values())

        # 2. 返回数据
        return JsonResponse(data=students_list, status=200, safe=False)


class StudentInfoView(View):
    def get(self, request, pk):
        """获取一条数据"""
        try:
            instance = Student.objects.get(pk=pk)
            return JsonResponse(data={
                "id": instance.pk,
                "name": instance.name,
                "sex": instance.sex,
                "age": instance.age,
                "classmate": instance.classmate,
                "description": instance.description,
            }, status=200)

        except Student.DoesNotExist:
            return JsonResponse(data=None, status=404)  # 没有内容

    def put(self, request, pk):
        """更新一个学生信息"""
        # 1. 接收客户单提交的数据，验证客户端的数据
        data = json.loads(request.body)
        name = data.get("name")  # alt+j 选中多个一样的
        sex = data.get("sex")
        age = data.get("age")
        classmate = data.get("classmate")
        description = data.get("description")

        # 2. 操作数据库，保存数据
        try:
            instance = Student.objects.get(pk=pk)
            instance.name = name
            instance.sex = sex
            instance.age = age
            instance.classmate = classmate
            instance.description = description
            instance.save()

        except Student.DoesNotExist:
            return JsonResponse(data={}, status=404)  # 没有内容

        # 3. 返回结果
        return JsonResponse(data={
            "id": instance.pk,
            "name": instance.name,
            "sex": instance.sex,
            "age": instance.age,
            "classmate": instance.classmate,
            "description": instance.description,
        }, status=201)

    def delete(self, request, pk):
        """删除一个学生信息"""
        try:
            Student.objects.filter(pk=pk).delete()
        except:
            pass
        return JsonResponse(data={}, status=204)

```

子应用下创建urls.py，代码：

```python
from django.urls import path,re_path
from . import views
urlpatterns = [
    path("students/", views.StudentView.as_view()),
    re_path("^students/(?P<pk>\d+)/$", views.StudentInfoView.as_view()),
]
```

在settings.py，配置子应用，

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'rest_framework',
    'stuapi',
    'students',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

总路由，urls.py，代码：

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include("stuapi.urls")),
]

```

