# View 视图

一个视图函数（类），简称视图，是一个简单的Python 函数（类），它接受Web请求并且返回Web响应。

其中，函数view我们称之为FBV（function base views），类view我们称之为CBV（class base views）

响应可以是一张网页的HTML内容，一个重定向，一个404错误，一个XML文档，或者一张图片。

无论视图本身包含什么逻辑，都要返回响应。代码写在哪里也无所谓，只要它在你当前项目目录下面。除此之外没有更多的要求了——可以说“没有什么神奇的地方”。为了将代码放在某处，大家约定成俗将视图放置在项目（project）或应用程序（app）目录中的名为`views.py`的文件中。

每个视图函数都负责返回一个 HttpResponse 对象，对象中包含生成的响应。

视图层中有两个重要的对象：请求对象(request)与响应对象(HttpResponse)。



## 创建一个简单的函数视图

- App.views.py

```python
from django.shortcuts import render, HttpResponse

# Create your views here.

def index(request):
    return HttpResponse('App内的index')
```

应用内的 view.py 定义了一个函数，接收用户的请求，然后返回信息

> 注意：无论当前是否有参数，都必须有一个request参数，接受请求过来的对象

- App.urls.py

```python
from django.contrib import admin
from django.urls import path
from App import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('index/', views.index),
]
```

应用内设置路由

- project.urls.py

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('App.urls')),
]
```

工程内引用子应用的路由

然后启动项目，浏览器访问，即可在浏览器中看到 《App内的index》

## 定义视图

### 无参视图

路由地址

```pythono
path(r'index/', views.index, name='index1'),
```

视图函数

```python
from django.shortcuts import render, HttpResponse


# Create your views here.
def index(request):
    print(request)
    return HttpResponse('index')
```

### 有参视图

+ path

  ```python
  # 不限定参数类型
  path(r'args/<name>/<age>/', views.args, name='args1'),
  # 限定参数类型为字符串和整数
  path(r'args/<str:name>/<int:age>/', views.args, name='args'),
  # 名字和视图函数参数对应的 不能随便写
  path(r'args/<str:name>/<int:age1>/', views.args, name='args'),
  # 作为了解  会认为name参数后面的所有 都为参数
  path(r'args/<str:name>/<path:age>/', views.args, name='args'),
  ```

+ re_path

  ```python
  # 不起名称的传参
  re_path(r'args/(\w+)/(\d{2})/', views.args, name='args'),
  # 起名称的传参  名字和视图函数参数对应的 不能随便写
  re_path(r'^args/(?P<name>\w+)/(?P<age>\d{2})/$', views.args, name='args'),
  ```

  参数限定类型 如果类型不匹配则为404  参数匹配成功类型为str

### 限制请求类型

```python
from django.shortcuts import render, HttpResponse
from django.views.decorators.http import require_http_methods


# Create your views here.

@require_http_methods(['POST'])
def index(request):
    print(request)
    return HttpResponse('index')
