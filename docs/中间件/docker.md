# 虚拟化技术

```
把真实物理机子中剩余的资源重新整合，基于虚拟机软件创建出来一台新的虚拟的计算机提供给开发者使用。
优点：
   虚拟化使用软件/硬件的方法重新定义划分计算机资源，可以实现计算机资源的动态分配、灵活调度、跨域共享，提高IT资源利用率，降低成本，加快部署，极大增强系统整体安全性和可靠性。使IT资源能够真正成为社会基础设施，服务于各行各业中灵活多变的应用需求。

虚拟化技术有5种不同的实现方案：
1. 硬件虚拟化
   需要购买虚拟化设备
2. 分区虚拟化
   一台电脑下可以实际安装多个操作系统。开机的时候，就固定分配好了内存。
3. 虚拟机技术[应用虚拟化]
   通过在操作系统中安装软件来实现，例如：VMware，virtualbox,在vm中创建虚拟机，搭建操作系统，在虚拟机运行的时候，由vm动态向真实电脑申请分配硬件资源[cpu,内存，显卡网络等等]。
4. 准虚拟机技术
   是上面第2和第3中的混合产物，不需要安装vm也不需要分区，而是由操作系统本身提供出来了一个虚拟层来实现的。
   例如；xven或者window10的HyperV都是这种实现。


因为上面不管哪一种虚拟化技术，都会出现一个问题就是为了让虚拟出来的操作系统能正常运作起来都需要实实在在向真实物理机申请固定的对应计算机资源。而且为了能让虚拟出来的操作系统能正常运作，物理机要分配各种的硬件资源，这样很大程度上存在资源的消耗，降低物理机的性能。而这个过程用户完全有可能仅仅只是了为运行某几个软件而已。所以为了更好的提升用户的体验，提升系统的性能，减低不必要的计算机资源消耗，所以出现了一种新的虚拟化技术。

容器化技术！！！
比较流行的容器化技术有：docker和podman
目前在外界使用过程中，比较常用的还是docker

docker提供给开发者使用的方式提供了3种：
1. 终端命令[通过终端命令逐步操作docker]
2. Dockerfile[通过脚本对docker进行封装和操作单个镜像]
3. docker-compose [通过脚本对docker的多个镜像/容器进行组合编排的技术, 是python实现的一种技术]
```



更新ubuntu的apt源索引

```shell
sudo apt-get update
```



安装包允许apt通过HTTPS使用仓库

```shell
sudo dpkg --configure -a
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
```



添加Docker官方GPG key【这个是国外服务器地址，所以网路不好的时候，会失败！在网路好的情况下，多执行几次就没问题了】

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
sudo apt-get install docker-ce
```

通过下载一个叫`hello-world`的镜像并运行起来，以此来检查Docker CE是否安装正确并能正常使用。

```shell
sudo docker run hello-world
```

出现了`helo from Docker`则表示上面的安装成功！

![1563502563172](docker.assets/1563502563172.png)



![1563503374720](docker.assets/1563503374720.png)



我们获取镜像文件，可以直接去官方网站上获取: https://hub.docker.com/



# Docker/podman 命令

docker在ubuntu使用过程中，需要左边加上sudo，而podman不需要。同时下面所有的命令在docker和podman里面是通用的。

所以这里全部写成docker了，在使用podman操作以下以下命令时，直接把docker换成podman即可。

### 通用命令

查看docker 当前版本

```shell
sudo docker version
# podman version
```

```bash
# 启动docker
sudo service docker start
# sudo service podman start

# 停止docker
sudo service docker stop
# sudo service podman stop

# 重启docker
sudo service docker restart
# sudo service podman restart
```



### 镜像操作[image]

#### 列出镜像

列出本地所有镜像。

```shell
sudo docker image ls --all
# 简写
# sudo docker image ls
# 简写
# sudo docker images

# podman
# podman images
```

终端显示效果：

| REPOSITORY   | TAG         | IMAGE ID   | CREATED  | SIZE     |
| ------------ | ----------- | ---------- | -------- | -------- |
| 镜像仓库地址 | 版本号/标签 | 镜像标记ID | 构建时间 | 文件大小 |

注意：

+ repository如果以docker.io/**library**开头的表示是docker官方提供的镜像，否则就是第三方封装的。
+ 某些公司里面针对第三方封装的镜像，一般是有使用要求的，不能使用低于100tart



#### 拉取镜像

docker/podman支持通过网络拉去镜像源站的所有镜像。默认使用的工具就是git工具。

拉取镜像时，如果不指定版本号，默认拉取最新版本的镜像

```shell
sudo docker image pull <镜像名称:版本号>
# 简写
# sudo docker pull <镜像名称:版本号>

