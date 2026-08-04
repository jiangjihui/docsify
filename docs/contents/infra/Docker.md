# Docker

## 概述

### 什么是 Docker

Docker 是一个开源的容器化平台，用于开发、部署和运行应用程序。Docker 通过容器（Container）来打包应用程序及其依赖项，实现"一次构建，多处运行"的理念。

**Docker 的核心优势：**
- **轻量级**：容器共享宿主机的内核，无需额外的操作系统开销
- **快速启动**：容器可以在秒级时间内启动
- **可移植性**：一次构建即可在不同的环境中运行
- **版本控制**：支持镜像版本管理
- **资源隔离**：提供进程级别的资源隔离

### 容器与微服务

容器技术是微服务架构的重要支撑。在微服务场景下，进程多、更新快，可能存在上百个容器实例。传统的虚拟机构镜像过大，而容器镜像小巧，非常适合微服务架构。

**容器化的本质**：基于镜像的跨环境迁移。镜像（Image）是 Docker 的核心发明，是封装和运行的标准。

### Docker 与 Kubernetes

Docker 和 Kubernetes（K8S）都是容器运行时，但作用范围不同：

| 特性 | Docker | Kubernetes |
|------|--------|------------|
| 作用范围 | 单机 | 集群 |
| 主要功能 | 容器创建与管理 | 容器编排与调度 |
| 适用场景 | 开发、测试 | 生产环境 |

> **说明**：Docker Desktop 已内置 Kubernetes，单机也可体验 K8S 功能。

#### Dockershim 的历史

自 K8S v1.24 起，Dockershim 已被移除。这是 K8S 项目推动标准化的重要举措。

**历史背景**：
- K8S 早期只支持 Docker Engine 作为容器运行时
- 随着容器运行时增多，K8S 引入 CRI（容器运行时接口）来支持多种运行时
- Docker Engine 不兼容 CRI，因此创建了 dockershim 作为适配层
- dockershim 从未打算永久存在，增加了维护复杂度和系统不稳定性

**总结**：dockershim 是 K8S 社区为兼容 Docker 而维护的过渡性程序。移除后，K8S 可以专注于维护 CRI 标准，任何兼容 CRI 的容器运行时都可以作为 K8S 的 runtime。

---

## 核心概念

### Docker 架构

Docker 采用客户端-服务器架构，主要组件包括：

```
┌─────────────────────────────────────────┐
│              Docker Host                │
│  ┌─────────────────────────────────┐   │
│  │         Docker Daemon           │   │
│  │   (管理镜像、容器、网络、卷)    │   │
│  └─────────────────────────────────┘   │
│  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  │
│  │容器1│  │容器2│  │容器3│  │容器4│  │
│  └─────┘  └─────┘  └─────┘  └─────┘  │
│  ┌─────────────────────────────────┐   │
│  │      Container Runtime         │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
          ▲
          │ REST API
          ▼
┌─────────────────────────────────┐
│       Docker Client            │
│    (docker 命令行工具)         │
└─────────────────────────────────┘
```

### 镜像（Image）

镜像是一个只读的模板，用于创建容器。镜像可以理解为面向对象中的"类"。

**镜像特点：**
- 分层存储：每一层都可以被复用
- 不可变性：创建后不可修改
- 版本化：支持 tag 标记版本

**常用镜像操作：**

```bash
# 拉取镜像
docker pull nginx:latest

# 列出本地镜像
docker images

# 删除镜像
docker rmi nginx:latest

# 标记镜像
docker tag nginx:latest myregistry/nginx:v1.0

# 构建镜像
docker build -t myapp:latest .
```

### 容器（Container）

容器是镜像的运行实例，可以理解为面向对象中的"对象"。

**容器特点：**
- 轻量级：共享宿主机内核
- 隔离性：独立的文件系统、网络、进程空间
- 临时性：容器中的数据在容器删除后丢失（除非使用卷）

**常用容器操作：**

