# mongodb集群

mongodb集群有三种模式，主从模式，副本集模式、sharding分片集模式。主从模式官网也不再推荐上生产环境，主要是安全性太低。副本集和sharding分片集模式目前使用的最广的方案，通常这2种方案的选择通过数据量和并发数来权衡。在GB级别的基本上副本集方案可满足，TB级别或以上采用sharding分片集模式，解决单机容量和单机并发能力。这两种既有自己的优势也有自己的缺点，比如sharding分片集模式分片越多，性能自然下降越多。

## 副本集

也叫复制集，本质上来说就是一种具有监控能力的主从模式，类似redis中的哨兵主从集群。

一组Mongodb复制集，就是一组mongod进程，这些进程维护同一个数据集合。复制集提供了数据冗余和高等级的可靠性，这是生产部署的基础。说白了，就是高级主从。

保证数据在生产部署时的冗余和可靠性，通过在不同的机器上保存副本来保证数据的不会因为单点损坏而丢失。能够随时应对数据丢失、机器损坏带来的风险。

### 复制集的基本架构

+   副本集：一个副本集就是一组MongoDB实例组成的集群，由一个主（Primary）服务器和多个备份（Secondary）服务器构成
+   主节点（primary）：主节点接收所有**写入操作**。主节点将对其数据集所做的所有更改记录到其 oplog日志。
+   副节点（secondary）：复制主节点的 oplog日志 并将操作应用到其数据集，如果主节点不可用，一个合格的副节点将被**选举**为新的主节点。
+   仲裁节点（arbiter）：负载选举，当主节点不可用，它将从副节点中选一个作为主节点。



复制集的基本架构由3台服务器组成，有2种组成模式：

+   三成员的复制集，有3个主从节点(1主2从)

+   两成员的复制集，有2个主从节点，1个仲裁节点(arbiter)。（1主1从1仲裁）

#### 不存在arbiter节点

>   一个主节点；
>
>   两个从节点组成，主节点宕机时，这两个从节点都可以经过primary选举以后被选为主节点。


![img](2mongo%E9%9B%86%E7%BE%A4.assets/1190037-20180106145148128-1854811460.png)

​    当主库宕机后,两个从库都会进行竞选，其中一个变为主库，当原主库恢复后，作为从库加入当前的复制集群即可。

![img](2mongo%E9%9B%86%E7%BE%A4.assets/1190037-20180106145154284-397901575.png) 



#### 存在arbiter节点

在三个成员的复制集中，有两个正常的主从节点和一个arbiter节点：

>     一个主节点
>
>     一个从节点，可以在选举中成为主库
>
>     一个仲裁节点，在选举中，只进行投票，不能成为主库

![img](2mongo%E9%9B%86%E7%BE%A4.assets/1190037-20180106145207237-1647029849.png) 

说明：由于arbiter节点没有复制数据，因此这个架构中仅提供一个完整的数据副本。arbiter节点只需要更少的资源，代价是更有限的冗余和容错。

当主库宕机时，将会选择从库成为主，主库修复后，将其加入到现有的复制集群中即可。

![img](2mongo%E9%9B%86%E7%BE%A4.assets/1190037-20180106145221393-1674631191.png) 



### Primary选举

　　复制集通过replSetInitiate命令（或mongo shell的rs.initiate()）进行初始化，初始化后各个成员间开始发送心跳消息，并发起Priamry选举操作，获得『大多数 总服务器-仲裁节点/ 2 > 50%』成员投票支持的节点，会成为Primary，其余节点成为Secondary。

**『大多数』的定义**

　　假设复制集内投票成员数量为N，则大多数为 N/2 + 1，当复制集内存活成员数量不足大多数时，整个复制集将无法选举出Primary，复制集将无法提供写服务，处于只读状态。

| **投票成员数** | **大多数** | **容忍失效数** |
| -------------- | ---------- | -------------- |
| **1**          | 1          | 0              |
| **2**          | 2          | 0              |
| **3**          | 2          | 1              |
| **4**          | 3          | 1              |
| **5**          | 3          | 2              |
| **6**          | 4          | 2              |
| **7**          | 4          | 3              |

　　通常建议将复制集成员数量设置为**奇数**，从上表可以看出3个节点和4个节点的复制集都只能容忍1个节点失效，从『服务可用性』的角度看，其效果是一样的。（但无疑4个节点能提供更可靠的数据存储）

### docker-compose搭建MongoDB复制集

keyfile是mongodb副本集的实例之间的权限认证，复制集要求每一个服务器下的keyfile内容必须相同。内容不同，那么该实例添加到副本集的时候，会出现不可达的状态。同时对于keyfile文件还要求如下：

