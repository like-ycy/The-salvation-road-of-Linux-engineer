# DRF 请求与响应

内容协商：drf在django原有的基础上，新增了一个request对象继承到了APIVIew视图类，并在django原有的HttpResponse响应类的基础上实现了一个子类rest_framework.response.Response响应类。这两个类，都是基于内容协商来完成数据的格式转换的。

request->parser解析类->识别客户端请求头中的Content-Type来完成数据转换成->类字典(QueryDict，字典的子类)

response->renderer渲染类->识别客户端请求头的"Accept"来提取客户端期望的返回数据格式，-> 转换成客户端的期望格式数据

## 一、Request

REST framework 传入视图的request对象不再是Django默认的HttpRequest对象，而是REST framework提供的扩展了HttpRequest类的**Request**类的对象。

REST framework 提供了**Parser**解析器，在接收到请求后会自动根据Content-Type指明的请求数据类型（如JSON、表单等）将请求数据进行parse解析，解析为类字典[QueryDict]对象保存到**Request**对象中。

**Request对象的数据是自动根据前端发送数据的格式进行解析之后的结果。**

无论前端发送的哪种格式的数据，我们都可以以统一的方式读取数据。

### 1.1、常用属性

##### 1）.data

`request.data` 返回解析之后的**请求体**数据。类似于Django中标准的`request.POST`和 `request.FILES`属性，但提供如下特性：

- 包含了解析之后的文件和非文件数据
- 包含了对POST、PUT、PATCH请求方式解析后的数据
- 利用了REST framework的parser解析器，不仅支持表单类型数据，也支持JSON数据

##### 2）.query_params

query_params，查询参数，也叫查询字符串(query string )

`request.query_params`与Django标准的`request.GET`相同，只是更换了更正确的名称而已。

##### 3）request._request 

获取django封装的Request对象

### 1.2、使用

视图代码：

```python
from django.views import View
from django.http.response import HttpResponse
from django.http.request import HttpRequest
from django.core.handlers.wsgi import WSGIRequest
class ReqView(View):
    def get(self,request):
        print(request)
        return HttpResponse("ok")

```

默认情况下, 编写视图类时，如果继承的是django内置的django.view.View视图基类，则视图方法中得到的request对象，是django默认提供的django.core.handlers.wsgi.WSGIRequest。

WSGIRequest这个请求处理对象，无法直接提供的关于json数据数据处理。

在编写api接口时很不方便，所以drf为了简写这块内容，在原来的HttpRequest的基础上面，新增了一个Request对象。这个Request对象是单独声明的和原来django的HttpRequest不是父子关系。

**同时注意：**

要使用drf提供的Request请求处理对象，必须在编写视图类时继承drf提供的视图基类

from rest_framework.views import APIView

如果使用drf提供的视图基类APIView编写类视图，则必须使用来自drf提供的Request请求对象和Response响应对象


```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
class ReqAPIView(APIView):
    def get(self,request):
        # rest_framework.request.Request对象
        print(request) # <rest_framework.request.Request: GET '/req/req2?name=xiaoming&age=17&lve=swim&lve=code'>
        # 获取查询字符串
        print(request.query_params)
        # 没有参数情况下： <QueryDict: {}>
        # 有参数的情况下： <QueryDict: {'name': ['xiaoming'], 'age': ['17'], 'lve': ['swim', 'code']}>
        # 所以，request.query_params的返回值操作和原来在django里面是一模一样的
        print(request.query_params.get("name")) # xiaoming
        print(request.query_params.getlist("lve")) # ['swim', 'code']

        return Response("ok")

    def post(self, request):
        # 获取请求体
        print(request.data) # {'name': 'xiaoming', 'age': 16, 'lve': ['swim', 'code']}
        """直接从请求体中提取数据转
        # 客户端如果上传了json数据，直接返回字典
        {'name': '灰太狼', 'age': 20, 'sex': 1, 'classmate': '301', 'description': '我还会再回来的~'}
        # 客户端如果上传了表单数据，直接返回QueryDict
        <QueryDict: {'name': ['xiaohui'], 'age': ['18']}>
        """
        print(request.FILES) # 获取上传文件列表
        
        # 要获取django原生提供的HttpRequest对象，可以通过request._request来获取到
        print(request._request.META.get("Accept")) # 当值为None时，drf默认在响应数据时按json格式返回
        # response = Response(data="not ok", status=204, headers={"Company":"Oldboy"})
        response = Response(data="not ok", status=status.HTTP_400_BAD_REQUEST, headers={"Company":"Oldboy"})
        return response
```



## 二、Response

```
rest_framework.response.Response
```

REST framework提供了一个响应类`Response`，使用该类构造响应对象时，响应的具体数据内容会被转换（renderer渲染器）成符合前端需求的类型。

