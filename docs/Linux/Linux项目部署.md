## 项目部署的相关概念

关于部署流程，主要包含以下3个方面：部署方案、部署环境、部署架构。

### 部署方案

```
1. 分析项目的产品需求文档，定好部署方案的方向
2. 分析项目开发文档，按照功能边界，设计部署的结点
3. 分析项目功能软件，合理的取舍，选符合当前业务场景的
4. 梳理项目部署涉及到的部署软件实现方案，根据上面第2点确定的结点，确定初版部署方案
5. 根据项目实际情况，调整优化并确定项目部署方案【短期内的持续方案】

devops  CI/CD
```



### 部署环境

##### 个人开发环境：

```
工作人员：自己
工作平台：个人笔记本、公司配的电脑
平台特点：根据公司的项目要求，环境是自己配的，团队中不同的个人开发环境可以不一样
工作内容：项目的子模块，子功能的开发和维护
完成标准：完成领导/leeder安排的内容[项目的功能子模块开发]
```

##### 公司开发环境：

```
工作人员：开发团队
工作平台：公司内部服务器
平台特点：服务器环境和线上的服务器环境完全一致
工作内容：项目子模块间的功能联调
完成标准：项目阶段开发、调试完成
```

##### 项目测试环境：

```
工作人员：测试团队
工作平台：公司内部服务器
平台特点：服务器环境和线上的服务器环境完全一致
工作内容：项目功能/非功能/探索等测试
完成标准：项目阶段功能正常运行
```

##### 项目预发布环境：

```
工作人员：运维团队
工作平台：公司线上服务器组中的一台
平台特点：服务器环境和线上的服务器环境完全一致
工作内容：特殊功能测试(比如支付)、数据压力测试、其他安全测试等
完成标准：项目阶段功能正常运行，最后一道防线
```

##### 项目生产环境：

```
工作人员：运维团队
工作平台：公司线上服务器组
平台特点：标准线上的服务器环境
工作内容：代码部署和维护、记录内部架构文档
完成标准：项目正常运行
```



![1563422044771](Linux项目部署.assets/1563422044771.png)



### 项目架构

单点部署：鲁棒性弱，容灾性低，不够健壮。

![1558922797316](Linux项目部署.assets/1558922797316.png)

分布式集群架构

![1591319646819](Linux项目部署.assets/1591319646819.png)

如果需要多个客户端对外提供服务，则架构如下：

![image-20211220122332759](Linux项目部署.assets/image-20211220122332759.png)



### 服务器相关

服务器： 根据节点来衡量，计算需要多少台服务器才能承受指定的QPS【每秒请求数】访问压力。

服务器类型：物理服务器，云服务器

并发瓶颈：

```python
短板原理[水桶原理]
1. 带宽：200M/40
2. 处理并发的能力[web服务器], nginx 客户端静态资源代理-> 4C4M上3-4W，Uwsgi/Gunicorn/Uvincorn -> 3000-5000
3. 数据库查询能力：
   mysql 默认最大连接数151个，最高可以达到16384个，处理QPS的能力，默认大概1000次/s 。    
   show variables like '%max_connections%';

   redis 默认是1W的连接数，最高可以配置到10W，并发处理能力与连接数有关，默认1W连接的话，5W/s。
   CONFIG GET maxclients;

4. 物理服务器本身的承载能力[8核16G起步...]
```

可以使用apache ab工具进行压测，根据短板效应得到系统的并发处理数的乐观数据。

域名影射

```bash
39.105.40.148 api.fuguang.jiangw.cc    api服务端
39.105.40.148 www.fuguang.jiangw.cc    web客户端
```

![image-20210903125251727](Linux项目部署.assets/image-20210903125251727.png)

注意：域名要正常使用必须要绑定服务器以后，进行ICP（工信部）备案和公安部备案，否则会出现无法通过域名访问的情况。

## 使用docker进行容器化集成部署

### 服务器更新源

更新ubuntu的apt源

```shell
sudo apt-get update
```

安装包允许apt通过HTTPS使用仓库

```shell
sudo dpkg --configure -a
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```

添加Docker官方GPG key【这个是国外服务器地址，所以网路不好的时候，会失败！在网路好的情况下，多执行几次就没问题了，如果下一行结果不是OK，则执行多几遍】

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```



设置Docker稳定版仓库

```shell
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

添加仓库后，更新apt源索引

```shell
sudo apt-get update
```

前面的准备工作完成以后，接下来安装最新版Docker CE（社区版）

```shell
sudo apt-get install -y docker-ce
```

检查Docker CE是否安装正确

```shell
sudo docker pull mysql:8.0.27
```

出现了以下图中内容则表示上面的安装成功！

![image-20210906085623708](Linux项目部署.assets/image-20210906085623708.png)

我们获取镜像文件，可以直接去官方网站上获取: https://hub.docker.com/



### 2. 启动与停止

安装完成Docker后，默认已经启动了docker服务，如需手动控制docker服务的启停，可执行如下命令

```shell
# 启动docker
sudo service docker start

# 停止docker
sudo service docker stop

# 重启docker
sudo service docker restart

# 列出镜像
sudo docker images
# podman images

# 拉取镜像
sudo docker pull <镜像名:版本号>
# podman pull <镜像名:版本号>

# 删除镜像
sudo docker rmi <镜像名:版本号/镜像id>  
# podman rmi <镜像名:版本号/镜像id>

# 创建容器
sudo docker run [选项参数] <镜像名:版本号/镜像ID> --name <容器名> [命令，可选]
# podman run [选项参数] <镜像名:版本号/镜像ID> [命令，可选]

# 停止一个已经在运行的容器
sudo docker stop <容器id/容器名>
# podman stop <容器id/容器名>

# 启动一个已经停止的容器
sudo docker start <容器id/容器名>
# podman start <容器id/容器名>

# kill掉一个已经在运行的容器
sudo docker kill <容器id/容器名>
# podman kill <容器id/容器名>

# 删除容器
sudo docker rm <容器id/容器名>
# podman container rm <容器id/容器名>

# 查看容器运行的日志信息
sudo docker log <容器id/容器名>
# podman log <容器id/容器名>

# 查看容器运行的端口
sudo docker port <容器id/容器名>

# 查看容器运行的主机信息
sudo docker inspect <容器id/容器名>
```

服务端和客户端的地址分配

```
39.105.40.148 www.fuguang.jiangw.cc
39.105.40.148 api.fuguang.jiangw.cc
```

安装docker-compose

```bash
apt install -y docker-compose
```



## 前端项目部署

前端自动化项目构建工具vite提供了一个build命令给我们可以将前端项目中的所有前端资源进行打包成普通的HTML/CSS/JS，生成到项目的dist文件夹中。这个dist目录下的内容会剔除掉vite中所有的测试服务器运行相关依赖包，只保留项目本身的代码和相关文件，这样将来我们在线上服务器运行时就可以直接通过nginx来运行，而不需要依赖于vite工具。

执行build命令进行项目编译之前，先修改配置的文件的信息，例如当前客户端的域名和api服务端的域名。src/settings.js，代码：

```javascript
export default {
    // api服务端所在地址
    host: "http://api.fuguang.jiangw.cc",
    // 防水墙验证码的应用ID[替换成公司的应用ID]
    captcha_app_id: "2019894193",
    // 订单超时时间[单位：秒]
    order_timeout: 15 * 60,
    // 视频更新进度的时间阈值
    seed_time: 5
}
```

src/utils/https.js，代码：

```javascript
import axios from "axios"
import settings from "../settings.js"

const http = axios.create({
    baseURL: settings.host,      // 设置api服务端的默认地址[如果基于服务端实现的跨域，这里可以填写api服务端的地址，如果基于nodejs客户端测试服务器实现的跨域，则这里不能填写api服务端地址]
    withCredentials: false,                    // 是否允许客户端ajax请求时携带cookie
})
```