+   keyfile文件的权限必须是600/400；
+   keyfile文件的内容必须在各个服务器下完全相同。

生成keyfile秘钥文件命令如下，但是因为在windows下容易出现权限问题，所以我们可以基于系统镜像创建Dockerfile构建自定义镜像，并在Dockerfile中编写生成keyfile文件的命令和设置权限的命令。

756就是秘钥长度，最好3的倍数。

```bash
openssl rand -base64 756 > data/conf/mongodb.key
chmod 600 data/conf/mongodb.key
```

Dockerfile，内容：

```dockerfile
FROM mongo:4.4.11

RUN mkdir -p /data/conf \
  && openssl rand -base64 756 > /data/conf/mongodb.key \
  && chown mongodb:mongodb /data/conf/mongodb.key \
  && chmod 400 /data/conf/mongodb.key
```

编译镜像，如下;

```bash
docker build -t mongo-replica-set .
```

docker-compose.yml，代码：

```yaml
version: '3.7'

services:
  # 主节点
  master:
    image: mongo-replica-set
    restart: always
    privileged: true
    container_name: master
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: 123
    command: --replSet replica-set --keyFile /data/conf/mongodb.key
    volumes:
      - ./data/mongo/d1:/data/db
  # 副节点
  secondary:
    image: mongo-replica-set
    restart: always
    privileged: true
    container_name: secondary
    ports:
      - 27018:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: 123
    command: --replSet replica-set --keyFile /data/conf/mongodb.key
    volumes:
      - ./data/mongo/d2:/data/db
  # 仲裁节点
  arbiter:
    image: mongo-replica-set
    restart: always
    privileged: true
    container_name: arbiter
    ports:
      - 27019:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: 123
    command: --replSet replica-set --keyFile /data/conf/mongodb.key
    volumes:
      - ./data/mongo/d3:/data/db

networks:
  # 默认使用自己创建的网络
  default:

```



#### 复制集初始化配置

```bash
docker-compose -mongo docker-compose.yml up -d

# 选择一个容器进行配置
docker exec -it master /bin/bash

# 配置时需要在admin数据库中
mongo
use admin

# 验证权限
db.auth('root','123')
# 验证通过后，则可以执行副本集的初始化操作initiate()，设置当前mongodb为primary节点
rs.initiate({
	"_id" : "replica-set",
	"members" : [
		{
			"_id" : 0,
			"host" : "master:27017"
		},
		{
			"_id" : 1,
			"host" : "secondary:27017"
		},
		{
			"_id" : 2,
			"host" : "arbiter:27017"
		}
	]
})


# 最后，SECONDARY节点默认是不可读写的，所以要进入从节点，执行如下操作：
docker exec -it secondary /bin/bash
mongo
rs.secondaryOk()

docker exec -it arbiter /bin/bash
mongo
rs.slaveOk()
```



## 分片集

当数据量比较大的时候，我们需要把数据分片保存并运行在不同的服务器中，以降低CPU、内存和IO的压力，Sharding分片集就是mongo数据库用于横向扩容的技术。

分片是数据跨多台机器存储，MongoDB使用分片来支持具有非常大的数据集和高吞吐量操作的部署。MongoDB分片技术类似MySQL的水平切分和垂直切分，mongo数据库实现Shardin分片集主要有两种方式：垂直扩展和横向切分（也叫水平扩展）。

-   垂直扩展的方式就是进行计算机资源扩容，添加更多的CPU，内存，磁盘空间等。

-   水平扩展则是通过数据分片的方式将数据集分布在多个服务器上。，通过集群统一提供服务。

### 分片设计思想

分片为应对高吞吐量与大数据量提供了方法。使用分片减少了每个分片需要处理的请求数，因此通过水平扩展，集群可以提高自己的存储容量和吞吐量。举例来说，当插入一条数据时，项目应用只需要访问存储这条数据的分片.

使用分片减少了每个分片存储的数据。例如如果数据库1tb的数据集，并有4个分片，然后每个分片可能仅持有256 GB的数据。如果有40个分片，那么每个切分可能只有25GB的数据。

![img](2mongo%E9%9B%86%E7%BE%A4.assets/13587608-58483213c588ee8f.png)

### 分片机制的优势

**1.对集群进行抽象，让集群“不可见”**

MongoDB自带了一个叫做mongos的专有路由进程。mongos就是掌握统一路口的路由器，其会将客户端发来的请求准确无误的路由到集群中的一个或者一组服务器上，同时会把接收到的响应拼装起来发回到客户端。

**2.保证集群总是可读写**

