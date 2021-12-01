# MySQL单表查询

## 比较（关系）运算符

- 语法：

```sql
SELECT 字段名1，字段名2，… FROM 表名 WHERE 条件表达式;
```

| **关系运算符** | **说     明** |
| -------------- | ------------- |
| =              | 等于          |
| <>             | 不等于        |
| !=             | 不等于        |
| <              | 小于          |
| <=             | 小于等于      |
| >              | 大于          |
| >=             | 大于等于      |

- 实例

  ```sql
  # 查询年龄大于20岁的
  select * from user where age>20;

  # 将年龄大于20的性别改为w  年龄减少2
  update user set sex='w', age=age-2 where age > 20;

  # 查询年龄小于20岁的
  select * from user where age<20;

  # 将年龄小于20的性别改为w  年龄减少2
  update user set sex='w', age=age-2 where age < 20;

  # 删除年龄大于等于30的数据
  delete from user where age >= 30;

  # 删除id 小于等于 5 的数据
  delete from user where id <= 5;

  # 查找性别为w的数据
  select * from user where sex='w';

  # 查询性别不为w的所有数据
  select * from user where sex != 'w';

  select * from user where sex <> 'w';
  ```

## 逻辑运算符

  | 逻辑运算符           | 说 明                  |
  | -------------------- | ---------------------- |
  | in                   | 在...里                |
  | not in               | 不在...里              |
  | and                  | 逻辑与  两侧为真才为真 |
  | or                   | 逻辑或  一侧为真即为真 |
  | between...and...     | 在...之间  包括本身    |
  | not between...and... | 不在...之间            |

- 实例

  ```sql
  ### and
  # 查询年龄为18 并且性别为w的数据
  select * from user where sex='w' and age=18;

  ### or
  # 修改性别为w或者m的数据 将年龄+2岁
  update user set age=age+2 where sex='w' or sex='m';

  ### between ... and ...
  # 查询年龄在18 -  28之间的数据
  select * from user where age>=18 and age <=28;
  select * from user where age between 18 and 28;

  ### not between...and...
  # 查询年龄不在18 -  28之间的数据
  select * from user where age < 18 or age >28;
  select * from user where age not between 18 and 28;

  ### in
  # 查询年龄为 18 28 38 的数据
  select * from user where age=18 or age=28 or age=38;
  select * from user where age in(18,28,38);

  ### not in
  # 查询年龄不为 18 28 38 的数据
  select * from user where age!=18 and age!=28 and age!=38;
  select * from user where age not in (18,28,38);
  ```

## order by 排序

- 语法：

```sql
SELECT 字段名1，字段名2，… FROM 表名 ORDER BY 字段名1 [ ASC | DESC ],字段名2 [ ASC | DESC ]…
```

- 实例

```sql
### 升序  asc
# 查询年龄从小到大
select * from user order by age;

select * from user order by age asc;

### 降序  desc
# 查询年龄从大到小
select * from user order by age desc;

select * from user where sex='w' order by age;
```

## limit 取值

- 作用

  限制查询结果的数量

- 语法

  ```sql
  SELECT 字段名2，字段名2，… FROM 表名 LIMIT [ OFFSET ,] 记录数;
  ```

+ 结构

  limit x     取出x条数据

  limit x,y   从x的位置取出y条数据

+ 实例

  ```sql
  # 取出3条数据
  select * from user limit 3;

  # 从第3条开始 取出3条
  select * from user limit 3,3;

  # 取出年龄最大的数据
  select * from user order by age desc limit 1;

  # 取出年龄最小的数据
  select * from user order by age asc limit 1;

  # 查询性别为w的 最大的年龄的数据
  select * from user where sex='w' order by age desc limit 1;
  ```

+ 分页

  100 条数据

  1页显示10条

  1 页数据    limit 0,10

  2 页数据     limit 10, 10

  limit  规律

  (nowpage -1)*everyPage

## 空值查询

+ is

  查询为空的数据

  select * from user where username is null;

+ is not

  查询不为空的数据

  select * from user where username is not null;

注意：

因为空值为特殊的值，所以不能使用= 、!= 等进行判断

## like 模糊查询

