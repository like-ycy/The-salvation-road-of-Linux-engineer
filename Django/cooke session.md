# Django框架之 Cookie与Session组件

## 一、Cookie

大家都知道**HTTP协议**是**无状态**的。

无状态的意思是每次请求都是独立的，它的执行情况和结果与前面的请求和之后的请求都无直接关系，它不会受前面的请求响应情况直接影响，也不会直接影响后面的请求响应情况。

一句有意思的话来描述就是人生只如初见，对服务器来说，每次的请求都是全新的。

随着交互式Web应用的兴起， 像在线购物网站，需要登录的网站等，马上面临一个问题，就是要管理会话，记住哪些人登录过系统，哪些人往自己的购物车中放商品，也就是说我必须把每个人区分开。会话中产生的数据又是我们需要保存的，也就是说要“保持状态”。因此Cookie就是在这样一个场景下诞生。

### 1.1、什么是cookie

cookie是浏览器保存在用户电脑上的一小段文本，通俗的来讲就是当一个用户通过 http访问到服务器时，服务器会将一些 Key/Value键值对返回给客户端浏览器，并给这些数据加上一些限制条件，在条件符合时这个用户下次访问这个服务器时，数据通过请求头又被完整地给带回服务器，服务器根据这些信息来判断不同的用户。

> cookie存储在客户端，也就是浏览器上，一般会限制cookie的个数为20个，并且单个cookie的大小不能超过4kb而且是明文存储不安全

### 1.2、设置cookie

先得到HttpResponse对象再进行设置

- 方法：response.set_cookie()

- 参数如下：

  | 参数         | 说明                                                         |
  | :----------- | :----------------------------------------------------------- |
  | **key**      | cookie的key                                                  |
  | **value**    | cookie的值，设置 Cookie key相对应的值，都必须是**字符串类型**。如果值为 Unicode 字符，需要为字符编码。  如果值为二进制数据，则需要使用 BASE64 编码。 |
  | **domain**   | 指定 cookie 所属域名                                         |
  | **path**     | **指定 cookie 在哪个路径（路由）下生效，默认是 '/'**。 如果设置为 `/abc`，则只有 `/abc` 下的路由可以访问到该 cookie，如：`/abc/read`。 |
  | **max_age**  | cookie 失效的时间，单位秒。如果为整数，则该 cookie 在 maxAge 秒后失效。如果为负数，该 cookie 为临时 cookie ，关闭浏览器即失效，浏览器也不会以任何形式保存该 cookie 。如果为 0，表示删除该 cookie 。默认为 -1。 - **比 expires 好用**。 |
  | **expires**  | 过期时间，在设置的某个时间点后该 cookie 就会失效。 默认为1年 |
  | **secure**   | 该 cookie 是否仅被使用安全协议传输。安全协议有 HTTPS，SSL等，在网络上传输数据之前先将数据加密。默认为false。 当 secure 值为 true 时，cookie 在 HTTP 中是无效，在 HTTPS 中才有效。 |
  | **httponly** | **如果给某个 cookie 设置了 httpOnly 属性，则无法通过 JS 脚本 读取到该 cookie 的信息，但还是能通过 Application 中手动修改 cookie，所以只是在一定程度上可以防止 XSS 攻击，不是绝对的安全** |

> **cookie 是不可跨域的：** 每个 cookie 都会绑定单一的域名，无法在别的域名下获取使用，**一级域名和二级域名之间是允许共享使用的**（**靠的是 domain）**。

+ 实例

  路由

  ```python
  path('set_cookie/', test_cookie.set_cookie, name='set_cookie'),
  path('set_cookie_lifetime/', test_cookie.set_cookie_lifetime, name='set_cookie_lifetime'),
  ```

  视图

  ```python
  # 设置cookie
  def set_cookie(req):
      res = HttpResponse('设置cookie')
      # 不设置过期时间 则为浏览会话结束时
      res.set_cookie('name', 'lucky')
      return res


  # 设置cookie并设置过期时间
  def set_cookie_lifetime(req):
      res = HttpResponse('设置cookie并设置过期时间')
      # 设置过期时间为2分钟
      res.set_cookie('name', 'lucky', max_age=120)
      return res
  ```