终端下，在前端项目的根目录下，执行以下命令：

```bash
cd fuguangweb
yarn upgrade
vite build
```

效果：

![image-20210906090853134](Linux项目部署.assets/image-20210906090853134.png)

编译完成的内容会被保存在dist目录下，并且编译出来的项目不能直接通过file://协议来访问，只能通过http服务器的运行这个项目。同时，我们之前如果通过node进行跨域代理的话，则编译后会丢失跨域代理，此时我们就需要依靠后台实现cors代理了。

![image-20210906090956993](Linux项目部署.assets/image-20210906090956993.png)

通过scp上传代码到线上服务器

```bash
# scp 本地路径 账号@远程地址:线上路径 
# 本地Ubuntu终端
cd ~/Desktop/fuguang/fuguangweb
tar -zcf fuguangweb.tar.gz dist
scp ~/Desktop/fuguang/fuguangweb/fuguangweb.tar.gz root@39.105.40.148:/home/

# 线上服务器操作[阿里云终端]
cd /home
tar -zxf fuguangweb.tar.gz
mkdir /home/fuguang
mv /home/dist /home/fuguang/fuguangweb
```



因为根据我们上面的部署方案，我们需要安装nginx来运行这个前端项目。

### 目录结构

服务器，目录结构：

```bash
/home/fuguang/
├── conf/
│   └── nginx/
│       └── web.conf
├── docker-compose.yml
└── fuguangweb/
```

阿里云服务器终端操作：

```bash
mkdir -p /home/fuguang/conf/nginx/
touch /home/fuguang/conf/nginx/web.conf
vim /home/fuguang/docker-compose.yml
```

### 编排 nginx服务容器

/home/fuguang/docker-compose.yml，代码：

```python
version: '3.7'
services:
  web1:
    image: nginx:1.21.4
    restart: always
    ports:
      - "80:80"
    container_name: "web1"
    volumes:
      - ./fuguangweb:/home/fuguangweb
      - ./conf/nginx/web.conf:/etc/nginx/conf.d/nginx.conf
    networks:
      - default
networks:
  default:
```

/home/fuguang/conf/nginx/web.conf，这个文件是准备给nginx容器内部的虚拟主机配置文件

```bash
server {
    listen       80;
    server_name  www.fuguang.jiangw.cc;
    # client_max_body_size    1000M;
    # root         /opt/dist;

    # client_max_body_size 20M;

    # Load configuration files for the default server block.

    location / {
        root  /home/fuguangweb;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
```

在阿里云终端启动容器服务

```bash
docker-compose -f docker-compose.yml up -d
```



## 后端项目部署

根据我们之前分析的项目部署架构，后端需要调用mysql、redis和elasticsearch。因此我们需要先把这些外部软件先预装。

### 目录结构

```bash
/home/fuguang/
├── conf/
│   ├── redis/
│   │   └── master.conf 
│   ├── mysql/
│   └── nginx/
│       └── web.conf
├── data/
│   ├── redis/
│   ├── elastic/
│   └── mysql/
├── docker-compose.yml
├── fuguangweb/
├── initdb/
│   └── fuguang.sql
├── logs/
│   └── mysql
└── plugins/
```

### 编排mysql服务容器

创建目录结构

```bash
mkdir -p data/mysql
mkdir -p logs/mysql
mkdir -p initdb
```

/home/fuguang/docker-compose.yml，代码：

```python
version: '3.7'
services:
  web1:
    image: nginx:1.21.4
    restart: always
    ports:
      - "80:80"
    container_name: "web1"
    volumes:
      - ./fuguangweb:/home/fuguangweb
      - ./conf/nginx/web.conf:/etc/nginx/conf.d/nginx.conf
    networks:
      - default

  mysql_master:
    image: mysql:8.0.26
    restart: always
    container_name: mysql_master
    networks:
      - default
    environment:
      - "MYSQL_ROOT_PASSWORD=root2021"
      - "MYSQL_USER=fuguang"
      - "MYSQL_PASSWORD=fuguang"
      - "MYSQL_DATABASE=fuguang"
      - "TZ=Asia/Shanghai"
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
    ports:
      - 3306:3306
    volumes:
      - ./data/mysql:/var/lib/mysql
      - ./logs/mysql:/logs
      - ./initdb:/docker-entrypoint-initdb.d/

networks:
  default:
```



#### 把本地的数据导入到SQL文件并上传到服务器

```python
# 把本地的数据库导出到桌面
mysqldump -uroot -p123 fuguang > ~/Desktop/fuguang/fuguangapi/fuguang.sql

# 把fuguangapi目录下导出的数据库文件fuguang.sql导入到远程阿里元服务器下的docker容器中的mysql数据库
scp ~/Desktop/fuguang/fuguangapi/fuguang.sql root@39.105.40.148:/home/fuguang/initdb/
```

启动服务

```bash
docker-compose -f docker-compose.yml down
docker-compose -f docker-compose.yml up -d

# 注意：如果mysql容器要初始化数据库，则必须要保证在运行mysql容器之前，data目录是没有任何内容的！！！
# rm -rf /home/fuguang/data/

# 可以通过以下命令进入mysql容器
docker exec -it mysql_master mysql -uroot -proot2021
use fuguang
show tables;
```



### 编排redis服务容器

创建目录和必须的文件

```bash
mkdir conf/redis
touch conf/redis/master.conf
mkdir -p data/redis/master
vim conf/redis/master.conf
```

conf/redis/master.conf，主库配置文件，代码：

```bash
requirepass 123456
masterauth 123456
bind 0.0.0.0

# RDB快照
save 900 1
save 300 10
save 60 10000
dir "/data"
rdbcompression yes
dbfilename dump.rdb

# AOF日志
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
```



/home/fuguang/docker-compose.yml，代码：

```python
version: '3.7'
services:
  web1:
    image: nginx:1.21.4
    restart: always
    ports:
      - "80:80"
    container_name: "web1"
    volumes:
      - ./fuguangweb:/home/fuguangweb
      - ./conf/nginx/web.conf:/etc/nginx/conf.d/nginx.conf
    networks:
      - default

  mysql_master:
    image: mysql:8.0.26
    restart: always
    container_name: mysql_master
    networks:
      - default
    environment:
      - "MYSQL_ROOT_PASSWORD=root2021"
      - "MYSQL_USER=fuguang"
      - "MYSQL_PASSWORD=fuguang"
      - "MYSQL_DATABASE=fuguang"
      - "TZ=Asia/Shanghai"
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
    ports:
      - 3306:3306
    volumes:
      - ./data/mysql:/var/lib/mysql
      - ./logs/mysql:/logs
      - ./initdb:/docker-entrypoint-initdb.d/

  redis_master:
    image: redis:6.2.6-alpine3.15
    restart: always
    networks:
      - default
    ports:
      - "6379:6379"
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    container_name: redis_master
    volumes:
      - ./conf/redis/master.conf:/usr/local/etc/redis/redis.conf
      - ./data/redis/master:/data
    
networks:
  default:
```

启动服务

```bash
docker-compose -f docker-compose.yml down

docker-compose -f docker-compose.yml up -d
# 测试
docker exec -it redis_master redis-cli -a 123456
```



### 编排elasticsearch搜索引擎服务容器

在网络安全组中，开放9200,9300这2个端口

![image-20210906103302159](Linux项目部署.assets/image-20210906103302159.png)

/home/fuguang/docker-compose.yml，代码：

