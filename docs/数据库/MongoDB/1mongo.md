# MongoDB

官方文档：https://docs.mongodb.com/

中文文档：https://www.mongodb.org.cn/

mongoDB的生态、理念非常先进而且成熟、但是mongoDB不仅有开源版本，还有企业版本。所以有部分公司比较担心，哪天无法使用mongoDB了。所以也会产生一些替代产品。

```
DynamoDB  : AWS
SequoiaDB : 巨杉数据库
```



## 基本介绍

MongoDB 是由C++语言编写并基于分布式文件存储的开源数据库，属于NOSQL 。

MongoDB 是一款介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的NOSQL数据库。它面向文档存储，而且安装和操作起来都比较简单和容易，而且它支持各种流行编程语言进行操作，如Python，Node.js，Java，C++，PHP，C#等。

目前在大数据、内容管理、持续交付、移动应用、社交应用、用户数据管理、数据中心等领域皆有广泛被使用。

### MongoDB相对于RDBMS的优势

游戏道具：

​	[{“id”:1, "name": "衣服", "防御值": 1000},

​	{“id”:2, "name": "武器", "攻击力": 9999,  "攻击距离": 50},

​	{“id”:3, "name": "鞋子", "移动速度": 1.1,  "颜色": "蓝色"},]

| id   | name | 防御值 | 攻击力 | 攻击距离 | 移动速度 | 颜色 |
| ---- | ---- | ------ | ------ | -------- | -------- | ---- |
| 2    | 武器 | None   | 9999   | 50       | None     | None |
| 1    | 衣服 | 1000   | None   | None     | None     | None |
| 3    | 鞋子 | None   | None   | None     | 1.1      | 蓝色 |



-   无固定结构 。
-   数据结构由键值(key=>value)对组成。MongoDB 的文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组，单个对象的结构是清晰的。
-   没有复杂的表连接。不需要维护表与表之间的内在关联关系。
-   查询功能强大。MongoDB的查询功能几乎与SQL一样强大，使用基于文档的查询语言，可以对文档进行动态查询。
-   易于调优和扩展。具备高性能、高可用性及可伸缩性等特性
-   应用程序对象与数据库对象天然对应。
-   可以基于内存存储或者硬盘文件存储，提供丰富的查询操作和索引支持，也有事务操作，可以更快地更稳定的访问数据。(mongoDB4.0以后才真正支持所谓的多文档事务操作)

### 术语对比

|       **SQL**       |     **Mongodb**     | 描述                                                 |
| :-----------------: | :-----------------: | ---------------------------------------------------- |
|   库（database）    |   库（database）    |                                                      |
|     表（Table）     | 集合（Collection）  |                                                      |
|   行/记录（Row）    |  文档（Document）   | Document就是json结构的一条数据记录                   |
|  列/字段（Column）  | 字段/键/域（Field） | 属性字段名                                           |
| 主键（Primary Key） | 对象ID（ObjectId）  | _id: ObjectId("10c191e8608f19729507deea")            |
|    索引（Index）    |    索引（Index）    | 也有普通索引, 唯一索引, 主键索引, 联合索引这么区分的 |

### 基本安装

目前ubuntu20.04 最新版本为4.4版本。需要完成以下命令步骤：

```bash
# 安装依赖包
sudo apt-get install -y libcurl4 openssl

# 关闭和卸载原有的mongodb
sudo systemctl stop mongod
sudo apt-get purge mongodb*
sudo apt-get auto-remove
sudo rm -r /var/lib/mongodb
sudo rm -r /var/log/mongodb

# 导入包管理系统使用的公钥
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
# 如果命令执行结果没有显示OK，则执行此命令在把上一句重新执行：sudo apt-get install gnupg

# 注册mongodb源
echo "deb https://mirrors.tuna.tsinghua.edu.cn/mongodb/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
# echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list

# 更新源
sudo apt-get update

# 安装mongodb
sudo apt-get install -y mongodb-org

# 等待上面的安装过程中，可以先创建mongoDB的数据存储目录
sudo mkdir -p /data/db
```

启动和关闭MongoDB