```

允许类型写在require_http_methods(['POST']) 列表里，只有写在里面的请求方式，才是允许的

如果不使用当前装饰器则为，允许所有请求类型

### 错误视图

+ 下面是常见的HTTP状态码：

  + 200 - 请求成功
  + 301 - 资源（网页等）被永久转移到其它URL
  + 404 - 请求的资源（网页等）不存在
  + 500 - 内部服务器错误

+ 配置模板路径

  settings.py

  ```python
  TEMPLATES = [
      {
          'BACKEND': 'django.template.backends.django.DjangoTemplates',
          'DIRS': [os.path.join(BASE_DIR, 'templates')]
  ```

+ 更改DEBUG

  settings.py

  ```python
  DEBUG = False
  
  ALLOWED_HOSTS = ['*']
  ```

+ 目录层级

  ```
  project/
  	App/
  	project/
  	templates/
  		404.html
  		500.html
  ```

+ 404模板定制

  在templates下创建404.html

+ 500模板定制

  在templates下创建500.html

## 请求对象request

### 1、概述与作用

+ 作用

  客户端给服务器端传递数据用的

+ 概述

  + 服务器端接受http协议请求以后，会根据请求创建出请求对象
  + 视图函数的第一个参数为request
  + request保存客户端传递过来的数据

### 2、属性

+ path 请求的完整路径，不包含域名和端口
+ method  请求的方式
+ GET  类似字典对象，包含get请求的所有参数
+ POST   类似字典对象，包含post请求的所有参数
+ FILES  接受文件上传
+ COOKIES  一个包含了字典的所有cookie
+ session   包含了当前session值的对象 表示当前会话消息
+ META  包含了http请求的所有头信息

实例

```python
# 测试请求对象
def test_req(req):
    print('path', req.path)
    print('method', req.method)
    print('GET',req.GET)
    print('POST', req.POST)
    print('FILES', req.FILES)
    print('COOKIES', req.COOKIES)
    print('session', req.session)
    print('META', req.META)
    print('META', req.META['HTTP_USER_AGENT'])
    return HttpResponse('请求对象')
```

### 3、方法

+ get_full_path()  获取完整请求
+ get_host()  请求的主机和端口
+ is_ajax()   是否为ajax请求

### 4、GET传参的方式

+ 方式

  + 通过路由进行传参
  + url拼接get传参

+ 实例

  + 通过路由进行传参

    ```python
    path(r'args/<name>/<age>/', views.args, name='args1'),
    path(r'args/<str:name>/<int:age>/', views.args, name='args'),
    def test(req, name, age):
        return HttpResponse('请求对象')
    ```

    注意 需要在视图函数上进行接收

  + url拼接get传参 GET接收

    使用get方法接收

    在url地址中 使用？的形式进行拼接

    请求地址：127.0.0.1:8000?key=value&key=value.....

    ```python
    def test(req):
        print(req.GET)
        return HttpResponse('请求对象')
    ```

    注意：key不要重名

    key重名的

+ get方法

  请求地址：127.0.0.1:8000?key=value&key=value.....

  ```python
  def test(req):
      print(req.GET.get('name'))
      print(req.GET.get('age'))
      print(req.GET.getlist('name'))
      return HttpResponse('请求对象')
  ```

+ getlist方法

  请求地址：127.0.0.1:8000?key=value&key=value.....

  ```python
  def test(req):
      print(req.GET.getlist('name'))
      return HttpResponse('请求对象')
  ```

  可以接收同名的key的值 不会发生覆盖



### 5、POST方法

register.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>注册</h1>
<form action="" method="post">
    {% csrf_token %}
    <p>用户名：<input type="text" name="username"></p>
    <p>密码：<input type="text" name="userpass"></p>
    <p><input type="submit"></p>
</form>
</body>
</html>
```

views.py

```python
def register(req):
    if req.method == 'POST':
        print(req.POST)
        return HttpResponse('数据提交过来了')
    return render(req, 'register.html')
```



## 响应对象 HttpResponse

### 1、概述与配置

+ 概述

  请求对象是由框架创建的    响应是由程序员来创建的

+ 配置

  ```python
  from django.shortcuts import render, HttpResponse
  from django.template.loader import render_to_string
  from django.http import JsonResponse
  ```

### 2、使用

+ HttpResponse 直接返回响应内容

  ```python
  def index(request):
      print(request)
      return HttpResponse('index')
  ```

+ **render**  渲染模板

  参数

  + req 请求对象
  + templates_name 模板名称
  + contenxt  传递给模板数据
  + status 状态码

  ```python
  def register(req):
      if req.method == 'POST':
          print(req.POST)
          return HttpResponse('数据提交过来了')
      return render(req, 'register.html',{'info':'后台python数据'})
  ```

+ **render_to_string**  返回渲染后的字符串

  ```python
  def index(request):
      # print(request)
      # raise ValueError
      data = render_to_string('register.html')
      print(data)
      return HttpResponse('index')
  ```

- **redirect** 

  ```python
  # 返回重定向信息
  def my_view(request):
      ...
      return redirect('/some/url/')
  
  # 重定向的地址也可以是一个完整的URL：
  def my_view(request):
      ...
      return redirect('http://www.baidu.com/')　
  ```

**重定向转态码301与302的区别（了解）**

  ```
  一、301和302的异同。
     1、相同之处：
     301和302状态码都表示重定向，具体点说就是浏览器在拿到服务器返回的这个状态码后会自动跳转到一个新的URL地址（浏览器会从响应头Location中获取新地址），用户看到的效果都是输入地址A后瞬间跳转到了另一个地址B
     
     2、不同之处：
  　　301表示旧地址A的资源已经被永久地移除了，即这个资源不可访问了。搜索引擎在抓取新内容的同时也将旧的网址转换为重定向之后的地址；
  　　302表示旧地址A的资源还在，即这个资源仍然可以访问，这个重定向只是临时地从旧地址A跳转到地址B，搜索引擎会抓取新的内容、并且会保存旧的网址。 从SEO层面考虑，302要好于301
  
  二、重定向原因：
     1、网站调整（如改变网页目录结构）；
     2、网页被移到一个新地址；
     3、网页扩展名改变(如应用需要把.php改成.Html或.shtml)。
        这种情况下，如果不做重定向，则用户收藏夹或搜索引擎数据库中旧地址只能让访问客户得到一个404页面错误信息，访问流量白白丧失；再者某些注册了多个域名的网站，也需要通过重定向让访问这些域名的用户自动跳转到主站点等。
  ```

##  jsonResponse

向前端返回一个json格式字符串的两种方式

方式一：

```python
Copyimport json

def my_view(request):
    data=['egon','kevin']
    return HttpResponse(json.dumps(data) )
```

方式二：

```python
Copyfrom django.http import JsonResponse

def my_view(request):
    data=['egon','kevin']
    return JsonResponse(data,safe=False)
    #默认safe=True代表只能序列化字典对象，safe=False代表可以序列化字典以外的对象
```

## 案例一

urls.py

```python
Copyfrom django.urls import re_path
from app01 import views

urlpatterns = [
    re_path(r'^login/$',views.login),
]
```

Views.py

```python
Copyfrom django.shortcuts import render,HttpResponse

def login(request):
    if request.method == 'GET':
        # 当请求url为：http://127.0.0.1:8001/login/?a=1&b=2&c=3&c=4&c=5
        # 请求方法是GET，?后的请求参数都存放于request.GET中
        print(request.GET)
        # 输出<QueryDict: {'a': ['1'], 'b': ['2'], 'c': ['3', '4', '5']}>

        # 获取？后参数的方式为
        a=request.GET.get('a') # 1
        b=request.GET.get('b') # 2
        c=request.GET.getlist('c') # ['3', '4', '5']

        return render(request,'login.html')
    elif request.method == 'POST':
        # 在输入框内输入用户名egon、年龄18，选择爱好，点击提交
        # 请求方法为POST，表单内的数据都会存放于request.POST中
        print(request.POST) 
        # 输出<QueryDict: {..., 'name': ['egon'], 'age': ['18'], 'hobbies': ['music', 'read']}>

        # 获取表单中数据的方式为
        name=request.POST.get('name') # egon
        age=request.POST.get('age') # 18
        hobbies=request.POST.getlist('hobbies') # ['music', 'read']

        return HttpResponse('提交成功')
```

在templates目录下新建login.html

```html
Copy<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录页面</title>
</head>
<body>

<!--
method="post"代表在提交表单时会以POST方法提交表单数据
action="/login/" 代表表单数据的提交地址为http://127.0.0.1:8001/login/,可以简写为action="/login/",或者action=""
-->
<form action="http://127.0.0.1:8001/login/" method="post">
    {% csrf_token %} <!--强调：必须加上这一行，后续我们会详细介绍-->
    <p>用户名：<input type="text" name="name"></p>
    <p>年龄：<input type="text" name="age"></p>
    <p>
        爱好：
        <input type="checkbox" name="hobbies" value="music">音乐
        <input type="checkbox" name="hobbies" value="read">阅读
        <input type="checkbox" name="hobbies" value="dancing">跳舞
    </p>
    <p><input type="submit" value="提交"></p>

</form>
</body>
</html>
```

## 案例二、form表单上传文件

urls.py

```python
Copyfrom django.urls import path,register_converter,re_path
from app01 import views

urlpatterns = [
    re_path(r'^register/$',views.register),
]
```

views.py

```python
Copyfrom django.shortcuts import render,HttpResponse

def register(request):
    if request.method == 'GET':
        return render(request,'register.html')
    elif request.method == 'POST':
        print(request.body)

        # 从request.POST中获取用户名
        name=request.POST.get('name') 
        # 从request.FILES获取文件对象
        file_obj=request.FILES.get('header_img') 
        
        # 上传的文件存放于templates文件夹下
        with open('templates/header.png','wb') as f: 
            for line in file_obj:
                f.write(line)

        return HttpResponse('注册成功')
```

在templates目录下新建register.html

```python
Copy<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>注册页面</title>
</head>
<body>

<form action="" method="POST" enctype="multipart/form-data" >
    {% csrf_token %}
    <p>
        用户名:<input type="text" name="name">
    </p>
    <p>
        头像:<input type="file" name="header_img">
    </p>
    <p>
        <input type="submit" value="提交">
    </p>
</form>
</body>
</html>
```

