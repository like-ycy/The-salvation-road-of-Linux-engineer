# time模块

time是python自带的模块，用于处理时间问题，提供了一系列的操作时间的函数。

1. UTC（Coordinated Universal Time，世界协调时）亦即格林威治天文时间，世界标准时间。在中国为UTC+8。DST（Daylight Saving Time）即夏令时。
2. 时间戳（timestamp）的方式：通常来说，时间戳表示的是从**1970年1月1日00:00:00**开始按秒计算的偏移量。我们运行“type(time.time())”，返回的是float类型。返回时间戳方式的函数主要有time()，clock()等。
3. 元组（struct_time）方式：struct_time元组共有9个元素，返回struct_time的函数主要有gmtime()，localtime()，strptime()。

| 索引（Index） | 属性（Attribute）         | 值（Values）       |
| :------------ | :------------------------ | :----------------- |
| 0             | tm_year（年）             | 比如2011           |
| 1             | tm_mon（月）              | 1 - 12             |
| 2             | tm_mday（日）             | 1 - 31             |
| 3             | tm_hour（时）             | 0 - 23             |
| 4             | tm_min（分）              | 0 - 59             |
| 5             | tm_sec（秒）              | 0 - 61             |
| 6             | tm_wday（weekday）        | 0 - 6（0表示周日） |
| 7             | tm_yday（一年中的第几天） | 1 - 366            |
| 8             | tm_isdst（是否是夏令时）  | 默认为-1           |



## time模块中常用的几个函数：

格式化日期函数

| 函数名                                              | 函数说明                                                     |
| --------------------------------------------------- | ------------------------------------------------------------ |
| time.time()                                         | 返回当前时间戳                                               |
| time.localtime([secs])                              | 将一个时间戳转换为当前时区的struct_time。secs参数未提供，则以当前时间为准。 |
| strftime("%Y-%m-%d  %H:%M:%S")                      | 接收时间元组 返回可读的字符串表示当地时间                    |
| asctime()                                           | 返回格式化后的英文的时间，如果没有参数，将会将time.localtime()作为参数传入。 |
| mktime()                                            | 将一个struct_time转化为时间戳。                              |
| sleep(seconds)                                      | 推迟线程调用的执行(进行阻塞)                                 |
| strptime("2021-8-25 11:15:00", '%Y-%m-%d %H:%M:%S') | 将给定时间转换为时间元组                                     |



时间格式化符号

| 符号 | 符号说明                |
| ---- | ----------------------- |
| %Y   | 四位的年                |
| %y   | 两位的年                |
| %m   | 月份（01-12）           |
| %d   | 月份内的某一天(1-31)    |
| %H   | 24小时制的时间(0-23)    |
| %l   | 12小时制的小时数(01-12) |
| %M   | 分钟数（0-59）          |
| %S   | 秒 （0-59）             |
| %a   | 本地简化星期的名称      |
| %j   | 年内的一天（011-366）   |
| %w   | 星期 (0-6)              |
| %x   | 本地相应的日期表示      |
| %X   | 本地相应的时间表示      |

使用

```python
import time
print(time.time())
print(time.localtime(time.time()-3600*24))
t_t = time.localtime(time.time()-3600*24)
print(time.strftime('%Y-%m-%d %H:%M:%S'))
print(time.mktime(t_t))
print(time.asctime())
time.sleep(2)
print(time.strptime('2021-08-25 11:30:00','%Y-%m-%d %H:%M:%S'))
```

案例

```python
import time

# 转换 为时间元组
t_t = time.strptime('1995-07-24 17:00:00', '%Y-%m-%d %H:%M:%S')
# 转换为时间戳
# print(t_t)
t = time.mktime(t_t)
# print(t)
print('活了多少秒：', t)
print('活了多少分', t/60)
print('活了多少小时', t/60/60)
print('活了多少天', t/60/60/24)
print('活了多少年', t/60/60/24/365)
```