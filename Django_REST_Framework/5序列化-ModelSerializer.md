# 序列化器 ModelSerializer

如果我们想要使用序列化器对应的是Django的模型类，DRF为我们提供了ModelSerializer模型类序列化器来帮助我们快速创建一个Serializer类。

ModelSerializer与常规的Serializer相同，但额外提供了：

- 基于模型类自动生成一系列字段
- 基于模型类自动为Serializer生成validators，比如unique_together
- 包含默认的create()和update()的实现

**什么时候声明的序列化器需要继承序列化器基类Serializer，什么时候继承模型序列化器类ModelSerializer？**

```
继承序列化器类Serializer
	字段声明
	验证
	添加/保存数据功能
继承模型序列化器类ModelSerializer
	字段声明[可选,看需要]
	Meta声明
	验证
	添加/保存数据功能[可选]
```

**看数据是否从mysql数据库中获取，如果是则使用ModelSerializer，不是则使用Serializer**

## 1、定义 ModelSerializer

**接着上篇文档，在sers子应用中继续实现ModelSerializer**

创建一个Model序列化器，sers子应用/serializers.py中

```python
from rest_framework import serializers
from rest_framework.serializers import ModelSerializer
from .models import Student


# ModelSerializer
class StudentModelSerializer(ModelSerializer):
    """学生信息序列化器"""
    # 1、转换模型类字段
    nickname = serializers.CharField(read_only=True)
    # 2. 如果当前序列化器继承的是ModelSerializer，则需要声明调用的模型信息
    # 必须给Meta声明2个属性
    class Meta:
        models = Student
        # fields = "__all__"
        fields = ["id", "name", "age", "sex", "nickname", "classmate", "description"]
        # read_only_fields = []  # 选填，只读字段列表，表示设置这里的字段只会在序列化阶段采用
        extra_kwargs = {  # 选填，字段额外选项声明
            "age": {
                "min_value": 1,
                "max_value": 100,
                "error_messages": {
                    "min_value": "年龄的最小值必须大于等于1",
                    "max_value": "年龄的最大值必须小于等于100",
                }
            },
        }

    # 3. 验证代码的对象方法
    # def create(self, validated_data):
    # 密码加密
    # validated_data["password"] = make_password(validated_data["password"])
    # super().create(validated_data)

    # 4. 模型操作的方法，
```

## 2、Meta属性及选项

Meta类里面必须声明2个属性：

- model 指明参照哪个模型类
- fields 指明为模型类的哪些字段生成

Meta类的其他属性及选项

- **exclude **                   可以明确排除掉哪些字段，不建议使用
- **read_only_fields**    指明只读字段，**只参与序列化**
- **write_only_fields**    代表该字段，**只参与反序列化**
- **extra_kwargs **          参数为ModelSerializer添加或修改原有的选项参数

## 3、定义视图

视图代码：