```python
version: '3.7'
services:
  web1:
    image: nginx:1.21.4
    restart: always
    ports:
      - "80:80"
    container_name: "web1"
    volumes:
      - ./fuguangweb:/home/fuguangweb
      - ./conf/nginx/web.conf:/etc/nginx/conf.d/nginx.conf
    networks:
      - default

  mysql_master:
    image: mysql:8.0.26
    restart: always
    container_name: mysql_master
    networks:
      - default
    environment:
      - "MYSQL_ROOT_PASSWORD=root2021"
      - "MYSQL_USER=fuguang"
      - "MYSQL_PASSWORD=fuguang"
      - "MYSQL_DATABASE=fuguang"
      - "TZ=Asia/Shanghai"
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
    ports:
      - 3306:3306
    volumes:
      - ./data/mysql:/var/lib/mysql
      - ./logs/mysql:/logs
      - ./initdb:/docker-entrypoint-initdb.d/

  redis_master:
    image: redis:6.2.6-alpine3.15
    restart: always
    networks:
      - default
    ports:
      - "6379:6379"
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    container_name: redis_master
    volumes:
      - ./conf/redis/master.conf:/usr/local/etc/redis/redis.conf
      - ./data/redis/master:/data

  elasticsearch:
    image: elasticsearch:7.16.1
    restart: always
    environment:
      - "discovery.type=single-node"
      - "bootstrap.memory_lock=true"
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - "xpack.security.enabled=false"
      - "http.cors.enabled=true"
      - "http.cors.allow-origin=*"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./data/elastic:/usr/share/elasticsearch/data
      - ./logs/elastic:/var/log/elasticsearch
      - ./plugins:/usr/share/elasticsearch/plugins
    ports:
      - 9200:9200
    networks:
      - default
networks:
  default:
```

把ik分词器下载解压到/home/fuguang/plugins/目录下。

```bash
mkdir -p /home/fuguang/data/elastic
mkdir -p /home/fuguang/plugins
chmod 777 /home/fuguang/data/elastic

# 下载ik中文分词插件
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.16.1/elasticsearch-analysis-ik-7.16.1.zip

# 解压保存到es的插件目录
unzip elasticsearch-analysis-ik-7.16.1.zip -d /home/fuguang/plugins/ik-7.16.1

docker-compose -f docker-compose.yml up

# postman中测试访问
post http://39.105.40.148:9200/_analyze?pretty

{
  "analyzer": "ik_max_word",
  "text": "我是中国人，我来自北京。"
}
```

启动服务

```bash
docker-compose -f docker-compose.yml down
docker-compose -f docker-compose.yml up -d
```

es索引的生成是要结合django项目的，所以暂时先不用执行其他操作，先完成django项目的部署先。



### 编排uwsgi运行服务端项目容器服务

#### 基于Dockerfile定制服务端项目镜像

##### 把fuguangapi服务端项目进行部署前的修改和调整

```python
1. 本地的项目配置，保存在dev.py中，不管mysql数据库密码或者redis的地址或者配置信息都是属于开发阶段，
   项目上线以后肯定换成公司的。所以mysql数据库等信息一定会修改，因为将来还需要本地继续开发新的功能业务，
   所以不能直接修改dev.py，因此我们需要把dev.py的配置内容复制粘贴一份到pro.py中，并修改对应的配置信息，
   改成公司的。

2. 在本地开发时，我们使用的django框架运行在debug模式下的，但是项目上线采用uwsgi运行时肯定会关闭debug模式的，
   而如果关闭了debug模式，则django框架就不再提供静态文件的浏览服务，也就是之前simpleUI，drf的接
   口调试页面的css/js/图片都不能访问了。我们需要使用django提供的命令提前收集这些静态文件到项目的
   指定目录下，后面通过nginx来对外提供http浏览服务

3. 原来的drf项目是运行在python内置的提供的web服务器中，wsgiref
   在项目上线的时候，我们不能使用这个模块来提供对外服务!
   wsgiref不支持多线程多进程，本身性能不好，本身在安全性上并不完善，
   python提供wsgiref这个模块仅供学习和开发测试使用。
   runserver 不能用于生产, 改成uwsgi, gunicorn, uvicorn

4. 原来drf项目在本地开发时，其实要运行这个项目，我们是安装了很多的第三方依赖模块。
   现在项目上线了，我们也肯定需要把本地的模块同步到线上服务器！否则项目跑不起来。
   因此需要使用pip freeze > requirements.txt 把项目运行的模块列表以及它们的版本
   记录到requirements.txt文件中，然后在线上服务器对requirements.txt执行重新安装模块
```



###### 修改现有的配置文件

在项目中复制开发配置文件dev.py 到生产配置pro.py

修改配置中的IP地址为公网地址，所有的127.0.0.1全部换成公网地址，所有域名替换成真实域名。

所有的第三方接口账号全部换成公司的账号信息。

settings.pro，代码：

