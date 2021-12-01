# Redis缓存列表页

因为导航在项目中每一个页面都会被用户访问到，所以我们可以实现redis缓存，减少MySQL数据库的查询压力。

utils/constants.py，代码：

```python
# 通用列表的缓存时间，单位：秒
LIST_PAGE_CACHE_TIME = 60 * 60 * 24
```

utils/views.py，代码：

```python
import constants
from rest_framework.generics import ListAPIView as DRFListAPIView
# cache_page是django提供给开发者使用的一个缓存类，专门缓存数据，以装饰器的方式提供操作
from django.views.decorators.cache import cache_page
# cache_page默认只能装饰函数视图，但是我们现在使用的普遍都是类视图，
# 所以又提供了一个method_decorators给开发者在不改动源码的情况下，就可以使用装饰器装饰一个类视图/类方法了
from django.utils.decorators import method_decorator


class ListAPIView(DRFListAPIView):
    """列表缓存视图类"""
    @method_decorator(cache_page(constants.LIST_PAGE_CACHE_TIME))
    def get(self, request, *args, **kwargs):
        # 重写ListAPIView的get方法，但是不改动源代码。仅仅装饰而已
        return super().get(request, *args, **kwargs)
```

home/views.py，代码：

```python
import constants
from .models import Nav, Banner
from .serializers import NavModelSerializer, BannerModelSerializer
from views import ListAPIView


class HeaderNavListAPIView(ListAPIView):
    """
    头部导航
    """
    queryset = Nav.objects.filter(is_delete=False, is_show=True, position=constants.NAV_HEADER).order_by("orders", "-id")[:constants.NAV_HEADER_SIZE]
    serializer_class = NavModelSerializer


class FooterNavListAPIView(ListAPIView):
    """
    脚部导航
    """
    queryset = Nav.objects.filter(is_delete=False, is_show=True, position=constants.NAV_FOOTER).order_by("orders", "-id")[:constants.NAV_FOOTER_SIZE]
    serializer_class = NavModelSerializer


class BannerListAPIView(ListAPIView):
    """
    轮播广告列表
    """
    queryset = Banner.objects.filter(is_delete=False, is_show=True).order_by("orders", "-id")[:constants.BANNER_SIZE]
    serializer_class = BannerModelSerializer

```

提交git版本

```bash
cd ~/Desktop/fuguang
git add .
git commit -m "客户端展示轮播广告数据，服务端实现导航和轮播的数据缓存"
git push origin master
```

