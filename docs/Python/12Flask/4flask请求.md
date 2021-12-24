# Flask 请求

## flask的生命周期

客户端--->wsgi应用程序->全局钩子--> 路由 --> 视图 --> 路由---> 全局钩子 ---> wsgi应用程序---> 客户端

## 请求

文档: https://flask.palletsprojects.com/en/2.0.x/api/#flask.Request

* **request**：flask中代表当前请求的 `request 对象`
* **作用**：在视图函数中取出本次客户端的请求数据
* **导入**：``from flask import request``
* **代码位置**：
  + 代理类  from flask.app import Request  ---> from flask.globals. Request
  +  源码类：from flask.wrappers. Request
  + 基类：from werkzeug.wrappers import Request as RequestBase

request，常用的属性如下：

| 属性    | 说明                                                         | 类型               |
| ------- | ------------------------------------------------------------ | ------------------ |
| data    | 记录请求体的数据，并转换为字符串<br>只要是通过其他属性无法识别转换的请求体数据<br>最终都是保留到data属性中<br>例如：有些公司开发微信小程序，原生IOS或者安卓，这一类客户端有时候发送过来的数据就不一样是普通的表单，查询字符串或ajax | bytes类型          |
| form    | 记录请求中的html表单数据                                     | ImmutableMultiDict |
| args    | 记录请求中的查询字符串, 也可以是query_string                  | ImmutableMultiDict |
| cookies | 记录请求中的cookie信息                                       | Dict               |
| headers | 记录请求中的请求头                                           | ImmutableMultiDict |
| method  | 记录请求使用的HTTP方法                                       | GET/POST           |
| url     | 记录请求的URL地址                                            | string             |
| files   | 记录请求上传的文件列表                                       | ImmutableMultiDict |
| json    | 记录ajax请求的json数据                                       | Dict               |

### 获取请求中各项数据

获取查询字符串，代码：

```python
from flask import Flask, request
from werkzeug.datastructures import ImmutableMultiDict
from werkzeug.wrappers import Request as RequestBase
from flask.wrappers import Request
# 项目实例应用对象
app = Flask(__name__)

# 加载配置
app.config.update({
    "DEBUG": True
})

# 在http的常用请求方法中，delete和get是没有请求体的！！！

@app.route(rule="/data", methods=["post", "put", "patch"])
def data():
    """获取请求体"""
    # 获取原生的请求体数据[当request对象的其他属性没法接受请求体数据时，会把数据保留在data中，如果有request对象的属性处理了请求体数据，则data就不再保留]
    # print(request.data)  # 如果客户端上传的是xml文档，html格式，二进制流格式，base64格式，就要使用data来接收

    """
    1. 没有上传任何数据：
        b''
    2. 上传json数据
        b'{\n    "username": "xiaoming",\n    "age": 16\n}'
    3. 上传表单数据
        b''
    4. 上传xml数据
        b'<goods-list>\n    <goods price="100">\xe5\x95\x86\xe5\x93\x81\xe6\xa0\x87\xe9\xa2\x981</goods>\n    <goods price="200">\xe5\x95\x86\xe5\x93\x81\xe6\xa0\x87\xe9\xa2\x982</goods>\n</goods-list>'
    """

    # 接收表单上传的数据
    # print(request.form)
    """
    ImmutableMultiDict([('name', '小明'), ('age', '19')])
    """

    # 接收ajax上传的json数据
    # print(request.json)     # {"username": "xiaoming", "age": 16}
    # print(request.is_json)  # True 往往用于判断是否是ajax请求

    # 上传文件列表 HTML必须以<form method="post" enctype="multipart/form-data"> # 表单属性才能上传文件
    # print(request.files)
    """
    ImmutableMultiDict([('avatar', <FileStorage: 'avatar.jpg' ('image/jpeg')>)])
    """

    # 接受上传文件
    # avatar = request.files["avatar"]
    # print(avatar)
    """
    <FileStorage: 'avatar.jpg' ('image/jpeg')>
    from werkzeug.datastructures import FileStorage
    FileStorage，上传文件处理对象，flask封装的一个上传文件处理对象，可以允许我们直接调用对应的方法进行文件的存储处理，
    也可以结合其他的ORM模块像djangoORM那样通过模型操作对上传自动存储处理
    """
    # 处理上传文件[一般不会这么做！！！而是采用专业的第三方存储设备来存储，]
    # from pathlib import Path
    # save_path = str(Path(__file__).parent / "uploads/avatar.jpeg")
    # avatar.save(save_path)

    # 获取请求头信息
    print(request.headers)  # 获取全部的而请求头信息
    print(request.headers.get("Host"))  # 127.0.0.1:5000，客户端请求地址，也相当于服务器地址

    # 获取客户端发送过来的自定义请求头
    print(request.headers.get("company"))  # beijing，不存在的键的结果：None，存在则得到就是值，
    print(request.headers.get("token"))    # jwt...xxx

    # 获取客户端的请求头中的相关数据
    print(request.user_agent)   # 用户访问服务器时使用的网络代理，一般就是浏览器标记信息，PostmanRuntime/7.26.10
    print(request.remote_addr)  # 客户端远程地址
    print(request.server)       # 服务端的端点，格式：(IP, 端口)

    # 获取请求方法
    print(request.method)  # POST

    # 本次请求的url地址
    print(request.url)  # http://127.0.0.1:5000/data
    print(request.root_url)  # 根路径
    print(request.path)      # /data

    return "获取请求体"

if __name__ == '__main__':
    app.run()

```

