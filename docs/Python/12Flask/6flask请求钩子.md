# Flask请求钩子

在客户端和服务器交互的过程中，有些准备工作或扫尾工作需要处理，比如：

- 在项目运行开始时，建立数据库连接，或创建连接池；
- 在客户端请求开始时，根据需求进行身份识别，权限校验；
- 在请求结束视图返回数据时，指定转换数据的格式，或者记录操作日志；

为了让每个视图函数避免编写重复功能的代码，Flask提供了通用设置的功能，即请求钩子。

请求钩子是**通过装饰器的形式**实现，Flask支持如下四种请求钩子（注意：钩子的装饰器名字是固定)：

- before_first_request
  - 在处理第一个请求前执行[项目刚运行第一次被客户端请求时执行的钩子]
- before_request
  - 在每一次请求前执行[项目运行后，每一次接收到客户端的request请求都会执行一次]
  - 如果在某修饰的函数中返回了一个响应，视图函数将不再被调用
- after_request
  - 如果没有抛出错误，在每次请求后执行
  - 接受一个参数：视图函数作出的响应
  - 在此函数中可以对响应值在返回之前做最后一步修改处理
  - 需要将参数中的响应在此参数中进行返回
- teardown_request：
  - 在每一次请求后执行
  - 接受一个参数：错误信息，如果有相关错误抛出
  - 需要设置flask的配置DEBUG=False，teardown_request才会接受到异常对象。



代码

```python
from flask import Flask, session

# 应用实例对象
app = Flask(__name__)

"""给app单独设置配置项"""
# 设置秘钥
app.config["SECRET_KEY"] = "my SECRET KEY"

@app.before_first_request
def before_first_request():
    """
    这个钩子会在项目启动后第一次被用户访问时执行
    可以编写一些初始化项目的代码，例如，数据库初始化，加载一些可以延后引入的全局配置
    """
    print("----before_first_request----")
    print("系统初始化的时候,执行这个钩子方法")
    print("会在接收到第一个客户端请求时,执行这里的代码")


@app.before_request
def before_request():
    """
    这个钩子会在每次客户端访问视图的时候执行
    # 可以在请求之前进行用户的身份识别，以及对于本次访问的用户权限等进行判断。..
    """
    print("----before_request----")
    print("每一次接收到客户端请求时,执行这个钩子方法")
    print("一般可以用来判断权限,或者转换路由参数或者预处理客户端请求的数据")


@app.after_request
def after_request(response):
    print("----after_request----")
    print("在处理请求以后,执行这个钩子方法")
    print("一般可以用于记录会员/管理员的操作历史,浏览历史,清理收尾的工作")

    response.headers["Content-Type"] = "application/json"
    response.headers["Company"] = "python oldboy..."

    # 必须返回response参数
    return response


@app.teardown_request
def teardown_request(exc):
    print("----teardown_request----")
    print("在每一次请求以后,执行这个钩子方法")
    print("如果有异常错误,则会传递错误异常对象到当前方法的参数中")
    # 在项目关闭了DEBUG模式以后，则异常信息就会被传递到exc中，我们可以记录异常信息到日志文件中
    print(f"错误提示：{exc}")  # 异常提示


@app.route("/")
def index():
    print("-----------视图函数执行了---------------")
    return "ok"

if __name__ == '__main__':
    # 启动项目的web应用程序
    app.run(host="0.0.0.0", port=5000, debug=False)
```

- 在第1次请求时的打印（关闭DEBUG模式，视图代码正确执行，没有异常的情况）：

```bash
----before_first_request----
系统初始化的时候,执行这个钩子方法
会在接收到第一个客户端请求时,执行这里的代码
----before_request----
每一次接收到客户端请求时,执行这个钩子方法
一般可以用来判断权限,或者转换路由参数或者预处理客户端请求的数据
-----------视图函数执行了---------------
----after_request----
在处理请求以后,执行这个钩子方法
一般可以用于记录会员/管理员的操作历史,浏览历史,清理收尾的工作
----teardown_request----
在每一次请求以后,执行这个钩子方法
如果有异常错误,则会传递错误异常对象到当前方法的参数中
错误提示：None
```

- 在第2次请求时的打印（关闭DEBUG模式，视图代码正确执行，没有异常的情况）：

```bash
----before_request----
每一次接收到客户端请求时,执行这个钩子方法
一般可以用来判断权限,或者转换路由参数或者预处理客户端请求的数据
-----------视图函数执行了---------------
----after_request----
在处理请求以后,执行这个钩子方法
一般可以用于记录会员/管理员的操作历史,浏览历史,清理收尾的工作
----teardown_request----
在每一次请求以后,执行这个钩子方法
如果有异常错误,则会传递错误异常对象到当前方法的参数中
错误提示：None
```

