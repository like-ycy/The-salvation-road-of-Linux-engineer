# Flask路由

##  一、路由的本质

flask 的路由直接在函数上使用装饰器的话非常的乱，那可不可以像 django 一样把路由都写到一起呢。

查看源码，我们知道**原来的路由就是一个装饰器**，装饰器的第一个参数就是路由规则

实际上在**装饰器内的闭包函数最终调用的是** `def add_url_rule(self, rule, endpoint=None, view_func=None, provide_automatic_options=None, **options):`

> rule: 路由规则
> endpoint: 反向解析别名，和CBV中as_view(name=...)的name是一样的。因为在源码内部，实际上也会去取参数view_func的__name__也就是指定的函数或类的名字
> view_func: 响应对象

因此我们可以直接改写路由。

将原来装饰器修饰的路由，改写为使用 `add_url_rule` 的路由

```python
from  flask import Flask
app = Flask(__name__)

@app.route("/")
def index():
    return "ok"

# 路由匹配规则和响应函数
app.add_url_rule("/index", view_func=index)

if __name__ == '__main__':
    app.run()
```

## 二、CBV

和 django 一样，flask 也是有 CBV 的，都是通过 as_view () 来返回 view 对象，通过匹配然后执行 view 函数完成 dispatch_request 的响应方式的分发。

不同的是。**as_view()中必须传递一个 name，这个 name 就是作为反向解析的名字给类的__name__**

**flask 导入的 views. View 类必须实现 dispatch_request 方法**，不然匹配路由成功后执行 view 函数完成分发会报错。因为源码中必须让你实现

```python
from flask import Flask,views
app = Flask(__name__)

#简单模式
class IndexView(views.View):
    def dispatch_request(self):
        print('Index')
        return 'Index!'

app.add_url_rule('/index1', view_func=IndexView.as_view(name='index1'))	# name=endpoint 反向解析的别名

if __name__ == '__main__':
    app.run()
```

### 2.1 使用 views. MethodView 类

这个类已经帮我们实现好了 `dispatch_request` 方法，就不需要自己再去写了。而他帮你实现的 `dispatch_request` 方法中做的事和 Django 中的 dispath 方法中做的事情是一致的。都是通过反射去取响应的请求方式名同名的函数

```python
from flask import Flask,views
app = Flask(__name__)

#通常用此方式
class IndexView(views.MethodView):
        def get(self):
            return 'Index.GET'

        def post(self):
            return 'Index.POST'

app.add_url_rule('/index', view_func=IndexView.as_view(name='index'))  # name=endpoint

if __name__ == '__main__':
    app.run()
```

### 2.2 指定响应的请求方式类型

无论是 FBV 还是 CBV 的实现方式都可以在函数内部或类内部添加 `methods = ['GET', "POST"]` . 当 `app.add_url_rule` 执行时会通过反射去函数或类内部的 `methods` 中取指定的响应方法，如果没有写，默认只响应 get 方式请求。

但如果是 CBV 的方式，没有写 `methods` 则和 django 一样去取类内部实现的方法。但如果写了 `methods` ，只会响应 `methods` 中的请求方式对应的方法

```python
from flask import Flask,views
app = Flask(__name__)

class IndexView(views.MethodView):
        methods = ['GET']	# 只响应GET请求对于的get方法

        def get(self):
            return 'Index.GET'

        def post(self):
            return 'Index.POST'

app.add_url_rule('/index', view_func=IndexView.as_view(name='index'))

if __name__ == '__main__':
    app.run()
```

### 2.3 指定装饰器修饰

在使用 CBV，添加对应的路由是需要通过 as_view 来得到 view 对象。而在 as_view () 内部，会根据类属性 `decorators=[auth, ]` 来执行 [] 中对应的装饰器。也就是说，当程序执行时，进入 as_view () 后，会给该装饰器传入一个 view 对象，然后进入对应的装饰器，该装饰器必须返回一个 view 对象，否则就会报错。

传进去一个 view，返回一个同名的 view 对象，这不就是装饰器嘛。给 view 加了一层装饰器，无论哪个路由匹配成功，都会先进入该装饰器。

```python
from flask import Flask,views,url_for,jsonify,render_template
app = Flask(__name__)

def auth(view):
    def inner(*args,**kwargs):
        if view.__name__=="index":
            res = view(*args,**kwargs)
        else:
            # res = render_template("404.html")
            res = jsonify({"status": 0,"msg":"傻逼"})
        return res
    return inner

#通常用此方式
class IndexView(views.MethodView):
        # 给dispatch_request加装饰器
        decorators = [auth, ]

        def get(self):
            return 'Index.GET'

        def post(self):
```

## 三、路由参数

`@app.route` 和 `app.add_url_rule` 参数:

```
rule:路由规则

endpoint:反向解析别名，和CBV中as_view(name=...)的name是一样的。因为在源码内部，实际上也会去取参数view_func的__name__也就是指定的函数或类的名字

view_func:响应对象

defaults:默认值为None, 当URL中无参数，函数需要参数时，使用defaults = {'k': 'v'}

methods:允许的请求方式，如：["GET", "POST"]

strict_slashes: 对URL最后的 / 符号是否严格要求，默认等于None

redirect_to: 重定向到指定地址
```