### 1.3、获取cookie

- 方法

  response.COOKIES

  是字典，可以get取值

+ 格式

  req.COOKIES.get('key')

+ 实例

  路由

  ```python
  path('look_cookie/', test_cookie.look_cookie, name='look_cookie'),
  ```

  视图

  ```python
  # 查看cookie
  def look_cookie(req):
      # 获取cookie  并设置默认值
      res = req.COOKIES.get('name','default')
      return HttpResponse(res)
  ```

### 1.4、删除cookie

+ 方法

  delete_cookie()

+ 格式

  response.delete_cookie()

+ 实例

  路由

  ```python
  path('delete_cookie/', test_cookie.delete_cookie, name='delete_cookie'),
  ```

  视图

  ```python
  def delete_cookie(req):
      res = HttpResponse('删除cookie')
      res.delete_cookie('name')
      return res
  ```



## 二、Session

Cookie虽然在一定程度上**解决了“保持状态”的需求**，但是由于Cookie本身最大支持4096字节，以及Cookie本身保存在客户端，可能被拦截或窃取，因此就需要有一种新的东西，它能支持更多的字节，并且他保存在服务器，有较高的安全性。这就是Session。

问题来了，基于HTTP协议的无状态特征，服务器根本就不知道访问者是“谁”。那么上述的Cookie就起到桥接的作用。

我们可以给每个客户端的Cookie分配一个唯一的id，这样用户在访问时，通过Cookie，服务器就知道来的人是“谁”。然后我们再根据不同的Cookie的id，在服务器上保存一段时间的私密资料，如“账号密码”等等。

总结而言：Cookie弥补了HTTP无状态的不足，让服务器知道来的人是“谁”；但是Cookie以文本的形式保存在本地，自身安全性较差；所以我们就通过Cookie识别不同的用户，对应的在Session里保存私密的信息以及超过4096字节的文本。

另外，上述所说的Cookie和Session其实是共通性的东西，不限于语言和框架。

### 2.1、session的存储

session的值存储于服务器端、数据库、内存、redis、文件中

将sessionid存储在cookie中，也就是当前数据的钥匙

**django**中 session默认的过期时间是**14天**

### 2.2、启用session（默认开启）

settings.py

```python
INSTALLED_APPS = [
    'django.contrib.sessions',
]
MIDDLEWARE = [
    'django.contrib.sessions.middleware.SessionMiddleware',
]
```

session开启后，req才会有session属性

### 2.3、sessionsession默认存储在数据库中

+ 生成数据库表

  python manage.py migrate

+ 数据库配置

  ```python
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': 'python36django',
          'HOST':'127.0.0.1',
          'PORT': 3306,
          'PASSWORD': '123456',
          'USER':'root',
      }
  }
  ```

+ 安装pymysql

  pip install pymysql

+ project/init.py

  ```python
  import pymysql
  pymysql.install_as_MySQLdb()
  ```

### 2.4、设置session

+ 属性

  req.session[key] = value

+ 过期时间

  req.session.set_expiry(value)

  + 如果value是个整数，session会在些秒数后失效。
  + 如果value是个datatime或timedelta，session就会在这个时间后失效。
  + 如果value是0，用户关闭浏览器session就会失效。
  + 如果value是None，session会依赖全局session失效策略。14天

+ 实例

  路由

  ```python
  path('set_session/', test_session.set_session, name='set_session'),
  path('set_session_lifetime/', test_session.set_session_lifetime, name='set_session_lifetime'),
  ```

  视图

  ```python
  def set_session(req):
      # 默认过期时间为2周
      req.session['name'] = 'lucky'
      req.session['age'] = '18'
      return HttpResponse('设置session
  def set_session_lifetime(req):
      #  过期时间为0  浏览会话结束时
      req.session.set_expiry(0)
      # 设置session存活时间为1分钟
      req.session.set_expiry(60)
      req.session['name'] = 'lucky'
      req.session['age'] = '18'
      return HttpResponse('设置session并设置过期时间')
  ```

在服务端设置session的时候，**会发生下面这几件事情**：