```python
from django.views import View
from django.http.response import JsonResponse
from .serializers import StudentSerializer1, StudentSerializer2, StudentModelSerializer
from stu.models import Student


# Model序列化器视图类
class StudentView3(View):
    """模型序列化视图类"""

    def get(self, request):
        # 1、获取数据集
        # 1.1、获取单个数据集
        # instance = Student.objects.first()
        # 1.1、获取多个个数据集
        instance = Student.objects.all()
        # 2、实例化序列化器，多个参数使用many=True
        # serializer = StudentModelSerializer(instance)
        serializer = StudentModelSerializer(instance, many=True)  # 多个数据集的时候要设置 many=True
        # 3、调用序列化对象的data方法获取转换后的数据
        data = serializer.data
        # 4、返回数据
        return JsonResponse(data=data, status=200, safe=False)


# Model反序列化视图类
class StudentView4(View):
    """模型序列化视图类"""

    def get1(self, request):
        """反序列化
        这里就不根据请求动作来写代码了，直接使用模拟数据
        """
        # 0、根据客户端访问的url地址中，获取pk值
        pk = 5
        try:
            instance = Student.objects.get(pk=pk)
        except Student.DoesNotExist:
            return JsonResponse({"errors": "当前学生不存在"}, status=400)
        # 1、接受客户端传入的数据
        # data = json.dumps(request.body)
        # 使用模拟数据
        data = {
            "name": "小紫",
            "age": 18,
            "sex": False,
            "classmate": 110,
            "description": "紫色"
        }
        # 2、实例化序列化器，多个参数使用many=True

        # 2.1 添加数据 、仅传入data数据即可
        serializer = StudentModelSerializer(data=data)
        # 2.2、修改数据 、需要传入instance和dta两个参数
        # serializer = StudentModelSerializer(instance=instance, data=data)

        # 3、调用锡类华对象的data方法获取转换后的数据
        # :parm: raise_exception 的值为True时，抛出异常，代码不会往下执行
        #                          值为False，不抛出异常，使用后面的else后的返回数据
        ret = serializer.is_valid(raise_exception=True)
        serializer.save()

        # 4、返回数据
        if ret:
            return JsonResponse(dict(serializer.validated_data))
        else:
            return JsonResponse(dict(serializer.errors))

    def get(self, request):
        """
        todo 这个函数用来验证 partial 参数
             可以允许用户只提交部分数据
        """
        pk = 7
        try:
            instance = Student.objects.get(pk=pk)
        except Student.DoesNotExist:
            return JsonResponse({"errors": "当前学生不存在"}, status=400)

        # 使用模拟数据, 修改刚才新添加的用户 小紫 的部分数据
        data = {
            "classmate": 333,
            "description": "777哈哈哈哈哈",
        }
        serializer = StudentModelSerializer(instance=instance, data=data, partial=True)
        ret = serializer.is_valid(raise_exception=True)
        serializer.save()

        # 4、返回数据
        if ret:
            return JsonResponse(data=serializer.data, status=200)
        else:
            return JsonResponse(dict(serializer.errors))

```

## 4、序列化器的嵌套使用

在序列化器使用过程中，一般一个序列化器对应一个模型数据。往往因为模型之间会存在外键关联，所以一般在输出数据时不仅要获取当前模型的数据，甚至其他模型的数据也需要同时返回，这种情况下，我们可以通过序列化器嵌套调用的方式，帮我们把当前模型数据进行转换以外还额可以同时转换外键对应的模型数据。

为了方便演示，此处我们再次创建一个新的子应用 school 来完成这块内容的学习。

```python
python manage.py startapp school
```

settings.py注册子应用，然后总路由总include子应用的路由

```python
INSTALLED_APPS = [
	.......
    "school",  # 序列化器嵌套
]
```

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
	......
    path('school/', include("school.urls")),
]
```



### 4.1、创建模型类

为了方便演示嵌套这个过程。所以我们在这里简单创建几个关联模型关系。

模型models.py，代码：

```python
from django.db import models
from django.utils import timezone as datetime

"""
学生     sch_student        1                id=1   name=xiaoming      
成绩     sch_achievement    m    m           id=17  student_id=1   score=97
课程     sch_course              1    m      id=12  teacher_id=2 course='python基础'
老师     sch_teacher                  1
"""


class Student(models.Model):
    name = models.CharField(max_length=255, verbose_name='学生姓名')
    age = models.IntegerField(verbose_name='年龄')
    sex = models.BooleanField(default=False, verbose_name='性别')

    class Meta:
        db_table = 'school_student'

    def __str__(self):
        return self.name


class Teacher(models.Model):
    name = models.CharField(max_length=255, verbose_name='老师名称')
    sex = models.BooleanField(default=False, verbose_name='老师性别')

    class Meta:
        db_table = 'school_teacher'

    def __str__(self):
        return self.name


class Course(models.Model):
    name = models.CharField(max_length=255, verbose_name='课程名称')
    teacher = models.ForeignKey(
        Teacher, on_delete=models.DO_NOTHING, related_name='course',
        db_constraint=False)

    class Meta:
        db_table = 'school_course'

    def __str__(self):
        return self.name


class Achievement(models.Model):
    score = models.DecimalField(verbose_name='课程分数', default=0, max_digits=4, decimal_places=1)
    student = models.ForeignKey(Student, on_delete=models.DO_NOTHING, related_name='s_achievement', db_constraint=False)
    course = models.ForeignKey(Course, on_delete=models.DO_NOTHING, related_name='c_course', db_constraint=False)
    create_time = models.DateTimeField(auto_created=datetime.now)

    class Meta:
        db_table = 'school_achievement'

    def __str__(self):
        return self.score