MongoDB通过多种途径来确保集群的可用性和可靠性。将MongoDB的分片和复制功能结合使用，在确保数据分片到多台服务器的同时，也确保了每分数据都有相应的备份，这样就可以确保有服务器换掉时，其他的从库可以立即接替坏掉的部分继续工作。

**3.*使集群易于扩展**

当系统需要更多的空间和资源的时候，MongoDB使我们可以按需方便的扩充系统容量。



### 分片集群架构

| 组件          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| Config Server | 配置服务器，是mongod实例，可以理解为真实数据的元数据，存储了集群的所有节点、分片数据路由等信息。默认需要配置3个Config Server节点。Config Server中存储的信息：所有存取数据的方式，所有shard节点的信息，分片功能的一些配置信息。 |
| Mongos        | 数据路由，作为与客户端打交道的模块，提供对外应用访问，所有操作均通过mongos执行。一般有多个mongos节点。数据迁移和数据自动平衡。Mongos本身并不持久化数据，Sharded cluster所有的元数据都会存储到Config Server，而用户的数据会议分散存储到各个shard。Mongos启动后，会从配置服务器加载元数据，开始提供服务，将用户的请求正确路由到对应的碎片。 |
| Mongod        | 真正的数据存储位置，存储应用数据记录。一般有多个Mongod节点，以chunk为单位存数据，达到数据分片目的。 |

![分片集群架构](2mongo%E9%9B%86%E7%BE%A4.assets/13587608-9f139e94370a15c8.png)

**Mongos的路由功能**

当数据写入时，MongoDB Cluster根据**分片键**设计写入数据。

当外部语句发起数据查询时，MongoDB根据数据分布自动路由至指定节点返回数据。



### 分片集群中的数据分布

（1）使用chunk（数据块）来把数据存储不同的分片（shard）

（2）集群搭建完成之后，默认开启一个chunk，大小是64M，chunk的大小容量可以通过配置chunksize修改。

（3）存储数据的容量只要超过64M，chunk会进行平均分裂，如果单位时间存储需求很大，设置更大的chunk

（4）chunk会被自动均衡迁移到各个Shard分片节点下面。

#### Chunk（数据库）是什么

就是一个存储数据的单位，是为了方便分割(分裂)数据和迁移数据的。在一个shard server内部，MongoDB还是会把数据分为chunks，每个chunk代表这个shard server内部一部分数据。chunk的产生，会有以下两个用途：

Splitting（切割）：当一个chunk的大小超过配置中的chunk size时，MongoDB的后台进程会把这个chunk切分成更小的chunk，从而避免chunk过大的情况

Balancing(迁移)：在MongoDB中，balancer是一个后台进程，负责chunk(数据块)的迁移，从而均衡各个shard server的负载，系统初始化时默认只有1个chunk，chunk size默认值64M，生产环境中根据业务选择合适的chunk size是最好的。mongoDB会自动拆分和迁移chunks。



#### Chunksize的选择

适合项目业务的chunksize是最好的。

chunk的分裂和迁移非常消耗IO资源；chunk分裂的时机：在插入和更新，读数据不会分裂。

小的chunksize：数据均衡是迁移速度快，数据分布更均匀。数据分裂频繁，路由节点消耗更多资源。

大的chunksize：数据分裂少。数据块迁移的时候就会比较集中消耗IO资源。

chunksize的值通过如果没有特殊的要求，可以设置为通常100-200M。

#### Chunk分裂及迁移

　　随着数据的增长，其中的数据大小超过了配置的chunksize，默认是64M，则这个chunk就会分裂成两个。数据的增长会让chunk分裂得越来越多。

 ![img](2mongo%E9%9B%86%E7%BE%A4.assets/1190037-20180106150630018-1017846225.png)

　　这时候，各个shard 上的chunk数量就会不平衡。这时候，mongos中的一个组件balancer 就会执行自动平衡。把chunk从chunk数量最多的shard节点挪动到数量最少的节点。

![img](2mongo%E9%9B%86%E7%BE%A4.assets/1190037-20180106150636096-1641567859.png) 

**chunkSize** **对分裂及迁移的影响**

　　MongoDB 默认的 chunkSize 为64MB，如无特殊需求，建议保持默认值或者100~200M左右；chunkSize 会直接影响到 chunk（数据块） 分裂、迁移的行为。

　　chunkSize 越小，chunk 分裂及迁移越多，数据分布越均衡；反之，chunkSize 越大，chunk 分裂及迁移会更少，但可能导致数据分布不均。

