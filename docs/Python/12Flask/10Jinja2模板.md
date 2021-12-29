# Jinja2模板引擎

Flask内置的模板语言Jinja2，它的设计思想来源于 Django 的模板引擎DTP(DjangoTemplates)，并扩展了其语法和一系列强大的功能。

- Flask提供的 **render_template** 函数封装了该模板引擎Jinja2
- **render_template** 函数的第一个参数是模板的文件名，后面的参数都是键值对，表示模板中变量对应的数据值。



## 模板基本使用

1. 在flask应用对象创建的时候，设置或者保留template_folder参数，创建模板目录

   ```python
   app = Flask(__name__,template_folder='templates')
   ```

2. 在项目下手动创建 `templates` 文件夹，用于存放所有的模板文件，并在目录下创建一个模板html文件 `index.html`

   ```django
   <!doctype html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>{{ title }}</title>
   </head>
   <body>
       <h1>{{ content }}</h1>
   </body>
   </html>
   ```

   

3. 在视图函数设置渲染模板并设置模板数据

   ```python
   from flask import Flask, render_template
   
   # 默认情况下，flask默认直接支持视图加载模板的。
   # Flask类在实例化的时候，默认当前根目录下的templates目录作为模板目录
   app = Flask(__name__, template_folder="templates")
   # 配置
   app.config.update({
       "DEBUG": True
   })
   
   @app.route("/")
   def index():
       data = {
           "title": "我的模板标题",
           "content": "我的模板内容"
       }
       return render_template("index.html", **data)
   
   @app.route("/user/<id>")
   def user(id):
       title = "User Center"
       content = "我的个人中心"
       print(locals()) # {'id': '100', 'title': 'User Center', 'content': '我的个人中心'}
       return render_template("user.html", **locals())
   
   if __name__ == '__main__':
       app.run()
   
   ```

### 输出变量

```
{{ 变量名 }}，这种 {{ }} 语法叫做 变量代码块
```

视图代码：

```python
import click
from flask import Flask,render_template
# flask开发小型的项目，直接在当前flask应用配置即可。手动创建模板目录。
# flask开发中大型项目，直接在当前flask的每一个子应用(蓝图)下构建目录。
app = Flask(import_name=__name__, template_folder="templates")

# 配置类
class Config(object):
    DEBUG = True  # 开启调试模式

# 加载配置
app.config.from_object(Config)

@app.route("/")
def index():
    title = "站点标题"
    user_list = [
        {"id":1, "name": "xiaoming", "age":16},
        {"id":2, "name": "xiaoming", "age":16},
        {"id":3, "name": "xiaoming", "age":16},
    ]
    return render_template("index.html", **locals())

if __name__ == '__main__':
    app.run()
```

模板代码

```jinja2
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{title}}</title>
</head>
<body>
    <h1>{{title}}</h1>
    <p>{{ user_list.0 }}</p>
    <p>{{ user_list.0.name }}</p>
</body>
</html>
```



pycharm中设置当前项目的模板语言：

files/settings/languages & frameworks/python template languages。

设置下拉框为jinja2，保存

![1596532209377](10Jinja2%E6%A8%A1%E6%9D%BF.assets/1596532209377.png)

设置指定目录为模板目录，鼠标右键->Mark Directory as ...-> Template Folder

![image-20211026175126077](10Jinja2%E6%A8%A1%E6%9D%BF.assets/image-20211026175126077.png)



Jinja2 模版中的变量代码块的输出的内容可以是Python的任意类型数据或者对象，只要它能够被 Python 的 `__str__` 方法或者str()转换为一个字符串就可以，比如，可以通过下面的方式显示一个字典或者列表中的某个元素:

视图代码：

