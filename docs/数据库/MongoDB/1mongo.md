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

所以针对账户管理员或者超级管理员，需要在admin下修改，而其他数据库管理员则必须到对应数据库下才能修改。

```js
db.changeUserPassword("账户名", "新密码");
```

mongo终端操作：

```javascript
use mofang
// 注册必须保证有这个管理员
db.changeUserPassword("mofang", "123");
```



#### 开启账户认证

开启账户认证功能，必须要修改配置文件，然后重启mongoDB才能生效。

```javascript
sudo vim /etc/mongod.conf
// 找到31行附近的 security，去掉左边注释符号(#)
security:
    authorization: enabled

:wq
// 重启mongdb，配置生效
sudo systemctl restart mongod

// 开启了账户认证机制以后，再次进入mofang
mongo
use mofang
show users    // 此处会报错如：uncaught exception: Error: command usersInfo requires authentication
db.auth("mofang","123")   // 此处认证时填写错误密码，报错如下：
// Error: Authentication failed.
// 0
exit
mongo
db.auth("mofang","123456")  // 此处认证时填写正确密码，效果如下：
// 1
show users    // 此时经过认证以后，当前命令就不会被限制了。 


// 注意：如果实现以某个库的账户管理员登录数据库以后，要切换账号操作其他数据库，则必须先退出当前登录状态。
```



### 库管理

+ 显示所有数据库列表【空数据库不会显示，或者说空数据库已经被mongoDB回收了。】

  ```javascript
  show dbs
  show databases
  ```

+ 切换数据库，如果数据库不存在则创建数据库。

  ```javascript
  use  <database>
  ```

+ 查看当前工作的数据库

  ```javascript
  db   // 是 db.getName() 的简写
  ```

+ 删除当前数据库，如果数据库不存在，也会返回`{"ok":1}`

  ```javascript
  use <db>          // 先切换到要删除的数据库中，然后才能删除数据库
  db.dropDatabase()
  ```

  

+ 查看当前数据库状态

  ```json
  > db.stats()
  
  {
  	"db" : "mofang",    // 当前数据库名
  	"collections" : 0,  // 当前数据库中的数据集合数量，相当于mysql的数据表
  	"views" : 0,        // 当前数据库中的视图数量
  	"objects" : 0,      // 当前数据库中的文档数据，相当于mysql中的数据记录
  	"avgObjSize" : 0,   // 当前数据库中的文档平均大小
  	"dataSize" : 0,     // 当前数据库中的数据总文件大小
  	"storageSize" : 0,  // 存储引擎占据的文件大小
  	"totalSize" : 0,    // 数据库中总数据总文件大小
  	"indexes" : 0,      // 当前数据库中的索引数量
  	"indexSize" : 0,    // 当前数据库中的索引文件大小
  	"scaleFactor" : 1,  // 
  	"fileSize" : 0,
  	"fsUsedSize" : 0,   // 文件系统空间占用大小
  	"fsTotalSize" : 0,  // 文件系统的总占用空间大小
  	"ok" : 1
  }
  
  ```

在mongoDB中，最重要的核心是文档，如果一个库或者一个库下的集合中的文档全部被删除了，则这个库和这个集合就会mongoDB回收删除。

### 集合管理

#### 创建集合

mongoDB中的集合有2种：固定集和动态集。

一般工作中使用的是动态集，但是在mongoDB优化时，可以对部分数据转换成使用固定集来保存，性能更好，查询速度更快。

在mongodb中如果使用的动态集，其实不需要专门创建集合，直接添加文档，mongodb也会自动生成动态集合的。

```javascript
// name为必填参数，options为可选参数。capped若设置值为true，则size必须也一并设置
db.createCollection(
	<集合名称>, 
	{ 
		capped : <boolean>,       // 当前创建的集合是否是固定集，固定集指限制固定数据大小的集合，当数据达到最大值会自动覆盖最早添加的文档内容
		size : <bytes_size>,      // 指定固定集合存储的最大字节数，单位：字节数.
		max : <collection_size>   // 指定固定集合中包含文档的最大数量，单位：字节数
	}
);

// db.createCollection不填写第二个参数则表示创建的是动态集
// 添加文档到不存在的集合中，mongodb会自动创建动态集合，
// db.<集合名称>.insert({"name":"python入门","price" : 31.4})
db.courses.insert({"name":"python入门","price" : 31.4})
```



#### 固定集的使用操作

固定集一般用于日志，历史记录中。

```javascript
// 创建固定集history
> db.createCollection("history", {capped:true, max: 10, size: 3000});
{ "ok" : 1 }

// 可以通过show tables 直接查看当前数据库中所有的集合列表
> show tables;
history
orders

// 添加数据到固定集
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })
> db.history.insert({"name":"python入门","price" : 31.4})
WriteResult({ "nInserted" : 1 })

// 上面一共添加了14条数据到固定集，当时因为创建固定时设置了只允许10条数据，
// 所以固定集中针对旧的数据已经删除，只保留最新的10数据。
> db.history.find()
{ "_id" : ObjectId("61d7eb6624402081a70c16fb"), "name" : "python入门", "price" : 31.4 }
{ "_id" : ObjectId("61d7eb6624402081a70c16fc"), "name" : "python入门", "price" : 31.4 }
{ "_id" : ObjectId("61d7eb6624402081a70c16fd"), "name" : "python入门", "price" : 31.4 }
{ "_id" : ObjectId("61d7eb6724402081a70c16fe"), "name" : "python入门", "price" : 31.4 }
{ "_id" : ObjectId("61d7eb6724402081a70c16ff"), "name" : "python入门", "price" : 31.4 }
{ "_id" : ObjectId("61d7eb6724402081a70c1700"), "name" : "python入门", "price" : 31.4 }
{ "_id" : ObjectId("61d7eb6724402081a70c1701"), "name" : "python入门", "price" : 31.4 }
{ "_id" : ObjectId("61d7eb6824402081a70c1702"), "name" : "python入门", "price" : 31.4 }
{ "_id" : ObjectId("61d7eb6824402081a70c1703"), "name" : "python入门", "price" : 31.4 }
{ "_id" : ObjectId("61d7eb6924402081a70c1704"), "name" : "python入门", "price" : 31.4 }
> 

```



#### 集合列表

```javascript
show collections // 或 show tables   或 db.getCollectionNames()
```

#### 删除集合

```javascript
db.集合.drop()
```

#### 查看集合

```javascript
db.getCollection("集合名称")
db.集合名称
```

##### 查看集合创建信息