```bash
# 安装完成以后，默认情况下moongDB是没有启动的，需要我们手动加载数据库配置，并启动mongodb
sudo systemctl daemon-reload
sudo systemctl start mongod

# 查看运行状态
sudo systemctl status mongod
# 如果mongodb状态为stop，则运行 sudo systemctl enable mongod

# 停止mongodb
sudo systemctl stop mongod

# 重启mongodb
sudo systemctl restart mongod
```

进入交互终端

>   MongoDB安装完成后，默认是没有进行用户登录的权限验证，默认是不需要输入用户名密码即可登录的
>
>   也可以启动权限认证，但是必须注意：
>
>   mongodb默认是没有管理员账号的，所以要先切换到admin数据库添加管理员账号，再开启权限认证，否则就玩大了。

```bash
# 进入交互终端
mongo
```

终端效果如下：

```bash
MongoDB shell version v4.4.2
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("2c920d56-ddbb-4649-9191-a3bd4506a2d2") }
MongoDB server version: 4.4.2
---
The server generated these startup warnings when booting:
		# 警告：强烈建议使用XFS文件系统，并使用WiredTiger存储引擎。
		# 解释：因为当前ubuntu使用的是ext4文件系统，mongodb官方建议使用XFS文件系统功能更能发挥mongodb的性能，忽略不管
        2020-11-23T16:23:34.416+08:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
        # 警告：当前mongodb没有为数据库启用访问控制。对数据和配置的读写访问是不受限制的。
        # 解释：后面会创建数据库用户采用密码登陆的。暂时不用管
        2020-11-23T16:23:35.046+08:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
```

成功运行了mongoDB以后，对于mongod和mongo的区别

mongod是处理MongoDB系统的主要进程。主要负责处理数据请求，管理数据存储，和执行后台管理操作。当我们运行mongod命令意味着正在启动MongoDB进程, 并且在后台运行。

mongo是一个命令行实现的客户端操作mongodb的工具，用于连接一个特定的mongod实例。当我们没有带参数运行mongo命令它将使用默认的localhost:27017和mongod进行连接。

mongoDB的默认端口：27017



退出交互终端

```python
exit
# quit()
```

查看版本

```bash
mongo --version
# 或者终端内部使用 version()
```



## 基本操作

### 通用操作

#### 查看帮助文档

```javascript
help
```

终端效果：

```bash
db.help()                    help on db methods   查看操作数据库的方法
db.mycoll.help()             help on collection methods  查看当前数据库的mycoll集合的操作方法
sh.help()                    sharding helpers  查看分片集share的帮助信息
rs.help()                    replica set helpers 查看复制集的帮助信息
help admin                   administrative help 查看管理的操作帮助信息
help connect                 connecting to a db help
help keys                    key shortcuts
help misc                    misc things to know
help mr                      mapreduce

show dbs                     show database names  查看当前系统所有的数据库
show collections             show collections in current database 查看当前数据库所有的数据集合
show users                   show users in current database  查看当前数据库中所有的管理员用户
show profile                 show most recent system.profile entries with time >= 1ms
show logs                    show the accessible logger names 查看全部日志
show log [name]              prints out the last segment of log in memory, 'global' is default                          查看指定日志信息
use <db_name>                set current database   切换操作的数据库
db.mycoll.find()             list objects in collection mycoll  列出当前指定集合下的所有文档
db.mycoll.find( { a : 1 } )  list objects in mycoll where a == 1 按条件查询指定集合下所有文档
it                           result of the last line evaluated; use to further iterate
查看更多的查询结果，相当于下一页
DBQuery.shellBatchSize = x   set default number of items to display on shell
修改返回结果数据的单页显示数量，默认20条
exit                         quit the mongo shell  退出终端
```



#### 当前服务器状态

```javascript
db.serverStatus()
```

终端效果：

