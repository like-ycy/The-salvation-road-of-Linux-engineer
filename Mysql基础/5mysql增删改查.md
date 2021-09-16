# MySQL增删改查

## insert数据添加

```sql
1、指定字段添加值

   insert into 表名(字段名[,字段名...]) values (值1[，值2...])

   insert into test(username, age) values('lucky', 18);

2、不指定字段名插入数据（所有字段都插入值）

   insert into 表名 values (值1[，值2...])

   insert into test values(null, 'lucky', 18);

   注意：从左到右  依次对应   所有字段都要给值

3、指定字段添加多个值

   insert into 表名(字段名[,字段名...]) values (值1[，值2...]),(值1[，值2...])...

   insert into test(username, age) values('lucky', 18),('zhangsan', 20);

不4、指定字段名插入多条数据（所有字段都插入值）

   insert into 表名 values (值1[，值2...]),(值1[，值2...])...

   insert into test values(null, 'lucky', 18),(null, 'zhangsan', 20);

   注意：从左到右  依次对应   所有字段都要给值
```

## DELETE删除

+ 主体结构

  ```sql
  delete from 表名 [where]
  ```

+ 实例

  delete from user;   删除user表中所有数值

  自增值依然保持不变

+ 注意

  如果不添加where条件 则为删除所有

## UPDATE 修改

1. 修改一个字段的值

   update 表名 set 字段名=值 [where]

   update user set sex='w';

2. 修改多个字段的值

   update 表名 set 字段名1=值1, set字段名2=值2...  [where]

   update user set sex='w', age=20;

3. 在字段原值的情况下 改变值

   update user set age=age+2;

注意： 如果不添加where条件则会修改整个表的数据

## SELECT查询

1. 不指定字段查询(不建议)

   select * from 表名;

2. 指定字段查询数据

   select 字段1,字段2...  from 表名;

3. 对查询的字段起别名

   select 字段1 as 别名1,字段2...  from 表名;

   select 字段1 别名1,字段2...  from 表名;

   select username as u,age a from test9;

4. 给查询的结果添加新的字段

   select username as u,age a,'北京' as address from test9;