```javascript
db.printCollectionStats()

// 运行效果：
orders
{
	"ns" : "demo.orders",  // ns 表示namespace，当前集合的具体名字
	"size" : 49761760,     // 当前集合的文件大小
	"count" : 200000,      // 当前集合的文档数量
	"avgObjSize" : 248,    // 平均每个文档的文件大小
	"storageSize" : 8900608,  // 当前集合中存储引擎的文件大小
	"freeStorageSize" : 0,
	"capped" : false,      // 是否是固定集,false表示当前集合是动态集
	"wiredTiger" : {       // 存储引擎相关配置
		
	},
	"nindexes" : 1,              // 当前集合的索引数量
	"indexBuilds" : [ ],   
	"totalIndexSize" : 2301952, // 当前集合的索引文件大小 
	"totalSize" : 11202560,     // 当前集合的全部数据
	"indexSizes" : {            // 当前集合的每个索引的文件大小
		"_id_" : 2301952
	},
	"scaleFactor" : 1,
	"ok" : 1
}

```



### 文档管理

mongodb中，文档也叫 object/document。对应的就是存储的json数据记录，对应的就是python中的字典或者列表。

mongodb中，文档中有各种的自定义字段，，每一个字典对应的数据值也存在不同数据格式，根据不同的格式产生不同的数据类型。

#### 数据类型

| Type               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| **ObjectID**       | 用于存储文档的ID,相当于主键，区分文档的唯一字段，mongoDB中就是一个对象的返回值 |
| **String**         | 字符串是最常用的数据类型，MongoDB中的字符串必须是UTF-8编码。 |
| **Integer**        | 整数类型用于存储数值。整数可以是32位，也可以是64位，这取决于你的服务器。 |
| **Double**         | 双精度类型用于存储浮点值,mongodb中没有float浮点数这个说法    |
| **Boolean**        | 布尔类型用于存储布尔值(true/ false)                          |
| **Arrays**         | 将数组、列表或多个值存储到一个键，[]                         |
| **Timestamp**      | 时间戳，用于记录文档何时被修改或创建。Date()，Timestamp()，ISODate() ,默认是ISODate() |
| **Object**         | 用于嵌入文档, 相当于子属性是另一个json文档而已，这种方式就可以实现嵌套。 [] |
| **Null**           | 空值,相当于 python的None                                     |
| Symbol             | 与字符串用法相同，常用于某些使用特殊符号的语言，二进制格式字符串 |
| Date               | 用于以UNIX时间格式存储当前日期或时间。                       |
| **Binary data**    | 二进制数据，常用于保存文件的内容，往往是图片，数据本身。     |
| Code               | 用于将JavaScript代码存储到文档中                             |
| Regular expression | 正则表达式                                                   |



#### 添加文档

mongodb中，文档的数据结构和 JSON 基本一样。所有存储在集合中的数据在内部存储的格式都是 BSON 格式。

BSON 是一种类似 JSON 的二进制形式的存储格式，是 Binary JSON 的简称。

```javascript
// 添加文档
// 方式1：
db.<集合名称>.insert(<document>)  // document就是一个json格式的数据

// 方式2
db.<集合名称>.insertOne(          // 如果文档存在_id主键为更新数据，否则就添加数据。
   <document>
)

// 方式3：
// 一次性添加多个文档, 多次给同一个集合建议使用insertMany比insertOne效率更好
db.<集合名称>.insertMany([
    <document>,
    <document>,
    ...
])
```

操作：

```javascript
use mofang;

// 添加一条数据[insert是过去版本的MongoDB提供的添加数据方法]
db.user_list.insert({
    "name": "laoli",  // string
    "age":33,         // integer
    "sex":true,       // boolean
    "child": {        // opject
        "name":"xiaohuihui",
        "age":3
    }
});
// WriteResult({ "nInserted" : 1 })

/*
// mongoDB原则上内置了js解释引擎，所以支持js语法
> db.user_list.find()[0]._id
ObjectId("61552b913ccd8ec29dbf6512")
> db.user_list.find()[0].name
laoli

// javascrit总可以通过typeof 来查看数据的类型
> typeof db.user_list.find()[0].name
string
> typeof db.user_list.find()[0]._id
object
> typeof db.user_list.find()[0].sex
boolean
> typeof db.user_list.find()[0].age
number
> typeof db.user_list.find()[0].child
object
> typeof db.user_list.find()[0].child.name
string
*/

// 添加一条数据
db.user_list.insertOne({"name":"xiaozhang","age":18,"sex":true, money: 300.50});
// {
// 	"acknowledged" : true,
// 	"insertedId" : ObjectId("605021e6d5c7a55cc95c1cb7")
// }


// 添加多条数据
document1 = {"name":"xiaolan","age":16}
document2 = {"name":"xiaoguang","age":16}
db.user_list.insertMany([document1,document2]);

// {
// 	"acknowledged" : true,
// 	"insertedIds" : [
// 		ObjectId("60502235d5c7a55cc95c1cba"),
// 		ObjectId("60502235d5c7a55cc95c1cbb")
// 	]
// }
db.user_list.find()
```



#### 删除文档

```json
// 方式1
db.<集合名称>.remove(
   <query>,  // removed的条件，一般写法：{"属性":{条件:值}}，如果不填写条件，删除所有文档
   {
     justOne: <boolean>,      // 可选删除，是否只删除查询到的第一个文档，默认为false，删除所有
     writeConcern: <document> // 可选参数，抛出异常的级别。
   }
)

// 方式2: 删除一条数据
db.<集合名称>.deleteOne(
   <query>,  // removed的条件，一般写法：{"属性":{条件:值}}，如果不填写条件，删除所有文档
   {
     justOne: <boolean>,      // 可选删除，是否只删除查询到的第一个文档，默认为false，删除所有
     writeConcern: <document> // 可选参数，抛出异常的级别。
   }
)
// 方式3：删除多条数据
db.<集合名称>.deleteMany(
   <query>,  // removed的条件，一般写法：{"属性":{条件:值}}，如果不填写条件，删除所有文档
   {
     justOne: <boolean>,      // 可选删除，是否只删除查询到的第一个文档，默认为false，删除所有
     writeConcern: <document> // 可选参数，抛出异常的级别。
   }
)
```

操作：

