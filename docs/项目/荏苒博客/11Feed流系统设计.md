# Feed流系统

## Feed流的定义

Feed流是Feed + 流，Feed的本意是饲料，Feed流的本意就是有人一直在往一个地方投递新鲜的饲料，如果需要饲料，只需要盯着投递点就可以了，这样就能源源不断获取到新鲜的饲料。 在信息工程里面，Feed其实是一个信息单元，比如一条朋友圈状态、一条微博、一条资讯或一条短视频等，所以Feed流就是不停更新的信息单元，只要关注某些发布者就能获取到源源不断的新鲜信息，我们的用户也就可以在移动设备上逐条去浏览这些信息单元。

当前最流行的Feed流产品有微博、微信朋友圈、头条的资讯推荐、快手抖音的视频推荐等，还有一些变种，比如私信、通知等，这些系统都是Feed流系统，接下来我们会介绍如何设计一个Feed流系统架构。

## Feed流系统特点

Feed流本质上是数据流，是服务端系统将 “多个发布者的信息内容” 通过 “关注收藏等关系” 推送给 “多个接收者”。

![1579146889360](11Feed%E6%B5%81%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1.assets/1579146889360-16414723556371.png)

-   多账号内容流：Feed流系统中肯定会存在成千上万的账号，账号之间可以关注，取关，加好友和拉黑等操作。只要满足这一条，那么就可以当做Feed流系统来设计。
-   非稳定的账号关系：由于存在关注，取关等操作，所以系统中的用户之间的关系就会一直在变化，是一种非稳定的状态。
-   读写比例100:1：读写严重不平衡，读多写少。
-   消息必达性要求高：比如A发送了一条朋友圈后，结果部分朋友看到了，部分朋友没看到，如果偏偏女朋友没看到，那么可能会产生很严重的感情矛盾，后果很严重。



## Feed流系统分类

Feed流的分类有很多种，但最常见的分类有两种：

-   Timeline：按发布的时间顺序排序，先发布的先看到，后发布的排列在最顶端，类似于微信朋友圈，微博等。这也是一种最常见的形式。产品如果选择Timeline类型，那么就是认为`Feed流中的Feed不多，但是每个Feed都很重要，都需要用户看到`。
-   Rank：按某个非时间的因子排序，一般是按照用户的喜好度排序，用户最喜欢的排在最前面，次喜欢的排在后面。这种一般假定用户可能看到的Feed非常多，而用户花费在这里的时间有限，那么就为用户选择出用户最想看的Top N结果，场景的应用场景有图片分享、新闻推荐类、商品推荐等。

上面两种是最典型，也是最常见的分类方式，另外的话，也有其他的分类标准，在其他的分类标准中的话，会多出两种类型：

-   Aggregate：聚合类型，比如好几个朋友都看了同一场电影，这个就可以聚合为一条Feed：A，B，C看了电影《你的名字》，这种聚合功能比较适合在客户端做。一般的Aggregate类型是Timeline类型 + 客户端聚合。
-   Notice：通知类型，这种其实已经是功能类型了，通知类型一般用于APP中的各种通知，私信等常见。这种也是Timeline类型，或者是Aggregate类型。



## 设计Feed流系统的2个核心

Feed流系统是一个数据流系统，如果要设计一个Feed流系统，最关键的两个核心，一个是数据存储(发布Feed)，一个是数据推送(读取Feed)。

这两个核心我们稍后再谈，我们先从数据层面看，数据分为三类，分别是：

- 发布者的数据：发布者发布数据，然后数据需要按照关注者进行组织，需要根据关注者查到所有数据，

  ​                          比如微博的个人页面、朋友圈的个人相册等。

- 关注关系：系统中个体间的关系，微博中是关注，是单向流，朋友圈是好友，是双向流。

  ​                   不管是单向还是双向，当发布者发布一条信息时，该条信息的流动永远是单向的。

- 粉丝的接收数据：从不同发布者那里获取到的数据，然后通过某种顺序（一般为时间timeline）组织到一起，

  ​                       比如微博首页、朋友圈首页等。

  ​                       这些数据具有时间热度属性，越新的数据越有价值，越新的数据就要排在最前面。

### Feed数据

针对这三类数据的保存和处理，我们可以定义为：

-   同步库：存储发布者的Feed数据，永久保存。我们已经存放到mysql中，所以同步库中可以单独保存消息的ID
-   关系库：用户关系表，永久保存。
-   未读池：存储接收者的时间热度数据，只需要保留最近一段时间的数据即可。



### 数据存储

Feed消息的特点：

-   Feed信息的最大特点就是数据量大，而且在Feed流系统里面很多时候都会选择写扩散（推模式）模式，这时候数据量会再膨胀几个数量级，所以这里的数据量很容易达到100TB，甚至PB级别。

-   数据格式简单
-   数据不能丢失，可靠性要求高
-   自增主键功能，保证个人发的Feed的消息ID在个人发件箱中都是严格递增的，这样读取时只需要一个范围读取即可。由于个人发布的Feed并发度很低，这里用时间戳也能满足基本需求，但是当应用层队列堵塞，网络延迟变大或时间回退时，用时间戳还是无法保证严格递增。这里最好是有自增功能。



根据上述这些Feed数据的特征，最佳的系统应该是`具有主键自增功能的分布式NoSQL数据库`，但是在之前的开源系统里面没有，所以以前常用的做法有两种：

-   关系型数据库 + 分库分表
-   关系型数据库 + 分布式NoSQL数据库：其中 关系型数据库提供主键自增功能。

目前业界大部分著名的Feed流产品，早期都是上面的2种模式之一，但是这会存在一个非常大的问题就是关系型数据库，比如开源MySQL数据库的主键自增功能性能差。不管是用MyISAM，还是InnoDB引擎，要保证自增ID严格递增，必须使用表锁，这个粒度非常大，会严重限制并发度，影响性能。

基于上述原因，部分技术公司早已经开始考虑使用表格存储(TableStore)。

表格存储是一个具有自增主键功能的分布式NoSQL数据库，这样就只需要使用一种系统即可，除此之外表格存储还有以下的特点：

-   天然分布式数据库，`无需分库分表`，单表可达10PB，10万亿行，可支持千万级TPS/QPS
-   号称SLA可用性可达到`10个9`，Feed内容不容易丢失。
-   主键自增功能性能极佳，其他所有系统在做自增功能的时候都需要加锁，但是表格存储的主键自增功能在写入自增列行的时候，完全不需要锁，既不需要表锁，也不需要行锁。



### 数据推送

数据推送的实现，有3种方案，分别是：

-   拉方案：也称为`读扩散`。很多Feed流产品的第一版会采用这种方案，但很快就抛弃了。
-   推方案：也成为`写扩散`。Feed流系统中最常用、有效的模式，用户关系数比较均匀，或者有上限，比较出名的有微信朋友圈。
-   推拉组合：大部分用户的账号关系都是几百个，但是有个别用户是1000万以上，比如微博。

| 类型         | 推模式      | 拉模式                               | 推拉结合模式 |
| :----------- | :---------- | :----------------------------------- | :----------- |
| 写放大       | 高          | 无                                   | 中           |
| 读放大       | 无          | 高                                   | 中           |
| 用户读取延时 | 毫秒        | 秒                                   | 秒           |
| 读写比例     | 1:99        | 99:1                                 | 50:50        |
| 系统要求     | 写能力强    | 读能力强                             | 读写都适中   |
| 常见系统     | 分布式NoSQL | 内存缓存或搜索系统<br>(推荐排序场景) | 两者结合     |
| 架构复杂度   | 简单        | 复杂                                 | 更复杂       |

-   如果产品中是双向关系，那么就采用推模式。
-   如果产品中是单向关系，且用户数少于1000万，那么也采用推模式，足够了。
-   如果产品是单向关系，单用户数大于1000万，那么采用推拉结合模式，这时候可以从推模式演进过来，不需要额外重新推翻重做。
-   永远不要只用拉模式。
-   如果是一个初创企业，先用推模式，快速把系统设计出来，然后让产品去验证、迭代，等客户数大幅上涨到1000万后，再考虑升级为推拉集合模式。

所以，接下来我们选择的先是写扩散(推模式)，然后推拉组合。



关系库设计结构：

Table：user_relation_table

| 主键顺序    | 第1列主键 | 第2列主键      | 属性列     |
| :---------- | :-------- | :------------- | :--------- |
| Table字段名 | user_id   | follow_user_id | other      |
| 备注        | 作者ID    | 粉丝ID         | 其他属性列 |



同步库表设计结构:

Table：user_message_table

| 主键列 | 第1列主键      | 第2列主键              | 第3列主键      | 第4列主键                                                    | 属性列                                     |
| :----- | :------------- | :--------------------- | :------------- | :----------------------------------------------------------- | :----------------------------------------- |
| 列名   | follow_user_id | sequence_id            | user_id        | message_id                                                   | other                                      |
| 解释   | 接收者ID       | 消息顺序ID，要求自增。 | 发送者的用户ID | 消息ID。通过sender_id和message_id可以到存储库中查询到消息内容 | 其他字段内容，同步库中不需要包括消息内容。 |



未读池表设计结构：

Table: user_message_session_table

| 主键列顺序 | 第一列主键     | 属性列                                 |
| :--------- | :------------- | :------------------------------------- |
| 列名       | follow_user_id | last_sequence_id                       |
| 备注       | 接收者用户ID   | 该接收者已经推送给客户端的最新的顺序ID |



# Feed流系统的实现前置准备

## TableStore基本入门