　　chunkSize 太小，容易出现 jumbo chunk（即分片键shardKey的某个取值出现频率很高，这些文档只能放到一个 chunk 里，无法再分裂）而无法迁移；chunkSize 越大，则可能出现 chunk 内文档数太多（chunk 内文档数不能超过 250000 ）而无法迁移。

　　chunk 自动分裂只会在数据写入时触发，所以如果将 chunkSize 改小，系统需要一定的时间来将 chunk 分裂到指定的大小。

　　**chunk 只会分裂，不会合并**，所以即使将 chunkSize 改大，现有的 chunk 数量不会减少，但 chunk 大小会随着写入不断增长，直到达到目标大小。



### 数据分片

#### 分片键

MongoDB中**数据的分片是以集合为基本单位的**，集合中的数据通过片键（Shard key）被分成多部分。其实分片键就是在集合中选一个键（字段），用该键的值作为数据拆分的依据。

所以一个好的分片键对分片至关重要。分片键必须是一个索引，通过sh.shardCollection加会自动创建索引（前提是此集合不存在相同索引的情况下）。一个自增的分片键对写入和数据均匀分布就不是很好，因为自增的分片键总会在一个分片上写入，后续达到某个阀值可能会写到别的分片。但是按照分片键查询会非常高效。

随机分片键对数据的均匀分布效果很好。注意尽量避免在多个分片上进行查询。在所有分片上查询，mongos会对结果进行归并排序。

对集合进行分片时，你需要选择一个分片键，**分片键是每条记录都必须包含的，且建立了索引的单个字段或复合字段**，MongoDB按照分片键将数据划分到不同的数据块(chunk)中，并将数据块(chunk)均衡地分布到所有分片节点中。

为了按照分片键划分数据块，MongoDB使用**基于范围**或者 **基于哈希**的分片方式。

**注意：**

-   分片键是不可变。

-   分片键必须有索引。

-   分片键大小限制512bytes。

-   分片键用于分片集中的路由查询(mongos)。

-   MongoDB不接受已进行collection级分片的collection上插入无分片

-   键的文档（也不支持空值插入）

#### 基于范围的分片

Sharded Cluster支持将单个集合的数据分散存储在多shard上，用户可以指定根据集合内文档的某个字段即shard key来进行范围分片（range sharding）。

#### 基于哈希的分片

对于基于哈希的分片，MongoDB计算一个字段的哈希值，并用这个哈希值来创建数据块。在使用基于哈希分片的系统中，拥有”相近”片键的文档很可能不会存储在同一个数据块中，因此数据的分离性更好一些。



**Hash分片与范围分片互补**，能将文档随机的分散到各个chunk，充分的扩展写能力，弥补了范围分片的不足，但不能高效的服务范围查询，所有的范围查询要分发到后端所有的Shard才能找出满足条件的文档。



### docker-compose搭建分片集集群

```bash
docker network create mongo
```

mongo-compose.yaml，代码：

```yaml
version: '3.7'
services:
  shard1:
    image: mongo:4.4.11
    container_name: mongo_shard1
    # --shardsvr: 这个参数仅仅只是将默认的27017端口改为27018,如果指定--port参数，可用不需要这个参数
    # --directoryperdb：每个数据库使用单独的文件夹
    command: mongod --shardsvr --directoryperdb --replSet shard1
    volumes:
      - ./data/mongo/localtime:/etc/localtime
      - ./data/mongo/shard1:/data/db
    privileged: true
    mem_limit: 16000000000
    networks:
      - mongo   

  shard2:
    image: mongo:4.4.11
    container_name: mongo_shard2
    command: mongod --shardsvr --directoryperdb --replSet shard2
    volumes:
      - ./data/mongo/localtime:/etc/localtime
      - ./data/mongo/shard2:/data/db
    privileged: true
    mem_limit: 16000000000
    networks:
      - mongo

  shard3:
    image: mongo:4.4.11
    container_name: mongo_shard3
    command: mongod --shardsvr --directoryperdb --replSet shard3
    volumes:
      - ./data/mongo/localtime:/etc/localtime
      - ./data/mongo/shard3:/data/db
    privileged: true
    mem_limit: 16000000000
    networks:
      - mongo

  config1:
    image: mongo:4.4.11
    container_name: mongo_config1
    # --configsvr: 这个参数仅仅是将默认端口由27017改为27019, 如果指定--port可不添加该参数
    command: mongod --configsvr --directoryperdb --replSet fates-mongo-config --smallfiles
    volumes:
      - ./data/mongo/localtime:/etc/localtime
      - ./data/mongo/config1:/data/configdb
    networks:
      - mongo

  config2:
    image: mongo:4.4.11
    container_name: mongo_config2
    command: mongod --configsvr --directoryperdb --replSet fates-mongo-config --smallfiles
    volumes:
      - ./data/mongo/localtime:/etc/localtime
      - ./data/mongo/config2:/data/configdb
    networks:
      - mongo

  config3:
    image: mongo:4.4.11
    container_name: mongo_config3
    command: mongod --configsvr --directoryperdb --replSet fates-mongo-config --smallfiles
    volumes:
      - ./data/mongo/localtime:/etc/localtime
      - ./data/mongo/config3:/data/configdb
    networks:
      - mongo

  mongos:
    image: mongo:4.4.11
    container_name: mongo_mongos
    # mongo3.6版默认绑定IP为127.0.0.1，此处绑定0.0.0.0是允许其他容器或主机可以访问
    command: mongos --configdb fates-mongo-config/config1:27019,config2:27019,config3:27019 --bind_ip 0.0.0.0 --port 27017
    ports:
     - 27017:27017
    volumes:
      - ./data/mongo/localtime:/etc/localtime
    depends_on:
      - config1
      - config2
      - config3
    networks:
      - mongo
networks:
  mongo:
    external: true
```