```bash
# 运行容器
docker run -d -p 8080:80 --name mynginx nginx

# 列出运行中的容器
docker ps

# 列出所有容器
docker ps -a

# 停止容器
docker stop mynginx

# 删除容器
docker rm mynginx

# 进入容器
docker exec -it mynginx /bin/sh
```

### 仓库（Repository）

仓库用于存储和分发镜像。Docker Hub 是官方公共仓库。

**常用仓库操作：**

```bash
# 登录仓库
docker login

# 推送镜像
docker push myregistry/nginx:v1.0

# 拉取镜像
docker pull myregistry/nginx:v1.0
```

---

## 环境配置

### 安装 Docker

#### CentOS 7

```bash
# 安装依赖包
yum install -y yum-utils device-mapper-persistent-data lvm2

# 添加 Docker 软件包源（国内使用阿里云镜像）
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 关闭测试版本
yum-config-manager --disable docker-ce-edge
yum-config-manager --disable docker-ce-test

# 更新 yum 索引
yum makecache fast

# 安装 Docker（指定版本）
yum list docker-ce --showduplicates | sort -r
yum install docker-ce-18.06.3.ce -y

# 或安装最新版本
yum install docker-ce -y

# 启动 Docker
systemctl start docker

# 设置开机自启
systemctl enable docker

# 验证安装
docker version
docker run hello-world
```

#### Ubuntu

```bash
# 更新软件源
sudo apt-get update

# 安装必要工具
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common

# 添加 GPG 密钥
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

# 添加软件源
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

# 更新并安装
sudo apt-get -y update
sudo apt-get -y install docker-ce

# 启动 Docker
sudo service docker start

# 设置开机自启
sudo systemctl enable docker

# 验证安装
docker version
```

#### Debian

```bash
# 更新软件源
apt-get update

# 安装依赖
apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common

# 添加 GPG 密钥
curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -

# 设置仓库
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
  $(lsb_release -cs) \
  stable"

# 更新并安装
apt-get update
apt-get install docker-ce=<VERSION_STRING>

# 验证
docker run hello-world
```

### Docker 服务管理

```bash
# 启动
systemctl start docker

# 停止
systemctl stop docker

# 重启
systemctl restart docker

# 查看状态
systemctl status docker
```

### Docker 配置

Docker 配置文件位于 `/etc/docker/daemon.json`。

```json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"],
  "insecure-registries": ["harbor.example.com", "192.168.1.200:5000"],
  "max-concurrent-downloads": 10,
  "dns": ["8.8.8.8", "8.8.4.4"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

**常用配置项说明：**

| 配置项 | 说明 | 示例 |
|--------|------|------|
| `registry-mirrors` | 镜像加速器地址 | `["https://docker.mirrors.ustc.edu.cn"]` |
| `insecure-registries` | 非 HTTPS 私有仓库 | `["harbor.example.com:5000"]` |
| `dns` | DNS 服务器 | `["8.8.8.8"]` |
| `log-driver` | 日志驱动 | `json-file`、`syslog` |
| `max-concurrent-downloads` | 最大并发下载数 | `10` |

> **说明**：修改配置后需要执行 `systemctl reload docker` 或 `systemctl restart docker` 使配置生效。

### 镜像加速器配置

#### 快速配置

```bash
# 使用 DaoCloud 提供的脚本
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s https://docker.mirrors.ustc.edu.cn
```

#### 手动配置

编辑 `/etc/docker/daemon.json`：

```json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

然后重启 Docker：

```bash
systemctl restart docker
```

### 卸载 Docker

```bash
# CentOS
yum remove docker-ce
rm -rf /var/lib/docker

# Ubuntu/Debian
apt-get remove docker-ce
```

---

## 镜像操作

### 常用镜像命令

| 命令 | 说明 |
|------|------|
| `docker pull` | 拉取镜像 |
| `docker images` | 列出本地镜像 |
| `docker rmi` | 删除镜像 |
| `docker tag` | 标记镜像 |
| `docker inspect` | 查看镜像详情 |
| `docker history` | 查看镜像层 |