```python
"""
Django settings for fuguangapi project.

Generated by 'django-admin startproject' using Django 3.2.9.

For more information on this file, see
https://docs.djangoproject.com/en/3.2/topics/settings/

For the full list of settings and their values, see
https://docs.djangoproject.com/en/3.2/ref/settings/
"""
import sys
from pathlib import Path

# __file__  是一个魔术变量，代表的是当前文件的路径
# Build paths inside the project like this: BASE_DIR / 'subdir'.
# 当前项目的主应用开发目录
BASE_DIR = Path(__file__).resolve().parent.parent
# 新增apps作为导包路径，导包路径默认保存sys.path属性中，所有的python的import或者from导包语句默认都是从sys.path中记录的路径下查找模块
sys.path.insert(0, str( BASE_DIR / "apps"))
sys.path.insert(0, str( BASE_DIR / "utils"))
# print(sys.path)

# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/3.2/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'django-insecure--g(e&jha#ednlr6yppwe(*gk=l3e=7+0m37i!a8^*^6o2epgym'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = False

# 是否处于测试环境下
IS_TEST = False

ALLOWED_HOSTS = ["*"]


# Application definition

INSTALLED_APPS = [
    'simpleui',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'rest_framework',
    'corsheaders',  # cors跨域子应用
    'ckeditor',     # 富文本插件核心应用
    'ckeditor_uploader',  # 用于支持富文本插件上传文件使用的
    'stdimage',     # 生成缩略图
    'haystack',

    'home',
    'users',
    'courses',
    'cart',
    'orders',
    'coupon',
    'payments',
]


MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',  # cors跨域的中间件
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware', # 认证用户登录状态的中间件
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'fuguangapi.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            BASE_DIR / "templates",  # BASE_DIR 是apps的父级目录，是主应用目录，templates需要手动创建
        ],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'fuguangapi.wsgi.application'


# Database
# https://docs.djangoproject.com/en/3.2/ref/settings/#databases
# mysql / postgreSQL
DATABASES = {
    'default': {  # master主库
        'ENGINE': 'django.db.backends.mysql',
        # 'ENGINE': 'dj_db_conn_pool.backends.mysql',
        'NAME': 'fuguang',
        'USER': 'fuguang_user',
        'PASSWORD': 'fuguang',
        'HOST': '39.105.40.148',
        'PORT': 3306,
        # 'POOL_OPTIONS' : {      # 连接池的配置信息
        #     'POOL_SIZE': 10,    # 连接池默认创建的链接对象的数量
        #     'MAX_OVERFLOW': 10  # 连接池默认创建的链接对象的最大数量
        # },
    },
    'read1': {    # slave01从库
        'ENGINE': 'django.db.backends.mysql',
        # 'ENGINE': 'dj_db_conn_pool.backends.mysql',
        'NAME': 'fuguang',
        'USER': 'fuguang_user',
        'PASSWORD': 'fuguang',
        'HOST': '39.105.40.148',
        'PORT': 3306,
        # 'POOL_OPTIONS': {  # 连接池的配置信息
        #     'POOL_SIZE': 10,  # 连接池默认创建的链接对象的数量
        #     'MAX_OVERFLOW': 10  # 连接池默认创建的链接对象的最大数量
        # },
    },

}

# 自动数据库路由
DATABASE_ROUTERS = ["fuguangapi.db_router.Router"]

# redis configration
# https://django-redis-chs.readthedocs.io/zh_CN/latest/#
# 设置redis缓存
CACHES = {
    # 默认缓存
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        # 项目上线时,需要调整这里的路径
        "LOCATION": "redis://:123456@39.105.40.148:6379/0",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    },
    # 提供给admin站点的session存储
    "session": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://:123456@39.105.40.148:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    },
    # 提供存储短信验证码
    "sms_code":{
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://:123456@39.105.40.148:6379/2",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    },
    # 提供存储搜索热门关键字
    "hot_word": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://:123456@39.105.40.148:6379/3",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    },
    # 提供存储购物车课程商品
    "cart": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://:123456@39.105.40.148:6379/4",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    },
    # 提供存储优惠券
    "coupon": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://:123456@39.105.40.148:6379/5",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    },
}

# 设置用户登录admin站点时,记录登录状态的session保存到redis缓存中
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
# 设置session保存的位置对应的缓存配置项
SESSION_CACHE_ALIAS = "session"

# Password validation
# https://docs.djangoproject.com/en/3.2/ref/settings/#auth-password-validators
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/3.2/topics/i18n/

LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True

# 关闭时区转换以后，django会默认使用TIME_ZONE作为时区。
USE_TZ = False


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.2/howto/static-files/

STATIC_URL = '/static/'

# 设置django的静态文件目录[手动创建]
STATICFILES_DIRS = [
    BASE_DIR / "static",
]

# 项目中存储上传文件的根目录[手动创建]，注意，uploads目录需要手动创建否则上传文件时报错
MEDIA_ROOT = BASE_DIR / "uploads"
# 访问上传文件的url地址前缀
MEDIA_URL = "/uploads/"

# Default primary key field type
# https://docs.djangoproject.com/en/3.2/ref/settings/#default-auto-field

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'


# 日志配置
# https://docs.djangoproject.com/zh-hans/3.2/topics/logging/
LOGGING = {
    'version': 1,  # 使用的日志模块的版本，目前官方提供的只有版本1，但是官方有可能会升级，为了避免升级出现的版本问题，所以这里固定为1
    'disable_existing_loggers': False,  # 是否禁用其他的已经存在的日志功能？肯定不能，有可能有些第三方模块在调用，所以禁用了以后，第三方模块无法捕获自身出现的异常了。
    'formatters': {  # 日志格式设置，verbose或者simple都是自定义的
        'verbose': {  # 详细格式，适合用于开发人员不在场的情况下的日志记录。
            # 格式定义：https://docs.python.org/3/library/logging.html#logrecord-attributes
            # levelname 日志等级
            # asctime   发生时间
            # module    文件名
            # process   进程ID
            # thread    线程ID
            # message   异常信息
            'format': '{levelname} {asctime} {module} {funcName} {process:d} {thread:d} {message}',
            'style': '{',  # 变量格式分隔符
        },
        'simple': {  # 简单格式，适合用于开发人员在场的情况下的终端输出
            'format': '{levelname} {module} {funcName} {message}',
            'style': '{',
        },
    },
    'filters': {  # 过滤器
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': { # 日志处理流程，console或者mail_admins都是自定义的。
        'console': {
            'level': 'DEBUG', # 设置当前日志处理流程中的日志最低等级
            'filters': ['require_debug_true'], # 当前日志处理流程的日志过滤
            'class': 'logging.StreamHandler',  # 当前日志处理流程的核心类，StreamHandler可以帮我们把日志信息输出到终端下
            'formatter': 'simple'              # 当前日志处理流程的日志格式
        },
        # 'mail_admins': {
        #     'level': 'ERROR',                  # 设置当前日志处理流程中的日志最低等级
        #     'class': 'django.utils.log.AdminEmailHandler',  # AdminEmailHandler可以帮我们把日志信息输出到管理员邮箱中。
        #     'filters': ['special']             # 当前日志处理流程的日志过滤
        # }
        # 按文件大小来分割日志
        'file': {
            'level': 'DEBUG',
            'class': 'logging.handlers.RotatingFileHandler',
            # 日志位置,日志文件名，日志保存目录logs必须手动创建
            'filename': BASE_DIR.parent / "logs/fuguang.log",
            # 单个日志文件的最大值，这里我们设置300M
            'maxBytes': 300 * 1024 * 1024,
            # 备份日志文件的数量，设置最大日志数量为10
            'backupCount': 20,
            # 日志格式: 详细格式
            'formatter': 'verbose'
        },
    },
    'loggers': {  # 日志处理的命名空间
        'django': {  # 要在django中调用当前配置项的loging写入日志到文件中，名字必须叫"django"
            'handlers': ['console','file'],  # 当基于django命名空间写入日志时，调用那几个日志处理流程
            'propagate': True,   # 是否在django命名空间对应的日志处理流程结束以后，冒泡通知其他的日志功能。True表示允许
        },
    }
}

# django自定义认证
AUTHENTICATION_BACKENDS = ['fuguangapi.utils.authenticate.CustomAuthBackend', ]

# drf配置
REST_FRAMEWORK = {
    # 自定义异常处理
    'EXCEPTION_HANDLER': 'fuguangapi.utils.exceptions.exception_handler',
    # 自定义认证
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',  # jwt认证
        'rest_framework.authentication.SessionAuthentication',           # session认证
        'rest_framework.authentication.BasicAuthentication',
    ),
}

import datetime
# jwt认证相关配置项
JWT_AUTH = {
    # 设置jwt的有效期
    # 如果内部站点，例如：运维开发系统，OA，往往配置的access_token有效期基本就是15分钟，30分钟，1~2个小时
    # 'JWT_EXPIRATION_DELTA': datetime.timedelta(weeks=1),  # 一周有效，
    'JWT_EXPIRATION_DELTA': datetime.timedelta(minutes=30),  # 一周有效，
    # 自定义载荷
    'JWT_PAYLOAD_HANDLER': 'fuguangapi.utils.authenticate.jwt_payload_handler',
    # 自定义响应数据
    'JWT_RESPONSE_PAYLOAD_HANDLER': 'fuguangapi.utils.authenticate.jwt_response_payload_handler'
}

# CORS(跨域资源共享)
CORS_ALLOW_ALL_ORIGINS = True

# 自定义认证用户使用的模型
# AUTH_USER_MODEL` 参数的设置以`点.`来分隔，表示`应用名.模型类名`。
AUTH_USER_MODEL = 'users.User'


# 腾讯云API接口配置[改成公司的]
TENCENTCLOUD = {
    # 腾讯云访问秘钥ID
    "SecretId": "AKIDSggmeI7z2qSUHoaf18zb4JKdZv61PEZf",
    # 腾讯云访问秘钥key
    "SecretKey": "06xbzB7VabOyY3asztbkdIfqlovtLYXG",
    # 验证码API配置
    "Captcha": {
        "endpoint": "captcha.tencentcloudapi.com", # 验证码校验服务端域名
        "CaptchaType": 9,  # 验证码类型，固定为9
        "CaptchaAppId": 2019894193,  # 验证码应用ID
        "AppSecretKey": "02CU7YteQDtXRb2QgOAwZIg**", # 验证码应用key
    },
}

# 容联云短信[改成公司的]
RONGLIANYUN = {
    "accId": '8a216da863f8e6c20164139687e80c1b',
    "accToken": '6dd01b2b60104b3dbc88b2b74158bac6',
    "appId": '8a216da863f8e6c20164139688400c21',
    "reg_tid": 1,        # 注册短信验证码的模板ID
    "sms_expire": 300,   # 短信有效期，单位：秒(s)
    "sms_interval": 60,  # 短信发送的冷却时间，单位：秒(s)
}


