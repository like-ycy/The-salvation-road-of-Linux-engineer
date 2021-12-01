# DRF 视图家族

Django REST framwork 提供的视图的主要作用：

- 控制序列化器的执行（检验、保存、转换数据）
- 控制数据库模型的操作

## 一、普通视图

REST framework 提供了众多的通用视图基类与扩展类，以简化视图的编写。

2个视图基类 APIView、GenericAPIView

5个视图扩展类 RetrieveModelMixin, ListModelMixin, CreateModelMixin, UpdateModelMixin, DestroyModelMixin

9个视图子类 ListAPIView,CreateAPIView,UpdateAPIView,DestroyAPIView,RetrieveAPIView

​							 ListCreateAPIView,RetrieveUpdateAPIView,RetrieveDestroyAPIView

​							 RetrieveUpdateDestroyAPIView



### 1、APIView [基本视图类]

```
rest_framework.views.APIView
```

`APIView`是REST framework提供的所有视图类的基类，继承自Django的`View`父类。

`APIView`与`View`的不同之处在于：

- 传入到视图方法中的是REST framework的`Request`对象，而不是Django的`HttpRequeset`对象；

- 视图方法可以返回REST framework的`Response`对象，视图会为响应数据设置（renderer）符合前端期望要求的格式；

- 任何`APIException`异常都会被捕获到，并且处理成合适格式的响应信息返回给客户端；

  django 的View中所有异常全部以HTML格式显示

  drf的APIVIew或者APIView的子类会自动根据客户端的Accept进行错误信息的格式转换。

- 重新声明了一个新的as_view方法并在dispatch()进行路由分发前，会对请求的客户端进行身份认证、权限检查、流量控制。

APIView除了继承了View原有的属性方法意外，还新增了类属性：

- **authentication_classes** 列表或元组，身份认证类
- **permissoin_classes** 列表或元组，权限检查类
- **throttle_classes** 列表或元祖，流量控制类

在`APIView`中仍以常规的类视图定义方法来实现get() 、post() 或者其他请求方式的方法。

#### 1.1、定义序列化器

**模型类这里不再创建，使用sers子应用中的Student模型类，[序列化-Serializer](./4序列化-SErializer.md)**

demo/serializers.py

```python
from rest_framework import serializers
from sers.models import Student


class StudentModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = Student
        fields = "__all__"
        extra_kwargs = {
            "age": {
                "max_value": 100,
                "error_messages": {
                    "max_value": "年龄不能超过100岁！",
                }
            }
        }
```

#### 1.2、定义视图

```python
from rest_framework import status
from rest_framework.views import APIView
from rest_framework.response import Response
from sers.models import Student
from .serializers import StudentModelSerializer
"""
GET  /demo/students/   获取所有学生信息
POST /demo/students/   添加一个学生信息

GET  /demo/students/<pk>   获取一个学生信息
PUT  /demo/students/<pk>   更新一个学生信息
DELETE  /demo/students/<pk>   删除一个学生信息
"""


# Create your views here.
class StudentAPIView(APIView):
    def get(self,request):
        """获取所有学生信息"""
        # 1. 从数据库中读取学生列表信息
        student_list = Student.objects.all()
        # 2. 实例化序列化器，获取序列化对象
        serializer = StudentModelSerializer(instance=student_list, many=True)

        # 3. 转换数据并返回给客户端
        return Response(serializer.data)

    def post(self,request):
        """添加一条数据"""
        # 1. 获取客户端提交的数据，实例化序列化器，获取序列化对象
        serializer = StudentModelSerializer(data=request.data)

        # 2. 反序列化[验证数据、保存数据到数据库]
        serializer.is_valid(raise_exception=True)
        serializer.save()

        # 3. 返回新增的模型数据给客户单
        return Response(serializer.data, status=status.HTTP_201_CREATED)


class StudentInfoAPIView(APIView):
    def get(self,request, pk):
        """获取一条数据"""
        # 1. 使用pk作为条件获取模型对象
        try:
            student = Student.objects.get(pk=pk)
        except Student.DoesNotExist:
            return Response(status=status.HTTP_404_NOT_FOUND)

        # 2. 序列化
        serializer = StudentModelSerializer(instance=student)

        # 3. 返回结果
        return Response(serializer.data)

    def put(self,request,pk):
        """更新数据"""
        # 1. 使用pk作为条件获取模型对象
        try:
            student = Student.objects.get(pk=pk)
        except Student.DoesNotExist:
            return Response(status=status.HTTP_404_NOT_FOUND)

        # 2. 获取客户端提交的数据
        serializer = StudentModelSerializer(instance=student, data=request.data)

        # 3. 反序列化[验证数据和数据保存]
        serializer.is_valid(raise_exception=True)
        serializer.save()

        # 4. 返回结果
        return Response(serializer.data, status=status.HTTP_201_CREATED)

    def delete(self,request,pk):
        """删除数据"""
        # 1. 根据PK值获取要删除的数据并删除
        try:
            Student.objects.get(pk=pk).delete()
        except Student.DoesNotExist:
            pass

        # 2. 返回结果
        return Response(status=status.HTTP_204_NO_CONTENT)

```