```bash
{
	"host" : "ubuntu",    # 主机名
	"version" : "4.4.11",  # mongodb版本
	"process" : "mongod", # mongodb进程，主要有mongod和mongos(分片集群中)两种
	"pid" : NumberLong(1034),  # mongod的pid进程号，可以在linux终端下使用命令 pidof mongod 验证
	"uptime" : 105063,    # mongodb服务启动的秒数，时间短则表示近期有重启，时间长则表示在稳定运行
	"uptimeMillis" : NumberLong(105063193), # mongod服务启动的毫秒数
	"uptimeEstimate" : NumberLong(105063),  # mongod内部自己计算的启动秒数
	"localTime" : ISODate("2020-12-08T16:01:08.230Z"), # 本地时间，相当于 python的 datetime
	# 连接数相关
	"connections" : {
		"current" : 1,  # 当前连接数
		"available" : 51199, # 可用最大连接数
		"totalCreated" : 1,  # 截止目前为止总共历史创建的连接数
		"active" : 1,   # 还在活跃的连接数，查看并发连接数主要参考这个值
	},
    # 全局锁相关信息
	"globalLock" : {
		"totalTime" : NumberLong("105063115000"), # mongod启动后到现在的总时间，单位微秒
		"currentQueue" : { # 当前等待锁队列
			"total" : 0,   # 当前全局锁的等待个数
			"readers" : 0, # 当前全局读锁等待个数
			"writers" : 0  # 当前全局写锁等待个数
		},
		"activeClients" : {
			"total" : 0,   # 当前活跃客户端的个数
			"readers" : 0, # 当前活跃客户端中进行读操作的个数
			"writers" : 0  # 当前活跃客户端中进行写操作的个数
		}
	},

	"network" : { # 网络相关
		"bytesIn" : NumberLong(1611),    # 数据库接收到的网络传输字节数
		"bytesOut" : NumberLong(51269),  # 从数据库发送出去的网络传输字节数
		"numRequests" : NumberLong(16),  # mongod接收到的总请求次数
	},

	# 操作计数器
	"opcounters" : {
		"insert" : NumberLong(0),  # 本次mongod实例启动至今收到的插入操作总数
		"query" : NumberLong(287), # 本次mongod实例启动至今收到的查询总数。
		"update" : NumberLong(0),  # 本次mongod实例启动至今收到的更新操作总数 。
		"delete" : NumberLong(0),  # 本次mongod实例启动至今的删除操作总数。
		"getmore" : NumberLong(0), # 本次mongod实例启动至今“getmore”操作的总数。
		"command" : NumberLong(588)# 本次mongod实例启动至今向数据库发出的命令总数 。
	},

	# 存储引擎,是MongoDB的核心组件, 负责管理数据如何存储在硬盘（Disk）和内存（Memory）上的组件
	# MongoDB 支持多种不同的存储引擎（Storage Engine），MongoDB支持的存储引擎有：WiredTiger，MMAPv1和In-Memory。
	# 1. WiredTiger，将数据持久化存储在硬盘文件中；从MongoDB 3.2 版本开始，成为MongDB默认存储引擎
	# 2. In-Memory，将数据存储在内存中
	# 3. MMAPv1，将数据持久化存储在硬盘文件中; MongoDB 3.2 版本以前的默认存储引擎

	# WiredTiger 是比MMAPv1更好用，更强大的存储引擎，WiredTiger的写操作会先写入缓存(Cache)中，并持久化到WAL(Write ahead log，写日志)，每60s或日志文件达到2GB时会做一次Checkpoint(检查点)，将当前数据进行持久化，产生一个新的快照。Wiredtiger连接初始化时，首先将数据恢复至最新的快照状态，然后根据WAL恢复数据，以保证存储可靠性。
	# Checkpoint，检测点。将内存中的数据变更冲刷到磁盘中的数据文件中，并做一个标记点。
	#             表示此前的数据表示已经持久存储在了数据文件中，此后的数据变更存在于内存和日志中.
	#             是一种让数据库redo（重做）和data（数据）文件保持一致的机制。
	#             并非Mongodb独有的，mysql中的InnoDB也有。

	"storageEngine" : {
		"name" : "wiredTiger",   # 当前mongoDB系统默认的存储引擎是 wiredTiger
		"supportsCommittedReads" : true,
		"oldestRequiredTimestampForCrashRecovery" : Timestamp(0, 0),
		"supportsPendingDrops" : true,
		"dropPendingIdents" : NumberLong(0),
		"supportsTwoPhaseIndexBuild" : true,
		"supportsSnapshotReadConcern" : true,
		"readOnly" : false,
		"persistent" : true,
		"backupCursorOpen" : false
	},

    # 多文档事务，mongodb4.0以后新增特性
	"transactions" : {
		"retriedCommandsCount" : NumberLong(0),
		"retriedStatementsCount" : NumberLong(0),
		"transactionsCollectionWriteCount" : NumberLong(0),
		"currentActive" : NumberLong(0),
		"currentInactive" : NumberLong(0),
		"currentOpen" : NumberLong(0),
		"totalAborted" : NumberLong(0),
		"totalCommitted" : NumberLong(0),
		"totalStarted" : NumberLong(0),
		"totalPrepared" : NumberLong(0),
		"totalPreparedThenCommitted" : NumberLong(0),
		"totalPreparedThenAborted" : NumberLong(0),
		"currentPrepared" : NumberLong(0)
	},
	"locks":{ # 锁相关

	},
	"mem" : { # 内存相关
		"bits" : 64, # 操作系统位数
		"resident" : 18,  # 物理内存消耗,单位：M
		"virtual" : 1566, # 虚拟内存消耗
		"supported" : true # 是否显示额外的内存信息
	},
}
```