# Celery异步任务队列框架的配置项[注意：django的配置项必须大写，所以这里的所有配置项必须全部大写]
# 任务队列
CELERY_BROKER_URL = 'redis://:123456@39.105.40.148:6379/14'
# 结果队列
CELERY_RESULT_BACKEND = 'redis://:123456@39.105.40.148:6379/15'
# 时区，与django的时区同步
CELERY_TIMEZONE = TIME_ZONE
# 防止死锁
CELERY_FORCE_EXECV = True
# 设置并发的worker数量
CELERYD_CONCURRENCY = 200
# 设置失败允许重试[这个慎用，如果失败任务无法再次执行成功，会产生指数级别的失败记录]
CELERY_ACKS_LATE = True
# 每个worker工作进程最多执行500个任务被销毁，可以防止内存泄漏，500是举例，根据自己的服务器的性能可以调整数值
CELERYD_MAX_TASKS_PER_CHILD = 500
# 单个任务的最大运行时间，超时会被杀死[慎用，有大文件操作、长时间上传、下载任务时，需要关闭这个选项，或者设置更长时间]
CELERYD_TIME_LIMIT = 10 * 60
# 任务发出后，经过一段时间还未收到acknowledge, 就将任务重新交给其他worker执行
CELERY_DISABLE_RATE_LIMITS = True
# celery的任务结果内容格式
CELERY_ACCEPT_CONTENT = ['json', 'pickle']

# 之前定时任务（定时一次调用），使用了apply_async({}, countdown=30);
# 设置定时任务（定时多次调用）的调用列表，需要单独运行SCHEDULE命令才能让celery执行定时任务：celery -A mycelery.main beat，当然worker还是要启动的
# https://docs.celeryproject.org/en/stable/userguide/periodic-tasks.html
from celery.schedules import crontab
CELERY_BEAT_SCHEDULE = {
    # "user-add": {  # 定时任务的注册标记符[必须唯一的]
    #     "task": "add",   # 定时任务的任务名称
    #     "schedule": 10,  # 定时任务的调用时间，10表示每隔10秒调用一次add任务
    #     # "schedule": crontab(hour=7, minute=30, day_of_week=1),,  # 定时任务的调用时间，每周一早上7点30分调用一次add任务
    # }
}

# ckeditor富文本编辑器配置
# 上传文件的存储路径
CKEDITOR_UPLOAD_PATH = "ckeditor/"

# 工具条配置
CKEDITOR_CONFIGS = {
    'default': {
        # 'toolbar': 'full', # full 显示全部工具
        # 'toolbar': 'Basic', # Basic 显示基本工具
        'toolbar': 'Custom',  # 自定义工具条的显示数量
        'toolbar_Custom': [
            ['Bold', 'Italic', 'Underline', 'Image', 'Styles', 'Format', 'Font', 'Fontsize'],
            ['NumberedList', 'BulletedList', '-', 'Outdent', 'Indent', '-', 'JustifyLeft', 'JustifyCenter',
             'JustifyRight', 'JustifyBlock'],
            ['Link', 'Unlink', 'Table'],
            ['RemoveFormat', 'Source']
        ],
        # 设置编辑器的高度
        'height': 120,
    },
}


# admin站点公共配置
from django.contrib import admin
admin.AdminSite.site_header = '浮光在线'
admin.AdminSite.site_title = '浮光在线教育站点管理'

# 登录界面logo
SIMPLEUI_LOGO = '/uploads/logo.png'
# 快速操作
SIMPLEUI_HOME_QUICK = True
# 服务器信息
SIMPLEUI_HOME_INFO = True
# 最新动作
SIMPLEUI_HOME_ACTION = True

# 关闭登录页面的离子动画
SIMPLEUI_LOGIN_PARTICLES = True

# 关闭simpleui内置的使用分析
SIMPLEUI_ANALYSIS = False

# 离线模式
SIMPLEUI_STATIC_OFFLINE = True

# 首页图标地址
SIMPLEUI_INDEX = 'http://www.fuguang.jiangw.cc/'


# 阿里云OSS云存储[改成公司的]
OSS_ACCESS_KEY_ID = "LTAI5t991uBJjk8TunKooM7M"
OSS_ACCESS_KEY_SECRET = "oEDvV9RaoCf6rHIZXlJCJAmk0phub2"
OSS_ENDPOINT = "oss-cn-beijing.aliyuncs.com"    # 访问域名, 根据服务器上的实际配置修改
OSS_BUCKET_NAME = "fuguangoline"                # oss 创建的 BUCKET 名称

# 添加下面配置后 Django admin 后台上传的 ImageField, FileField 类型的字段都会被自动上传到 oss 的服务器中, 访问路径也会自动替换
# 如果注释掉的话 oss 的配置会失效, 上传文件会存储到本地, 且访问路径也会变成本地
DEFAULT_FILE_STORAGE = 'django_oss_storage.backends.OssMediaStorage'


# haystack连接elasticsearch的配置信息
HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack.backends.elasticsearch7_backend.Elasticsearch7SearchEngine',  # haystack操作es的核心模块
        'URL': 'http://39.105.40.148:9200',  # es服务端地址
        'INDEX_NAME': 'haystack',         # es索引仓库
    },
}

# 当ORM操作数据库改变时，自动更新es的索引，否则es的索引会找不到新增的数据
HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'


# 支付宝相关配置[改成公司的, 改成真实网关]
ALIPAY = {
    # 'gateway': 'https://openapi.alipay.com/gateway.do',   # 真实网关地址[]
    'gateway': 'https://openapi.alipaydev.com/gateway.do',  # 沙箱网关地址
    'appid': '2016091600523592',  # 支付应用ID
    'sign_type': 'RSA2',  # 签证的加密算法
    'debug': True,  # 沙箱模式下必须设置为True
    'verbose': True,  # 是否在调试模式下输出调试数据
    'timeout': 15,  # 请求超时时间，单位：秒，默认15s
    "app_private_key_path": BASE_DIR / "apps/payments/keys/app_private_key.pem",  # 应用私钥路径
    "alipay_public_key_path": BASE_DIR / "apps/payments/keys/alipay_public_key.pem",  # 支付宝公钥路径
    "return_url": "http://www.fuguang.jiangw.cc/alipay",  # 同步回调结果通知js地址
    "notify_url": "http://api.fuguang.jiangw.cc/payments/alipay/notify",  # 异步回调结果通知地址
}