#### 1.3、定义路由

```python
from django.urls import path, re_path
from . import views

urlpatterns = [
    path("students/", views.StudentAPIView.as_view()),
    re_path("^students/(?P<pk>\d+)/$", views.StudentInfoAPIView.as_view()),
]
```



### 2、GenericAPIView [通用视图类]

APIView中的api接口代码，除了部分涉及到调用模型和序列化器的代码以外，其他代码几乎都是固定写法。

所以，当我们将来针对增删查改的通用api接口编写时，完全可以基于原有的代码进行复用，

那么，drf也考虑到了这个问题，所以提供了一个GenericAPIView（通用视图类），让我们可以把接口中独特的代码单独提取出来作为属性存在。

rest_framework.generics.GenericAPIView是APIView的子类，在APIView的基础上进行属性扩展提供了2个属性，4个方法，方便我们针对通用接口进行编写。

通用视图类主要作用就是把视图中的**独特的代码抽取出来**，让视图方法中的代码更加通用，方便把通用代码进行简写。

```
rest_framework.generics.GenericAPIView
```

继承自`APIView`，**主要增加了操作序列化器和数据库查询的方法，作用是为下面Mixin扩展类的执行提供方法支持。通常在使用时，可搭配一个或多个Mixin扩展类。**

**提供的关于序列化器使用的属性与方法**

- 属性：

  - **serializer_class** 指明视图使用的序列化器类

- 方法：

  - **get_serializer_class(self)**

    当出现一个视图类中调用多个序列化器时,那么可以通过条件判断在get_serializer_class方法中通过返回不同的序列化器类名就可以让视图方法执行不同的序列化器对象了。

    返回序列化器类，默认返回`serializer_class`，可以重写

  - ##### get_serializer(self, *args, \**kwargs)

    返回序列化器对象，主要用来提供给Mixin扩展类使用，如果我们在视图中想要获取序列化器对象，也可以直接调用此方法。

    **注意，该方法在提供序列化器对象的时候，会向序列化器对象的context属性补充三个数据：request、format、view，这三个数据对象可以在定义序列化器时使用。**

    - **request** 当前视图的请求对象
    - **view** 当前请求的类视图对象
    - format 当前请求期望返回的数据格式

**提供的关于数据库查询的属性与方法**

- 属性：

  - **queryset** 指明使用的数据查询集

- 方法：

  - **get_queryset(self)**

    返回视图使用的查询集，主要用来提供给Mixin扩展类使用，是列表视图与详情视图获取数据的基础，默认返回`queryset`属性，可以重写，例如：

    ```python
    def get_queryset(self):
        user = self.request.user
        return user.accounts.all()
    ```

  - **get_object(self)**

    返回详情视图所需的模型类数据对象，主要用来提供给Mixin扩展类使用。

    在试图中可以调用该方法获取详情信息的模型类对象。

    **若详情访问的模型类对象不存在，会返回404。**

    该方法会默认使用APIView提供的check_object_permissions方法检查当前对象是否有权限被访问。

    举例：

    ```python
    # url(r'^books/(?P<pk>\d+)/$', views.BookDetailView.as_view()),
    class BookDetailView(GenericAPIView):
        queryset = BookInfo.objects.all()
        serializer_class = BookInfoSerializer
    
        def get(self, request, pk):
            book = self.get_object() # get_object()方法根据pk参数查找queryset中的数据对象
            serializer = self.get_serializer(book)
            return Response(serializer.data)
    ```