run.sh，代码：

```bash
#!/bin/sh
# docker-compose启动mongodb分片容器组服务
docker-compose -f mongo-compose.yaml up -d

# 睡眠半分钟，等待mongodb所有容器起来之后将它们配置加入分片
sleep 30s

# 操作config1，配置config副本集，将config容器组作为config角色,此时config1作为config副本集里的主节点
docker-compose -f mongo-compose.yaml exec config1 bash -c "echo 'rs.initiate({_id: \"mongo-config\",configsvr: true, members: [{ _id : 0, host : \"config1:27019\" },{ _id : 1, host : \"config2:27019\" }, { _id : 2, host : \"config3:27019\" }]})' | mongo --port 27019"

# 操作shard1、shard2、shard3，将shard容器组作为shard角色。
docker-compose -f mongo-compose.yaml exec shard1 bash -c "echo 'rs.initiate({_id: \"shard1\",members: [{ _id : 0, host : \"shard1:27018\" }]})' | mongo --port 27018"
docker-compose -f mongo-compose.yaml exec shard2 bash -c "echo 'rs.initiate({_id: \"shard2\",members: [{ _id : 0, host : \"shard2:27018\" }]})' | mongo --port 27018"
docker-compose -f mongo-compose.yaml exec shard3 bash -c "echo 'rs.initiate({_id: \"shard3\",members: [{ _id : 0, host : \"shard3:27018\" }]})' | mongo --port 27018"

# 将shard1、shard2、shard3加入分片集群组。
docker-compose -f mongo-compose.yaml exec mongos bash -c "echo 'sh.addShard(\"shard1/shard1:27018\")' | mongo"
docker-compose -f mongo-compose.yaml exec mongos bash -c "echo 'sh.addShard(\"shard2/shard2:27018\")' | mongo"
docker-compose -f mongo-compose.yaml exec mongos bash -c "echo 'sh.addShard(\"shard3/shard3:27018\")' | mongo"
```

运行

```bash
sh run.sh
```

上面仅仅借助docker-compose实现分片集的部署机制。真正的运维环境中，Mongodb集群应该分布在不同的机器，同时，Mongodb库默认是不会将你的表进行分片的，是需要手动配置的，如果不配置，数据都会全部存放在主节点，不会均匀分配到各个分片。

举例：

```javascript
// 注意：分片的片键必须是索引

// 先让当前库支持分片，school是举例数据库，可以填写实际要分片的库名。
sh.enableSharding("school")


// 把当前集合加入分片，递增片键(timestamp已经作为索引了)，student表示要分片的集合，可以填写实际要分片的集合名称。
sh.shardCollection('school.student',{timestamp:1})

// 刷新路由
db.adminCommand("flushRouterConfig")

// 让当前分片支持均衡调度， 当某个shard中的chunk达到一定的数量，则balancer会自动的将部分chunk块迁移到其它shard中，保证所有的shard中拥有的 chunk 块数量基本一致。
sh.enableBalancing("school.student")

// 开启均衡调度
sh.startBalancer()

// 查看详细分片信息
sh.status({"verbose":1})

// 查看该表的分片数据信息
db.student.getShardDistribution()
```



# 常见面试题

### 什么是mongodb？

MongoDB是一个文档数据库，提供好的性能，领先的非关系型数据库。采用BSON存储文档数据。

BSON（）是一种类json的一种二进制形式的存储格式，简称Binary JSON.