# podman
# podman pull <镜像名称:版本号>
```

#### 删除镜像

删除镜像时必须注意是否有容器在运行当前镜像文件，如果有则需要先删除基于该镜像的所有容器，才能删除镜像。

```shell
sudo docker image rm <镜像名称/镜像ID>
# 简写
# sudo docker rmi <镜像名称/镜像ID>

# podman
# podman rmi <镜像名称/镜像ID>
```

删除的镜像如果被容器提前使用了，则错误如下：

![1563504236734](docker.assets/1563504236734.png)



#### 镜像打包

把docker中的镜像打包成文件，用于分享发送给他人，或备份

```shell
sudo docker save -o <文件名.tar.gz>  <镜像名:版本号>

# podman
# podman save -o <文件名.tar.gz>  <镜像名:版本号>
```



#### 镜像加载

把镜像文件加载到docker中。

```shell
sudo docker load -i <文件名.tar.gz>

# podman
# podman load -i  <文件名.tar.gz>
```



#### 上传镜像

使用之前，必须先到阿里云/dockerhub官方注册账号并创建对应的仓库。

```bash
sudo docker login -u <账号名>
# podman login -u <账号名>
# 1. 有专业的运维团队都会搭建私有的dockerhub，连接这种dockerhub时就要设置服务端地址。
# 2, 也有些公司使用了第三方云服务商（阿里云、gitlab、腾讯云、亚马逊）的devops技术栈时，往往会采用服务商提供的dockerhub。

docker login -u cheergoivan registry.cn-hangzhou.aliyuncs.com

sudo docker push <镜像名称/镜像ID>:<版本号>
# podman push <镜像名称/镜像ID>:<版本号>
```



### 容器操作[container]

#### 创建容器

必须先有镜像，才能运行创建容器，需要指定使用的镜像名，并且设置创建容器以后，执行对应的第一条命令

注意：容器运行以后，如果没有至少一个阻塞程序，或者一个阻塞的命令在运行，则docker为了节约资源，会关闭容器。

```shell
sudo docker run <选项参数> <镜像名称:镜像版本> <容器启动后的第一个命令>

# podman run <选项参数>  <镜像名称:镜像版本> <容器启动后的第一个命令>
```

例如：使用"hello-world"镜像，创建一个容器，但没有进行任何操作（注意：如果运行容器时，本地没有对应的镜像或对应镜像的版本，则docker/podman会自动往线上的源服务器中搜索是否有对应的镜像并自动下载的）

```shell
sudo docker run hello-world
# podman run hello-world
```

例如：使用ubuntu镜像，创建一个名为ubuntu1

```bash
# sudo docker pull ubuntu:20.04
sudo docker run -it --name=ubuntu1 ubuntu:20.04 bash

# podman pull ubuntu:20.04
# podman run -ti --name=ubuntu1 ubuntu:20.04 bash
```

注意：必须启动的时候，让容器运行bash解析器，才能在接下来的操作让容器不会立刻关闭，而且也能够让我们可以输入linux终端命令， 如果我们一般创建一个容器，选项都是： -itd



##### docker run的选项

-t    表示容器启动后会进入其命令行终端

-i     表示以“交互模式”运行容器

--name  表示设置容器的名称

-d   表示以守护进程的模式来运行容器

-v    目录影射，相当于把容器外部的物理机的目录与容器内部的目录实现共享，改了里面相当于改了外面

-p    端口影射，把物理机的一个端口和容器内部的端口进行绑定。访问了物理机的端口相当于访问了容器的对应端口

-e    设置容器内部的环境变量，在部分容器中需要设置环境变量来完成软件的配置。

--restart=always    设置容器开机自启，podman中的容器管理是没有开机自启这个选项。

--network=host     设置网络模式，与-p冲突，一般设置-p以后不要设置--network

例如，使用ubuntu镜像，创建一个名为ubuntu2，并且在后台运行的容器像

```bash
sudo docker run -itd --name=ubuntu2 ubuntu:<版本>
# podman run -itd --name=ubuntu2 ubuntu:<版本>
```

-d   创建一个守护进程模式的容器在后台运行(这样创建容器后不会自动登录进入容器，如果只加-i -t 两个参数，创建后不会自动进去容器内部，将来可以使用docker exec -it <容器名> 命令即可进入容器)



#### 列出容器

```shell
sudo docker container ls                      # 所有正在启动运行的容器
# 简写 sudo docker ps
# podman ps
sudo docker container ls --all                # 所有容器[不管是否在启动运行中]
# 简写 sudo docker ps --all
# podman ps --all
```

显示效果：

| 选项         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| CONTAINER ID | 容器唯一标记符                                               |
| IMAGE        | 当前容器依赖的镜像名以及镜像版本                             |
| COMMAND      | 容器开机以后的第一个执行命令                                 |
| CREATED      | 容器构建时间                                                 |
| STATUS       | 运行状态<br>Up表示运行中，Exit表示退出运行了，Created表示创建过程中出现异常退出 |
| PORTS        | 容器外部和容器内部的端口映射                                 |
| NAMES        | 容器的唯一名称<br>创建容器时如果不指定，则docker会自动生成一个随机的唯一名称。 |



#### 启动容器

可以同时启动多个容器，容器之间使用空格隔开

```shell
# 启动一个容器[被开启的容器默认会以守护式容器在后台持续运行]
sudo docker container start <容器名称/容器ID>
# 简写 sudo docker start  <容器名称/容器ID>
# podman start  <容器名称/容器ID>