# 保利威视频加密服务[改成公司的]
POLYV = {
    "userId": "705f4d7a89",
    "secretkey": "rnYLITxlpW",
    "writetoken": "7322d5b7-1539-4524-b65c-bd6d5d433f0b",
    "readtoken": "4dbba420-98e3-449a-bdd8-0b56e154a584",
    "tokenUrl": "https://hls.videocc.net/service/v1/token",
}
```



###### 从服务端项目中收集静态文件

当Django运行在生产环境中，我们会关闭debug调试，那么项目将不再提供静态文件的支持，需要将静态文件交给静态文件的nginx服务器来提供访问支持。

我们先收集所有静态文件。项目中的静态文件除了我们使用的上传文件之外，django本身还有自己的静态文件，如rest_framework、simpleUI、admin、ckeditor等。我们需要收集这些静态文件，集中一起放到静态文件服务器中。

**我们要将收集的静态文件放到项目的static目录中，所以先创建目录static。**

Django提供了收集静态文件的方法。先在配置文件中配置收集之后存放的目录

settings/dev.py 和settings/pro.py，追加以下代码：

```python
# 收集到的静态文件存储目录
STATIC_ROOT = BASE_DIR.parent / 'static'
```

因为目前在本地我们依然使用了dev.py配置文件通过manage.py运行项目，所以我们在本地项目收集静态文件就需要把配置信息同步到dev.py

在本地终端下fuguangapi根目录下执行收集命令

```shell
cd fuguangapi
python manage.py collectstatic
```

执行收集命令以后，效果如下：

![image-20210906113140902](Linux项目部署.assets/image-20210906113140902.png)



###### django的程序通常使用uwsgi服务器或者 gunicorn/Uvincorn来运行

uwsgi/gunicorn服务器 同步模式运行项目

uvicorn服务器 异步模式运行项目

先修改fuguangapi/wsgi.py和fuguangapi/asgi.py，加载线上的pro配置文件：

```python
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'fuguangapi.settings.pro')
```

我们目前针对uwsgi服务器的了解还少，所以我们可以先在本地环境中先安装uwsgi模块进行使用，然后成功使用uwsgi启动项目，后面再一并上传到服务器上面重新安装并运行。

```python
conda install -c conda-forge uwsgi
uwsgi --version    # 查看 uwsgi 版本
```



在项目根目录下创建uwsgi配置文件uwsgi.ini

注意：在manage.py的同级目录下创建uwsgi.ini配置文件。

```ini
[uwsgi]
# 使用nginx连接时使用，Django程序所在服务器地址[反向代理】
# socket=0.0.0.0:8000
# 直接做web服务器使用，Django程序所在服务器地址
http=0.0.0.0:8000
# 项目根目录[一定要使用绝对路径，否则无法访问]
chdir=/home/moluo/Desktop/fuguang/fuguangapi
# 项目中主应用下的wsgi.py文件的路径[相对于项目根目录]
wsgi-file=fuguangapi/wsgi.py
# 进程数[CPU*2]
processes=4
# 线程数[CPU线程]
threads=4
# uwsgi服务器的角色[是否开启主进程管理模式]
master=True
# 存放进程编号的文件[相对于项目根目录]
pidfile=uwsgi.pid
# 日志文件，因为uwsgi可以脱离终端在后台运行，日志看不见。我们以前的runserver是依赖终端的[相对于项目根目录]
daemonize=uwsgi.log
# 指定依赖的虚拟环境[聚堆路径，写到虚拟环境目录名]
virtualenv=/home/moluo/anaconda3/envs/fuguang
```

启动uwsgi服务器，必须在uwsgi.ini配置文件所在的目录中运行下面的命令

注意，在本地环境中必须先把runserver关闭

```shell
uwsgi --ini uwsgi.ini
```

停止uwsgi服务器，还可以通过使用kill命令停止

```
uwsgi --stop uwsgi.pid
```

访问项目 http://127.0.0.1:8000/nav/header/

![image-20211021153910264](Linux项目部署.assets/image-20211021153910264.png)



###### 把django项目代码上传部署到服务器并使用uwsgi服务器运行

在本地导出当前虚拟环境中的模块包列表到requirements.txt文件中

```bash
cd ~/Desktop/fuguang/fuguangapi
pip freeze > requirements.txt

# 注意：在requirements.txt中有conda安装的模块以 file://开头的，务必删除，否则将来报错。
# PyMySQL @ file:///home/conda/feedstock_root/build_artifacts/pymysql_1610208984992/work
# uWSGI @ file:///home/conda/feedstock_root/build_artifacts/uwsgi_1640027751896/work

# 因为drf-haystack依赖的django-haystack必须是3.1以下版本，所以调整代码
django-haystack==3.1.1 改成 django-haystack==3.0
```

本地把服务端项目打包成fuguangapi.tar.gz，并在阿里云服务器中解压。

```bash
# 本地终端，执行打包命令并上传压缩包
cd ~/Desktop/fuguang/
tar -zcf fuguangapi.tar.gz fuguangapi
scp ~/Desktop/fuguang/fuguangapi.tar.gz root@39.105.40.148:/home/fuguang/

# 阿里云服务器终端，删除原来就api服务端代码
cd /home/fuguang
rm -rf fuguangapi
# 把上传到服务端的压缩包进行解压
tar -zxf fuguangapi.tar.gz
```



##### 编写Dockerfile定制服务端项目的镜像

```bash
vim /home/fuguang/Dockerfile
```

/home/fuguang/Dockerfile，代码：

```dockerfile
FROM ubuntu:20.04
LABEL maintainer="python36.Edu"

ENV PYTHONUNBUFFERED 1

COPY ./sources.list /etc/apt/sources.list
COPY ./fuguangapi /fuguangapi
COPY ./fuguangapi/requirements.txt /requirements.txt

RUN apt-get update \
    && apt-get -y install wget \
    && wget https://repo.anaconda.com/miniconda/Miniconda3-py38_4.10.3-Linux-x86_64.sh \
    && mkdir /root/.conda \
    && bash Miniconda3-py38_4.10.3-Linux-x86_64.sh -b \
    && rm -f Miniconda3-py38_4.10.3-Linux-x86_64.sh

ENV PATH /root/miniconda3/bin:$PATH

RUN conda install pymysql -c conda-forge \
    && conda install uwsgi -c conda-forge \
    && pip install -r /requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple \
    && pip install django-haystack==3.1.1 -i https://pypi.tuna.tsinghua.edu.cn/simple \
    && chmod -R 755 /fuguangapi

WORKDIR /fuguangapi

EXPOSE 8000

VOLUME /fuguangapi
```

注释版：

```python
# 设置当前镜像的基础镜像,FROM指令只能使用一次，而且在Dockerfile顶部第一个使用
FROM ubuntu:20.04
# 设置描述信息
LABEL maintainer="python36.Edu"
# 设置环境变量，PYTHONUNBUFFERED设置python解析器在终端下stdout错误信息，一次性输出，要不在暂存区积攒
ENV PYTHONUNBUFFERED 1

# 复制文件    容器外部:容器内部
COPY ./sources.list /etc/apt/sources.list
COPY ./fuguangapi /fuguangapi
COPY ./fuguangapi/requirements.txt /requirements.txt

# 运行系统命令
RUN apt-get update \  # 更新源
    && apt-get -y install wget \  # 安装wget下载工具
    && wget https://repo.anaconda.com/miniconda/Miniconda3-py38_4.10.3-Linux-x86_64.sh \
    && mkdir /root/.conda \
    && bash Miniconda3-py38_4.10.3-Linux-x86_64.sh -b \
    && rm -f Miniconda3-py38_4.10.3-Linux-x86_64.sh  # 设置环境变量

ENV PATH /root/miniconda3/bin:$PATH  # 设置环境变量

# 给conda全局base虚拟环境安装pymysql和uwsgi
RUN conda install pymysql -c conda-forge \
    && conda install uwsgi -c conda-forge \
    # 递归安装requirements.txt中的所有依赖模块
    && pip install -r /requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple \
    # 因为drf-haystack安装以后，会自动帮django-haystack降版本到3.1.0，但是django-haystack没有支持es7版本的类，所以在requirements.txt文件执行完成以后，重新把 django-haystack更新到3.1.1版本，实现兼容
    && pip install django-haystack==3.1.1 -i https://pypi.tuna.tsinghua.edu.cn/simple \
    # 设置目录权限
    && chmod -R 755 /fuguangapi

# 设置进入容器的默认工作目录
WORKDIR /fuguangapi
# 暴露容器的端口
EXPOSE 8000
# 设置/fuguangapi目录为逻辑卷，方便创建容器时，可以把当前目录的内容映射到容器外面进行持久化存储
VOLUME /fuguangapi
```



```bash
vim /home/fuguang/sources.list
```

/home/fuguang/sources.list，代码：

```bash
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

通过docker-compose启动fuguangapi镜像，docker-compose.yml，代码：