```javascript
// 添加多条测试数据
document1 = {"name":"xiaohei","age":16}
document2 = {"name":"xiaobai","age":16}
db.user_list.insertMany([document1,document2]);

// 删除满足条件的第一条数据
// mongoDB中的条件格式: {字段名:{$运算符:值}
// 条件：{"age":{$eq:16}}   相当于SQL语句的age=16
// db.user_list.remove({"age":{$eq:16}},{"justOne":true})
// 删除满足条件的所有数据，条件中$wq可以不写
// db.user_list.remove({"age":16}); // 等于可以省略不写，相当于 db.user_list.remove({"age":{$eq:16}});


// 再次 添加多条测试数据
document1 = {"name":"xiaolan","age":16}
document2 = {"name":"xiaoguang","age":16}
document3 = {"name":"xiaohei","age":16}
document4 = {"name":"xiaobai","age":16}
db.user_list.insertMany([document1,document2,document3,document4]);

// 删除一条
db.user_list.deleteOne({"age":16})
// db.user_list.deleteOne({"age":{$eq:16}})
// 删除多条
db.user_list.deleteMany({"age":16})
// db.user_list.deleteMany({"age":{$eq:16}})
```

mongoDB中的条件，不仅用于删除数据的条件，也可以是查询或者更新的条件。



#### 查询文档

```json
// 直接显示查询的所有，find和findOne的第二个参数，也是一个json对象，一般称之为字段投影，表示设置是否显示或隐藏指定数据字段。

// 获取一条
db.集合.findOne(
	<query>，     // 查询条件，删除、查询、修改都需要条件、条件写法基本一样的。
    {             // 查询结果的字段投影，用于指定查询结果以后，显示的字段列
    	<key>: 0, // 隐藏指定字段，例如："_id":0,
    	<key>: 1, // 显示指定字段，例如："title":1,
    	....
    }
)

// 获取多条
db.集合.find(
	<query>,      // 查询条件
    {
    	<key>: 0, // 隐藏指定字段，例如："_id":0,
    	<key>: 1, // 显示指定字段，例如："title":1,
    	....
    }
)

// 以易读的方式来格式化显示读取到的数据，只能在find方法后面使用。
db.集合.find().pretty()
```

操作：

```javascript
// 切换数据库
use mofang;

// 添加测试数据
docs = [
    {"name":"xiaohuang","sex":0,"age":15,"mobile":"13301234568"},
    {"name":"xiaofei","sex":1,"age":16,"mobile":"1351234568"},
    {"name":"xiaolong","sex":1,"age":19,"mobile":"15001234568"},
    {"name":"xiaomianyang","sex":0,"age":13,"mobile":"15001234568"}
]
db.user_list.insertMany(docs);

// 查询一条数据
db.user_list.findOne()    // 获取集合中第一条数据
db.user_list.findOne({})  // 同上
db.user_list.findOne({},{_id:0,child:0})    // 获取集合中第一条数据，并隐藏_id 和 child属性的数据
db.user_list.findOne({},{_id:0,name:1,mobile:1})  // 获取集合中第一条数据，只查询文档的name和mobile属性的数据
db.user_list.findOne({name:"xiaolong"},{_id:0,name:1,mobile:1})
db.user_list.findOne({name:"xiaoming", age:18})

// 查询多条数据
db.user_list.find()    // 获取集合的所有数据
db.user_list.find({})  // 同上
db.user_list.find().pretty()
db.user_list.find({sex:1}) // db.user_list.find({sex:{$eq:1}})
db.user_list.find({sex:0},{_id:0,name:1,mobile:1})
```



##### 条件运算符

mongoDB中，条件运算符也叫查询器，queryter

###### 比较运算

| 操作       | 格式                                     | 语法例子                              | SQL中的类似语句           |
| :--------- | :--------------------------------------- | :------------------------------------ | :------------------------ |
| 等于       | `{<key>:<val>`}<br>`{<key>:{$eq:<val>}}` | `db.集合.find({"name":"xiaoming"})`   | `where name = 'xiaoming'` |
| 小于       | `{<key>:{$lt:<val>}}`                    | `db.集合.find({"age":{$lt:17}})`      | `where age  < 17`         |
| 小于或等于 | `{<key>:{$lte:<val>}}`                   | `db.集合.find({"age":{$lte:17}})`     | `where age  <= 17`        |
| 大于       | `{<key>:{$gt:<val>}}`                    | `db.集合.find({"age":{$gt:17}})`      | `where age  > 17`         |
| 大于或等于 | `{<key>:{$gte:<val>}}`                   | `db.集合.find({"age":{$gte:17}})`     | `where age  >= 17`        |
| 不等于     | `{<key>:{$ne:<val>}}`                    | `db.集合.find({"age":{$ne:17}})`      | `where age != 17`         |
| 包含       | `{<key>:{$in:[<val>...]}}`               | `db.集合.find({"age":{$in:[1,2,3]}})` | `where age in (1,2,3)`    |

终端运行

```javascript
db.user_list.insertOne({"name":"laowang"});
db.user_list.find({"age":{$lte:18}})    // 注意：没有当前字段的文档是不会被查询出来，因为默认条件不成立
db.user_list.find({"age":{$gte:18}})
db.user_list.find({"age":{$in:[16,33]}})
db.user_list.find({"sex":{$in:[1,true]}})

// 添加测试数据
db.user_list.insert({"name":"laowang","age":32,"sex":true,"child": {"name":"xiaowang","age":4}});
db.user_list.insert({"name":"laozhang","age":33,"sex":true,"child": {"name":"xiaozhang","age":5}});
db.user_list.find({"child.age":{$gte:3}})
db.user_list.find({"child.age":{$in:[3,5]}})

// 嵌套文档的查询：订单评价, 
```



###### 逻辑运算

| 操作                                                         | 语法                                                         | 语法例子                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `$and`                                                       | `{<key>:<val>,<key>:<val>,...}`<br>`{$and: [{key:{$运算符:<val>}},....]}` | db.集合.find({key1:value1, key2:value2})                     |
| `$or`                                                        | `{$or: [{<key>: {$运算符:<val>}}, ....]}`                    | db.集合.find({$or: [{key1: value1}, {key2:value2}]})         |
| `$and`和`$or`                                                | `{<key>:<val>, $or: [{<key>: {<$运算符>:<val>}},...]}`<br>`{$and:[{$or:[{<key>:{<$运算符>:<val>}},..]},$or:[{<key>:{<$运算符>:<val>}},..]}]}` | db.集合.find({key1:value1, $or: [{key1: value1}, {key2:value2}]}) |
| $not          | `{<key>:{$not:{<$运算符>:<val>}}}`                           | db.集合.find({key1:{$not:{$运算符: val1}}}) |                                                              |                                                              |

```javascript
SQL:
	class=1 and sex =true  or class=2 and sex=false
mongo:
	{$or: [
        {$and: [{class:1}, {sex:true}]},
        {$and: [{class:2}, {sex:false}]},
    ]}

```



终端操作：

