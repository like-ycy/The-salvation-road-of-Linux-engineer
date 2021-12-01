# Django ORM常用字段和参数

## ORM常用字段

### 1、字段类型

| 字段名称         | 字段说明                                                     | 参数                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| AutoField        | 一个根据实际id进行自动增长的inter field（默认创建的）        | primary_key = True                                           |
| CharField        | varchar类型字段                                              | max_length 存储值最大长度                                    |
| TextField        | longtext 长文本                                              |                                                              |
| IntegerField     | int类型字段 存储整形                                         |                                                              |
| SmallInterField  | 小整数                                                       | -32768-32767                                                 |
| DecimalField     | 存储浮点型 更加精准                                          | max_digits=None 位数长度decimal_places=None 小数的位数       |
| FloatField       | 浮点型                                                       |                                                              |
| BooleanField     | 存储bool值 True/False                                        |                                                              |
| NullBooleanField | 存储Null/True/False                                          |                                                              |
| DateField        | date字段                                                     | auto_now=False 如果对数据进行修改则会自动保存修改时间 <br/>auto_now_add=False 会自动添加第一次保存的时间  俩个不同同时设定 |
| TimeField        | time字段                                                     | 同上                                                         |
| DateTimeField    | datetimefield字段                                            | 同上                                                         |
| FileField        | 文件上传                                                     | upload_to = '' 文件上传保存的路径                            |
| ImageField       | 图片上传字段（继承了FileField 对图像进行了验证 确保是一个有效的图片） | 同上                                                         |

**orm字段与mysql字段对应关系**

```
对应关系：
    'AutoField': 'integer AUTO_INCREMENT',
    'BigAutoField': 'bigint AUTO_INCREMENT',
    'BinaryField': 'longblob',
    'BooleanField': 'bool',
    'CharField': 'varchar(%(max_length)s)',
    'CommaSeparatedIntegerField': 'varchar(%(max_length)s)',
    'DateField': 'date',
    'DateTimeField': 'datetime',
    'DecimalField': 'numeric(%(max_digits)s, %(decimal_places)s)',
    'DurationField': 'bigint',
    'FileField': 'varchar(%(max_length)s)',
    'FilePathField': 'varchar(%(max_length)s)',
    'FloatField': 'double precision',
    'IntegerField': 'integer',
    'BigIntegerField': 'bigint',
    'IPAddressField': 'char(15)',
    'GenericIPAddressField': 'char(39)',
    'NullBooleanField': 'bool',
    'OneToOneField': 'integer',
    'PositiveIntegerField': 'integer UNSIGNED',
    'PositiveSmallIntegerField': 'smallint UNSIGNED',
    'SlugField': 'varchar(%(max_length)s)',
    'SmallIntegerField': 'smallint',
    'TextField': 'longtext',
    'TimeField': 'time',
    'UUIDField': 'char(32)',
```



### 2、字段选项

通过字段选项，可以实现对字段约束，在字段对象使用关键字参数进行指定

| 选项        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| null        | 如果设置为True  则当前字段可以为null                         |
| blank       | 如果设置为True 则当前字段可以什么值都没有 设置在字符串类型的字段上 |
| db_column   | 设置字段名称  如果不设置 默认为属性名                        |
| db_index    | 设置为True  常规索引                                         |
| unique      | 设置为True  唯一索引                                         |
| primary_key | 设置为True   主键索引                                        |
| default     | 默认值                                                       |



[更多字段类型和参数请点击](https://www.cnblogs.com/liuqingzheng/articles/9627915.html)