1. [开通阿里云表格存储服务](https://www.aliyun.com/product/ots?spm=a2c4g.11186623.2.7.6f0b23a5RBru3P)

   ![1579170876748](11Feed%E6%B5%81%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1.assets/1579170876748-16414723556382.png)

2. 获取python操作TableStore的[SDK](https://github.com/aliyun/aliyun-tablestore-python-sdk)

   <https://help.aliyun.com/document_detail/31723.html?spm=a2c4g.11186623.6.891.563c3d76sdVMpI>


```bash
pip install tablestore
```

settings/dev.py，添加TableStore的API接口配置

```python
# tablestore
# 应用ID
OTS_ID = "LTAI4G9ja6BxsF6yLCKJPzv9"
# 应用密钥
OTS_SECRET = "HJzpNpQ8dgdamS7j3844eNvvBJpKNT"
# 实例名称
OTS_INSTANCE = "renranpy29"
# 访问实例的公网地址
OTS_ENDPOINT = "https://renranpy29.cn-hangzhou.ots.aliyuncs.com"
```

应用ID和应用密钥通过控制台accesskeys来获取。

![1595812102385](11Feed%E6%B5%81%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1.assets/1595812102385.png)

为了方便演示,所以我们另外创建一个单独的子应用store来编写tablestore入门的学习代码.

```
cd renranapi/apps
python ../../manage.py startapp store
```

在settings/dev.py注册,

```python
INSALL_APPS = [
    'store', # 用于演示tableStore，后续删除掉即可
]
```

Tablestore目前只支持四种数据类型：INTEGER、STRING、DOUBLE和BOOLEAN。其中DOUBLE类型不能做主键类型，BOOLEAN不可以做主键的第一列(分区键)。

### 表操作

>   tablestore里面创建表的时候必须设置表名，主键列，还有表元信息和表的描述项[有效期，版本，吞吐量]
>
>   创建表的时候，除了主键列以外，还可以设置预设字段列，这个不常用，因为当前使用tablestore是NoSQL数据，所以我们表结构的字段列可以在添加数据的时候再指定。

视图代码:

```python
"""表格存储的基本入门代码"""
"""初始化tablestore sdk对象"""
from tablestore import
# from tablestore import OTSClient,TableMeta,TableOptions,ReservedThroughput,CapacityUnit,PK_AUTO_INCR
from tablestore import *
from rest_framework.views import APIView
from rest_framework.response import Response
from django.conf import settings

class OTS(object):
    def __init__(self):
        self.client = OTSClient(settings.OTS_ENDPOINT,settings.OTS_ID,settings.OTS_SECRET, settings.OTS_INSTANCE)

    def list_table(self):
        """列出所有的存储表"""
        return self.client.list_table()

    def create_table(self,table_name, schema_of_primary_key):
        """
        :param table_name: 字符串，表名
        :param schema_of_primary_key: 列表，每一个成员是一个元祖，表示字段的名称和类型
        :return:
        """
        # 通过表名和主键列的schema创建一个tableMeta。
        table_meta = TableMeta(table_name, schema_of_primary_key)
        # 创建TableOptions，数据保留31536000秒，超过后自动删除；最大3个版本；写入时指定的版本值和当前标准时间相差不能超过1天。
        table_options = TableOptions(31536000, 3, 86400)
        # 设置预留读吞吐量为0，预留写吞吐量为0。
        reserved_throughput = ReservedThroughput(CapacityUnit(0, 0))

        try:
            self.client.create_table(table_meta, table_options, reserved_throughput)
            return True
        # 处理异常。
        except Exception:
            return False

    def delete_table(self,table_name):
        """
        删除表
        :param table_name: 参数如果是字符串，则表示删除一张表，
                           参数如果是列表，则表示删除多张表
        :return:
        """
        tables = []
        if type(table_name) is str:
            tables.append(table_name)
        else:
            tables = table_name

        ret = {
            "success": [],
            "fails": []
        }
        for table in tables:
            try:
                self.client.delete_table(table)
                ret["success"].append(table)
            except Exception:
                ret["fails"].append(table)

        return ret

class TableAPIView(APIView):

    def get(self,request):
        """获取当前仓库中所有的数据表"""
        tables = OTS().list_table()
        print(tables)
        return Response("ok")

    def post(self,request):

        OTS().create_table("user_relation_table",[
            ("user_id","INTEGER"),
            ("follow_user_id","INTEGER")
        ])

        OTS().create_table("user_message_table", [
            ("follow_user_id", "INTEGER"),
            ("sequence_id", "INTEGER",PK_AUTO_INCR), # 设为自增字段
            ("user_id", "INTEGER"),
            ("message_id", "INTEGER"),
        ])

        OTS().create_table("user_message_session_table", [
            ("follow_user_id", "INTEGER"),
            ("last_sequence_id", "INTEGER"),
        ])
        return Response("ok")

    def delete(self,request):
        """删除指定的表"""
        # 删除一张表
        ret = OTS().delete_table("user_message_session_table")
        print(ret)
        # 删除多张表
        ret = OTS().delete_table(["user_message_table","user_relation_table"])
        print(ret)
        return Response("ok")
```

 ```
注意：
1. 创建表后需要等待1分钟进行加载，在此期间对该表的读/写数据操作均会失败。
   应用程序应该等待表加载完毕后再进行数据操作。
2. 创建表格存储的表时必须指定表的主键。
   一张数据镖一般可以包含 1~4 个主键列，每一个主键列都有名字和类型。
   主键的类型可以有INTEGER,STRING和BINARY.其中第一主键默认是分区键，主要用于将来存储的数据量大了系统自动进行分区保存使用的，第二个主键以后，就可以设置自增主键，注意，一般一张表有0或1个主键。
 ```

路由:

```python
from django.urls import path,re_path
from . import views
urlpatterns = [
    path("table/", views.TableAPIView.as_view()),
]

# 总路由
    path('store/', include("store.urls")),
```



### 一条数据的操作

```python
"""表格存储的基本入门代码"""
"""初始化tablestore sdk对象"""
# from tablestore import OTSClient,TableMeta,TableOptions,ReservedThroughput,CapacityUnit,PK_AUTO_INCR
from tablestore import *
from rest_framework.views import APIView
from rest_framework.response import Response
from django.conf import settings

class OTS(object):
    def __init__(self):
        self.client = OTSClient(settings.OTS_ENDPOINT,settings.OTS_ID,settings.OTS_SECRET, settings.OTS_INSTANCE)

    def list_table(self):
        """列出所有的存储表"""
        return self.client.list_table()

    def create_table(self,table_name, schema_of_primary_key):
        """
        :param table_name: 字符串，表名
        :param schema_of_primary_key: 列表，每一个成员是一个元祖，表示字段的名称和类型
        :return:
        """
        # 通过表名和主键列的schema创建一个tableMeta。
        table_meta = TableMeta(table_name, schema_of_primary_key)
        # 创建TableOptions，数据保留31536000秒，超过后自动删除；最大3个版本；写入时指定的版本值和当前标准时间相差不能超过1天。
        table_options = TableOptions(31536000, 3, 86400)
        # 设置预留读吞吐量为0，预留写吞吐量为0。
        reserved_throughput = ReservedThroughput(CapacityUnit(0, 0))

        try:
            self.client.create_table(table_meta, table_options, reserved_throughput)
            return True
        # 处理异常。
        except Exception:
            return False

    def delete_table(self,table_name):
        """
        删除表
        :param table_name: 参数如果是字符串，则表示删除一张表，
                           参数如果是列表，则表示删除多张表
        :return:
        """
        tables = []
        if type(table_name) is str:
            tables.append(table_name)
        else:
            tables = table_name

        ret = {
            "success": [],
            "fails": []
        }
        for table in tables:
            try:
                self.client.delete_table(table)
                ret["success"].append(table)
            except Exception:
                ret["fails"].append(table)

        return ret

    def put_row(self,table_name,primary_key,attribute_columns={}):
        """
        添加一条数据
        :param table_name: 本次添加数据的表名
        :param primary_key: 主键列
        :param attribute_columns: 其他属性列
        :return: 新增数据的主键
        """

        primary_key_list = []
        for key,value in primary_key.items():
            primary_key_list.append((key,value))

        attribute_columns_list = []
        for key,value in attribute_columns.items():
            attribute_columns_list.append((key,value))

        row = Row(primary_key_list, attribute_columns_list)
        # 添加数据的条件
        # EXPECT_NOT_EXIST 如果主键重复，则报错！
        condition = Condition(RowExistenceExpectation.EXPECT_NOT_EXIST)
        try:
            # 调用put_row方法, ReturnType.RT_PK表示返回新增数据的主键信息，如果不设置则返回None
            consumed, return_row = self.client.put_row(table_name, row, condition, ReturnType.RT_PK)
            return True, return_row.primary_key
        except OTSClientError as e:
            """网络异常"""
            return False, "put row failed, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())

        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            """参数有误"""
            return False, "put row failed, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
        e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            """其他错误"""
            return False, "未知的异常"

    def get_row(self,table_name, primary_key,columns_to_get=[],column_filter=None):
        """"""
        # 指定查询结果返回的属性列
        # 过滤条件
        """
        # ====================== 多条件 ========================
        # 逻辑条件
        cond = CompositeColumnCondition(LogicalOperator.AND)
        cond = CompositeColumnCondition(LogicalOperator.OR)
        cond = CompositeColumnCondition(LogicalOperator.NOT)
        # 比较条件
        ComparatorType.NOT_EQUAL  !=
        ComparatorType.EQUAL      ==
        GREATER_THAN              >
        GREATER_EQUAL             >=
        LESS_THAN                 <
        LESS_EQUAL                <=
        举例：
          查询一个学生信息，性别为男的，小于20岁===>   sex=男 and age < 20
            cond = CompositeColumnCondition(LogicalOperator.AND)
            cond.add_sub_condition(SingleColumnCondition("sex", '男', ComparatorType.EQUAL))
            cond.add_sub_condition(SingleColumnCondition("age", 20, ComparatorType.LESS_THAN))
      
      
        # ====================== 单条件 ======================== 
        cond = SingleColumnCondition("age", 20, ComparatorType.LESS_THAN)
        """
        primary_key_list = []
        for key,value in primary_key.items():
            primary_key_list.append((key,value))
        try:
            # 调用get_row接口查询
            consumed, return_row, next_token = self.client.get_row(table_name, primary_key_list, columns_to_get, column_filter, 1)
            data = {}
            for att in return_row.primary_key:
                # 打印每一个字段的内容
                data[att[0]] = att[1]
            if len(columns_to_get)>0:
                """如果有指定要返回其他属性列"""
                for att in return_row.attribute_columns:
                    # 打印每一个字段的内容
                    data[att[0]] = att[1]
            return True, data
            # 客户端异常，一般为参数错误或者网络异常。
        except OTSClientError as e:
            return False, "获取数据失败, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            return False, "获取数据失败, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
        e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False,"未知异常"

    def update_row(self,table_name,primary_key,attribute_columns):
        """更新一条数据"""
        primary_key_list = []
        for key, value in primary_key.items():
            primary_key_list.append((key, value))

        attribute_columns_list = []
        for key, value in attribute_columns.items():
            attribute_columns_list.append((key, value))

        update_of_attribute_columns = {
            'PUT': attribute_columns_list,
        }

        row = Row(primary_key_list, update_of_attribute_columns)

        try:
            consumed, return_row = self.client.update_row(table_name, row, condition=None, return_type=ReturnType.RT_PK)
            data = {}
            for att in return_row.primary_key:
                # 打印每一个字段的内容
                data[att[0]] = att[1]

            return True, data
        # 客户端异常，一般为参数错误或者网络异常。
        except OTSClientError as e:
            return False, "更新失败, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())

        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            return False, "更新失败, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
        e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False,"未知异常"

    def delete_row(self,table_name,primary_key):
        """根据主键删除一条数据"""
        primary_key_list = []
        for key, value in primary_key.items():
            primary_key_list.append((key, value))
        row = Row(primary_key_list)
        try:
            consumed, return_row = self.client.delete_row(table_name, row,None)
            return True,"删除成功"
        except OTSClientError as e:
            return False, "更新失败！网络异常, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        except OTSServiceError as e:
            return False, "更新失败，参数有误, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
        e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False,"未知异常"

class TableAPIView(APIView):

    def get(self,request):
        """获取当前仓库中所有的数据表"""
        tables = OTS().list_table()
        print(tables)
        return Response("ok")

    def post(self,request):

        OTS().create_table("user_relation_table",[
            ("user_id","INTEGER"),
            ("follow_user_id","INTEGER")
        ])

        OTS().create_table("user_message_table", [
            ("follow_user_id", "INTEGER"),
            ("sequence_id", "INTEGER", PK_AUTO_INCR),
            ("user_id", "INTEGER"),
            ("message_id", "INTEGER"),
        ])

        OTS().create_table("user_message_session_table", [
            ("follow_user_id", "INTEGER"),
            ("last_sequence_id", "INTEGER"),
        ])
        return Response("ok")

    def delete(self,request):
        """删除指定的表"""
        # 删除一张表
        ret = OTS().delete_table("user_message_session_table")
        print(ret)
        # 删除多张表
        ret = OTS().delete_table(["user_message_table","user_relation_table"])
        print(ret)
        return Response("ok")

from django.utils import timezone as datetime
class DataAPIView(APIView):
    def post(self,request):
        """添加一条数据"""
        ## 主键 = {"主键名":"值","主键名":"值",,,}
        primary_key = {"user_id":1,'follow_user_id': 3}

        # 属性列 = {"字段名":"值"}
        attribute_columns = {'timestamp':datetime.now().timestamp()}

        ret = OTS().put_row("user_relation_table",primary_key,attribute_columns)

        print(ret[0])

        return Response("OK")

    def get(self,request):
        """获取一条数据"""
        primary_key = {"user_id":1,'follow_user_id': 2}
        ret = OTS().get_row("user_relation_table",primary_key, columns_to_get=["timestamp"])
        return Response(ret)

    def put(self,request):
        primary_key = {"user_id": 1, 'follow_user_id': 2}
        attribute_columns = {"timestamp": 1000866}
        ret = OTS().update_row("user_relation_table",primary_key,attribute_columns)
        return Response(ret)

    def delete(self,request):
        primary_key = {"user_id": 1, 'follow_user_id': 2}
        ret = OTS().delete_row("user_relation_table", primary_key)
        return Response(ret)
```

路由,代码:

```python
from django.urls import path
from . import views
urlpatterns = [
    path("table/", views.TableAPIView.as_view()),
    path("data/", views.DataAPIView.as_view()),
]
```



### 多条数据的操作

```python
"""表格存储的基本入门代码"""
"""初始化tablestore sdk对象"""
# from tablestore import OTSClient,TableMeta,TableOptions,ReservedThroughput,CapacityUnit,PK_AUTO_INCR
from tablestore import *
from rest_framework.views import APIView
from rest_framework.response import Response
from django.conf import settings

class OTS(object):
    def __init__(self):
        self.client = OTSClient(settings.OTS_ENDPOINT,settings.OTS_ID,settings.OTS_SECRET, settings.OTS_INSTANCE)

    def list_table(self):
        """列出所有的存储表"""
        return self.client.list_table()

    def create_table(self,table_name, schema_of_primary_key):
        """
        :param table_name: 字符串，表名
        :param schema_of_primary_key: 列表，每一个成员是一个元祖，表示字段的名称和类型
        :return:
        """
        # 通过表名和主键列的schema创建一个tableMeta。
        table_meta = TableMeta(table_name, schema_of_primary_key)
        # 创建TableOptions，数据保留31536000秒，超过后自动删除；最大3个版本；写入时指定的版本值和当前标准时间相差不能超过1天。
        table_options = TableOptions(31536000, 3, 86400)
        # 设置预留读吞吐量为0，预留写吞吐量为0。
        reserved_throughput = ReservedThroughput(CapacityUnit(0, 0))

        try:
            self.client.create_table(table_meta, table_options, reserved_throughput)
            return True
        # 处理异常。
        except Exception:
            return False

    def delete_table(self,table_name):
        """
        删除表
        :param table_name: 参数如果是字符串，则表示删除一张表，
                           参数如果是列表，则表示删除多张表
        :return:
        """
        tables = []
        if type(table_name) is str:
            tables.append(table_name)
        else:
            tables = table_name

        ret = {
            "success": [],
            "fails": []
        }
        for table in tables:
            try:
                self.client.delete_table(table)
                ret["success"].append(table)
            except Exception:
                ret["fails"].append(table)

        return ret

    def put_row(self,table_name,primary_key,attribute_columns={}):
        """
        添加一条数据
        :param table_name: 本次添加数据的表名
        :param primary_key: 主键列
        :param attribute_columns: 其他属性列
        :return: 新增数据的主键
        """

        primary_key_list = []
        for key,value in primary_key.items():
            primary_key_list.append((key,value))

        attribute_columns_list = []
        for key,value in attribute_columns.items():
            attribute_columns_list.append((key,value))

        row = Row(primary_key_list, attribute_columns_list)
        # 添加数据的条件
        # EXPECT_NOT_EXIST 如果主键重复，则报错！
        condition = Condition(RowExistenceExpectation.EXPECT_NOT_EXIST)
        try:
            # 调用put_row方法, ReturnType.RT_PK表示返回新增数据的主键信息，如果不设置则返回None
            consumed, return_row = self.client.put_row(table_name, row, condition, ReturnType.RT_PK)
            return True, return_row.primary_key
        except OTSClientError as e:
            """网络异常"""
            return False, "put row failed, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())

        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            """参数有误"""
            return False, "put row failed, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
        e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            """其他错误"""
            return False, "未知的异常"

    def get_row(self,table_name, primary_key,columns_to_get=[],column_filter=None):
        """"""
        # 指定查询结果返回的属性列
        # 过滤条件
        """
        # ====================== 多条件 ========================
        # 逻辑条件
        cond = CompositeColumnCondition(LogicalOperator.AND)
        cond = CompositeColumnCondition(LogicalOperator.OR)
        cond = CompositeColumnCondition(LogicalOperator.NOT)
        # 比较条件
        ComparatorType.NOT_EQUAL  !=
        ComparatorType.EQUAL      ==
        GREATER_THAN              >
        GREATER_EQUAL             >=
        LESS_THAN                 <
        LESS_EQUAL                <=
        举例：
          查询一个学生信息，性别为男的，小于20岁===>   (sex=男 and age < 20) or (sex=女 and age < 17) 
            cond = CompositeColumnCondition(LogicalOperator.AND)
            cond.add_sub_condition(SingleColumnCondition("sex", '男', ComparatorType.EQUAL))
            cond.add_sub_condition(SingleColumnCondition("age", 20, ComparatorType.LESS_THAN))
      
        # ====================== 单条件 ======================== 
        cond = SingleColumnCondition("age", 20, ComparatorType.LESS_THAN)
        """
        primary_key_list = []
        for key,value in primary_key.items():
            primary_key_list.append((key,value))
        try:
            # 调用get_row接口查询
            consumed, return_row, next_token = self.client.get_row(table_name, primary_key_list, columns_to_get, column_filter, 1)
            data = {}
            for att in return_row.primary_key:
                # 打印每一个字段的内容
                data[att[0]] = att[1]
            if len(columns_to_get)>0:
                """如果有指定要返回其他属性列"""
                for att in return_row.attribute_columns:
                    # 打印每一个字段的内容
                    data[att[0]] = att[1]
            return True, data
            # 客户端异常，一般为参数错误或者网络异常。
        except OTSClientError as e:
            return False, "获取数据失败, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            return False, "获取数据失败, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
        e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False,"未知异常"

    def update_row(self,table_name,primary_key,attribute_columns):
        """更新一条数据"""
        primary_key_list = []
        for key, value in primary_key.items():
            primary_key_list.append((key, value))

        attribute_columns_list = []
        for key, value in attribute_columns.items():
            attribute_columns_list.append((key, value))

        update_of_attribute_columns = {
            'PUT': attribute_columns_list,
        }

        row = Row(primary_key_list, update_of_attribute_columns)

        try:
            consumed, return_row = self.client.update_row(table_name, row, condition=None, return_type=ReturnType.RT_PK)
            data = {}
            for att in return_row.primary_key:
                # 打印每一个字段的内容
                data[att[0]] = att[1]

            return True, data
        # 客户端异常，一般为参数错误或者网络异常。
        except OTSClientError as e:
            return False, "更新失败, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())

        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            return False, "更新失败, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
        e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False,"未知异常"

    def delete_row(self,table_name,primary_key):
        """根据主键删除一条数据"""
        primary_key_list = []
        for key, value in primary_key.items():
            primary_key_list.append((key, value))
        row = Row(primary_key_list)
        try:
            consumed, return_row = self.client.delete_row(table_name, row,None)
            return True,"删除成功"
        except OTSClientError as e:
            return False, "更新失败！网络异常, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        except OTSServiceError as e:
            return False, "更新失败，参数有误, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
        e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False,"未知异常"

    def get_list_0(self,table_name,primary_key_list,columns_to_get=[],column_filter=None):
        """
        根据指定主键查询多条数据
        :param table: 字符串，表名
        :param primary_key_list: 主键列表
        :param columns_to_get: 返回属性字段列表
        :param column_filter: 条件
        :return: 列表， 查询结果
        """

        # 读取3行。
        rows_to_get = []
        for item in primary_key_list:
            primary_key = []
            for key,value in item.items():
                primary_key.append((key,value))
            rows_to_get.append(primary_key)

        # 构造批量读请求。
        request = BatchGetRowRequest()

        # 增加表table_name中需要读取的行，最后一个参数1表示读取最新的一个版本。
        request.add(TableInBatchGetRowItem(table_name, rows_to_get, columns_to_get, column_filter, 1))

        try:
            result = self.client.batch_get_row(request)
            if result.is_all_succeed():
                """只有在获取到全部数据以后才表示读取多条数据成功"""
                table_result = result.get_result_by_table(table_name)
                data = []
                for item in table_result:
                    row = {}

                    if item.is_ok:
                        for att in item.row.primary_key:
                            # 打印每一个字段的内容
                            row[att[0]] = att[1]

                        for att in item.row.attribute_columns:
                            # 打印每一个字段的内容
                            row[att[0]] = att[1]

                    else:
                        return False, '部分数据参数有误，读取数据失败。 error code: %s, error message: %s' % (item.error_code, item.error_message)

                    data.append(row)

                return True,data

            else:
                return False,'部分数据参数有误，读取数据失败。'

        # 客户端异常，一般为参数错误或者网络异常。
        except OTSClientError as e:
            return False, "读取数据失败，网络异常。 http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            return False, "读取数据失败，参数有误。 http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False,"未知异常"

    def add_list(self,table_name,primary_key_list,attribute_columns_list=[]):
        """
        添加多条数据
        :param table_name: 字符串，表名
        :param primary_key_list: 主键列
        :param attribute_columns_list: 属性列
        :return:
        """
        put_row_items = self.row_items(PutRowItem, primary_key_list, attribute_columns_list)
        status,result = self.request(table_name,put_row_items)
        if status == False:
            return False,result
        # 输出每一行添加数据的结果。
        succ, fail = result.get_put()
        for item in succ:
            print('添加数据成功, consume %s write cu.' % item.consumed.write)
        for item in fail:
            print('添加数据失败, error code: %s, error message: %s' % (item.error_code, item.error_message))

        return True

    def update_list(self,table_name,primary_key_list,attribute_columns_list=[]):
        """更新多条数据"""
        update_row_items = self.row_items(UpdateRowItem,primary_key_list,attribute_columns_list)
        status,result = self.request(table_name, update_row_items)
        if status == False:
            return False,result

        # 输出每一行更新数据的结果
        succ, fail = result.get_update()
        for item in succ:
            print('更新数据成功, consume %s write cu.' % item.consumed.write)
        for item in fail:
            print('更新数据失败, error code: %s, error message: %s' % (item.error_code, item.error_message))

        return True

    def delete_list(self,table_name,primary_key_list):
        """删除多条数据"""
        delete_row_items = self.row_items(DeleteRowItem, primary_key_list)
        status, result = self.request(table_name, delete_row_items)
        if status == False:
            return False,result
        # 输出每一行删除数据的结果。
        succ, fail = result.get_delete()
        for item in succ:
            print('删除数据成功, consume %s write cu.' % item.consumed.write)
        for item in fail:
            print('删除数据失败, error code: %s, error message: %s' % (item.error_code, item.error_message))

        return True

    def row_items(self,cls,primary_key_list,attribute_columns_list=[]):

        if len(primary_key_list) != len(attribute_columns_list) and len(attribute_columns_list) != 0:
            return False, "参数有误！主键和属性列数量不对应，无法完成添加操作"

        # 增加多行组件的组装。
        row_items = []
        for key0, primary_item in enumerate(primary_key_list):
            primary_row = []
            for key1, value in primary_item.items():
                primary_row.append((key1, value))

            attribute_row = None
            if len(attribute_columns_list) > 0:
                if cls == PutRowItem:
                    """添加多条"""
                    attribute_row = []
                    for key2, value in attribute_columns_list[key0].items():
                        attribute_row.append((key2, value))
                elif cls == UpdateRowItem:
                    """更新多条"""
                    attribute_row = {"put":[]}
                    for key2, value in attribute_columns_list[key0].items():
                        attribute_row["put"].append((key2, value))

            row = Row(primary_row, attribute_row)
            condition = Condition(RowExistenceExpectation.IGNORE)
            item = cls(row, condition)
            row_items.append(item)

        return row_items

    def request(self,table_name,put_row_items):
        """
        构造批量写请求
        :param table_name:
        :param put_row_items:
        :return:
        """
        request = BatchWriteRowRequest()
        request.add(TableInBatchWriteRowItem(table_name, put_row_items))
        try:
            result = self.client.batch_write_row(request)
            if result.is_all_succeed():
                return True,result

        except OTSClientError as e:
            print(e.get_http_status(), e.get_error_message())
            return False, "网络异常, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        except OTSServiceError as e:
            return False, "参数有误, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
            e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False, "未知异常"

    def get_list(self,table_name,start_key,end_key,columns_to_get = [],limit=90, cond=None,derection=Direction.FORWARD):
        """
        根据指定范围查询数据
        :param table_name: 字符串，表名
        :param start_key: 字典，查询的开始主键位置
        :param end_key: 字典，查询的结束主键位置
        :param columns_to_get: 列表，属性列
        :param limit: 整型, 查询结果数量
        :param cond: Cond类对象，查询条件
        :param derection: 查询的排列方式，Direction.FORWARD 正序，Direction.BACKWARD 倒序
        :return:
        """
        # 设置范围查询的起始主键。
        inclusive_start_primary_key=[]
        for key,value in start_key.items():
            inclusive_start_primary_key.append((key,value))
        # 设置范围查询的结束主键。
        exclusive_end_primary_key=[]
        for key,value in end_key.items():
            exclusive_end_primary_key.append((key,value))

        try:
            # 读取数据
            consumed, next_start_primary_key, row_list, next_token = self.client.get_range(
                table_name, derection,
                inclusive_start_primary_key, exclusive_end_primary_key,
                columns_to_get,
                limit,
                column_filter=cond,
                max_version=1,
            )

            all_rows = []
            all_rows.extend(row_list)

            # 打印主键和属性列。
            data = []
            for row in all_rows:
                data_item = {}
                for att in row.primary_key:
                    data_item[att[0]] = att[1]
                if len(columns_to_get) > 0:
                    """如果有指定要返回其他属性列"""
                    for att in row.attribute_columns:
                        data_item[att[0]] = att[1]
                data.append(data_item)

            return {"status":True, "data": data, "token": next_start_primary_key}
        # 客户端异常，一般为参数错误或者网络异常。
        except OTSClientError as e:
            return False, "网络异常, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            return False, "参数有误, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
            e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False, "未知异常"

class TableAPIView(APIView):

    def get(self,request):
        """获取当前仓库中所有的数据表"""
        tables = OTS().list_table()
        print(tables)
        return Response("ok")

    def post(self,request):

        OTS().create_table("user_relation_table",[
            ("user_id","INTEGER"),
            ("follow_user_id","INTEGER")
        ])

        OTS().create_table("user_message_table", [
            ("follow_user_id", "INTEGER"),
            ("sequence_id", "INTEGER", PK_AUTO_INCR),
            ("user_id", "INTEGER"),
            ("message_id", "INTEGER"),
        ])

        OTS().create_table("user_message_session_table", [
            ("follow_user_id", "INTEGER"),
            ("last_sequence_id", "INTEGER"),
        ])
        return Response("ok")

    def delete(self,request):
        """删除指定的表"""
        # 删除一张表
        ret = OTS().delete_table("user_message_session_table")
        print(ret)
        # 删除多张表
        ret = OTS().delete_table(["user_message_table","user_relation_table"])
        print(ret)
        return Response("ok")
from django.utils import timezone as datetime
class DataAPIView(APIView):
    def post(self,request):
        """添加一条数据"""
        ## 主键 = {"主键名":"值","主键名":"值",,,}
        primary_key = {"user_id":1,'follow_user_id': 3}

        # 属性列 = {"字段名":"值"}
        attribute_columns = {'timestamp':datetime.now().timestamp()}

        ret = OTS().put_row("user_relation_table",primary_key,attribute_columns)
        print(ret[0])

        return Response("OK")
    def get(self,request):
        """获取一条数据"""
        primary_key = {"user_id":1,'follow_user_id': 2}
        ret = OTS().get_row("user_relation_table",primary_key, columns_to_get=["timestamp"])
        return Response(ret)
    def put(self,request):
        primary_key = {"user_id": 1, 'follow_user_id': 2}
        attribute_columns = {"timestamp": 1000866}
        ret = OTS().update_row("user_relation_table",primary_key,attribute_columns)
        return Response(ret)
    def delete(self,request):
        primary_key = {"user_id": 1, 'follow_user_id': 2}
        ret = OTS().delete_row("user_relation_table", primary_key)
        return Response(ret)
class RowAPIView(APIView):
    """多行数据操作"""
    def get0(self,request):
        """根据主键查询多条数据[少用]"""
        primary_key_list = [
            {"user_id": 1, 'follow_user_id': 3},
            {"user_id": 1, 'follow_user_id': 4},
            {"user_id": 1, 'follow_user_id': 5},
        ]

        data = OTS().get_list_0("user_relation_table",primary_key_list,["timestamp"])

        return Response(data)

    def post(self,request):
        """添加多条数据"""
        primary_key = [
            {"user_id":1,'follow_user_id': 5},
            {"user_id":1,'follow_user_id': 6},
            {"user_id":1,'follow_user_id': 7},
            {"user_id":1,'follow_user_id': 8},
        ]
        # 属性列 = {"字段名":"值"}
        attribute_columns = [
            {'timestamp': datetime.now().timestamp()},
            {'timestamp': datetime.now().timestamp()},
            {'timestamp': datetime.now().timestamp()},
            {'timestamp': datetime.now().timestamp()},
        ]

        ret = OTS().add_list("user_relation_table",primary_key,attribute_columns)
        return Response(ret)

    def put(self,request):
        """注意，所谓更新数据，仅仅是更新属性列而已，tablestore中是不提供修改主键的！！！"""
        primary_key = [
            {"user_id": 1, 'follow_user_id': 5},
            {"user_id": 1, 'follow_user_id': 6},
            {"user_id": 1, 'follow_user_id': 7},
            {"user_id": 1, 'follow_user_id': 8},
        ]

        attribute_columns = [
            {'timestamp': datetime.now().timestamp()},
            {'timestamp': datetime.now().timestamp()},
            {'timestamp': datetime.now().timestamp()},
            {'timestamp': datetime.now().timestamp()},
        ]

        ret = OTS().update_list("user_relation_table", primary_key, attribute_columns)
        return Response(ret)

    def delete(self,request):
        primary_key = [
            {"user_id": 1, 'follow_user_id': 5},
            {"user_id": 1, 'follow_user_id': 6},
            {"user_id": 1, 'follow_user_id': 7},
            {"user_id": 1, 'follow_user_id': 8},
        ]
        ret = OTS().delete_list("user_relation_table", primary_key)
        return Response(ret)

    def get(self,request):
        start_key = {"user_id": INF_MIN, 'follow_user_id': INF_MIN}
        end_key = {"user_id": INF_MAX, 'follow_user_id': INF_MAX}

        # 只是按照主键查找数据
        # ret = OTS().get_list("user_relation_table",start_key, end_key,columns_to_get=["timestamp"])
        # 根据条件过滤属性列
        # 第一： 条件查询，字段值的类型严格区分，不同类型，无法查询
        # 第二： 条件查询，只能用于属性列进行，不能使用主键
        cond = SingleColumnCondition("timestamp", 15000033334, ComparatorType.GREATER_THAN)
        ret = OTS().get_list("user_relation_table", start_key, end_key, columns_to_get=["timestamp"],cond=cond)

        return Response(ret)
```

多行路由,代码:

```python
from django.urls import path
from . import views
urlpatterns = [
    path("table/", views.TableAPIView.as_view()),
    path("data/", views.DataAPIView.as_view()),
    path("row/", views.RowAPIView.as_view()),
]
```





## Django自定义终端命令

1. 在app子应用 home目录下创建`management`包，并在`management`包下面创建命令包目录`commands`，`commands`下面就可以创建命令模块文件了。【注意，app子应用必须注册到INSTALL_APPS应用列表中】

2. 在commands包下面创建命令文件，并在文件中声明命令类，例如：tablestore.py，代码：

   ```python
   from django.core.management import BaseCommand
   
   class Command(BaseCommand):
       help = """测试命令的帮助文档"""
   
       def add_arguments(self,parser):
           """参数设置"""
           parser.add_argument("argument",nargs="*", help="必填参数的说明") # 位置参数
           parser.add_argument("--option",'-p', default=None, help="可选参数的说明") # 选项参数
   
       def handle(self, *args, **options):
           """命令主方法
           options: 参数列表
           """
           argument = options.get("argument") # 获取位置参数
           option = options.get("option") # 获取选项参数
   
           self.stdout.write("argument: %s" % argument)
           self.stdout.write("option: %s" % option)
   
           if option is None:
               self.stdout.write("没有设置option选项参数")
   ```

   注意：

   ```
   1. 命令类必须继承于django.core.management.BaseCommand，并且类名必须叫Command。
   2. 命令名称就是文件名，例如，命令文件叫table，则终端下调用命令为： python manage.py table
   3. 命令参数左边加上--，则表示可选参数，可选参数建议设置默认值，方便在handle方法中判断进行默认处理。
   4. 命令参数如果没有--，则表示位置参数，则调用命令时，必须为当前命令传递参数，否则报错！
   ```

   

   接下来我们直接在项目中提供TableStore的操作用于创建Feed流系统的表结构。

   首先把前面封装的OTS工具类，粘贴移动到utils/tablestore.py文件中进行保存。renranapi/utils/tablestore.py，代码：

   ```python
   from tablestore import *
   from django.conf import settings
   
   class OTS(object):
       """工具类： 表格存储"""
       def __init__(self):
           self.client = OTSClient(settings.OTS_ENDPOINT, settings.OTS_ID, settings.OTS_SECRET, settings.OTS_INSTANCE)
   
       def list_table(self):
           """列出所有的存储表"""
           return self.client.list_table()
   
       def create_table(self, table_name, schema_of_primary_key):
           """
           :param table_name: 字符串，表名
           :param schema_of_primary_key: 列表，每一个成员是一个元祖，表示字段的名称和类型
           :return:
           """
           # 通过表名和主键列的schema创建一个tableMeta。
           table_meta = TableMeta(table_name, schema_of_primary_key)
           # 创建TableOptions，数据保留31536000秒，超过后自动删除；最大3个版本；写入时指定的版本值和当前标准时间相差不能超过1天。
           table_options = TableOptions(31536000, 3, 86400)
           # 设置预留读吞吐量为0，预留写吞吐量为0。
           reserved_throughput = ReservedThroughput(CapacityUnit(0, 0))
   
           try:
               self.client.create_table(table_meta, table_options, reserved_throughput)
               return True
           # 处理异常。
           except Exception:
               return False
   
       def delete_table(self, table_name):
           """
           删除表
           :param table_name: 参数如果是字符串，则表示删除一张表，
                              参数如果是列表，则表示删除多张表
           :return:
           """
           tables = []
           if type(table_name) is str:
               tables.append(table_name)
           else:
               tables = table_name
   
           ret = {
               "success": [],
               "fails": []
           }
           for table in tables:
               try:
                   self.client.delete_table(table)
                   ret["success"].append(table)
               except Exception:
                   ret["fails"].append(table)
   
           return ret
   
       def put_row(self, table_name, primary_key, attribute_columns={}):
           """
           添加一条数据
           :param table_name: 本次添加数据的表名
           :param primary_key: 主键列
           :param attribute_columns: 其他属性列
           :return: 新增数据的主键
           """
   
           primary_key_list = []
           for key, value in primary_key.items():
               primary_key_list.append((key, value))
   
           attribute_columns_list = []
           for key, value in attribute_columns.items():
               attribute_columns_list.append((key, value))
   
           row = Row(primary_key_list, attribute_columns_list)
           # 添加数据的条件
           # EXPECT_NOT_EXIST 如果主键重复，则报错！
           condition = Condition(RowExistenceExpectation.EXPECT_NOT_EXIST)
           try:
               # 调用put_row方法, ReturnType.RT_PK表示返回新增数据的主键信息，如果不设置则返回None
               consumed, return_row = self.client.put_row(table_name, row, condition, ReturnType.RT_PK)
               return True, return_row.primary_key
           except OTSClientError as e:
               """网络异常"""
               return False, "put row failed, http_status:%d, error_message:%s" % (
               e.get_http_status(), e.get_error_message())
   
           # 服务端异常，一般为参数错误或者流控错误。
           except OTSServiceError as e:
               """参数有误"""
               return False, "put row failed, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                   e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
           except Exception as e:
               """其他错误"""
               return False, "未知的异常"
   
       def get_row(self, table_name, primary_key, columns_to_get=[], column_filter=None):
           """"""
           # 指定查询结果返回的属性列
           # 过滤条件
           """
           # ====================== 多条件 ========================
           # 逻辑条件
           cond = CompositeColumnCondition(LogicalOperator.AND)
           cond = CompositeColumnCondition(LogicalOperator.OR)
           cond = CompositeColumnCondition(LogicalOperator.NOT)
           # 比较条件
           ComparatorType.NOT_EQUAL  !=
           ComparatorType.EQUAL      ==
           GREATER_THAN              >
           GREATER_EQUAL             >=
           LESS_THAN                 <
           LESS_EQUAL                <=
           举例：
             查询一个学生信息，性别为男的，小于20岁===>   (sex=男 and age < 20) or (sex=女 and age < 17) 
               cond = CompositeColumnCondition(LogicalOperator.AND)
               cond.add_sub_condition(SingleColumnCondition("sex", '男', ComparatorType.EQUAL))
               cond.add_sub_condition(SingleColumnCondition("age", 20, ComparatorType.LESS_THAN))
   
           # ====================== 单条件 ======================== 
           cond = SingleColumnCondition("age", 20, ComparatorType.LESS_THAN)
           """
           primary_key_list = []
           for key, value in primary_key.items():
               primary_key_list.append((key, value))
           try:
               # 调用get_row接口查询
               consumed, return_row, next_token = self.client.get_row(table_name, primary_key_list, columns_to_get,
                                                                      column_filter, 1)
               data = {}
               for att in return_row.primary_key:
                   # 打印每一个字段的内容
                   data[att[0]] = att[1]
               if len(columns_to_get) > 0:
                   """如果有指定要返回其他属性列"""
                   for att in return_row.attribute_columns:
                       # 打印每一个字段的内容
                       data[att[0]] = att[1]
               return True, data
               # 客户端异常，一般为参数错误或者网络异常。
           except OTSClientError as e:
               return False, "获取数据失败, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
           # 服务端异常，一般为参数错误或者流控错误。
           except OTSServiceError as e:
               return False, "获取数据失败, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                   e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
           except Exception as e:
               return False, "未知异常"
   
       def update_row(self, table_name, primary_key, attribute_columns):
           """更新一条数据"""
           primary_key_list = []
           for key, value in primary_key.items():
               primary_key_list.append((key, value))
   
           attribute_columns_list = []
           for key, value in attribute_columns.items():
               attribute_columns_list.append((key, value))
   
           update_of_attribute_columns = {
               'PUT': attribute_columns_list,
           }
   
           row = Row(primary_key_list, update_of_attribute_columns)
   
           try:
               consumed, return_row = self.client.update_row(table_name, row, condition=None, return_type=ReturnType.RT_PK)
               data = {}
               for att in return_row.primary_key:
                   # 打印每一个字段的内容
                   data[att[0]] = att[1]
   
               return True, data
           # 客户端异常，一般为参数错误或者网络异常。
           except OTSClientError as e:
               return False, "更新失败, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
   
           # 服务端异常，一般为参数错误或者流控错误。
           except OTSServiceError as e:
               return False, "更新失败, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                   e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
           except Exception as e:
               return False, "未知异常"
   
       def delete_row(self, table_name, primary_key):
           """根据主键删除一条数据"""
           primary_key_list = []
           for key, value in primary_key.items():
               primary_key_list.append((key, value))
           row = Row(primary_key_list)
           try:
               consumed, return_row = self.client.delete_row(table_name, row, None)
               return True, "删除成功"
           except OTSClientError as e:
               return False, "更新失败！网络异常, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
           except OTSServiceError as e:
               return False, "更新失败，参数有误, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                   e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
           except Exception as e:
               return False, "未知异常"
   
       def get_list_0(self, table_name, primary_key_list, columns_to_get=[], column_filter=None):
           """
           根据指定主键查询多条数据
           :param table: 字符串，表名
           :param primary_key_list: 主键列表
           :param columns_to_get: 返回属性字段列表
           :param column_filter: 条件
           :return: 列表， 查询结果
           """
   
           # 读取3行。
           rows_to_get = []
           for item in primary_key_list:
               primary_key = []
               for key, value in item.items():
                   primary_key.append((key, value))
               rows_to_get.append(primary_key)
   
           # 构造批量读请求。
           request = BatchGetRowRequest()
   
           # 增加表table_name中需要读取的行，最后一个参数1表示读取最新的一个版本。
           request.add(TableInBatchGetRowItem(table_name, rows_to_get, columns_to_get, column_filter, 1))
   
           try:
               result = self.client.batch_get_row(request)
               if result.is_all_succeed():
                   """只有在获取到全部数据以后才表示读取多条数据成功"""
                   table_result = result.get_result_by_table(table_name)
                   data = []
                   for item in table_result:
                       row = {}
   
                       if item.is_ok:
                           for att in item.row.primary_key:
                               # 打印每一个字段的内容
                               row[att[0]] = att[1]
   
                           for att in item.row.attribute_columns:
                               # 打印每一个字段的内容
                               row[att[0]] = att[1]
   
                       else:
                           return False, '部分数据参数有误，读取数据失败。 error code: %s, error message: %s' % (
                           item.error_code, item.error_message)
   
                       data.append(row)
   
                   return True, data
   
               else:
                   return False, '部分数据参数有误，读取数据失败。'
   
           # 客户端异常，一般为参数错误或者网络异常。
           except OTSClientError as e:
               return False, "读取数据失败，网络异常。 http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
           # 服务端异常，一般为参数错误或者流控错误。
           except OTSServiceError as e:
               return False, "读取数据失败，参数有误。 http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
               e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
           except Exception as e:
               return False, "未知异常"
   
       def add_list(self, table_name, primary_key_list, attribute_columns_list=[]):
           """
           添加多条数据
           :param table_name: 字符串，表名
           :param primary_key_list: 主键列
           :param attribute_columns_list: 属性列
           :return:
           """
           put_row_items = self.row_items(PutRowItem, primary_key_list, attribute_columns_list)
           status, result = self.request(table_name, put_row_items)
           if status == False:
               return False, result
           # 输出每一行添加数据的结果。
           succ, fail = result.get_put()
           for item in succ:
               print('添加数据成功, consume %s write cu.' % item.consumed.write)
           for item in fail:
               print('添加数据失败, error code: %s, error message: %s' % (item.error_code, item.error_message))
   
           return True
   
       def update_list(self, table_name, primary_key_list, attribute_columns_list=[]):
           """更新多条数据"""
           update_row_items = self.row_items(UpdateRowItem, primary_key_list, attribute_columns_list)
           status, result = self.request(table_name, update_row_items)
           if status == False:
               return False, result
   
           # 输出每一行更新数据的结果
           succ, fail = result.get_update()
           for item in succ:
               print('更新数据成功, consume %s write cu.' % item.consumed.write)
           for item in fail:
               print('更新数据失败, error code: %s, error message: %s' % (item.error_code, item.error_message))
   
           return True
   
       def delete_list(self, table_name, primary_key_list):
           """删除多条数据"""
           delete_row_items = self.row_items(DeleteRowItem, primary_key_list)
           status, result = self.request(table_name, delete_row_items)
           if status == False:
               return False, result
           # 输出每一行删除数据的结果。
           succ, fail = result.get_delete()
           for item in succ:
               print('删除数据成功, consume %s write cu.' % item.consumed.write)
           for item in fail:
               print('删除数据失败, error code: %s, error message: %s' % (item.error_code, item.error_message))
   
           return True
   
       def row_items(self, cls, primary_key_list, attribute_columns_list=[]):
   
           if len(primary_key_list) != len(attribute_columns_list) and len(attribute_columns_list) != 0:
               return False, "参数有误！主键和属性列数量不对应，无法完成添加操作"
   
           # 增加多行组件的组装。
           row_items = []
           for key0, primary_item in enumerate(primary_key_list):
               primary_row = []
               for key1, value in primary_item.items():
                   primary_row.append((key1, value))
   
               attribute_row = None
               if len(attribute_columns_list) > 0:
                   if cls == PutRowItem:
                       """添加多条"""
                       attribute_row = []
                       for key2, value in attribute_columns_list[key0].items():
                           attribute_row.append((key2, value))
                   elif cls == UpdateRowItem:
                       """更新多条"""
                       attribute_row = {"put": []}
                       for key2, value in attribute_columns_list[key0].items():
                           attribute_row["put"].append((key2, value))
   
               row = Row(primary_row, attribute_row)
               condition = Condition(RowExistenceExpectation.IGNORE)
               item = cls(row, condition)
               row_items.append(item)
   
           return row_items
   
       def request(self, table_name, put_row_items):
           """
           构造批量写请求
           :param table_name:
           :param put_row_items:
           :return:
           """
           request = BatchWriteRowRequest()
           request.add(TableInBatchWriteRowItem(table_name, put_row_items))
           try:
               result = self.client.batch_write_row(request)
               if result.is_all_succeed():
                   return True, result
   
           except OTSClientError as e:
               print(e.get_http_status(), e.get_error_message())
               return False, "网络异常, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
           except OTSServiceError as e:
               return False, "参数有误, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                   e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
           except Exception as e:
               return False, "未知异常"
   
       def get_list(self, table_name, start_key, end_key, columns_to_get=[], limit=90, cond=None,
                    derection=Direction.FORWARD):
           """
           根据指定范围查询数据
           :param table_name: 字符串，表名
           :param start_key: 字典，查询的开始主键位置
           :param end_key: 字典，查询的结束主键位置
           :param columns_to_get: 列表，属性列
           :param limit: 整型, 查询结果数量
           :param cond: Cond类对象，查询条件
           :param derection: 查询的排列方式，Direction.FORWARD 正序，Direction.BACKWARD 倒序
           :return:
           """
           # 设置范围查询的起始主键。
           inclusive_start_primary_key = []
           for key, value in start_key.items():
               inclusive_start_primary_key.append((key, value))
           # 设置范围查询的结束主键。
           exclusive_end_primary_key = []
           for key, value in end_key.items():
               exclusive_end_primary_key.append((key, value))
   
           try:
               # 读取数据
               consumed, next_start_primary_key, row_list, next_token = self.client.get_range(
                   table_name, derection,
                   inclusive_start_primary_key, exclusive_end_primary_key,
                   columns_to_get,
                   limit,
                   column_filter=cond,
                   max_version=1,
               )
   
               all_rows = []
               all_rows.extend(row_list)
   
               # 打印主键和属性列。
               data = []
               for row in all_rows:
                   data_item = {}
                   for att in row.primary_key:
                       data_item[att[0]] = att[1]
                   if len(columns_to_get) > 0:
                       """如果有指定要返回其他属性列"""
                       for att in row.attribute_columns:
                           data_item[att[0]] = att[1]
                   data.append(data_item)
   
               return {"status": True, "data": data, "token": next_start_primary_key}
           # 客户端异常，一般为参数错误或者网络异常。
           except OTSClientError as e:
               return False, "网络异常, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
           # 服务端异常，一般为参数错误或者流控错误。
           except OTSServiceError as e:
               return False, "参数有误, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                   e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
           except Exception as e:
               return False, "未知异常"
   ```

   home/management/commands/tablestore.py，编辑命令代码：

   ```python
   from django.core.management import BaseCommand
   from renranapi.utils.tablestore import *
   class Command(BaseCommand):
       help = """表格存储命令必须接收而且只接收1个命令参数，如下：
       python manage.py tablestore create  表示创建项目使用的表格 
       python manage.py tablestore delete  表示删除项目使用的表格
       """
       def __init__(self, *args, **kwargs):
           self.client = OTS()
           super().__init__(*args, **kwargs)
   
   
       def add_arguments(self, parser):
           """参数设置"""
           parser.add_argument("argument",nargs="*", help="操作类型") # 位置参数
   
       def handle(self, *args, **options):
           """
           表格存储的初始化
           :param args:
           :param options:  参数列表
           :return:
           """
           argument = options.get("argument")
           if len(argument) != 1:
               self.stderr.write("操作有误！")
               self.stdout.write(self.help)
               return None
   
           if argument[0] == "create":
               """创建表格"""
               self.create_table()
   
           elif argument[0] == "delete":
               """删除表格"""
               self.delete_table()
           else:
               self.stderr.write("操作有误！")
               self.stdout.write(self.help)
   
       def create_table(self):
           # 存储库[收件箱]
           table_name = "user_message_table"
           schema_of_primary_key = [  # 主键列
               ('id','INTEGER'),      # 分区键
               ('user_id', 'INTEGER'),
               ('sequence_id', 'INTEGER', PK_AUTO_INCR),
               ("message_id", 'INTEGER')
           ]
           self.stdout.write("表格《%s》创建开始..." % table_name)
           self.client.create_table(table_name, schema_of_primary_key)
           self.stdout.write("表格《%s》创建完成..." % table_name)
   
           # 关系库
           table_name = "user_relation_table"
           schema_of_primary_key = [ # 主键列
               ('id', 'INTEGER'),    # 分区键
               ('user_id', 'INTEGER'),
               ("follow_user_id", 'INTEGER'),
           ]
   
           self.stdout.write("表格《%s》创建开始..." % table_name)
           self.client.create_table(table_name, schema_of_primary_key)
           self.stdout.write("表格《%s》创建完成..." % table_name)
   
           # 未读池
           table_name = "user_message_session_table"
           # 主键列
           schema_of_primary_key = [
               ('id', 'INTEGER'),
               ('follow_user_id', 'INTEGER'),
               ("last_sequence_id", 'INTEGER'),
           ]
   
           self.client.create_table(table_name, schema_of_primary_key)
           self.stdout.write("表格《%s》创建完成..." % table_name)
   
       def delete_table(self):
           """删除表操作"""
           table_list = OTS().list_table()
           for table_name in table_list:
               self.client.delete_table(table_name)
               self.stdout.write("表格《%s》删除完成..." % table_name)
   
           self.stdout.write("所有表格删除完成...")
   ```

   以后,我们创建项目相关的表格,就可以使用`python manage.py create`,如果删除表格则`python manage.py delete`.



因为Feed流系统主要依赖于用户之间的非稳定关系而存在的，所以我们先完成用户之间的关系构建，然后才有数据的推送

## 判断访问者是否关注了作者

在文章详情页中判断当前用户是否关注了文章作者

article/views.py,代码：

```python
from users.models import User
from renranapi.utils.tablestore import OTS
from renranapi.settings import constants
class FocusAPIView(APIView):
    def get(self,request):
        if not isinstance(request.user,User):
            return Response({"status":-2,"detail":"当前用户尚未登录！无法查询关注关系"})

        try:
            author_id = int( request.query_params.get("author_id") )
            User.objects.get(pk=author_id)
        except User.DoesNotExist:
            return Response({"detail": "参数有误，当前作者不存在！"}, status=status.HTTP_400_BAD_REQUEST)

        if author_id == request.user.id:
            return Response({"status":-1,"detail":"当前用户是作者!"})

        ret = OTS().get_row("user_relation_table",{"id": constants.RELATION_TABLE_ID, "user_id":author_id, "follow_user_id":request.user.id})
        return Response({"status": int(ret[0])})
```

settings/constants.py，代码：

```python
# Feed流系统的分区键
RELATION_TABLE_ID = 1   # 关系表的分区键
```



article/urls.py，代码：

```python
    path("focus/", views.FocusAPIView.as_view()),
```



在客户端中根据返回的关注状态`focus_status`来判断显示关注按钮

```vue
<template>
  <div class="_21bLU4 _3kbg6I">
   <Header></Header>
   <div class="_3VRLsv" role="main">
    <div class="_gp-ck">
     <section class="ouvJEz">
      <h1 class="_1RuRku">{{article.title}}</h1>
      <div class="rEsl9f">
       <div class="_2mYfmT">
        <router-link class="_1OhGeD" to="/user" target="_blank" rel="noopener noreferrer"><img class="_13D2Eh" :src="article.user.avatar" alt="" /></router-link>
        <div style="margin-left: 8px;">
         <div class="_3U4Smb">
          <span class="FxYr8x"><router-link class="_1OhGeD" to="/user">{{article.user.nickname?article.user.nickname:article.user.username}}</router-link></span>
          <button data-locale="zh-CN" type="button" class="_3kba3h _1OyPqC _3Mi9q9 _34692-" v-if="focus_status==1"><span>已关注</span></button>
          <button data-locale="zh-CN" type="button" class="_3kba3h _1OyPqC _3Mi9q9 _34692-" v-if="focus_status==0"><span>关注</span></button>
          <button data-locale="zh-CN" type="button" class="_3kba3h _1OyPqC _3Mi9q9 _34692-" v-if="focus_status==-2"><span>关注</span></button>
         </div>
         <div class="s-dsoj">
          <time :datetime="format(article.created_time)">{{format(article.created_time)}}</time>
          <span>字数 {{article.html_content==null?0:article.html_content.length}}</span>
          <span>阅读 {{article.read_count}}</span>
         </div>
        </div>
       </div>
      </div>
      <article class="_2rhmJa">
       <div v-html="article.html_content"></div>
      </article>
      <div></div>
      <div class="_1kCBjS">
       <div class="_18vaTa">
        <div class="_3BUZPB">
         <div class="_2Bo4Th" role="button" tabindex="-1" aria-label="给文章点赞">
          <i aria-label="ic-like" class="anticon">
           <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
            <use xlink:href="#ic-like"></use>
           </svg></i>
         </div>
         <span class="_1LOh_5" role="button" tabindex="-1" aria-label="查看点赞列表">8人点赞<i aria-label="icon: right" class="anticon anticon-right">
           <svg viewbox="64 64 896 896" focusable="false" class="" data-icon="right" width="1em" height="1em" fill="currentColor" aria-hidden="true">
            <path d="M765.7 486.8L314.9 134.7A7.97 7.97 0 0 0 302 141v77.3c0 4.9 2.3 9.6 6.1 12.6l360 281.1-360 281.1c-3.9 3-6.1 7.7-6.1 12.6V883c0 6.7 7.7 10.4 12.9 6.3l450.8-352.1a31.96 31.96 0 0 0 0-50.4z"></path>
           </svg></i></span>
        </div>
        <div class="_3BUZPB">
         <div class="_2Bo4Th" role="button" tabindex="-1">
          <i aria-label="ic-dislike" class="anticon">
           <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
            <use xlink:href="#ic-dislike"></use>
           </svg></i>
         </div>
        </div>
       </div>
       <div class="_18vaTa">
        <a class="_3BUZPB _1x1ok9 _1OhGeD" href="/nb/38290018" target="_blank" rel="noopener noreferrer"><i aria-label="ic-notebook" class="anticon">
          <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
           <use xlink:href="#ic-notebook"></use>
          </svg></i><span>{{article.collection.name}}</span></a>
        <div class="_3BUZPB ant-dropdown-trigger">
         <div class="_2Bo4Th">
          <i aria-label="ic-others" class="anticon">
           <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
            <use xlink:href="#ic-others"></use>
           </svg></i>
         </div>
        </div>
       </div>
      </div>
      <div class="_19DgIp" style="margin-top:24px;margin-bottom:24px"></div>
      <div class="_13lIbp">
       <div class="_191KSt">
        &quot;小礼物走一走，来简书关注我&quot;
       </div>
       <button type="button" class="_1OyPqC _3Mi9q9 _2WY0RL _1YbC5u" @click="is_show_reward_window=true"><span>赞赏支持</span></button>
       <span class="_3zdmIj" v-if="article.reward_count==0">还没有人赞赏，支持一下</span>
       <span class="_3zdmIj" v-else>共{{article.reward_count}}人赞赏了作者</span>
      </div>
      <div class="d0hShY">
       <a class="_1OhGeD" href="/u/a70487cda447" target="_blank" rel="noopener noreferrer"><img class="_27NmgV" src="https://upload.jianshu.io/users/upload_avatars/18529254/.png?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp" alt="  " /></a>
       <div class="Uz-vZq">
        <div class="Cqpr1X">
         <a class="HC3FFO _1OhGeD" href="/u/a70487cda447" title="書酱" target="_blank" rel="noopener noreferrer">書酱</a>
         <span class="_2WEj6j" title="你读书的样子真好看。">你读书的样子真好看。</span>
        </div>
        <div class="lJvI3S">
         <span>总资产0</span>
         <span>共写了78.7W字</span>
         <span>获得6,072个赞</span>
         <span>共1,308个粉丝</span>
        </div>
       </div>
       <button data-locale="zh-CN" type="button" class="_1OyPqC _3Mi9q9"><span>关注</span></button>
      </div>
     </section>
     <div id="note-page-comment">
      <div class="lazyload-placeholder"></div>
     </div>
    </div>
    <aside class="_2OwGUo">
     <section class="_3Z3nHf">
      <div class="_3Oo-T1">
       <a class="_1OhGeD" href="/u/a70487cda447" target="_blank" rel="noopener noreferrer"><img class="_3T9iJQ" src="https://upload.jianshu.io/users/upload_avatars/18529254/.png?imageMogr2/auto-orient/strip|imageView2/1/w/90/h/90/format/webp" alt="" /></a>
       <div class="_32ZTTG">
        <div class="_2O0T_w">
         <div class="_2v-h3G">
          <span class="_2vh4fr" title="書酱"><a class="_1OhGeD" href="/u/a70487cda447" target="_blank" rel="noopener noreferrer">書酱</a></span>
         </div>
         <button data-locale="zh-CN" type="button" class="tzrf9N _1OyPqC _3Mi9q9 _34692-"><span>关注</span></button>
        </div>
        <div class="_1pXc22">
         总资产0
        </div>
       </div>
      </div>
      <div class="_19DgIp"></div>
     </section>
     <div>
      <div class="">
       <section class="_3Z3nHf">
        <h3 class="QHRnq8 QxT4hD"><span>推荐阅读</span></h3>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="这些话没人告诉你，但必须知道的社会规则">
          <a class="_1-HJSV _1OhGeD" href="/p/a3e56a0559ff" target="_blank" rel="noopener noreferrer">这些话没人告诉你，但必须知道的社会规则</a>
         </div>
         <div class="_19haGh">
          阅读 5,837
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="浙大学霸最美笔记曝光：真正的牛人，都“变态”到了极致">
          <a class="_1-HJSV _1OhGeD" href="/p/d2a3724e2839" target="_blank" rel="noopener noreferrer">浙大学霸最美笔记曝光：真正的牛人，都“变态”到了极致</a>
         </div>
         <div class="_19haGh">
          阅读 12,447
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="征服一个女人最好的方式：不是讨好她，而是懂得去折腾她">
          <a class="_1-HJSV _1OhGeD" href="/p/f6acf67f039b" target="_blank" rel="noopener noreferrer">征服一个女人最好的方式：不是讨好她，而是懂得去折腾她</a>
         </div>
         <div class="_19haGh">
          阅读 5,311
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="告别平庸的15个小方法">
          <a class="_1-HJSV _1OhGeD" href="/p/cff7eb6b232b" target="_blank" rel="noopener noreferrer">告别平庸的15个小方法</a>
         </div>
         <div class="_19haGh">
          阅读 7,040
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="轻微抑郁的人，会说这3句“口头禅”，若你一个不占，偷着乐吧">
          <a class="_1-HJSV _1OhGeD" href="/p/2a0ca1729b4b" target="_blank" rel="noopener noreferrer">轻微抑郁的人，会说这3句“口头禅”，若你一个不占，偷着乐吧</a>
         </div>
         <div class="_19haGh">
          阅读 16,411
         </div>
        </div>
       </section>
      </div>
     </div>
    </aside>
   </div>

   <div class="_23ISFX-body" v-if="is_show_reward_window" @click.stop="is_show_reward_window=true">
     <div class="_3uZ5OL">
      <div class="_2PLkjk">
       <img class="_2R1-48" :src="article.user.avatar" alt="" />
       <div class="_2h5tnQ">
        给作者送糖
       </div>
      </div>
      <div class="_1-bCJJ">
       <div class="LMa6S_" :class="reward_info.money==num?'_1vONvL':''" @click="reward_info.money=num" v-for="num in reward_list"><span>{{num}}元</span></div>
      </div>
      <textarea class="_1yN79W" placeholder="给Ta留言..." v-model="reward_info.content"></textarea>
      <div class="_1_B577">选择支付方式</div>
      <div class="_1-bCJJ">
       <div class="LMa6S_ _3PA8BN" :class="reward_info.pay_type==type?'_1vONvL':''" @click="reward_info.pay_type=type" v-for="type in pay_type_list"><span>{{type}}</span></div>
      </div>
      <button type="button" class="_3A-4KL _1OyPqC _3Mi9q9 _1YbC5u" @click="payhandler"><span>确认支付</span><span> ￥</span>{{reward_info.money}}</button>
     </div>
    </div>

   <Footer></Footer>
  </div>
</template>

<script>
    import Header from "./common/Header";
    import Footer from "./common/Footer";
    export default {
        name: "Article",
        components:{
          Header,
          Footer,
        },
        data(){
          return {
            token: "",
            article: {
                collection:{},
                user:{},
            },
            is_show_reward_window:false, // 是否显示打赏窗口
            reward_list:[2,5,10,20,50,100],  // 赞赏的金额
            pay_type_list: ["支付宝","余额支付"], // 赞赏的支付方式
            reward_info:{  // 赞赏的表单信息
                money: 2,
                content:"",
                pay_type:"支付宝",
            },
            focus_status: -2, // 当前用户是否关注作者的状态，-2表示没登录,-1表示用户与作者是同一个人，0表示没有关注，1表示已关注
          }
        },
        created(){
            // 获取路由参数
            this.article.id = this.$route.params.id;
            this.get_article();
        },
        methods:{
            payhandler(){
              this.$message.info("正在打赏处理中。。。请稍后！～");
              // 打赏作者的支付处理
              if(!this.token){
                  this.token = this.$settings.check_user_login(this);
              }

              if(this.token && this.reward_info.pay_type=="支付宝"){
                // 发起支付
                this.$axios.post(`${this.$settings.Host}/payments/alipay/`,{
                    money: this.reward_info.money,
                    message: this.reward_info.content,
                    article_id: this.article.id,
                },{
                    headers:{
                      Authorization: "jwt " + this.token
                    }
                }).then(response=>{
                    // location.href=response.data;
                    this.get_reward_status(response.data.out_trade_no);
                    open(response.data.url,"_blank");
                }).catch(error=>{
                    this.$message.error("网络异常，无法进行打赏！");
                })
              }
            },
            get_reward_status(out_trade_no){
              // 查询打赏支付的结果
              this.is_show_reward_window = false;
              let timer = setInterval(()=>{
                  this.$axios.get(`${this.$settings.Host}/payments/alipay/result/`,{
                      params:{
                          out_trade_no:out_trade_no,
                      }
                  }).then(response=>{
                     if(response.data.status){
                       this.$message.success("支付成功了！");
                       this.article.reward_count+=1;
                       clearInterval(timer);
                     }
                  }).catch(error=>{
                     this.$message.error("网络异常，无法查询打赏支付的结果!");
                     clearInterval(timer);
                  });
              },5000);
            },
            get_article(){
              // 获取文章内容信息
              this.$axios.get(`${this.$settings.Host}/article/${this.article.id}/retrieve/`).then(response=>{
                  this.article = response.data;
                  // 查询当前访问用户与作者的关系
                  this.get_focus_status();
              }).catch(error=>{
                  this.$message.error("网络异常，获取文章信息失败！");
              })
            },
            get_focus_status(){
              // 查询用户是否关注了作者
              if(!this.token){
                  this.token = this.$settings.check_user_login(this);
              }
              if(this.token == false){
                  // 如果token不存在，则表示没有登录
                  return;
              }
              this.$axios.get(`${this.$settings.Host}/article/focus/`,{
                  params:{
                      author_id: this.article.user.id
                  },
                  headers:{
                    Authorization: "jwt " + this.token
                  }
              }).then(response=>{
                  this.focus_status = response.data.status;
              }).catch(error=>{
                  this.$message.error(error.response.data.detail);
              });
            },
            format(time){
              time = new Date(time);
              let Y = time.getFullYear();
              let m = time.getMonth()+1;
              m = m<10?('0'+m):m;
              let d = time.getDate();
              d = d<10?('0'+d):d;
              let H = time.getHours();
              H = H<10?('0'+H):H;
              let M = time.getMinutes();
              M = M<10?('0'+M):M;
              let S = time.getSeconds();
              S = S<10?('0'+S):S;
              return `${Y}-${m}-${d} ${H}:${M}:${S}`;
            }
        }
    }
</script>
```



## 用户关注或取关作者

服务端实现用户关注作者的api接口

users/views.py,代码:

```python
from users.models import User
from renranapi.utils.tablestore import OTS
from renranapi.settings import constants
class FocusAPIView(APIView):
    def get(self,request):
        if not isinstance(request.user,User):
            return Response({"status":-2,"detail":"当前用户尚未登录！无法查询关注关系"})

        try:
            author_id = int( request.query_params.get("author_id") )
            User.objects.get(pk=author_id)
        except User.DoesNotExist:
            return Response({"detail": "参数有误，当前作者不存在！"}, status=status.HTTP_400_BAD_REQUEST)

        if author_id == request.user.id:
            return Response({"status":-1,"detail":"当前用户是作者!"})

        ret = OTS().get_row("user_relation_table",{"id": constants.RELATION_TABLE_ID, "user_id":author_id, "follow_user_id":request.user.id})
        return Response({"status": int(ret[0])})

    def put(self,request):
        """切换关注关系"""
        if not isinstance(request.user, User):
            return Response({"detail":"用户尚未用户，无法操作"}, status=status.HTTP_401_UNAUTHORIZED)

        try:
            author_id = int( request.data.get("author_id") )
            User.objects.get(pk=author_id)
        except User.DoesNotExist:
            return Response({"detail": "参数有误，当前作者不存在！"}, status=status.HTTP_400_BAD_REQUEST)

        if author_id == request.user.id:
            return Response({"detail":"当前用户是作者!"}, status=status.HTTP_400_BAD_REQUEST)

        focus_status = bool( int( request.data.get("status") ) )

        client = OTS()
        primary_key = {"id": constants.RELATION_TABLE_ID, "user_id":author_id, "follow_user_id":request.user.id}
        if focus_status:
            message = "取消关注"
            ret = client.delete_row("user_relation_table",primary_key)
        else:
            message = "关注"
            ret = client.put_row("user_relation_table",primary_key)

        if ret[0] == False:
            return Response({"detail": "%s失败！" % message}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

        return Response({"detail": "%s成功！" % message})
```



客户端发送请求,申请 关注/取消关注

```vue
<template>
  <div class="_21bLU4 _3kbg6I">
   <Header></Header>
   <div class="_3VRLsv" role="main">
    <div class="_gp-ck">
     <section class="ouvJEz">
      <h1 class="_1RuRku">{{article.title}}</h1>
      <div class="rEsl9f">
       <div class="_2mYfmT">
        <router-link class="_1OhGeD" to="/user" target="_blank" rel="noopener noreferrer"><img class="_13D2Eh" :src="article.user.avatar" alt="" /></router-link>
        <div style="margin-left: 8px;">
         <div class="_3U4Smb">
          <span class="FxYr8x"><router-link class="_1OhGeD" to="/user">{{article.user.nickname?article.user.nickname:article.user.username}}</router-link></span>
          <button data-locale="zh-CN" type="button" class="_3kba3h _1OyPqC _3Mi9q9 _34692-" v-if="focus_status==1" @click="focus_author(focus_status)"><span>已关注</span></button>
          <button data-locale="zh-CN" type="button" class="_3kba3h _1OyPqC _3Mi9q9 _34692-" v-if="focus_status==0" @click="focus_author(focus_status)"><span>关注</span></button>
          <button data-locale="zh-CN" type="button" class="_3kba3h _1OyPqC _3Mi9q9 _34692-" v-if="focus_status==-2" @click="focus_author(focus_status)"><span>关注</span></button>
         </div>
         <div class="s-dsoj">
          <time :datetime="format(article.created_time)">{{format(article.created_time)}}</time>
          <span>字数 {{article.html_content==null?0:article.html_content.length}}</span>
          <span>阅读 {{article.read_count}}</span>
         </div>
        </div>
       </div>
      </div>
      <article class="_2rhmJa">
       <div v-html="article.html_content"></div>
      </article>
      <div></div>
      <div class="_1kCBjS">
       <div class="_18vaTa">
        <div class="_3BUZPB">
         <div class="_2Bo4Th" role="button" tabindex="-1" aria-label="给文章点赞">
          <i aria-label="ic-like" class="anticon">
           <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
            <use xlink:href="#ic-like"></use>
           </svg></i>
         </div>
         <span class="_1LOh_5" role="button" tabindex="-1" aria-label="查看点赞列表">8人点赞<i aria-label="icon: right" class="anticon anticon-right">
           <svg viewbox="64 64 896 896" focusable="false" class="" data-icon="right" width="1em" height="1em" fill="currentColor" aria-hidden="true">
            <path d="M765.7 486.8L314.9 134.7A7.97 7.97 0 0 0 302 141v77.3c0 4.9 2.3 9.6 6.1 12.6l360 281.1-360 281.1c-3.9 3-6.1 7.7-6.1 12.6V883c0 6.7 7.7 10.4 12.9 6.3l450.8-352.1a31.96 31.96 0 0 0 0-50.4z"></path>
           </svg></i></span>
        </div>
        <div class="_3BUZPB">
         <div class="_2Bo4Th" role="button" tabindex="-1">
          <i aria-label="ic-dislike" class="anticon">
           <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
            <use xlink:href="#ic-dislike"></use>
           </svg></i>
         </div>
        </div>
       </div>
       <div class="_18vaTa">
        <a class="_3BUZPB _1x1ok9 _1OhGeD" href="/nb/38290018" target="_blank" rel="noopener noreferrer"><i aria-label="ic-notebook" class="anticon">
          <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
           <use xlink:href="#ic-notebook"></use>
          </svg></i><span>{{article.collection.name}}</span></a>
        <div class="_3BUZPB ant-dropdown-trigger">
         <div class="_2Bo4Th">
          <i aria-label="ic-others" class="anticon">
           <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
            <use xlink:href="#ic-others"></use>
           </svg></i>
         </div>
        </div>
       </div>
      </div>
      <div class="_19DgIp" style="margin-top:24px;margin-bottom:24px"></div>
      <div class="_13lIbp">
       <div class="_191KSt">
        &quot;小礼物走一走，来简书关注我&quot;
       </div>
       <button type="button" class="_1OyPqC _3Mi9q9 _2WY0RL _1YbC5u" @click="is_show_reward_window=true"><span>赞赏支持</span></button>
       <span class="_3zdmIj" v-if="article.reward_count==0">还没有人赞赏，支持一下</span>
       <span class="_3zdmIj" v-else>共{{article.reward_count}}人赞赏了作者</span>
      </div>
      <div class="d0hShY">
       <a class="_1OhGeD" href="/u/a70487cda447" target="_blank" rel="noopener noreferrer"><img class="_27NmgV" src="https://upload.jianshu.io/users/upload_avatars/18529254/.png?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp" alt="  " /></a>
       <div class="Uz-vZq">
        <div class="Cqpr1X">
         <a class="HC3FFO _1OhGeD" href="/u/a70487cda447" title="書酱" target="_blank" rel="noopener noreferrer">書酱</a>
         <span class="_2WEj6j" title="你读书的样子真好看。">你读书的样子真好看。</span>
        </div>
        <div class="lJvI3S">
         <span>总资产0</span>
         <span>共写了78.7W字</span>
         <span>获得6,072个赞</span>
         <span>共1,308个粉丝</span>
        </div>
       </div>
       <button data-locale="zh-CN" type="button" class="_1OyPqC _3Mi9q9"><span>关注</span></button>
      </div>
     </section>
     <div id="note-page-comment">
      <div class="lazyload-placeholder"></div>
     </div>
    </div>
    <aside class="_2OwGUo">
     <section class="_3Z3nHf">
      <div class="_3Oo-T1">
       <a class="_1OhGeD" href="/u/a70487cda447" target="_blank" rel="noopener noreferrer"><img class="_3T9iJQ" src="https://upload.jianshu.io/users/upload_avatars/18529254/.png?imageMogr2/auto-orient/strip|imageView2/1/w/90/h/90/format/webp" alt="" /></a>
       <div class="_32ZTTG">
        <div class="_2O0T_w">
         <div class="_2v-h3G">
          <span class="_2vh4fr" title="書酱"><a class="_1OhGeD" href="/u/a70487cda447" target="_blank" rel="noopener noreferrer">書酱</a></span>
         </div>
         <button data-locale="zh-CN" type="button" class="tzrf9N _1OyPqC _3Mi9q9 _34692-"><span>关注</span></button>
        </div>
        <div class="_1pXc22">
         总资产0
        </div>
       </div>
      </div>
      <div class="_19DgIp"></div>
     </section>
     <div>
      <div class="">
       <section class="_3Z3nHf">
        <h3 class="QHRnq8 QxT4hD"><span>推荐阅读</span></h3>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="这些话没人告诉你，但必须知道的社会规则">
          <a class="_1-HJSV _1OhGeD" href="/p/a3e56a0559ff" target="_blank" rel="noopener noreferrer">这些话没人告诉你，但必须知道的社会规则</a>
         </div>
         <div class="_19haGh">
          阅读 5,837
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="浙大学霸最美笔记曝光：真正的牛人，都“变态”到了极致">
          <a class="_1-HJSV _1OhGeD" href="/p/d2a3724e2839" target="_blank" rel="noopener noreferrer">浙大学霸最美笔记曝光：真正的牛人，都“变态”到了极致</a>
         </div>
         <div class="_19haGh">
          阅读 12,447
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="征服一个女人最好的方式：不是讨好她，而是懂得去折腾她">
          <a class="_1-HJSV _1OhGeD" href="/p/f6acf67f039b" target="_blank" rel="noopener noreferrer">征服一个女人最好的方式：不是讨好她，而是懂得去折腾她</a>
         </div>
         <div class="_19haGh">
          阅读 5,311
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="告别平庸的15个小方法">
          <a class="_1-HJSV _1OhGeD" href="/p/cff7eb6b232b" target="_blank" rel="noopener noreferrer">告别平庸的15个小方法</a>
         </div>
         <div class="_19haGh">
          阅读 7,040
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="轻微抑郁的人，会说这3句“口头禅”，若你一个不占，偷着乐吧">
          <a class="_1-HJSV _1OhGeD" href="/p/2a0ca1729b4b" target="_blank" rel="noopener noreferrer">轻微抑郁的人，会说这3句“口头禅”，若你一个不占，偷着乐吧</a>
         </div>
         <div class="_19haGh">
          阅读 16,411
         </div>
        </div>
       </section>
      </div>
     </div>
    </aside>
   </div>

   <div class="_23ISFX-body" v-if="is_show_reward_window" @click.stop="is_show_reward_window=true">
     <div class="_3uZ5OL">
      <div class="_2PLkjk">
       <img class="_2R1-48" :src="article.user.avatar" alt="" />
       <div class="_2h5tnQ">
        给作者送糖
       </div>
      </div>
      <div class="_1-bCJJ">
       <div class="LMa6S_" :class="reward_info.money==num?'_1vONvL':''" @click="reward_info.money=num" v-for="num in reward_list"><span>{{num}}元</span></div>
      </div>
      <textarea class="_1yN79W" placeholder="给Ta留言..." v-model="reward_info.content"></textarea>
      <div class="_1_B577">选择支付方式</div>
      <div class="_1-bCJJ">
       <div class="LMa6S_ _3PA8BN" :class="reward_info.pay_type==type?'_1vONvL':''" @click="reward_info.pay_type=type" v-for="type in pay_type_list"><span>{{type}}</span></div>
      </div>
      <button type="button" class="_3A-4KL _1OyPqC _3Mi9q9 _1YbC5u" @click="payhandler"><span>确认支付</span><span> ￥</span>{{reward_info.money}}</button>
     </div>
    </div>

   <Footer></Footer>
  </div>
</template>

<script>
    import Header from "./common/Header";
    import Footer from "./common/Footer";
    export default {
        name: "Article",
        components:{
          Header,
          Footer,
        },
        data(){
          return {
            token: "",
            article: {
                collection:{},
                user:{},
            },
            is_show_reward_window:false, // 是否显示打赏窗口
            reward_list:[2,5,10,20,50,100],  // 赞赏的金额
            pay_type_list: ["支付宝","余额支付"], // 赞赏的支付方式
            reward_info:{  // 赞赏的表单信息
                money: 2,
                content:"",
                pay_type:"支付宝",
            },
            focus_status: -2, // 当前用户是否关注作者的状态，-2表示没登录,-1表示用户与作者是同一个人，0表示没有关注，1表示已关注
          }
        },
        created(){
            // 获取路由参数
            this.article.id = this.$route.params.id;
            this.get_article();
        },
        methods:{
            payhandler(){
              this.$message.info("正在打赏处理中。。。请稍后！～");
              // 打赏作者的支付处理
              if(!this.token){
                  this.token = this.$settings.check_user_login(this);
              }

              if(this.token && this.reward_info.pay_type=="支付宝"){
                // 发起支付
                this.$axios.post(`${this.$settings.Host}/payments/alipay/`,{
                    money: this.reward_info.money,
                    message: this.reward_info.content,
                    article_id: this.article.id,
                },{
                    headers:{
                      Authorization: "jwt " + this.token
                    }
                }).then(response=>{
                    // location.href=response.data;
                    this.get_reward_status(response.data.out_trade_no);
                    open(response.data.url,"_blank");
                }).catch(error=>{
                    this.$message.error("网络异常，无法进行打赏！");
                })
              }
            },
            get_reward_status(out_trade_no){
              // 查询打赏支付的结果
              this.is_show_reward_window = false;
              let timer = setInterval(()=>{
                  this.$axios.get(`${this.$settings.Host}/payments/alipay/result/`,{
                      params:{
                          out_trade_no:out_trade_no,
                      }
                  }).then(response=>{
                     if(response.data.status){
                       this.$message.success("支付成功了！");
                       this.article.reward_count+=1;
                       clearInterval(timer);
                     }
                  }).catch(error=>{
                     this.$message.error("网络异常，无法查询打赏支付的结果!");
                     clearInterval(timer);
                  });
              },5000);
            },
            get_article(){
              // 获取文章内容信息
              this.$axios.get(`${this.$settings.Host}/article/${this.article.id}/retrieve/`).then(response=>{
                  this.article = response.data;
                  // 查询当前访问用户与作者的关系
                  this.get_focus_status();
              }).catch(error=>{
                  this.$message.error("网络异常，获取文章信息失败！");
              })
            },
            get_focus_status(){
              // 查询用户是否关注了作者
              if(!this.token){
                  this.token = this.$settings.check_user_login(this);
              }
              if(this.token == false){
                  // 如果token不存在，则表示没有登录
                  return;
              }
              this.$axios.get(`${this.$settings.Host}/article/focus/`,{
                  params:{
                      author_id: this.article.user.id
                  },
                  headers:{
                    Authorization: "jwt " + this.token
                  }
              }).then(response=>{
                  this.focus_status = response.data.status;
              }).catch(error=>{
                  this.$message.error(error.response.data.detail);
              });
            },
            focus_author(status){
                if(!this.token){
                  this.token = this.$settings.check_user_login(this);
                }
                if(this.token == false){
                    // 如果token不存在，则表示没有登录
                    return;
                }
                if(status == -2){
                    this.$confirm('您尚未登录，无法完成操作，是否登录继续？', '提示', {
                      confirmButtonText: '登录',
                      cancelButtonText: '取消',
                      type: 'warning'
                    }).then(() => {
                      this.$router.push("/login");
                    }).catch(() => {

                    });
                    return;
                }

                this.$axios.put(`${this.$settings.Host}/article/focus/`,{
                   author_id: this.article.user.id,
                   status: status,
                },{
                   headers:{
                      Authorization: "jwt " + this.token
                   }
                }).then(response=>{
                    this.focus_status = response.data.status;
                    this.$message.success(response.data.detail);
                }).catch(error=>{
                    this.$message.error(error.response.data.detail);
                });

            },
            format(time){
              time = new Date(time);
              let Y = time.getFullYear();
              let m = time.getMonth()+1;
              m = m<10?('0'+m):m;
              let d = time.getDate();
              d = d<10?('0'+d):d;
              let H = time.getHours();
              H = H<10?('0'+H):H;
              let M = time.getMinutes();
              M = M<10?('0'+M):M;
              let S = time.getSeconds();
              S = S<10?('0'+S):S;
              return `${Y}-${m}-${d} ${H}:${M}:${S}`;
            }
        }
    }
</script>
```



## 定时发布时推送Feed流

把关于推送和取消推送Fee的信息的代码封装到模型。

视图article/views.py，ArticleAPIView的代码，调整：

```python
from .models import Article
from .serializers import ArticleModelSerializer
from rest_framework.response import Response
class ArticleAPIView(ListAPIView,CreateAPIView):
    """文章视图接口"""
    # queryset = Article.objects.all()
    serializer_class = ArticleModelSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        # /article/collection/<collection>\d/articles/
        # 获取路由参数的另一种方式 self.kwargs.get("参数名称")
        collection_id = self.kwargs.get("collection")
        return Article.objects.filter(is_deleted=False, is_show=True, user=self.request.user, collection_id=collection_id).order_by("orders", "id")

    def patch(self,request,pk):
        try:
            article = Article.objects.get(pk=pk,is_deleted=False, is_show=True, user=self.request.user)
        except Article.DoesNotExist:
            return Response({"detail":"当前文章不存在!"}, status=status.HTTP_400_BAD_REQUEST)
        """
        1. 隐私文章 is_public=False, pub_date=None
        2. 发布文章 is_public=True, pub_date=None
        3. 定时文章 is_public=False, pub_date=时间
        """
        if article.pub_date:
            """取消定时发布文章"""
            article.pub_date=None
        elif article.is_public:
            """把文章设置为隐私文章"""
            article.is_public=False
            # 取消推送Feed流
            article.cancel_push_feed()
        else:
            """发布文章"""
            article.is_public=True
            # 推送feed流给粉丝
            article.push_feed()
        article.save()
        return Response({"detail":"发布状态切换成功!"})

```

article/models.py，Article代码，调整：

```python
from renranapi.utils.tablestore import *
from django.utils import timezone as datetime
from renranapi.settings import constants
class Article(BaseModel):
    title = models.CharField(max_length=150, db_index=True, verbose_name="文章")
    content = models.TextField(null=True, blank=True, verbose_name="文章内容")
    html_content = models.TextField(null=True, blank=True, verbose_name="文章内容[html格式]")
    user = models.ForeignKey(User,related_name="myarticles", on_delete=models.DO_NOTHING, verbose_name="作者")
    collection = models.ForeignKey(ArticleCollection, related_name="article_list", on_delete=models.CASCADE, verbose_name="文集")
    pub_date = models.DateTimeField(null=True,blank=True, default=None, verbose_name="发布时间")
    access_pwd = models.CharField(max_length=15, null=True, blank=True, verbose_name="访问密码")
    read_count = models.IntegerField(default=0, null=True, blank=True, verbose_name="阅读量")
    like_count = models.IntegerField(default=0, null=True, blank=True, verbose_name="点赞量")
    collect_count = models.IntegerField(default=0, null=True, blank=True, verbose_name="收藏量")
    comment_count = models.IntegerField(default=0, null=True, blank=True, verbose_name="评论量")
    reward_count = models.IntegerField(default=0, null=True, blank=True, verbose_name="赞赏量")
    is_public = models.BooleanField(default=False, verbose_name="是否公开")

    class Meta:
        db_table = "rr_article"
        verbose_name = "文章"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.title

    def push_feed(self):
        """推送feed流"""
        # 获取当前作者的粉丝列表
        flowers_list = self.get_author_flowers(self.user.id)
        if len(flowers_list)<1:
            return False

        # 把推送数据填写到同步库中
        primary_key_list = [{
                "id": constants.MESSAGE_TABLE_ID,
                "user_id":flower,
                "sequence_id": PK_AUTO_INCR,
                "message_id": self.id
        } for flower in flowers_list]

        attribute_columns_list = [{
                "timestamp": datetime.now().timestamp(), # 推送时间
                "is_read": False,    # 是否阅读
                "is_cancel": False,  # 是否取消推送
        } for flower in flowers_list]

        ret = OTS().add_list("user_message_table",primary_key_list,attribute_columns_list)

    def get_author_flowers(self,author_id):
        """获取当前作者的所有粉丝"""
        start_key = {"id":1,"user_id": author_id,"follow_user_id":INF_MIN} # 查询开始的主键列
        end_key = {"id":1,"user_id": author_id,"follow_user_id":INF_MAX}   # 查询结束的主键列

        # 获取粉丝数据，默认一次最多只能获取90
        ret = OTS().get_list("user_relation_table",start_key,end_key)
        data = []
        if ret["status"]:
            data.extend(ret["data"])
            while ret["token"]: # 当数据超过90条时，token表示下一次查询的开始主键列
                start_key = ret["token"]
                ret = OTS().get_list("user_relation_table", start_key, end_key)
                data.extend(ret["data"])

        flowers_list = [item["follow_user_id"] for item in data ]
        return flowers_list

    def cancel_push_feed(self):
        # 取消推送记录
        # 把推送数据填写到同步库中
        primary_key_list = self.get_feed_list() # 查询当前文章的所有推送记录
        print(primary_key_list)
        if len(primary_key_list) < 1:
            return False

        attribute_columns_list = [{"is_cancel":True} for i in primary_key_list]
        OTS().update_list("user_message_table",primary_key_list,attribute_columns_list)

    def get_feed_list(self):
        """获取指定文章的推送记录"""
        start_key = {
            "id": constants.MESSAGE_TABLE_ID,
            "user_id": INF_MIN,
            "sequence_id":INF_MIN,
            "message_id":self.id
        }
        end_key = {
            "id": constants.MESSAGE_TABLE_ID,
            "user_id": INF_MAX,
            "sequence_id": INF_MAX,
            "message_id":self.id
        }
        # 获取文章推送记录，默认一次最多只能获取90
        ret = OTS().get_list("user_message_table",start_key, end_key)

        data = []
        if ret["status"]:
            data.extend(ret["data"])
            while ret["token"]:  # 当数据超过90条时，token表示下一次查询的开始主键列
                start_key = ret["token"]
                ret = OTS().get_list("user_message_table", start_key, end_key)
                data.extend(ret["data"])

        return data

```

mycelery/article/tasks.py，代码：

```python
from mycelery.main import app
from article.models import Article
from django.utils import timezone as datetime
@app.task(name="interval_pub_article")
def interval_pub_article():
    """定时发布"""
    article_list = Article.objects.filter(pub_date__lte=datetime.now())
    print(article_list)
    for article in article_list:
        article.is_public = True
        article.pub_date = None
        article.save()
        # 推送feed流
        article.push_feed()
        print("文章《%s》发布成功" % article.title )
```



## 首页数据内容展示

首页数据在Feed流系统中, 需要考虑2大问题:

针对没有登录的游客显示内容的问题:

1. 纯粹的游客
2. 已经注册但是没有关注过任何作者

```
解决:
   1. 从数据库查找到热门内容推送给用户[评论量高的, 赞赏量高的, 点赞量高的]
      如果是登录用户, 在查看了内容后, 针对用户ID保存查看记录, 哪一篇用户看过了就记录到tablestore里面
      如果是游客, 1. 在本地存储中,记录用户的浏览历史
                 2. 在tablestore里面记录当前IP的浏览历史
```



针对登录的用户显示内容的问题:

3.  用户关注了很多作者，内容足够展示给用户

4.  用户关注了作者很少, 这些作者可能没有新的内容产生



```python
解决第一种推送内容不足的情况, 我们可以根据用户行为进行分析的实现基于物品的协同过滤算法来计算出用户的兴趣, 进行智能推荐.
```

综合上面所述,我们必须要显示首页的内容先然后对显示的内容进行一下步骤的过滤

```
1. 判断用户是否登录
   1.1. 用户登录了
        1.1.1 用户已经关注了其他作者
          1.1.1.1 用户关注作者中,有足够的内容展示给用户
          1.1.1.2 用户关注作者中,没有足够内容展示给用户【智能推荐】
        1.1.2 用户没有关注任何的作者
          1.1.1.1 根据热度热度推荐
   1.2. 用户未登录
        1.1.3 根据热度热度推荐
```

### 服务端提供推送文章的内容

home/views.py,代码:

```python
from article.models import Article
from .serializers import ArticleModelSerializer
from .paginations import HomeArticlePageNumberPagination
class ArticleListAPIView(ListAPIView):
    serializer_class = ArticleModelSerializer
    queryset = Article.objects.filter(is_public=True, is_show=True, is_deleted=False).order_by("-reward_count", "-comment_count", "-like_count", "-id")
    pagination_class = HomeArticlePageNumberPagination
```

home/paginations.py,代码:

```python
from rest_framework.pagination import PageNumberPagination
class HomeArticlePageNumberPagination(PageNumberPagination):
    """首页推送文章的分页器"""
    page_query_param = "page" # 地址上面代表页码的参数名
    max_page_size = 20 # 每一页显示的最大数据量
    page_size = 10     # 默认每一页显示的数据量
    page_size_query_param = "size" # 地址上面代表数据量的参数名
```

home/serializers.py,代码:

```python
from article.models import Article
from article.serializers import UserModelSerializer
class ArticleModelSerializer(serializers.ModelSerializer):
    user = UserModelSerializer()
    class Meta:
        model = Article
        fields = ["id", "title", "html_content", "user", "pub_date", "read_count", "like_count", "collect_count", "comment_count", "reward_count",]
```

home/urls.py 子应用路由。

```python
    path("push/article/", views.ArticleListAPIView.as_view()),
```



### 客户端请求获取文章列表数据

```vue
<template>
  <div id="home">
    <Header></Header>
    <div class="container">
      <div class="row">
        <div class="main">
          <!-- Banner -->
          <div class="banner">
            <el-carousel height="272px" indicator-position="none" :interval="2000">
              <el-carousel-item v-for="item,key in banner_list" :key="key">
                <a :href="item.link" v-if="item.is_http"><img :src="item.image" alt=""></a>
                <router-link :to="item.link" v-else><img :src="item.image" alt=""></router-link>
              </el-carousel-item>
            </el-carousel>
          </div>
          <div id="list-container">
            <!-- 文章列表模块 -->
            <ul class="note-list">
              <li class="" v-for="article in article_list">
                <div class="content">
                  <router-link class="title" :to="`/article/${article.id}`">{{article.title}}</router-link>
                  <p class="abstract">{{article.html_content|format}}</p>
                  <div class="meta">
                    <router-link class="nickname" :to="`/user/${article.user.id}`">{{article.user.nickname?article.user.nickname:article.user.username}}</router-link>
                    <router-link :to="`/article/${article.id}#comment`">
                      <img src="/static/image/comment.svg" alt=""> {{article.comment_count}}
                    </router-link>
                    <span v-if="article.like_count>0"><img src="/static/image/like.svg" alt=""> {{article.like_count}}</span>
                    <span v-if="article.reward_count>0"><img src="/static/image/shang.svg" alt=""> {{article.reward_count}}</span>
                  </div>
                </div>
              </li>
<!--              <li class="have-img">-->
<!--                <a class="wrap-img" href="" target="_blank">-->
<!--                  <img class="img-blur-done" src="/static/image/10907624-107943365323e5b9.jpeg" />-->
<!--                </a>-->
<!--                <div class="content">-->
<!--                  <a class="title" target="_blank" href="">“不耻下问”，正在毁掉你的人生</a>-->
<!--                  <p class="abstract">-->
<!--                    在过去，遇到不懂的问题，你不耻下问，找个人问问就行；在现在，如果你还这么干，多半会被认为是“搜商低”。 昨天，35岁的表姐把我拉黑了。 表姐是医...-->
<!--                  </p>-->
<!--                  <div class="meta">-->
<!--                    <span class="jsd-meta">-->
<!--                      <img src="/static/image/paid1.svg" alt=""> 6.7-->
<!--                    </span>-->
<!--                    <a class="nickname" target="_blank" href="">_飞鱼</a>-->
<!--                    <a target="_blank" href="">-->
<!--                      <img src="/static/image/comment.svg" alt=""> 33-->
<!--                    </a>-->
<!--                    <span><img src="/static/image/like.svg" alt=""> 113</span>-->
<!--                    <span><img src="/static/image/shang.svg" alt=""> 2</span>-->
<!--                  </div>-->
<!--                </div>-->
<!--              </li>-->
            </ul>
            <!-- 文章列表模块 -->
          </div>
        <a href="" class="load-more">阅读更多</a></div>
        <div class="aside">
          <!-- 推荐作者 -->
          <div class="recommended-author-wrap">
            <!---->
            <div class="recommended-authors">
              <div class="title">
                <span>推荐作者</span>
                <a class="page-change"><img class="icon-change" src="/static/image/exchange-rate.svg" alt="">换一批</a>
              </div>
              <ul class="list">
                <li>
                  <a href="" target="_blank" class="avatar">
                    <img src="/static/image/avatar.webp" />
                  </a>
                  <a class="follow" state="0"><img src="/static/image/follow.svg" alt="" />关注</a>
                  <a href="" target="_blank" class="name">董克平日记</a>
                  <p>写了807.1k字 · 2.5k喜欢</p>
                </li>
                <li>
                  <a href="" target="_blank" class="avatar">
                    <img src="/static/image/avatar.webp" />
                  </a>
                  <a class="follow" state="0"><img src="/static/image/follow.svg" alt="" />关注</a>
                  <a href="" target="_blank" class="name">董克平日记</a>
                  <p>写了807.1k字 · 2.5k喜欢</p>
                </li>

              </ul>
              <a href="" target="_blank" class="find-more">查看全部 ></a>
              <!---->
            </div>
          </div>
        </div>
      </div>
    </div>
    <Footer></Footer>
  </div>
</template>
<script>
  import Header from "./common/Header";
  import Footer from "./common/Footer";
  export default {
      name:"Home",
      data(){
          return {
            banner_list: [],
            article_list:[
                {
                    user:{}
                },
            ],
          }
      },
      created(){
        this.get_banner();
        this.get_article();
      },
      filters:{
          format(content){
              if(content==null){
                  return ""
              }
              content = String(content);
              let content2 = ""
              while(true){
                  content2 = content.replace(/<.*?>/,"")
                  if(content2==content){
                      break
                  }else{
                      content = content2
                  }
              }
              return content
          }
      },
      methods:{
        get_banner(){
            this.$axios.get(`${this.$settings.Host}/banner/`).then(response=>{
              this.banner_list = response.data;
            }).catch(error=>{
              this.$message.error("网络异常,无法获取轮播图内容!");
            });
        },
        get_article(){
            // 获取文章列表
            this.$axios.get(`${this.$settings.Host}/push/article/`).then(response=>{
              this.article_list = response.data.results;
            }).catch(error=>{
              this.$message.error("网络异常,无法获取文章列表内容!");
            });
        }
      },
      components:{
        Header,
        Footer,
      }
  }
</script>
```



### 用户点击查看更多显示文章

使用函数节流, 在用户频繁点击ajax按钮时, 阻止点击

```vue
<template>
  <div id="home">
    <Header></Header>
    <div class="container">
      <div class="row">
        <div class="main">
          <!-- Banner -->
          <div class="banner">
            <el-carousel height="272px" indicator-position="none" :interval="2000">
              <el-carousel-item v-for="item,key in banner_list" :key="key">
                <a :href="item.link" v-if="item.is_http"><img :src="item.image" alt=""></a>
                <router-link :to="item.link" v-else><img :src="item.image" alt=""></router-link>
              </el-carousel-item>
            </el-carousel>
          </div>
          <div id="list-container">
            <!-- 文章列表模块 -->
            <ul class="note-list">
              <li class="" v-for="article in article_list">
                <div class="content">
                  <router-link class="title" :to="`/article/${article.id}`">{{article.title}}</router-link>
                  <p class="abstract">{{article.html_content|format}}</p>
                  <div class="meta">
                    <router-link class="nickname" :to="`/user/${article.user.id}`">{{article.user.nickname?article.user.nickname:article.user.username}}</router-link>
                    <router-link :to="`/article/${article.id}#comment`">
                      <img src="/static/image/comment.svg" alt=""> {{article.comment_count}}
                    </router-link>
                    <span v-if="article.like_count>0"><img src="/static/image/like.svg" alt=""> {{article.like_count}}</span>
                    <span v-if="article.reward_count>0"><img src="/static/image/shang.svg" alt=""> {{article.reward_count}}</span>
                  </div>
                </div>
              </li>
<!--              <li class="have-img">-->
<!--                <a class="wrap-img" href="" target="_blank">-->
<!--                  <img class="img-blur-done" src="/static/image/10907624-107943365323e5b9.jpeg" />-->
<!--                </a>-->
<!--                <div class="content">-->
<!--                  <a class="title" target="_blank" href="">“不耻下问”，正在毁掉你的人生</a>-->
<!--                  <p class="abstract">-->
<!--                    在过去，遇到不懂的问题，你不耻下问，找个人问问就行；在现在，如果你还这么干，多半会被认为是“搜商低”。 昨天，35岁的表姐把我拉黑了。 表姐是医...-->
<!--                  </p>-->
<!--                  <div class="meta">-->
<!--                    <span class="jsd-meta">-->
<!--                      <img src="/static/image/paid1.svg" alt=""> 6.7-->
<!--                    </span>-->
<!--                    <a class="nickname" target="_blank" href="">_飞鱼</a>-->
<!--                    <a target="_blank" href="">-->
<!--                      <img src="/static/image/comment.svg" alt=""> 33-->
<!--                    </a>-->
<!--                    <span><img src="/static/image/like.svg" alt=""> 113</span>-->
<!--                    <span><img src="/static/image/shang.svg" alt=""> 2</span>-->
<!--                  </div>-->
<!--                </div>-->
<!--              </li>-->
            </ul>
            <!-- 文章列表模块 -->
          </div>
        <a href="" class="load-more" v-if="next_page" @click.prevent.stop="get_article">阅读更多</a></div>
        <div class="aside">
          <!-- 推荐作者 -->
          <div class="recommended-author-wrap">
            <!---->
            <div class="recommended-authors">
              <div class="title">
                <span>推荐作者</span>
                <a class="page-change"><img class="icon-change" src="/static/image/exchange-rate.svg" alt="">换一批</a>
              </div>
              <ul class="list">
                <li>
                  <a href="" target="_blank" class="avatar">
                    <img src="/static/image/avatar.webp" />
                  </a>
                  <a class="follow" state="0"><img src="/static/image/follow.svg" alt="" />关注</a>
                  <a href="" target="_blank" class="name">董克平日记</a>
                  <p>写了807.1k字 · 2.5k喜欢</p>
                </li>
                <li>
                  <a href="" target="_blank" class="avatar">
                    <img src="/static/image/avatar.webp" />
                  </a>
                  <a class="follow" state="0"><img src="/static/image/follow.svg" alt="" />关注</a>
                  <a href="" target="_blank" class="name">董克平日记</a>
                  <p>写了807.1k字 · 2.5k喜欢</p>
                </li>

              </ul>
              <a href="" target="_blank" class="find-more">查看全部 ></a>
            </div>
          </div>
        </div>
      </div>
    </div>
    <Footer></Footer>
  </div>
</template>
<script>
  import Header from "./common/Header";
  import Footer from "./common/Footer";
  export default {
      name:"Home",
      data(){
          return {
            banner_list: [],
            next_page: null, // 控制显示下一页按钮
            is_send_ajax: false, // 锁，用于实现ajax节流
            article_list:[],
          }
      },
      created(){
        this.get_banner();
        // 初始化文章列表的url地址
        this.next_page = `${this.$settings.Host}/push/article/?size=3`;
        this.get_article();
      },
      filters:{
          format(content){
              if(content==null){
                  return ""
              }
              content = String(content);
              let content2 = "";
              while(true){
                  content2 = content.replace(/<.*?>/,"")
                  if(content2==content){
                      break
                  }else{
                      content = content2
                  }
              }
              return content
          }
      },
      methods:{
        get_banner(){
            this.$axios.get(`${this.$settings.Host}/banner/`).then(response=>{
              this.banner_list = response.data;
            }).catch(error=>{
              this.$message.error("网络异常,无法获取轮播图内容!");
            });
        },
        get_article(){
            // 获取文章列表
            if(this.is_send_ajax){ // 判断锁的状态
                return;
            }
            // 上锁
            this.is_send_ajax=true;
            this.$axios.get(this.next_page).then(response=>{
              this.article_list = this.article_list.concat(response.data.results);
              this.next_page = response.data.next;
              // 解锁
              this.is_send_ajax=false;
            }).catch(error=>{
              this.$message.error("网络异常,无法获取文章列表内容!");
            });
        }
      },
      components:{
        Header,
        Footer,
      }
  }
</script>
```

上面我们已经实现了获取首页的文章列表并展示出来，但是这个功能目前并没有接入我们的Feed流推送。

```
1. 判断用户是否登录
   1.1. 用户登录了
        1.1.1 用户已经关注了其他作者
          1.1.1.1 用户关注作者中,有足够的内容展示给用户
          1.1.1.2 用户关注作者中,没有足够内容展示给用户
        1.1.2 用户没有关注任何的作者
          1.1.1.1 根据浏览历史查找内容进行热度推荐
   1.2. 用户未登录
        1.1.3 根据浏览历史查找内容进行热度推荐
```

因为用户查看首页时, 显示的文章是不能重复的,所以接下来我们需要在每次推送文章给用户的时候,必须要进行推送的保存工作。

### 实现用户和Feed内容的日志记录

用户和文章的推送日志

表格: user_message_log_table

| 分区键 | 第一主键 | (第二主键) | 属性列                                           |
| ------ | -------- | ---------- | ------------------------------------------------ |
| id     | user_id  | message_id | is_read_retrieve, is_like, is_reward, is_comment |

在home/management/commands/tablestore.py,自定义命令中,新增创建日志的命令:

```python
from django.core.management import BaseCommand
from renranapi.utils.tablestore import *
class Command(BaseCommand):
    help = """表格存储命令必须接收而且只接收1个命令参数，如下：
    python manage.py tablestore create  表示创建项目使用的表格 
    python manage.py tablestore delete  表示删除项目使用的表格
    """
    def __init__(self, *args, **kwargs):
        self.client = OTS()
        super().__init__(*args, **kwargs)


    def add_arguments(self, parser):
        """参数设置"""
        parser.add_argument("argument",nargs="*", help="操作类型") # 位置参数

    def handle(self, *args, **options):
        """
        表格存储的初始化
        :param args:
        :param options:  参数列表
        :return:
        """
        argument = options.get("argument")
        if len(argument) != 1:
            self.stderr.write("操作有误！")
            self.stdout.write(self.help)
            return None

        if argument[0] == "create":
            """创建表格"""
            self.create_table()

        elif argument[0] == "delete":
            """删除表格"""
            self.delete_table()
        else:
            self.stderr.write("操作有误！")
            self.stdout.write(self.help)

    def create_table(self):
        # 存储库[收件箱]
        table_name = "user_message_table"
        schema_of_primary_key = [  # 主键列
            ('id','INTEGER'),      # 分区键
            ('user_id', 'INTEGER'),
            ('sequence_id', 'INTEGER', PK_AUTO_INCR),
            ("message_id", 'INTEGER')
        ]
        self.stdout.write("表格《%s》创建开始..." % table_name)
        self.client.create_table(table_name, schema_of_primary_key)
        self.stdout.write("表格《%s》创建完成..." % table_name)

        # 关系库
        table_name = "user_relation_table"
        schema_of_primary_key = [ # 主键列
            ('id', 'INTEGER'),    # 分区键
            ('user_id', 'INTEGER'),
            ("follow_user_id", 'INTEGER'),
        ]

        self.stdout.write("表格《%s》创建开始..." % table_name)
        self.client.create_table(table_name, schema_of_primary_key)
        self.stdout.write("表格《%s》创建完成..." % table_name)

        # 未读池
        table_name = "user_message_session_table"
        # 主键列
        schema_of_primary_key = [
            ('id', 'INTEGER'),
            ('follow_user_id', 'INTEGER'),
            ("last_sequence_id", 'INTEGER'),
        ]

        self.client.create_table(table_name, schema_of_primary_key)
        self.stdout.write("表格《%s》创建完成..." % table_name)

        # 推送日志
        table_name = "user_message_log_table"
        schema_of_primary_key = [  # 主键列
            ('id', 'INTEGER'),
            ('user_id', 'INTEGER'),
            ("message_id", 'INTEGER'),
        ]

        self.client.create_table(table_name, schema_of_primary_key)
        self.stdout.write("表格《%s》创建完成..." % table_name)

    def delete_table(self):
        """删除表操作"""
        table_list = OTS().list_table()
        for table_name in table_list:
            self.client.delete_table(table_name)
            self.stdout.write("表格《%s》删除完成..." % table_name)

        self.stdout.write("所有表格删除完成...")
```

重置下tablestore的数据表，终端执行命令：

```bash
python manage.py tablestore delete
python manage.py tablestore create
```



### 提供用户登录状态下，首页查询推送内容

```python
from article.models import Article
from .serializers import ArticleModelSerializer
from .paginations import HomeArticlePageNumberPagination
from users.models import User
from renranapi.utils.tablestore import *
from renranapi.settings import constants
class ArticleListAPIView(ListAPIView):
    serializer_class = ArticleModelSerializer
    pagination_class = HomeArticlePageNumberPagination

    def get_queryset(self):
        user = self.request.user



        if isinstance(self.request.user,User):
            """登录"""
            start_key = {"id": constants.MESSAGE_TABLE_ID, "user_id":user.id, "sequence_id": INF_MIN, "message_id": INF_MIN}
            end_key   = {"id": constants.MESSAGE_TABLE_ID, "user_id":user.id, "sequence_id": INF_MAX, "message_id": INF_MAX}
            cond = SingleColumnCondition("is_cancel", False, ComparatorType.EQUAL)
            message_list = []
            # 接受客户端执行返回的单页数据量，如果客户端没有指定，则默认采用分页器的单页数据量
            size = int(self.request.query_params.get("size")) or self.pagination_class.page_size
            ret = OTS().get_list("user_message_table",start_key,end_key,limit=size, cond=cond)
            if ret:
                for item in ret["data"]:
                    message_list.append(item["message_id"])
                while ret["token"]:
                    # print(ret["token"]) # [('id', 1), ('user_id', 2), ('sequence_id', 1596081490522997), ('message_id', 23)]
                    start_key = ret["token"]
                    end_key = {"id": constants.MESSAGE_TABLE_ID, "user_id": user.id, "sequence_id": INF_MAX,
                               "message_id": INF_MAX}
                    cond = SingleColumnCondition("is_cancel", False, ComparatorType.EQUAL)
                    ret = OTS().get_list("user_message_table", start_key, end_key, limit=size, cond=cond)
                    for item in ret["data"]:
                        message_list.append(item["message_id"])

            print(message_list)
            queryset = Article.objects.filter(is_public=True, is_show=True, is_deleted=False, pk__in=message_list).order_by("-id")
            print(queryset)
        else:
            queryset = Article.objects.filter(is_public=True, is_show=True, is_deleted=False).order_by("-reward_count", "-comment_count", "-like_count", "-id")

        return queryset
```

### 在内容推送以后，记录推送状态到同步库中

```python
from article.models import Article
from .serializers import ArticleModelSerializer
from .paginations import HomeArticlePageNumberPagination
from users.models import User
from renranapi.utils.tablestore import *
from renranapi.settings import constants
class ArticleListAPIView(ListAPIView):
    serializer_class = ArticleModelSerializer
    pagination_class = HomeArticlePageNumberPagination

    def get_queryset(self):
        user = self.request.user
        if isinstance(self.request.user,User):
            """登录"""
            start_key = {"id": constants.MESSAGE_TABLE_ID, "user_id":user.id, "sequence_id": INF_MIN, "message_id": INF_MIN}
            end_key   = {"id": constants.MESSAGE_TABLE_ID, "user_id":user.id, "sequence_id": INF_MAX, "message_id": INF_MAX}
            cond = CompositeColumnCondition(LogicalOperator.AND)
            cond.add_sub_condition(SingleColumnCondition("is_cancel", False, ComparatorType.EQUAL))
            cond.add_sub_condition(SingleColumnCondition("is_read", False, ComparatorType.EQUAL))
            message_list = []
            primary_list = []
            # 接受客户端执行返回的单页数据量，如果客户端没有指定，则默认采用分页器的单页数据量
            size = int(self.request.query_params.get("size")) or self.pagination_class.page_size
            client = OTS()
            ret = client.get_list("user_message_table",start_key,end_key,limit=size, cond=cond)

            if ret["status"]:
                for item in ret["data"]:
                    message_list.append(item["message_id"])
                    primary_list.append(item)
                while ret["token"]:
                    # print(ret["token"]) # [('id', 1), ('user_id', 2), ('sequence_id', 1596081490522997), ('message_id', 23)]
                    start_key = ret["token"]
                    end_key = {"id": constants.MESSAGE_TABLE_ID, "user_id": user.id, "sequence_id": INF_MAX,
                               "message_id": INF_MAX}
                    ret = client.get_list("user_message_table", start_key, end_key, limit=size, cond=cond)
                    for item in ret["data"]:
                        message_list.append(item["message_id"])
                        primary_list.append(item)

                queryset = Article.objects.filter(is_public=True, is_show=True, is_deleted=False, pk__in=message_list).order_by("-id")

                # 记录推送状态到同步库中
                page = self.request.query_params.get("page")
                if page is None:
                    page = 1
                page = int(page)

                update_primary_list = []
                attribute_columns_list = []
                article_list = queryset[(page-1)*size:page*size]
                for article in article_list:
                    for data in primary_list:
                        if data["user_id"] == user.id and data["message_id"] == article.id:
                            update_primary_list.append(data)
                            attribute_columns_list.append({"is_read": True})

                client.update_list("user_message_table", update_primary_list,attribute_columns_list)

            if len(queryset) < 1:
                """进行智能推荐"""
                print("智能推荐")

        else:
            queryset = Article.objects.filter(is_public=True, is_show=True, is_deleted=False).order_by("-reward_count", "-comment_count", "-like_count", "-id")

        return queryset
```

接下来，我们需要针对同步库中没有推送数据的用户，进行智能推荐。目前市面上大部分资讯网站，或者Feed系统在关注数据干涸的情况，或者商城推荐商品给用户时，基本都是根据当前登录用户过往的行为特征进行推荐的。推荐的常用核心算法：协同过滤算法。

### 协同过滤

协同过滤有2种算法：1. 基于用户的协同过滤，2.基于物品的协同过滤。
所谓的协同过滤主要是计算用户或者物品之间的相似度，然后进行推荐。

#### 基于用户的协同过滤

所谓基于用户的协同过滤，就是根据当前推荐用户A的过往行为，找出与他相似度最高的用户B。把用户B阅读的文章或者购买的商品推荐给用户A。



![1596161428326](11Feed%E6%B5%81%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1.assets/1596161428326.png)



#### 基于物品的协同过滤

所谓基于物品的协同过滤，就是根据当前添加用户A的过往行为，列出用户A所有阅读的文章或购买的商品，查找出与用户A有购买同一商品或者阅读同一文章的用户所有信息，进行整体的推荐度计算累加。

![1596161563924](11Feed%E6%B5%81%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1.assets/1596161563924.png)



基于这种协同过滤算法，进行的推荐功能有以下特点：

```
1. 数据量越大，推荐的准确率就越高。反之，准确率就下降。
2. 因为用户的兴趣爱好，会随着时间而改变的。所以我们提取的行为记录，必须有限定的时间。
```

同时，上面的协同推荐只是用户与物品之间存在一种交互行为的基础而已。但是在实际工作很多产品中，其实用户和物品之间的行为是存在多种的。

例如：商城，用户会购买商品，会收藏商品。。

例如：荏苒网，用户会赞赏文章，会评论文章，会点赞文章或收藏文章，也可以阅读文章。

在这种情况下，用户每次与物品之间的交互都会产生不同推荐度或相似度。

所以，针对这种情况，我们一般会设置一个推荐度的权重值。例如：

| 行为 | 推荐值 |
| ---- | ------ |
| 打赏 | 10     |
| 评论 | 5      |
| 点赞 | 2      |
| 阅读 | 1      |

有了这种权重值的推荐度表以后，用户与物品之间的推荐度就会有所改变。就会产生新的推荐顺序，但是原来该推荐的物品还是原来的的物品。

![1596162633139](11Feed%E6%B5%81%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1.assets/1596162633139.png)



### 在文章详情中, 添加用户的行为记录

article/views.py，代码：

```python
from rest_framework.viewsets import ViewSet
from renranapi.utils.tablestore import *
from renranapi.settings import constants
from rest_framework.decorators import action
from django.utils import timezone as datetime
class UserArticleAPIView(ViewSet):
    permission_classes = [IsAuthenticated]
    """用户和文章之间的交互行为记录api"""
    client = OTS()

    def check_params(self):
        user = self.request.user
        pk = int(self.kwargs.get("pk"))
        try:
            article = Article.objects.get(pk=pk, is_deleted=False, is_show=True, is_public=True)
        except Article.DoesNotExist:
            return Response({"detail": "当前文章不存在!"}, status=status.HTTP_400_BAD_REQUEST)

        # 如果用户是作者，则不需要记录
        if article.user.id == user.id:
            return Response({"detail": "当前是作者。"}, status=status.HTTP_400_BAD_REQUEST)

        primary_key = {"id": constants.LOG_TABLE_ID, "user_id": user.id, "message_id": pk}
        ret = self.client.get_row("user_message_log_table", primary_key)

        return user,primary_key,ret

    @action(methods=["get"], detail=True)
    def read(self,request,pk):
        response = self.check_params()
        if isinstance(response,Response):
            return response
        user, primary_key, ret = response
        if ret[0] and len(ret[1])<1:
            attribute_columns = {
                "is_read": 1,
                "is_comment": 0,
                "is_reward": 0,
                "is_like": 0,
                "timestamp": datetime.now().timestamp(),
            }
            self.client.put_row("user_message_log_table", primary_key,attribute_columns)

        return Response("ok")

    @action(methods=["get"], detail=True)
    def status(self,request,pk):
        """添加评论，点赞等行为"""
        response = self.check_params()
        if isinstance(response,Response):
            return response
        user, primary_key, ret = response
        status_type = request.query_params.get("type")
        if status_type not in ("is_comment","is_like"):
            return Response({"detail": "参数有误，无法添加行为记录"}, status=status.HTTP_400_BAD_REQUEST)
        if ret[0] and len(ret[1]) > 0:
            attribute_columns = {
                status_type: 1,
                "timestamp": datetime.now().timestamp(),
            }
            self.client.update_row("user_message_log_table", primary_key, attribute_columns)
        else:
            return Response({"detail":"当前用户没有阅读文章的记录，无法添加行为记录"}, status=status.HTTP_400_BAD_REQUEST)

        return Response({"detail":"ok"})
```

renranapi/settings/constants.py代码：

```python
LOG_TABLE_ID = 1 # 行为日志的分区键
```

article/urls.py，路由代码：

```python
from rest_framework.routers import SimpleRouter
router = SimpleRouter()
router.register("log",views.UserArticleAPIView,basename="log")
urlpatterns+=router.urls
```

针对tablestore查询数据返回结果的代码调整，renranapi/utils/tablestore.py，代码：

```python
from tablestore import *
from django.conf import settings

class OTS(object):
    """工具类： 表格存储"""
    def __init__(self):
        self.client = OTSClient(settings.OTS_ENDPOINT, settings.OTS_ID, settings.OTS_SECRET, settings.OTS_INSTANCE)

    def list_table(self):
        """列出所有的存储表"""
        return self.client.list_table()

    def create_table(self, table_name, schema_of_primary_key):
        """
        :param table_name: 字符串，表名
        :param schema_of_primary_key: 列表，每一个成员是一个元祖，表示字段的名称和类型
        :return:
        """
        # 通过表名和主键列的schema创建一个tableMeta。
        table_meta = TableMeta(table_name, schema_of_primary_key)
        # 创建TableOptions，数据保留31536000秒，超过后自动删除；最大3个版本；写入时指定的版本值和当前标准时间相差不能超过1天。
        table_options = TableOptions(31536000, 3, 86400)
        # 设置预留读吞吐量为0，预留写吞吐量为0。
        reserved_throughput = ReservedThroughput(CapacityUnit(0, 0))

        try:
            self.client.create_table(table_meta, table_options, reserved_throughput)
            return True
        # 处理异常。
        except Exception:
            return False

    def delete_table(self, table_name):
        """
        删除表
        :param table_name: 参数如果是字符串，则表示删除一张表，
                           参数如果是列表，则表示删除多张表
        :return:
        """
        tables = []
        if type(table_name) is str:
            tables.append(table_name)
        else:
            tables = table_name

        ret = {
            "success": [],
            "fails": []
        }
        for table in tables:
            try:
                self.client.delete_table(table)
                ret["success"].append(table)
            except Exception:
                ret["fails"].append(table)

        return ret

    def put_row(self, table_name, primary_key, attribute_columns={}):
        """
        添加一条数据
        :param table_name: 本次添加数据的表名
        :param primary_key: 主键列
        :param attribute_columns: 其他属性列
        :return: 新增数据的主键
        """

        primary_key_list = []
        for key, value in primary_key.items():
            primary_key_list.append((key, value))

        attribute_columns_list = []
        for key, value in attribute_columns.items():
            attribute_columns_list.append((key, value))

        row = Row(primary_key_list, attribute_columns_list)
        # 添加数据的条件
        # EXPECT_NOT_EXIST 如果主键重复，则报错！
        condition = Condition(RowExistenceExpectation.EXPECT_NOT_EXIST)
        try:
            # 调用put_row方法, ReturnType.RT_PK表示返回新增数据的主键信息，如果不设置则返回None
            consumed, return_row = self.client.put_row(table_name, row, condition, ReturnType.RT_PK)
            return True, return_row.primary_key
        except OTSClientError as e:
            """网络异常"""
            return False, "put row failed, http_status:%d, error_message:%s" % (
            e.get_http_status(), e.get_error_message())

        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            """参数有误"""
            return False, "put row failed, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            """其他错误"""
            return False, "未知的异常"

    def get_row(self, table_name, primary_key, columns_to_get=[], column_filter=None):
        """"""
        # 指定查询结果返回的属性列
        # 过滤条件
        """
        # ====================== 多条件 ========================
        # 逻辑条件
        cond = CompositeColumnCondition(LogicalOperator.AND)
        cond = CompositeColumnCondition(LogicalOperator.OR)
        cond = CompositeColumnCondition(LogicalOperator.NOT)
        # 比较条件
        ComparatorType.NOT_EQUAL  !=
        ComparatorType.EQUAL      ==
        GREATER_THAN              >
        GREATER_EQUAL             >=
        LESS_THAN                 <
        LESS_EQUAL                <=
        举例：
          查询一个学生信息，性别为男的，小于20岁===>   (sex=男 and age < 20) or (sex=女 and age < 17) 
            cond = CompositeColumnCondition(LogicalOperator.AND)
            cond.add_sub_condition(SingleColumnCondition("sex", '男', ComparatorType.EQUAL))
            cond.add_sub_condition(SingleColumnCondition("age", 20, ComparatorType.LESS_THAN))

        # ====================== 单条件 ======================== 
        cond = SingleColumnCondition("age", 20, ComparatorType.LESS_THAN)
        """
        primary_key_list = []
        for key, value in primary_key.items():
            primary_key_list.append((key, value))
        try:
            # 调用get_row接口查询
            consumed, return_row, next_token = self.client.get_row(table_name, primary_key_list, columns_to_get,
                                                                   column_filter, 1)
            print(return_row)
            data = {}
            if return_row is not None:
                for att in return_row.primary_key:
                    # 打印每一个字段的内容
                    data[att[0]] = att[1]
                if len(columns_to_get) > 0:
                    """如果有指定要返回其他属性列"""
                    for att in return_row.attribute_columns:
                        # 打印每一个字段的内容
                        data[att[0]] = att[1]
            return True, data
            # 客户端异常，一般为参数错误或者网络异常。
        except OTSClientError as e:
            return False, "网络异常，获取数据失败, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            return False, "参数有误，获取数据失败, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False, "未知异常, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())

    def update_row(self, table_name, primary_key, attribute_columns):
        """更新一条数据"""
        primary_key_list = []
        for key, value in primary_key.items():
            primary_key_list.append((key, value))

        attribute_columns_list = []
        for key, value in attribute_columns.items():
            attribute_columns_list.append((key, value))

        update_of_attribute_columns = {
            'PUT': attribute_columns_list,
        }

        row = Row(primary_key_list, update_of_attribute_columns)

        try:
            consumed, return_row = self.client.update_row(table_name, row, condition=None, return_type=ReturnType.RT_PK)
            data = {}
            for att in return_row.primary_key:
                # 打印每一个字段的内容
                data[att[0]] = att[1]

            return True, data
        # 客户端异常，一般为参数错误或者网络异常。
        except OTSClientError as e:
            return False, "更新失败, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())

        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            return False, "更新失败, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False, "未知异常"

    def delete_row(self, table_name, primary_key):
        """根据主键删除一条数据"""
        primary_key_list = []
        for key, value in primary_key.items():
            primary_key_list.append((key, value))
        row = Row(primary_key_list)
        try:
            consumed, return_row = self.client.delete_row(table_name, row, None)
            return True, "删除成功"
        except OTSClientError as e:
            return False, "更新失败！网络异常, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        except OTSServiceError as e:
            return False, "更新失败，参数有误, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False, "未知异常"

    def get_list_0(self, table_name, primary_key_list, columns_to_get=[], column_filter=None):
        """
        根据指定主键查询多条数据
        :param table: 字符串，表名
        :param primary_key_list: 主键列表
        :param columns_to_get: 返回属性字段列表
        :param column_filter: 条件
        :return: 列表， 查询结果
        """

        # 读取3行。
        rows_to_get = []
        for item in primary_key_list:
            primary_key = []
            for key, value in item.items():
                primary_key.append((key, value))
            rows_to_get.append(primary_key)

        # 构造批量读请求。
        request = BatchGetRowRequest()

        # 增加表table_name中需要读取的行，最后一个参数1表示读取最新的一个版本。
        request.add(TableInBatchGetRowItem(table_name, rows_to_get, columns_to_get, column_filter, 1))

        try:
            result = self.client.batch_get_row(request)
            if result.is_all_succeed():
                """只有在获取到全部数据以后才表示读取多条数据成功"""
                table_result = result.get_result_by_table(table_name)
                data = []
                for item in table_result:
                    row = {}

                    if item.is_ok:
                        for att in item.row.primary_key:
                            # 打印每一个字段的内容
                            row[att[0]] = att[1]

                        for att in item.row.attribute_columns:
                            # 打印每一个字段的内容
                            row[att[0]] = att[1]

                    else:
                        return False, '部分数据参数有误，读取数据失败。 error code: %s, error message: %s' % (
                        item.error_code, item.error_message)

                    data.append(row)

                return True, data

            else:
                return False, '部分数据参数有误，读取数据失败。'

        # 客户端异常，一般为参数错误或者网络异常。
        except OTSClientError as e:
            return False, "读取数据失败，网络异常。 http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            return False, "读取数据失败，参数有误。 http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
            e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False, "未知异常"

    def add_list(self, table_name, primary_key_list, attribute_columns_list=[]):
        """
        添加多条数据
        :param table_name: 字符串，表名
        :param primary_key_list: 主键列
        :param attribute_columns_list: 属性列
        :return:
        """
        put_row_items = self.row_items(PutRowItem, primary_key_list, attribute_columns_list)
        status, result = self.request(table_name, put_row_items)
        if status == False:
            return False, result
        # 输出每一行添加数据的结果。
        succ, fail = result.get_put()
        for item in succ:
            print('添加数据成功, consume %s write cu.' % item.consumed.write)
        for item in fail:
            print('添加数据失败, error code: %s, error message: %s' % (item.error_code, item.error_message))

        return True

    def update_list(self, table_name, primary_key_list, attribute_columns_list=[]):
        """更新多条数据"""
        update_row_items = self.row_items(UpdateRowItem, primary_key_list, attribute_columns_list)
        status, result = self.request(table_name, update_row_items)
        if status == False:
            return False, result

        # 输出每一行更新数据的结果
        succ, fail = result.get_update()
        for item in succ:
            print('更新数据成功, consume %s write cu.' % item.consumed.write)
        for item in fail:
            print('更新数据失败, error code: %s, error message: %s' % (item.error_code, item.error_message))

        return True

    def delete_list(self, table_name, primary_key_list):
        """删除多条数据"""
        delete_row_items = self.row_items(DeleteRowItem, primary_key_list)
        status, result = self.request(table_name, delete_row_items)
        if status == False:
            return False, result
        # 输出每一行删除数据的结果。
        succ, fail = result.get_delete()
        for item in succ:
            print('删除数据成功, consume %s write cu.' % item.consumed.write)
        for item in fail:
            print('删除数据失败, error code: %s, error message: %s' % (item.error_code, item.error_message))

        return True

    def row_items(self, cls, primary_key_list, attribute_columns_list=[]):

        if len(primary_key_list) != len(attribute_columns_list) and len(attribute_columns_list) != 0:
            return False, "参数有误！主键和属性列数量不对应，无法完成添加操作"

        # 增加多行组件的组装。
        row_items = []
        for key0, primary_item in enumerate(primary_key_list):
            primary_row = []
            for key1, value in primary_item.items():
                primary_row.append((key1, value))

            attribute_row = None
            if len(attribute_columns_list) > 0:
                if cls == PutRowItem:
                    """添加多条"""
                    attribute_row = []
                    for key2, value in attribute_columns_list[key0].items():
                        attribute_row.append((key2, value))
                elif cls == UpdateRowItem:
                    """更新多条"""
                    attribute_row = {"put": []}
                    for key2, value in attribute_columns_list[key0].items():
                        attribute_row["put"].append((key2, value))

            row = Row(primary_row, attribute_row)
            condition = Condition(RowExistenceExpectation.IGNORE)
            item = cls(row, condition)
            row_items.append(item)

        return row_items

    def request(self, table_name, put_row_items):
        """
        构造批量写请求
        :param table_name:
        :param put_row_items:
        :return:
        """
        request = BatchWriteRowRequest()
        request.add(TableInBatchWriteRowItem(table_name, put_row_items))
        try:
            result = self.client.batch_write_row(request)
            if result.is_all_succeed():
                return True, result

        except OTSClientError as e:
            print(e.get_http_status(), e.get_error_message())
            return False, "网络异常, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        except OTSServiceError as e:
            return False, "参数有误, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False, "未知异常"

    def get_list(self, table_name, start_key, end_key, columns_to_get=[], limit=90, cond=None,
                 derection=Direction.FORWARD):
        """
        根据指定范围查询数据
        :param table_name: 字符串，表名
        :param start_key: 字典，查询的开始主键位置
        :param end_key: 字典，查询的结束主键位置
        :param columns_to_get: 列表，属性列
        :param limit: 整型, 查询结果数量
        :param cond: Cond类对象，查询条件
        :param derection: 查询的排列方式，Direction.FORWARD 正序，Direction.BACKWARD 倒序
        :return:
        """
        # 设置范围查询的起始主键。
        inclusive_start_primary_key = []
        if type(start_key) is dict:
            for key, value in start_key.items():
                inclusive_start_primary_key.append((key, value))
        else:
            inclusive_start_primary_key=start_key

        # 设置范围查询的结束主键。
        exclusive_end_primary_key = []
        if type(end_key) is dict:
            for key, value in end_key.items():
                exclusive_end_primary_key.append((key, value))
        else:
            exclusive_end_primary_key=end_key

        try:
            # 读取数据
            consumed, next_start_primary_key, row_list, next_token = self.client.get_range(
                table_name, derection,
                inclusive_start_primary_key, exclusive_end_primary_key,
                columns_to_get,
                limit,
                column_filter=cond,
                max_version=1,
            )

            all_rows = []
            all_rows.extend(row_list)

            # 打印主键和属性列。
            data = []
            for row in all_rows:
                data_item = {}
                for att in row.primary_key:
                    data_item[att[0]] = att[1]
                if len(columns_to_get) > 0:
                    """如果有指定要返回其他属性列"""
                    for att in row.attribute_columns:
                        data_item[att[0]] = att[1]
                data.append(data_item)

            return {"status": True, "data": data, "token": next_start_primary_key}
        # 客户端异常，一般为参数错误或者网络异常。
        except OTSClientError as e:
            return False, "网络异常, http_status:%d, error_message:%s" % (e.get_http_status(), e.get_error_message())
        # 服务端异常，一般为参数错误或者流控错误。
        except OTSServiceError as e:
            return False, "参数有误, http_status:%d, error_code:%s, error_message:%s, request_id:%s" % (
                e.get_http_status(), e.get_error_code(), e.get_error_message(), e.get_request_id())
        except Exception as e:
            return False, "未知异常"
```



### 客户端发送请求，添加行为记录

```vue
<template>
  <div class="_21bLU4 _3kbg6I">
   <Header></Header>
   <div class="_3VRLsv" role="main">
    <div class="_gp-ck">
     <section class="ouvJEz">
      <h1 class="_1RuRku">{{article.title}}</h1>
      <div class="rEsl9f">
       <div class="_2mYfmT">
        <router-link class="_1OhGeD" to="/user" target="_blank" rel="noopener noreferrer"><img class="_13D2Eh" :src="article.user.avatar" alt="" /></router-link>
        <div style="margin-left: 8px;">
         <div class="_3U4Smb">
          <span class="FxYr8x"><router-link class="_1OhGeD" to="/user">{{article.user.nickname?article.user.nickname:article.user.username}}</router-link></span>
          <button data-locale="zh-CN" type="button" class="_3kba3h _1OyPqC _3Mi9q9 _34692-" v-if="focus_status==1" @click="focus_author(focus_status)"><span>已关注</span></button>
          <button data-locale="zh-CN" type="button" class="_3kba3h _1OyPqC _3Mi9q9 _34692-" v-if="focus_status==0" @click="focus_author(focus_status)"><span>关注</span></button>
          <button data-locale="zh-CN" type="button" class="_3kba3h _1OyPqC _3Mi9q9 _34692-" v-if="focus_status==-2" @click="focus_author(focus_status)"><span>关注</span></button>
         </div>
         <div class="s-dsoj">
          <time :datetime="format(article.created_time)">{{format(article.created_time)}}</time>
          <span>字数 {{article.html_content==null?0:article.html_content.length}}</span>
          <span>阅读 {{article.read_count}}</span>
         </div>
        </div>
       </div>
      </div>
      <article class="_2rhmJa">
       <div v-html="article.html_content"></div>
      </article>
      <div></div>
      <div class="_1kCBjS">
       <div class="_18vaTa">
        <div class="_3BUZPB">
         <div class="_2Bo4Th" role="button" tabindex="-1" aria-label="给文章点赞">
          <i aria-label="ic-like" class="anticon">
           <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
            <use xlink:href="#ic-like"></use>
           </svg></i>
         </div>
         <span class="_1LOh_5" role="button" tabindex="-1" aria-label="查看点赞列表">8人点赞<i aria-label="icon: right" class="anticon anticon-right">
           <svg viewbox="64 64 896 896" focusable="false" class="" data-icon="right" width="1em" height="1em" fill="currentColor" aria-hidden="true">
            <path d="M765.7 486.8L314.9 134.7A7.97 7.97 0 0 0 302 141v77.3c0 4.9 2.3 9.6 6.1 12.6l360 281.1-360 281.1c-3.9 3-6.1 7.7-6.1 12.6V883c0 6.7 7.7 10.4 12.9 6.3l450.8-352.1a31.96 31.96 0 0 0 0-50.4z"></path>
           </svg></i></span>
        </div>
        <div class="_3BUZPB">
         <div class="_2Bo4Th" role="button" tabindex="-1">
          <i aria-label="ic-dislike" class="anticon">
           <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
            <use xlink:href="#ic-dislike"></use>
           </svg></i>
         </div>
        </div>
       </div>
       <div class="_18vaTa">
        <a class="_3BUZPB _1x1ok9 _1OhGeD" href="/nb/38290018" target="_blank" rel="noopener noreferrer"><i aria-label="ic-notebook" class="anticon">
          <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
           <use xlink:href="#ic-notebook"></use>
          </svg></i><span>{{article.collection.name}}</span></a>
        <div class="_3BUZPB ant-dropdown-trigger">
         <div class="_2Bo4Th">
          <i aria-label="ic-others" class="anticon">
           <svg width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
            <use xlink:href="#ic-others"></use>
           </svg></i>
         </div>
        </div>
       </div>
      </div>
      <div class="_19DgIp" style="margin-top:24px;margin-bottom:24px"></div>
      <div class="_13lIbp">
       <div class="_191KSt">
        &quot;小礼物走一走，来简书关注我&quot;
       </div>
       <button type="button" class="_1OyPqC _3Mi9q9 _2WY0RL _1YbC5u" @click="is_show_reward_window=true"><span>赞赏支持</span></button>
       <span class="_3zdmIj" v-if="article.reward_count==0">还没有人赞赏，支持一下</span>
       <span class="_3zdmIj" v-else>共{{article.reward_count}}人赞赏了作者</span>
      </div>
      <div class="d0hShY">
       <a class="_1OhGeD" href="/u/a70487cda447" target="_blank" rel="noopener noreferrer"><img class="_27NmgV" src="https://upload.jianshu.io/users/upload_avatars/18529254/.png?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp" alt="  " /></a>
       <div class="Uz-vZq">
        <div class="Cqpr1X">
         <a class="HC3FFO _1OhGeD" href="/u/a70487cda447" title="書酱" target="_blank" rel="noopener noreferrer">書酱</a>
         <span class="_2WEj6j" title="你读书的样子真好看。">你读书的样子真好看。</span>
        </div>
        <div class="lJvI3S">
         <span>总资产0</span>
         <span>共写了78.7W字</span>
         <span>获得6,072个赞</span>
         <span>共1,308个粉丝</span>
        </div>
       </div>
       <button data-locale="zh-CN" type="button" class="_1OyPqC _3Mi9q9"><span>关注</span></button>
      </div>
     </section>
     <div id="note-page-comment">
      <div class="lazyload-placeholder"></div>
     </div>
    </div>
    <aside class="_2OwGUo">
     <section class="_3Z3nHf">
      <div class="_3Oo-T1">
       <a class="_1OhGeD" href="/u/a70487cda447" target="_blank" rel="noopener noreferrer"><img class="_3T9iJQ" src="https://upload.jianshu.io/users/upload_avatars/18529254/.png?imageMogr2/auto-orient/strip|imageView2/1/w/90/h/90/format/webp" alt="" /></a>
       <div class="_32ZTTG">
        <div class="_2O0T_w">
         <div class="_2v-h3G">
          <span class="_2vh4fr" title="書酱"><a class="_1OhGeD" href="/u/a70487cda447" target="_blank" rel="noopener noreferrer">書酱</a></span>
         </div>
         <button data-locale="zh-CN" type="button" class="tzrf9N _1OyPqC _3Mi9q9 _34692-"><span>关注</span></button>
        </div>
        <div class="_1pXc22">
         总资产0
        </div>
       </div>
      </div>
      <div class="_19DgIp"></div>
     </section>
     <div>
      <div class="">
       <section class="_3Z3nHf">
        <h3 class="QHRnq8 QxT4hD"><span>推荐阅读</span></h3>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="这些话没人告诉你，但必须知道的社会规则">
          <a class="_1-HJSV _1OhGeD" href="/p/a3e56a0559ff" target="_blank" rel="noopener noreferrer">这些话没人告诉你，但必须知道的社会规则</a>
         </div>
         <div class="_19haGh">
          阅读 5,837
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="浙大学霸最美笔记曝光：真正的牛人，都“变态”到了极致">
          <a class="_1-HJSV _1OhGeD" href="/p/d2a3724e2839" target="_blank" rel="noopener noreferrer">浙大学霸最美笔记曝光：真正的牛人，都“变态”到了极致</a>
         </div>
         <div class="_19haGh">
          阅读 12,447
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="征服一个女人最好的方式：不是讨好她，而是懂得去折腾她">
          <a class="_1-HJSV _1OhGeD" href="/p/f6acf67f039b" target="_blank" rel="noopener noreferrer">征服一个女人最好的方式：不是讨好她，而是懂得去折腾她</a>
         </div>
         <div class="_19haGh">
          阅读 5,311
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="告别平庸的15个小方法">
          <a class="_1-HJSV _1OhGeD" href="/p/cff7eb6b232b" target="_blank" rel="noopener noreferrer">告别平庸的15个小方法</a>
         </div>
         <div class="_19haGh">
          阅读 7,040
         </div>
        </div>
        <div class="cuOxAY" role="listitem">
         <div class="_3L5YSq" title="轻微抑郁的人，会说这3句“口头禅”，若你一个不占，偷着乐吧">
          <a class="_1-HJSV _1OhGeD" href="/p/2a0ca1729b4b" target="_blank" rel="noopener noreferrer">轻微抑郁的人，会说这3句“口头禅”，若你一个不占，偷着乐吧</a>
         </div>
         <div class="_19haGh">
          阅读 16,411
         </div>
        </div>
       </section>
      </div>
     </div>
    </aside>
   </div>

   <div class="_23ISFX-body" v-if="is_show_reward_window" @click.stop="is_show_reward_window=true">
     <div class="_3uZ5OL">
      <div class="_2PLkjk">
       <img class="_2R1-48" :src="article.user.avatar" alt="" />
       <div class="_2h5tnQ">
        给作者送糖
       </div>
      </div>
      <div class="_1-bCJJ">
       <div class="LMa6S_" :class="reward_info.money==num?'_1vONvL':''" @click="reward_info.money=num" v-for="num in reward_list"><span>{{num}}元</span></div>
      </div>
      <textarea class="_1yN79W" placeholder="给Ta留言..." v-model="reward_info.content"></textarea>
      <div class="_1_B577">选择支付方式</div>
      <div class="_1-bCJJ">
       <div class="LMa6S_ _3PA8BN" :class="reward_info.pay_type==type?'_1vONvL':''" @click="reward_info.pay_type=type" v-for="type in pay_type_list"><span>{{type}}</span></div>
      </div>
      <button type="button" class="_3A-4KL _1OyPqC _3Mi9q9 _1YbC5u" @click="payhandler"><span>确认支付</span><span> ￥</span>{{reward_info.money}}</button>
     </div>
    </div>

   <Footer></Footer>
  </div>
</template>

<script>
    import Header from "./common/Header";
    import Footer from "./common/Footer";
    export default {
        name: "Article",
        components:{
          Header,
          Footer,
        },
        data(){
          return {
            token: "",
            article: {
                collection:{},
                user:{},
            },
            is_show_reward_window:false, // 是否显示打赏窗口
            reward_list:[2,5,10,20,50,100],  // 赞赏的金额
            pay_type_list: ["支付宝","余额支付"], // 赞赏的支付方式
            reward_info:{  // 赞赏的表单信息
                money: 2,
                content:"",
                pay_type:"支付宝",
            },
            focus_status: -2, // 当前用户是否关注作者的状态，-2表示没登录,-1表示用户与作者是同一个人，0表示没有关注，1表示已关注
          }
        },
        created(){
            // 获取路由参数
            this.article.id = this.$route.params.id;
            this.get_article();
        },
        methods:{
            payhandler(){
              this.$message.info("正在打赏处理中。。。请稍后！～");
              // 打赏作者的支付处理
              if(!this.token){
                  this.token = this.$settings.check_user_login(this);
              }

              if(this.token && this.reward_info.pay_type=="支付宝"){
                // 发起支付
                this.$axios.post(`${this.$settings.Host}/payments/alipay/`,{
                    money: this.reward_info.money,
                    message: this.reward_info.content,
                    article_id: this.article.id,
                },{
                    headers:{
                      Authorization: "jwt " + this.token
                    }
                }).then(response=>{
                    // location.href=response.data;
                    this.get_reward_status(response.data.out_trade_no);
                    open(response.data.url,"_blank");
                }).catch(error=>{
                    this.$message.error("网络异常，无法进行打赏！");
                })
              }
            },
            get_reward_status(out_trade_no){
              // 查询打赏支付的结果
              this.is_show_reward_window = false;
              let timer = setInterval(()=>{
                  this.$axios.get(`${this.$settings.Host}/payments/alipay/result/`,{
                      params:{
                          out_trade_no:out_trade_no,
                      }
                  }).then(response=>{
                     if(response.data.status){
                       this.$message.success("支付成功了！");
                       this.article.reward_count+=1;
                       clearInterval(timer);
                     }
                  }).catch(error=>{
                     this.$message.error("网络异常，无法查询打赏支付的结果!");
                     clearInterval(timer);
                  });
              },5000);
            },
            get_article(){
              // 获取文章内容信息
              this.$axios.get(`${this.$settings.Host}/article/${this.article.id}/retrieve/`).then(response=>{
                  this.article = response.data;
                  // 查询当前访问用户与作者的关系
                  this.get_focus_status();
              }).catch(error=>{
                  this.$message.error("网络异常，获取文章信息失败！");
              })
            },
            get_focus_status(){
              // 查询用户是否关注了作者
              if(!this.token){
                  this.token = this.$settings.check_user_login(this);
              }
              if(this.token == false){
                  // 如果token不存在，则表示没有登录
                  return;
              }
              this.$axios.get(`${this.$settings.Host}/article/focus/`,{
                  params:{
                      author_id: this.article.user.id
                  },
                  headers:{
                    Authorization: "jwt " + this.token
                  }
              }).then(response=>{
                  this.focus_status = response.data.status;
                  // 5s以后，记录用户的阅读行为
                  setTimeout(this.add_read_log,5000);
              }).catch(error=>{
                  this.$message.error(error.response.data.detail);
              });
            },
            focus_author(status){
                if(!this.token){
                  this.token = this.$settings.check_user_login(this);
                }
                if(this.token == false){
                    // 如果token不存在，则表示没有登录
                    return;
                }
                if(status == -2){
                    this.$confirm('您尚未登录，无法完成操作，是否登录继续？', '提示', {
                      confirmButtonText: '登录',
                      cancelButtonText: '取消',
                      type: 'warning'
                    }).then(() => {
                      this.$router.push("/login");
                    }).catch(() => {

                    });
                    return;
                }

                this.$axios.put(`${this.$settings.Host}/article/focus/`,{
                   author_id: this.article.user.id,
                   status: status,
                },{
                   headers:{
                      Authorization: "jwt " + this.token
                   }
                }).then(response=>{
                    this.focus_status = response.data.status;
                    this.$message.success(response.data.detail);
                }).catch(error=>{
                    this.$message.error(error.response.data.detail);
                });

            },
            add_read_log(){
                // 记录用户的阅读行为
                this.$axios.get(`${this.$settings.Host}/article/log/${this.article.id}/read/`,{
                  headers:{
                    Authorization: "jwt " + this.token
                  }
                });
            },
            format(time){
              time = new Date(time);
              let Y = time.getFullYear();
              let m = time.getMonth()+1;
              m = m<10?('0'+m):m;
              let d = time.getDate();
              d = d<10?('0'+d):d;
              let H = time.getHours();
              H = H<10?('0'+H):H;
              let M = time.getMinutes();
              M = M<10?('0'+M):M;
              let S = time.getSeconds();
              S = S<10?('0'+S):S;
              return `${Y}-${m}-${d} ${H}:${M}:${S}`;
            }
        }
    }
</script>
```

#### 增加打赏成功以后，记录用户的打赏行为

payments/views.py，视图代码：

```python
from renranapi.utils.tablestore import *
from renranapi.settings import constants
class AlipayResultAPIView(APIView):
    """支付结果的处理"""
    def get1(self,request):
        """同步回调结果的处理"""
        data = request.query_params.dict() # 把查询字符串转换成字典
        signature = data.pop("sign")

        alipay = self.get_alipay()

        # todo 验证支付的同步通知结果
        success = alipay.verify(data, signature)
        if True:
            # 完成支付结果的处理
            # 根据订单号提取订单信息
            out_trade_no = data.get("out_trade_no")
            try:
                reward = Reward.objects.get(out_trade_no=out_trade_no,status=False)
            except Reward.DoesNotExist:
                return Response({"detail":"当前打赏记录不存在或者已完成！"}, status=status.HTTP_400_BAD_REQUEST)

            # 根据订单的付款状态进行修改
            self.change_reward(data,reward)

            # 返回结果
            return Response({"detail":"支付处理成功！","article_id": reward.article.id})

        return Response({"detial":"支付结果参数有误！"}, status=status.HTTP_400_BAD_REQUEST)
    def get(self,request):
        """提供给客户端查询订单支付结果"""
        out_trade_no = request.query_params.get("out_trade_no")
        try:
            reward = Reward.objects.get(out_trade_no=out_trade_no)
        except Reward.DoesNotExist:
            return Response({"detail":"当前打赏记录不存在！"}, status=status.HTTP_400_BAD_REQUEST)

        return Response({"status": reward.status})

    def post(self,request):
        """异步回调结果的处理"""
        data = request.data
        signature = data.pop("sign")

        alipay = self.get_alipay()

        # 验证支付的异步通知结果
        success = alipay.verify(data, signature)
        if success and data["trade_status"] in ("TRADE_SUCCESS", "TRADE_FINISHED" ):
            # 完成支付结果的处理
            # 根据订单号提取订单信息
            out_trade_no = data.get("out_trade_no")
            try:
                reward = Reward.objects.get(out_trade_no=out_trade_no)
                if reward.status:
                    return Response("success",content_type="text/plain")
            except Reward.DoesNotExist:
                return Response("success",content_type="text/plain")

            self.change_reward(data,reward)

            # 返回结果
            return Response("success",content_type="text/plain")

        return Response("fail", status=status.HTTP_400_BAD_REQUEST,content_type="text/plain")

    def change_reward(self,data,reward):
        # 根据订单的付款状态进行修改
        reward.status = True
        reward.trade_no = data.get("trade_no")
        reward.save()

        # 增加文章作者的资金
        reward.article.user.money += reward.money
        reward.article.user.save()

        # 增加文章的赞赏人数
        reward.article.reward_count += 1
        reward.article.save()

        # 记录用户的赞赏行为
        primary_key = {"id": constants.LOG_TABLE_ID, "user_id": reward.user.id, "message_id": reward.article.id}
        attribute_columns = {
            "is_reward": 1,
            "timestamp": datetime.now().timestamp(),
        }
        OTS().update_row("user_message_log_table", primary_key, attribute_columns)

    def get_alipay(self):
        # 读取秘钥文件的内容
        app_private_key_string = open(settings.ALIPAY.get("app_private_key_path")).read()
        alipay_public_key_string = open(settings.ALIPAY.get("alipay_public_key_path")).read()
        # 根据支付宝sdk生成支付链接
        alipay = AliPay(
            appid=settings.ALIPAY.get("appid"),  # appid
            app_notify_url=settings.ALIPAY.get("app_notify_url"),  # 默认回调url
            app_private_key_string=app_private_key_string,
            alipay_public_key_string=alipay_public_key_string,
            sign_type=settings.ALIPAY.get("sign_type"),
            debug=settings.ALIPAY.get("debug")
        )

        return alipay
```

为了方便实现智能推荐，我们在manage自定义命令中新增一个添加测试数据的命令。

home/managment/commands/tablestroe.py，代码：

```python
from django.core.management import BaseCommand
from renranapi.utils.tablestore import *
from renranapi.settings import constants
from django.utils import timezone as datetime
class Command(BaseCommand):
    help = """表格存储命令必须接收而且只接收1个命令参数，如下：
    python manage.py tablestore create  表示创建项目使用的表格 
    python manage.py tablestore delete  表示删除项目使用的表格
    python manage.py tablestore log  表示添加行为日志的测试数据
    """
    def __init__(self, *args, **kwargs):
        self.client = OTS()
        super().__init__(*args, **kwargs)


    def add_arguments(self, parser):
        """参数设置"""
        parser.add_argument("argument",nargs="*", help="操作类型") # 位置参数

    def handle(self, *args, **options):
        """
        表格存储的初始化
        :param args:
        :param options:  参数列表
        :return:
        """
        argument = options.get("argument")
        if len(argument) != 1:
            self.stderr.write("操作有误！")
            self.stdout.write(self.help)
            return None

        if argument[0] == "create":
            """创建表格"""
            self.create_table()

        elif argument[0] == "delete":
            """删除表格"""
            self.delete_table()
        elif argument[0] == "log":
            """添加行为日志的测试数据"""
            self.create_log()
        else:
            self.stderr.write("操作有误！")
            self.stdout.write(self.help)

    def create_table(self):
        # 存储库[收件箱]
        table_name = "user_message_table"
        schema_of_primary_key = [  # 主键列
            ('id','INTEGER'),      # 分区键
            ('user_id', 'INTEGER'),
            ('sequence_id', 'INTEGER', PK_AUTO_INCR),
            ("message_id", 'INTEGER')
        ]
        self.stdout.write("表格《%s》创建开始..." % table_name)
        self.client.create_table(table_name, schema_of_primary_key)
        self.stdout.write("表格《%s》创建完成..." % table_name)

        # 关系库
        table_name = "user_relation_table"
        schema_of_primary_key = [ # 主键列
            ('id', 'INTEGER'),    # 分区键
            ('user_id', 'INTEGER'),
            ("follow_user_id", 'INTEGER'),
        ]

        self.stdout.write("表格《%s》创建开始..." % table_name)
        self.client.create_table(table_name, schema_of_primary_key)
        self.stdout.write("表格《%s》创建完成..." % table_name)

        # 未读池
        table_name = "user_message_session_table"
        # 主键列
        schema_of_primary_key = [
            ('id', 'INTEGER'),
            ('follow_user_id', 'INTEGER'),
            ("last_sequence_id", 'INTEGER'),
        ]

        self.client.create_table(table_name, schema_of_primary_key)
        self.stdout.write("表格《%s》创建完成..." % table_name)

        # 推送日志
        table_name = "user_message_log_table"
        schema_of_primary_key = [  # 主键列
            ('id', 'INTEGER'),
            ('user_id', 'INTEGER'),
            ("message_id", 'INTEGER'),
        ]

        self.client.create_table(table_name, schema_of_primary_key)
        self.stdout.write("表格《%s》创建完成..." % table_name)

    def delete_table(self):
        """删除表操作"""
        table_list = OTS().list_table()
        for table_name in table_list:
            self.client.delete_table(table_name)
            self.stdout.write("表格《%s》删除完成..." % table_name)

        self.stdout.write("所有表格删除完成...")

    def create_log(self):
        """添加行为日志测试数据"""
        primary_key_list = [
            {'id':constants.LOG_TABLE_ID,"user_id":1,"message_id":23,},
            {'id':constants.LOG_TABLE_ID,"user_id":1,"message_id":5,},
            {'id':constants.LOG_TABLE_ID,"user_id":1,"message_id":6,},
            {'id':constants.LOG_TABLE_ID,"user_id":1,"message_id":8,},
            {'id':constants.LOG_TABLE_ID,"user_id":1,"message_id":13,},
            {'id':constants.LOG_TABLE_ID,"user_id":2,"message_id":5,},
            {'id':constants.LOG_TABLE_ID,"user_id":2,"message_id":16,},
            {'id':constants.LOG_TABLE_ID,"user_id":2,"message_id":13,},
            {'id':constants.LOG_TABLE_ID,"user_id":2,"message_id":8,},
            {'id':constants.LOG_TABLE_ID,"user_id":2,"message_id":23,},
            {'id':constants.LOG_TABLE_ID,"user_id":3,"message_id":17,},
            {'id':constants.LOG_TABLE_ID,"user_id":3,"message_id":13,},
            {'id':constants.LOG_TABLE_ID,"user_id":3,"message_id":23,},
            {'id':constants.LOG_TABLE_ID,"user_id":3,"message_id":7,},
            {'id':constants.LOG_TABLE_ID,"user_id":3,"message_id":4,},
            {'id':constants.LOG_TABLE_ID,"user_id":4,"message_id":8,},
            {'id':constants.LOG_TABLE_ID,"user_id":4,"message_id":6,},
            {'id':constants.LOG_TABLE_ID,"user_id":4,"message_id":13,},
            {'id':constants.LOG_TABLE_ID,"user_id":4,"message_id":16,},
            {'id':constants.LOG_TABLE_ID,"user_id":4,"message_id":9,},
            {'id':constants.LOG_TABLE_ID,"user_id":4,"message_id":23,},
            {'id':constants.LOG_TABLE_ID,"user_id":5,"message_id":5,},
            {'id':constants.LOG_TABLE_ID,"user_id":5,"message_id":6,},
            {'id':constants.LOG_TABLE_ID,"user_id":5,"message_id":7,},
            {'id':constants.LOG_TABLE_ID,"user_id":5,"message_id":8,},
            {'id':constants.LOG_TABLE_ID,"user_id":5,"message_id":16,},
            {'id':constants.LOG_TABLE_ID,"user_id":5,"message_id":17,},
        ]

        attribute_columns_list = [
            {"is_comment": 1, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 1, "is_read": 1, "is_reward": 1, "timestamp": datetime.now().timestamp()},
            {"is_comment": 1, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 1, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 1, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 1, "timestamp": datetime.now().timestamp()},
            {"is_comment": 1, "is_like": 1, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 1, "is_read": 1, "is_reward": 1, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 1, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 1, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 1, "is_like": 1, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 1, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 1, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 1, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 1, "is_like": 0, "is_read": 1, "is_reward": 1, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 1, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 1, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
            {"is_comment": 1, "is_like": 0, "is_read": 1, "is_reward": 1, "timestamp": datetime.now().timestamp()},
            {"is_comment": 0, "is_like": 0, "is_read": 1, "is_reward": 0, "timestamp": datetime.now().timestamp()},
        ]

        table_name = "user_message_log_table"
        self.client.add_list(table_name,primary_key_list=primary_key_list,attribute_columns_list=attribute_columns_list)
        self.stdout.write("%s表的测试数据创建完成..." % table_name)
```

在终端下面执行操作

```bash
python manage.py tablestore log
```



接下来，就可以在首页中当关注的数据没有了。则进行智能推荐。

### 首页实现基于物品实现文章智能推荐

获取用户的行为记录以及相关用户的行为记录，视图home/views.py，代码：

```python
from article.models import Article
from .serializers import ArticleModelSerializer
from .paginations import HomeArticlePageNumberPagination
from users.models import User
from renranapi.utils.tablestore import *
from renranapi.settings import constants
class ArticleListAPIView(ListAPIView):
    serializer_class = ArticleModelSerializer
    pagination_class = HomeArticlePageNumberPagination

    def get_queryset(self):
        user = self.request.user
        if isinstance(self.request.user,User):
            """登录"""
            start_key = {"id": constants.MESSAGE_TABLE_ID, "user_id":user.id, "sequence_id": INF_MIN, "message_id": INF_MIN}
            end_key   = {"id": constants.MESSAGE_TABLE_ID, "user_id":user.id, "sequence_id": INF_MAX, "message_id": INF_MAX}
            cond = CompositeColumnCondition(LogicalOperator.AND)
            cond.add_sub_condition(SingleColumnCondition("is_cancel", False, ComparatorType.EQUAL))
            cond.add_sub_condition(SingleColumnCondition("is_read", False, ComparatorType.EQUAL))
            message_list = []
            primary_list = []
            # 接受客户端执行返回的单页数据量，如果客户端没有指定，则默认采用分页器的单页数据量
            size = int(self.request.query_params.get("size")) or self.pagination_class.page_size
            client = OTS()
            ret = client.get_list("user_message_table",start_key,end_key,limit=size, cond=cond)

            if ret["status"]:
                for item in ret["data"]:
                    message_list.append(item["message_id"])
                    primary_list.append(item)
                while ret["token"]:
                    # print(ret["token"]) # [('id', 1), ('user_id', 2), ('sequence_id', 1596081490522997), ('message_id', 23)]
                    start_key = ret["token"]
                    end_key = {"id": constants.MESSAGE_TABLE_ID, "user_id": user.id, "sequence_id": INF_MAX,
                               "message_id": INF_MAX}
                    ret = client.get_list("user_message_table", start_key, end_key, limit=size, cond=cond)
                    for item in ret["data"]:
                        message_list.append(item["message_id"])
                        primary_list.append(item)

                queryset = Article.objects.filter(is_public=True, is_show=True, is_deleted=False, pk__in=message_list).order_by("-id")

                # 记录推送状态到同步库中
                page = self.request.query_params.get("page")
                if page is None:
                    page = 1
                page = int(page)

                update_primary_list = []
                attribute_columns_list = []
                article_list = queryset[(page-1)*size:page*size]
                for article in article_list:
                    for data in primary_list:
                        if data["user_id"] == user.id and data["message_id"] == article.id:
                            update_primary_list.append(data)
                            attribute_columns_list.append({"is_read": True})

                client.update_list("user_message_table", update_primary_list,attribute_columns_list)

            if len(queryset) < 1:
                """进行智能推荐"""
                # 1. 获取当前登录用户近3个月的行为记录
                current_user_log_list = user.get_user_log()

                # 2. 把阅读过与当前登录用户一样的文章的所有关联用户查询出来
                user_list = user.get_log_list(current_user_log_list)

                # 3. 获取每个关联用户近期的行为记录
                for user_id in user_list:
                    user_log_list = user.get_user_log(user_id)
                    print(user_log_list)
                
                # 4. 进行统计和计算获取推荐列表

        else:
            queryset = Article.objects.filter(is_public=True, is_show=True, is_deleted=False).order_by("-reward_count", "-comment_count", "-like_count", "-id")

        return queryset
```

users/models.py，代码：

```python
from django.db import models
from django.contrib.auth.models import AbstractUser
# Create your models here.
# AbstractUser 是django内部的auth模块里面声明的抽象模型，
# 抽象模型，是django内部用于提供给开发者声明模型父类的工具类，这种模型类，
# 在数据迁移的时候django不会为他们单独建表，所以这种模型的作用就是用于声明公共字段的
# 怎么声明一个django的抽象模型类？
"""
class 模型类(models.Model):
    字段列表
    字段列表
    class Meta:
        abstract = True # 表示当前模型是抽象模型
"""
from renranapi.utils.tablestore import *
from renranapi.settings import constants
from django.utils import timezone as datetime
class User(AbstractUser):
    nickname = models.CharField(max_length=20, null=True, verbose_name="用户昵称")
    mobile = models.CharField(max_length=15, unique=True, verbose_name="手机号")
    avatar = models.ImageField(upload_to="avatar", null=True, verbose_name="头像")
    wxchat = models.CharField(max_length=100, null=True, unique=True, verbose_name="微信账号")
    alipay = models.CharField(max_length=100, null=True, unique=True, verbose_name="支付宝账号")
    qq_number = models.CharField(max_length=11, null=True, unique=True, verbose_name="QQ号")
    money = models.DecimalField(decimal_places=2,max_digits=8,default=0, verbose_name="用户资金")
    class Meta:
        # 自定义表名
        db_table = "rr_users"
        verbose_name = "用户信息"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.nickname if self.nickname else self.username

    def get_user_log(self,user_id=None, limit=50):
        """获取指定用户最近3个月的行为记录"""
        if user_id is None:
            user_id = self.id
        start_key = {"id": constants.LOG_TABLE_ID, "user_id": user_id,"message_id": INF_MIN}
        end_key   = {"id": constants.LOG_TABLE_ID, "user_id": user_id,"message_id": INF_MAX}
        columns_to_get = ["is_read","is_comment","is_reward","is_like"]
        timestamp = datetime.now().timestamp() - 3 * 30 * 24 * 60 * 60
        cond = SingleColumnCondition("timestamp", timestamp, ComparatorType.GREATER_EQUAL)
        ret = OTS().get_list("user_message_log_table", start_key,end_key,limit=limit,columns_to_get=columns_to_get, cond=cond)
        log_list = []
        if ret["status"]:
            """查询成功"""
            log_list.extend(ret["data"])
            while ret["token"] and len(log_list)<limit:
                ret = OTS().get_list("user_message_log_table", start_key, end_key,limit=limit,columns_to_get=columns_to_get, cond=cond)
                if ret["status"]:
                    log_list.extend(ret["data"])
        return log_list

    def get_log_list(self,log_list):

        start_key = {"id": constants.LOG_TABLE_ID, "user_id": INF_MIN, "message_id": INF_MIN}
        end_key = {"id": constants.LOG_TABLE_ID, "user_id": INF_MAX, "message_id": INF_MAX}
        columns_to_get = ["is_read", "is_comment", "is_reward", "is_like"]
        timestamp = datetime.now().timestamp() - 15 * 24 * 60 * 60
        limit = 50
        user_list = set()
        for log_item in log_list:
            cond = CompositeColumnCondition(LogicalOperator.AND)
            cond.add_sub_condition(SingleColumnCondition("timestamp", timestamp, ComparatorType.GREATER_EQUAL))
            cond.add_sub_condition(SingleColumnCondition("message_id", log_item["message_id"], ComparatorType.EQUAL))
            ret = OTS().get_list("user_message_log_table", start_key, end_key, limit=limit,
                                 columns_to_get=columns_to_get, cond=cond)
            if ret["status"]:
                for data in ret["data"]:
                    user_list.add( data["user_id"] )

        return list(user_list)
```

经过上面的处理，我们获取了用户以及相关用户的行为记录，接下来就要实现类似如下图标的功能。

黑色背景表示当前用户阅读过的文章【当前登录用户为用户2】

![1596183551447](11Feed%E6%B5%81%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1.assets/1596183551447.png)

#### 根据现有的行为记录，给当前用户进行推荐

```python
from article.models import Article
from .serializers import ArticleModelSerializer
from .paginations import HomeArticlePageNumberPagination
from users.models import User
from renranapi.utils.tablestore import *
from renranapi.settings import constants
class ArticleListAPIView(ListAPIView):
    serializer_class = ArticleModelSerializer
    pagination_class = HomeArticlePageNumberPagination

    def get_queryset(self):
        user = self.request.user
        queryset = []
        if isinstance(self.request.user,User):
            """登录"""
            start_key = {"id": constants.MESSAGE_TABLE_ID, "user_id":user.id, "sequence_id": INF_MIN, "message_id": INF_MIN}
            end_key   = {"id": constants.MESSAGE_TABLE_ID, "user_id":user.id, "sequence_id": INF_MAX, "message_id": INF_MAX}
            cond = CompositeColumnCondition(LogicalOperator.AND)
            cond.add_sub_condition(SingleColumnCondition("is_cancel", False, ComparatorType.EQUAL))
            cond.add_sub_condition(SingleColumnCondition("is_read", False, ComparatorType.EQUAL))
            message_list = []
            primary_list = []
            # 接受客户端执行返回的单页数据量，如果客户端没有指定，则默认采用分页器的单页数据量
            size = int(self.request.query_params.get("size")) or self.pagination_class.page_size
            client = OTS()
            ret = client.get_list("user_message_table",start_key,end_key,limit=size, cond=cond)

            if ret["status"]:
                for item in ret["data"]:
                    message_list.append(item["message_id"])
                    primary_list.append(item)
                while ret["token"]:
                    # print(ret["token"]) # [('id', 1), ('user_id', 2), ('sequence_id', 1596081490522997), ('message_id', 23)]
                    start_key = ret["token"]
                    end_key = {"id": constants.MESSAGE_TABLE_ID, "user_id": user.id, "sequence_id": INF_MAX,
                               "message_id": INF_MAX}
                    ret = client.get_list("user_message_table", start_key, end_key, limit=size, cond=cond)
                    for item in ret["data"]:
                        message_list.append(item["message_id"])
                        primary_list.append(item)

                queryset = Article.objects.filter(is_public=True, is_show=True, is_deleted=False, pk__in=message_list).order_by("-id")

                # 记录推送状态到同步库中
                page = self.request.query_params.get("page")
                if page is None:
                    page = 1
                page = int(page)

                update_primary_list = []
                attribute_columns_list = []
                article_list = queryset[(page-1)*size:page*size]
                for article in article_list:
                    for data in primary_list:
                        if data["user_id"] == user.id and data["message_id"] == article.id:
                            update_primary_list.append(data)
                            attribute_columns_list.append({"is_read": True})

                client.update_list("user_message_table", update_primary_list,attribute_columns_list)

            if len(queryset) < 1:
                """进行智能推荐"""
                # 1. 获取当前登录用户近3个月的行为记录
                current_user_log_list = user.get_user_log()

                # 2. 把阅读过与当前登录用户一样的文章的所有关联用户查询出来
                user_list,message_list = user.get_log_list(current_user_log_list)

                # 3. 获取每个关联用户近期的行为记录,把所有用户的行为记录合并成一个大数组
                cf_list = {}
                for user_id in user_list:
                    user_log_list = user.get_user_log(user_id)
                    # 遍历每一个相关用户的行为记录，得到每一个文章的推荐度
                    for user_log in user_log_list:
                        cf_num = user_log["is_read"] * constants.CF_READ + user_log["is_like"] * constants.CF_LIKE + user_log['is_comment'] * constants.CF_COMMENT + user_log["is_reward"] * constants.CF_REWARD
                        if user_log["message_id"] not in message_list: # 只有当前用户没有阅读过的文章，才统计推荐度
                            if user_log["message_id"] in cf_list:
                                """累加推荐度"""
                                cf_list[user_log["message_id"]] += cf_num
                            else:
                                """新增一个推荐文章"""
                                cf_list[user_log["message_id"]] = cf_num

                # 4.3 字典排序，按推荐度的值从大到小进行排列
                cf_list = sorted(cf_list.items(), key=lambda x: x[1], reverse=True)
                # [(6, 18), (9, 16), (7, 9), (4, 8), (17, 2)]
                # 我们只需要文章ID即可。
                cf_list = [cf_item[0] for cf_item in cf_list]
                queryset = Article.objects.filter(is_public=True, is_show=True, is_deleted=False, pk__in=cf_list).order_by("-id")

        if len(queryset) < 1:
            queryset = Article.objects.filter(is_public=True, is_show=True, is_deleted=False).order_by("-reward_count", "-comment_count", "-like_count", "-id")

        return queryset
```

用户模型调整：

```python
from django.db import models
from django.contrib.auth.models import AbstractUser
# Create your models here.
# AbstractUser 是django内部的auth模块里面声明的抽象模型，
# 抽象模型，是django内部用于提供给开发者声明模型父类的工具类，这种模型类，
# 在数据迁移的时候django不会为他们单独建表，所以这种模型的作用就是用于声明公共字段的
# 怎么声明一个django的抽象模型类？
"""
class 模型类(models.Model):
    字段列表
    字段列表
    class Meta:
        abstract = True # 表示当前模型是抽象模型
"""
from renranapi.utils.tablestore import *
from renranapi.settings import constants
from django.utils import timezone as datetime
class User(AbstractUser):
    nickname = models.CharField(max_length=20, null=True, verbose_name="用户昵称")
    mobile = models.CharField(max_length=15, unique=True, verbose_name="手机号")
    avatar = models.ImageField(upload_to="avatar", null=True, verbose_name="头像")
    wxchat = models.CharField(max_length=100, null=True, unique=True, verbose_name="微信账号")
    alipay = models.CharField(max_length=100, null=True, unique=True, verbose_name="支付宝账号")
    qq_number = models.CharField(max_length=11, null=True, unique=True, verbose_name="QQ号")
    money = models.DecimalField(decimal_places=2,max_digits=8,default=0, verbose_name="用户资金")
    class Meta:
        # 自定义表名
        db_table = "rr_users"
        verbose_name = "用户信息"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.nickname if self.nickname else self.username

    def get_user_log(self,user_id=None, limit=50):
        """获取指定用户最近3个月的行为记录"""
        if user_id is None:
            user_id = self.id
        start_key = {"id": constants.LOG_TABLE_ID, "user_id": user_id,"message_id": INF_MIN}
        end_key   = {"id": constants.LOG_TABLE_ID, "user_id": user_id,"message_id": INF_MAX}
        columns_to_get = ["is_read","is_comment","is_reward","is_like"]
        timestamp = datetime.now().timestamp() - 3 * 30 * 24 * 60 * 60
        cond = SingleColumnCondition("timestamp", timestamp, ComparatorType.GREATER_EQUAL)
        ret = OTS().get_list("user_message_log_table", start_key,end_key,limit=limit,columns_to_get=columns_to_get, cond=cond)
        log_list = []
        if ret["status"]:
            """查询成功"""
            log_list.extend(ret["data"])
            while ret["token"] and len(log_list)<limit:
                ret = OTS().get_list("user_message_log_table", start_key, end_key,limit=limit,columns_to_get=columns_to_get, cond=cond)
                if ret["status"]:
                    log_list.extend(ret["data"])
        return log_list

    def get_log_list(self,log_list):

        start_key = {"id": constants.LOG_TABLE_ID, "user_id": INF_MIN, "message_id": INF_MIN}
        end_key = {"id": constants.LOG_TABLE_ID, "user_id": INF_MAX, "message_id": INF_MAX}
        columns_to_get = ["is_read", "is_comment", "is_reward", "is_like"]
        timestamp = datetime.now().timestamp() - 15 * 24 * 60 * 60
        limit = 50
        # 与用户2有相同浏览历史的用户列表
        user_list = set()
        # 保存用户2看过的文章列表
        message_list = []
        for log_item in log_list:
            message_list.append(log_item["message_id"])
            cond = CompositeColumnCondition(LogicalOperator.AND)
            cond.add_sub_condition(SingleColumnCondition("timestamp", timestamp, ComparatorType.GREATER_EQUAL))
            cond.add_sub_condition(SingleColumnCondition("message_id", log_item["message_id"], ComparatorType.EQUAL))
            ret = OTS().get_list("user_message_log_table", start_key, end_key, limit=limit,
                                 columns_to_get=columns_to_get, cond=cond)
            if ret["status"]:
                for data in ret["data"]:
                    user_list.add( data["user_id"] )

        return list(user_list),message_list
```

renranapi/settings/constants.py代码：

```python
# 首页文章推荐度
CF_READ = 1     # 阅读文章的推荐度
CF_LIKE = 2     # 点赞文章的推荐度
CF_COMMENT= 5   # 评论文章的推荐度
CF_REWARD = 10  # 打赏文章的推荐度
```

一般在市面上Feed系统都会搭配一个站内搜索出现的。