**其他可以设置的属性**

- **pagination_class** 指明分页控制类
- **filter_backends** 指明数据过滤控制后端

#### 1、定义序列化器

沿用上面的序列化器代码即可，不需要再次创建

```python
from rest_framework import serializers
from sers.models import Student


class StudentModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = Student
        fields = "__all__"
        extra_kwargs = {
            "age": {
                "max_value": 100,
                "error_messages": {
                    "max_value": "年龄不能超过100岁！",
                }
            }
        }
```

#### 2、定义视图

```python
from rest_framework import status
from rest_framework.views import APIView
from rest_framework.response import Response
from stuapi.models import Student
from .serializers import StudentModleSerializer
from rest_framework.generics import GenericAPIView


"""
GET  /demo/students/   获取所有学生信息
POST /demo/students/   添加一个学生信息

GET  /demo/students/<pk>   获取一个学生信息
PUT  /demo/students/<pk>   更新一个学生信息
DELETE  /demo/students/<pk>   删除一个学生信息
"""

# 通用视图类
class StudentGenericApiView(GenericAPIView):
    queryset = Student.objects.all()
    serializer_class = StudentModleSerializer

    def get(self, request):
        """获取所有数据"""
        # 1、从数据库中读取模型列表信息
        instance = self.get_queryset()
        # 2. 序列化
        serializers = self.get_serializer(instance=instance, many=True)
        # 3. 转换数据并返回给客户端
        return Response(serializers.data)

    def post(self, request):
        """添加数据"""
        # 1、获取用户提交的数据
        serializer = self.get_serializer(data=request.data)
        # 2. 反序列化
        serializer.is_valid(raise_exception=True)
        serializer.save()
        # 3. 转换数据并返回给客户端
        return Response(serializer.data, status=status.HTTP_201_CREATED)


# 通用视图类
class StudentInfoGenericApiView(GenericAPIView):
    queryset = Student.objects.all()
    serializer_class = StudentModleSerializer

    def get(self, request, pk):
        """获取一个数据"""
        # 1. 使用pk作为条件获取模型对象
        instance = self.get_object()
        # 2.序列化
        serializer = self.get_serializer(instance=instance)
        # 返回数据
        return Response(data=serializer.data, status=status.HTTP_200_OK)

    def put(self, request, pk):
        """修改数据"""
        # 数据库数据实例
        instance = self.get_object()
        # 用户提交数据
        data = request.data
        # 反序列化
        serializer = self.get_serializer(instance=instance, data=data)
        # 数据验证
        serializer.is_valid(raise_exception=True)
        # 数据保存
        serializer.save()
        # 返回结果
        return Response(serializer.data, status=status.HTTP_200_OK)


    def delete(self, request, pk):
        """删除数据"""
        self.get_object().delete()

        return Response(status=status.HTTP_204_NO_CONTENT)
```

#### 3、定义路由

```
from django.urls import path, re_path
from . import views

urlpatterns = [

    # GenericAPIView
    path("students_generic/", views.StudentGenericApiView.as_view()),
    re_path("^students_generic/(?P<pk>\d+)/$", views.StudentInfoGenericApiView.as_view()),
]
```



### 3、5个视图扩展类

也叫混入类。

作用：

提供了几种后端视图（对数据资源进行增删改查）处理流程的实现，如果需要编写的视图属于这五种，则视图可以通过继承相应的扩展类来复用代码，减少自己编写的代码量。

这五个扩展类需要搭配GenericAPIView通用视图基类，因为五个扩展类的实现需要调用GenericAPIView提供的序列化器与数据库查询的方法。

#### 1）ListModelMixin

列表视图扩展类，提供`list(request, *args, **kwargs)`方法快速实现列表视图，返回200状态码。

该Mixin的list方法会对数据进行过滤和分页。

#### 2）CreateModelMixin

创建视图扩展类，提供`create(request, *args, **kwargs)`方法快速实现创建资源的视图，成功返回201状态码。

