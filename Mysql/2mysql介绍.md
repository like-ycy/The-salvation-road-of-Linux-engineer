# MySQL介绍

## 一、MySQL简介

MySQL是一个**[关系型数据库管理系统](https://baike.baidu.com/item/关系型数据库管理系统/696511)**，由瑞典[MySQL AB](https://baike.baidu.com/item/MySQL AB/2620844) 公司开发，属于 [Oracle](https://baike.baidu.com/item/Oracle) 旗下产品。MySQL 是最流行的[关系型数据库管理系统](https://baike.baidu.com/item/关系型数据库管理系统/696511)之一，在 [WEB](https://baike.baidu.com/item/WEB/150564) 应用方面，MySQL是最好的 [RDBMS](https://baike.baidu.com/item/RDBMS/1048260) (Relational Database Management System，关系数据库管理系统) 应用软件之一。

MySQL是一种关系型数据库管理系统，关系数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。

MySQL所使用的 SQL 语言是用于访问[数据库](https://baike.baidu.com/item/数据库/103728)的最常用标准化语言。MySQL 软件采用了双授权政策，分为社区版和商业版，由于其体积小、速度快、总体拥有成本低，尤其是[开放源码](https://baike.baidu.com/item/开放源码/7176422)这一特点，一般中小型网站的开发都选择 MySQL 作为网站数据库。

## 二、MySQL的发展历程

让我们牢记一个人：**天才少年[蒙提](https://en.wikipedia.org/wiki/Michael_Widenius)**。 MySQL的历史最早可以追溯到1979年，那时候，蒙提用[BASIC](https://zh.wikipedia.org/zh-hans/BASIC)设计了一个报表工具，没过多久，就用C语言重写了该工具，移植到了Unix平台，在当时，他只是一个很底层的面向报表的存储引擎，这个工具叫做Unireg。

- 1985年，他与Allan Larsson一起创办了TCX DataKonsult AB（一家瑞典数据仓库公司）。
- 1990年，在最初，他们只是自己设计了一个利用索引顺序取数据的方法，也就是ISAM（Indexed Sequential Access Method）存储引擎核心算法的前身，利用ISAM结合[mSQL](https://en.wikipedia.org/wiki/MSQL)来实现需求。早期的时候，该公司主要为瑞典的一些大型零售商提供数据仓库服务，但是随着数据量增大、系统复杂度越来越高，ISAM和mSQL的组合逐渐不堪重负。在分析性能的瓶颈后，问题出现在mSQL上，后来他们抛弃了mSQL，重新开发一套功能类似的数据存储引擎，这就是ISAM存储引擎。
- 1995年，蒙提使用[David Axmark](https://en.wikipedia.org/wiki/David_Axmark)编写第一版MySQL数据库，于1996年发布。需要说明的蒙提和David Axmark及Allan Larsson在同年创立了MySQL AB公司。
- 1996年，MySQL的1.0版本发布，当时真是面向一部分人，直到同年的10月，MySQL的3.11.1发布！是的，这群不按套路的天才们，直接跳过了2.x版本！最开始，只是提供了[Solaris](https://zh.wikipedia.org/wiki/Solaris)下的二进制版本。一个月后，有了Linux版本。 在接下来的两年里，MySQL不断的移植到各个平台下。
- 1999年至2000年，MySQL AB与[Sleepycat](https://en.wikipedia.org/wiki/Sleepycat_Software)合作开发[Berkeley DB](https://en.wikipedia.org/wiki/Berkeley_DB)引擎，因为该引擎支持事物处理。
- 2000年，MySQL对旧的存储引擎进行了整理，命名为MyISAM。
- 2001年，[Innobase](https://en.wikipedia.org/wiki/Innobase)公司与MySQL AB达成合作，共同开发InnoDB引擎，该引擎同样支持事物，并且支持行级锁。同年MySQL发布MySQL4.0版本，该版本正式支持InnoDB引擎。
- 2005年，MySQL发布了经典的5.0版本，MySQL在该版本中加入了游标、触发器、存储过程、视图等功能。同年，Oracle公司以迅雷不及掩耳之势收购了innobase公司。
- 2008年，Sun公司以10亿美金收购了MySQL AB公司。
- 2010年，可惜，Sun公司好景不长，因为2010年，Oracle公司又收购了Sun公司。从此，MySQL归Oracle所有。Oracle公司对MySQL分为社区版和企业版，社区版免费，企业版收费，当然，也提供更多的功能！同年4月22日MySQL5.5和MySQL Cluster7.1版本发布。
- 2013年，2013年2月6日MySQL5.6发布，InnoDB性能和复制一致性加强。
- 2014年，2014年4月4日Oracle发布MySQL里程碑式的版本MySQL5.7版本，此版本满足网络、云和嵌入式需求，性能更强、扩展性和可靠性得到提高。
- 2016年，2016年9月12日，Oracle正式发布MySQL8.0版本。没错，直接跳过了6.x、7.x版本。

**sakila**

我们都知道MySQL的logo是海豚（sakila），而sakila的由来则是来自MySQL AB公司创始人从"海豚命名"竞赛中得来的，这个sakila是来自于坦桑尼亚的Arusha的一个小镇的名字。

### 三、MySQL的特点

**开源**

开源就意味着免费！在中国，免费意味着什么？还用我多说吗？ 当然，自从MySQL被Oracle收购后，就不太好了，但是我们还有mariaDB。

**健壮的社区**

这是一个要命的话题！MySQL有着健壮的社区，有专门的人员在维护这个社区。 一个资源丰富的社区，是一个软件成功的关键，这点从Python就可以看出来，Python的社区做的可以说是非常的好，大家可以在这个社区找到自己想要的东西，碰到问题能很快的找到解决办法，有了新的见解，也愿意写成博客之类的供大家参考，良性发展下，Python也越发展越好，这个道理同样适用于各个软件，包括MySQL，MySQL在世界范围内有着庞大的用户群，这些用户都在为MySQL的发展提供了直接或间接的力量。

**稳定**

没错，对于一个公司，尤其是互联网公司而言，开发出来的软件稳定肯定是要放在第一位的。更何况重中之重的数据库。MySQL经过几十年的发展，已经越来越稳定，极少出现宕机情况。

**跨平台**

MySQL支持AIX、FreeBSD、HP-UX、Linux、Mac OS、NovellNetware、OpenBSD、OS/2 Wrap、Solaris、Windows等多种操作系统

**丰富的API接口**

不管你数据库再厉害，那也是要搭配编程语言来实现具体的功能，丰富的API接口意味着有更多的编程语言可以与MySQL无缝协作，完成高效的开发。这些编程语言包括C、C++、Python、Java、Perl、PHP、Eiffel、Ruby和Tcl等

**支持标准的SQL语句**

世界范围内，有那么多的数据库软件，你有你的一套操作规则，他有他的一套操作规则，那么，一个公司在选择一个数据库，还要专门的学习这门数据库的相关规则，查询语法什么的，这就增了生产成本。为了解决这个问题，在1970年的[埃德加·科德](https://zh.wikipedia.org/wiki/埃德加·科德)的一篇相当有影响力的论文中《[一个对于大型共享型数据库的关系模型](https://web.archive.org/web/20070612235326/http://www.acm.org/classics/nov95/toc.html)》描述了[SQL](https://zh.wikipedia.org/wiki/SQL)这个结构化查询语言，主要应用与数据库的管理。后来这玩意儿成了数据库语言的标准。MySQL使用这种标准的SQL语言。

**多种存储引擎**

是的，MySQL有不同的引擎来支持不同的应用场景。

## 四、社区版和企业版的区别

主要的区别有以下俩种

1. 企业版只包含稳定之后的功能  社区版包含MySQL所有最新的功能

2. 官方的服务支持只针对企业版 用户在使用社区版的时候出现的任何问题MySQL官方概不负责

## 五、目前主流的MySQL版本及分支

目前较为主流的MySQL版本：

- 5.6
- 5.7
- 8.0

其他版本：

- Oracle： - MySQL官方版
- 红帽： - MariaDB，MySQL被Oracle收购后，MySQL的开发者就搞了个MariaDB继续开源！感谢大佬！！！
- Percona: - PerconaDB，了解即可。

建议版本选择：

- Oracle的官方版本，5.6和5.7，当然，目前还是推荐5.7。
- GA(稳定发布)

如何获取MySQL：

- 企业版Enterprise，互联网行业一般不选择
- 社区版，推荐
- 源码包，通用二进制