1. 在浏览器请求服务端的时候，django内部自动生成了随机加密字符串
2. 在django_session表中存入数据
3. 响应走的时候有中间件帮你设置随机字符串给cookie

**提示1：**在服务端添加session值之前，必须要先将django需要用到的表用数据库的迁移命令迁移

**提示2：**这个django_session表中有三个字段

```
session_key : key	这个是要与浏览器中的cookie 匹配的内容，随机字符串
session_data: value 这个是用户真正的数据，加密数据
exprie_date:session 过期时间，django默认的是14天
```

### 2.4、获取session

+ 方法

  req.session.get(key)

+ 实例

  ```python
  def get_session(req):
      val = req.session.get('name', 'default')
      return HttpResponse('获取session---'+val)
  ```

在获取浏览器的session时，**会发生下面几件事情**：

1. 浏览器发送cookie到django后端之后，django会自动获取到cookie值
2. 拿着随机字符串去django_session表中比对，是否有对应的数据
3. 如果比对上了，就将随机字符串所对应的数据，取出并将数据解密，给request.session
   如果对不上，那么request.session就是个空

### 2.5、删除session

方法：

+ pop()  通过key删除值

+ del req.session[key]    删除指定的key

+ clear()   清除当前用户的session数据

+ flush()  一般用于退出 清空session表中的数据

+ logout(req)  退出  清空session表中的数据

  ```python
  from django.contrib.auth import logout
  ```

+ clear_expired()  清除过期的session

实例

```python
def delete_session(req):
    req.session.pop('name')
    del req.session['name']
    req.session.clear()
    req.session.flush()
    logout(req)
    req.session.clear_expired()
    return HttpResponse('删除session')
```

### 2.6、session的其他参数

```python
# 获取、设置、删除Session中数据
request.session['k1']
request.session.get('k1',None)
request.session['k1'] = 123
request.session.setdefault('k1',123) # 存在则不设置
del request.session['k1']


# 所有 键、值、键值对
request.session.keys()
request.session.values()
request.session.items()
request.session.iterkeys()
request.session.itervalues()
request.session.iteritems()

# 会话session的key
request.session.session_key

# 将所有Session失效日期小于当前日期的数据删除
request.session.clear_expired()

# 检查会话session的key在数据库中是否存在
request.session.exists("session_key")
```

### 2.7、存储位置

#### 1、默认存储在数据库中

```python
SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）
```

也可以改为使用mysql

```python
SESSION_ENGINE = 'django.contrib.sessions.backends.mysql'
```

#### 2、内存Session

```python
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置
```

#### 3、文件Session

```python
SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
SESSION_FILE_PATH = '/home/xlg/桌面/session_cache'           # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir()
```

#### 4、内存+数据库

```python
SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'        # 引擎
```

#### 5、缓存在redis中

- 安装

sudo pip install redis

sudo pip install django-redis-sessions==0.5.6

- 配置

```python
# 更改存储在redis中
SESSION_ENGINE = 'redis_sessions.session'
# 主机
SESSION_REDIS_HOST = '127.0.0.1'
#  端口
SESSION_REDIS_PORT = '6379'
#  数据库
SESSION_REDIS_DB = 0
```



#### 6、加密Cookie Session

```python
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'   # 引擎
```

#### 7、其他公用设置项：

```
SESSION_COOKIE_NAME ＝ "sessionid"                       # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
SESSION_COOKIE_PATH ＝ "/"                               # Session的cookie保存的路径（默认）
SESSION_COOKIE_DOMAIN = None                             # Session的cookie保存的域名（默认）
SESSION_COOKIE_SECURE = False                            # 是否Https传输cookie（默认）
SESSION_COOKIE_HTTPONLY = True                           # 是否Session的cookie只支持http传输（默认）
SESSION_COOKIE_AGE = 60 * 60 * 24 * 7 * 2                # Session的cookie失效日期（2周）（默认）
SESSION_EXPIRE_AT_BROWSER_CLOSE = False                  # 是否关闭浏览器使得Session过期（默认）
SESSION_SAVE_EVERY_REQUEST = False                       # 是否每次请求都保存Session，默认修改之后才保存（默认）
```