如果序列化器对前端发送的数据验证失败，返回400错误。

#### 3）RetrieveModelMixin

详情视图扩展类，提供`retrieve(request, *args, **kwargs)`方法，可以快速实现返回一个存在的数据对象。

如果存在，返回200， 否则返回404。

#### 4）UpdateModelMixin

更新视图扩展类，提供`update(request, *args, **kwargs)`方法，可以快速实现更新一个存在的数据对象。

同时也提供`partial_update(request, *args, **kwargs)`方法，可以实现局部更新。

成功返回200，序列化器校验数据失败时，返回400错误。

#### 5）DestroyModelMixin

删除视图扩展类，提供`destroy(request, *args, **kwargs)`方法，可以快速实现删除一个存在的数据对象。

成功返回204，不存在返回404。



#### 6）Mixins扩展类的整体代码实例

整体代码，使用GenericAPIView结合视图扩展类，实现5个基本api接口

视图代码：

```python
"""
使用drf内置的模型扩展类[混入类]结合GenericAPIView实现通用视图方法的简写操作
from rest_framework.mixins import ListModelMixin   获取多条数据，返回响应结果    list
from rest_framework.mixins import CreateModelMixin 添加一条数据，返回响应结果    create
from rest_framework.mixins import RetrieveModelMixin 获取一条数据，返回响应结果  retrieve
from rest_framework.mixins import UpdateModelMixin 更新一条数据，返回响应结果    update(更新全部字段)和partial_update(更新单个或部分字段，例如修改密码，修改头像)
from rest_framework.mixins import DestroyModelMixin 删除一条数据，返回响应结果   destroy
"""
from rest_framework.mixins import ListModelMixin, CreateModelMixin


class StudentMixinView(GenericAPIView, ListModelMixin, CreateModelMixin):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer
    def get(self,request):
        """获取所有数据"""
        return self.list(request)

    def post(self,request):
        """添加一条数据"""
        return self.create(request)


from rest_framework.mixins import RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin

class StudentInfoMixinView(GenericAPIView, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer

    def get(self,request,pk):
        return self.retrieve(request, pk=pk)

    def put(self,request, pk):
        return self.update(request, pk=pk)

    def delete(self,request, pk):
        return self.destroy(request, pk=pk)

```

序列化器，代码：

```python
from rest_framework import serializers
from stuapi.models import Student


class StudentModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = Student
        fields = "__all__"
        extra_kwargs = {
            "age": {
                "max_value": 25,
                "error_messages": {
                    "max_value": "年龄不能超过25岁！",
                }
            }
        }
```

路由代码：

```python
from django.urls import path, re_path
from . import views

urlpatterns = [

    # GenericAPIView + mixins
    path("students3/", views.StudentMixinView.as_view()),
    re_path("^students3/(?P<pk>\d+)/$", views.StudentInfoMixinView.as_view()),
]

```



## 二、视图集 ViewSet

使用视图集ViewSet，可以将一系列逻辑相关的动作放到一个类中：

- list() 提供一组数据
- retrieve() 提供单个数据
- create() 创建数据
- update() 保存数据
- destory() 删除数据

ViewSet视图集类不再限制视图方法名只允许get()、post()等这种情况了，而是实现允许开发者根据自己的需要定义自定义方法名，例如  list() 、create() 等，然后经过路由中使用http和这些视图方法名进行绑定调用。

视图集只在使用as_view()方法的时候，才会将**action**动作与具体请求方式对应上。如：

视图代码：

```python
from rest_framework.viewsets import ViewSet

class StudentViewSet(ViewSet):
    def get_student_list(self, request):
        """获取所有学生信息"""
        instance = Student.objects.all()
        serializer = StudentModleSerializer(instance=instance, many=True)
        return Response(serializer.data, status=status.HTTP_200_OK)
```

在设置路由时，我们可以如下操作

```python
from django.urls import path, re_path
from . import views

urlpatterns = [
    # ViewSet
    path("student_viewset/", views.StudentViewSet.as_view({"get": "get_student_list"})),
]
```



### 常用视图集父类

#### 1） ViewSet

继承自`APIView`与`ViewSetMixin`，作用也与APIView基本类似，提供了身份认证、权限校验、流量管理等。

