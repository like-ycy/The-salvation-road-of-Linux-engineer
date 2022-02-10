# Tornado

文档：https://tornado-zh-cn.readthedocs.io/zh_CN/latest/

github：https://github.com/tornadoweb/tornado

## 介绍

Tornado是使用Python开发的全栈式（full-stack）Web框架和异步网络库，最早由4名Google前软件工程师（布雷特·泰勒）2007创办的Friendfeed(一个社交聚合网站)开发而来的。通过使用非阻塞IO，Tornado可以处理数以万计的开放连接，是long polling、WebSockets和其他需要为用户维护长连接应用的理想选择。

目前最新版本6.1, 我们实际项目开发是使用的不可能是最新版本，所以在此我们在tornado基础阶段所学所用的版本为6.0.

### 特点

- 开源的轻量级全栈式Web框架，提供了一整套完善的异步编码方案。

- 高性能

  基于协程，底层就是基于asyio来实现的完整的协程调度

  采用异步非阻塞IO处理方式，不依赖多进程或多线程
  采用单进程单线程异步IO的网络模式，其高性能源于Tornado基于Linux的Epoll（UNIX为kqueue）的异步网络IO，具有出色的抗负载能力

  Tornado为了实现高并发和高性能，使用了一个`IOLoop`事件循环来处理`socket`的读写事件

- WSGI全栈替代产品，Tornado把应用（Application）和服务器（Server）结合起来，既是WSGI应用也可以是WSGI服务，通俗来讲就是说，Tornado既是web服务器也是web框架，甚至可以通过Tornado替代uwsgi/gunicorn来运行Flask或者django框架

django,flask和tornado对比

```
内置功能模块来说: django > flask > tornado
使用入门门槛: django < flask < tornado
```

Tornado 可以被分为以下四个主要部分:

