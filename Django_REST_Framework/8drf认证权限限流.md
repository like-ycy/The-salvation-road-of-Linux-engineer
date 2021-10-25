# DRF 认证 权限 限流

为了方便接下来的学习，我们创建一个新的子应用 opt

```python
python manage.py startapp opt
```

注册子应用

```python
INSTALLED_APPS = [
    'opt',     # drf提供的组件使用
]
```

总路由，代码：

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("opt/", include("opt.urls")),
]

```

因为接下来的认证组件中需要使用到登陆功能，所以我们使用django内置admin站点并创建一个管理员。

admin运营站点的访问地址：http://127.0.0.1:8000/admin

```
python manage.py createsuperuser
# 如果之前有账号，但是忘了，可以通过终端下的命令修改指定用户的密码，这里的密码必须8位长度以上的
python manage.py changepassword 用户名
```

其他Admin站点相关操作可已查看 [Django高级应用](/Django/9django高级应用.md)



## 1、认证Authentication

**全局配置：**

可以在配置文件中配置全局默认的认证方案

常见的认证方式：cookie、session、token

/site-packages/rest_framework/settings.py 默认配置文件

```python
REST_FRAMEWORK = {
    # 配置认证方式的选项
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.SessionAuthentication', # session认证
        'rest_framework.authentication.BasicAuthentication',   # basic认证
    )
}
```

**局部配置：**

也可以在具体的视图类中通过设置authentication_classess类属性来设置单独的不同的认证方式

```python
from rest_framework.authentication import SessionAuthentication, BasicAuthentication
from rest_framework.views import APIView

class ExampleView(APIView):
    # 类属性
    authentication_classes = [SessionAuthentication, BasicAuthentication]
    def get(self, request):
        if request.user.id:
            print("通过认证")
        else:
            print("未通过认证")
        return Response({"msg": "ok"})
```

认证失败会有两种可能的返回值，这个需要我们配合权限组件来使用：

- 401 Unauthorized 未认证
- 403 Permission Denied 权限被禁止

### 自定义认证类的实现

authentication.py

```python
from rest_framework.authentication import BaseAuthentication
from django.contrib.auth import get_user_model


class CustomAuthentication(BaseAuthentication):
    """
    自定义认证方式
    """
    def authenticate(self, request):
        """
        认证方法
        :param request:客户端发送过来的请求
        :return:返回（用户模型对象，None）
        """
        user = request.query_params.get('user')
        pwd = request.query_params.get('pwd')
        if user != 'root' and pwd != '123456':
            return None
        # get_user_model获取当前系统中用户表对应的用户模型类
        user = get_user_model().objects.first()
        return (user, None)
```

视图，代码：

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from opt.authentication import CustomAuthentication


class HomeAPIView(APIView):
    authentication_classes = [CustomAuthentication]

    def get(self, request):
        """单独设置认证模式"""
        print(request.user)  # 未登录用户：AnonymousUser
        if request.user.id:
            print("通过认证")
        else:
            print("未通过认证")
        return Response({"msg": "ok"})

```

当然，也可以注释掉上面视图中的`authentication_classes`配置，改成全局配置。settings.py，代码：

```python
"""drf配置信息必须全部写在REST_FRAMEWORK配置项中"""
REST_FRAMEWORK = {
    # 配置认证方式的选项【drf的认证是内部循环遍历每一个注册的认证类，一旦认证通过识别到用户身份，则不会继续循环】
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'opt.authentication.CustomAuthentication',          # 自定义认证
        'rest_framework.authentication.SessionAuthentication',  # session认证
        'rest_framework.authentication.BasicAuthentication',    # 基本认证
    )
}
```

路由，代码

```python
from django.urls import path
from opt import views

urlpatterns = [
    path("example/", views.ExampleView.as_view()),
    path("home_apiview/", views.HomeAPIView.as_view()),
]
```

验证方式可以使用，登录、注销admin站点的当时去验证；也可以使用在地址栏中添加参数的方式，如：`http://127.0.0.1:8000/opt/home_apiview/?user=root&pwd=123456`

然后根据运行日志判断代码是否走了自定义的认证类

## 2、权限Permissions

权限控制可以限制用户对于视图的访问和对于具有模型对象的访问。

- 在执行视图的as_view()方法的dispatch()方法前，会先进行视图访问权限的判断
- 在通过get_object()获取具体模型对象时，会进行模型对象访问权限的判断

### 提供的权限

- AllowAny 允许所有用户，默认权限
- IsAuthenticated 仅通过登录认证的用户
- IsAdminUser 仅管理员用户
- IsAuthenticatedOrReadOnly 已经登陆认证的用户可以对数据进行增删改操作，没有登陆认证的只能查看数据。

### 使用

默认配置 rest_framework/settings.py， **允许所有**

