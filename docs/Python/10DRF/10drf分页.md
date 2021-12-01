# DRF 分页器

## 分页Pagination

因为django默认提供的分页器主要使用于前后端不分离的业务场景，所以REST framework也提供了分页的支持。

我们可以在配置文件中设置全局的分页方式，如：

```python
REST_FRAMEWORK = {
    # 分页，全局配置
    
    # 页码分页器，  ?page=页码&page_size=单页数据量
    # 'DEFAULT_PAGINATION_CLASS':  'rest_framework.pagination.PageNumberPagination',
    
    # 偏移量分页器， ?limit=单页数据量&offset=数据开始下标
    'DEFAULT_PAGINATION_CLASS':  'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 10  # 每页数目，如果不设置，则没有进行分配
}
```

如果在settings.py配置文件中， 设置了全局分页，那么在drf中凡是调用了ListModelMixin的list()，都会自动分页。如果项目中出现大量需要分页的数据，只有少数部分的不需要分页，则可以在少部分的视图类中关闭分页功能。

另外，视图类在使用过分页以后，务必在编写queryset属性时，模型.objects后面调用结果。例如：


```python
# 
#Student.objects.all()
class Hom5APIView(ListAPIView):
	pagination_class = None
```

也可通过自定义Pagination类，来为视图添加不同分页行为。在视图中通过`pagination_clas`属性来指明。

## 可选分页器

1） **PageNumberPagination**

前端访问网址形式：

```http
GET  http://127.0.0.1:8000/students/?page=4
```

可以在子类中定义的属性：

- page_size 每页数目
- page_query_param 前端发送的页数关键字名，默认为"page"
- page_size_query_param 前端发送的每页数目关键字名，默认为None
- max_page_size 前端最多能设置的每页数量

分页器类，`paginations`，代码：

```python
from  rest_framework.pagination import PageNumberPagination,LimitOffsetPagination
# PageNumberPagination，以页码作为分页条件
# page=1&page_size=10      第1页
# page=2&page_size=10      第2页
# ...
# LimitOffsetPagination，以数据库查询的limit和offset数值作为分页条件
# limit=10&offset=0   第1页
# limit=10&offset=10  第2页
# ...

# 自定义分页器，PageNumberPagination
class StudentPageNumberPagination(PageNumberPagination):
    page_query_param = "page" # 查询字符串中代表页码的变量名
    page_size_query_param = "size" # 查询字符串中代表每一页数据的变量名
    page_size = 2 # 每一页的数据量
    max_page_size = 4 # 允许客户端通过查询字符串调整的最大单页数据量
```

视图，`views` ，代码：

```python
from rest_framework.pagination import LimitOffsetPagination, PageNumberPagination

# 分页类往往会单独的保存到一个独立的模块，例如：当前子应用目录下创建一个pagination.py保存，使用时导包
class Hom5PageNumberPagination(PageNumberPagination):
    page_size = 10                  # 默认分页的每一页数据量
    max_page_size = 20              # 设置允许客户端通过地址栏参数调整的最大单页数据量
    page_query_param = "pn"         # 地址栏上代表页码的变量名，默认是 page
    page_size_query_param = "size"  # 地址栏上代表单页数据量的变量名，默认是page_size

class Hom5APIView(ModelViewSet):
    # queryset = Student.objects  # 这句话在没有进行分页时不会报错，调用了分页则会报错！
    queryset = Student.objects.all()
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



2）**LimitOffsetPagination**

前端访问网址形式：

```http
GET http://127.0.0.1/four/students/?limit=100&offset=100
```

可以在子类中定义的属性：

- default_limit 默认限制，默认值与`PAGE_SIZE`设置一直
- limit_query_param limit参数名，默认'limit'
- offset_query_param offset参数名，默认'offset'
- max_limit 最大limit限制，默认None

分页类，代码：

```python
from  rest_framework.pagination import PageNumberPagination,LimitOffsetPagination
# PageNumberPagination，以页码作为分页条件
# page=1&size=10      第1页
# page=2&size=10      第2页
# LimitOffsetPagination，以数据库查询的limit和offset数值作为分页条件
# limit=10&offset=0   第1页
# limit=10&offset=10  第2页

# LimitOffsetPagination
class StudentLimitOffsetPagination(LimitOffsetPagination):
    limit_query_param = "limit" # 查询字符串中代表每一页数据的变量名
    offset_query_param = "offset" # 查询字符串中代表页码的变量名
    default_limit = 2 # 每一页的数据量
    max_limit = 4 # 允许客户端通过查询字符串调整的最大单页数据量
```

视图，`views`，代码：

```python
from .paginations import StudentPageNumberPagination,StudentLimitOffsetPagination
class Student3ModelViewSet(ModelViewSet):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializer
    # 取消当前视图类的分页效果
    # pagination_class = None
    # 局部分页
    pagination_class = StudentLimitOffsetPagination
```