获取请求体，代码：

```python
from flask import Flask, request
from werkzeug.datastructures import ImmutableMultiDict
from werkzeug.wrappers import Request as RequestBase
from flask.wrappers import Request
# 项目实例应用对象
app = Flask(__name__)

# 加载配置
app.config.update({
    "DEBUG": True
})

# 在http的常用请求方法中，delete和get是没有请求体的！！！

@app.route(rule="/data", methods=["post", "put", "patch"])
def data():
    """获取请求体"""
    # 获取原生的请求体数据[当request对象的其他属性没法接受请求体数据时，会把数据保留在data中，如果有request对象的属性处理了请求体数据，则data就不再保留]
    # print(request.data)  # 如果客户端上传的是xml文档，html格式，二进制流格式，base64格式，就要使用data来接收

    """
    1. 没有上传任何数据：
        b''
    2. 上传json数据
        b'{\n    "username": "xiaoming",\n    "age": 16\n}'
    3. 上传表单数据
        b''
    4. 上传xml数据
        b'<goods-list>\n    <goods price="100">\xe5\x95\x86\xe5\x93\x81\xe6\xa0\x87\xe9\xa2\x981</goods>\n    <goods price="200">\xe5\x95\x86\xe5\x93\x81\xe6\xa0\x87\xe9\xa2\x982</goods>\n</goods-list>'
    """

    # 接收表单上传的数据
    # print(request.form)
    """
    ImmutableMultiDict([('name', '小明'), ('age', '19')])
    """

    # 接收ajax上传的json数据
    # print(request.json)     # {"username": "xiaoming", "age": 16}
    # print(request.is_json)  # True 往往用于判断是否是ajax请求

    # 上传文件列表 HTML必须以<form method="post" enctype="multipart/form-data"> # 表单属性才能上传文件
    # print(request.files)
    """
    ImmutableMultiDict([('avatar', <FileStorage: 'avatar.jpg' ('image/jpeg')>)])
    """

    # 接受上传文件
    # avatar = request.files["avatar"]
    # print(avatar)
    """
    <FileStorage: 'avatar.jpg' ('image/jpeg')>
    from werkzeug.datastructures import FileStorage
    FileStorage，上传文件处理对象，flask封装的一个上传文件处理对象，可以允许我们直接调用对应的方法进行文件的存储处理，
    也可以结合其他的ORM模块像djangoORM那样通过模型操作对上传自动存储处理
    """
    # 处理上传文件[一般不会这么做！！！而是采用专业的第三方存储设备来存储，]
    # from pathlib import Path
    # save_path = str(Path(__file__).parent / "uploads/avatar.jpeg")
    # avatar.save(save_path)

    # 获取请求头信息
    print(request.headers)  # 获取全部的而请求头信息
    print(request.headers.get("Host"))  # 127.0.0.1:5000，客户端请求地址，也相当于服务器地址

    # 获取客户端发送过来的自定义请求头
    print(request.headers.get("company"))  # beijing，不存在的键的结果：None，存在则得到就是值，
    print(request.headers.get("token"))    # jwt...xxx

    # 获取客户端的请求头中的相关数据
    print(request.user_agent)   # 用户访问服务器时使用的网络代理，一般就是浏览器标记信息，PostmanRuntime/7.26.10
    print(request.remote_addr)  # 客户端远程地址
    print(request.server)       # 服务端的端点，格式：(IP, 端口)

    # 获取请求方法
    print(request.method)  # POST

    # 本次请求的url地址
    print(request.url)  # http://127.0.0.1:5000/data
    print(request.root_url)  # 根路径
    print(request.path)      # /data

    return "获取请求体"

if __name__ == '__main__':
    app.run()

```

### 总结

```bash
    # request.method  提交的方法
    # request.args  get请求提及的数据
    # request.form   post请求提交的数据
    # request.values  post和get提交的数据总和
    # request.cookies  客户端所带的cookie
    # request.headers  请求头
    # request.path     不带域名，请求路径
    # request.full_path  不带域名，带参数的请求路径
    # request.script_root
    # request.url           带域名带参数的请求路径
    # request.base_url      带域名请求路径
    # request.url_root      域名
    # request.host_url      域名
    # request.host          127.0.0.1:500
    # request.files
    # obj = request.files['the_file_name']
    # obj.save('/var/www/uploads/' + secure_filename(f.filename))
```