相对于json多了date类型和二进制数组。

### 为什么用MOngoDB？

1.  架构简单，不需要固定结构

2.  没有复杂的连接，没有复杂的外键约束

3.  深度查询能力,MongoDB支持动态查询。

4.  容易调试

5.  容易扩展

6.  不需要转化/映射应用对象到数据库对象

7.  使用内部内存作为存储工作区,以便更快的存取数据。

### mongodb的应用场景

1.  大数据
2.  内容管理系统
3.  移动端Apps
4.  数据管理
5.  日志处理
6.  游戏道具处理

### mongodb的日志你是如何处理的？

```javascript
show logs
show logs global

```



### MongoDB中的命名空间是什么意思?

mongodb存储bson对象在丛集(collection)中。数据库名字和丛集名字以句点连结起来叫做名字空间(namespace)。一个集合命名空间又有多个数据域(extent)，集合命名空间里存储着集合的元数据，比如集合名称，集合的

第一个数据域和最后一个数据域的位置等等。而一个数据域由若干条文档(document)组成，每个数据域都有一个

头部，记录着第一条文档和最后一条文档的为知，以及该数据域的一些元数据。extent之间，document之间通过双向链表连接。索引的存储数据结构是B树，索引命名空间存储着对B树的根节点的指针。

### monogodb 中的分片什么意思

分片是将数据水平切分到不同的物理节点。当应用数据越来越大的时候，数据量也会越来越大。当数据量增长时，单台机器有可能无法存储数据或可接受的读取写入吞吐量。利用分片技术可以添加更多的机器来应对数据量增加以及读写操作的要求。

### Mongodb的一些基本操作命令（列举一些常用命令即可）？

1.  db.help(); Help 查看命令提示

2.  use yourDB; 切换/创建数据库

3.  show dbs; 查询所有数据库

4.  db.dropDatabase(); 删除当前使用数据库

5.  db.getName(); 查看当前使用的数据库

6.  db.version(); 当前 db 版本

7.  db.addUser("name"); 添加用户

8.  db.addUser("userName", "pwd123", true);

9.  show users; 显示当前所有用户

10.  db.removeUser("userName"); 删除用户

11.  db.yourColl.count(); 查询当前集合的数据条数

### 什么是集合(表)

集合就是一组 MongoDB 文档。它相当于关系型数据库（RDBMS）中的表这种概念。集合位于单独的一个数据库中。

一个集合内的多个文档可以有多个不同的字段。一般来说，集合中的文档都有着相同或相关的目的。

### 什么是文档(记录)

文档由一组key value组成。文档是动态模式,这意味着同一集合里的文档不需要有相同的字段和结构。在关系型数据库中table中的每一条记录相当于MongoDB中的一个文档。

### MongoDB和关系型数据库术语对比图

| Mongodb    | relational database |
| ---------- | ------------------- |
| database   | database            |
| collection | table               |
| document   | record/row          |
| field      | column              |

### Python 中调用 mongodb 数据库的包叫什么？

同步库：Pymongo、MongoEngine

异步库：Moter

详情参考：https://www.cnblogs.com/Neeo/articles/14271710.html

### MongoDB 成为优秀的 NoSQL 数据库的原因是什么?

类似的问题包括：使用 MongoDB 的优点？

以下特点使得 MongoDB 成为优秀的 NoSQL 数据库：

1.  面向文档，以 JSON 格式的文档保存数据

2.  高性能

3.  高可用性

4.  易扩展性

5.  丰富的查询语言

6.  可分片

7.  对数据存储友好

8.  任何属性都可以建立索引

### MongoDB支持主键外键关系吗

默认MongoDB不支持主键和外键关系。 用Mongodb本身的API需要硬编码才能实现外键关联，不够直观且难度较大

### MongoDB支持哪些数据类型

1.  String

2.  Integer

3.  Double

4.  Boolean

5.  Object

6.  Object ID

7.  Arrays

8.  Min/Max Keys

9.  Datetime

10.  Code

11.  Regular Expression等

### 为什么要在MongoDB中用"Code"数据类型

"Code"类型用于在文档中存储 JavaScript 代码。

### 为什么要在MongoDB中用"Regular Expression"数据类型

"Regular Expression"类型用于在文档中存储正则表达式

### 为什么在MongoDB中使用"Object ID"数据类型

"ObjectID"数据类型用于存储文档id

### "ObjectID"有哪些部分组成(2)

改题目也有这么问的：mongodb的objectid包含哪些信息？

一共有四部分组成：时间戳、客户端ID、客户进程ID、三个字节的增量计数器

### 在MongoDb中什么是索引