### 构建镜像

#### Dockerfile 方式

```bash
# 在项目根目录下执行
docker build -t myapp:latest .
```

```dockerfile
# 示例 Dockerfile
FROM openjdk:8-jdk-alpine

WORKDIR /app

COPY target/app.jar /app/app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

### 镜像导入导出

```bash
# 导出镜像为文件
docker save -o myapp.tar myapp:latest

# 从文件导入镜像
docker load -i myapp.tar

# 或
docker load < myapp.tar
```

### 容器打包为镜像

```bash
# 将容器打包为镜像
docker commit 87fdc5475352 myapp:v1.0
```

> **说明**：`docker commit` 会保存容器的当前状态，但不适合用于可重现的构建。推荐使用 Dockerfile 来定义镜像构建过程。

---

## 容器操作

### 创建与运行容器

```bash
# 运行一个容器
docker run -d -p 8080:80 --name mynginx nginx

# 运行并进入交互式容器
docker run -it --name mycontainer alpine /bin/sh

# 运行容器并挂载卷
docker run -d -v /host/path:/container/path --name myapp myapp:latest

# 运行容器并设置环境变量
docker run -d -e MYSQL_ROOT_PASSWORD=123456 --name mysql mysql:5.7
```

**常用参数说明：**

| 参数 | 说明 |
|------|------|
| `-d` | 后台运行 |
| `-i` | 交互模式 |
| `-t` | 分配伪终端 |
| `-p` | 端口映射（宿主机:容器） |
| `-v` | 卷挂载 |
| `-e` | 环境变量 |
| `--name` | 容器名称 |
| `--link` | 容器链接（已废弃，推荐使用网络） |
| `--restart` | 自动重启策略 |

**`--restart` 取值：**

| 值 | 说明 |
|----|------|
| `no` | 不自动重启（默认） |
| `always` | 总是自动重启 |
| `on-failure` | 失败时重启 |
| `unless-stopped` | 除非手动停止 |

### 进入容器

```bash
# 推荐：使用 exec（退出不会停止容器）
docker exec -it mycontainer /bin/sh

# 不推荐：使用 attach（退出可能停止容器）
docker attach mycontainer
```

### 文件拷贝

```bash
# 复制文件到容器
docker cp /host/path/file.txt mycontainer:/container/path/

# 从容器复制文件到宿主机
docker cp mycontainer:/container/path/file.txt /host/path/
```

### 容器日志

```bash
# 查看日志
docker logs mycontainer

# 实时查看最后 20 行
docker logs -f --tail=20 mycontainer

# 查看最近 30 分钟的日志
docker logs --since 30m mycontainer

# 查看指定时间段的日志
docker logs -t --since="2024-01-01T00:00:00" --until="2024-01-02T00:00:00" mycontainer
```

### 容器管理命令

```bash
# 列出运行中的容器
docker ps

# 列出所有容器（包括已停止）
docker ps -a

# 启动容器
docker start mycontainer

# 停止容器
docker stop mycontainer

# 重启容器
docker restart mycontainer

# 删除容器
docker rm mycontainer

# 强制删除运行中的容器
docker rm -f mycontainer

# 删除所有已停止的容器
docker container prune

# 查看容器内进程
docker top mycontainer

# 查看容器详细信息
docker inspect mycontainer
```

### 端口映射

在 Docker 中运行服务时，需要将容器端口映射到宿主机：

```bash
# 基本端口映射
docker run -d -p 8080:80 --name nginx nginx

# 指定协议
docker run -d -p 8080:80/tcp -p 8081:53/udp --name dns dns-server