REST framework提供了`Renderer` 渲染器，用来根据请求头中的`Accept`（接收数据类型声明）来自动转换响应数据到对应格式。如果前端请求中未进行声明Accept，则会采用Content-Type方式处理响应数据，我们可以通过配置来修改默认响应格式。

可以在**rest_framework.settings**查找所有的drf默认配置项

```python
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (  # 默认响应渲染类
        'rest_framework.renderers.JSONRenderer',  # json渲染器，返回json数据
        'rest_framework.renderers.BrowsableAPIRenderer',  # 浏览器API渲染器，返回调试界面
    )
}
```

### 2.1、构造方式

```python
Response(data, status=None, template_name=None, headers=None, content_type=None)
```

drf的响应处理类和请求处理类不一样，Response就是django的HttpResponse响应处理类的子类。

`data`数据不要是render处理之后的数据，只需传递python的内建类型数据即可，REST framework会使用`renderer`渲染器处理`data`。

`data`不能是复杂结构的数据，如Django的模型类对象，对于这样的数据我们可以使用`Serializer`序列化器序列化处理后（转为了Python字典类型）再传递给`data`参数。

参数说明：

- `data`: 为响应准备的序列化处理后的数据；
- `status`: 状态码，默认200；
- `template_name`: 模板名称，如果使用`HTMLRenderer` 时需指明；
- `headers`: 用于存放响应头信息的字典；
- `content_type`: 响应数据的Content-Type，通常此参数无需传递，REST framework会根据前端所需类型数据来设置该参数。

### 2.2、response对象的属性

##### 1）.data

传给response对象的序列化后，但尚未render处理的数据

##### 2）.status_code

状态码的数字

##### 3）.content

经过render处理后的响应数据



### 2.3、状态码

为了方便设置状态码，REST framewrok在`rest_framework.status`模块中提供了常用http状态码的常量。

##### 1）信息告知 - 1xx

```python
HTTP_100_CONTINUE
HTTP_101_SWITCHING_PROTOCOLS
```

##### 2）成功 - 2xx

```python
HTTP_200_OK
HTTP_201_CREATED
HTTP_202_ACCEPTED
HTTP_203_NON_AUTHORITATIVE_INFORMATION
HTTP_204_NO_CONTENT
HTTP_205_RESET_CONTENT
HTTP_206_PARTIAL_CONTENT
HTTP_207_MULTI_STATUS
```

##### 3）重定向 - 3xx

```python
HTTP_300_MULTIPLE_CHOICES
HTTP_301_MOVED_PERMANENTLY
HTTP_302_FOUND
HTTP_303_SEE_OTHER
HTTP_304_NOT_MODIFIED
HTTP_305_USE_PROXY
HTTP_306_RESERVED
HTTP_307_TEMPORARY_REDIRECT
```

##### 4）客户端错误 - 4xx

```python
HTTP_400_BAD_REQUEST
HTTP_401_UNAUTHORIZED
HTTP_402_PAYMENT_REQUIRED
HTTP_403_FORBIDDEN
HTTP_404_NOT_FOUND
HTTP_405_METHOD_NOT_ALLOWED
HTTP_406_NOT_ACCEPTABLE
HTTP_407_PROXY_AUTHENTICATION_REQUIRED
HTTP_408_REQUEST_TIMEOUT
HTTP_409_CONFLICT
HTTP_410_GONE
HTTP_411_LENGTH_REQUIRED
HTTP_412_PRECONDITION_FAILED
HTTP_413_REQUEST_ENTITY_TOO_LARGE
HTTP_414_REQUEST_URI_TOO_LONG
HTTP_415_UNSUPPORTED_MEDIA_TYPE
HTTP_416_REQUESTED_RANGE_NOT_SATISFIABLE
HTTP_417_EXPECTATION_FAILED
HTTP_422_UNPROCESSABLE_ENTITY
HTTP_423_LOCKED
HTTP_424_FAILED_DEPENDENCY
HTTP_428_PRECONDITION_REQUIRED
HTTP_429_TOO_MANY_REQUESTS
HTTP_431_REQUEST_HEADER_FIELDS_TOO_LARGE
HTTP_451_UNAVAILABLE_FOR_LEGAL_REASONS
```

##### 5）服务器错误 - 5xx

```python
HTTP_500_INTERNAL_SERVER_ERROR
HTTP_501_NOT_IMPLEMENTED
HTTP_502_BAD_GATEWAY
HTTP_503_SERVICE_UNAVAILABLE
HTTP_504_GATEWAY_TIMEOUT
HTTP_505_HTTP_VERSION_NOT_SUPPORTED
HTTP_507_INSUFFICIENT_STORAGE
HTTP_511_NETWORK_AUTHENTICATION_REQUIRED
```

