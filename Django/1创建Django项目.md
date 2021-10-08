# 创建 Django 项目

## 创建第一个项目

使用 django-admin 来创建 HelloWorld 项目：

```
django-admin startproject HelloWorld
```

创建完成后我们可以查看下项目的目录结构：

```
mysite/
├── manage.py
└── mysite/
    ├── asgi.py
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

目录说明：

- **外层的 mysite 目录:** 项目的容器。
  - **manage.py:** 一个实用的命令行工具，可让你以各种方式与该 Django 项目进行交互。
  - **里层的 mysite 目录**: 包含你的项目，它是一个纯 Python 包。它的名字就是当你引用它内部任何东西时需要用到的 Python 包名。 (比如 `mysite.urls`)
    - **\_\_init\_\_.py:** 一个空文件，告诉 Python 该目录是一个 Python 包。
    - **asgi.py:** 一个 ASGI 兼容的 Web 服务器的入口，django3.0 新增文件，用于让 django 运行在异步的编程模式的对象。
    - **settings.py:** 该 Django 项目的设置/配置。
    - **urls.py:** 该 Django 项目的 URL 声明; 一份由 Django 驱动的网站"目录"。
    - **wsgi.py:** 一个 WSGI 兼容的 Web 服务器的入口，以便运行你的项目。

### 启动项目

```
python3 manage.py runserver 0.0.0.0:8000
```

0.0.0.0 让其它电脑可连接到开发服务器，8000 为端口号。如果不说明，那么端口号默认为 8000。

在浏览器输入你服务器的 ip（这里我们输入本机 IP 地址： **127.0.0.1:8000**） 及端口号，如果正常启动，输出结果如下：

![img](1%E5%88%9B%E5%BB%BADjango%E9%A1%B9%E7%9B%AE.assets/225A52EA-25EF-4BF1-AA5A-B91490CBF26D.jpg)

## 创建应用

一个项目底下可以有多个应用

进入项目内，创建应用

```
django-admin startapp App
```

创建完成后我们可以查看下应用的目录结构：

```
APP/
├── admin.py
├── apps.py
├── __init__.py
├── migrations/
│   └── __init__.py
├── models.py
├── tests.py
└── views.py

```

目录说明：

- \_\_init\_\_.py 文件初始化
- migrations 生成迁移文件的目录
- admin.py 管理站点文件
- apps.py 该应用的一些配置,Django1.9 以后自动生成
- models.py 模型文件 以后在这里创建模型类
- test.py 自动化测试模块,Django 提供了自动化测试功能,在这里编写测试脚本(语句)
- views.py 执行响应的代码所在模块,代码逻辑处理的主要地点,项目中大部分代码均在这里编写

### 启动应用

修改**HelloWorld 项目**下的 settings.py 文件，在 INSTALLED_APPS 中添加应用名称。

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'App',
]
```