```

数据迁移：

```
python manage.py makemigrations
python manage.py migrate
```

### 4.2、数据库插入测试数据

```sql
INSERT INTO school_teacher (id, name, sex) VALUES (1, '李老师', 0);
INSERT INTO school_teacher (id, name, sex) VALUES (2, '曹老师', 1);
INSERT INTO school_teacher (id, name, sex) VALUES (3, '许老师', 1);

INSERT INTO school_student (id, name, age, sex) VALUES (1, '小明', 18, 1);
INSERT INTO school_student (id, name, age, sex) VALUES (2, '小黑', 22, 1);
INSERT INTO school_student (id, name, age, sex) VALUES (3, '小白', 18, 1);
INSERT INTO school_student (id, name, age, sex) VALUES (4, '小红', 17, 0);
INSERT INTO school_student (id, name, age, sex) VALUES (5, '小程', 18, 0);
INSERT INTO school_student (id, name, age, sex) VALUES (6, '小辉', 16, 0);
INSERT INTO school_student (id, name, age, sex) VALUES (7, '小明2', 18, 1);
INSERT INTO school_student (id, name, age, sex) VALUES (8, '小黑2', 22, 1);
INSERT INTO school_student (id, name, age, sex) VALUES (9, '小白2', 18, 1);
INSERT INTO school_student (id, name, age, sex) VALUES (10, '小红2', 17, 0);
INSERT INTO school_student (id, name, age, sex) VALUES (11, '小程2', 18, 0);
INSERT INTO school_student (id, name, age, sex) VALUES (12, '小辉2', 16, 0);

INSERT INTO school_course (id, name, teacher_id) VALUES (1, 'python入门', 1);
INSERT INTO school_course (id, name, teacher_id) VALUES (2, 'python进阶', 2);
INSERT INTO school_course (id, name, teacher_id) VALUES (3, 'python高级', 3);
INSERT INTO school_course (id, name, teacher_id) VALUES (4, 'django入门', 2);
INSERT INTO school_course (id, name, teacher_id) VALUES (5, 'django进阶', 3);
INSERT INTO school_course (id, name, teacher_id) VALUES (6, 'django高级', 3);

INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 1, 100.0, 1, 1);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 2, 100.0, 2, 2);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 3, 100.0, 3, 3);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 4, 78.0, 4, 4);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 5, 78.0, 5, 5);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 6, 78.0, 6, 6);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 7, 78.0, 1, 7);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 8, 80.0, 2, 8);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 9, 80.0, 3, 9);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 10, 80.0, 4, 10);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 11, 80.0, 5, 11);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 12, 60.0, 6, 12);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 13, 100.0, 1, 2);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 14, 100.0, 2, 3);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 15, 100.0, 3, 4);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 16, 78.0, 4, 5);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 17, 78.0, 5, 6);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 18, 78.0, 6, 7);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 19, 78.0, 1, 8);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 20, 80.0, 2, 9);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 21, 80.0, 3, 10);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 22, 80.0, 4, 11);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 23, 80.0, 5, 12);
INSERT INTO school_achievement (create_time, id, score, course_id, student_id) VALUES ('2021-06-16 01:18:25.496000', 24, 60.0, 6, 1);
```

### 4.3、新建序列化器，重新指定关联属性的值

#### 4.3.1、一对多或者多对多的序列化器嵌套返回多个数据情况

默认情况下，模型经过序列化器的数据转换，对于外键的信息，仅仅把数据库里面的外键ID返回。

序列化器，serializers.py，代码：

```python
from rest_framework import serializers
from school.models import Student, Teacher, Course, Achievement


class CourseModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = Course
        # fields = "__all__"
        fields = ["id", "name"]


class TeacherModelSeializer(serializers.ModelSerializer):
    # 1. 重写模型对应的外键，指定返回的数据是什么
    class Meta:
        model = Teacher
        fields = ["id", "name", "course"]
```

视图代码：

```python
from school.models import Teacher
from school.serializers import TeacherModelSeializer
from django.views import View
from django.http.response import JsonResponse