# 启动多个容器
sudo docker container start <容器名称/容器ID>  <容器名称/容器ID> <容器名称/容器ID>
# 简写 sudo docker start <容器名称/容器ID>  <容器名称/容器ID> <容器名称/容器ID>
# podman start <容器名称/容器ID>  <容器名称/容器ID> <容器名称/容器ID>
```



#### 停止容器

```shell
sudo docker container stop <容器名称/容器ID>
# 简写 sudo docker stop  <容器名称/容器ID>
# podman stop  <容器名称/容器ID>

# 停止多个容器
sudo docker container stop <容器名称/容器ID>  <容器名称/容器ID>
# 简写 sudo docker stop <容器名称/容器ID>  <容器名称/容器ID>
# podman stop <容器名称/容器ID>  <容器名称/容器ID>
```



#### 杀死容器

该命令在容器无法停止的时使用，注意不能滥用，这种操作有可能被导致容器里面运行的文件丢失！！！

```shell
sudo docker container kill <容器名称/容器ID>
# 简写 sudo docker kill <容器名称/容器ID>
# podman kill <容器名称/容器ID>

# 杀死多个容器
sudo docker container kill <容器名称/容器ID>  <容器名称/容器ID>
# 简写 sudo docker kill <容器名称/容器ID>  <容器名称/容器ID>
# podman kill <容器名称/容器ID>  <容器名称/容器ID>
```



#### 进入容器

要进入容器，必须当前容器是启动状态的，这个命令不需要加上-d

```shell
sudo docker container exec -it <容器名称/容器ID>  <第一个命令>
# sudo docker exec -it <容器名称/容器ID>  <第一个命令>
# podman exec -it <容器名称/容器ID>  <第一个命令>
```

第一个命令一般都是bash，也可以是其他允许开发者输入信息的其他软件命令

```
podman exec -it ubuntu1 bash
```



#### 删除容器

```shell
sudo docker  container rm <容器名称/容器ID>
# 简写 sudo docker rm <容器名称/容器ID>
# podman rm <容器名称/容器ID>

# 删除多个容器
sudo docker  container rm <容器名称/容器ID>  <容器名称/容器ID>
# 简写 sudo docker rm <容器名称/容器ID>  <容器名称/容器ID>
# podman rm <容器名称/容器ID>  <容器名称/容器ID>
```



#### 复制文件

即便容器没有启动，也可以复制。

```bash
# 命令基本格式：
sudo docker container cp <源文件地址> <保存文件地址>


# 从物理机中复制一个文件到指定容器的内部指定路径中
sudo docker container cp <物理机路径> <容器名称/容器ID>:<容器路径>
# 简写 sudo docker cp <物理机路径> <容器名称/容器ID>:<容器路径>
# podman cp  <物理机路径> <容器名称/容器ID>:<容器路径>


# 指定容器的内部指定路径中复制一个文件到物理机指定路径中保存
sudo docker container cp <容器名称/容器ID>:<容器路径> <物理机路径>
# 简写 sudo docker cp <容器名称/容器ID>:<容器路径> <物理机路径>
# podman cp <容器名称/容器ID>:<容器路径> <物理机路径>
```



#### 容器保存

把容器保存成镜像。

```shell
sudo docker container commit <容器名称/容器ID>  <新镜像名:镜像自定义版本>
# 简写 sudo docker commit <容器名称/容器ID>  <新镜像名:镜像自定义版本>
# podman commit <容器名称/容器ID>  <新镜像名:镜像自定义版本>
```

本地封装的镜像默认是以localhost开头的。

容器保存成镜像，就可以基于镜像操作，导出成压缩包，或者基于dockerhub上传镜像仓库分享给其他人。

