# 会话

所谓的会话（session），就是客户端浏览器和服务端网站之间一次完整的交互过程.

会话的开始是在用户通过**浏览器第一次访问服务端网站**开始.

会话的结束时在用户通过**关闭浏览器**以后，与服务端断开.

所谓的会话控制，就是在客户端浏览器和服务端网站之间，进行**多次http请求响应之间，记录、跟踪和识别用户的信息**而已。



为什么要有会话控制？因为 http 是一种无状态协议，浏览器请求服务器是无状态的。

**无状态**：指一次用户请求时，浏览器、服务器无法知道之前这个用户做过什么，对于服务端而言，客户端的每次请求都是一次新的请求。

**无状态原因**：浏览器与服务器是使用 socket 套接字进行通信的，服务器将请求结果返回给浏览器之后，会关闭当前的 socket 连接，而且客户端也会在处理页面完毕之后销毁页面对象。

有时需要保持下来用户浏览的状态，比如用户是否登录过，浏览过哪些商品等

实现状态保持主要有两种方式：

- 在客户端存储信息使用`Cookie(废弃)，token[jwt,oauth]`
- 在服务器端存储信息使用`Session`，数据库

## Cookie

Cookie是由服务器端生成，发送给客户端浏览器，浏览器会将Cookie的key/value保存，下次请求同一网站时就随着请求头自动发送该Cookie给服务器（前提是浏览器设置为启用cookie）。Cookie的key/value可以由服务器端自己定义。

使用场景: 登录状态, 浏览历史, 网站足迹,购物车 [不登录也可以使用购物车]

Cookie是存储在浏览器中的一段**纯文本信息**，建议不要存储敏感信息如密码，因为电脑上的浏览器可能被其它人使用

Cookie基于域名安全，不同域名的Cookie是不能互相访问的

如访问fuguang.com时向浏览器中写了Cookie信息，使用同一浏览器访问baidu.com时，无法访问到fuguang.com写的Cookie信息，只能获取到baidu.com的Cookie信息。

浏览器的同源策略针对cookie也有限制作用.

当浏览器请求某网站时，浏览器会自动将本网站下所有Cookie信息随着http请求头提交给服务器，所以在request中可以读取Cookie信息

### 设置cookie

设置cookie需要通过flask的Response响应对象来进行设置,由响应对象会提供了方法set_cookie给我们可以快速设置cookie信息。

```python
@app.route("/set_cookie")
def set_cookie():
    """设置cookie，通过response传递到客户端进行保存"""
    response = make_response('默认首页')
    response.set_cookie('username', 'xiaoming')            # session会话期有效，关闭浏览器后当前cookie就会被删除
    response.set_cookie('user', 'xiaoming', max_age=30 )   # 指定有效时间，过期以后浏览器删除cookie，max_age=150秒
    return response
```



### 获取cookie

```python
@app.route("/get_cookie")
def get_cookie():
    """获取来自客户端的cookie"""
    print(request.cookies)  # ImmutableMultiDict([])
    username = request.cookies.get('username')  # 没有值则返回None
    user = request.cookies.get('user')          # 没有值则返回None
    print(f"username={username},user={user}")   # username=xiaoming,user=xiaoming
    return "get cookie"
```



### 删除cookie

```python
@app.route("/del_cookie")
def del_cookie():
    # 方法一
    resp = Response('Delete cookie')
    resp.delete_cookie('answer')
    return resp
    
    # 方法二
    """删除cookie，重新设置cookie的时间，让浏览器自己根据有效期来删除"""
    response = make_response('del cookie')
    # 删除操作肯定是在浏览器完成的，所以我们重置下cookie名称的对饮有效时间为0，此时cookie的值已经不重要了。
    response.set_cookie('user', '', max_age=0)
    response.set_cookie('username', '', max_age=0)
    return response
```



## Session

对于敏感、重要的信息，建议要存储在服务器端，不能存储在浏览器中，如手机号、验证码等信息

在服务器端进行状态保持的方案就是`Session`

**Session依赖于Cookie**,session的ID一般默认通过cookie来保存到客户端。名字一般叫：sessionid

flask中的session需要加密,所以使用session之前必须配置SECRET_KEY选项,否则报错.

```
如果将来希望session的生命周期延长，可以通过修改cookie中的sessionID的有效期来完成配置。
```

注意：一般框架都是把session数据保存到服务端，但是，flask里面的session是基于token方式存储在客户端的，并没有安装传统的方式保存在服务端的文件中。

session的ID存在有效期的，默认是**会话期**，会话结束了，session_id就废弃了。

### 设置session

```python
@app.route("/set_session")
def set_session():
    """设置session"""
    session['username'] = 'xiaoming'
    session['info'] = {
        "name": "xiaohong",
        "age": 16,
    }
    return "set_session"
```

可以通过客户端浏览器中开发者工具中-应用-Cookie 观察，其实默认情况下，flask中的session数据会被加密保存到cookie中的。当然，将来，我们可以采用flask-session第三方模块把数据转存到其他的存储设备，例如：redis或者mysql中。

### 获取session

```python
@app.route("/get_session")
def get_session():
    """获取session"""
    print(session.get('username'))
    print(session.get('info'))
    return "get session"
```



### 删除session

```python
@app.route("/del_session")
def del_session():
    """删除session，键如果不存在，则会抛出异常，所以删除之前需要判断键是否存在。"""
    if "username" in session:
        session.pop("username")
    if "info" in session:
        session.pop("info")
    return "del_session"
```

使用过程中，session是依赖于Cookie的，所以当cookie在客户端被删除时，对应的session就无法被使用了。