```javascript
// 查询age=18 并且 sex=true
db.user_list.find({
    $and:[
        {"age":{$eq:18}},
        {"sex":{$eq:true}}
    ]
})
// 简写：
db.user_list.find({
    $and:[
        {"age":18},
        {"sex":true}
    ]
})
// 继续简写；
db.user_list.find({ "age":18, "sex":true })

// 查询age=16或者age=18
db.user_list.find({
    $or:[
        {"age":{$eq:16}},
        {"age":{$eq:18}}
    ]
})
// 简写：
db.user_list.find({
    $or:[
        {"age":16},
        {"age":18}
    ]
})

// 查询年龄!=16的
db.user_list.find({"age":{$not:{$eq:16}}})
// 简写：
db.user_list.find({"age":{$ne:16}})


// 查询age=33的男生 或者 age=18的男生
db.user_list.find({
    "sex":true,
    $or:[
        {"age":18},
        {"age":33}
    ]
});

db.user_list.find({
    "sex":true,
    "age":{
    	$in:[18,33]
 	}
});

db.user_list.find({
    $or:[
        {$and:[{"sex":true},{"age":18}]},
        {$and:[{"sex":true},{"age":33}]},
    ]
});

db.user_list.find({
    $or:[
        {"sex":true,"age":18},
        {"sex":true,"age":33},
    ]
});


// 查询 age=19,手机号码以150开头 或者 儿子年龄>=4岁的男生，
// 判断手机以指定字符开头可以使用$regex

db.user_list.find({
    $or: [
        {$and: [{age:19}, {mobile:{$regex:/^150/}}]},
        {$and: [{"child.age":{$gte:4}}, {sex: true}]},
    ]
})


// 简写
db.user_list.find({
	$or: [
		{$and: [{age:19}, {mobile:{$regex:/^150/}}]},
		{"child.age":{$gte:4}, sex: true},
	]
})


// 再次简写
db.user_list.find({
     $or: [
         {age:19, mobile:{$regex:/^150/}},
         {"child.age":{$gte:4}, sex: true},
     ]
})

```



###### 其他运算符

| 操作    | 格式                                                         | 语法例子                                  | 说明                                                         |
| ------- | ------------------------------------------------------------ | ----------------------------------------- | ------------------------------------------------------------ |
| $type   | `{<key>:{$type: <datetype>}}`                                | `db.集合.find({"name":{$type:'string'}})` | 匹配指定键是指定数据类型的文档<br>number 数值型<br>string 字符串<br>bool 布尔类型<br>object json文档对象类型<br>array 数组类型 |
| $exists | `{<key>:{$exists:<bool>}`                                    | `db.集合.find({"title":{$exists:true}})`  | 匹配具有指定键的文档，存在指定字段的文档                     |
| $regex  | `{ <key>:/模式/<修正符>}`<br>`{<key>:{$regex:/模式/<修正符>}}` | `db.集合.find({"name":{$regex:/张$/}})`   | 按正则匹配                                                   |
| $mod    | `{<key>: {$mod: [除数, 余数]}}`                              | `db.集合.find({"age":{$mod:[10,0]}})`     | 算数运算，取模，语法中举例是age除以10==0                     |
|         |                                                              |                                           |                                                              |

终端操作：

```javascript
db.user_list.insert({"name":"xiaoming","sex":0,"age":"18"});
db.user_list.insert({"name":"xiaoming","sex":1,"age":"18"});
db.user_list.insert({"name":"xiaoming","sex":1,"age":"33"});
db.user_list.insert({"name":"xiaoming","sex":0,"age":"33"});
// $type
db.user_list.find({"sex":{$type:"number"}});
db.user_list.find({"age":{$type:"string"}});


// $exists
db.user_list.find({"child":{$exists:true}});

// $regex 正则匹配
db.user_list.insert({"name":"xiaoming","sex":0,"age":"18","mobile":"13301234568"});
db.user_list.insert({"name":"xiaoming","sex":1,"age":"18","mobile":"1351234568"});
db.user_list.insert({"name":"xiaoming","sex":1,"age":"33","mobile":"15001234568"});
db.user_list.insert({"name":"xiaoming","sex":0,"age":"33","mobile":"15001234568"});

// 符合手机格式
db.user_list.find({"mobile":{$regex: /1[3-9]\d{9}/ }});
// 不符合手机号码格式的
db.user_list.find({"mobile":{$not:{$regex: /1[3-9]\d{9}/ }}});


// $mod
db.user_list.find({"age":{$mod: [3,0] }});
```



###### 自定义条件函数

慎用！！！效率差

```json
// 用法1，逻辑比较复杂的情况，可以使用更多的javascript进行运算处理：结果函数结果为true，则当前数据被查询出来。
db.<集合名称>.find({$where: ()=>{   // this代表的就是查询过程中，被循环的每一条文档数据
    return <this.字段> <$运算符> <条件值>;
}}});

// 用法2，相对没那么复杂的，取函数的返回值作为条件值:
db.集合.find({$where:"<this.字段> <运算符> <条件值>"});
// db.集合.find({$where:"this.name=='xiaoming'"});
```

操作：

```javascript
db.user_list.find({$where: ()=>{
    return this.name=="xiaoming" && this.age<30;
}});

// 把字符串作为代码条件执行，当结果为true，则返回当前符合的数据
db.user_list.find({$where: "this.name=='xiaoming' && this.age>30"});
```



##### 排序显示

```json
db.集合.find().sort({<key>:1})  // 升序，默认为升序
db.集合.find().sort({<key>:-1}) // 倒序， 
```

终端操作：

```javascript
db.user_list.find().sort({age:-1});
db.user_list.find().sort({age:-1, sex:1});
```



##### 字段投影

`find()`方法默认将返回文档的所有数据，但是可以通过设置`find()`的第二个参数projection，设置值查询部分数据。

语法：

```json
// 获取一条
db.集合.findOne(
	<query>，     // 查询条件
    {
    	<key>: 0, // 隐藏指定字段，例如："_id":0,
    	<key>: 1, // 显示指定字段，例如："title":1,
    	....
    }
)
// 获取多条
db.集合.find(
	<query>,      // 查询条件
    {
    	<key>: 0, // 隐藏指定字段，例如："_id":0,
    	<key>: 1, // 显示指定字段，例如："title":1,
    	....
    }
)
```

操作：