# 绑定特定 IP
docker run -d -p 127.0.0.1:8080:80 --name nginx nginx
```

> **注意**：使用 `-p` 参数会在 iptables 中自动建立规则，可能绕过 firewalld。

---

## Dockerfile

Dockerfile 是用于构建镜像的脚本文件。

### 基本指令

| 指令 | 说明 | 示例 |
|------|------|------|
| `FROM` | 指定基础镜像 | `FROM openjdk:8` |
| `MAINTAINER` | 维护者信息 | `MAINTAINER name` |
| `COPY` | 复制文件 | `COPY app.jar /app/` |
| `ADD` | 复制文件（支持 URL） | `ADD app.tar /app/` |
| `WORKDIR` | 设置工作目录 | `WORKDIR /app` |
| `RUN` | 执行命令（构建时） | `RUN apt-get update` |
| `ENV` | 设置环境变量 | `ENV JAVA_HOME=/opt/java` |
| `EXPOSE` | 声明端口 | `EXPOSE 8080` |
| `CMD` | 容器启动命令 | `CMD ["java", "-jar", "app.jar"]` |
| `ENTRYPOINT` | 入口点 | `ENTRYPOINT ["java", "-jar"]` |

### 指令详解

**FROM**

```dockerfile
# 指定基础镜像
FROM openjdk:8-jdk-alpine

# 使用特定标签
FROM openjdk:8u322-jre
```

> **最佳实践**：尽量使用官方镜像并指定具体版本标签。

**WORKDIR**

```dockerfile
# 设置工作目录
WORKDIR /app

# 使用相对路径（相对于上一次 WORKDIR）
WORKDIR logs
# 实际路径为 /app/logs
```

**RUN vs CMD**

- `RUN`：在镜像构建时执行，用于安装软件、配置环境
- `CMD`：在容器运行时执行，是容器的默认启动命令

```dockerfile
# RUN 示例：构建时安装依赖
RUN apk add --no-cache redis

# CMD 示例：运行时执行
CMD ["redis-server"]
```

**COPY vs ADD**

- `COPY`：只能复制本地文件
- `ADD`：支持 URL 源和自动解压 tar 文件

```dockerfile
# 推荐使用 COPY
COPY app.jar /app/

# ADD 用于特殊场景（如远程文件、自动解压）
ADD https://example.com/file.jar /app/
ADD data.tar.gz /app/
```

**CMD vs ENTRYPOINT**

```dockerfile
# CMD 可被 docker run 参数覆盖
CMD ["java", "-jar", "app.jar"]

# ENTRYPOINT 组合 CMD 使用，CMD 作为默认参数
ENTRYPOINT ["java", "-jar"]
CMD ["app.jar"]
```

### 示例

**Java 应用 Dockerfile**

```dockerfile
FROM openjdk:8-jdk-alpine

# 创建应用目录
WORKDIR /app

# 复制应用文件
COPY target/app.jar /app/app.jar

# 暴露端口
EXPOSE 8080

# 设置时区
ENV TZ=Asia/Shanghai
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

# 启动命令
ENTRYPOINT ["java", "-server", "-Xms256m", "-Xmx512m", "-jar", "/app/app.jar"]
```

**多阶段构建**

```dockerfile
# 构建阶段
FROM maven:3.8 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# 运行阶段
FROM openjdk:8-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/app.jar /app/app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

> **最佳实践**：
> - 使用多阶段构建减小镜像体积
> - 使用 `.dockerignore` 排除不需要的文件
> - 合理安排指令顺序以利用缓存
> - 不在镜像中存储敏感信息

---

## Docker Compose

### 概述

Docker Compose 是 Docker 官方提供的容器编排工具，用于定义和运行多容器应用。

**核心功能：**
- 定义多容器应用
- 统一管理容器生命周期
- 服务依赖管理
- 环境变量配置

### 安装

```bash
# 下载并安装
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 添加执行权限
chmod +x /usr/local/bin/docker-compose

# 验证安装
docker-compose --version
```

> **注意**：Docker Desktop 已内置 Compose，Linux 环境需要单独安装。

### docker-compose.yml 语法

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:1.25
    container_name: mynginx
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    environment:
      - TZ=Asia/Shanghai
    restart: always
    networks:
      - mynetwork
    depends_on:
      - app

  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: myapp
    environment:
      - SPRING_PROFILES_ACTIVE=prod
    restart: always
    networks:
      - mynetwork

