# 路由层

Django中的路由指的就是urls.py文件，称之为路由层

路由层即**用户请求地址与视图函数的映射关系**，如果被一个网站比喻成一本字典的话，我们这个路由（urls.py）就好比是这个字典的目录，在Django中路由默认在urls.py文件中



## 路由配置

### 1、路由方法

+ path

  参数

  + route  书写匹配url
  + view   当django匹配到了url，就会执行特定的视图函数
  + name  为url进行取名，目的是在任何位置，都能反向解析出当前路由地址

  实例

  ```python
  path('index/', views.index, name='index'),
  ```

+ re_path

  参数

  + route  书写正则表达式的路由地址
  + view   当django匹配到了url，就会执行特定的视图函数
  + name  为url进行取名，目的是在任何位置，都能反向解析出当前路由地址

  ```python
  re_path(r'^index2/$', views.index, name='index2'),
  ```

  完全匹配  如果不添加`^$` ，路由中包含index2就可以访问了

  例如：http://127.0.0.1/aaindex 也可以访问到index网页，因为aaindex包含了index

+ 注意

  一个视图函数可以存在多个路由地址

  ```python
  from django.urls import path
  from django.urls import re_path
  
  from . import views
  
  urlpatterns = [
      # 首页的路由配置
      path(r'', views.index, name='index1'),
      path(r'index/', views.index, name='index2'),
      re_path(r'^index2/$', views.index, name='index3'),
  
  ]
  ```

### 2、项目和子应用的路由关联（子路由、路由分发）

在项目目录（mysite项目）下的urls.py里，使用include进行关联

+ App/urls.py

  ```python
  from django.urls import path
  from . import views
  
  urlpatterns = [
      # 首页的路由配置
      path('index/', views.index),
  ]
  ```

+ mysite/urls.py

  ```python
  from django.contrib import admin
  from django.urls import path, include
  
  urlpatterns = [
      path('admin/', admin.site.urls),
      path('', include('App.urls')),
  ]
  ```

+ include

  + 导入

    from django.urls import path, include

  + 作用

    找到子路由模块

  + 参数

    + 子路由模块路径
    + 命名空间namespace（用于反向解析，动态构造路由地址）

  + 实例

    ```python
    path('', include(('App.urls','App'), namespace='App')),
    ```

  + 多个自路由模块

    ```python
    from django.contrib import admin
    from django.urls import path, include
    
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('App/', include(('App.urls','App'), namespace='App')),
        path('App2/', include(('App2.urls','App2'), namespace='App2')),
    ]
    ```

    注意：如果说自路由前面添加了前缀，访问的时候，就要添加上

    http://127.0.0.1:8000/app/index/