```python
'DEFAULT_PERMISSION_CLASSES': (
   'rest_framework.permissions.AllowAny',
)
```

可以在配置文件settings中**全局设置**默认的权限管理类，如

```python
REST_FRAMEWORK = {
    ....
    
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    )
}
```

也可以在具体的视图中通过permission_classes属性来进行**局部设置**，如

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from opt.authentication import CustomAuthentication
from rest_framework.permissions import AllowAny,IsAuthenticated


class HomeAPIView(APIView):
    # authentication_classes = [CustomAuthentication, ]
    # permission_classes = [AllowAny]       # 如果项目中大部分视图接口不许直接访问，少数接口可以直接访问，则可以全局配置权限认证，仅在部分不需要认证的视图类中加上AllowAny即可
    permission_classes = [IsAuthenticated]  # 如果项目中大部分视图接口允许直接访问，少数接口必修认证权限，则可以不要权限认证的全局配置，仅在需要认证权限的视图下，局部配置IsAuthenticated即可
    def get(self,request):
        """单独设置认证方式"""
        print(request.user) # 在中间件AuthenticationMiddleware中完成用户身份识别的，如果没有登录request.user值为AnonymousUser
        if request.user.id is None:
            return Response("未登录用户：游客")
        else:
            return Response(f"已登录用户：{request.user}")
```



### 举例

视图代码：

```python
from rest_framework.viewsets import ModelViewSet
from school.models import Student
from school.serializers import StudentModelSerializer
from rest_framework.permissions import IsAuthenticatedOrReadOnly

class HomAPIView3(ModelViewSet):
    authentication_classes = [CustomAuthentication, ]
    # 游客仅能查看数据，登录用户可以进行数据的修改/添加/删除
    permission_classes = [IsAuthenticatedOrReadOnly]
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer
```

路由代码：

```python
from django.urls import path
from . import views
urlpatterns = [
     path("example/", views.ExampleView.as_view()),
     path("home_apiview/", views.HomeAPIView.as_view()),
     path("home_apiview2/", views.HomeAPIView2.as_view()),
]

from rest_framework.routers import DefaultRouter
router = DefaultRouter()
router.register("home_apiview3", views.Hom2APIView, "home_apiview3")
urlpatterns += router.urls

```

### 自定义权限

如需自定义权限，需继承rest_framework.permissions.BasePermission父类，并实现以下两个任何一个方法或全部

- `.has_permission(self, request, view)`

  是否可以访问视图， view表示当前视图对象

- `.has_object_permission(self, request, view, obj)`

  是否可以访问模型对象， view表示当前视图， obj为模型数据对象

例如：

在当前子应用下，创建一个权限文件permissions.py中声明自定义权限类:

```python
from rest_framework.permissions import BasePermission

class IsXiaoMingPermission(BasePermission):
    """
    自定义权限，可用于全局配置，也可以用于局部配置
    """

    def has_permission(self, request, view):
        """
        视图权限
        返回结果未True则表示允许访问视图类
        request: 本次客户端提交的请求对象
        view: 本次客户端访问的视图类
        """
        # # 写在自己要实现认证的代码过程。
        # user = request.query_params.get("user")
        # # 返回值为True，则表示通行
        # return user == "xiaoming"
        return True

    def has_object_permission(self, request, view, obj):
        """
        模型权限，写了视图权限(has_permission)方法，一般就不需要写这个了。
        返回结果未True则表示允许操作模型对象
        """
        from school.models import Student
        if isinstance(obj, Student):
            # 限制只有小明才能操作Student模型
            user = request.query_params.get("user")
            return user == "xiaoming"  # 如果不是xiaoming，返回值为False，不能操作
        else:
            # 操作其他模型，直接放行
            return True