networks:
  mynetwork:
    driver: bridge

volumes:
  mydata:
    driver: local
```

#### 常用配置项

| 配置项 | 说明 | 示例 |
|--------|------|------|
| `image` | 使用的镜像 | `nginx:1.25` |
| `build` | 构建配置 | `context: .` |
| `container_name` | 容器名称 | `mynginx` |
| `ports` | 端口映射 | `"8080:80"` |
| `volumes` | 卷挂载 | `"./data:/data"` |
| `environment` | 环境变量 | `TZ: Asia/Shanghai` |
| `restart` | 重启策略 | `always` |
| `networks` | 网络配置 | `mynetwork` |
| `depends_on` | 依赖关系 | `app` |
| `command` | 启动命令 | `java -jar app.jar` |

#### volumes 语法

```yaml
volumes:
  # 匿名卷
  - /var/lib/mysql

  # 命名卷
  - mydata:/var/lib/mysql

  # 绑定挂载
  - ./html:/usr/share/nginx/html

  # 只读挂载
  - ./config:/etc/nginx:ro

  # 指定用户
  - ~/configs:/etc/configs:ro
```

#### ports 语法

```yaml
ports:
  # 简单映射
  - "8080:80"

  # 指定协议
  - "8080:80/tcp"
  - "8081:53/udp"

  # 绑定特定地址
  - "127.0.0.1:8080:80"

  # 范围映射
  - "21000-21020:1000-1020"
```

#### restart 策略

| 值 | 说明 |
|----|------|
| `no` | 不自动重启 |
| `always` | 总是自动重启 |
| `on-failure` | 失败时重启 |
| `unless-stopped` | 除非手动停止 |

### 常用命令

```bash
# 启动所有服务
docker-compose up -d

# 停止所有服务
docker-compose down

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f

# 构建镜像
docker-compose build

# 重启服务
docker-compose restart

# 进入容器
docker-compose exec nginx sh

# 扩展服务（需要部署到 swarm）
docker-compose up -d --scale nginx=3
```

---

## 私有仓库

### 搭建私有仓库

```bash
# 使用 Docker Compose 搭建
version: '3'
services:
  registry:
    image: registry:2
    container_name: docker-registry
    ports:
      - "5000:5000"
    volumes:
      - ./registry-data:/var/lib/registry
    restart: always
```

### 配置 HTTP 访问

默认情况下，Docker 要求仓库使用 HTTPS。需要配置让 Docker 信任 HTTP 仓库。

编辑 `/etc/docker/daemon.json`：

```json
{
  "insecure-registries": ["192.168.1.200:5000"]
}
```

然后重启 Docker：

```bash
systemctl restart docker
```

### 推送镜像

```bash
# 为镜像打标签
docker tag myapp:latest 192.168.1.200:5000/myapp:latest

# 推送到私有仓库
docker push 192.168.1.200:5000/myapp:latest
```

### 拉取镜像

```bash
# 从私有仓库拉取
docker pull 192.168.1.200:5000/myapp:latest
```

### 查看仓库内容

```
# 查看仓库列表
curl http://192.168.1.200:5000/v2/_catalog

# 查看镜像标签
curl http://192.168.1.200:5000/v2/myapp/tags/list
```

---

## 常用服务部署

### MySQL

```bash
# 单机 MySQL
docker run -d \
  --name mysql57 \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root \
  -v /data/mysql/data:/var/lib/mysql \
  -v /data/mysql/conf:/etc/mysql/conf.d \
  mysql:5.7
```

**docker-compose.yml：**

```yaml
version: '3.8'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql57
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      TZ: Asia/Shanghai
    volumes:
      - ./data:/var/lib/mysql
      - ./conf:/etc/mysql/conf.d
      - /etc/localtime:/etc/localtime:ro
    restart: always
