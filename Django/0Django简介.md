# Django 简介

Django是一个开放[源代码](https://baike.baidu.com/item/源代码/3814213)的[Web应用框架](https://baike.baidu.com/item/Web应用框架/4262233)，由[Python](https://baike.baidu.com/item/Python/407313)写成。采用了MTV的框架模式。它最初是被开发来用于管理劳伦斯出版集团旗下的一些以新闻内容为主的网站的，即是CMS（内容管理系统）软件。并于2005年7月在[BSD许可证](https://baike.baidu.com/item/BSD许可证/10642412)下发布。这套框架是以比利时的吉普赛[爵士吉他](https://baike.baidu.com/item/爵士吉他/4696201)手Django Reinhardt来命名的。

## Django具有以下特点：

- Django是一个全栈Web框架。所谓全栈框架，是指除了封装网络和线程操作，还提供HTTP请求和响应、数据库读写管理、HTML模板渲染等一系列功能的框架。你可以不太准确地理解为全栈工程师包办了前后端和数据库访问的所有开发工作，整个网站都是一个人搭建的。
- 功能完善、要素齐全。该有的、可以没有的都有，常用的、不常用的工具都提供。Django提供了大量的特性和工具，无须你自己定义、组合、增删及修改。但是，在有些人眼里这被认为是臃肿不够灵活，发挥不了程序员的能动性。（一体机和DIY你更喜欢哪个？^-^）
- 完善的文档。经过长期的发展和完善，Django有广泛的实践经验和完善的在线文档。开发者遇到问题时可以搜索在线文档寻求解决方案。
- 强大的数据库访问API。Django的Model层自带数据库ORM组件，开发者无须学习其他数据库访问技术（例如SQLALchemy）。当然你也可以使用SQLALchemy，甚至不适用ORM组件。
- 灵活的路由系统。Django具备路由转发、正则表达式、命名空间、URL反向解析等功能。
- 丰富的Template模板功能：Django自带类似jinjia的模板语言，不但原生功能丰富，还可以自定义模板标签和过滤器。并且以类似Python的调用机制和视图默契配合。
- 自带后台管理应用admin：只需要通过简单的几行配置和代码就可以实现一个完整的后台数据管理控制平台。这是Django最受欢迎的功能。
- 完整的错误信息提示：在开发调试过程中如果出现运行错误或者异常，Django可以提供非常完整的错误信息帮助定位问题。

## MVC 与 MTV模型

### MVC 模型

MVC 模式（Model–view–controller）是软件工程中的一种软件架构模式，把软件系统分为三个基本部分：模型（Model）、视图（View）和控制器（Controller）。

MVC 以一种插件式的、松耦合的方式连接在一起。

- 模型（M）- 编写程序应有的功能，负责业务对象与数据库的映射(ORM)。
- 视图（V）- 图形界面，负责与用户的交互(页面)。
- 控制器（C）- 负责转发请求，对请求进行处理。

简易图：

![img](0Django%E7%AE%80%E4%BB%8B.assets/ModelViewControllerDiagramZh.png)

用户操作流程图：

![img](0Django%E7%AE%80%E4%BB%8B.assets/1589776521-2356-JxrlTyMyPgYnQpOV.png)

------

### MTV 模型

Django 的 MTV 模式本质上和 MVC 是一样的，也是为了各组件间保持松耦合关系，只是定义上有些许不同，Django 的 MTV 分别是指：

- M 表示模型（Model）：编写程序应有的功能，负责业务对象与数据库的映射(ORM)。
- T 表示模板 (Template)：负责如何把页面(html)展示给用户。
- V 表示视图（View）：负责业务逻辑，并在适当时候调用 Model和 Template。

除了以上三层之外，还需要一个 URL 分发器，它的作用是将一个个 URL 的页面请求分发给不同的 View 处理，View 再调用相应的 Model 和 Template，MTV 的响应模式如下所示：

简易图：

![img](0Django%E7%AE%80%E4%BB%8B.assets/MTV-Diagram.png)

用户操作流程图：

![img](0Django%E7%AE%80%E4%BB%8B.assets/1589777036-2760-fs1oSv4dOWAwC5yW.png)

**解析：**

用户通过浏览器向我们的服务器发起一个请求(request)，这个请求会去访问视图函数：

- a.如果不涉及到数据调用，那么这个时候视图函数直接返回一个模板也就是一个网页给用户。
- b.如果涉及到数据调用，那么视图函数调用模型，模型去数据库查找数据，然后逐级返回。

视图函数把返回的数据填充到模板中空格中，最后返回网页给用户。