索引用于高效的执行查询,没有索引的MongoDB将扫描整个集合中的所有文档,这种扫描效率很低,需要处理大量的数据.  索引是一种特殊的数据结构,将一小块数据集合保存为容易遍历的形式.索引能够存储某种特殊字段或字段集的值,并按照索引指定的方式将字段值进行排序.

### mongodb索引基于什么数据结构实现？

>   https://docs.mongodb.com/v3.6/indexes/

答：B+tree

### 如何添加索引

使用db.collection.createIndex()在集合中创建一个索引

###  MongoDB索引有哪几种类型？

>   https://www.cnblogs.com/Neeo/articles/14325130.html

MongoDB中支持以下几种索引类型：

-   单列索引(Single Field)。
-   复合索引(Compound Indexes)。
-   多键索引(Multikey Indexes)。
-   文本索引(Text Indexes)。
-   哈希索引(Hashed Indexes)。

### 如何查询集合中的文档

```javascript
db.collectionName.find({key:value})
```

### 用什么方法可以格式化输出结果

```javascript
db.collectionName.find().pretty()
```

### 如何使用"AND"或"OR"条件循环查询集合中的文档

```javascript
db.mycol.find({   
    "$or": [key1: value1}, {key2:value2}]  
}).pretty()
```



### 更新数据

```javascript
db.collectionName.update({key:value},{$set:{newkey:newValue}})
```

### 如何删除文档

```javascript
db.collectionName.remove({key:value})
```

### 在MongoDB中如何排序

使用 1 和 -1 来指定排序方式，其中 1 表示升序，而 -1 表示降序。

```javascript
db.connectionName.find({key:value}).sort({columnName:1})
```

### 什么是聚合

聚合操作能够处理数据记录并返回计算结果。聚合操作能将多个文档中的值组合起来，对成组数据执行各种操作，返回单一的结果。它相当于 SQL 中的 count(*) 组合 group by。对于 MongoDB 中的聚合操作，应该使用aggregate()方法。

```javascript
db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```

### 在MongoDB中什么是副本集（避免单点故障）

在MongoDB中副本集由一组MongoDB实例组成，包括一个主节点多个次节点，MongoDB客户端的所有数据都写入主节点(Primary),副节点从主节点同步写入数据，以保持所有复制集内存储相同的数据，提高数据可用性。

### 分析器在 MongoDB 中的作用是什么?

相似的问题：为什么要在MongoDB中使用分析器？

MongoDB 中包括了一个可以显示数据库中每个操作性能特点的数据库分析器。通过这个分析器你可以找到比预期慢的查询(或写操作);利用这一信息，比如，可以确定是否需要添加索引 explain()

### 怎么查看 MongoDB 正在使用的链接?

```javascript
db._adminCommand("connPoolStats");
```

### MongoDB支持存储过程吗？如果支持的话，怎么用？

MongoDB支持存储过程，它是javascript写的，保存在db.system.js集合中。

### 如何理解MongoDB中的GridFS机制，MongoDB为何使用GridFS来存储文件？

GridFS是一种将大型文件存储在MongoDB中的文件规范。使用GridFS可以将大文件分隔成多个小文档存放，这样我们能够有效的保存大文档，而且解决了BSON对象有限制的问题

### 为什么MongoDB的数据文件很大？

MongoDB采用的预分配空间的方式来防止文件碎片。

### 当更新一个正在被迁移的块（Chunk）上的文档时会发生什么？

更新操作会立即发生在旧的块（Chunk）上，然后更改才会在所有权转移前复制到新的分片上。

###  如果用户移除对象的属性,该属性是否从存储层中删除?

是的,用户移除属性然后对象会重新保存(re-save()).

### 更新操作立刻fsync到磁盘?

不会,磁盘写操作默认是延迟执行的.写操作可能在两三秒(默认在60秒内)后到达磁盘.例如,如果一秒内数据库收到一千个对一个对象递增的操作,仅刷新磁盘一次.

### 如何执行事务/加锁?

mongodb没有使用传统的锁或者复杂的带回滚的事务,因为它设计的宗旨是轻量,快速以及可预计的高性能.可以把它类比成mysql mylsam的自动提交模式.通过精简对事务的支持,性能得到了提升,特别是在一个可能会穿过多个服务器的系统里.

 

### 启用备份故障恢复需要多久?

从备份数据库声明主数据库宕机到选出一个备份数据库作为新的主数据库将花费10到30秒时间.这期间在主数据库上的操作将会失败–包括写入和强一致性读取(strong consistent read)操作.然而,你还能在第二数据库上执行最终一致性查询(eventually consistent query)(在slaveok模式下),即使在这段时间里.

 