```yaml
version: '3.7'
services:
  web1:
    image: nginx:1.21.4
    restart: always
    ports:
      - "80:80"
    container_name: "web1"
    volumes:
      - ./fuguangweb:/home/fuguangweb
      - ./conf/nginx/web.conf:/etc/nginx/conf.d/nginx.conf
    networks:
      - default

  mysql_master:
    image: mysql:8.0.26
    restart: always
    container_name: mysql_master
    networks:
      - default
    environment:
      - "MYSQL_ROOT_PASSWORD=root2021"
      - "MYSQL_USER=fuguang"
      - "MYSQL_PASSWORD=fuguang"
      - "MYSQL_DATABASE=fuguang"
      - "TZ=Asia/Shanghai"
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
    ports:
      - 3306:3306
    volumes:
      - ./data/mysql:/var/lib/mysql
      - ./logs/mysql:/logs
      - ./initdb:/docker-entrypoint-initdb.d/

  redis_master:
    image: redis:6.2.6-alpine3.15
    restart: always
    networks:
      - default
    ports:
      - "6379:6379"
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    container_name: redis_master
    volumes:
      - ./conf/redis/master.conf:/usr/local/etc/redis/redis.conf
      - ./data/redis/master:/data

  elasticsearch:
    image: elasticsearch:7.16.1
    restart: always
    environment:
      - "discovery.type=single-node"
      - "bootstrap.memory_lock=true"
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - "xpack.security.enabled=false"
      - "http.cors.enabled=true"
      - "http.cors.allow-origin=*"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./data/elastic:/usr/share/elasticsearch/data
      - ./logs/elastic:/var/log/elasticsearch
      - ./plugins:/usr/share/elasticsearch/plugins
    ports:
      - 9200:9200
    networks:
      - default

  api1:
    build: .
    image: fuguangapi:1.0.0
    container_name: api1
    restart: always
    ports:
      - 8000:8000
    networks:
      - default
    volumes:
      - ./fuguangapi:/fuguangapi
    command: >
      sh -c "uwsgi --http :8000 --workers 4 --master --enable-threads --module fuguangapi.wsgi"
    environment:
      - "DJANGO_SETTINGS_MODULE=fuguangapi.settings.pro"
    depends_on:
      - mysql_master
      - redis_master
      - elasticsearch

networks:
  default:
```

启动服务

```bash
docker-compose -f docker-compose.yml down

docker-compose -f docker-compose.yml up -d
# 等待1分钟后给elasticsearch构建索引
docker exec -it api1 python manage.py rebuild_index
```

可以看到api已经启动了，但是我们通过浏览器`http://api.fuguang.jiangw.cc:8000`访问发现，访问不了，我们尝试访问到后台运营站点admin，`http://api.fuguang.jiangw.cc:8000/admin/`。

![image-20210907091715845](Linux项目部署.assets/image-20210907091715845.png)



### 使用nginx对uwsgi进行反向代理

接下来，我们就需要把api1服务端容器里面的uwsgi的运行模式从http改成socket模式。

docker-compose.yml，修改如下。

```yaml
version: '3.7'
services:
  web1:
    image: nginx:1.21.4
    restart: always
    ports:
      - "80:80"
    container_name: "web1"
    volumes:
      - ./fuguangweb:/home/fuguangweb
      - ./conf/nginx/web.conf:/etc/nginx/conf.d/nginx.conf
    networks:
      - default

  mysql_master:
    image: mysql:8.0.26
    restart: always
    container_name: mysql_master
    networks:
      - default
    environment:
      - "MYSQL_ROOT_PASSWORD=root2021"
      - "MYSQL_USER=fuguang"
      - "MYSQL_PASSWORD=fuguang"
      - "MYSQL_DATABASE=fuguang"
      - "TZ=Asia/Shanghai"
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
    ports:
      - 3306:3306
    volumes:
      - ./data/mysql:/var/lib/mysql
      - ./logs/mysql:/logs
      - ./initdb:/docker-entrypoint-initdb.d/

  redis_master:
    image: redis:6.2.6-alpine3.15
    restart: always
    networks:
      - default
    ports:
      - "6379:6379"
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    container_name: redis_master
    volumes:
      - ./conf/redis/master.conf:/usr/local/etc/redis/redis.conf
      - ./data/redis/master:/data

  elasticsearch:
    image: elasticsearch:7.16.1
    restart: always
    environment:
      - "discovery.type=single-node"
      - "bootstrap.memory_lock=true"
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - "xpack.security.enabled=false"
      - "http.cors.enabled=true"
      - "http.cors.allow-origin=*"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./data/elastic:/usr/share/elasticsearch/data
      - ./logs/elastic:/var/log/elasticsearch
      - ./plugins:/usr/share/elasticsearch/plugins
    ports:
      - 9200:9200
    networks:
      - default

  api1:
    build: .
    image: fuguangapi:1.0.0
    container_name: api1
    restart: always
    ports:
      - 8000:8000
    networks:
      - default
    volumes:
      - ./fuguangapi:/fuguangapi
    command: >
      sh -c "uwsgi --socket :8000 --workers 4 --master --enable-threads --module fuguangapi.wsgi"
   environment:
      - "DJANGO_SETTINGS_MODULE=fuguangapi.settings.pro"
    depends_on:
      - mysql
      - redis
      - elasticsearch

networks:
  default:
```

修改nginx配置文件

```bash
vim conf/nginx/web.conf
```

conf/nginx/web.conf，如下：

```bash
upstream fuguang {
    server 39.105.40.148:8000;
}

server {
    listen  80;
    server_name api.fuguang.jiangw.cc;

    location / {
        include uwsgi_params;
        uwsgi_pass fuguang;
    }
}

server {
    listen       80;
    server_name  www.fuguang.jiangw.cc;
    # client_max_body_size    1000M;
    # root         /opt/dist;

    # client_max_body_size 20M;

    # Load configuration files for the default server block.

    location / {
        root  /home/fuguangweb;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
```

重启docker-compose

```bash
docker-compose -f docker-compose.yml down
docker-compose -f docker-compose.yml up -d
# 接下来，我们直接在浏览器中，通过如下地址访问：
https://api.fuguang.jiangw.cc/courses/search/
http://www.fuguang.jiangw.cc/
```



### 针对后端服务器的运营站点提供静态文件的访问支持

直接让前端的nginx容器同时提供服务端静态文件的访问支持。

修改docker-compose.yml，把服务端的静态文件目录映射到nginx容器中。代码：

```python
version: '3.7'
services:
  web1:
    image: nginx:1.21.4
    restart: always
    ports:
      - "80:80"
    container_name: "web1"
    volumes:
      - ./fuguangapi/static:/home/fuguangapi/static
      - ./fuguangweb:/home/fuguangweb
      - ./conf/nginx/web.conf:/etc/nginx/conf.d/nginx.conf
    networks:
      - default

  mysql_master:
    image: mysql:8.0.26
    restart: always
    container_name: mysql_master
    networks:
      - default
    environment:
      - "MYSQL_ROOT_PASSWORD=root2021"
      - "MYSQL_USER=fuguang"
      - "MYSQL_PASSWORD=fuguang"
      - "MYSQL_DATABASE=fuguang"
      - "TZ=Asia/Shanghai"
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
    ports:
      - 3306:3306
    volumes:
      - ./data/mysql:/var/lib/mysql
      - ./logs/mysql:/logs
      - ./initdb:/docker-entrypoint-initdb.d/

  redis_master:
    image: redis:6.2.6-alpine3.15
    restart: always
    networks:
      - default
    ports:
      - "6379:6379"
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    container_name: redis_master
    volumes:
      - ./conf/redis/master.conf:/usr/local/etc/redis/redis.conf
      - ./data/redis/master:/data

  elasticsearch:
    image: elasticsearch:7.16.1
    restart: always
    environment:
      - "discovery.type=single-node"
      - "bootstrap.memory_lock=true"
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - "xpack.security.enabled=false"
      - "http.cors.enabled=true"
      - "http.cors.allow-origin=*"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./data/elastic:/usr/share/elasticsearch/data
      - ./logs/elastic:/var/log/elasticsearch
      - ./plugins:/usr/share/elasticsearch/plugins
    ports:
      - 9200:9200
    networks:
      - default

  api1:
    build: .
    image: fuguangapi:1.0.0
    container_name: api1
    restart: always
    ports:
      - 8000:8000
    networks:
      - default
    volumes:
      - ./fuguangapi:/fuguangapi
    command: >
      sh -c "uwsgi --socket :8000 --workers 4 --master --enable-threads --module fuguangapi.wsgi"
   environment:
      - "DJANGO_SETTINGS_MODULE=fuguangapi.settings.pro"
    depends_on:
      - mysql
      - redis
      - elasticsearch

networks:
  default:
```



