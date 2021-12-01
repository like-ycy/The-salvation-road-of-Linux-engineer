# Django Admin站点管理

## Admin

django内置了一个强大的组件叫Admin，提供给网站管理员快速开发运营后台的管理站点。

提醒：虽然django内置的运营站点功能齐全，但是在实际工作中如果要实现高定制性后台运营站点，很多公司都是自己另行自己从0开始搭建的。

站点文档： https://docs.djangoproject.com/zh-hans/3.2/ref/contrib/admin/

辅助文档：https://www.runoob.com/django/django-admin-manage-tool.html

```
注意：要使用Admin，必须先创建超级管理员.   python manage.py createsuperuser
```

访问地址：http://127.0.0.1:8000/admin，访问效果如下：

![1585638280577](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/1585638280577.png)

admin站点默认并没有提供其他的操作给我们，所以一切功能都需要我们进行配置，在项目中，我们每次创建子应用的时候都会存在一个admin.py文件，这个文件就是用于配置admin站点功能的文件。这些admin.py最终都会被项目运营的时候被django所加载并识别。

````
一般而言，我们在开发中经常以下几个术语：
1. 前台站点
   一个独立站点，这个站点和后台站点共用了一个数据库，甚至在一个目录下，大家一起运行。
   前台站点一般是允许任何人进行注册，登录并访问。我们在网络中经常访问的网站基本都是各种网站的前台站台。
   例如：京东、淘宝、

2. 后台站点
   也是一个独立站点，这个站点和前台站点共用了一个数据库，甚至在一个目录下，大家一起运行。
   后台一般是提供公司内部不懂代码的工作人员，方便这些人更好的维护或对站点已有的数据库中的内容进行增删查改用的