```python
from flask import Flask, render_template

app = Flask(__name__, template_folder="templates")


@app.route("/")
def index():
    title = "我的flask"
    data_list = ["a", "b", "c"]
    data_dict = {
        "name": "xiaoming",
        "id": 100,
    }
    user_list = [
        {"id":1, "name": "xiaoming", "age":16},
        {"id":2, "name": "xiaoming", "age":16},
        {"id":3, "name": "xiaoming", "age":16},
    ]

    return render_template("index.html", **locals())


if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000, debug=True)

```

模板代码：

```jinja2
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
</head>
<body>
    <h1>{{ title }}!!</h1>
    <p>{{ data_list }}</p>
    <p>{{ data_list.1 }}</p>
    <p>{{ data_list[-1]}}</p>
    <p>{{ data_list | last }}</p>
    <p>{{ data_list | first }}</p>
    <p>{{ data_dict }}</p>
    <p>{{ data_dict.name }}</p>
    <p>{{ user_list.0 }}</p>
    <p>{{ user_list.0.name }}</p>
</body>
</html>
```

使用 {# #} 进行注释，注释的内容不会在html中被渲染出来

```jinja2
{#    <h1>{{ title }}!!</h1>#}
{#    <p>{{ data_list }}</p>#}
{#    <p>{{ data_list.1 }}</p>#}
    <p>{{ data_list[-1]}}</p>
    <p>{{ data_list | last }}</p>
    <p>{{ data_list | first }}</p>
    <p>{{ data_dict }}</p>
    <p>{{ data_dict.name }}</p>
    <p>{{ user_list.0 }}</p>
{#    <p>{{ user_list.0.name }}</p>#}
```



## 模板中内置的变量和函数

你可以在自己的模板中访问一些 Flask 默认内置的函数和对象

#### config

你可以从模板中直接访问Flask当前的config对象:

```jinja2
    <p>{{ config.ENV }}</p>
    <p>{{ config.DEBUG }}</p>
```

#### request

就是flask中代表当前请求的request对象：

```jinja2
    <p>{{ request.url }}</p>
    <p>{{ request.path }}</p>
    <p>{{ request.method }}</p>
```

#### session

为Flask的session对象，显示session数据

```jinja2
{{session.new}}
False
```

#### g变量

在视图函数中设置g变量的 name 属性的值，然后在模板中直接可以取出

```jinja2
{{ g.name }}
```

#### url_for()

url_for会根据传入的路由器函数名,返回该路由对应的URL,在模板中始终使用url_for()就可以安全的修改路由绑定的URL,则不比担心模板中渲染出错的链接:

```jinja2
{{url_for('home')}}
```

如果我们定义的路由URL是带有参数的,则可以把它们作为关键字参数传入url_for(),Flask会把他们填充进最终生成的URL中:

```jinja2
{{ url_for('index', id=1)}}
/index/1      {#  /index/<int:id> id被声明成路由参数 #}
/index?id=1   {#  /index          id被声明成路由参数 #}
```

课堂代码：

主程序 manage.py：

```python
from flask import Flask, render_template,g

app = Flask(__name__, template_folder="templates")


@app.route("/")
def index():
    g.name = "xiaohei"
    title = "我的flask"
    data_list = ["a", "b", "c"]
    data_dict = {
        "name": "xiaoming",
        "id": 100,
    }
    user_list = [
        {"id":1, "name": "xiaoming", "age":16},
        {"id":2, "name": "xiaoming", "age":16},
        {"id":3, "name": "xiaoming", "age":16},
    ]

    return render_template("index.html", **locals())


@app.route("/user/<int:uid>")
def user(uid):
    print(uid)
    return f"uid={uid}"

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000, debug=True)

```

模板 templates/index.html：

```jinja2
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
</head>
<body>
{#    <h1>{{ title }}!!</h1>#}
{#    <p>{{ data_list }}</p>#}
{#    <p>{{ data_list.1 }}</p>#}
    <p>{{ data_list[-1]}}</p>
    <p>{{ data_list | last }}</p>
    <p>{{ data_list | first }}</p>
    <p>{{ data_dict }}</p>
    <p>{{ data_dict.name }}</p>
    <p>{{ user_list.0 }}</p>
{#    <p>{{ user_list.0.name }}</p>#}

    <p>{{ config.ENV }}</p>
    <p>{{ config.DEBUG }}</p>
    <p>{{ request.url }}</p>
    <p>{{ request.path }}</p>
    <p>{{ request.method }}</p>

    <p>{{ session.new }}</p>
    <p>{{ g.name }}</p>

    <p>{{ url_for("user", uid=3) }}</p>    {# /user/3 #}

</body>
</html>
```



## 流程控制

主要包含两个：

```
- if / elif /else / endif
- for / else / endfor

```



### if语句

Jinja2 语法中的if语句跟 Python 中的 if 语句相似,后面的布尔值或返回布尔值的表达式将决定代码中的哪个流程会被执行.

用 {**%%**} 定义的**控制代码块**，可以实现一些语言层次的功能，比如循环或者if语句

视图代码：

```python
import random
from flask import Flask, render_template

app = Flask(__name__, template_folder="templates")


@app.route("/")
def index():
    data = random.randint(1,100)
    return render_template("index6.html", **locals())


@app.route("/user/<int:uid>")
def user(uid):
    print(uid)
    return f"uid={uid}"

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000, debug=True)
```

index.html，模板代码：

```jinja2
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    {% if data < 60 %}
        <p>本次生成的数字不及格！</p>
    {% else %}
        <p>本次生成的数字及格了！</p>
    {% endif %}


    {% if data < 60 %}
        <p>本次生成的数字不及格！</p>
    {% elif data <80 %}
        <p>本次生成的数字一般般，不算高！</p>
    {% else %}
        <p>本次生成的数字超过80，非常好！</p>
    {% endif %}

</body>
</html>
```



### 循环语句

- 我们可以在 Jinja2 中使用循环来迭代任何列表或者生成器函数

```jinja2
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
    table, td,th{
        border: 1px solid red;
        border-collapse: collapse;
    }
    table{
        width: 800px;
    }
    td,th{
        padding: 4px;
    }
    .last{
        background: orange;
    }
    </style>
</head>
<body>
    <table>
        <tr>
            <th>序号</th>
            <th>ID</th>
            <th>name</th>
            <th>age</th>
        </tr>
        {% for user in user_list %}
        {% if loop.last %}
        <tr class="last">
        {% else %}
        <tr>
        {% endif %}
{#            <th>{{ loop.index }}</th>#}
{#            <th>{{ loop.index0 }}</th>#}
{#            <th>{{ loop.revindex0 }}</th>#}
            <th>{{ loop.revindex }}</th>
            <th>{{ user.id }}</th>
            <th>{{ user.name }}</th>
            <th>{{ user.age }}</th>
        </tr>
        {% endfor %}

    </table>
</body>
</html>
```

- 循环和if语句可以组合使用，以模拟 Python 循环中的 continue 功能，下面这个循环将只会渲染user.id为偶数的那些user：

```jinja2
    <table border="1" width="1200">
        <tr>
            <th width="200">序号[从小到大，从0开始]</th>
            <th width="200">序号[从小到大，从1开始]</th>
            <th width="200">序号[从大到小，到0结束]</th>
            <th width="200">序号[从大到小，到1结束]</th>
            <th>ID</th>
            <th>name</th>
            <th>age</th>
        </tr>
        {% for user in user_list if user.id%2==0 %}
        <tr>
            <td>{{ loop.index0 }}</td>
            <td>{{ loop.index }}</td>
            <td>{{ loop.revindex0 }}</td>
            <td>{{ loop.revindex }}</td>
            <td>{{ user.id }}</td>
            <td>{{ user.name }}</td>
            <td>{{ user.age }}</td>
        </tr>
        {% endfor %}
    </table>
```

- 在一个 for 循环块中你可以访问这些特殊的变量:

| 变量           | 描述                                           |
| :------------- | :--------------------------------------------- |
| loop.index     | 当前循环迭代的次数（从 1 开始）                |
| loop.index0    | 当前循环迭代的次数（从 0 开始）                |
| loop.revindex  | 到循环结束需要迭代的次数（从 1 开始）          |
| loop.revindex0 | 到循环结束需要迭代的次数（从 0 开始）          |
| loop.first     | 如果是第一次迭代，为 True 。                   |
| loop.last      | 如果是最后一次迭代，为 True 。                 |
| loop.length    | 序列中的项目数。                               |
| loop.cycle     | 在一串序列间期取值的辅助函数。见下面示例程序。 |

- 在循环内部,你可以使用一个叫做loop的特殊变量来获得关于for循环的一些信息
  - 比如：要是我们想知道当前被迭代的元素序号，并模拟Python中的enumerate函数做的事情，则可以使用loop变量的index属性,例如:

```jinja2
<ul>
{% for item in data_list %}
<li>{{ loop.index0 }}，item={{ item }}</li>
{% endfor %}
</ul>
```

- 会输出这样的结果

```python
0，item=a
1，item=b
2，item=c
```

- loop.cycle函数会在每次循环的时候,返回其参数中的下一个元素,可以拿上面的例子来说明:

```python
    <ul>
        {% for item in data_list %}
        <li>{{ loop.cycle("男","女")}}，item={{ item }}</li>
        {% endfor %}
    </ul>
```

- 会输出这样的结果：

```python
男，item=a
女，item=b
男，item=c
```

#### 课堂代码

视图代码：

```python
import click
from flask import Flask,render_template,g
# flask开发小型的项目，直接在当前flask应用配置即可。手动创建模板目录。
# flask开发中大型项目，直接在当前flask的每一个子应用(蓝图)下构建目录。
app = Flask(import_name=__name__, template_folder="templates")

# 加载配置
app.config.from_object(Config)

@app.route("/")
def index():
    title = "站点标题"
    data_list = ["a","b","c"]
    data_dict = {
        "name": "xiaoming",
        "id": 100,
    }
    user_list = [
        {"id":2, "name": "xiaoming", "age":15},
        {"id":3, "name": "xiaoming", "age":16},
        {"id":4, "name": "xiaoming", "age":17},
    ]

    g.name = "来自视图"

    return render_template("index.html", **locals())

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000, debug=True)
```

index.html，模板代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{title}}</title>
</head>
<body>
{#    <h1>{{title}}</h1>
    <p>{{ data_list }}</p>
    <p>{{ data_list.1 }}</p>
    <p>{{ data_list | first }}</p>
    <p>{{ data_list | last }}</p>
    <p>{{ data_dict }}</p>
    <p>{{ data_dict.name }}</p>
#}{#    <p>{{ data_list.-1 }}</p>#}{#
    <p>{{ user_list.0 }}</p>
    <p>{{ user_list.0.name }}</p>
#}

{#    <p>{{ data_list[0] }}</p>#}
{#    <p>{{ data_list[-1] }}</p>#}
{#    <p>{{ data_dict['name'] }}</p>#}
{#    <p>{{ config }}</p>#}
{#    <p>{{ config.ENV }}</p>#}
{#    <p>{{ request }}</p>#}
{#    <p>获取地址栏参数：id={{ request.args.id }}</p>#}
{#    <p>{{ session }}</p>#}
{#    <p>{{ session.new }}</p>#}
{#    <p>{{ g.name }}</p>#}
{#    <p>{{ url_for("index",id=1) }}</p>#}


    {% if "a" in data_list %}
        <p>{{ data_list }}包含a字符</p>
    {% endif %}

    {% if "A" in data_list %}
        <p>{{ data_list }}包含A字符</p>
    {% else  %}
        <p>{{ data_list }}不包含A字符</p>
    {% endif %}

    <table border="1" width="1200">
        <tr>
            <th width="200">序号[从小到大，从0开始]</th>
            <th width="200">序号[从小到大，从1开始]</th>
            <th width="200">序号[从大到小，到0结束]</th>
            <th width="200">序号[从大到小，到1结束]</th>
            <th>ID</th>
            <th>name</th>
            <th>age</th>
        </tr>
        {% for user in user_list if user.id%2==0 %}
        <tr>
            <td>{{ loop.index0 }}</td>
            <td>{{ loop.index }}</td>
            <td>{{ loop.revindex0 }}</td>
            <td>{{ loop.revindex }}</td>
            <td>{{ user.id }}</td>
            <td>{{ user.name }}</td>
            <td>{{ user.age }}</td>
        </tr>
        {% endfor %}
    </table>


    <ul>
        {% for item in data_list %}
        <li>{{ loop.index0 }}，item={{ item }}</li>
        {% endfor %}
    </ul>

    <ul>
        {% for item in data_list %}
        <li>{{ loop.cycle("男","女")}}，item={{ item }}</li>
        {% endfor %}
    </ul>

</body>
</html>
```



## 过滤器

django中的模板引擎里面曾经使用过滤器，在flask中也有过滤器，并且也可以被用在 if 语句或者for语句中:

视图代码：

```python
import random
from flask import Flask, render_template

app = Flask(__name__, template_folder="templates")


@app.route("/")
def index():
    book_list = [
        {"id":1, "price": 78.50, "title":"javascript入门", "cover": "<img src='/static/images/course.png'>"},
        {"id":2, "price": 78.5, "title":"python入门", "cover": "<img src='/static/images/course.png'>"},
        {"id":3, "price": 78.6666, "title":"django web项目实战", "cover": "<img src='/static/images/course.png'>"}
    ]
    return render_template("index8.html", **locals())

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000, debug=True)

```

index8.html，模板代码：

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
    table, td,th{
        border: 1px solid red;
        border-collapse: collapse;
    }
    table{
        width: 800px;
    }
    td,th{
        padding: 4px;
    }
    img{
        width: 100px;
    }
    </style>
</head>
<body>
    <table>
        <tr>
            <th>ID</th>
            <th>title</th>
            <th>price</th>
            <th>cover</th>
        </tr>
        {% for book in book_list %}
        <tr>
            <th>{{ book.id }}</th>
            <th>{{ book.title | title }}</th>
            <th>{{ book.price }}</th>
            <th>{{ book.cover | safe }}</th>
        </tr>
        {% endfor %}
    </table>
</body>
</html>
```

flask中， **过滤器的本质就是函数**。有时候我们不仅仅只是需要输出变量的值，我们还需要修改变量的显示，甚至格式化、运算等等，而在模板中是不能直接调用 Python 的方法，那么这就用到了过滤器。

使用方式：

- 过滤器的使用方式为：变量名 | 过滤器 | 。。。。。

```python
{{variable | filter_name(args1,args2,....)}}
```

- 如果没有任何参数传给过滤器,则可以把括号省略掉

```python
{{variable | title }}
```

- 如：`title`过滤器的作用：把变量的值的首字母转换为大写，其他字母转换为小写



在 jinja2 中，过滤器是可以支持链式调用的，示例如下：

```python
{{ "hello world" | reverse | upper }}
```



### 常见的内建过滤器

源代码：`from jinja2.filters import FILTERS`

#### 字符串操作

- safe：禁用实体字符的转义

```python
{{ '<h1>hello</h1>' | safe }}
```

- lower：把值转成小写

```python
<p>{{ 'HELLO' | lower }}</p>
```

- upper：把值转成大写

```python
<p>{{ 'hello' | upper }}</p>
```

- reverse：字符串反转

```python
<p>{{ 'olleh' | reverse }}</p>
```

- format：格式化输出

```python
<p>{{ '%s = %d' | format('name',17) }}</p>

<p>{{ '%s = %d' % ('name', 17) }}</p>
```

- striptags：渲染之前把值中所有的HTML标签都删掉

  如果内容中，存在大小于号的情况，则不要使用这个过滤器，容易误删内容。

```python
<p>{{ '<script>alert("hello")</script>' | striptags }}</p>
<p>{{ "如果x<y，z>x，那么x和z之间是否相等？" | striptags }}</p>
```

- truncate: 字符串截断

```python
<p>{{ 'hello every one' | truncate(9)}}</p>
```

#### 列表操作

- first：取第一个元素

```python
<p>{{ [1,2,3,4,5,6] | first }}</p>
```

- last：取最后一个元素

```python
<p>{{ [1,2,3,4,5,6] | last }}</p>
```

- length：获取列表长度

```python
<p>{{ [1,2,3,4,5,6] | length }}</p>
<p>{{ [1,2,3,4,5,6] | count }}</p>
```

- sum：列表求和

```python
<p>{{ [1,2,3,4,5,6] | sum }}</p>
```

- sort：列表排序

```
<p>{{ [6,2,3,1,5,4] | sort }}</p>
```

#### 语句块过滤

```pyhton
    {% filter upper %}
        <p>abc</p>
        <p>{{ ["a","c"] }}</p>
        <p>{{ ["a","c"] }}</p>
        <p>{{ ["a","c"] }}</p>
        <p>{{ ["a","c"] }}</p>
    {% endfilter %}
```



### 自定义过滤器

过滤器的本质是函数。当模板内置的过滤器不能满足项目需求，可以自定义过滤器。自定义过滤器有两种实现方式：

- 一种是通过Flask应用对象的 **app.add_template_filter** 方法进行注册
- 通过装饰器来实现自定义过滤器进行注册

**注意：自定义的过滤器名称如果和内置的过滤器重名，会覆盖内置的过滤器。**



需求：编写一个过滤器，保留2位小数

方式一

通过调用应用程序实例的 add_template_filter 方法实现自定义过滤器。该方法第一个参数是函数名，第二个参数是自定义的过滤器名称：

```python
# 自定义过滤器
def do_fixed(data, length):
    return f"%.{length}f" % data


# 注册过滤器到当前应用实例对象
app.add_template_filter(do_fixed, "fixed")
```

方式二

用装饰器来实现自定义过滤器。装饰器传入的参数是自定义的过滤器名称。

```python
# 自定义过滤器，通过装饰器注册到app应用实例对象
@app.template_filter("fixed")
def do_fixed(data, length):
    return f"%.{length}f" % data
```

- 主程序中创建和注册过滤器

```python
from flask import Flask, render_template

app = Flask(__name__, template_folder="templates")

# 自定义过滤器，通过装饰器注册到app应用实例对象
@app.template_filter("fixed")
def do_fixed(data):
    return f"{data:.2f}"

@app.route("/")
def index():
    title = "网页标题"
    book_list = [
        {"id":1, "price": 78.50, "title":"javascript入门", "cover": "<img src='/static/images/course.png'>"},
        {"id":2, "price": 78.5, "title":"python入门", "cover": "<img src='/static/images/course.png'>"},
        {"id":3, "price": 78.6666, "title":"django web项目实战", "cover": "<img src='/static/images/course.png'>"}
    ]
    return render_template("index9.html", **locals())

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000, debug=True)

```

- html9.html，调用过滤器，代码：

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
    table, td,th{
        border: 1px solid red;
        border-collapse: collapse;
    }
    table{
        width: 800px;
    }
    td,th{
        padding: 4px;
    }
    img{
        width: 100px;
    }
    </style>
</head>
<body>
    <table>
        <tr>
            <th>ID</th>
            <th>title</th>
            <th>price</th>
            <th>cover</th>
        </tr>
        {% for book in book_list %}
        <tr>
            <th>{{ book.id }}</th>
            <th>{{ book.title | title }}</th>
            <th>{{ book.price | fixed }}</th>
            <th>{{ book.cover | safe }}</th>
        </tr>
        {% endfor %}
    </table>
</body>
</html>
```



#### 练习：给手机进行部分屏蔽  `13112345678` ---> `131****5678`

```python
from flask import Flask, render_template

app = Flask(__name__, template_folder="templates", static_folder="static")
# 配置
app.config.update({
    "DEBUG": True
})

# 自定义过滤器
@app.template_filter("mobile")
def do_mobile(data, flag):
    return data[:3] + str(flag) + data[-4:]

@app.route("/")
def index():
    user_list = [
        {"id":1,"name":"张三","mobile":"13112345678"},
        {"id":2,"name":"张三","mobile":"13112345678"},
        {"id":3,"name":"张三","mobile":"13112345678"},
        {"id":4,"name":"张三","mobile":"13112345678"},
    ]
    return render_template("index5.html", **locals())


if __name__ == '__main__':
    app.run()

```

index2.html，模板代码：

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
    table, td,th{
        border: 1px solid red;
        border-collapse: collapse;
    }
    table{
        width: 800px;
    }
    td,th{
        padding: 4px;
    }
    img{
        width: 100px;
    }
    </style>
</head>
<body>
    <table>
        <tr>
            <th>ID</th>
            <th>name</th>
            <th>mobile</th>
        </tr>
        {% for user in user_list %}
        <tr>
            <th>{{ user.id }}</th>
            <th>{{ user.name }}</th>
            <th>{{ user.mobile | mobile("****") }}</th>
        </tr>
        {% endfor %}
    </table>
</body>
</html>
```

效果：

![image-20211228113255976](10Jinja2%E6%A8%A1%E6%9D%BF.assets/image-20211228113255976.png)



## 模板继承

在模板中，可能会遇到以下情况：

- 多个模板具有完全相同的顶部和底部内容
- 多个模板中具有相同的模板代码内容，但是内容中部分值不一样，弹窗
- 多个模板中具有完全相同的 html 代码块内容，侧边栏

像遇到这种情况，可以使用 JinJa2 模板中的 **模板继承** 来进行实现

模板继承是为了重用模板中的公共内容。一般Web开发中，继承主要使用在网站的顶部菜单、底部、弹窗。这些内容可以定义在父模板中，子模板直接继承，而不需要重复书写。

- block标签定义的可重写的内容范围

```python
{% block 区块名称 %} {% endblock 区块名称 %}

{% block 区块名称 %} {% endblock %}

例如：顶部菜单
{% block menu %}{% endblock %}
```

- block相当于在父模板中挖个坑，当子模板继承父模板时，可以进行对应指定同名区块进行代码填充。
- 子模板使用 extends 标签声明继承自哪个父模板
- 父模板中定义的区块在子模板中被重新定义，在子模板中调用父模板的内容可以使用super()调用父模板声明的区块内容。



manage.py，视图代码：

```python
from flask import Flask, render_template

app = Flask(__name__, template_folder="templates", static_folder="static")
# 配置
app.config.update({
    "DEBUG": True
})

@app.route("/")
def index():
    title = "我的首页"
    return render_template("base.html", **locals())

@app.route("/list")
def list_page():
    title = "商品列表"
    content = "商品列表ng "
    return render_template("list.html", **locals())

if __name__ == '__main__':
    app.run()

```



父模板代码：

base.html

```jinja2
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ title | default("默认标题") }}</title>
    {% block css %}
    {% endblock %}
</head>
<body>
    <h1>头部公共内容-菜单-登录窗口</h1>
    {% block main %}
    <div>公共页面中间部分内容 - {{ title }}</div>
    {% endblock main %}
    <h1>脚部公共内容-版权信息，网点地图</h1>
</body>
</html>
```



子模板代码：

- extends指令声明这个模板继承自哪

list.html，代码：

```jinja2
{% extends "./base.html" %}

{% block css %}
    <style>
    body{
        color: red;
    }
    </style>
{% endblock css %}

{% block main %}
    {{ super() }} {# 继承父级模板的内容 #}
    <p>{{ content }}</p>
{% endblock %}
```



模板继承使用时注意点：

1. 不支持多继承，不能使用多个extends

2. 为了便于阅读，在子模板中使用extends时，尽量写在模板的第一行。

3. 不能在一个模板文件中定义多个相同名字的block标签，否则会覆盖。

4. 当在页面中使用多个block标签时，建议给结束标签起个名字，当多个block嵌套时，阅读性更好。

## CSRF 攻击防范

CSRF: 跨域请求伪造攻击。

```bash
pip install flask_wtf
```

flask_wtf本身提供了生成表单HTML页面的功能(基于wtforms提供)，常用于开发前后端不分离的表单页面，同时Flask-wtf 扩展模块还提供了一套完善的 csrf 防护体系，对于我们开发者来说，使用flask_wtf模块就可以非常简单解决CSRF攻击问题。

1. 设置应用程序的 secret_key，用于加密生成的 csrf_token 的值

```python
# 1. session加密的时候已经配置过了.如果没有在配置项中设置,则如下:
app.secret_key = "#此处可以写随机字符串#"

# 2. 也可以写在配置类中。
class Config(object):
    DEBUG = True
    SECRET_KEY = "dsad32DASSLD*13%^32"
    
"""加载配置"""
app.config.from_object(Config)
```



2. 导入 flask_wtf 中的 CSRFProtect类，进行初始化，并在初始化的时候关联 app

```python
# 方式1：
from flask_wtf import CSRFProtect
csrf = CSRFProtect() # 这块代码可能在文件中。
app = Flask(import_name=__name__, template_folder="templates")
# 项目配置代码之后
csrf.init_app(app) # 避免出现引用导包，所以提供了init_app的用法

# 方式2：
# from flask_wtf import CSRFProtect
# app = Flask(import_name=__name__, template_folder="templates")
# 项目配置代码之后
# CSRFProtect(app)
```



1. 在表单中使用 CSRF 令牌:

```html
    <form action="/login" method="post">
        <input type="hidden" name="csrf_token" value="{{ csrf_token() }}" />
    </form>
```



视图代码;

```python
from flask import Flask, render_template, request, session
from flask_wtf import CSRFProtect
from flask_wtf.csrf import generate_csrf
app = Flask(__name__, template_folder="templates", static_folder="static")

# csrf = CSRFProtect()



# 配置
app.config.update({
    "DEBUG": True,
    "SECRET_KEY": ":123.,2,s,"
})

# csrf.init_app(app)

CSRFProtect(app)



@app.route("/")
def index():
    title = "网站首页"
    return render_template("index.html", **locals())


@app.route("/login", methods=["GET","POST"])
def login():
    title = "登录页面"
    print(">>>> 1")
    if request.method == "GET":
        print(">>>> 2")
        token = generate_csrf()
        return render_template("login.html", **locals())
    else:
        print(">>>> 3")
        username = request.form.get("username")
        password = request.form.get("password")
        if username == "xiaoming" and password == "123456":
            print(">>>> 4")
            session["username"] = "xiaoming"
            session["is_login"] = True
            print(">>>> 6")
            return "登录成功！<br><a href='/'>返回首页</a>"
        else:
            print(">>>> 5")
            return "登录失败！<br><a href='/login'>重新登录！</a>"

if __name__ == '__main__':
    app.run()

```

login.html，模板代码：

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
</head>
<body>
    <form action="http://127.0.0.1:5000/login" method="post">
        <input type="hidden" name="csrf_token" value="{{ token2() }}" />
        <input type="hidden" name="csrf_token" value="{{ token }}" />
        登录账号： <input type="text" name="username"> <br><br>
        登录密码： <input type="password" name="password"> <br><br>
        <button>登录</button>
    </form>
</body>
</html>
```

index.html，代码：

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
</head>
<body>
    {% if session.get("is_login") %}
        <a href="/user">欢迎回到网站，{{ session["username"] }}</a>
    {% else %}
    <a href="{{ url_for('login') }}">登录</a>
    {% endif %}
</body>
</html>
```