```javascript
> db.user_list.find({"mobile":{$regex:/^133\d{8}$/}},{"_id":0}).sort({"mobile":-1})
    { "name" : "xiaoming", "mobile" : "13333355678" }
    { "name" : "xiaoming", "mobile" : "13333345678" }
    { "name" : "xiaoming", "mobile" : "13312345678" }

> db.user_list.find({"mobile":{$regex:/^133\d{8}$/}},{"_id":0,"name":0}).sort({"mobile":-1})
    { "mobile" : "13333355678" }
    { "mobile" : "13333345678" }
    { "mobile" : "13312345678" }

> db.user_list.find({"mobile":{$regex:/^133\d{8}$/}},{"name":1}).sort({"mobile":-1})
    { "_id" : ObjectId("60502fb7d5c7a55cc95c1cc4"), "name" : "xiaoming" }
    { "_id" : ObjectId("60502fb4d5c7a55cc95c1cc3"), "name" : "xiaoming" }
    { "_id" : ObjectId("60502fb1d5c7a55cc95c1cc2"), "name" : "xiaoming" }

> db.user_list.find({"mobile":{$regex:/^133\d{8}$/}},{"name":1,"_id":0}).sort({"mobile":-1})
    { "name" : "xiaoming" }
    { "name" : "xiaoming" }
    { "name" : "xiaoming" }
```



##### 限制与偏移

`limit`方法用于限制返回结果的数量

`skip`方法用于设置返回结果的开始位置

```json
db.集合.find(...).limit(结果数量).skip(开始下标)
```

终端操作：

```javascript
db.user_list.find({},{"_id":0,"name":1,"age":1}).sort({"age":1}).limit(5);
db.user_list.find({},{"_id":0,"name":1,"age":1}).sort({"age":1}).limit(5).skip(0);

db.user_list.find({},{"_id":0,"name":1,"age":1}).sort({"age":1}).limit(5).skip(5);
```



#### 更新文档

```json
// 更新数据
db.集合.update(
    <query>,
    <update>,
    {
     upsert: <boolean>, // 可选参数，如果文档不存在，是否插入objNew, true为插入，默认是false，不插入
     multi: <boolean>,  // 可选参数，是否把满足条件的所有数据全部更新，设置更新1条还是多条
     writeConcern: <document> // 可选参数，抛出异常的级别。
   }
)

// 更新一条
db.集合.updateOne(
   <query>,   // update的查询条件，一般写法：{"属性":{条件:值}}
   <update>,  // update的更新数据，一般写法 { $set:{"属性":"值",....} } 或者 { $inc:{"属性":"值"} }
   {
     upsert: <boolean>, // 可选参数，如果文档不存在，是否插入objNew, true为插入，默认是false，不插入
     multi: <boolean>,  // 可选参数，是否把满足条件的所有数据全部更新，设置更新1条还是多条
     writeConcern: <document> // 可选参数，抛出异常的级别。
   }
)

// 更新多条
db.集合.updateMany(
   <query>,   // update的查询条件，一般写法：{"属性":{条件:值}}
   <update>,  // update的对象，一般写法 { $set:{"属性":"值"} } 或者 { $inc:{"属性":"值"} }
   {
     upsert: <boolean>, // 可选参数，如果文档不存在，是否插入objNew, true为插入，默认是false，不插入
     multi: <boolean>,  // 可选参数，是否把满足条件的所有数据全部更新
     writeConcern: <document> // 可选参数，抛出异常的级别。
   }
)
```



##### update更新运算符[修改器]

| 操作       | 语法                                                         |                                                              |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `$inc`     | `db.集合.update({<key1>:<val1>},{$inc:{<key2>:<val2>}})`     | 更新key1=val1的文档中key2的值为val2，类似python的递增递减<br>递减，则`{ $inc:{<key2>:-<val2>} }` |
| `$set`     | `db.集合.update({<key1>:<val>}, {$set:{<key2>:<val2>}})`     | 更新key1=val1的文档中key2的值为val2，如果key2不存在则新增对应键值对 |
| `$unset`   | `db.集合.update({<key1>:<val>}, {$unset:{<key2>:<val2>}})`   | 移除key1=val1的文档中key2=val2这个键值对                     |
| `$push`    | `db.集合.update({<key1>:<val>}, {$push:{<key2>:<val2>}})`    | 给key1=val1的文档中key2列表增加1个数组成员val2。<br>key2必须是数组。 |
| `$pull`    | `db.集合.update({<key1>:<val>}, {$pull:{<key2>:<val2>}})`    | 与push相反，给key1=val1的文档中key2列表删除1个指定成员val2   |
| `$pullAll` | db.集合.update({<key1>:<val>}, {$pullAll:{<key2>:[<val2>,<val3>]}}) | 与$pull作用一样，用于删除多个指定成员 |                                                              |
| `$pop`     | `db.集合.update({<key1>:<val>}, {$pop:{<key2>:<val2>}})`     | 给key1=val1的文档中key2列表移除第一个或最后一个成员。<br>val2只能是1(最后面)或-1(最前面)，与python相反 |

终端操作：

```javascript
// $inc
// 把laoli的年龄+10岁
db.user_list.update({"name":"laoli"},{$inc:{"age":10}}); // 更新一条
db.user_list.updateMany({"name":"xiaoming"},{$inc:{"age":10}}); // 更新多条
// 把laoli的孩子年龄+10岁
db.user_list.update({"name":"laoli"},{$inc:{"child.age":10}});


// $set
// 如果字段不存在，则新增字段的键值对，如果字段存在，则修改字段的值
//更新laoli的手机号码
db.user_list.update({"name":"laoli"},{$set:{"mobile":"18012312312"}}); // 更新一条
// 更新laoli孩子的手机号码
db.user_list.update({"name":"laoli"},{$set:{"child.mobile":"18012312312"}});

// $unset
// 移除laoli的性别键值对
db.user_list.update({"name":"laoli"},{$unset:{"sex":true}});

// $push
db.user_list.update({"name":"laoli"},{$set:{"lve":["TV","game"]}});
db.user_list.update({"name":"laoli"},{$push:{"lve":"code"}}); // 往列表属性中追加成员

// $addToSet 结合 $each 把一个数组中每一个成员添加到数组中
db.user_list.update({"name":"laoli"},{$addToSet:{"lve":{$each:["code","music","TV"]}}});

// $pull
db.user_list.update({"name":"laoli"},{$pull:{"lve":"TV"}});

// $pullAll
db.user_list.update({"name":"laoli"},{$pullAll:{"lve":["TV","game"]}});


// $pop
db.user_list.update({"name":"laoli"},{$pop:{"lve":-1}}); // 左边移除列表的第一个成员
db.user_list.update({"name":"laoli"},{$pop:{"lve":1}}); // 右边移除列表的最后一个成员

// $rename 字段名重命名
db.user_list.update({"name":"laoli"},{$rename:{"lve":"love"}});
```



### 索引操作