在绝大部分情况下，我们所说的一个项目，往往都会涵盖了前台和后台站点，除了OA，CRM，ERP类似这种内部企业项目以外。
不管是前台站点或者后台站点，都是由前端代码(html,css,js)和后端代码(python,java,c++,go,php,.net,C#)所实现的。

3. 前端开发
   指代就是客户端开发: web前端，移动端开发，游戏客户端开发
   web前端: 使用基于http获取服务端数据提供给客户端展示的这一类的开发。目前来说，绝大部分指代的就是js，....
   移动端开发：js，andorid，IOS。。
   游戏客户端：C++，js，java，。。

4. 后端开发[服务端开发]
   python,java,c++,go,php,.net,C#
````

django的admin站点在没有经过任何配置的之前，就已经实现了目前业内来说比较完善的用户认证机制和权限分配系统了。

用户认证机制：基于django.contrib.auth子应用对外提供的。里面有内置的视图，模板，模型等等。

​                          已经实现了关于管理员的登录，退出，修改密码等等。

权限认证系统：基于django.contrib.auth子应用对外提供的。里面基于模型完成了基于RBAC的权限认证机制.

## RBAC权限认证机制

实现了权限认证机制以后，我们可以让不同的用户得到不同的权限，基于用户拥有的权限不同，能操作的功能或者能看到的站点内容也会产生不一样。

RBAC= Role-Base Access Control   基于角色分配的访问控制机制。

在开发中，我们一般用于项目权限的分配机制无非3种：RBAC，Auth授权认证、RLS(Row Level Security、行级数据安全)。

在网站后台运营站点这种单个站点内部，单个站点集群场景下，一般使用的都是RBAC。

在对外开发的业务站点中，基于不同的渠道，不同的领域，不同站点之间，一般都是使用OAuth2.0授权认证。

#### RBAC的实现

在实现过程中，因为项目业务的复杂程度不一致。所以存在有3表或5表的实现方案。

django的admin站点实际上就是基于5表的实现方案扩展出来的6表RBAC认证机制。

##### 3表RBAC认证

顾名思义，就是使用了3张表保存了权限相关的所有数据，这3张表分别是用户表（user），角色表（role/group），权限表（permission/auth）

![image-20210622092906059](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20210622092906059.png)

常见的场景：单个网站：小论坛，小商城，普通的后台站点。

##### 5表RBAC认证

顾名思义，就是在3表的基础上新增了**2张关系表**保存权限相关的所有数据，这5张表分别是用户表（user），角色表（role/group），权限表（permission/auth），以及新增的2张关系表：用户与角色之间的关系表（user_group）和角色与权限的关系表（group_permission）。

![image-20211025092728749](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20211025092728749.png)

常见的业务场景：大型综合论坛，大型的商城(商家入驻)，有分公司的企业内部站点。



#### Django的RBAC

是在传统的5表RBAC基础上，增加了一个关系表，用户与权限之间的关系表（user_permission）。

因为RBAC的本质是用户随着角色不同，而拥有不同的权限，而django的admin站点中，允许针对某一个用户，可以单独分配权限的。

![image-20211025092944971](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20211025092944971.png)



admin.py里面允许我们编写的代码一共可以分成2部分：

## 列表页配置

主要用于针对项目中各个子应用里面的models.py里面的模型，根据这些模型自动生成后台运营站点的管理功能。

stuapi/admin.py，代码：

```python
from django.contrib import admin
from .models import Student
# Register your models here.


class StudentModelAdmin(admin.ModelAdmin):
    """学生的模型管理器"""
    pass

# admin.site.register(模型类, 模型管理类)
admin.site.register(Student, StudentModelAdmin)
```

子应用的英文名改成中文显示，`stuapi.apps`，代码：

```python
from django.apps import AppConfig


class StuapiConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'stuapi'
    verbose_name = "学生管理"
    verbose_name_plural = verbose_name


```

关于列表页的配置，`stuapi.admin`，代码：

```python
from django.contrib import admin
from .models import Student
from datetime import datetime
# Register your models here.


class StudentModelAdmin(admin.ModelAdmin):
    """学生的模型管理器"""
    # 设置列表页显示的模型字段
    list_display = ["id", "name", "sex", "age", "classmate", "born", "year"]
    # 默认排序
    ordering = ['-age','id']

    # 动作是否在上下方显示
    actions_on_top = True     # 上方控制栏是否显示,默认False表示隐藏
    actions_on_bottom = True  # 下方控制栏是否显示,默认False表示隐藏

    # 设置过滤器的字段条件
    list_filter = ["classmate","sex"]  # 过滤器,按指定字段的不同值来进行展示

    # 允许直接点击列表页进入编辑的字段
    list_editable = ["name", "age"]

    # 单页数据量
    list_per_page = 5

    # 搜索框搜索字段条件
    search_fields = ["name"]  # 搜索内容

    # 列表增加自定义字段列[django3.0以前, 3.0以后，官方建议设置自定义字段列的代码放在models.py中]
    def born(self, obj):
        """
        显示学生的年龄
        obj 当前模型
        """
        return datetime.now().year - obj.age
    # 自定义字段列的文本描述
    born.short_description = "出生年份"
    # 设置自定义字段列的排序参考字段
    born.admin_order_field = "age"


# admin.site.register(模型类, 模型管理类)
admin.site.register(Student, StudentModelAdmin)
```

stuapi/models.py，代码：

```python
from datetime import datetime
from django.contrib import admin
from django.db import models


# Create your models here.
class Student(models.Model):
    """学生信息"""
    name = models.CharField(max_length=255, verbose_name="姓名", help_text="姓名")
    sex = models.BooleanField(default=1, verbose_name="性别", help_text="性别")
    age = models.IntegerField(verbose_name="年龄", help_text="年龄不能小于0")
    classmate = models.CharField(max_length=5, verbose_name="班级编号")
    description = models.TextField(max_length=1000, verbose_name="个性签名")

    class Meta:
        db_table = "tb_student"
        verbose_name = "学生"
        verbose_name_plural = verbose_name


    def __str__(self):
        return self.name

    # 自定义字段列提供给admin站点使用[django3.0]
    @admin.display(ordering='age', description='出生年份(2)')
    def year(self):
        return datetime.now().year - self.age

```

school.admin，代码：

```python
from django.contrib import admin
from .models import Achievement
# Register your models here.


class AchievmentModelAdmin(admin.ModelAdmin):
    list_display = ["id", "score", "student", "course", "create_dtime"]
    # 按时间线筛选列表信息
    date_hierarchy = "create_dtime"

admin.site.register(Achievement, AchievmentModelAdmin)
```



数据库和本地Django的时区有冲突，所以需要根据实际开发过程中的业务来考虑是否要修改时区关闭USE_TZ，settings.py，代码：

```python
USE_TZ = False
```

给students.models中的Student增加一个is_delete的虚拟删除字段，并通过SQL语句同步到数据库。

```python
from django.db import models
from django.contrib import admin
from datetime import datetime


# Create your models here.
class Student(models.Model):
    SEX_OPTION = (
        (0, "保密"),
        (1, "男"),
        (2, "女"),
    )
    name = models.CharField(max_length=20, verbose_name="姓名", help_text="请输入一个正常的人名！")
    age  = models.SmallIntegerField( verbose_name="年龄", help_text="请输入一个人的年龄，范围在1-100之间~")
    sex  = models.SmallIntegerField(choices=SEX_OPTION, verbose_name="性别", help_text="性别")
    classmate = models.CharField(db_column="class", max_length=15, verbose_name="班级", help_text="班级")
    description = models.TextField(null=True, blank=True, verbose_name="个性签名", help_text="个性签名")
    is_delete = models.BooleanField(default=False, null=True, blank=True, verbose_name="逻辑删除")
    # 自定义字段提供给admin站点使用[django3.0]
    @admin.display(ordering='age', description='出生年份')
    def year(self):
        return datetime.now().year - self.age

    class Meta:
        db_table = "db_student"
        verbose_name = "学生信息"
        verbose_name_plural = verbose_name
```

SQL语句：

```sql
alter table db_student add is_delete tinyint null default 0;
```



## 详情页配置

```python
from django.contrib import admin
from .models import Student
from datetime import datetime
# Register your models here.


class StudentModelAdmin(admin.ModelAdmin):
    """学生的模型管理器"""

    """::列表页配置::"""
    # 设置列表页显示的模型字段
    list_display = ["id", "name", "sex", "age", "classmate", "born", "year"]
    # 默认排序
    ordering = ['-age','id']

    # 动作是否在上下方显示
    actions_on_top = True     # 上方控制栏是否显示,默认False表示隐藏
    actions_on_bottom = True  # 下方控制栏是否显示,默认False表示隐藏

    # 设置过滤器的字段条件
    list_filter = ["classmate","sex"]  # 过滤器,按指定字段的不同值来进行展示

    # 允许直接点击列表页进入编辑的字段
    list_editable = ["name", "age"]

    # 单页数据量
    list_per_page = 5

    # 搜索框搜索字段条件
    search_fields = ["name"]  # 搜索内容

    # 列表增加自定义字段列[django3.0以前, 3.0以后，官方建议设置自定义字段列的代码放在models.py中]
    def born(self, obj):
        """
        显示学生的年龄
        obj 当前模型
        """
        return datetime.now().year - obj.age
    # 自定义字段列的文本描述
    born.short_description = "出生年份"
    # 设置自定义字段列的排序参考字段
    born.admin_order_field = "age"

    # 列表页查询钩子【查询数据时，过滤掉被逻辑删除的数据】
    def get_queryset(self, request):
        qs = self.model._default_manager.get_queryset()
        # TODO: this should be handled by some parameter to the ChangeList.
        ordering = self.get_ordering(request)
        if ordering:
            qs = qs.order_by(*ordering)
        qs = qs.filter(is_delete=False)
        # 可以在这里增加自己的查询逻辑和查询条件
        return qs

    # 列表页删除数据钩子[把原有的模型删除代码改成更改虚拟删除状态]
    def delete_queryset(self, request, queryset):
        """Given a queryset, delete it from the database."""
        # queryset.delete()
        queryset.update(is_delete=True)



    """::详情页配置::"""
    # 设置html表单中显示的字段列表
    # fields = ['name', 'age', 'sex', 'classmate', "description"]
    # exclude 作用与fields相反,2个配置项是互斥的
    # exclude = ["is_delete"]

    # 字段分组展示,字段分组和上面的字段列表配置是互斥的。
    # 更新数据的字段列配置
    fieldsets = (
        ("必填项", {
            'fields': ('name', 'age', 'description')
        }),
        ('可选项', {
            'classes': ('collapse',),  # 折叠样式
            'fields': ('sex', 'classmate'),
        }),
    )

    # 添加数据的字段列配置
    add_fieldsets = (
        ("必填项", {
            'classes': ('wide',),
            'fields': ('name', 'age'),
        }),
        ('可选项', {
            'classes': ('collapse',),  # 折叠样式
            'fields': ('sex', 'classmate', 'description'),
        }),
    )

    # readonly_fields = ["name"]  # 设置只读字段[仅适用一些具有默认值而且不能修改的主键、外键字段使用。]

    # 识别当前操作是添加数据还是更新数据，方便加载不同的fieldsets配置
    def get_fieldsets(self, request, obj=None):
        """
        :request 本次客户端的请求处理对象
        :obj     本次客户端更新的模型对象[如果本次操作属于添加数据操作，则obj的值为None]
        """
        if not obj:
            return self.add_fieldsets
        return super().get_fieldsets(request, obj)

    # 详情页删除数据钩子[把原有的模型删除代码改成更改虚拟删除状态]
    def delete_model(self, request, obj):
        """
        Given a model instance delete it from the database.
        :request 本次客户端的请求处理对象
        :obj     本次客户端删除的模型对象
        """
        # obj.delete()
        obj.is_delete = True
        obj.save()

    def save_model(self, request, obj, form, change):
        """
        当站点保存(添加、编辑)当前模型时
        区分：依靠ID来进行区分,obj是否有id
        :request 本次客户端的请求处理对象
        :obj     本次客户端操作的模型对象[如果本次操作属于添加数据操作，则obj的值为None]
        :form    本次客户端提交的表单信息
        :change  本次客户端更新数据后的表单信息
        """
        # 验证数据或者补充代码操作
        if obj.age > 100 or obj.age < 1:
            raise TypeError("年龄必须是正常人类的年龄！")

        if obj.id:
            print("有人修改了模型信息")
        else:
            print("有人添加了模型信息")

        return super().save_model(request, obj, form, change)  # 让当前模型继续保存数据,如果上面代码已经完成操作，这一句代码可以注释掉了。

# admin.site.register(模型类, 模型管理类)
admin.site.register(Student, StudentModelAdmin)
```



django提供的admin站点已经非常强大了。但是定制性仅限于它提供的配置，要深度定制，只能自己开发。

一般可以借助antd-vue、element-plus。

然后，当然如果使用admin内置运营站点，也可以在admin基础上进行站点外观美化，或者功能的扩展。

一般站点美化模块： xadmin(dj2.0)、simpleui(3.0)，simplepro（收费）

```bash
pip install simpleui
```

settings.py，注册simpleUI，代码：

```python
INSTALLED_APPS = [
    'simpleui', # admin界面美化,必须写在admin上面
    'django.contrib.admin', # 内置的admin运营站点

    # ...
]
```

注册完成以后，直接访问原来的站点即可查看效果：

![image-20210622120344117](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20210622120344117.png)

simpleUI的官方站点：https://simpleui.72wo.com/

## Xadmin

xadmin是Django2.0时代最流行的admin站点第三方扩展，是一个比Django的admin站点使用更方便的后台站点。

构建于admin站点之上。除了Xadmin以为，市面上比较常见的还有simpleui等。

文档：http://sshwsfc.github.io/xadmin/

​           https://xadmin.readthedocs.io/en/latest/index.html

### 1.1. 安装

通过如下命令安装xadmin的2.0版本

```shell
pip install https://codeload.github.com/sshwsfc/xadmin/zip/django2
```

在配置文件settings.py中注册如下应用

```python
INSTALLED_APPS = [
    # ...
    'xadmin',
    'crispy_forms',
    'reversion',
    # ...
]

# 修改使用中文界面
LANGUAGE_CODE = 'zh-Hans'

# 修改时区
TIME_ZONE = 'Asia/Shanghai'
```

因为我们当前安装的django是3.2版本，所以xadmin在注册进来以后，会多个错误。

### 解决版本带来的异常

#### 1. six缺失

错误信息：

![image-20210622121107777](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20210622121107777.png)

```python
pip install six
# 安装完成以后，six模块就是一个six.py文件。记住这个安装的位置，一般就是python解析器对应的site-packages包中。我们需要把six.py文件复制到报six模块缺失的位置。
```

![image-20210622121236908](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20210622121236908.png)

Ctrl+C->Ctrl+V 复制文件，粘贴文件

![image-20210622121321342](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20210622121321342.png)

​	![image-20210622121451315](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20210622121451315.png)

#### 2. python_2_unicode_compatible 缺失

错误信息如下：

![image-20210622121922932](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20210622121922932.png)

解决：

```python
# 使用
from six import python_2_unicode_compatible

# 替代
from django.utils.encoding import python_2_unicode_compatible
```

#### 3. pretty_name缺失

报错信息：

![image-20210622122257967](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20210622122257967.png)

解决：

```python
# 使用
from django.forms import forms

# 替代：
from django.forms.forms import pretty_name
```

#### 4.templatetags缺失

报错信息：

![image-20210622122404767](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20210622122404767.png)

解决：

```python
使用：
from django.templatetags.static import static
替换：
from django.contrib.staticfiles.templatetags.staticfiles import static

```

#### 5. FieldDoesNotExist缺失

报错信息：

![image-20210622122600879](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20210622122600879.png)

解决：

```python
使用：
from django.core.exceptions import FieldDoesNotExist
替换：
from django.db.models.fields import FieldDoesNotExist
```

#### 6. SKIP_ADMIN_LOG缺失

报错信息：

![image-20210622122949840](13admin%E7%AB%99%E7%82%B9%E7%AE%A1%E7%90%86.assets/image-20210622122949840.png)

解决：

```python
# 去掉
from import_export.admin import DEFAULT_FORMATS, SKIP_ADMIN_LOG, TMP_STORAGE_CLASS
# 改成
from import_export.admin import DEFAULT_FORMATS, ImportMixin,ImportExportMixinBase

# 找到当前文件中的109行和115行左右位置，改动代码如下：
    def get_skip_admin_log(self):
        if self.skip_admin_log is None:
            # return SKIP_ADMIN_LOG
            return ImportMixin(ImportExportMixinBase).get_skip_admin_log()
        else:
            return self.skip_admin_log

    def get_tmp_storage_class(self):
        if self.tmp_storage_class is None:
            # return TMP_STORAGE_CLASS
            return ImportMixin(ImportExportMixinBase).get_tmp_storage_class()
        else:
            return self.tmp_storage_class
```

完成了上面操作以后，xadmin就不在报错了。接下来就可以正常使用Xadmin了。

### 初始化

xadmin有建立自己的数据库模型类，需要进行数据库迁移

```shell
python manage.py makemigrations
python manage.py migrate
```



在总路由中添加xadmin的路由信息

```python
import xadmin
xadmin.autodiscover()

# version模块自动注册需要版本控制的 Model
from xadmin.plugins import xversion
xversion.register_models()

urlpatterns = [
    path('xadmin/', xadmin.site.urls),
]
```



如果没有创建超级用户，则需要创建用户，如果之前创建了， 是可以共享一个数据库的用户的。

```python
python manage.py createsuperuser
```



### 1.2. 使用

- xadmin不再使用Django的admin.py进行功能配置了，而是需要编写代码在adminx.py文件中。所以我们需要在每一个子应用下单独创建adminx.py配置文件。
- xadmin的模型管理类不用继承`admin.ModelAdmin`，而是普通类即可。

例如：在子应用students中创建adminx.py文件。

#### 1.2.1 站点的全局配置

```python
import xadmin
from xadmin import views

class BaseSetting(object):
    """xadmin的基本配置"""
    enable_themes = True  # 开启主题切换功能
    use_bootswatch = True

xadmin.site.register(views.BaseAdminView, BaseSetting)

class GlobalSettings(object):
    """xadmin的全局配置"""
    site_title = "路飞学城"  # 设置站点标题
    site_footer = "路飞学城有限公司"  # 设置站点的页脚
    menu_style = "accordion"  # 设置菜单折叠

xadmin.site.register(views.CommAdminView, GlobalSettings)
```



#### 1.2.2 站点Model管理

xadmin可以使用的页面样式控制基本与Django原生的admin一致。

- **list_display** 控制列表展示的字段

  ```
  list_display = ['id', 'title', 'read', 'comment']
  ```

- **search_fields** 控制可以通过搜索框搜索的字段名称，xadmin使用的是模糊查询

  ```
  search_fields = ['id','title']
  ```

- **list_filter** 可以进行过滤操作的列，对于分类、性别、状态

  ```
  list_filter = ['is_delete']
  ```

- **ordering** 默认排序的字段

- **readonly_fields** 在编辑页面的只读字段

- **exclude** 在编辑页面隐藏的字段

- **list_editable** 在列表页可以快速直接编辑的字段

- **show_detail_fields** 在列表页提供快速显示详情信息

- **refresh_times** 指定列表页的定时刷新

  ```
  refresh_times = [5, 10,30,60]  # 设置允许后端管理人员按多长时间(秒)刷新页面
  ```

- **list_export** 控制列表页导出数据的可选格式

  ```
  list_export = ('xls', 'xml', 'json')   list_export设置为None来禁用数据导出功能
  list_export_fields = ('id', 'title', 'pub_date') # 允许导出的字段
  ```

- **show_bookmarks** 控制是否显示书签功能

  ```
  show_bookmarks = True
  ```

- **data_charts** 控制显示图表的样式

  ```python
  data_charts = {
          "order_amount": {
            'title': '图书发布日期表', 
            "x-field": "bpub_date", 
            "y-field": ('btitle',),
            "order": ('id',)
          },
      #    支持生成多个不同的图表
      #    "order_amount": {
      #      'title': '图书发布日期表', 
      #      "x-field": "bpub_date", 
      #      "y-field": ('btitle',),
      #      "order": ('id',)
      #    },
      }
  ```

  - title 控制图标名称
  - x-field 控制x轴字段
  - y-field 控制y轴字段，可以是多个值
  - order 控制默认排序



- **model_icon** 控制菜单的图标

  这里使用的图标是来自bootstrap3的图标。https://v3.bootcss.com/components/

  ```python
  class BookInfoAdmin(object):
      model_icon = 'fa fa-gift'
  
  xadmin.site.register(models.BookInfo, BookInfodmin)
  ```



django3.0以前的版本，修改admin或者xadmin站点下的子应用成中文内容。

```python
# 在子应用的apps.py下面的配置中，新增一个属性verbose_name
from django.apps import AppConfig

class StudentsConfig(AppConfig):
    name = 'students'
    verbose_name = "学生管理"

# 然后在当前子应用的__init__.py里面新增一下代码：
default_app_config = "students.apps.StudentsConfig"
```