class TeacherView(View):
    def get1(self, request):
        """获取一个老师信息"""
        teacher = Teacher.objects.last()
        serializer = TeacherModelSeializer(instance=teacher)
        return JsonResponse(
            data=serializer.data, safe=False, json_dumps_params={'ensure_ascii':False}
        )
    
    def get(self, request):
        """获取多个老师信息"""
        teacher = Teacher.objects.all()
        serializer = TeacherModelSerializer(instance=teacher, many=True)
        return JsonResponse(
            data=serializer.data, safe=False, json_dumps_params={'ensure_ascii': False}
        )
```

路由代码：

```python
from django.urls import path, re_path
from school import views

urlpatterns = [
    path("teacher1/", views.TeacherView.as_view()),
]
```

访问效果：

- 一个老师信息

![image-20211021161043343](5%E5%BA%8F%E5%88%97%E5%8C%96-ModelSerializer.assets/image-20211021161043343.png)

- 多个老师信息

![image-20211021161836563](5%E5%BA%8F%E5%88%97%E5%8C%96-ModelSerializer.assets/image-20211021161836563.png)


#### 4.3.2、多对一或者一对一的序列化器嵌套返回一个数据情况

序列化器，代码：

```python
from rest_framework import serializers
from school.models import Student, Teacher, Course, Achievement

class Teacher2ModelSerializer(serializers.ModelSerializer):
    # 1. 重写模型对应的外键，指定返回的数据是什么
    class Meta:
        model = Teacher
        fields = ["id", "name"]


class Course2ModelSerializer(serializers.ModelSerializer):
    teacher = Teacher2ModelSerializer()

    class Meta:
        model = Course
        fields = ["id", "name", "teacher"]

```

视图类，代码：

```python
from school.models import Teacher, Course
from school.serializers import TeacherModelSerializer, Course2ModelSerializer
from django.views import View
from django.http.response import JsonResponse

class CourseView(View):
    def get1(self, request):
        """获取一个课程"""
        instance = Course.objects.last()
        serializer = Course2ModelSerializer(instance=instance)
        return JsonResponse(
            data=serializer.data, safe=False, json_dumps_params={'ensure_ascii': False}
        )

    def get(self, request):
        """获取多个课程"""
        instance = Course.objects.all()
        serializer = Course2ModelSerializer(instance=instance, many=True)
        return JsonResponse(
            data=serializer.data, safe=False, json_dumps_params={'ensure_ascii': False}
        )
```

路由，代码：

```python
from django.urls import path, re_path
from school import views

urlpatterns = [
    path("course1/", views.CourseView.as_view()),
]
```

访问效果：

![image-20211021164032792](5%E5%BA%8F%E5%88%97%E5%8C%96-ModelSerializer.assets/image-20211021164032792.png)

### 4.4、source选项取代主键数值

在多对一或者一对一的序列化器嵌套中，通过source选项，直接通过外键指定返回的某个字段数据。

序列化器，`serializers.py`，代码：

```python
class Course2ModelSerializer(serializers.ModelSerializer):
    # teacher = Teacher2ModelSerializer()
    teacher = serializers.CharField(source="teacher.name")
    teacher_id = serializers.CharField(source="teacher.id")
    class Meta:
        model = Course
        fields = ["id","name","teacher","teacher_id"]
```

访问效果：

![image-20211021164411772](5%E5%BA%8F%E5%88%97%E5%8C%96-ModelSerializer.assets/image-20211021164411772.png)

### 4.5、直接指定关联深度属性

序列化器，代码：

```python
from rest_framework import serializers
from school.models import Student, Teacher, Course, Achievement

class Course2ModelSerializer(serializers.ModelSerializer):
    # teacher = Teacher2ModelSerializer()
    # teacher = serializers.CharField(source="teacher.name")
    # teacher_id = serializers.CharField(source="teacher.id")

    class Meta:
        model = Course
        # fields = ["id", "name", "teacher", "teacher_id"]
        fields = ["id", "name", "teacher"]
        # 指定关联深度
        # 从老师模型->课程 = 1
        # 从老师模型->课程->成绩 = 2
        # 从老师模型->课程->成绩->学生 = 3
        # ....
        depth = 1
```

视图代码：

```python
from school.models import Teacher, Course, Student
from school.serializers import TeacherModelSerializer, Course2ModelSerializer, StudentModelSerializer
from django.views import View
from django.http.response import JsonResponse