### 什么是master或primary?

它是当前备份集群(replica set)中负责处理所有写入操作的主要节点/成员.在一个备份集群中,当失效备援(failover)事件发生时,一个另外的成员会变成primary

### 我应该启动一个集群分片(sharded)还是一个非集群分片的 mongodb 环境?

(数据量大用集群分片,数据量小用非集群)

为开发便捷起见,我们建议以非集群分片(unsharded)方式开始一个 mongodb 环境,除非一台服务器不足以存放你的初始数据集.从非集群分片升级到集群分片(sharding)是无缝的,所以在你的数据集还不是很大的时候没必要考虑集群分片(sharding).

### 分片(sharding)和复制(replication)是怎样工作的?

每一个分片(shard)是一个分区数据的逻辑集合.分片可能由单一服务器或者集群组成,我们推荐为每一个分片(shard)使用集群.

 

### 数据在什么时候才会扩展到多个分片(shard)里?

mongodb 分片是基于区域(range)的.所以一个集合(collection)中的所有的对象都被存放到一个块(chunk)中.只有当存在多余一个块的时候,才会有多个分片获取数据的选项.现在,每个默认块的大小是 64mb,所以你需要至少 64 mb 空间才可以实施一个迁移.

### 如果在一个分片(shard)停止或者很慢的时候,我发起一个查询会怎样?

如果一个分片(shard)停止了,除非查询设置了“partial”选项,否则查询会返回一个错误.如果一个分片(shard)响应很慢,mongodb则会等待它的响应.

### 可以把movechunk目录里的旧文件删除吗?

没问题,这些文件是在分片(shard)进行均衡操作(balancing)的时候产生的临时文件.一旦这些操作已经完成,相关的临时文件也应该被删除掉.但目前清理工作是需要手动的,所以请小心地考虑再释放这些文件的空间.

### 如果块移动操作(movechunk)失败了,我需要手动清除部分转移的文档吗?

不需要,移动操作是一致(consistent)并且是确定性的(deterministic);一次失败后,移动操作会不断重试;当完成后,数据只会出现在新的分片里(shard).

 

### MySQL 与 MongoDB 本质之间最基本的差别是什么？

差别在多方面，例如：数据的表示、查询、关系、事务、模式的设计和定义、速度和性能。

MongoDB 是由 C++语言编写的，是一个基于分布式文件存储的开源数据库系统。在高负载的

情况下，添加更多的节点，可以保证服务器性能。

MongoDB 旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。

MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类

似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

MongoDB 是一个面向文档的数据库，目前由 10gen 开发并维护，它的功能丰富齐全，所以完全可以替代 MySQL。

与 MySQL 等关系型数据库相比，MongoDB 的优点如下：

1.  弱一致性，更能保证用户的访问速度。

2.  文档结构的存储方式，能够更便捷的获取数据。

3.  内置 GridFS，支持大容量的存储。

4.  内置 Sharding。

5.  第三方支持丰富。(这是与其他的 NoSQL 相比，MongoDB 也具有的优势)

6.  性能优越

MongoDB 相对于MySQL，它还算比较年轻的一个产品，所以它的问题，就是成熟度肯定没有传统 MySQL那么成熟稳定。所以在使用的时候，第一，尽量使用稳定版，不要在线上使用开发版，这是一个大原则；

另外一点，备份很重要，MongoDB 如果出现一些异常情况，备份一定是要能跟上。除了通过传统的复制的方式来做备份，离线备份也还是要有，不管你是用什么方式，都要有一个完整的离线备份。往往最后出现了特殊情况，它能帮助到你；

另外，MongoDB 性能的一个关键点就是索引，索引是不是能有比较好的使用效率，索引是不是能够放在内存中，这样能够提升随机读写的性能。如果你的索引不能完全放在内存中，一旦出现随机读写比较高的时候，它就会频繁地进行磁盘交换，这个时候，MongoDB 的性能就会急剧下降，会出现波动。

另外，MongoDB 还有一个最大的缺点，就是它占用的空间很大，因为它属于典型空间换时间原则的类型。那么它的磁盘空间比普通数据库会浪费一些，而且到目前为止它还没有实现在线压缩功能，在 MongoDB 中频繁的进行数据增删改时，如果记录变了，例如数据大小发生了变化，这时候容易产生一些数据碎片，出现碎片引发的结果，一个是索引会出现性能问题，另外一个就是在一定的时间后，所占空间会莫明其妙地增大，所以要定期把数据库做修复，定期重新做索引，这样会提升 MongoDB 的稳定性和效率。