在第1次请求时的打印（关闭DEBUG模式，视图执行错误，有异常的情况）：

```bash
----before_first_request----
系统初始化的时候,执行这个钩子方法
会在接收到第一个客户端请求时,执行这里的代码
----before_request----
每一次接收到客户端请求时,执行这个钩子方法
一般可以用来判断权限,或者转换路由参数或者预处理客户端请求的数据
-----------视图函数执行了---------------
----after_request----
在处理请求以后,执行这个钩子方法
一般可以用于记录会员/管理员的操作历史,浏览历史,清理收尾的工作
----teardown_request----
在每一次请求以后,执行这个钩子方法
如果有异常错误,则会传递错误异常对象到当前方法的参数中
错误提示：division by zero
```

在第1次请求时的打印（关闭DEBUG模式，视图执行有异常的情况）：

```bash
----before_request----
每一次接收到客户端请求时,执行这个钩子方法
一般可以用来判断权限,或者转换路由参数或者预处理客户端请求的数据
-----------视图函数执行了---------------
----after_request----
在处理请求以后,执行这个钩子方法
一般可以用于记录会员/管理员的操作历史,浏览历史,清理收尾的工作
----teardown_request----
在每一次请求以后,执行这个钩子方法
如果有异常错误,则会传递错误异常对象到当前方法的参数中
错误提示：division by zero
```

钩子装饰器装饰了多个函数的执行顺序如下：

```python
from flask import Flask, session

# 应用实例对象
app = Flask(__name__)

"""给app单独设置配置项"""
# 设置秘钥
app.config["SECRET_KEY"] = "my SECRET KEY"

@app.before_first_request
def before_first_request():
    """
    这个钩子会在项目启动后第一次被用户访问时执行
    可以编写一些初始化项目的代码，例如，数据库初始化，加载一些可以延后引入的全局配置
    """
    print("----before_first_request----")
    print("系统初始化的时候,执行这个钩子方法")
    print("会在接收到第一个客户端请求时,执行这里的代码")


@app.before_request
def before_request():
    """
    这个钩子会在每次客户端访问视图的时候执行
    # 可以在请求之前进行用户的身份识别，以及对于本次访问的用户权限等进行判断。..
    """
    print("----before_request----")
    print("每一次接收到客户端请求时,执行这个钩子方法")
    print("一般可以用来判断权限,或者转换路由参数或者预处理客户端请求的数据")


@app.after_request
def after_request1(response):
    print("----after_request1----")
    print("在处理请求以后,执行这个钩子方法")
    print("一般可以用于记录会员/管理员的操作历史,浏览历史,清理收尾的工作")

    response.headers["Content-Type"] = "application/json"
    response.headers["Company"] = "python oldboy..."

    # 必须返回response参数
    return response


@app.after_request
def after_request2(response):
    print("----after_request2----")

    # 必须返回response参数
    return response

@app.after_request
def after_request3(response):
    print("----after_request3----")

    # 必须返回response参数
    return response

@app.teardown_request
def teardown_request(exc):
    print("----teardown_request----")
    print("在每一次请求以后,执行这个钩子方法")
    print("如果有异常错误,则会传递错误异常对象到当前方法的参数中")
    # 在项目关闭了DEBUG模式以后，则异常信息就会被传递到exc中，我们可以记录异常信息到日志文件中
    print(f"错误提示：{exc}")  # 异常提示


@app.route("/")
def index():
    print("-----------视图函数执行了---------------")
    return "ok"

if __name__ == '__main__':
    # 启动项目的web应用程序
    app.run(host="0.0.0.0", port=5000, debug=False)

```

执行效果：

```python
----before_first_request----
系统初始化的时候,执行这个钩子方法
会在接收到第一个客户端请求时,执行这里的代码
----before_request----
每一次接收到客户端请求时,执行这个钩子方法
一般可以用来判断权限,或者转换路由参数或者预处理客户端请求的数据
-----------视图函数执行了---------------
----after_request3----
----after_request2----
----after_request1----
在处理请求以后,执行这个钩子方法
一般可以用于记录会员/管理员的操作历史,浏览历史,清理收尾的工作
----teardown_request----
在每一次请求以后,执行这个钩子方法
如果有异常错误,则会传递错误异常对象到当前方法的参数中
错误提示：None
```

结论：以视图执行为中心点，before_request之前（请求过程）的先装饰先执行，after_request之后（响应过程），后装饰的先执行。