**ViewSet主要通过继承ViewSetMixin来实现在调用as_view()时传入字典{“http请求”：“视图方法”}的映射处理工作，如{'get':'list'}，**

在ViewSet中，没有提供任何动作action方法，需要我们自己实现action方法。

#### 2）GenericViewSet

继承自GenericAPIView和ViewSetMixin，作用让视图集的视图代码变得更加通用，抽离独特代码作为视图类的属性。

使用ViewSet通常并不方便，因为list、retrieve、create、update、destory等方法都需要自己编写，而这些方法与前面讲过的Mixin扩展类提供的方法同名，所以我们可以通过继承Mixin扩展类来复用这些方法而无需自己编写。但是Mixin扩展类依赖与`GenericAPIView`，所以还需要继承`GenericAPIView`。

**GenericViewSet**就帮助我们完成了这样的继承工作，继承自`GenericAPIView`与`ViewSetMixin`，在实现了调用as_view()时传入字典（如`{'get':'list'}`）的映射处理工作的同时，还提供了`GenericAPIView`提供的基础方法，可以直接搭配Mixin扩展类使用。

视图代码：

```python
"""GenericViewSet 通用视图集"""
from rest_framework.viewsets import GenericViewSet


class StudentGenericViewSet(GenericViewSet):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer

    def list(self, request):
        """获取所有数据"""
        # 1. 从数据库中读取模型列表信息
        queryset = self.get_queryset()  # GenericAPIView提供的get_queryset
        # 2. 序列化
        serializer = self.get_serializer(instance=queryset, many=True)

        # 3. 转换数据并返回给客户端
        return Response(serializer.data)


    def create(self,request):
        """添加一个数据"""
        # 1. 获取客户端提交的数据，实例化序列化器，获取序列化对象
        serializer = self.get_serializer(data=request.data)

        # 2. 反序列化[验证数据、保存数据到数据库]
        serializer.is_valid(raise_exception=True)
        serializer.save()

        # 3. 返回新增的模型数据给客户单
        return Response(serializer.data, status=status.HTTP_201_CREATED)

```

路由代码：

```python
from django.urls import path, re_path
from . import views

urlpatterns = [
    # 视图集: GenericViewSet
    path("students6/", views.StudentGenericViewSet.as_view({
        "get": "list",
        "post": "create",
    })),

    re_path("^students6/(?P<pk>\d+)/$", views.StudentGenericViewSet.as_view({
        "get": "retrieve",
        "put": "update",
        "delete": "destroy",
    })),

]
```

#### 3）ReadOnlyModelViewSet

继承自`GenericViewSet`，同时包括了ListModelMixin、RetrieveModelMixin。

#### 4）ModelViewSet

继承自`GenericViewSet`，同时包括了ListModelMixin、RetrieveModelMixin、CreateModelMixin、UpdateModelMixin、DestoryModelMixin。

!> 视图代码最终版，注意，是视图代码

视图代码：

```python
"""
GenericViewSet结合Mixins的混入类，直接视图接口，这次连视图子类都不需要了。
ViewSet
GenericViewSet
ModelViewSet = GenericViewSet + ListModelMixin + CreateModelMixin + UpdateModelMixin + RetrieveModelMixin + DestroyModelMixin
ReadOnlyModelViewSet = GenericViewSet + ListModelMixin + RetrieveModelMixin
"""
class StudentMixinViewSet(ModelViewSet):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer
```

路由代码：

```python
from django.urls import path, re_path
from . import views

urlpatterns = [    
    # 视图集: ModelViewSet
    path("students7/", views.StudentModelViewSet.as_view({
        "get": "list",
        "post": "create",
    })),

    re_path("^students7/(?P<pk>\d+)/$", views.StudentModelViewSet.as_view({
        "get": "retrieve",
        "put": "update",
        "delete": "destroy",
    })),
]
```

> 写完路由后会发现，视图代码是简写了，但是路由代码变多了
>
> 这个时候就需要用到下边的 Douters



## 三、路由 Routers

对于视图集ViewSet，我们除了可以自己手动指明请求方式与动作action之间的对应关系外，还可以使用Routers来帮助我们快速实现路由信息。如果是非视图集，不需要使用路由集routers

