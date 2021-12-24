## 一个最小的应用

一个最小的 Flask 应用如下：

```python
from flask import Flask

app = Flask (__name__)

def hello_world () :
    return "<p>Hello, World!</p>"

if __name__ == '__main__':
    app.run ()
```

那么，这些代码是什么意思呢？

1.  首先我们导入了 [`Flask`](https://dormousehole.readthedocs.io/en/latest/api.html#flask.Flask) 类。该类的实例将会成为我们的 WSGI 应用。
2.  接着我们创建一个该类的实例。第一个参数是应用模块或者包的名称。 `__name__` 是一个适用于大多数情况的快捷方式。有了这个参数， Flask 才能知道在哪里可以找到模板和静态文件等东西。
3.  然后我们使用 [`route ()`](https://dormousehole.readthedocs.io/en/latest/api.html#flask.Flask.route) 装饰器来告诉 Flask 触发函数的 URL 。
4.  函数返回需要在用户浏览器中显示的信息。默认的内容类型是 HTML ，因此字符串中的 HTML 会被浏览器渲染。

把它保存为 `hello.py` 或其他类似名称。请不要使用 `flask.py` 作为应用名称，这会与 Flask 本身发生冲突。

## 运行应用

推荐使用 `flask run` 命令行脚本运行开发服务器。这需要设置 `FLASK_APP` 环境变量指向您的应用，且设置 `FLASK_ENV=development` ， 以用于启用开发模式。

```bash
$ export FLASK_APP=hello.py
$ export FLASK_ENV=development
$ flask run
```

这样就启动了开发环境，包括交互调试器和重载器，并在 http://localhost:5000/ 提供服务。

`flask run` 默认使用 127.0.0.1 和 5000 端口，以及 debug=False 模式来运行项目

也可以手动指定监听 ip 和端口，修改启动方法

```bash
if __name__ == '__main__':
     app.run (host="0.0.0.0",port=5000, debug=True)
```

## 调试模式

1.  flask 代码中如果出现了异常，我们在浏览器中不会提示具体的错误信息，开启 debug 模式后会把具体的错误信息发送到浏览器上。
2.  flask 代码如果被修改了，必须要重启项目修改的代码才会有效，开启 debug 模式后我们修改了代码只要 `ctrl+s` 我们的 flask 项目就会自动重新加载，不需要手动加载整个网站。

打开 Debug 模式的两种方式：

其一：

```python
if __name__ == '__main__':
     app.run (host="0.0.0.0",port=5000, debug=True) # debug=True 打开调试模式
```

其二，导入配置

```bash
from flask import Flask
app = Flask (__name__)

# 注册配置
settings = {
    "DEBUG": True
}
app.config.update (settings)

@app.route ("/")
def index () :
    return "hello, flask!!!"

if __name__ == '__main__':
    app.run ()
```

## Flask 加载配置的方式

flask 支持通过字典、模块、对象对项目配置进行扩展

```python
from flask import Flask
app = Flask (__name__)

# 方法一：字典
settings = {
    "DEBUG": True
}
app.config.update (settings)

# 方法二：对象
class Config (object) :
    DEBUG = True

app.config.from_object (Config) # 对象

# 方法三：模块
import settings
app.config.from_object (settings) # 模块

# 路由和视图
@app.route ("/")
def index () :
    return "<h1>hello, flask!!!</h1>"

if __name__ == '__main__':
    app.run (host='0.0.0.0', port=5000)
```