前面学习过MySQL，我们知道数据库里给数据构建索引通常能够**极大的提高数据查询的效率**，缩短查询耗时，如果没有索引，数据库在查询数据时必然会扫描数据表中的每个记录并提取那些符合查询条件的记录。同理，在MongoDB中构建索引也可以提高数据的查询效率和缩短查询耗时，没有索引的情况也是一样，MongoDB也会再查询数据时扫描集合中的每个文档并提取符合查询条件的文档。这种扫描全集合的查询效率是无疑是非常低下的，特别在处理大量的集合数据时，查询时间可能会达到几十秒甚至几分钟，这对用户体验来说是非常致命的。

文档：https://docs.mongodb.com/manual/indexes/#default-id-index

#### 准备数据

```javascript
use demo
fotmatnumber = (start, end)=>{
    num = Math.round(Math.random() * (end-start)) + start
    if(num<10){
        return "0"+num;
    }else{
        return num;
    }
}

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

for(var i=0; i<200000; i++){  
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
    if(i%10000==0){
        print("已经添加了"+parseInt(i/10000)+"万条数据！");
    }
}
```



#### 注意事项

1. MongoDB的索引是存储在运行内存(RAM)中的，所以必须确保索引的大小不超过内存的限制。

   如果索引的大小超过了运行内存的限制，MongoDB会删除一些索引，这将导致性能下降。

2. MongoDB的索引在部分查询条件下是不会生效的。

   -   正则表达式及非操作符，如 `$nin`,`$not` , 等。
   -   算术运算符，如 $mod, 等。
   -   $where自定义查询函数。
   -   ...

3. 索引会在写入数据（添加、更新和删除）时重排，如果项目如果是写多读少，则建议少使用或者不要使用索引。

4. 一个集合中索引数量不能超过64个。

5. 索引名的长度不能超过128个字符。

6. 一个复合索引最多可以有31个字段。

7. mongodb索引统一在`system.indexes`集合中管理。这个集合只能通过`createIndex`和`dropIndexes`来操作。

#### 查看索引

```javascript
// 获取当前集合中已经创建的所有索引信息
db.集合.getIndexes()
/*
[{ 
	"v" : 2,   // 索引版本
	"key" : {  // 索引的字段及排序方向(1表示升序，-1表示降序)
		"_id" : 1   // 根据_id字段升序索引
    }, 
    "name" : "_id"   // 索引的名称
}]
*/
// 获取当前集合中已经创建的索引总大小，以字节为单位返回结果
db.集合.totalIndexSize()
// 获取当前数据库中所有的索引【不会显示默认主键_id】
db.system.indexes.find()
```

MongoDB会为插入的文档默认生成`_id`字段（如果文档本身没有指定该字段），`_id`是文档唯一的主键，为了保证能根据文档id快速查询文档，MongoDB默认会为集合创建_id字段的主键索引。

#### 查询分析

与SQL语句类似，MongoDB也提供了一个explain，供开发者进行查询分析，优化查询语句。

explain的使用有3个参数，分别是：queryPlanner、executionStats、allPlansExecution，默认是queryPlanner，开发中常用的是executionStats。

```javascript
db.orders.find({"title":"愉快购物-6"}).explain("executionStats");
/*
{
	"queryPlanner" : {  # 被查询优化器选择出来的查询计划
		"plannerVersion" : 1,  # 查询计划版本
		"namespace" : "test.orders", # 要查询的集合
		"indexFilterSet" : false,  # 是否了使用索引
		"parsedQuery" : {  # 查询条件
			"title" : {
				"$eq" : "购买商品-19"
			}
		},
		"winningPlan" : {     # 最佳执行计划
			"stage" : "COLLSCAN", # 扫描类型/扫描阶段
			"filter" : {     # 过滤条件
				"title" : {
					"$eq" : "购买商品-19"
				}
			},
			"direction" : "forward"  # 查询方向，forward为升序，backward表示倒序。
		},
		"rejectedPlans" : [ ]   # 拒绝的执行计划
	},
	"executionStats" : {  # 最佳执行计划的一些统计信息
		"executionSuccess" : true,  # 是否执行成功
		"nReturned" : 1,   # 返回的结果数
		"executionTimeMillis" : 346,  # 执行耗时
		"totalKeysExamined" : 0,      # 索引扫描次数
		"totalDocsExamined" : 1000000,  # 文档扫描次数，所谓的优化无非是让totalDocsExamined和nReturned的值接近。
		"executionStages" : {     # 执行状态
			"stage" : "COLLSCAN",  # 扫描方式/扫描阶段
			"filter" : {
				"title" : {
					"$eq" : "购买商品-19"
				}
			},
			"nReturned" : 1,   # 返回的结果数
			"executionTimeMillisEstimate" : 5,   # 预估耗时
			"works" : 1000002,   # 工作单元数
			"advanced" : 1,      # 优先返回的结果数
			"needTime" : 1000000,
			"needYield" : 0,
			"saveState" : 1000,
			"restoreState" : 1000,
			"isEOF" : 1,
			"direction" : "forward",
			"docsExamined" : 1000000   # 文档检查数目，与totalDocsExamined一致
		}
	},
	"serverInfo" : {   # 服务器信息
		"host" : "ubuntu",
		"port" : 27017,
		"version" : "4.4.2",
		"gitVersion" : "15e73dc5738d2278b688f8929aee605fe4279b0e"
	},
	"ok" : 1
}

*/

```

stage的扫描类型：

| 类型名称   | 描述                               | 期望  |
| ---------- | ---------------------------------- | ----- |
| COLLSCAN   | 全表扫描                           | False |
| IXSCAN     | 索引扫描                           | True  |
| FETCH      | 根据索引去检索指定document         | True  |
| IDHACK     | 针对_id进行查询                    | True  |
| COUNTSCAN  | count不使用Index进行count时返回    | False |
| COUNT_SCAN | count使用了Index进行count时返回    | True  |
| SUBPLA     | 未使用到索引的$or查询时返回        | False |
| TEXT       | 使用全文索引进行查询时返回         | -     |
| SORT       | 使用sort排序但是无index时返回      | False |
| SKIP       | 使用skip跳过但是无index时返回      | False |
| PROJECTION | 使用limit限定结果但是无index时返回 | False |



#### 创建索引