REST framework提供了两个router类，使用方式一致的。结果多一个或少一个根目录url地址的问题而已。

- **SimpleRouter**
- **DefaultRouter**

### 3.1、使用方法

1） 创建router对象，并注册视图集，例如

```python
from . import views
from rest_framework.routers import DefaultRouter

urlpatterns = []
router = DefaultRouter()
router.register("student_url", views.StudentModleViewSet, "student_url")
urlpatterns += router.urls
```

register(prefix, viewset, basename)

- prefix 该视图集的路由前缀
- viewset 视图集
- basename 路由别名的前缀

如上述代码会形成的路由如下：

```
'^student_url/$' basename='student_url-list'
'^student_url/(?P<pk>[^/.]+)/$' basename='^student_url/(?P<pk>[^/.]+)/$'
```

2）把路由对象生成的视图集的路由列表添加到django的路由中可以有两种方式：

```python
urlpatterns = [
    ...
]

urlpatterns += router.urls
```

或

```python
from django.urls import include,re_path
urlpatterns = [
    ...
    re_path('^', include(router.urls))
]
```



### 3.2 视图集中附加action的声明

在视图集中，如果想要让Router自动帮助我们为自定义的动作生成路由信息，需要使用`rest_framework.decorators.action`装饰器。

以action装饰器装饰的方法名会作为action动作名，与list、retrieve等同。

action装饰器可以接收两个参数：

- **methods**: 声明该action对应的请求方式，列表参数

- **detail**: 声明该action的路径是否与单一资源对应

  ```
  路由前缀/<pk>/action方法名/
  ```

  - True 表示路径格式是`xxx/<pk>/action方法名/`
  - False 表示路径格式是`xxx/action方法名/`

- url_path：声明该action的路由尾缀。

举例：

```python
from rest_framework.viewsets import ModelViewSet
from rest_framework.decorators import action

class StudentModelViewSet(ModelViewSet):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer


    # 路由对象给视图集生成路由信息时，只会生成5个基本api接口，这主要是router只识别5个混入类的原因，
    # 而针对我们开发者自定义的视图方法，路由对象不会自动生成路由信息，
    # 所以下面这个login，如果希望被外界访问到，则必须通过action装饰器告诉路由对象要给它生成一个路由信息。
    @action(methods=["get","post"], detail=False, url_path="login")
    # action的参数
    # methods, 列表，指定允许哪些http请求方法可以访问当前视图方法
    # detail, 布尔值，告诉路由对象在生成路由信息时，是否要自动生成pk值，True表示需要，False表示不需要。
    # url_path，字符串，访问视图的url地址，如果不设置，则默认采用视图方法名作为访问后缀
    # http://127.0.0.1:8000/demo/student9/login/
    def login(self, request):
        """登录视图"""
        return Response({"msg":"登录成功"})


    @action(methods=["get"], detail=True, url_path="login/log")
    # http://127.0.0.1:8000/demo/student9/20/login/log/
    def login_log(self,request,pk):
        """用户登录历史记录"""
        # 视图集类中比普通的视图类，多一个属性，action
        # 可以通过self.method获取本次客户端的http请求
        # 可以通过self.action获取本次客户端请求的视图方法名[ViewSet提供的]
        print(self.action) # login_log
        return Response({"msg": "用户登录历史记录"})
```

由路由器自动为此视图集自定义action方法形成的路由会是如下内容：

```
url: ^student9/login/$                      basename: student9-login
url: ^student9/(?P<pk>[^/.]+)/login/log/$   basename: student9-login-log
```

### 3.3、路由router形成URL的方式

1） SimpleRouter（prefix=“路由前缀”，viewset=视图集类，basename=“路由别名”）

![SimpleRouter](7drf%E8%A7%86%E5%9B%BE%E5%AE%B6%E6%97%8F.assets/SimpleRouter.png)

2）DefaultRouter

![DefaultRouter](7drf%E8%A7%86%E5%9B%BE%E5%AE%B6%E6%97%8F.assets/DefaultRouter.png)

DefaultRouter与SimpleRouter的区别是，DefaultRouter会多附带一个默认的API根视图，返回一个包含所有列表视图的超链接响应数据。