```

> **注意**：MySQL 5.7 配置文件目录为 `/etc/mysql/conf.d`，5.6 为 `/etc/mysql/my.cnf`。

### PostgreSQL

```bash
docker run -d \
  --name postgres \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=root \
  -e POSTGRES_USER=root \
  -v /data/postgres/data:/var/lib/postgresql/data \
  postgres:14
```

**docker-compose.yml：**

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:14
    container_name: postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_PASSWORD: root
      POSTGRES_USER: root
      TZ: Asia/Shanghai
    volumes:
      - ./data:/var/lib/postgresql/data
      - /etc/localtime:/etc/localtime:ro
    restart: always
    shm_size: 128mb
```

### Redis

```bash
# 基础运行
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis:7-alpine

# 带持久化
docker run -d \
  --name redis \
  -p 6379:6379 \
  -v /data/redis/data:/data \
  redis:7-alpine redis-server --appendonly yes

# 带密码
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis:7-alpine redis-server --requirepass mypassword
```

**docker-compose.yml：**

```yaml
version: '3.8'
services:
  redis:
    image: redis:7-alpine
    container_name: redis
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes --requirepass mypassword
    volumes:
      - ./data:/data
      - /etc/localtime:/etc/localtime:ro
    environment:
      TZ: Asia/Shanghai
    restart: always
```

### RocketMQ

```yaml
version: '3.8'
services:
  namesrv:
    image: apache/rocketmq:5.1.0
    container_name: rmqnamesrv
    ports:
      - "9876:9876"
    restart: always
    volumes:
      - ./namesrv/logs:/home/rocketmq/logs
    environment:
      - MAX_HEAP_SIZE=256M
      - HEAP_NEWSIZE=128M
    command: ["sh","mqnamesrv"]

  broker:
    image: apache/rocketmq:5.1.0
    container_name: rmqbroker
    ports:
      - "10909:10909"
      - "10911:10911"
    restart: always
    volumes:
      - ./broker/logs:/home/rocketmq/logs
      - ./broker/store:/home/rocketmq/store
      - ./broker/conf/broker.conf:/home/rocketmq/broker.conf
    environment:
      - NAMESRV_ADDR=namesrv:9876
      - MAX_HEAP_SIZE=512M
      - HEAP_NEWSIZE=256M
    command: ["sh","mqbroker","-c","/home/rocketmq/broker.conf"]
    depends_on:
      - namesrv

  dashboard:
    image: apacherocketmq/rocketmq-dashboard:latest
    container_name: rmqdashboard
    ports:
      - "8080:8080"
    restart: always
    environment:
      - JAVA_OPTS=-Xmx256M -Xms256M -Xmn128M -Drocketmq.namesrv.addr=namesrv:9876
    depends_on:
      - namesrv
```

**broker.conf 配置：**

```properties
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
brokerIP1 = 192.168.1.100
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
deleteWhen = 04
fileReservedTime = 72
autoCreateTopicEnable=true
autoCreateSubscriptionGroup=true
```

### APISIX

```yaml
version: '3'
services:
  apisix:
    image: apache/apisix:3.11.0-debian
    container_name: apisix
    ports:
      - "9180:9180"
      - "9080:9080"
      - "9443:9443"
    volumes:
      - ./apisix/conf/config.yaml:/usr/local/apisix/conf/config.yaml:ro
    depends_on:
      - etcd
    networks:
      apisix:

  etcd:
    image: bitnami/etcd:3.5.11
    container_name: etcd
    ports:
      - "2379:2379"
    volumes:
      - etcd_data:/bitnami/etcd
    environment:
      ETCD_ENABLE_V2: "true"
      ALLOW_NONE_AUTHENTICATION: "yes"
    networks:
      apisix:

networks:
  apisix:
    driver: bridge

volumes:
  etcd_data:
    driver: local
```

### Java 应用

```yaml
version: '3.8'
services:
  java-app:
    image: openjdk:17-slim
    container_name: java-app
    ports:
      - "8080:8080"
    environment:
      - TZ=Asia/Shanghai
      - SPRING_PROFILES_ACTIVE=prod
    volumes:
      - ./app.jar:/app/app.jar:ro
      - /etc/localtime:/etc/localtime:ro
    logging:
      driver: "json-file"
      options:
        max-size: "1g"
    restart: always
    command: java -server -Xms512m -Xmx1024m -jar /app/app.jar
```