* **url_for**: 可以根据反向解析的别名来拿到对应路由

```python
from  flask import Flask,views,url_for
app = Flask(__name__)

@app.route("/index/",endpoint="a",methods=["POST","GET"],strict_slashes=None,redirect_to="/index2")
def index():
    return "ok"

@app.route("/index2")
def index2():
    # 打印反向解析的别名是a的路由
    print(url_for("a"))
    return "吃饭去了"

if __name__ == '__main__':
    app.run()
```

### 接收任意路由参数

```python
# 路由可以接收不限定类型的路由参数，默认按/分割参数
@app.route(rule="/users/<cid>/<num>")
# 路由设置了路由参数，在被装饰的视图中必须声明同名的视图参数用于接收，否则报错
def index(num, cid):
    print(cid, type(cid))
    print(num, type(num)) # <class 'str'>
    return f"<h1>cid={cid}, num={num}, flask!!!</h1>"

# 可以发现，上面的路由参数在视图接收以后，默认格式是字符串，即便url地址中写的数字，
# 所以我们需要对路由参数的类型或者格式要进一步的约束
```

### 接收限定类型参数

限定路由参数的类型，flask系统自带转换器编写在werkzeug/routing.py文件中。底部可以看到以下字典：

```python
#: the default converter mapping for the map.
DEFAULT_CONVERTERS: t.Mapping[str, t.Type[BaseConverter]] = {
    "default": UnicodeConverter, # 默认类型，也就是string
    "string": UnicodeConverter, # 字符串，不包含 /
    "any": AnyConverter,    # 任意类型
    "path": PathConverter,  # 也是字符串，但是包含了 /
    "int": IntegerConverter,
    "float": FloatConverter,
    "uuid": UUIDConverter,
}
```

系统自带的转换器具体使用方式在每种转换器的注释代码中有写，请留意每种转换器初始化的参数。

| 转换器名称 | 描述                                                    |
| ---------- | ------------------------------------------------------- |
| string     | 默认类型，接受不带斜杠的任何文本                        |
| int        | 接受正整数                                              |
| float      | 接受正浮点值                                            |
| path       | 接收 `string` 但也接受斜线                                |
| uuid       | 接受UUID（通用唯一识别码）字符串  xxxx-xxxx-xxxxx-xxxxx |

代码：

```python
from flask import Flask

# 项目实例应用对象
app = Flask(__name__)

# 路由可以接收不限定类型的路由参数，默认按/分割参数
@app.route(rule="/users/<cid>/<num>")
# 路由设置了路由参数，在被装饰的视图中必须声明同名的视图参数用于接收，否则报错
def index(num, cid):
    print(cid, type(cid))
    print(num, type(num)) # <class 'str'>
    return f"<h1>cid={cid}, num={num}, flask!!!</h1>"

# 可以发现，上面的路由参数在视图接收以后，默认格式是字符串，即便url地址中写的数字，
# 所以我们需要对路由参数的类型或者格式要进一步的约束

# 限定类型的路由参数
# 不仅可以转换参数的数据类型，还可以达到匹配的目的，匹配不上，则404
@app.route("/goods/<int:cid>")
def goods_list(cid):
    print(cid, type(cid))  # <class 'int'>
    return f"cid={cid}"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

## 四、路由转换器

路由转化器，其实就是 django 中路由的有名分组，通过参数名来得到路由的参数。区别是不用正则，但是可以指定参数类型。

```python
from  flask import Flask
app = Flask(__name__)

# 和有名分组一样，只不过不用正则了
@app.route("/<string:flag>")
def index(flag):
    return f"jason is sb ? {flag}"

if __name__ == '__main__':
    app.run()
```

**默认路由转化器参数类型**

```python
DEFAULT_CONVERTERS = {
    'default':          UnicodeConverter,
    'string':           UnicodeConverter,
    'any':              AnyConverter,
    'path':             PathConverter,
    'int':              IntegerConverter,
    'float':            FloatConverter,
    'uuid':             UUIDConverter,
}
```

**4.1 自定义转换器**

* 1 写类，继承 BaseConverter，实现三个方法 init、to_python、to_url
* 2 注册：`app.url_map.converters['regex'] = RegexConverter`
* 3 使用：`@app.route('/index/<regex("\d+"):nid>')` 正则表达式会当作第二个参数传递到类中

```python
#1 写类，继承BaseConverter，实现三个方法init、to_python、to_url
#2 注册：app.url_map.converters['regex'] = RegexConverter
#3 使用：@app.route('/index/<regex("\d+"):nid>')  正则表达式会当作第二个参数传递到类中
from flask import Flask, views, url_for
from werkzeug.routing import BaseConverter

app = Flask(import_name=__name__)

class RegexConverter(BaseConverter):
    """
    自定义URL匹配正则表达式
    """
    def __init__(self, map, regex):
        super(RegexConverter, self).__init__(map)
        self.regex = regex

    def to_python(self, value):
        """
        路由匹配时，匹配成功后传递给视图函数中参数的值
        """
        return int(value)+123

    def to_url(self, value):

 # 正则匹配路由
@app.route("/sms/<regex('1[3-9]\d{9}'):mobile>")
def sms(mobile):
    return f"mobile={mobile}"
```