- Web 框架 (包括用来创建 Web 应用程序的 [`RequestHandler`](https://tornado-zh-cn.readthedocs.io/zh_CN/latest/web.html#tornado.web.RequestHandler) 类, 还有很多其它支持的类).
- HTTP 客户端和服务器的实现 ([`HTTPServer`](https://tornado-zh-cn.readthedocs.io/zh_CN/latest/httpserver.html#tornado.httpserver.HTTPServer) 和 [`AsyncHTTPClient`](https://tornado-zh-cn.readthedocs.io/zh_CN/latest/httpclient.html#tornado.httpclient.AsyncHTTPClient)).
- 异步网络库 ([`IOLoop`](https://tornado-zh-cn.readthedocs.io/zh_CN/latest/ioloop.html#tornado.ioloop.IOLoop) 和 [`IOStream`](https://tornado-zh-cn.readthedocs.io/zh_CN/latest/iostream.html#tornado.iostream.IOStream)), 对 HTTP 的实现提供构建模块, 还可以用来实现其他协议.
- 协程库 ([`tornado.gen`](https://tornado-zh-cn.readthedocs.io/zh_CN/latest/gen.html#module-tornado.gen)) 让用户通过更直接的方法来实现异步编程, 而不是通过回调的方式.



### 安装

```bash
mkvirtualenv tornado_demo
pip install tornado==6.0.4
cd ~/Desktop
mkdir tdemo
cd tdemo
```

## 入门

### 项目基本运行

server.py

```python
from tornado import ioloop
from tornado import web

class Home(web.RequestHandler):
    def get(self):
		# self.write 响应数据
        self.write("hello!")

def make_app():
    # Application是tornado web框架的核心应用类，是与服务器对应的接口，里面保存了路由映射表
    # handlers 设置路由列表
    return web.Application(handlers=[
        (r"/", Home),
    ])

if __name__ == "__main__":
    # 创建应用实例对象
    app = make_app()
    # 设置监听的端口和地址
    app.listen(8888)
    # ioloop，全局的tornado事件循环，是服务器的引擎核心，start表示创建IO事件循环,等待客户端连接
    ioloop.IOLoop.current().start()
```

![img](1Tornado%E4%BB%8B%E7%BB%8D.assets/4933701-8ff98bf8d7bda560.png)

### 终端运行项目

server.py，代码：

```python
from tornado import ioloop
from tornado import web
from tornado.options import define,options,parse_command_line
define("port", default=8888, type=int,help="设置监听端口号，默认为8888")
class Home(web.RequestHandler):
    def get(self):
		# self.write 响应数据
        self.write("hello!")

def make_app():
    # handlers 设置路由列表
    return web.Application(handlers=[
        (r"/", Home),
    ])

if __name__ == "__main__":
    # 解析终端启动命令，格式：python server.py --port=端口号
    parse_command_line()
    # 创建应用实例对象
    app = make_app()
    # 设置监听的端口和地址
    app.listen(options.port) # options.port 接收参数
    # ioloop，全局的tornado事件循环，是服务器的引擎核心，start表示创建IO事件循环
    ioloop.IOLoop.current().start()
```

### 开启调试模式

开启自动加载和调试模式，开启了debug模式以后, 项目在编辑python文件的时候自动重启项目并且在出现异常时显示错误跟踪信息

server.py, 代码：

```python
from tornado import ioloop
from tornado import web
from tornado import autoreload
from tornado.options import define,options,parse_command_line

# 配置信息
settings = {
    'debug' : True,
}

define("port", default=8888, type=int,help="设置监听端口号，默认为8888")

# 类视图
class Home(web.RequestHandler):
    # 视图方法
    def get(self):
		# self.write 响应数据
        self.write("hello!")

def make_app():
    # handlers 设置路由列表
    return web.Application(handlers=[
        (r"/", Home),
    ],**settings) # 加载配置

if __name__ == "__main__":
    # 创建应用实例对象
    parse_command_line()
    app = make_app()
    # 设置监听的端口和地址
    app.listen(options.port)
    # ioloop，全局的tornado事件循环，是服务器的引擎核心，start表示创建IO事件循环
    ioloop.IOLoop.current().start()
```

### 路由拆分

代码：

```python
from tornado import ioloop
from tornado import web
from tornado.options import define,options,parse_command_line
# 项目配置
settings = {
    "debug": True, # 开启debug模式
}

# 视图类必须要直接或者间接继承于 web.RequestHandler
class Home(web.RequestHandler):
    def get(self): # http请求
        # 响应数据
        self.write("<h1>hello! oldboyEdu</h1>")
        self.write("hello tornado!") # 这里也是可以被执行的

        return
        self.write("hello world") # 只有在return以后，才不会被执行

# 路由列表
urls = [
    # (r"uri路径", 视图类),
    (r"/", Home),
]

if __name__ == "__main__":

    # 定义终端命令行参数
    define(name="port",default=8888,type=int) # python 主程序.py --port=8888
    define(name="host", default="127.0.0.1", type=str)  # python 主程序.py --port=8888 --host=127.0.0.1
    define(name="debug", default=False, type=bool) # # python 主程序.py --debug=True
    # 启动终端命令行参数解析方法
    parse_command_line()
    # 创建web应用实例对象

    # Application是tornado web框架的核心应用类，是与服务器对应的接口，里面保存了路由映射表
    settings["debug"] = options.debug
    app = web.Application(
        urls,
        **settings,
    )
    # 设置监听的端口和地址
    app.listen(port=options.port,address=options.host)
    # ioloop，全局的tornado事件循环，是服务器的引擎核心，start表示创建IO事件循环,等待客户端连接
    ioloop.IOLoop.current().start()
```



### 视图编写

在tornado中, tornado.web框架本身就默认提供了rest风格的API接口模式. 可以直接通过CBV(类视图)对外提供基本的http 视图接口.

```python
from tornado import ioloop
from tornado import web
from tornado.options import define,options,parse_command_line
# 项目配置
settings = {
    "debug": True, # 开启debug模式
}

# 视图类必须要直接或者间接继承于 web.RequestHandler
class Home(web.RequestHandler):
    def get(self): # http请求
        # 响应数据
        self.write("<h1>hello! oldboyEdu</h1>")
        self.write("hello tornado!") # 这里也是可以被执行的

        return
        self.write("hello world") # 只有在return以后，才不会被执行

    def post(self):
        self.write("hello!post")

    def put(self):
        self.write("hello!put")

    def patch(self):
        self.write("hello!patch")

    def delete(self):
        self.write("hello!delete")

# 路由列表
urls = [
    # (r"uri路径", 视图类),
    (r"/", Home),
]

if __name__ == "__main__":

    # 定义终端命令行参数
    define(name="port",default=8888,type=int) # python 主程序.py --port=8888
    define(name="host", default="127.0.0.1", type=str)  # python 主程序.py --port=8888 --host=127.0.0.1
    define(name="debug", default=False, type=bool) # # python 主程序.py --debug=True
    # 启动终端命令行参数解析方法
    parse_command_line()
    # 创建web应用实例对象

    # Application是tornado web框架的核心应用类，是与服务器对应的接口，里面保存了路由映射表
    settings["debug"] = options.debug
    app = web.Application(
        urls,
        **settings,
    )
    # 设置监听的端口和地址
    app.listen(port=options.port,address=options.host)
    # ioloop，全局的tornado事件循环，是服务器的引擎核心，start表示创建IO事件循环,等待客户端连接
    ioloop.IOLoop.current().start()
```



### 多进程模式

```python
from tornado import ioloop
from tornado import web,httpserver
from tornado import autoreload
from tornado.options import define,options,parse_command_line

settings = {
    'debug' : False,
}

define("port", default=8888, type=int,help="设置监听端口号，默认为8888")

class Home(web.RequestHandler):
    def get(self):
        # self.write 响应数据
        self.write("hello!get")

    def post(self):
        self.write("hello!post")

    def put(self):
        self.write("hello!put")

    def patch(self):
        self.write("hello!patch")

    def delete(self):
        self.write("hello!delete")

# 路由列表
urls = [
    (r"/", Home),
]

if __name__ == "__main__":
    app = web.Application(urls,**settings)
    # 创建应用实例对象
    parse_command_line()
    server = httpserver.HTTPServer(app)
    # 设置监听的端口和地址
    server.bind(options.port)
    server.start(0) # 0表示进程=CPU核数
    # ioloop，全局的tornado事件循环，是服务器的引擎核心，start表示创建IO事件循环
    ioloop.IOLoop.current().start()
```