#### 查看当前db的连接机器地址

```
db.getMongo()
```

#### 查看日志

```javascript
show logs
// global
// startupWarnings

// 如果要查看具体文件的日志。
show log global
// global是全局日志，默认保存的日志文件在 /var/log/mongodb/mongod.log
// 如果mongoDB无法启动，查看错误就可以来到默认日志文件查看
```



#### 数据备份与恢复

MongdoDB一共提供了4个命令行工具给我们对数据进行备份与恢复以及导入与导出数据。

备份与恢复操作的数据文件格式是bson格式，二进制文件，不需要进入mongo终端

导入与导出数据的数据文件格式是json格式，文本文档格式，不需要进入mongo终端



##### 准备测试数据

mongoDB终端下进行以下操作：

```javascript
use demo  // mongoDB中可以使用use切换数据库，针对不存在的数据库会自动创建，可通过db在终端查看当前所在数据库

// 声明了一个函数，格式化数字
fotmatnumber = (start, end)=>{
    num = Math.round(Math.random() * (end-start)) + start
    if(num<10){
        return "0"+num;
    }else{
        return num;
    }
}

// 声明了一个函数，生成随机标题
rand_title = (i)=>{
    num = Math.round( Math.random() * 10 );
    num1 = Math.round( Math.random() * 10 );
    return [
        "赠送礼品-"+num,
        "购物狂欢-"+num,
        "随便买买-"+num,
        "愉快购物-"+num,
        "赠送礼物-"+num,
        "商品购买-"+num,
        "买多送多-"+num,
        "买年货-"+num,
        "买买买买-"+num,
        "充值会员-"+num
    ][num1];
}

// 循环生成指定的数据
for(var i=0; i<200000; i++){
    // 往当前数据库的orders集合(相当于mysql的orders数据表) 添加1条数据
    db.orders.insert({
        "onumber": ( "0000000000000000" + i ).substr( String(i).length ),
        "date": "20"+fotmatnumber(0,21)+"-"+fotmatnumber(1,12)+"-"+fotmatnumber(1,31),
        "title": rand_title(i),
        "user_id": parseInt(i/200)+1,
        "items" :[{
        	"goods_id" : parseInt(i/200)+1,
        	"goods_attr" : i,
        	"price" : 100.0
        },{
        	"goods_id" : parseInt(i/200)+2,
        	"goods_attr" : i+1,
        	"price" : 80.0
        }]
    })
    // 判断循环过程中，i每逢1000则打印一次数据
    if(i%10000==0){
        print("已经添加了"+Math.ceil(i/10000)+"万条数据！");
    }
}
```



##### 数据备份

```bash
mongodump -h dbhost -d dbname -o dbdirectory
# 例如：备份上面的demo数据库
mongodump -h 127.0.0.1:27017 -d demo -o /home/moluo/Desktop/

# 删除demo数据库
use demo
db.dropDatabase()
```

参数说明：

