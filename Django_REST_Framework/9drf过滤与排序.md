# DRF 过滤器



## 1. 过滤Filtering

**使用django-filter模块**

django-filter模块可以做一些更高级的过滤，也可以自定义过滤，比如**分类筛选、筛选区间等**

可以对列表数据根据字段进行过滤

django-filter模块为django和drf都做了相应的支持。如果是drf做前后端分离的项目就使用`from django_filters.rest_framework import DjangoFilterBackend`



### 1）模块安装与配置

```shel
pip install django-filter
```

settings.py，代码：

```python
INSTALLED_APPS = [
    # ....
    'django_filters',
]
```

### 2）全局配置

在配置文件中增加过滤器类的**全局设置**：

```python
"""drf配置信息必须全部写在REST_FRAMEWORK配置项中"""
REST_FRAMEWORK = {
    # ....代码省略。。。
    
    # 过滤查询，全局配置
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend'
    ],
}
```

在视图类中添加类属性filter_fields，指定可以过滤的字段

```python
class Hom4APIView(ModelViewSet):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer
    filter_fields = ["sex","classmate"]
    # list方法中进行调用->调用了GenericAPIView中声明的filter_queryset方法---> 配置中的过滤器类的filter_queryset---> filter_fields
    
# 单个字段过滤
# http://127.0.0.1:8000/opt/s4/?classmate=301
# http://127.0.0.1:8000/opt/s4/?sex=1
# 多个字段过滤
# http://127.0.0.1:8000/opt/s4/?classmate=502&sex=2
```

路由代码：

```python
from django.urls import path,include
from . import views

from rest_framework.routers import DefaultRouter
router = DefaultRouter()
router.register("s2", views.Hom2APIView, "s2")
router.register("s4", views.Hom4APIView, "s4")

urlpatterns = [
    path("s1/", views.HomeAPIView.as_view()),
    path("", include(router.urls)),  # 等价于  urlpatterns += router.urls
    path("s3/", views.Hom3APIView.as_view()),
    path("s3/", views.Hom3APIView.as_view()),
]
```

3）局部配置

**局部设置**，直接在视图中，指定当前视图类中调用的过滤器类

```python
# 先注释掉settings中的全局过滤配置
from rest_framework.viewsets import ModelViewSet
from students.models import Student
from students.serializers import StudentModelSerializer

from django_filters.rest_framework import DjangoFilterBackend
class Hom4APIView(ModelViewSet):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer
    filter_backends = [DjangoFilterBackend, ]
    filter_fields = ["sex","classmate"]
    # list方法中进行调用->调用了GenericAPIView中声明的filter_queryset方法---> 配置中的过滤器类的filter_queryset---> filter_fields
```

## 2. 基于django-filter模块自定义过滤器类实现价格区间

- **自定义过滤器类。继承`from django_filters.rest_framework.filterset import FilterSet`类**
- **自定义分类筛选的字段。在Meta类中通过fields来指定筛选的字段。**和定义序列化类方法一致

过滤器类，代码：

```python
# 基于django-filter插件，完成指定区间筛选（一般都是对应数字字段）
from django_filters.rest_framework.filterset import FilterSet
from django_filters import filters
from . import models
class CourseFilterSet(FilterSet):
    # field_name：要过滤的数据库字段，lookup_expr：过滤条件
    min_price = filters.NumberFilter(field_name='price', lookup_expr='gte') # 条件：大于price
    max_price = filters.NumberFilter(field_name='price', lookup_expr='lte') # 条件：小于price
    class Meta:
        model = models.Course
        # 参与分类筛选的字段：
        fields = ['course_category', 'max_price', 'min_price']
```

视图，代码：

```python
# 分类筛选：django-filter：filter_backends配置DjangoFilterBackend，再在filter_fields中配置分组筛选的字段
from django_filters.rest_framework import DjangoFilterBackend
# 自定义过滤器类
from . import filters
class FreeCourseListAPIView(ListAPIView):
    queryset = models.Course.objects.filter(is_delete=False, is_show=True).order_by(
        "-orders").all()
    serializer_class = serializers.FreeCourseModelSerializer
    
    # 局部配置过滤器类
    filter_backends = [DjangoFilterBackend]
    # 使用基于django-filter自定义过滤器类
    filter_class = filters.CourseFilterSet

# 携带分类筛选字段的参数的url: http://127.0.0.1:8000/course/free?min_price=20&max_price=80
```



## 3. 排序Ordering

对于列表数据，REST framework提供了**OrderingFilter**过滤器来帮助我们快速指明数据按照指定字段进行排序。

使用方法：

在类视图中设置filter_backends，使用`rest_framework.filters.OrderingFilter`过滤器，REST framework会在请求的查询字符串参数中检查是否包含了ordering参数，如果包含了ordering参数，则按照ordering参数指明的排序字段对数据集进行排序。

前端可以传递的ordering参数的可选字段值需要在ordering_fields中指明。

配置文件，settings.py，代码：

```python
"""drf配置信息必须全部写在REST_FRAMEWORK配置项中"""
REST_FRAMEWORK = {
    # 。。。代码省略
    # 过滤查询，全局配置
    # 过滤和排序使用了一个公用的配置项，所以2个组件要么一起全局配置，要么就一起局部配置
    'DEFAULT_FILTER_BACKENDS': [
    #     'django_filters.rest_framework.DjangoFilterBackend',  # 过滤
        'rest_framework.filters.OrderingFilter',  # 排序
    ],
}
```

视图代码：

```python
from rest_framework.viewsets import ModelViewSet
from students.models import Student
from students.serializers import StudentModelSerializer

from django_filters.rest_framework import DjangoFilterBackend
class Hom4APIView(ModelViewSet):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer
    # 局部过滤
    # filter_backends = [DjangoFilterBackend, OrderingFilter]
    # 过滤字段
    # filter_fields = ["sex","classmate"]
    # 数据排序
    ordering_fields = ['id', 'age']

# 127.0.0.1:8000/books/?ordering=-age
# -id 表示针对id字段进行倒序排序
# id  表示针对id字段进行升序排序
```



上面提到，因为过滤和排序公用了一个配置项，所以排序和过滤要一起使用，则必须整个项目，要么一起全局过滤排序，要么一起局部过滤排序。决不能出现，一个全局，一个局部的这种情况，局部会覆盖全局的配置。

```python
from rest_framework.viewsets import ModelViewSet
from students.models import Student
from students.serializers import StudentModelSerializer

from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import OrderingFilter
class Hom4APIView(ModelViewSet):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer
    # 局部过滤
    filter_backends = [DjangoFilterBackend, OrderingFilter]
    # 过滤字段
    filter_fields = ["sex","classmate"]
    # list方法中进行调用->调用了GenericAPIView中声明的filter_queryset方法---> 配置中的过滤器类的filter_queryset---> filter_fields

    # 数据排序
    ordering_fields = ['id', 'age']
    
# http://127.0.0.1:8000/opt/demo4/?ordering=id&classmate=301
```