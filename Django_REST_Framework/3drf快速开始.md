# 快速开始

我们将创建一个简单的 API 来实现用户的增删改查，保存在mysql数据库中。

## 1、环境安装与配置

DRF需要以下依赖：

- Python (3.5 以上)
- Django (2.2 以上)

**DRF是以Django子应用的方式提供的，所以我们可以直接利用已有的Django环境而无需从新创建。（若没有Django环境，需要先创建环境安装Django）**

### 1.1 安装DRF

建议安装在虚拟环境

```python
pip install djangorestframework -i https://pypi.douban.com/simple

# 因为我们需要接下来，需要开发api接口肯定要操作数据，所以安装pymysql
pip install pymysql -i https://pypi.douban.com/simple
```

### 1.2 创建django项目和子应用

```
django-admin startproject drfdemo
django-admin startapp students
```

### 1.3 添加rest_framework应用和子应用

在**settings.py**的**INSTALLED_APPS**中添加'rest_framework'。

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'students',
]
```

### 1.4 创建模型类

students/models.py

```python
from django.db import models


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

### 1.5 执行数据迁移

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



接下来就可以使用DRF提供的功能进行api接口开发了。在项目中如果使用rest_framework框架实现API接口，主要有以下三个步骤：

- 将请求的数据（如JSON格式）转换为模型类对象
- 操作数据库
- 将模型类对象转换为响应的数据（如JSON格式）

## 2、接口规划

```bash
5个接口:
    添加1条数据   post /api/student/
    获取所有数据  get /api/student/
    获取1条数据   get /api/student/1/
    更新1条数据   put /api/student/1/
    删除1条数据   delete /api/student/1/
```



## 3、创建序列化器

在students子应用目录中新建serializers.py用于保存该应用的序列化器。

创建一个StudentModelSerializer用于序列化与反序列化。

```python
from rest_framework import serializers
from .models import Student


class StudentModelSerializer(serializers.ModelSerializer):

    class Meta:
        model = Student
        fields = "__all__"
        # fields = ["id","name","age"]
```

- **model** 指明该序列化器处理的数据字段从模型类Student参考生成
- **fields** 指明该序列化器包含模型类中的哪些字段，'\_\_all\_\_'指明包含所有字段，也可以使用列表指定字段

## 4、编写视图

在students应用的views.py中创建视图StudentModelViewSet，这是一个视图集合。

```python
from rest_framework.viewsets import ModelViewSet
from .models import Student
from .serializers import StudentModelSerializer


class StudentModelViewSet(ModelViewSet):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer
```

- **queryset** 指明该视图集在查询数据时使用的查询集
- **serializer_class** 指明该视图在进行序列化或反序列化时使用的序列化器

## 5、定义路由

在students应用的urls.py中定义路由信息。

```python
from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register("student", views.StudentModelViewSet, basename="student"),

# 路由列表
urlpatterns = [

] + router.urls
```

最后把students子应用中的路由文件加载到总路由文件中.

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include("students.urls")),  # 新增的代码
]

```

## 6、运行测试

运行当前程序（与运行Django一样）

```shell
python manage.py runserver 0.0.0.0:8000
```

1、在浏览器中输入网址127.0.0.1:8000/api/，可以看到DRF提供的API Web浏览页面：

![image-20211019194133675](3drf%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B.assets/image-20211019194133675.png)

2、接着在浏览器输入网址127.0.0.1:8000/api/student/，可以访问**获取所有数据的接口**

![image-20211019194445868](3drf%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B.assets/image-20211019194445868.png)

3、在页面底下表单部分填写学生信息，可以访问**添加新学生的接口**，保存学生信息：

![image-20211019194551580](3drf%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B.assets/image-20211019194551580.png)

在其中输入内容，点击 POST 的后，即可得到返回值

4、在浏览器中输入网址127.0.0.1:8000/api/student/4/，可以访问**获取单一学生信息的接口**（id为4的学生），呈现如下页面：

![image-20211019194730901](3drf%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B.assets/image-20211019194730901.png)

5、在页面底部表单中填写学生信息，可以访问**修改学生的接口**：

![image-20211019194755479](3drf%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B.assets/image-20211019194755479.png)

修改部分数据后，点击 PUT ，得到返回信息，查看修改成功与否

6、点击上边的 DELETE ，可以访问**删除学生的接口**：

![image-20211019194936113](3drf%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B.assets/image-20211019194936113.png)

返回如下页面

![image-20211019195015903](3drf%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B.assets/image-20211019195015903.png)

