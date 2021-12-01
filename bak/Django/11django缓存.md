# Django 缓存

## 什么是缓存Cache

缓存是一类可以更快的读取数据的介质统称，也指其它可以加快数据读取的存储方式。一般用来存储临时数据，常用介质的是读取速度很快的内存。一般来说从数据库多次把所需要的数据提取出来，要比从内存或者硬盘等一次读出来付出的成本大很多。对于中大型网站而言，使用缓存减少对数据库的访问次数是提升网站性能的关键之一。

## 为什么要使用缓存Cache

在Django中，当用户请求到达视图后，视图会先从数据库提取数据放到模板中进行动态渲染，渲染后的结果就是用户看到的网页。如果用户每次请求都从数据库提取数据并渲染，将极大降低性能，不仅服务器压力大，而且客户端也无法即时获得响应。如果能将渲染后的结果放到速度更快的缓存中，每次有请求过来，先检查缓存中是否有对应的资源，如果有，直接从缓存中取出来返回响应，节省取数据和渲染的时间，不仅能大大提高系统性能，还能提高用户体验。

## 缓存Cache的应用场景

缓存主要适用于对页面实时性要求不高的页面。存放在缓存的数据，通常是频繁访问的，而不会经常修改的数据。我们来举几个应用例子:

- 博客文章。假设用户一天更新一篇文章，那么可以为博客设置1天的缓存，一天后会刷新。
- 购物网站。商品的描述信息几乎不会变化，而商品的购买数量需要根据用户情况实时更新。我们可以只选择缓存商品描述信息。
- 缓存网页片段。比如缓存网页导航菜单和脚部(Footer)。

## Django缓存在redis中

因为大多数缓存使用redis，这里只讲述redis，其他缓存方式请自行百度

### 1、安装模块

pip install django-redis

### 2、配置 settings.py

```python
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/0",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            "CONNECTION_POOL_KWARGS": {"max_connections": 100}
            # "PASSWORD": "密码",
        }
    }
}
```

### 3、单个视图缓存

cache_page 装饰器

```python
from django.views.decorators.cache import cache_page
```

+ time  (秒)  过期时间
+ cache  缓存配置，选择使用哪一个（默认default）
+ key_prefix 缓存前缀

```python
from django.shortcuts import render, HttpResponse
from django.views.decorators.cache import cache_page

@cache_page(10)
def index(req):
    print('你能看到我吗？')
    return render(req, 'index.html')
```

### 4、手动缓存（底层缓存API）

+ 导入

  ```python
  from django.core.cache import cache
  ```

+ 缓存操作

  + 设置缓存

    cache.set(key, value, expires)

  + 获取缓存

    cache.get(key)

  + 删除缓存

    cache.delete()

  + 清空缓存

    cache.clear()

+ 使用

  ```python
  from django.core.cache import cache
  from django.shortcuts import render,HttpResponse
  from django.views.decorators.cache import cache_page
  from django.template import loader
  
  # 手动缓存
  def index(req):
      res = cache.get('tem')
      if not res:
          print('你能看到我吗？')
          tem = loader.get_template('index.html')
          res = tem.render()
          cache.set('tem', res, 20)
      return HttpResponse(res)
  ```

作业：

+ 登录注册使用缓存完成
+ 中间件中添加缓存