class CourseView(View):
    def get1(self, request):
        """获取一个课程"""
        instance = Course.objects.last()
        serializer = Course2ModelSerializer(instance=instance)
        return JsonResponse(
            data=serializer.data, safe=False, json_dumps_params={'ensure_ascii': False}
        )

    def get(self, request):
        """获取多个课程"""
        instance = Course.objects.all()
        serializer = Course2ModelSerializer(instance=instance, many=True)
        return JsonResponse(
            data=serializer.data, safe=False, json_dumps_params={'ensure_ascii': False}
        )

class StudentView(View):
    def get1(self, request):
        instance = Student.objects.first()
        serializer = StudentModelSerializer(instance=instance)
        return JsonResponse(
            data=serializer.data, safe=False, json_dumps_params={'ensure_ascii': False}
        )

    def get(self, request):
        instance = Student.objects.all()
        serializer = StudentModelSerializer(instance=instance, many=True)
        return JsonResponse(
            data=serializer.data, safe=False, json_dumps_params={'ensure_ascii': False}
        )

```

路由代码

```python
from django.urls import path, re_path
from school import views

urlpatterns = [
    path("course1/", views.CourseView.as_view()),
    path("student1/", views.StudentView.as_view()),
]
```

### 4.6、自定义模型属性方法

模型，models.py，代码：

```python
from django.db import models
from django.utils import timezone as datetime

"""
学生     sch_student        1                id=1   name=xiaoming      
成绩     sch_achievement    m    m           id=17  student_id=1   score=97
课程     sch_course              1    m      id=12  teacher_id=2 course='python基础'
老师     sch_teacher                  1
"""


class Student(models.Model):
    name = models.CharField(max_length=255, verbose_name='学生姓名')
    age = models.IntegerField(verbose_name='年龄')
    sex = models.BooleanField(default=False, verbose_name='性别')

    class Meta:
        db_table = 'school_student'

    def __str__(self):
        return self.name

    # 自定义模型方法
    @property
    def s_achievement(self):
        return self.s_achievement.values()
        # return self.s_achievment.values("course__teacher__name", "course__name", "score")



class Teacher(models.Model):
    name = models.CharField(max_length=255, verbose_name='老师名称')
    sex = models.BooleanField(default=False, verbose_name='老师性别')

    class Meta:
        db_table = 'school_teacher'

    def __str__(self):
        return self.name


class Course(models.Model):
    name = models.CharField(max_length=255, verbose_name='课程名称')
    teacher = models.ForeignKey(
        Teacher, on_delete=models.DO_NOTHING, related_name='course',
        db_constraint=False)

    class Meta:
        db_table = 'school_course'

    def __str__(self):
        return self.name


class Achievement(models.Model):
    score = models.DecimalField(verbose_name='课程分数', default=0, max_digits=4, decimal_places=1)
    student = models.ForeignKey(Student, on_delete=models.DO_NOTHING, related_name='s_achievement', db_constraint=False)
    course = models.ForeignKey(Course, on_delete=models.DO_NOTHING, related_name='c_course', db_constraint=False)
    create_time = models.DateTimeField(auto_created=datetime.now)

    class Meta:
        db_table = 'school_achievement'

    def __str__(self):
        return self.score

```

序列化器，代码：

```python
class Student2ModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = Student
        fields = ["id", "name", "s_achievement"]
        depth = 3
```

视图代码：

```python
class StudentView(View):
    def get1(self, request):
        instance = Student.objects.first()
        # serializer = StudentModelSerializer(instance=instance)
        serializer = Student2ModelSerializer(instance=instance)
        return JsonResponse(
            data=serializer.data, safe=False, json_dumps_params={'ensure_ascii': False}
        )

    def get(self, request):
        instance = Student.objects.all()
        # serializer = StudentModelSerializer(instance=instance, many=True)
        serializer = Student2ModelSerializer(instance=instance, many=True)
        return JsonResponse(
            data=serializer.data, safe=False, json_dumps_params={'ensure_ascii': False}
        )
```

路由代码

```python
from django.urls import path, re_path
from school import views

urlpatterns = [
    path("course1/", views.CourseView.as_view()),
    path("student1/", views.StudentView.as_view()),
]
```

