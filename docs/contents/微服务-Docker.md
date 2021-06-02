## **容器化**[**本质**](https://segmentfault.com/a/1190000005717596)

如果你遇到了以下问题：

**变化快、扛不住**

那就[拆分成微服务](https://mp.weixin.qq.com/s/PEIoBRhRoHvcfxEP_umZ8w)

**什么情况下应该考虑做一些改变？**

传统业务突然被互联网业务冲击了，应用老是变，三天两头要更新，而且流量增大了，原来支付系统是取钱刷卡的，现在要互联网支付了，流量扩大了N倍。

**没办法，一个字：拆！**

拆开了，每个子模块独自变化，少相互影响。

拆开了，原来一个进程扛流量，现在多个进程一起扛。

所以称为**微服务**。

 

微服务场景下，进程多，更新快，于是出现100个进程，每天一个镜像。

容器乐了，每个容器镜像小，没啥问题，虚拟机哭了，因为虚拟机每个镜像太大了。

所以微服务场景下，可以开始考虑用**容器**了。**容器的本质是基于镜像的跨环境迁移。**

镜像是容器的根本性发明，是封装和运行的标准，其它什么namespace，cgroups，早就有了。这是技术方面。

 

 

## **DevOps**

DevOps的出现有其必然性。在软件开发生命周期中，遇到了两次瓶颈。第一次瓶颈是在需求阶段和开发阶段之间，针对不断变化的需求，对软件开发者提出了高要求，后来出现了敏捷方法论，强调适应需求、快速迭代、持续交付。第二个瓶颈是在开发阶段和构建部署阶段之间，大量完成的开发任务可能阻塞在部署阶段，影响交付，于是有了DevOps。

**DevOps的**[**三大原则**](https://blog.csdn.net/difffate/article/details/77542768)

**1、基础设施即代码（Infrastructure as Code）**

DeveOps的基础是将重复的事情使用自动化脚本或软件来实现，例如Docker（容器化）、Jenkins（持续集成）、Puppet（基础架构构建）、Vagrant（虚拟化平台）等

**2、持续交付（Continuous Delivery）**

持续交付是在生产环境发布可靠的软件并交付给用户使用。而持续部署则不一定交付给用户使用。涉及到2个时间，TTR（Time to Repair）修复时间，TTM（Time To Marketing）产品上线时间。要做到高效交付可靠的软件，需要尽可能的减少这2个时间。部署可以有多种方式，比如蓝绿部署、金丝雀部署等。

**3、协同工作（Culture of Collaboration）**

开发者和运维人员必须定期进行密切的合作。开发应该把运维角色理解成软件的另一个用户群体。协作有几个的建议：1、自动化（减少不必要的协作）；2、小范围（每次修改的内容不宜过多，减少发布的风险）；3、统一信息集散地（如wiki，让双方能够共享信息）；4、标准化协作工具（比如jenkins）

**描述**

DevOps是一种[**文化**](https://www.cnblogs.com/jetzhang/p/6068773.html)，一种理念，且是和IT糅合成一整体的。

DevOps早在九年前就有人提出来，但是，为什么这两年才开始受到越来越多的企业重视和实践呢？因为DevOps的发展是独木不成林的，现在有越来越多的技术支撑。**微服务架构理念、容器技术使得DevOps的实施变得更加容易**，计算能力提升和云环境的发展使得快速开发的产品可以立刻获得更广泛的使用。

老一派的软件开发团队会包含业务，系统架构师，前端开发，后端开发，测试等等。优化如敏捷和精益原则等的软件开发流程的关注点就在这些地方。比如，软件一旦达到”可以生产“的程度，就会发到系统工程师、发布工程师、DBA、网络工程师，安全专家这些“运维人员”的手上。这里该如何**将横在Dev(开发)和Ops(运维)之间的鸿沟给填平**，这就是DevOps的主要关注点了。

要采纳DevOps的原则，理解整个运作系统的重要性并对工作事项进行合适的**优先级排序**是组织首先要学的事情。在整个价值流中不能允许任何人产生瓶颈并降低整个工作流程。

确保工作流程的不可中断是身处流程中的所有成员的终极目标。**无论**一个成员或者团队的**角色**是什么，他们都必须力图对**整个系统**进行**深入**的**理解**。这种思维方式对质量会有着直接的影响，因为缺陷永远不会被下放到“下游“中，这样做的话将会导致瓶颈的产生。

DevOps原则不关心你身处哪个团队，你是否是系统架构师，DBA，QA，或者是网络管理员。相同的规则覆盖所有的成员，每个成员都应该遵循两个简单的原则：

- 保持系统运作流程不可中断
- 随时提升和优化工作流程

 

**总结**

使用现代化的DevOps工具，如Chef、Docker、Ansible、Packer、Troposphere、Consul、Jenkins、SonarQube、AWS等，并不代表你就在正确的应用DevOps的原则。DevOps是一种思维方式。我们所有人都是该系统流程的一部分，我们一起分享共同的时光和交付价值。每个参加到这个软件交付流程上来的成员都能够加速或减缓整个系统的运作速度。系统出现的一个缺陷，以及错误配置的团队之间的“防火墙”，都可能会使得整个系统瘫痪，所有的人都是DevOps的一部分，一旦你的组织明白了这一点，能够帮你管理好这些的工具和技术栈就自然而然的会出现在你眼前了。

 

 

## **Docker启动关闭**

启动：

```
systemctl start docker
```

关闭：

```
systemctl stop docker
```

重启：

```
systemctl restart docker
```

 



## **Ubuntu 18.04** **安装** [**Docker**](http://www.runoob.com/docker/ubuntu-docker-install.html)

```
可指定版本：
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common

# step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

# Step 3: 写入软件源信息，获取可安装的docker版本 （/etc/apt/sources.list.d/docker.list）
# 其中$(lsb_release -cs) 为获取当前系统的版本，bionic为18.04，xenial为16.04，如果要安装16.04版本的docker，这里就需要单独指定为xenial
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

# Step 4: 更新并安装 Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce

# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages

# Step 2: 安装指定版本的Docker-CE: (VERSION 例如上面的 18.06.3~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]
```





## **Ubuntu删除卸载**

删除所有 Docker 容器

```
docker rm $(docker ps -a -q)
```

删除所有镜像

```
docker rmi $(docker images -q)
```

删除所有的卷

```
docker volume rm $(docker volume ls -q)
```

卸载docker

```
apt-get remove docker-ce
```





## **CenOS 7 安装** [**Docker**](http://blog.51cto.com/11887934/2050590)

```
# 安装依赖包
yum install -y yum-utils device-mapper-persistent-data lvm2

# 添加Docker软件包源（这里在安装完docker之后，需要把/etc/yum.repos.d/docker-ce.repo删除掉，否则下次安装其他程序的时候会有异常）
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
// 国内安装：sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#关闭测试版本list（只显示稳定版）
sudo yum-config-manager --disable docker-ce-edge
sudo yum-config-manager --disable docker-ce-test

# 更新yum包索引
yum makecache fast

#NO.2 指定版本安装
yum list docker-ce --showduplicates|sort -r  
yum install docker-ce-18.06.3.ce -y

# 启动
systemctl start docker

# 测试
docker run hello-world
docker version

# 设置开机启动
systemctl enable docker

# 卸载
yum remove docker-ce
rm -rf /var/lib/docker
```



## **Debian 安装** [**Docker**](https://www.runoob.com/docker/debian-docker-install.html)

```
# 更新 apt 包索引
apt-get update

# 安装 apt 依赖包
apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common

# 添加 Docker 的官方 GPG 密钥
curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -

# 更新 apt 包索引
apt-get update

# 设置稳定版仓库
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
  $(lsb_release -cs) \
  stable"
  
# 更新 apt 包索引
apt-get update

# 使用第二列中的版本字符串安装特定版本，例如 5:18.09.1~3-0~debian-stretch
apt-get install docker-ce=<VERSION_STRING>

# 测试 Docker 是否安装成功
docker run hello-world
```



 

## **Docker默认配置文件**

[配置文件](https://blog.csdn.net/kozazyh/article/details/79795559)的默认路径：/etc/docker/daemon.json

常用参数选项：

```
{
  "registry-mirrors": ["http://2fce9c54.m.daocloud.io"], #镜像加速地址
  "insecure-registries": ["harbor.test.com","192.168.1.202:5000"], # Docker如果需要从非SSL源管理镜像比如私有镜像源，这里加上。
  "max-concurrent-downloads": 10,
  "dns":["8.8.8.8","8.8.4.4"]
}
```

> 部分参数（registry-mirrors、insecure-registries ...）修改，只要reconfigure(systemctl reload docker) 就[生效](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-user-namespace-options)。



**更新Docker镜像源**

快速方式：

```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://2fce9c54.m.daocloud.io
```

稳定方式：

```
# vi /etc/docker/daemon.json

{
 "registry-mirrors": ["http://2fce9c54.m.daocloud.io"],
 "dns":["8.8.8.8","8.8.4.4"]
}
```





## **基础命令操作**

 

### **文件**[**上传下载**](https://imnerd.org/docker-in-action.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

**docker cp** [OPTIONS] SRC_PATH CONTAINER**:**DEST_PATH

上传文件到Docker容器中。

```
docker cp /root/app/jjh.txt 87fdc5475352:/home
```

 

**docker cp** [OPTIONS] CONTAINER**:**SRC_PATH DEST_PATH

从docker容器下载文件。

```
docker cp 87fdc5475352:/home /root/app/jjh.txt
```



### **进入Docker容器**

**docker attach** [OPTIONS] CONTAINER

要attach上去的容器必须正在运行才行，但是这样登录上去的容器有一个缺点，那就是一旦我们执行退出操作（比如说当我们执行exit指令），正在运行的容器也立即停止了。

```
docker attach 87fdc5475352
```



**docker** **exec** [OPTIONS] CONTAINER COMMAND[ARG...]

执行exit或者ctrl+D不会停止容器。

> -d :分离模式: 在后台运行
>
> -i :即使没有附加也保持STDIN 打开
>
> -t :分配一个伪终端

```
docker exec -it 87fdc5475352 /bin/sh
```



### **Docker容器打包导出导入**

**docker** **commit** CONTAINER IMAGENAME 

打包容器为镜像，第一个参数为容器名称或者容器ID，第二个为镜像名称。

```
docker commit 87fdc5475352 docker-image
```



**docker save -o** FILENAME IMAGENAME 

保存镜像为文件，第一个参数为要保存的文件名，第二个为要保存的镜像(保存的文件存在当前命令[pwd]所在的目录下，保存完可ls查看)

```
docker save -o redis.tar docker-image
```

 

**docker** **load** **--input** FILENAME **或者** **docker load** < FILENAME

从文件载入镜像，第一个参数为要导入的文件名

```
docker load < redis.tar
```

 

### **镜像操作**

**docker inspect** IMAGEID|IMAGENAME 

查看镜像信息，第一个参数为镜像ID或者镜像名称。

```
docker inspect jjh-docker-redis
```



**docker** **tag** IMAGEID REPOSITORY:TAG 

重命名镜像，第一个参数为镜像ID，第二个为新镜像名称（仓库：标签）。

```
docker tag 87fdc5475352 jjh-redis:v1.0
```



**docker** **rmi** IMAGEID|IMAGENAME 

删除镜像，第一个参数为镜像ID或者镜像名称。

```
docker rmi jjh-docker-redis
```



 

### **容器操作**

**docker inspect** [CONTAINER...]

查看容器的配置信息，包含容器名、环境变量、运行命令、主机配置、网络配置和数据卷配置等。

```
docker inspect redis
```



**docker** **top** [CONTAINER...]

查看容器中正在运行的进程。

```
docker top redis
```



**docker** **rm** CONTAINER [CONTAINER...]

删除一个或者多个容器。如果删除所有停止的容器，使用：docker rm $(docker ps -a -q)，慎用。

```
docker rm redis
```



### **Docker日志**

**docker** **logs** IMAGEID|IMAGENAME 

docker容器的日志存放在/var/lib/docker/containers/下对于容器的目录下面。

> /var/lib/docker/containers/**[容器ID]**-json.log

使用**truncate**命令，可以将nginx容器的日志文件“清零”：

```
truncate -s 0 /var/lib/docker/containers/a376aa694b22ee497f6fc9f7d15d943de91c853284f8f105ff5ad6c7ddae7a53/*-json.log
```

查看最近30分钟的日志：

```
docker logs --since 30m CONTAINER_ID
```

查看某时间段日志：

```
docker logs -t --since="2018-02-08T13:23:37" --until "2018-02-09T12:23:37" CONTAINER_ID
```

实时输出最后20行日志

```
docker logs -f --tail=20 CONTAINER_ID
```



设置容器服务的日志大小[上限](https://blog.csdn.net/yjk13703623757/article/details/80283729)，要从根本上解决问题，需要限制容器服务的日志大小上限。这个通过配置容器docker-compose的max-size选项来实现：

```
nginx: 
  image: nginx:1.12.1 
  restart: always 
  logging: 
    driver: “json-file” 
    options: 
      max-size: “5g”
```



### **Docker容器路径**

在默认情况下，Docker镜像和容器的默认存放位置为**:/var/lib/docker**

该路径存放docker所使用的镜像文件。



 

## **Dockerfile**

使用dockerfile生成镜像的时候，进入到项目根路径下面，执行：**docker build -t** **myimage** **.** （注意后面的点不要省略）即可。

示例[来源：](https://segmentfault.com/a/1190000010541792)

**FROM**

语法：FROM <image>[:<tag>]
解释：设置要制作的镜像基于哪个镜像，FROM指令必须是整个Dockerfile的第一个指令，如果指定的镜像不存在默认会自动从Docker Hub上下载。

**MAINTAINER**

语法：**MAINTAINER** <name>
解释：**MAINTAINER**指令允许你给将要制作的镜像设置作者信息。

**ADD**

语法：**ADD** <src> <dest>
解释：**ADD**指令用于从指定路径拷贝一个文件或目录到容器的指定路径中，<src>是一个文件或目录的路径，也可以是一个url，路径是相对于该Dockerfile文件所在位置的相对路径，<dest>是目标容器的一个绝对路径。

> COPY指令和ADD指令的唯一区别在于是否支持从远程URL获取资源。COPY指令只能从执行docker build所在的主机上读取资源并复制到镜像中。而ADD指令还支持通过URL从远程服务器读取资源并复制到镜像中。

**WORKDIR**

语法：**WORKDIR** /path/to/workdir
解释：**WORKDIR**指令用于设置Dockerfile中的RUN、CMD和ENTRYPOINT指令执行命令的工作目录(默认为/目录)，该指令在Dockerfile文件中可以出现多次，如果使用相对路径则为相对于WORKDIR上一次的值，例如WORKDIR /data，WORKDIR logs，RUN pwd最终输出的当前目录是/data/logs。

**RUN**

语法：① **RUN** <command>  #将会调用/bin/sh -c <command>
    ② **RUN** ["executable", "param1", "param2"] #将会调用exec执行，以避免有些时候shell方式执行时的传递参数问题，而且有些基础镜像可能不包含/bin/sh
解释：**RUN**指令会在一个新的容器中执行任何命令，然后把执行后的改变提交到当前镜像，提交后的镜像会被用于Dockerfile中定义的下一步操作，RUN中定义的命令会按顺序执行并提交，这正是Docker廉价的提交和可以基于镜像的任何一个历史点创建容器的好处，就像版本控制工具一样。

**ENV**

语法：**ENV** <key> <value>
解释：**ENV**指令用于设置环境变量，在Dockerfile中这些设置的环境变量也会影响到**RUN**指令，当运行生成的镜像时这些环境变量依然有效，如果需要在运行时更改这些环境变量可以在运行docker run时添加–env <key>=<value>参数来修改。
 注意：最好不要定义那些可能和系统预定义的环境变量冲突的名字，否则可能会产生意想不到的结果。

**EXPOSE**

语法：**EXPOSE** <port> [ ...]
解释：**EXPOSE**指令用来告诉Docker这个容器在运行时会监听哪些端口，Docker在连接不同的容器(使用–link参数)时使用这些信息。

**CMD**

语法：

​	① **CMD** ["executable", "param1", "param2"]  #将会调用exec执行，首选方式
​	② **CMD** ["param1", "param2"]    #当使用ENTRYPOINT指令时，为该指令传递默认参数
​    ③ **CMD** <command>     #将会调用/bin/sh -c执行
解释：**CMD**指令中指定的命令会在镜像运行时执行，在Dockerfile中只能存在一个，如果使用了多个CMD指令，则只有最后一个CMD指令有效。当出现ENTRYPOINT指令时，CMD中定义的内容会作为ENTRYPOINT指令的默认参数，也就是说可以使用CMD指令给ENTRYPOINT传递参数。

> 注意：**RUN**和CMD都是执行命令，他们的差异在于RUN中定义的命令会在执行docker build命令创建镜像时执行，而CMD中定义的命令会在执行docker run命令运行镜像时执行，另外使用第一种语法也就是调用exec执行时，命令必须为绝对路径。





## **Docker-Compose**

Compose 是 Docker 公司推出的一个[工具软件](https://www.jianshu.com/p/2217cfed29d7)，可以管理多个 Docker 容器组成一个应用（称为一个 project，即项目）。Compose 定位是“Define and run multi-container applications with Docker”，其前身是 Fig，兼容 Fig 的模板文件。你需要定义一个 YAML 格式的配置文件docker-compose.yml，写好多个容器之间的调用关系。然后，只要一个命令，就能同时启动/关闭这些容器。

**安装**

\# 下载docker-compose，将下载好的文件放入/usr/local/bin/docker-compose目录，并赋予可执行权限。

```
sudo curl -L "[https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)](https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m))" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

> 注意：如果命令docker-compose在安装后失败，也可以创建指向/usr/bin或路径中任何其他目录的符号链接。
>
> sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose



**使用**

编辑docker-compose.yml。

执行文件：docker-compose up **-d**

**注：**-d 参数为后台运行

其他常用命令：

```
  down           Stop and remove containers, networks, images, and volumes
  kill               Kill containers
  logs             View output from containers
  port             Print the public port for a port binding
  ps                List containers
  restart         Restart services
  rm               Remove stopped containers
  start             Start services
  stop             Stop services
  up               Create and start containers
```

**示例：**

```
elk:
  image: sebp/elk
  ports:
    - "5601:5601"
    - "9200:9200"
    - "5044:5044"
    - "9400:9400"
  volumes:
    - /root/soft/elk/data/logstash:/opt/logstash/data
  command:
	sh tracker.sh
```

 

**部分**[**语法**](https://blog.csdn.net/pushiqiang/article/details/78682323)

**environment**

设置镜像变量，它可以保存变量到镜像里面，也就是说启动的容器也会包含这些变量设置

```
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SHOW=true
- SESSION_SECRET
```

 

**ports**

映射端口，使用HOST:CONTAINER格式或者只是指定容器的端口，宿主机会随机映射端口。**（如果是udp要注意要声明）**

```
ports:
 - "3000"
 - "8000:8000/udp"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```

批量映射端口：

```
- "21000-21020:1000-1020"
```

 

**volumes**

挂载一个目录或者一个已存在的数据卷容器，可以直接使用 [HOST:CONTAINER] 这样的格式，或者使用 [HOST:CONTAINER:ro] 这样的格式，后者对于容器来说，数据卷是只读的，这样可以有效保护宿主机的文件系统。

```
volumes:
  // 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）。
  - /var/lib/mysql
  // 使用绝对路径挂载数据卷
  - /opt/data:/var/lib/mysql
  // 以 Compose 配置文件为中心的相对路径作为数据卷挂载到容器。
  - ./cache:/tmp/cache
  // 使用用户的相对路径（~/ 表示的目录是 /home/<用户目录>/ 或者 /root/）。
  - ~/configs:/etc/configs/:ro
  // 已经存在的命名的数据卷。
- datavolume:/var/lib/mysql
```

 

**network_mode**

网络模式，yml文件中与ports同级。与Docker client的--net参数类似，只是相对多了一个service:[service name] 的格式。

```
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

 

**restart**

```
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```

 

**command**

使用 command 可以覆盖容器启动后默认执行的命令

```
command: bundle exec thin -p 3000
```

或者

```
command: [bundle, exec, thin, -p, 3000]
```



## **Docker轻量级管理**[**工具**](https://segmentfault.com/a/1190000016413932)

**[Portainer](https://portainer.readthedocs.io/en/latest/deployment.html)**

```
docker run -d -p 9001:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v $PWD/data:/data portainer/portainer
```



## Docker端口映射

在docker中运行第三方服务时，通常需要绑定服务端口到本地主机。但使用 -p 参数进行的端口映射，会**自动在iptables中建立规则**，绕过firewalld。





## Docker应用



### Docker-MySQL5.6

**docker run -p** 3306:3306 **--name** mymysql **-v** $PWD/conf:/etc/mysql/conf.d **-v** $PWD/logs:/logs **-v** $PWD/data:/var/lib/mysql **-e** MYSQL_ROOT_PASSWORD=123456 **-d** mysql:5.6

命令说明：

> -p 3306:3306：将容器的3306端口映射到主机的3306端口
> -v $PWD/conf:/etc/mysql/conf.d：将主机当前目录下的conf/my.cnf挂载到容器的/etc/mysql/my.cnf
> -v $PWD/logs:/logs：将主机当前目录下的logs目录挂载到容器的/logs
> -v $PWD/data:/var/lib/mysql：将主机当前目录下的data目录挂载到容器的/mysql_data
> -e MYSQL_ROOT_PASSWORD=123456：初始化root用户的密码
> -d mysql:5.6：镜像名

注意数据库版本：5.6和5.7在创建时映射的配置文件目录有所不同

**docker-compose.yml**

```
version: '2'
services:
        mysql56:
                image: mysql:5.6
                volumes:
                        - ./conf:/etc/mysql/conf.d
                        - ./logs:/logs
                        - ./data:/var/lib/mysql
                ports:
                        - "3306:3306"
                environment:
                        - MYSQL_ROOT_PASSWORD=root
```



### **Docker-MySQL5.7**

**docker run -p** 3307:3306 **--name** mysql5.7 **-v** $PWD/logs:/logs **-v** $PWD/data:/var/lib/mysql **-e** MYSQL_ROOT_PASSWORD=root **-d mysql:5.7** 

> 注：MySQL(5.7.19)的默认配置文件是 /etc/mysql/my.cnf 文件。如果想要自定义配置，建议向 /etc/mysql/conf.d 目录中创建 .cnf 文件。新建的文件可以任意起名，只要保证后缀名是 cnf 即可。新建的文件中的配置项可以覆盖 /etc/mysql/my.cnf 中的配置项。

**特别注意：**mysql容器启动后mysql服务不会立即启动，需要等待1分多钟初始化，不要马上登陆mysql，等初始化完成之后再登陆。

**具体操作：**首先需要创建将要映射到容器中的目录以及.cnf文件，然后再创建容器



### **MySQL主从库**

**docker run -d -e** REPLICATION_MASTER=true **-e** REPLICATION_PASS=mypass **-p** 3306:3306 **--name** mysql tutum/mysql

**docker run -d -e** REPLICATION_SLAVE=true **-p** 3307:3306 **--link** mysql:mysql tutum/mysql

 

### **Docker-Redis**

**docker run -p** 6379:6379 **-v** $PWD/data:/data **-d** redis **redis-server --appendonly** yes

命令说明：

> -p 6379:6379 : 将容器的6379端口映射到主机的6379端口
>
> -v $PWD/data:/data : 将主机中当前目录下的data挂载到容器的/data
>
> redis-server --appendonly yes : 在容器执行redis-server启动命令，并打开redis持久化配置

运行redis容器设置redis访问密码：

**docker run** -d --name myredis -p 6379:6379 redis --requirepass "mypassword"



### **Docker-influxdb**

docker-compose.yml

```
version: '2'
services:
	    influxdb:
	        image: influxdb
	        environment:
	            INFLUXDB_ADMIN_ENABLED: "true"
	            INFLUXDB_ADMIN_USER: "root"
	            INFLUXDB_ADMIN_PASSWORD: "123456"
	            INFLUXDB_DB: railway
	        volumes:
	            - ./data:/var/lib/influxdb
	        ports:
            - "8086:8086"
```



### **Docker-PostgreSQL**

**docker run -p** 54321:5432 **-e** POSTGRES_PASSWORD=root **-d** postgres

命令说明：

> -p 54321:5432 : 将容器的5432端口映射到主机的54321端口
>
> -e POSTGRES_PASSWORD=root : 设置环境变量，指定数据库的登录口令为root
>
> -d postgres : 所使用镜像的名称



### **Docker-OracleXE**

**docker pull** wnameless/oracle-xe-11g

**docker run -d -p** 49160:22 **-p** 49161:1521 **wnameless/oracle-xe-11g**

> 注：
>
> docker pull 比较耗时，中间可能会中断，继续[拉取即可](https://blog.csdn.net/wm5920/article/details/78770556)
>
> 数据库信息：**username**: system/sys **password**: oracle



### **Docker-SQLServer**

docker-compose.yml

```
version: '2'
services:
        sqlserver2017:
                image: microsoft/mssql-server-linux:2017-latest
                volumes:
                        - ./mssql:/var/opt/mssql
                ports:
                        - "1401:1433"
                environment:
                        - MSSQL_SA_PASSWORD=sqlserver123!
                        - ACCEPT_EULA=Y
```

> [注：](https://blog.csdn.net/xyh153996626/article/details/79022703)
>
> ACCEPT_EULA=Y的意思是同意许可协议，必选；
> MSSQL_SA_PASSWORD为密码，要求是最少8位的强密码，要有大写字母，小写字母，数字以及特殊符号，不然会有一个大坑（docker启动sqlserver容器后过几秒就停止了）；



### **Docker-WordPress**

**docker run** **--name** mywordpress **--link** 23144a2854f0:mysql **-p 808****1****:80 -****d** wordpress

命令说明：

> -p 8081:80 : 将容器的80端口映射到主机的8081端口
>
> --name mywordpress : 指定新实例名称
>
> --link 23144a2854f0:mysql：指定要使用的Docker MySQL实例
>
> -d postgres : 所使用镜像的名称



### Docker-Java

docker-compose.yml：

```yaml
version: '2'
services:
    java:
        image: openjdk:8u222-jdk
        # 日志容量限制
        logging:
            driver: "json-file"
            options:
                max-size: "1g"
        environment:
            TZ: Asia/Shanghai
        volumes:
            # 时区同步
            - /etc/localtime:/etc/localtime:ro
            # 应用存放目录
            - ./app:/app
        ports:
            - "18610:8084"
        command: java -server -jar /app/app.jar --spring.profiles.active=test
```



### Docker-Tomcat

docker-compose.yml：

```yaml
version: '2'
services:
    tm:
        image: tomcat:8.5.46
        # 日志容量限制
        logging:
            driver: "json-file"
            options:
                max-size: "1g"
        environment:
            TZ: Asia/Shanghai
        volumes:
            - ./webapps:/usr/local/tomcat/webapps/
#            - ./conf:/usr/local/tomcat/conf/
#            - ./bin/catalina.sh:/usr/local/tomcat/bin/catalina.sh
            - ./logs:/usr/local/tomcat/logs/
            - /etc/localtime:/etc/localtime:ro
        ports:
            - "29091:8080/tcp"
```

> FAQ：映射tomcat的catalina.sh错误
>
> ERROR: for my_tm Cannot start service my_tm: b'OCI runtime create failed: container_linux.go:348: starting container process caused "exec: [\\"catalina.sh\\](file://"catalina.sh/)": executable file not found in $PATH": unknown'
>
> 解决：需要给映射的文件赋权：chmod 777 ./catalina.sh



## **Maven结合Docker**

docker开启[远程访问](https://blog.csdn.net/she_lock/article/details/79557022)：

```
vi /lib/systemd/system/docker.service
```

 在[Service]这个部分，加上下面两行参数：

```
ExecStart= 
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```

 

## **重启docker**

systemctl daemon-reload //重新读取配置文件 

systemctl restart docker //重新启动服务

 

 

## **Docker-Machine**

[Docker Machine](https://www.jianshu.com/p/0d9659080bd5) 是 Docker 官方提供的一个工具，它可以帮助我们在远程的机器上安装 Docker，或者在虚拟机 host 上直接安装虚拟机并在虚拟机中安装 Docker。我们还可以通过 docker-machine 命令来管理这些虚拟机和 Docker。

 

 

## **Docker** **版本**

**有关docker,docker.io,docker-engine,lxc-docker 的**[**区别**](https://www.cnblogs.com/lizichao1991/p/7646917.html)**？**

其中，RHEL/CentOS 软件源中的 Docker 包名为 docker；Ubuntu 软件源中的 Docker 包名为 docker.io；而很古老的 Docker 源中 Docker 也曾叫做 lxc-docker。这些都是非常老旧的 Docker 版本，并且基本不会更新到最新的版本，而对于使用 Docker 而言，使用最新版本非常重要。另外，17.04 以后，包名从 docker-engine 改为 docker-ce，因此从现在开始安装，应该都使用 docker-ce 这个包。





## **Docker时区**

Base Image 使用的基本上都是 Docker 官方的，所以里面的时间设置大多是 Etc/UTC，也就是标准的 UTC 时间，所以要简单的[调整](https://tommy.net.cn/2015/02/05/config-timezone-in-docker/)一下，变成中国标准时间。

1 如果已经创建了 container 的话，可以用docker的root用户进入到 container 里面，用命令行实现时区的更改：

```
docker exec -it -uroot 18c3d53deff3 /bin/bash
echo "Asia/Shanghai" > /etc/timezone
dpkg-reconfigure -f noninteractive tzdata
```

2 如果是在Dockerfile中进行修改，则可以使用如下命令：

```
RUN echo "Asia/Shanghai" > /etc/timezone
RUN dpkg-reconfigure -f noninteractive tzdata
```

3 使用docker-compose.yml，其中设置volumes为:ro表示只读：

```
volumes:
    - /etc/localtime:/etc/localtime:ro
environment:
             - TZ=Asia/Shanghai
```

4 快速方法（应用不可靠）：

```
docker cp /etc/localtime [容器ID或者NAME]:/etc/localtime
```



## **Docker搭建私有仓库**

1 使用docker-compose.yml创建镜像服务器：

```
version: '2'
services:
    docker-registry:
        image: registry:latest
        ports:
            - "5000:5000"
        volumes:
            - /docker/registry
        restart: always
        privileged: true
```

2 开放http服务器（免去SSL验证[https]）：

```
vim /etc/docker/daemon.json
# 添加配置文件 
{"insecure-registries":["192.168.66.100:5000"]} 

# 重启服务 
systemctl reload docker
```

3 推送镜像到私有仓库：

```
#必须要先将镜像的名称给变成  域名或ip/镜像名
docker tag centos7:latest 192.168.66.100:5000/centos7
#推送到本地的仓库上
docker push 192.168.66.100:5000/centos7
```

4 访问网页查看是否成功：

http://192.168.1.202:5000/v2/_catalog

5 下载私有仓库镜像：

```
#删除本地上传的那个仓库，然后下载看看
docker rmi 192.168.66.100:5000/centos7
#下载本地仓库的镜像
docker pull 192.168.66.100:5000/centos7
```



## **DockerWine**

在Docker容器中运行Microsoft Windows应用[程序](https://hub.docker.com/r/suchja/wine/)

在挂在程序文件的时候如果需要新建文件之类的，建议使用直接挂载的目录，而不是挂载目录的子目录，否则可能出现权限问题导致命令执行失败

```
#下载镜像
docker pull suchja/wine

#启动容器
docker run --rm -it --entrypoint /bin/bash suchja/wine:latest
# 或者
docker run --rm -it -v /home/app/wine/file:/file --entrypoint /bin/bash suchja/wine:latest

#初始化
wine wineboot --init

#开始使用Wine
wine notepad.exe
```

**自定义wine**[**镜像**](http://cache.baiducontent.com/c?m=9f65cb4a8c8507ed4fece76310508c31490797634b87834e29938448e435061e5a24febd2d201704d5c0776604bb0c01aaa639246a5279e0dbdf883b98e0ce7f&p=8e57c54ad5c34af812b9c7710f48cf&newp=997dce1785cc43b310bd9b7d0d1d91231610db2151d6d2106b82c825d7331b001c3bbfb423261206d7c47f6406ae4e5be8f63676350923a3dda5c91d9fb4c57479946f&user=baidu&fm=sc&query=docker+wine&qid=92eac7620000a6e4&p1=2)

Dockerfile：

```dockerfile
FROM ubuntu:16.04
RUN useradd -m -g users docker
RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
RUN dpkg --add-architecture i386
RUN apt update
RUN apt install --no-install-recommends --assume-yes software-properties-common
RUN add-apt-repository ppa:wine/wine-builds
RUN apt update
RUN apt install --no-install-recommends --assume-yes winehq-staging winetricks
ENV DISPLAY :0
RUN mkdir /home/docker/game && mkdir /home/docker/game/ED6_FC
RUN chown -R docker /home/docker/game
WORKDIR /home/docker/game/ED6_FC
USER docker
RUN winetricks d3dxof
```

构建运行

```
# 构建镜像
docker build -t mywine .
# 运行容器
docker run --rm -it -v /home/app/wine/file:/file mywine:latest
# 执行wine
wine /file/xxx.exe
```



## **日志驱动**

| **驱动**                                                     | **描述**                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| none                                                         | 没有可用于容器的日志，docker logs也不返回任何输出。          |
| [json-file](https://docs.docker.com/config/containers/logging/json-file/) | 日志格式为JSON。Docker的默认日志记录驱动程序。               |
| [syslog](https://docs.docker.com/config/containers/logging/syslog/) | 将日志消息写入syslog设施。该syslog守护程序必须在主机上运行。 |
| [journald](https://docs.docker.com/config/containers/logging/journald/) | 将日志消息写入journald。该journald守护程序必须在主机上运行。 |
| [gelf](https://docs.docker.com/config/containers/logging/gelf/) | 将日志消息写入Graylog扩展日志格式（GELF）端点，例如Graylog或Logstash。 |
| [fluentd](https://docs.docker.com/config/containers/logging/fluentd/) | 将日志消息写入fluentd（转发输入）。该fluentd守护程序必须在主机上运行。 |
| [awslogs](https://docs.docker.com/config/containers/logging/awslogs/) | 将日志消息写入Amazon CloudWatch Logs。                       |
| [splunk](https://docs.docker.com/config/containers/logging/splunk/) | splunk使用HTTP事件收集器将日志消息写入。                     |
| [etwlogs](https://docs.docker.com/config/containers/logging/etwlogs/) | 将日志消息写为Windows事件跟踪（ETW）事件。仅适用于Windows平台。 |
| [gcplogs](https://docs.docker.com/config/containers/logging/gcplogs/) | 将日志消息写入Google Cloud Platform（GCP）日志记录。         |
| [logentries](https://docs.docker.com/config/containers/logging/logentries/) | 将日志消息写入Rapid7 Logentries。                            |



## 查看docker的连接（获取docker连接数）

**1** **查找docker的进程号 ：**

```
docker inspect -f '{{.State.Pid}}' <containerid>
```

**2** **查看连接：** 

```
sudo nsenter -t <pid> -n netstat | grep ESTABLISHED
```

> **示例：**
> $ docker inspect -f '{{.State.Pid}}' 49b98b2fbad2
> 1840
> $ nsenter -t 1840 -n netstat |grep ESTABLISHED
> udp    0   0 node-2:45963    10.254.0.2:domain    ESTABLISHED



## **Docker Swarm**

创建主[节点](https://www.cnblogs.com/franknihao/p/8490416.html)

```
docker swarm init
```

从节点加入 （使用创建主节点时生成的token）

```
docker swarm join --token SWMTKN-1-2mi0u8jhfqe1ihvx2n9nc569cxe77pggoq71nl9lkf033m0554-0t2j9gcd9o5fj228vyxzxihww 192.168.1.103:2377
```

从节点离开

```
docker swarm leave
```



 

## **进入容器内命令行闪退**

当docker容器的命令行进入闪退的时候，有可能是由于所使用的shell版本不一样导致的，可以更换/bin/bash为/bin/sh[再试](https://blog.csdn.net/qq_34018840/article/details/94397743)



 

## **Docker Stack**

[服务部署](https://blog.csdn.net/huangjun0210/article/details/86502021)

```
docker stack deploy example --compose-file=docker-compose.yml 
```

**常用命令**

- docker stack deploy    部署新的堆栈或更新现有堆栈
- docker stack ls    列出现有堆栈
- docker stack ps    列出堆栈中的任务
- docker stack rm    删除一个或多个堆栈
- docker stack services    列出堆栈中的服务



## **修改docker容器**[**目录**](https://segmentfault.com/a/1190000039426040)

```shell
# 1.停止docker服务
$ sudo systemctl stop docker

# 2.开始迁移目录
$ sudo mv /var/lib/docker /data/

# 3.改动docker启动配置文件
$ sudo vim /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd --graph=/data/docker/

# 4.改动docker启动配置文件
$ sudo vim /etc/docker/daemon.json
{
    "live-restore": true,
    "graph": [ "/data/docker/" ]
}
# 操作注意事项：在迁移 docker 目录的时候注意使用的命令，要么使用 mv 命令直接移动，要么使用 cp 命令复制文件，但是需要注意同时复制文件权限和对应属性，不然在使用的时候可能会存在权限问题。如果容器中，也是使用 root 用户，则不会存在该问题，但是也是需要按照正确的操作来迁移目录。

# 使用mv命令
$ sudo mv /var/lib/docker /data/docker

# 使用cp命令
$ sudo cp -arv /data/docker /data2/docker
```





## [创建不自动关闭的容器](https://blog.csdn.net/xiaojinran/article/details/104112874)

```yaml
version: '2'
services:
    pymssql:
        image: python:3.9.2
        container_name: pymssql
        logging:
            driver: "json-file"
            options:
                max-size: "100m"
        environment:
            TZ: Asia/Shanghai
        volumes:
            - /etc/localtime:/etc/localtime:ro
        entrypoint: ping baidu.com
```