| 选项 | 作用                          | 备注                           |
| ---- | ----------------------------- | ------------------------------ |
| -h   | MongoDB服务端地址和端口的简写 | --host=MongoDB地址 --port=端口 |
| -d   | 备份的数据库名称              | --db=数据库名称                |
| -o   | 备份数据保存目录              | output 目录需要提前创建        |



##### 数据恢复

```bash
mongorestore -h dbhost -d dbname --dir dbdirectory --drop
# 例如：恢复上面备份的demo数据库
mongorestore -h 127.0.0.1:27017 -d demo --dir /home/moluo/Desktop/demo

# 进入数据库查看
use demo
# 查看orders下面的表是否有20万数据
db.orders.count()
```

参数说明：

| 选项   | 作用                                | 备注                           |
| ------ | ----------------------------------- | ------------------------------ |
| -h     | MongoDB服务端地址和端口的简写       | --host=MongoDB地址 --port=端口 |
| -d     | 备份的数据库名称                    | --db=数据库名称                |
| --dir  | 备份数据所在目录                    |                                |
| --drop | 恢复数据前，先删除MongoDB中的旧数据 |                                |



##### 数据导出

```bash
mongoexport -h dbhost -d dbname -c collectionname -o file --type json/csv -f field
# 例如：导出demo数据库的orders集合的所有数据
mongoexport -d demo -c orders -o /home/moluo/Desktop/demo/orders.json --type json
```

参数说明：

| 选项   | 作用                          | 备注                                                         |
| ------ | ----------------------------- | ------------------------------------------------------------ |
| -h     | MongoDB服务端地址和端口的简写 | --host=MongoDB地址 --port=端口                               |
| -d     | 要导出的数据库名称            | --db=数据库名称                                              |
| -c     | 要导出的集合名称              | --collection=集合名称                                        |
| -o     | 导出数据保存的文件名          |                                                              |
| --type | 导出数据的文件格式            | 默认是json，也可以是csv，当数据格式为csv时，另需加上-f "字段1,字段2,...." |



##### 数据导入

```bash
mongoimport -h dbhost -d dbname -c collectionname --file filename --headerline --type json/csv -f field
# 例如：把上面导出的json文件中的数据，导入到demo数据库的orders集合中
mongoimport -d demo -c orders --file /home/moluo/Desktop/backup/orders.json --type json
```

参数说明：

| 选项   | 作用                          | 备注                                                         |
| ------ | ----------------------------- | ------------------------------------------------------------ |
| -h     | MongoDB服务端地址和端口的简写 | --host=MongoDB地址 --port=端口                               |
| -d     | 要导入的数据库名称            | --db=数据库名称                                              |
| -c     | 要导入的集合名称              | --collection=集合名称                                        |
| --file | 导入数据保存的文件名          |                                                              |
| --type | 导入数据的文件格式            | 默认是json，<br>也可以是csv，当数据格式为csv时：<br>1. 需加上-f "字段1,字段2,...."<br>2. 可以选择加上--headerline，设置首行为导入字段 |



### 用户管理

#### 创建用户

**db.createUser(user, pwd, writeConcern)**

创建一个数据库新用户用db.createUser()方法，如果用户存在则返回一个用户重复错误。

错误信息：`uncaught exception: Error: couldn't add user: User "用户名@数据库" already exists`

**语法**：

```javascript
db.createUser({
    user: "<用户名>",
    pwd: "<密码>",
	customData: { <any information> }, // 任意内容，主要是为了表示用户身份的相关介绍
	roles: [ // 角色和权限分配
		{ role: "<role>", db: "<database>" },  // 也可以直接填写由mongoDB内置的角色，例如: "<role>"
		...
    ]
})
```

>   mongo的用户是**以数据库为单位**来分别建立和管理的，每个数据库有属于自己的一个或多个管理员。
>
>   管理员可以管理自己的数据库，但是不能直接管理其他数据库，要先在内置数据库admin认证后才可以。
>
>   管理员的权限设置包含了2块，分别是角色和权限，由创建用户时通过roles属性进行设置。

##### 内置角色