修改nginx配置文件web.conf，服务端对应的server配置项中增加一个location配置信息。conf/nginx/web.conf，如下：

```bash
upstream fuguang {
    server 39.105.40.148:8000;
}

server {
    listen  80;
    server_name api.fuguang.jiangw.cc;

    location / {
        include uwsgi_params;
        uwsgi_pass fuguang;
    }

    location /static {
        root /home/fuguangapi;
    }

}

server {
    listen       80;
    server_name  www.fuguang.jiangw.cc;
    # client_max_body_size    1000M;
    # root         /opt/dist;

    # client_max_body_size 20M;

    # Load configuration files for the default server block.

    location / {
        root  /home/fuguangweb;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
```

完成上面操作以后，我们可以看到当前api服务端的静态文件就提供了http正常浏览了。



### 启动celery

关于celery的启动，因为需要依赖于django项目的。所以celery的启动自然也就和当前api容器一并运行。

修改celery的启动配置文件

/home/fuguang/fuguangapi/scripts/fuguang_celery_worker.ini，代码：

````ini
[program:fuguang_celery_worker]
# 启动命令 conda env list
command=/root/miniconda3/bin/celery -A fuguangapi worker -l info -n worker1
# 项目根目录的绝对路径，通过pwd查看
directory=/fuguangapi
# 项目虚拟环境
enviroment=PATH="/root/miniconda3/bin"
# 输出日志绝对路径
stdout_logfile=/fuguangapi/logs/celery.worker.info.log
# 错误日志绝对路径
stderr_logfile=/fuguangapi/logs/celery.worker.error.log
# 自动启动，开机自启
autostart=true
# 重启
autorestart=true

# 进程启动后跑了几秒钟，才被认定为成功启动，默认1
startsecs=10
# 进程结束后60秒才被认定结束
stopwatisecs=60

# 优先级
priority=999
````

/home/fuguang/fuguangapi/scripts/fuguang_celery_beat.ini，代码：

```ini
[program:fuguang_celery_beat]
# 启动命令 conda env list
command=/root/miniconda3/bin/celery -A fuguangapi  beat -l info
# 项目根目录的绝对路径，通过pwd查看
directory=/fuguangapi
# 项目虚拟环境
enviroment=PATH="/root/miniconda3/bin"
# 输出日志绝对路径
stdout_logfile=/fuguangapi/logs/celery.beat.info.log
# 错误日志绝对路径
stderr_logfile=/fuguangapi/logs/celery.beat.error.log
# 自动启动，开机自启
autostart=true
# 重启
autorestart=true

# 进程启动后跑了几秒钟，才被认定为成功启动，默认1
startsecs=10

# 进程结束后60秒才被认定结束
stopwatisecs=60

# 优先级
priority=998
```

/home/fuguang/fuguangapi/scripts/fuguang_celery_flower.ini，代码：

```ini
[program:fuguang_celery_flower]
# 启动命令 conda env list
command=/root/miniconda3/bin/celery -A fuguangapi flower --port=5555
# 项目根目录的绝对路径，通过pwd查看
directory=/fuguangapi
# 项目虚拟环境
enviroment=PATH="/root/miniconda3/bin"
# 输出日志绝对路径
stdout_logfile=/fuguangapi/logs/celery.flower.info.log
# 错误日志绝对路径
stderr_logfile=/fuguangapi/logs/celery.flower.error.log
# 自动启动，开机自启
autostart=true
# 重启
autorestart=true

# 进程启动后跑了几秒钟，才被认定为成功启动，默认1
startsecs=10

# 进程结束后60秒才被认定结束
stopwatisecs=60

# 优先级
priority=990
```

/home/fuguang/fuguangapi/scripts/supervisor.service，代码：

```shell
[Unit]
Description=supervisor
After=network.target

[Service]
Type=forking
ExecStart=/root/miniconda3/bin/supervisord -n -c /fuguangapi/scripts/supervisord.conf
ExecStop=/root/miniconda3/bin/supervisorctl $OPTIONS shutdown
ExecReload=/root/miniconda3/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

docker-compose.yml，代码：

```yaml
version: '3.7'
services:
  web1:
    image: nginx:1.21.4
    restart: always
    ports:
      - "80:80"
    container_name: "web1"
    volumes:
      - ./fuguangapi/static:/home/fuguangapi/static
      - ./fuguangweb:/home/fuguangweb
      - ./conf/nginx/web.conf:/etc/nginx/conf.d/nginx.conf
    networks:
      - default

  mysql_master:
    image: mysql:8.0.26
    restart: always
    container_name: mysql_master
    networks:
      - default
    environment:
      - "MYSQL_ROOT_PASSWORD=root2021"
      - "MYSQL_USER=fuguang"
      - "MYSQL_PASSWORD=fuguang"
      - "MYSQL_DATABASE=fuguang"
      - "TZ=Asia/Shanghai"
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
    ports:
      - 3306:3306
    volumes:
      - ./data/mysql:/var/lib/mysql
      - ./logs/mysql:/logs
      - ./initdb:/docker-entrypoint-initdb.d/

  redis_master:
    image: redis:6.2.6-alpine3.15
    restart: always
    networks:
      - default
    ports:
      - "6379:6379"
    command:
      ["redis-server", "/usr/local/etc/redis/redis.conf"]
    container_name: redis_master
    volumes:
      - ./conf/redis/master.conf:/usr/local/etc/redis/redis.conf
      - ./data/redis/master:/data

  elasticsearch:
    image: elasticsearch:7.16.1
    restart: always
    environment:
      - "discovery.type=single-node"
      - "bootstrap.memory_lock=true"
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - "xpack.security.enabled=false"
      - "http.cors.enabled=true"
      - "http.cors.allow-origin=*"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./data/elastic:/usr/share/elasticsearch/data
      - ./logs/elastic:/var/log/elasticsearch
      - ./plugins:/usr/share/elasticsearch/plugins
    ports:
      - 9200:9200
    networks:
      - default

  api1:
    build: .
    image: fuguangapi:1.0.0
    container_name: api1
    restart: always
    ports:
      - 8000:8000
    networks:
      - default
    volumes:
      - ./fuguangapi:/fuguangapi
    command: >
      sh -c "supervisord -c scripts/supervisord.conf && uwsgi --socket :8000 --workers 4 --master --enable-threads --module fuguangapi.wsgi"
   environment:
      - "DJANGO_SETTINGS_MODULE=fuguangapi.settings.pro"
      - "C_FORCE_ROOT=1"
    depends_on:
      - mysql
      - redis
      - elasticsearch

networks:
  default:
```

记得开启服务端的安全组，开发5555端口。

![image-20211022181209537](Linux项目部署.assets/image-20211022181209537.png)

启动服务

```bash
docker-compose -f docker-compose.yml down
docker-compose -f docker-compose.yml up -d
```