- 语法

  ```sql
  SELECT * | 字段名1，字段名2，… FROM 表名 WHERE 字段名 [ NOT ] LIKE ‘匹配字符串’;
  ```

| 符号     | 说 明                     |
| -------- | ------------------------- |
| '%...'   | 查询以某个字符结尾的数据  |
| '...%'   | 查询以某个字符开头的数据  |
| '%...%'  | 查询包含在两个%号中的数据 |
| '\_'     | 通配符，匹配任意一位字符  |
| not like | 模糊取反                  |

- 实例

  ```sql
  ### '%字符'
  # 查询用户名以y结尾的数据
  select * from user where username like '%y';

  ### '字符%'
  # 查询用户名以l开头的数据
  select * from user where username like 'l%';

  ### '%字符%'
  # 查询包含y的用户名
  select * from user where username like '%y%'

  ### '_'
  # 匹配俩位的用户名
  select * from user where username like '__';

  ### not like
  # 查询俩个字符以外的数据
  select * from user where username not like '__';
  ```



## DISTINCT 去重

- 语法

  ```sql
  SELECT DISTINCT 字段名 FROM 表名;
  ```

- 查询student2表中gender字段的值，结果中不允许出行重复的值。

  SELECT DISTINCT gender FROM student2;

!> 注意：DISTINCT 关键字还可作用于多个字段，则只有多个字段的值都完全相同时才会被认作是重复记录。

## GROUP BY 分组查询

在对表中数据进行统计的时候，可以使用GROUP BY 按某个字段或者多个字段进行分组，字段中值相同的为一组，如男生分为一组，女生分为一组。

- 语法

  ```sql
  SELECT  字段名1，字段名2，… FROM 表名 GROUP BY 字段名1，字段名2，… [ HAVING 条件表达式 ];
  ```

**单独使用 GROUP BY 进行分组**

```sql
# 统计男生女生分别有多少人
select sex,count(*) from user group by sex;

# 统计每个班级有多少人
select grade, count(*) from user group by grade;

# 统计每个班级的男生和女生分别有多少人
select grade,sex, count(*) from user group by grade,sex;
```

**GROUP BY 和聚合函数一起使用**

```sql
# 将student2表按照gender字段进行分组查询，计算出每组共有多少个人。
SELECT COUNT(*) ,gender FROM student2 GROUP BY gender;
```

**GROUP BY 和 HAVING 关键字一起使用**

```sql
# 查询班级人数大于2人
select grade,count(*) as count from user group by grade having count>2;

# 查看python35和python36班级的人数
select grade,count(*) as count from user group by grade having grade in('python35', 'python36');

# 查看python35和python36班级的人数  人数大于2人
select grade,count(*) as count from user group by grade having grade in('python35', 'python36') and count > 2;
```

## 聚合函数

| **函数名称** | **作用**         |
| ------------ | ---------------- |
| COUNT()      | 返回某列的行数   |
| SUM()        | 返回某列值的和   |
| AVG()        | 返回某列的平均值 |
| MAX()        | 返回某列的最大值 |
| MIN()        | 返回某列的最小值 |

## 正则表达式

| 模式   | 描述                          |
| ------ | ----------------------------- |
| ^      | 以某个字符作为开始            |
| $      | 以某个字符作为结尾            |
| .      | 匹配任意一个字符  \n以外      |
| []     | 字符集合  任意一个字符 [abc]  |
| [^...] | 取反                          |
| \|     | 或                            |
| *      | 匹配零次到多次                |
| +      | 匹配前面正则表达式的1次到多次 |
| {n}    | 匹配前面表达式的n次           |
| {n,m}  | 匹配前面表达式的n~m次         |

- 实例

  ```sql
  # 查询username中以l作为开头的数据
  select * from user where username regexp '^l';

  # 查询username中以y作为结尾的数据
  select * from user where username regexp 'y$';

  # 查询username中包含y的数据
  select * from user where username regexp 'y'

  # 查询为俩个字符的数据
  select * from user where username regexp '^[a-z]{2}$';

  # 查询字母以外的用户名
  select * from user where username regexp '\[^a-z]';
  ```


