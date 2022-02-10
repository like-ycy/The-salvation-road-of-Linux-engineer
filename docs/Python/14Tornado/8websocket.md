# Tornado的WebSocket

WebSocket是HTML5规范中新提出的客户端-服务器通信协议，协议本身使用新的ws://URL格式。

WebSocket 是独立的、创建在 TCP 上的协议，和 HTTP 的唯一关联是使用 HTTP 协议的101状态码进行协议升级，使用的 TCP 端口是80，可以用于绕过大多数防火墙的限制。

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端直接向客户端主动推送数据而不需要客户端进行再次请求，两者之间可以创建持久性连接，并允许数据进行双向通信。

Tornado提供支持WebSocket的模块是tornado.websocket，其中提供了一个WebSocketHandler类用来处理通讯。

## 常用方法

#### open()

当一个WebSocket连接建立后被调用。

#### on_message(message)

当客户端发送消息message过来时被调用，**注意此方法必须被重写**。

#### on_close()

当WebSocket连接关闭后被调用。

#### write_message(message, binary=False)

向客户端发送消息messagea，message可以是字符串或字典（字典会被转为json字符串）。若binary为False，则message以utf8编码发送；二进制模式（binary=True）时，可发送任何字节码。

#### close()

关闭WebSocket连接。

#### check_origin(origin)

判断源origin，对于符合条件（返回判断结果为True）的请求源origin允许其连接，否则返回403。可以重写此方法来解决WebSocket的跨域请求（如始终return True）。

## 快速使用

server.py，代码：

```python
from tornado import web,ioloop
from tornado.websocket import WebSocketHandler

class Index(web.RequestHandler):
    def get(self):
        self.render("templates/index.html")

class Home(WebSocketHandler):
    def open(self):
        # 
        self.write_message("欢迎来到socket.html")

    def on_message(self, message):
        print("接收数据：%s" % message)

    def on_close(self):
        print("socket连接断开")

    def check_origin(self, origin):
        return True  # 允许WebSocket的跨域请求

# 设置路由列表
urls = [
    (r"/", Index),
    (r"/home", Home),
]

if __name__ == "__main__":
    # 创建应用实例对象
    app = web.Application(urls, debug=True)
    # 设置监听的端口和地址
    app.listen(port=8888)
    # ioloop，全局的tornado事件循环，是服务器的引擎核心，start表示创建IO事件循环
    ioloop.IOLoop.current().start()
```

tempales/index.html，代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<div id="content"></div>
<script>
    var ws = new WebSocket("ws://127.0.0.1:8888/home"); // 新建一个ws连接
    ws.onopen = function() {  // 连接建立好后的回调
       ws.send("Hello, world");  // 向建立的连接发送消息
    };
    ws.onmessage = function (evt) {  // 收到服务器发送的消息后执行的回调
       content.innerHTML+=evt.data+"<br>";  // 接收的消息内容在事件参数evt的data属性中
    };
</script>
</body>
</html>
```

## 案例：聊天室

server.py，代码：

```python
from tornado import web,ioloop,httpserver,options
import datetime

from tornado.web import RequestHandler
from tornado.websocket import WebSocketHandler

class Index(RequestHandler):
    def get(self):
        self.render("templates/chat.html")

class Chat(WebSocketHandler):
    users = set()  # 用来存放用户的容器，必须类静态属性

    def open(self):
        self.users.add(self)  # 建立连接后保存客户端的socket连接对象到users容器中
        key = list(self.users).index(self)
        for user in self.users:  # 向已在线用户发送消息
            user.write_message("[%s]-%02d-[%s]-登录" % (self.request.remote_ip, key, datetime.datetime.now().strftime("%H:%M:%S")))

    def on_message(self, message):
        key = list(self.users).index(self)
        for user in self.users:  # 向在线用户广播消息
            user.write_message("[%s]-%02d-[%s]-发送：%s" % (self.request.remote_ip, key, datetime.datetime.now().strftime("%H:%M:%S"), message))

    def on_close(self):
        key = list(self.users).index(self)
        self.users.remove(self) # 用户关闭连接后从容器中移除用户
        for user in self.users:
            user.write_message("[%s]-%02d-[%s]-退出" % (self.request.remote_ip, key, datetime.datetime.now().strftime("%H:%M:%S")))

    def check_origin(self, origin):
        return True  # 允许WebSocket的跨域请求

urls = [
    (r"/", Index),
    (r"/chat", Chat),
]


if __name__ == '__main__':
    # 创建应用实例对象
    app = web.Application(urls)
    server = httpserver.HTTPServer(app)
    # 设置监听的端口和地址
    server.listen(port=8888,address="0.0.0.0")
    server.start(1)
    # ioloop，全局的tornado事件循环，是服务器的引擎核心，start表示创建IO事件循环
    ioloop.IOLoop.current().start()
```



templates/index.html，代码：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>聊天室</title>
</head>
<body>
    <div>
        <textarea id="msg"></textarea>
        <button onclick="sendMsg()">发送</button>
    </div>
    <div id="content" style="height:500px;overflow:auto;"></div>
    <script>
        var ws = new WebSocket("ws://127.0.0.1:8888/chat");
        ws.onmessage = function(message) {
            console.log("接收数据:", message);
            content.innerHTML +="<p>" + message.data + "</p>";
        };

        function sendMsg() {
            console.log("发送数据:", msg.value);
            ws.send(msg.value);
            msg.value = "";
        }
    </script>
</body>
</html>
```