### Tomcat

```yaml
version: '3.8'
services:
  tomcat:
    image: tomcat:9-jdk17
    container_name: tomcat
    ports:
      - "8080:8080"
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./webapps:/usr/local/tomcat/webapps
      - ./logs:/usr/local/tomcat/logs
      - /etc/localtime:/etc/localtime:ro
    logging:
      driver: "json-file"
      options:
        max-size: "500m"
    restart: always
```

---

## 性能与运维

### 时区配置

#### 方式一：环境变量

```yaml
environment:
  - TZ=Asia/Shanghai
```

#### 方式二：挂载时区文件

```yaml
volumes:
  - /etc/localtime:/etc/localtime:ro
```

#### 方式三：Dockerfile 中配置

```dockerfile
RUN echo "Asia/Shanghai" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata
```

### 日志配置

#### 容器级别配置

```yaml
logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"
```

#### 全局配置（daemon.json）

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

#### 日志驱动类型

| 驱动 | 说明 |
|------|------|
| `json-file` | JSON 格式（默认） |
| `syslog` | 写入 syslog |
| `journald` | 写入 journald |
| `gelf` | 写入 Graylog |
| `fluentd` | 写入 Fluentd |
| `awslogs` | 写入 AWS CloudWatch |
| `splunk` | 写入 Splunk |
| `none` | 禁用日志 |

### 资源限制

```yaml
services:
  app:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### 容器目录迁移

当 `/var/lib/docker` 空间不足时，可以迁移到其他目录：

```bash
# 1. 停止 Docker 服务
sudo systemctl stop docker

# 2. 迁移目录
sudo mv /var/lib/docker /data/docker

# 3. 创建软链接
sudo ln -s /data/docker /var/lib/docker

# 4. 启动 Docker
sudo systemctl start docker
```

> **注意**：如果使用软链接方式，配置文件中不需要额外修改。如果不适用软链接，需要在 `docker.service` 中添加 `--graph` 参数。

---

## 常见问题

### 容器启动失败

**可能原因：**
- 端口被占用
- 权限问题
- 配置文件错误

**排查方法：**

```bash
# 查看容器日志
docker logs mycontainer

# 查看详细错误信息
docker inspect mycontainer

# 检查端口占用
netstat -tlnp | grep 8080
```

### 进入容器后闪退

有些容器的默认 shell 不是 bash，可能是 sh：

```bash
# 尝试使用 sh
docker exec -it mycontainer /bin/sh
```

### 无法连接容器

```bash
# 检查容器网络
docker inspect mycontainer | grep -A 20 Networks

# 检查网络连通性
docker exec -it mycontainer ping -c 3 8.8.8.8
```

### 镜像拉取失败

```bash
# 检查网络
curl -I https://registry-1.docker.io/v2/

# 检查镜像加速器
docker info | grep "Registry Mirrors"

# 手动拉取
docker pull library/nginx:latest
```

### 磁盘空间不足

```bash
# 清理已停止的容器
docker container prune

# 清理悬空镜像
docker image prune

# 清理所有未使用的镜像、容器、网络
docker system prune -a

# 查看磁盘使用
docker system df
```

### 容器内无法解析域名

```bash
# 检查 DNS 配置
docker inspect mycontainer | grep -A 10 DNS

# 重新配置 DNS
docker run --dns 8.8.8.8 myimage
```

### 修改容器时区后未生效

```yaml
# 需要同时设置环境变量和挂载时区文件
environment:
  - TZ=Asia/Shanghai
volumes:
  - /etc/localtime:/etc/localtime:ro
```

---

## 参考资料

- [Docker 官方文档](https://docs.docker.com/)
- [Docker Compose 官方文档](https://docs.docker.com/compose/)
- [Dockerfile 最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)