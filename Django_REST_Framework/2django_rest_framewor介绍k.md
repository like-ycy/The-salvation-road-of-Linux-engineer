# Django RESTful framework

核心思想: 大量缩减编写api接口的代码

Django REST framework是一个建立在Django基础之上的Web 应用开发框架，可以快速的开发REST API接口应用。在REST framework中，提供了序列化器Serialzier的定义，可以帮助我们简化序列化与反序列化的过程，不仅如此，还提供丰富的类视图、扩展类、视图集来简化视图的编写工作。REST framework还提供了认证、权限、限流、过滤、分页、接口文档等功能支持。REST framework还提供了一个调试API接口 的Web可视化界面来方便查看测试接口。

中文文档：https://q1mi.github.io/Django-REST-framework-documentation/#django-rest-framework

github: https://github.com/encode/django-rest-framework/tree/master

## 特点

- 提供了定义序列化器Serializer的方法，可以快速根据 Django ORM 或者其它库自动序列化/反序列化；
- 提供了丰富的类视图、Mixin扩展类，简化视图的编写；
- 丰富的定制层级：函数视图、类视图、视图集合到自动生成 API，满足各种需要；
- 多种身份认证和权限认证方式的支持；[jwt]
- 内置了限流系统；
- 直观的 API web 界面；【方便我们调试开发api接口】
- 可扩展性，插件丰富



## 要求 (Requirements)

REST framework 需要以下内容：

- Python (2.7, 3.4, 3.5, 3.6, 3.7)
- Django (1.11, 2.0, 2.1)

以下包是可选的：

- [coreapi](https://pypi.org/project/coreapi/) (1.32.0+) - 模式生成支持。
- [Markdown](https://pypi.org/project/Markdown/) (2.1.0+) - 对可浏览 API 的 Markdown 支持。
- [django-filter](https://pypi.org/project/django-filter/) (1.0.1+) - 过滤支持。
- [django-crispy-forms](https://github.com/maraujop/django-crispy-forms) - 改进的 HTML 过滤显式。
- [django-guardian](https://github.com/django-guardian/django-guardian) (1.1.1+) - 对象级权限支持。

## 安装 (Installation)

使用 `pip` 安装，包括您想要的任何可选包...

```
pip install djangorestframework
pip install markdown       # 对可浏览 API 的 Markdown 支持。
pip install django-filter  # 过滤支持
```

...或者从 github 克隆项目。

```
git clone git@github.com:encode/django-rest-framework.git
```

将 `'rest_framework'` 添加到 `INSTALLED_APPS` 设置中。

```
INSTALLED_APPS = (
    ...
    'rest_framework',
)
```

如果您打算使用可浏览的 API，您可能还需要添加 REST framework 的登录和注销视图。将以下内容添加到根 `urls.py` 文件中。

```
urlpatterns = [
    ...
    url(r'^api-auth/', include('rest_framework.urls'))
]
```

请注意，URL路径可以是您想要的任何内容。
