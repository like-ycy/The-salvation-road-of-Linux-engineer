# DRF 异常处理 Exceptions

默认情况下，DRF框架通过内置的 `exception_handler` 方法，处理了如下异常：

- django内置异常
  - Http404
  - PermissionDenied
- DRF框架异常
  - APIException - DRF 框架异常的父类，以下为它的子类
    - NotFound 未找到
    - PermissionDenied 权限拒绝
    - ParseError 解析错误
    - AuthenticationFailed 认证失败
    - NotAuthenticated 尚未认证
    - MethodNotAllowed 请求方式不支持
    - Throttled 超过限流次数
    - ValidationError 校验失败
    - NotAcceptable 要获取的数据格式不支持
- 如果 `exception_handler` 方法处理异常成功，则返回一个Response对象，否则返回`None`

如果发生了上面的这些异常，DRF框架的excaption_handler函数可以自动捕获异常，然后进行处理，返回Response对象。如果不是上面的这些异常，则处理不了，返回None。

也就是说，很多的没有在上面列出来的异常，就需要我们在自定义异常中自己处理了。



REST framework本身在APIView提供了异常处理，但是仅针对drf内部现有的接口开发相关的异常进行格式处理.

但是开发中我们还会使用到各种的数据或者进行各种网络请求，这些都有可能导致出现异常，这些异常在drf中是没有进行处理的，所以就会冒泡给django框架了。django框架会进行组织错误信息，作为html页面返回给客户端。

所在在前后端分离项目中，可能js无法理解或者无法接收到这种数据，甚至导致js出现错误的情况。因此为了避免出现这种情况，我们可以自定义一个属于自己的异常处理函数，对于drf无法处理的异常，我们自己编写异常处理的代码逻辑。

针对于现有的drf的异常处理进行额外添加属于开发者自己的逻辑代码，一般我们编写的异常处理函数，会写一个公共的目录下或者主应用目录下。这里主应用下直接创建一个excepitions.py，代码：

```python
from rest_framework.views import exception_handler
from rest_framework.response import Response
from rest_framework import status


# 针对mysql、mongoDB、redis、第三方数据接口
def custom_exception_handler(exc, context):
    """
    自定义异常函数
    exc: 异常实例对象，发生异常时实例化出来的
    context: 字典，异常发生时python解释器收集的执行上下文信息。
             所谓的执行上下文就是python解释器在执行代码时保存在内存中的变量、函数、类、对象、模块等一系列的信息组成的环境信息。
    """
    response = exception_handler(exc, context)

    if response is None:
        """当前异常，drf无法处理"""
        if isinstance(exc, ZeroDivisionError):
            response = Response({"detail":"数学老师还有30秒达到战场，0不能作为除数！"}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)
    return response
```

在配置文件中声明自定义的异常处理，`settings`，代码：

```python
REST_FRAMEWORK = {
    # 异常配置
    'EXCEPTION_HANDLER': 'drfdemo.exceptions.custom_exception_handler'
}
```

如果未声明自定义异常的话，drf会采用默认的方式，使用自己封装的异常处理函数，rest_frame/settings.py

```python
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'rest_framework.views.exception_handler'
}
```

例如：

补充上处理关于数据库的异常，这里使用其他异常来举例：

`主应用.exceptions`，代码：

```python
from rest_framework.views import exception_handler
from rest_framework.response import Response
from rest_framework import status
from django.db import DatabaseError
# 针对mysql、mongoDB、redis、第三方数据接口
def custom_exception_handler(exc, context):
    """
    自定义异常函数
    exc: 异常实例对象，发生异常时实例化出来的
    context: 字典，异常发生时python解释器收集的执行上下文信息。
             所谓的执行上下文就是python解释器在执行代码时保存在内存中的变量、函数、类、对象、模块等一系列的信息组成的环境信息。
    """
    response = exception_handler(exc, context)
    print(f"context={context}")
    if response is None:
        """当前异常，drf无法处理"""
        view = context["view"] # 获取异常发生时的视图类
        request = context["request"] # 获取异常发生时的客户端请求对象
        if isinstance(exc, ZeroDivisionError):
            response = Response({"detail":"数学老师还有30秒达到战场，0不能作为除数！"}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

        if isinstance(exc, TypeError):
            print('[%s]: %s' % (view, exc))
            response = Response({'detail': '服务器内部错误'}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

    return response
```

视图中，故意报错：

```python
from rest_framework.pagination import LimitOffsetPagination, PageNumberPagination

class Hom5PageNumberPagination(PageNumberPagination):
    page_size = 10                  # 默认分页的每一页数据量
    max_page_size = 20              # 设置允许客户端通过地址栏参数调整的最大单页数据量
    page_query_param = "pn"         # 地址栏上代表页码的变量名，默认是 page
    page_size_query_param = "size"  # 地址栏上代表单页数据量的变量名，默认是page_size

class Hom5APIView(ModelViewSet):
    queryset = Student.objects  # 这句话在没有进行分页时不会报错，调用了分页则会报错，所以会进入异常处理！
    # queryset = Student.objects.all()
    serializer_class = StudentModelSerializer

    # 局部分页
    # pagination_class = None  # 关闭分页功能
    # 局部分页往往采用自定义分页类，进行分页数据量的配置
    pagination_class = Hom5PageNumberPagination

    def list(self, request, *args, **kwargs):
        # 获取django的配置项
        from django.conf import settings

        # 获取rest_framework的配置项
        # from rest_framework.settings import api_settings
        # print(api_settings.DEFAULT_PAGINATION_CLASS)
        return super().list(request, *args, **kwargs)
```