```javascript
数据库用户角色：read、readWrite
数据库管理角色：dbAdmin、dbOwner、userAdmin
集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager
备份恢复角色：backup、restore
所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
超级用户角色：root
// 有几个角色间接或直接提供了系统超级用户的访问权限（dbOwner 、userAdmin、userAdminAnyDatabase、dbAdminAnyDatabase）
```

##### 内置权限

```
Read：允许用户读取指定数据库
readWrite：允许用户读写指定数据库
dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
root：只在admin数据库中可用。超级账号，擁有超级权限
```



##### 给Admin数据库创建账户管理员（相当于给mpongoDB创建了一个HR）

当前账号只能用于管理admin数据库账号，不能进行其他数据库的操作。

```javascript
// 进入/切换数据库到admin中
use admin
// 创建账户管理员
// [用户名:admin，密码:123456，数据库:admin，角色userAdminAnyDatabase，表示admin用户在admin数据库中具有超级用户权限]
db.createUser({
	user: "admin",
	pwd: "123456",
	roles: [
		{role: "userAdminAnyDatabase",db:"admin"},
	]
});

// 终端效果如下：
Successfully added user: {
	"user" : "admin",
	"roles" : [
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		}
	]
}
```



##### 创建超级管理员

当前账号可以进行数据库相关操作。

```javascript
// 进入/切换数据库到admin中
use admin
// 创建超级管理员账号
db.createUser({
    user: "root",
    pwd: "123456",
    roles: [
    	{role:"root", db:"admin"},  // 也可以这样简写："root",
    ]
})

// 终端效果如下：
Successfully added user: {
	"user" : "root",
	"roles" : [
		{
			"role" : "root",
			"db" : "admin"
		}
	]
}

```



##### 创建用户自己的数据库的角色

帐号是跟着数据库绑定的，所以是什么数据库的用户，就必须先到指定库里授权和验证！！！

一般开发中，往往一个项目就分配一个数据库，并给这个数据库分配一个管理员！！！

```javascript
// 切换数据库，如果当前库不存在则自动创建
use mofang;
// 创建管理员用户，为了保证不会报错，所以先删除同名管理员 db.system.users.remove({user:"mofang"});
db.createUser({
    user: "mofang",
    pwd: "123456",
    roles: [
        { role: "dbOwner", db: "mofang"}
    ]
})

// 终端下的效果
Successfully added user: {
	"user" : "mofang",
	"roles" : [
		{
			"role" : "dbOwner",
			"db" : "mofang"
		}
	]
}

// 查看当前仓库下的用户
show users;
// 效果：
{
	"_id" : "mofang.mofang",
	"userId" : UUID("126f2bd2-cddc-450a-9814-3e8937827ab3"),
	"user" : "mofang",
	"db" : "mofang",
	"roles" : [
		{
			"role" : "dbOwner",
			"db" : "mofang"
		}
	],
	"mechanisms" : [
		"SCRAM-SHA-1",
		"SCRAM-SHA-256"
	]
}

// 也可以在admin库中全局显示用户
use admin
db.system.users.find()
```



#### 用户信息

##### 查看当前数据库下的管理用户

只需要切换到对应的库中即可查看

```javascript
use mofang
show users
```

##### 查看系统中所有的用户

需要切换到admin中使用账号管理员的权限进行操作

```javascript
use admin
// db.auth("root","123456") // 如果开启了mongoDB的访问控制，就需要下用户认证密码才能继续往下操作
db.system.users.find() // 只能在admin数据库中使用。
db.system.users.find().pretty()  // pretty表示在返回多个查询结果时，以结构化格式显示数据
```



#### 删除用户

>   db.system.users.remove(json条件)

```javascript
// 有多种删除方式，下面是根据user用户名删除用户
use admin
db.system.users.remove({user:"mofang"});

// 删除效果：
WriteResult({ "nRemoved" : 1 })  // nRemoved 大于0表示成功删除管理员，等于0则表示没有删除。
```



#### 修改密码

必须切换到对应的数据库下才能给用户修改密码。

```js
db.changeUserPassword("账户名", "新密码")
```

mongo终端操作：

```javascript
use mofang
// 注册必须保证有这个管理员
db.changeUserPassword("mofang", "123")
```