MongoDB支持多种类型的索引，包括普通索引(单列索引)、复合索引、多列索引、全文索引、[哈希索引](https://docs.mongodb.org/manual/core/index-hashed/)、[地理位置索引](https://docs.mongodb.org/manual/core/2d/)等，每种类型的索引有不同的使用场合。ttl索引本质上就是普通索引，只是给索引添加一个过期时间而已。另外，MongoDB的全文索引很弱智，如果真要用在开发中，还是建议使用elasticsearch或者Sphinx。

```json
// 创建索引
db.集合.createIndex({
    // 单个字段，则为普通索引，    // sort的值表示排序，值为1表示升序索引，-1表示降序索引
    "字段名1": <sort|type>,       // type的值可以是text，表示创建全文索引。db.集合.find({$text:{$search:"字符串"}})
    "字段名2": <sort|type>,       // 多个字段，则为复合索引
    "字段名3": [<值1>,<值2>,...],  // 多列索引
    ....
}, {
    background: <Boolean>,   // 建索引过程会阻塞数据库的其它操作，background可指定以后台方式创建索引，默认为false
    unique: <Boolean>,  // 是否建立唯一索引，默认值为false，也叫唯一索引
    name: <String>,   // 索引的名称，不填写，则MongoDB会通过连接索引的字段名和排序顺序生成一个索引名称 
    expireAfterSeconds: <integer>, // 设置索引的过期时间，类似redis的expire，也叫TTL索引
    sparse: <Boolean>,  // 对文档中不存在的字段数据是否不启用索引，默认为False
});


// 单字段索引[普通索引]
 db.集合.createIndex({
    "字段名": <sort>,    // sort的值表示排序，值为1表示升序索引，-1表示降序索引
 }, {
	....
 })
// 普通索引创建： db.orders.createIndex({"title":1})
// 查询基本使用： db.orders.find({"title":"愉快购物-6"}).explain("executionStats");


// 多字段索引，也叫复合索引。[类似mysql里面的联合索引]
 db.集合.createIndex({
    "字段名1": <sort>,    // sort的值表示排序，值为1表示升序索引，-1表示降序索引
    "字段名2": <sort>,    // sort的值表示排序，值为1表示升序索引，-1表示降序索引
 }, {
	....
 })

// 复合索引的使用对单字段条件的查找是没有帮助的，必须多字段[必须包含复合索引的字段]条件使用
// 复合索引创建：db.orders.createIndex({"date":1,"title":1});
// 查询基本使用：
//     db.orders.find({"date":"2014-06-12","title":"买年货-7"}).explain("executionStats");
//     db.orders.find({"date":"2014-06-12","onumber":"0000000000030014","title":"买年货-7"});


// 全文索引
 db.集合.createIndex({
    "字段名1": "text",    // type的值只能是text，表示创建全文索引。db.集合.find({$text:{$search:"字符串"}})
 }, {
	....
 })

// 全文索引创建： db.orders.createIndex({"title":"text"})
// 查询基本使用： db.orders.find({$text:{$search:"商品-19"}}).explain("executionStats")



// 多列索引[应用的地方是在列表属性]
 db.集合.createIndex({
    "字段名3": [<值1>,<值2>,...],
 }, {
	....
 });

// 创建测试数据
db.doc.drop()
db.doc.insert({"title":"标题1","tags":["python","django"]})
db.doc.insert({"title":"标题1","tags":["python","django"]})
db.doc.insert({"title":"标题1","tags":["python","django"]})
db.doc.insert({"title":"标题2","tags":["java","mvp"]})
db.doc.insert({"title":"标题3","tags":["java","mvp"]})
db.doc.insert({"title":"标题2","tags":["java","mvp"]})
db.doc.insert({"title":"标题3","tags":["python"]})
db.doc.insert({"title":"标题4","tags":["python"]})
db.doc.insert({"title":"标题2","tags":["python","flask"]})
db.doc.insert({"title":"标题3","tags":["java"]})
// 创建多列索引： db.doc.createIndex({"tags":1})
// 查询数据： db.doc.find({"tags":["python"]}).explain("executionStats")



// 唯一索引
db.集合.createIndex({
    "字段名1": <sort>,
}, {
    unique: true,     // 是否建立唯一索引，默认值为false，也叫唯一索引
})
// 创建唯一索引： db.orders.createIndex({"onumber":1},{unique:true});
// 查询数据： db.orders.find({"onumber":"0000000000001019"}).explain("executionStats")



// ttl索引
// 使用ttl索引，索引关键字段的值类型必须是Date类型，如果该字段不是date类型或者文档中不存在该字段，则文档不会进行过期处理
// 数据过期的删除工作是在mongoDB中的独立线程内执行的，默认平均60s扫描一次，不会立即删除。

// 例如：在文档创建10秒后删除文档
db.orders.dropIndex("date_1")
db.orders.createIndex({"date": 1},{expireAfterSeconds: 10});
db.orders.insertOne({
   "date": new Date("2022-01-10 17:30:00"), // 在python中需要通过 utctime
   "user_id": 2,
   "username": "xiaohong"
})

// 在文档创建后，由索引字段值指定的时间删除文档
// 创建索引：db.tasks.createIndex({"expire_time":1},{expireAfterSeconds:0})
// 创建测试数据
db.tasks.insert( {
   "expire_time": new Date('2022-01-10 17:32:05'), // 在python中需要通过 utctime
   "user_id": 2,
   "username": "xiaoming",
   "onumber": "200003"
});

db.tasks.insert( {
   "expire_time": new Date('2022-01-10 17:34:05'), // 在python中需要通过 utctime
   "user_id": 2,
   "username": "xiaoming"
   "onumber": "200004"
});
db.tasks.insert( {
   "expire_time": new Date('2022-01-10 17:35:10'), // 在python中需要通过 utctime
   "user_id": 2,
   "username": "xiaoming"
   "onumber": "200005"
});

// 重建索引[一般是在长期项目运行下来，索引创建时间太久了，性能下降的时候使用。]
// !!!!不能在高峰期时运行以下操作
db.集合.reIndex();
```



#### 删除索引

MongoDB给文档主键`_id`默认创建单字段索引是无法删除的。

```json
// 删除单个索引
db.集合.dropIndex("索引名称")
// db.orders.dropIndex("date_1")

// 删除所有索引，慎用
db.集合.dropIndexes()
```



在python当中，一般常用于开发中操作monoDB的模块无非三个：pymongo, mongoengine,  moter

moter是python中基于pymongo实现的异步操作库，类似于aiomysql，aiomysql也是python基于pymysql实现的异步库。

```python
mysql  pymysql  mysqlDB  aiomysql[基于pymysql实现的异步库]
redis  pyredis  redis    aioredis[基于pyredis实现的异步库]
mongo  pymongo  mongoengine[基于pymongo实现的ORM，高仿django的ORM]  moter[基于pymongo实现的异步库]
```



## PyMongo

安装：

```bash
pip install pymongo
```

### 数据库连接

**数据库连接，无密码**

```python
import pymongo
mongo = pymongo.MongoClient("mongodb://127.0.0.1:27017/")
```

**数据库连接，有密码**

```python
import pymongo
from urllib.parse import quote_plus

if __name__ == '__main__':
    # 方式1：
    username = quote_plus("mofang") # 字符转移，防止出现多字节的字符，例如中文或者特殊符号
    password = quote_plus("123456")
    database = quote_plus("mofang")
    # 获取数据库连接对象
    mongo = pymongo.MongoClient(f"mongodb://{username}:{password}@127.0.0.1:27017/{database}")
    print(mongo)
    # 获取数据库操作对象
    db = mongo[database]
    print(db)
    # 获取集合操作对象
    collection = db["user_list"]
    print(collection)

    # 方式2：
    mongo = pymongo.MongoClient('mongodb://127.0.0.1:27017')
    db = mongo["mofang"]
    username = "mofang"
    password = "123456"
    db.authenticate(username, password) # 相当于在monogDB终端下的db.auth()
    print(db)
    collection = db["user_list"]
    print(collection)
```



### 数据库管理

```python
import pymongo
from urllib.parse import quote_plus

if __name__ == '__main__':
    username = quote_plus("mofang")
    password = quote_plus("123456")
    # 获取数据库连接对象
    mongo = pymongo.MongoClient(f"mongodb://{username}:{password}@127.0.0.1:27017/mofang")
    print(mongo)

    # 新建一个数据库
    shop = mongo["shop"]
    print(shop)
    # 查看数据库列表[只会列出有文档数据的数据库]
    print(mongo.list_database_names())  # 上面的 my_db 因为没有内容，所以没有被创建的。
    print(mongo["mofang"])
```



### 集合管理

```python
import pymongo
from urllib.parse import quote_plus

if __name__ == '__main__':
    username = quote_plus("mofang")
    password = quote_plus("123456")
    # 获取数据库连接对象
    mongo = pymongo.MongoClient(f"mongodb://{username}:{password}@127.0.0.1:27017/mofang")
    print(mongo)

    # 新建一个数据库
    shop = mongo["shop"]
    print(shop)
    # 查看数据库列表[只会列出有文档数据的数据库]
    print(mongo.list_database_names())  # 上面的 my_db 因为没有内容，所以没有被创建的。

    db = mongo["mofang"]
    # 新建集合
    user_list = db["user_list"] # 新建一个集合

    # 查看指定数据的集合列表[只会列出有文档数据的集合]
    print( db.list_collection_names() )

    # 获取指定名称对应的集合操作对象
    user_list = db["user_list"]
    # 删除集合
    user_list.drop()
```



### 文档管理

#### 添加文档

```python
import pymongo
mongo = pymongo.MongoClient("mongodb://127.0.0.1:27017/")
mofang = mongo["mofang"]
user_list = mofang["user_list"]

# 添加一个文档
document = { "name": "xiaoming", "mobile": "13012345678","age":16}
ret = user_list.insert_one(document)
print(ret.inserted_id) # 返回InsertOneResult对象
# 插入文档时，如果没有指定_id，将自动分配一个唯一的id。

# 添加多个文档
document_list = [
  { "name": "xiaoming", "mobile": "13033345678","age":17},
  { "name": "xiaohong", "mobile": "13044345678","age":18},
  { "name": "xiaohei",  "mobile": "13612345678","age":18},
]
ret = user_list.insert_many(document_list)

# 打印文档_id值列表:
print(ret.inserted_ids)
```



#### 删除文档

```python
import pymongo
from urllib.parse import quote_plus
from bson import ObjectId

if __name__ == '__main__':
    username = quote_plus("mofang")
    password = quote_plus("123456")
    # 获取数据库连接对象
    mongo = pymongo.MongoClient(f"mongodb://{username}:{password}@127.0.0.1:27017/mofang")
    mofang = mongo["mofang"]
    user_list = mofang["user_list"]

    """删除一个文档"""
    query = {"_id": ObjectId("6157dcdeec192e9a0b7ea0e0")}
    ret = user_list.delete_one(query)
    print(ret)
    print(ret.deleted_count)


    """删除多个文档"""
    query = {"name":"xiaolan"}
    ret = user_list.delete_many(query)
    print(ret)
    print(ret.deleted_count)

    """可以通过删除集合的方式，达到删除所有文档的效果"""
```



#### 更新文档

```python
import pymongo
from urllib.parse import quote_plus
from bson import ObjectId

if __name__ == '__main__':
    username = quote_plus("mofang")
    password = quote_plus("123456")
    # 获取数据库连接对象
    mongo = pymongo.MongoClient(f"mongodb://{username}:{password}@127.0.0.1:27017/mofang")
    mofang = mongo["mofang"]
    user_list = mofang["user_list"]

    """更新一条数据"""
    query = {"name": "小白"}
    data = {"$set": {"age": 22}}
    ret = user_list.update_one(query, data)

    print(ret)
    print(ret.modified_count)

    """更新所有文档"""
    query = {"mobile": {"$regex": "^13"}}
    data = {"$inc": {"age": 2}}
    ret = user_list.update_many(query, data)
    print(ret)
    print(ret.modified_count)
```



#### 查询文档

```python
import pymongo

mongo = pymongo.MongoClient("mongodb://127.0.0.1:27017/")
mofang = mongo["mofang"]
user_list = mofang["user_list"]

# 查看一个文档
ret = user_list.find_one()
print(ret)

# 查看所有文档
for document in user_list.find():
	print(document)

# 查看文档部分字段，find和find_one的第二个参数表示控制字段的显示隐藏，1为显示，0为隐藏
for document in user_list.find({},{ "_id": 0, "name": 1, "mobile": 1 }):
	print(document)

# 条件查询
query = { "age": 18 }
document_list = user_list.find(query)
for document in document_list:
	print(document)

# 比较运算符
query = { "age": {"$gt":17} }
document_list = user_list.find(query)
for document in document_list:
	print(document)

# 排序显示
# 单个字段排序：
# 		sort("键", 1) 升序
# 		sort("键",-1) 降序

# 多个字段排序：
#       sort([("键1",1),("键2",-1)])
document_list = user_list.find().sort("age")
for document in document_list:
	print(document)
    
# 限制查询结果数量
document_list = user_list.find().limit(3)
for document in document_list:
	print(document)

# 偏移、跳过
#	skip(int)
document_list = user_list.find().limit(3).skip(3) # 从第3篇文档开始获取3篇文档
print(document_list)

# 自定义条件函数
document_list = user_list.find({"$where":"this.age==18"})
print(document_list)
```