```

视图代码：

```python
from rest_framework.viewsets import ModelViewSet
from school.models import Student
from school.serializers import StudentModelSerializer
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from opt.permissions import IsXiaoMingPermission
class HomeAPIView4(ModelViewSet):
    # authentication_classes = [CustomAuthentication, ]
    # 游客仅能查看数据，登录用户可以进行数据的修改/添加/删除
    # permission_classes = [IsAuthenticatedOrReadOnly]

    # 自定义权限
    permission_classes = [IsXiaoMingPermission]
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer
```

认证和权限的举例代码：

settings.py，全局配置，代码：

```python
"""drf配置信息必须全部写在REST_FRAMEWORK配置项中"""
REST_FRAMEWORK = {
    # 配置认证方式的选项【drf的认证是内部循环遍历每一个注册的认证类，一旦认证通过识别到用户身份，则不会继续循环】
    'DEFAULT_AUTHENTICATION_CLASSES': (
        # 'drfdemo.authentication.CustomAuthentication',          # 自定义认证
        'rest_framework.authentication.SessionAuthentication',  # session认证
        'rest_framework.authentication.BasicAuthentication',    # 基本认证
    ),
    # 权限设置[全局配置，在视图中可以通过permission_classes进行局部配置，局部配置优先级高于全局配置]
    'DEFAULT_PERMISSION_CLASSES': (
        # 'rest_framework.permissions.IsAuthenticated',
        'drfdemo.permissions.IsXiaoMingPermission',
    ),
}
```

认证主要的作用就是识别客户端的访问者的身份，但是不能拦截客户端的访问。

权限是基于认证来实现的，但是权限可以针对不同身份的用户，进行拦截用户对视图、模型的访问操作。



## 3、限流Throttling

可以对接口访问的频次进行限制，以减轻服务器压力，或者实现特定的业务。

一般用于付费购买次数,投票等场景使用。

### 基本使用

可以在配置文件中，使用`DEFAULT_THROTTLE_CLASSES` 和 `DEFAULT_THROTTLE_RATES`进行全局配置，

```python
REST_FRAMEWORK = {
    # 限流全局配置
    # 'DEFAULT_THROTTLE_CLASSES':[ # 限流配置类
    #     'rest_framework.throttling.AnonRateThrottle', # 未认证用户[未登录用户]
    #     'rest_framework.throttling.UserRateThrottle', # 已认证用户[已登录用户]
    # ],
    'DEFAULT_THROTTLE_RATES':{ # 频率配置
        'anon': '2/day',  # 针对游客的访问频率进行限制，day可以简写为d，实际上，drf只是识别首字母，但是为了提高代码的维护性，建议写完整单词
        'user': '5/day', # 针对会员的访问频率进行限制，
    }
}
```

`DEFAULT_THROTTLE_RATES` 可以使用 `second`, `minute`, `hour` 或`day`来指明周期。

也可以在具体视图中通过throttle_classess属性来配置，如

```python
from rest_framework.throttling import UserRateThrottle
class Student2ModelViewSet(ModelViewSet):
    queryset = Student.objects
    serializer_class = StudentModelSerializer
    # 限流局部配置[这里需要配合在全局配置中的DEFAULT_THROTTLE_RATES来设置频率]
    throttle_classes = [UserRateThrottle]
```

### 可选限流类

1） AnonRateThrottle

限制所有匿名未认证用户，使用IP区分用户。【很多公司这样的，IP结合设备信息来判断，当然比IP要靠谱一点点而已】

使用`DEFAULT_THROTTLE_RATES['anon']` 来设置频次

2）UserRateThrottle

限制认证用户，使用User模型的 id主键 来区分。

使用`DEFAULT_THROTTLE_RATES['user']` 来设置频次

3）ScopedRateThrottle

限制用户对于每个视图的访问频次，使用ip或user id。

settings.py，代码：

```python
"""drf配置信息必须全部写在REST_FRAMEWORK配置项中"""
REST_FRAMEWORK = {
    # 配置认证方式的选项【drf的认证是内部循环遍历每一个注册的认证类，一旦认证通过识别到用户身份，则不会继续循环】
    'DEFAULT_AUTHENTICATION_CLASSES': (
        # 'drfdemo.authentication.CustomAuthentication',          # 自定义认证
        'rest_framework.authentication.SessionAuthentication',  # session认证
        'rest_framework.authentication.BasicAuthentication',    # 基本认证
    ),
    # 权限设置[全局配置，在视图中可以通过permission_classes进行局部配置，局部配置优先级高于全局配置]
    'DEFAULT_PERMISSION_CLASSES': (
        # 'rest_framework.permissions.IsAuthenticated',
        'drfdemo.permissions.IsXiaoMingPermission',
    ),
    # 限流全局配置
    'DEFAULT_THROTTLE_CLASSES':[  # 限流配置类
    #     'rest_framework.throttling.AnonRateThrottle',  # 未认证用户[未登录用户]
    #     'rest_framework.throttling.UserRateThrottle',  # 已认证用户[已登录用户]
        'rest_framework.throttling.ScopedRateThrottle',  # 基于自定义的命名空间来限流
    ],
    'DEFAULT_THROTTLE_RATES': {  # 频率配置
        'anon': '1/s',  # 针对游客的访问频率进行限制
        'user': '1/s',  # 针对会员的访问频率进行限制，
        'Hom3': '3/m',  # 针对自定义命名空间，进行限流
    }
}
```

视图代码：

```python
from rest_framework.throttling import AnonRateThrottle,UserRateThrottle
class HomeAPIView5(APIView):
    # authentication_classes = [CustomAuthentication]  # 调用自定义认证
    # 局部配置限流
    # throttle_classes = [AnonRateThrottle, UserRateThrottle]
    # 配置自定义限流
    throttle_scope = "Hom3"   # 自定义命名空间[少用，因为对于大部分的集体环境下都是公用一个IP地址]
    def get(self,request):
        return Response(f"访问了视图")
```



