# Django模板templates

## 一、模板概述与配置

### 1.1、概述

作为Web框架，Django提供了模板，用于编写html代码，还可以嵌入模板代码更快更方便的完成页面开发，再通过在视图中渲染模板，将生成最终的html字符串返回给客户端浏览器。模板致力于表达外观，而不是程序逻辑。模板的设计实现了业务逻辑view与显示内容template的分离，一个视图可以使用任意一个模板，一个模板可以供多个视图使用。

> 简单来说：模板就是HTML代码＋Django模板语言，用来把后端的变量嵌入到HTML中渲染后，实现页面的动态显示

模板包含两部分：

- 静态部分，包含html、css、js。
- 动态部分，就是模板语言。

### 1.2、配置

Django模板语言，简写DTL，定义在django.template包中。 创建项目后，在"项目名称/settings.py"文件中定义了关于模板的配置。

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
        ,
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

### 1.3、配置模板目录

`DIRS`定义一个目录列表，模板引擎按列表顺序搜索这些目录以查找模板文件，通常是在项目的根目录下创建templates目录。

### 1.4、模板的渲染

Django处理模板分为两个阶段：

- 1.加载：根据给定的路径找到模板文件，编译后放在内存中。
- 2.渲染：使用上下文数据对模板插值并返回生成的字符串。

有以下两种方式进行渲染

+ render   直接渲染模板
+ render_to_string   返回渲染模板后的内容

## 模板语言

模板语言包括4种类型，分别是：

- 变量
- 标签
- 过滤器
- 注释

## 二、模板变量

模板变量的作用是计算并输出，变量名必须由字母、数字、下划线（不能以下划线开头）和点组成。

语法如下：

```django
{{ 变量 }}
```

### 传递变量

视图函数中

```python
def test_tem(req):
    args = {'title': '首页'}
    return render(req, 'index.html', args)
```

将字典args传递模板中进行渲染

模板中

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
</head>
<body>
<h1>首页</h1>
</body>
</html>
```

!> 如果变量不存在则插入空字符串，啥都没有



## 三、模板标签

语法如下：

```django
{% 标签名 %}
```

### 运算符

==  !=  > >= < <=  and  or  in  not in   not

!> 注意：运算符左右两侧不能紧挨变量或常量，必须有空格。并且 算数语运算符 不允许使用

### 1、if 标签（支持if嵌套）

+ 单一分支

  ```
  {% if ... %}
  	...
  {% endif %}
  ```

+ 双向条件分支

  ```
  {% if ... %}
  	...
  {% else %}
      ...
  {% endif %}
  ```

+ 多向条件分支

  ```
  {% if ... %}
  	...
  {% elif %}
      ...
  {% else %}
      ...
  {% endif %}
  ```

### 2 、for 标签

与python中的 for循环 使用方式一样，`for in 可迭代对象`

- 格式：

  不搭配empty

  ```
  {% for var in seq %}
  	...
  {% endfor %}
  ```

  搭配empty

  empty : for 循环取不到值的时候执行empty下的代码

  ```
  {% for var in seq %}
  	...
  {% empty %}
  	...
  {% endfor %}
  ```

- 使用

  ```html
  {% for k in info %}
      <li>{{ k }}</li>
  {% endfor %}

  {% for k,v in info.items %}
      <li>{{ k }}====={{ v }}</li>
  {% endfor %}
  ```

  empty

  ```html
  {% for k in xxx %}
      <li>{{ k }}</li>
  {% empty %}
      <li>empty</li>
  {% endfor %}
  ```

  循环的变量不存在，执行empty

### 3、ifequal/ifnotequal 标签

+ 作用

  判断是否相等/不相等

+ 格式

  ```
  {% ifequal 表达式1  表达式2 %}
  	...
  {% else %}
  	...
  {% endifequal %}

  {% ifnotequal 表达式1  表达式2 %}
  	...
  {% else %}
  	...
  {% endifnotequal %}
  ```

+ 实例

  ```html
  {% ifequal 'l' 'a' %}
      <li>为真</li>
      {% else %}
      <li>为假</li>
  {% endifequal %}


  {% ifnotequal 'l' 'a' %}
      <li>为真</li>
  {% else %}
      <li>为假</li>
  {% endifnotequal %}
  ```

### 4、注释标签

+ 说明

  + 代码调试
  + 解释说明

+ 单行注释

  ```
  {#  #}
  ```

+ 多行注释

  ```
  {% comment %}
  {% endcomment %}
  ```

### 5、include导入标签

+ 说明

  include 语句可以把一个模板引入到另一个模板中，类似于把一个模板的代码copy到当前指定的位置

+ 目录结构

  ```
  project/
  	templates/
  		common/
  			header.html
  			footer.html
  ```

+ 使用

  test_include.html

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style>
          body{
              padding: 0;
              margin: 0;
          }
          header{
              width: 100%;
              height: 150px;
              background-color: #000;
              color: #fff;
              font-size: 40px;
          }
          footer{
              width: 100%;
              height: 150px;
              background-color: #000;
              color: #fff;
              font-size: 40px;
              position: absolute;
              bottom: 0;
          }
      </style>
  </head>
  <body>
  {% include 'common/header.html' %}
  <h1>测试include方法</h1>
  {% include 'common/footer.html' %}
  </body>
  </html>
  ```

  common/header.html

  ```html
  <header>
      头部
  </header>
  ```

  common/footer.html

  ```html
  <footer>
      底部
  </footer>
  ```

  !> 导入文件内部的代码一定是你要使用的代码  不要包含其它代码（不需要在引入的html代码中写必须结构体，如\<!DOCTYPE html>，\<html lang="en">）

  视图函数

  ```python
  # 测试导入标签
  def test_include(req):
      return render(req, 'test_include.html')
  ```

### 6、模板继承 extend

+ 概述

  django中的模板可以继承的，通过继承可以将页面中许多元素抽取出来，放在父模板中，父模板中通过\<block>标签进行标记，在子模板中实现替换

+ 作用

  实现页面代码的复用，减少重复代码

+ 标签

  + 继承标签  {% extend  名称 %}
  + 模板替换  {% block 名称 %}{% endblock %}

+ 使用

  common/base.html

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
  {% block head %}
      <meta charset="UTF-8">
      {% block meta %}{% endblock %}
      <title>{% block title %}Title{% endblock %}</title>
      {% block link %}{% endblock %}
      {% block script %}{% endblock %}
      <style>
          {% block style %}
          {% endblock %}
      </style>
  {% endblock %}
  </head>
  <body>
  {% block body %}

  {% endblock %}
  </body>
  </html>
  ```

  test_base.html

  ```html
  {% extends 'common/base.html' %}
  {% block title %}
  {{ block.super }}
      首页
  {% endblock %}
  {% block con %}
      <i>首页</i>
  {% endblock %}
  ```

  视图

  ```python
  def test_base(req):
      return render(req, 'test_base.html')
  ```

  !> 注意：如果子模板替换了父母板 还想显示原来的内容  {{ block.super }}


### 8、include与extend区别

+ 相同点

  均实现了代码的重用

+ 不同点

  包含是将目标文件整个渲染出来

  继承的本质是代码的替换，一般用来实现页面中重复不变的区域

### 9、模板中动态构造路由地址

配置

project/urls.py

```python
urlpatterns = [
    path('', include(('App.urls', 'App'), namespace='App')),
]
```

App/urls.py

```python
path('base/', index.test_base, name='test_base'),
path('args/<name>/<age>/', index.args, name='args'),
```

模板中进行构造

```html
<h3>构造路由地址</h3>
<ul>
    <li>{% url 'App:test_base' %}</li>
    <li><a href="{% url 'App:test_base' %}" target="_blank">test_base</a></li>
    <li>{% url 'App:args' 'lucky' 18 %}</li>
    <li>{% url 'App:args' name='lucky' age=18 %}</li>
    <li>{% url 'App:args' name='lucky' age=18 %}?name=lucky&age=18</li>
</ul>
```

### 10、django使用bootstrap

+ 安装

  pip install django-bootstrap3

+ 配置

  ```python
  INSTALLED_APPS = [
      ...
      'App',
      'bootstrap3',
  ]
  ```

+ index.html

  ```html
  {% extends 'bootstrap3/bootstrap3.html' %}
  {% block bootstrap3_content %}
  <nav class="navbar navbar-inverse">
    <div class="container-fluid">
      <!-- Brand and toggle get grouped for better mobile display -->
      <div class="navbar-header">
        <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
          <span class="sr-only">Toggle navigation</span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
        </button>
        <a class="navbar-brand" href="#">Brand</a>
      </div>

      <!-- Collect the nav links, forms, and other content for toggling -->
      <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
        <ul class="nav navbar-nav">
          <li class="active"><a href="#">Link <span class="sr-only">(current)</span></a></li>
          <li><a href="#">Link</a></li>
          <li class="dropdown">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Dropdown <span class="caret"></span></a>
            <ul class="dropdown-menu">
              <li><a href="#">Action</a></li>
              <li><a href="#">Another action</a></li>
              <li><a href="#">Something else here</a></li>
              <li role="separator" class="divider"></li>
              <li><a href="#">Separated link</a></li>
              <li role="separator" class="divider"></li>
              <li><a href="#">One more separated link</a></li>
            </ul>
          </li>
        </ul>
        <form class="navbar-form navbar-left">
          <div class="form-group">
            <input type="text" class="form-control" placeholder="Search">
          </div>
          <button type="submit" class="btn btn-default">Submit</button>
        </form>
        <ul class="nav navbar-nav navbar-right">
          <li><a href="#">Link</a></li>
          <li class="dropdown">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Dropdown <span class="caret"></span></a>
            <ul class="dropdown-menu">
              <li><a href="#">Action</a></li>
              <li><a href="#">Another action</a></li>
              <li><a href="#">Something else here</a></li>
              <li role="separator" class="divider"></li>
              <li><a href="#">Separated link</a></li>
            </ul>
          </li>
        </ul>
      </div><!-- /.navbar-collapse -->
    </div><!-- /.container-fluid -->
  </nav>

      <div id="carousel-example-generic" class="carousel slide" data-ride="carousel">
    <!-- Indicators -->
    <ol class="carousel-indicators">
      <li data-target="#carousel-example-generic" data-slide-to="0" class="active"></li>
      <li data-target="#carousel-example-generic" data-slide-to="1"></li>
      <li data-target="#carousel-example-generic" data-slide-to="2"></li>
    </ol>

    <!-- Wrapper for slides -->
    <div class="carousel-inner" role="listbox">
      <div class="item active">
        <img src="..." alt="...">
        <div class="carousel-caption">
          ...
        </div>
      </div>
      <div class="item">
        <img src="..." alt="...">
        <div class="carousel-caption">
          ...
        </div>
      </div>
      ...
    </div>

    <!-- Controls -->
    <a class="left carousel-control" href="#carousel-example-generic" role="button" data-slide="prev">
      <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>
      <span class="sr-only">Previous</span>
    </a>
    <a class="right carousel-control" href="#carousel-example-generic" role="button" data-slide="next">
      <span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span>
      <span class="sr-only">Next</span>
    </a>
  </div>
  {% endblock %}
  ```

### 11、csrf跨站请求伪造

+ 说明

  某些恶意的网站上包含链接、表单按钮js代码，他们会利用你登陆过的信息视图在你的网站上完成某些操作，这就是跨站攻击

+ 作用

  在页面中生成csrf token的cookie

  在页面中生成csrf的隐藏域

  用于跨站请求伪造保护

+ 原理

  请求会带着cookie和隐藏域的值访问服务器，服务器端根据cookie和表单隐藏域的值进行对比，如果对比成功则通过，否则拒绝请求 403

### 12、autoescape 标签

+ 作用

  HTML转义标签

模板对上下文传递的字符串进行输出时，会对以下字符自动转义。

```
小于号< 转换为 &lt;

大于号> 转换为 &gt;

单引号' 转换为 &#39;

双引号" 转换为 &quot;

与符号& 转换为 &amp;
```

+ 使用

  开启转义（默认转义）

  ```python
  {% autoescape on %}
      {{ con }}
  {% endautoescape %}
  ```

  关闭转义

  ```python
  {% autoescape off %}
  	{{ con }}
  {% endautoescape %}
  ```

+ 过滤关闭转义，告诉模板这个变量是安全的，可以解释执行

  ```python
  {{ con|safe }}
  ```



## 四、过滤器

### 1、说明

格式：

```
{ { 变量 | 过滤器 }}
```

例如{ { name | lower }}，表示将变量name的值变为小写输出

- 使用管道符号`|`来应用过滤器，用于进行计算、转换操作，可以使用在变量、标签中。

- 如果过滤器需要参数，则使用冒号`:`传递参数，参数使用引号包起来

  ```
  变量|过滤器:参数
  ```

1）可以在 if 标签中使用过滤器结合运算符

```
if list1|length > 1
```

2）过滤器能够被串联，构成过滤器链

```
name|lower|upper
```

3）过滤器可以传递参数，参数使用引号包起来

```
list|join:", "
```

### 2、内置过滤器

+ first  获取第一个成员
+ last  获取最后一个成员
+ lower  转换为小写
+ upper  转换为大写
+ cut  查找删除指定字符
+ default  默认值
+ escape  转换为实体符号标签
+ floatformat  保留小数   {{1.2345|floatformat:2}}
+ length  返回长度
+ random  返回随机项
+ join  字符串拼接
+ striptags   取出标签

```html
<ol>
    <li>{{ 'abcd'|first }}</li>
    <li>{{ 'abcd'|last }}</li>
    <li>{{ 'abAHGHJcd'|lower }}</li>
    <li>{{ 'abAHGHJcd'|upper }}</li>
    <li>{{ 'abAHGHJcd'|upper|first }}</li>
    <li>{{ 'abcd'|cut:'a' }}</li>
    <li>{{ abc|default:'default' }}</li>
    <li>{{1.2345|floatformat:2}}</li>
    <li>{{ 'abcd'|length }}</li>
    <li>{{ 'abcd'|random }}</li>
    <li>{{ List|join:',' }}</li>
    <li>{{ '<b>b标签</b>'|striptags }}</li>
</ol>
```

日期date，用于对日期类型的值进行字符串格式化，常用的格式化字符如下：

```
value|date:'Y-m-d'
```

- Y表示年，格式为4位，y表示两位的年。
- m表示月，格式为01,02,12等。
- d表示日, 格式为01,02等。
- j表示日，格式为1,2等。
- H表示时，24进制，h表示12进制的时。
- i表示分，为0-59。
- s表示秒，为0-59。

### 3、自定义过滤器

+ 目录结构

  ```
  project/
  	App/
  		templatetags/ 名字固定的
  			__init__.py
  			mytag.py  自定义的文件
  ```

+ mytags.py

  ```python
  from django import template
  register = template.Library()  # 实例化对象


  # 自定义过滤器
  # 如果起别名 则使用  @register.filter('sum_filter')
  @register.filter
  # @register.filter('sum_filter')
  def sum_filter(x, y):
      return x + y


  # 自定义标签
  # @register.simple_tag()
  @register.simple_tag
  def sum_tag(x, y):
      return x + y
  ```

+ 模板中使用

  ```html
  <li>{{ '<b>b标签</b>'|striptags }}</li>
  <li>自定义过滤器</li>
  <li>{{ 1|sum_filter:1 }}</li>
  <li>自定义标签</li>
  <li>{% sum_tag 1 2 %}</li>
  ```



## 五、静态文件

### 1、说明

网站通常需要提供其它文件，比如图片、css、js，在django中我们称为静态文件

### 2、目录结构

```
project/
	App/
	templates/
	static/
		css/
		js/
		img/
```

### 3、配置静态文件

+ INSTALL_APPS  中 'django.contrib.staticfiles',

+ 设置url

  ```
  STATIC_URL = '/static/'
  ```

### 4、使用

```html
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="{% static 'css/test.css' %}">
</head>
<body>
<h1>测试静态文件</h1>
</body>
</html>
```

