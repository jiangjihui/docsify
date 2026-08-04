# Nginx

Nginx ("engine x") 是一个高性能的 HTTP 和反向代理服务器，也是一个 IMAP/POP3/SMTP 代理服务器。

## 概述

### Nginx简介

Nginx具有以下特点：

- **热部署** - master管理进程与worker工作进程分离，支持在不停止服务的情况下升级、重载配置
- **高并发** - 理论上支持高达10万并发连接，取决于内存配置
- **低内存消耗** - 10000个非活跃HTTP Keep-Alive连接仅消耗2.5M内存
- **响应快** - 单次请求响应快，高峰期性能优于其他Web服务器
- **高可靠性** - 核心框架代码优秀，模块稳定，宕机概率极低

### 核心功能

1. **反向代理** - 代理内部服务器，保护内网安全
2. **正向代理** - 代理客户端访问外部资源
3. **负载均衡** - 分发请求到多个后端服务器
4. **HTTP服务器** - 支持静态资源和动静分离

---

## 安装部署

### CentOS/Yum安装

```bash
# 安装EPEL仓库
sudo yum install epel-release

# 安装Nginx
sudo yum install nginx

# 启动服务
sudo systemctl start nginx

# 停止服务
sudo systemctl stop nginx

# 开机自启
sudo systemctl enable nginx

# 查看状态
sudo systemctl status nginx
```

### Ubuntu/Apt安装

```bash
sudo apt-get install nginx

# 相关路径
/usr/sbin/nginx        # 主程序
/etc/nginx             # 配置文件
/usr/share/nginx       # 静态文件
/var/log/nginx         # 日志文件
```

### 编译安装

```bash
# 安装依赖
yum install -y gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel

# 下载并解压
cd /usr/local/
mkdir nginx && cd nginx
wget -c https://nginx.org/download/nginx-1.24.0.tar.gz
tar -zxvf nginx-1.24.0.tar.gz
cd nginx-1.24.0

# 编译安装
./configure --with-http_ssl_module
make && make install

# 启动Nginx
/usr/local/nginx/sbin/nginx

# 测试
curl localhost
```

### 目录结构

```
/etc/nginx/
├── nginx.conf          # 主配置文件
├── conf.d/             # 子配置文件目录
├── sites-enabled/     # 启用的站点
├── sites-available/    # 可用的站点
├── modules/            # 模块目录
└── mime.types          # MIME类型映射
```

---

## 基础命令

```bash
# 启动Nginx
nginx

# 停止Nginx（快速停止）
nginx -s stop

# 优雅停止（处理完当前请求）
nginx -s quit

# 重新加载配置
nginx -s reload

# 重新打开日志文件
nginx -s reopen

# 测试配置文件
nginx -t

# 指定配置文件
nginx -c /etc/nginx/nginx.conf

# 查看版本信息
nginx -v
nginx -V  # 详细版本信息
```

---

## 进程模型

### Master-Worker模式

Nginx启动后会有一个master进程和多个worker进程：

**Master进程**
- 接收外界信号，向worker进程发送信号
- 监控worker进程运行状态
- 负责配置重载、热部署、日志切换

```bash
# 优雅重启
kill -HUP `cat /var/run/nginx.pid`
nginx -s reload

# 优雅停止
nginx -s quit
```

**Worker进程**
- 处理具体的网络事件
- 多个worker进程平等竞争请求
- 个数通常设置为CPU核心数

### 工作原理

```
         请求
           │
           ▼
    ┌─────────────┐
    │ Master进程  │  (管理进程，不处理请求)
    └──────┬──────┘
           │ fork
           ▼
    ┌─────────────┐
    │ Worker进程1 │ ──┐
    └─────────────┘   │
    ┌─────────────┐   │ accept_mutex
    │ Worker进程2 │ ──┼──► 竞争处理请求
    └─────────────┘   │
    ┌─────────────┐   │
    │ Worker进程n │ ──┘
    └─────────────┘
```

### 异步非阻塞

Nginx采用异步非阻塞方式处理请求，一个worker可以同时处理成千上万个请求：

- **阻塞调用** - 等待事件完成，CPU空闲
- **非阻塞调用** - 立即返回，事件未准备好时稍后重试

Nginx通过epoll等事件机制实现高效的事件处理，相比Apache的进程/线程模型更节省资源。

---

## 正反向代理

### 正向代理

正向代理代理客户端，替客户端访问外部资源。

**典型场景：**
- 访问无法直接访问的资源（如翻墙）
- 加速访问（缓存代理）
- 上网认证
- 上网行为管理

```nginx
server {
    listen 8080;
    resolver 8.8.8.8;

    location / {
        proxy_pass $scheme://$host$request_uri;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 反向代理

反向代理代理服务端，客户端不知道实际提供服务的服务器。

**作用：**
- 保护内网安全
- 负载均衡
- 隐藏真实服务器

```nginx
upstream backend {
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 正反向代理对比

| 特性 | 正向代理 | 反向代理 |
|------|----------|----------|
| 代理对象 | 客户端 | 服务端 |
| 客户端感知 | 感知不到代理存在 | 认为是真实服务器 |
| 典型用途 | 翻墙、加速 | 负载均衡、安全防护 |

---

## 负载均衡

Nginx支持多种负载均衡策略：

### 轮询（Round Robin）

默认方式，每个请求按时间顺序分配到不同服务器：

```nginx
upstream backend {
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
}
```

### 权重（Weight）

指定轮询权重，用于服务器性能不均的情况：

```nginx
upstream backend {
    server 127.0.0.1:8080 weight=9;
    server 127.0.0.1:8081 weight=1;
}
```

### IP哈希（IP Hash）

同一客户端固定访问同一服务器，解决session问题：

```nginx
upstream backend {
    ip_hash;
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
}
```

### 最少连接（Least Connections）

连接数最少的服务器优先分配：

```nginx
upstream backend {
    least_conn;
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
}
```

### 第三方策略

**fair** - 按响应时间分配：
```nginx
upstream backend {
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
    fair;
}
```

**url_hash** - 按URL哈希分配，适合缓存场景：
```nginx
upstream backend {
    hash $request_uri;
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
}
```

### 后端服务器状态

```nginx
upstream backend {
    server 127.0.0.1:8080 max_fails=3 fail_timeout=10s;
    server 127.0.0.1:8081 down;  # 暂时下线
    server 127.0.0.1:8082 backup;  # 备用服务器
}
```

| 参数 | 说明 |
|------|------|
| max_fails | 失败次数，超过则视为不可用 |
| fail_timeout | 失败超时时间 |
| down | 标记为永久下线 |
| backup | 备用服务器，其他都失败时启用 |

---

## 静态资源与动静分离

### 静态资源服务

```nginx
server {
    listen 80;
    server_name static.example.com;
    root /usr/share/nginx/html;

    location / {
        index index.html index.htm;
    }

    # 静态资源缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }
}
```

### 动静分离

将静态资源和动态请求分离到不同服务器：

```nginx
server {
    listen 80;
    server_name example.com;
    root /usr/share/nginx/html;

    # 静态资源直接返回
    location /static/ {
        alias /usr/share/nginx/static/;
    }

    # 图片资源缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        alias /usr/share/nginx/images/;
        expires 7d;
    }

    # 动态请求代理到后端
    location /api/ {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Vue/React History模式路由
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

## 缓存与压缩

### Gzip压缩

```nginx
http {
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";

    # 缓存配置
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=cache_zone:10m inactive=60m max_size=1g;

    server {
        # 使用缓存
        location / {
            proxy_cache cache_zone;
            proxy_cache_valid 200 60m;
            proxy_cache_key $scheme$host$request_uri;
            add_header X-Cache-Status $upstream_cache_status;
            proxy_pass http://backend;
        }
    }
}
```

### 代理缓存配置

```nginx
# 缓存配置
proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=api_cache:100m max_size=1g inactive=60m use_temp_path=off;

server {
    location /api/ {
        proxy_cache api_cache;
        proxy_cache_valid 200 304 10m;
        proxy_cache_valid any 1m;
        proxy_cache_key $scheme$host$request_uri;
        proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
        add_header X-Cache-Status $upstream_cache_status;

        proxy_pass http://backend;
    }
}
```

---

## 访问限流

### 请求频率限制

```nginx
http {
    # 限制每个IP每秒10个请求
    limit_req_zone $binary_remote_addr zone=req_perip:50m rate=10r/s;

    server {
        location /api/ {
            # 允许突发50个请求
            limit_req zone=req_perip burst=50 nodelay;
            limit_req_status 503;
            limit_conn_status 503;

            proxy_pass http://backend;
        }
    }
}
```

### 连接数限制

```nginx
http {
    # 限制每个IP最多10个连接
    limit_conn_zone $binary_remote_addr zone=conn_perip:10m;

    server {
        location / {
            limit_conn conn_perip 10;
            proxy_pass http://backend;
        }
    }
}
```

---

## SSL/Https配置

### 基础SSL配置

```nginx
server {
    listen 80;
    listen 443 ssl http2;
    server_name example.com;

    # SSL证书
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # SSL协议和加密套件
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers on;

    # 会话缓存
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;

    # HSTS（可选）
    add_header Strict-Transport-Security "max-age=31536000" always;
}
```

### HTTP重定向到HTTPS

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # 其他配置...
}
```

### SSL证书参数

| 参数 | 说明 |
|------|------|
| ssl_certificate | 证书文件（包含公钥） |
| ssl_certificate_key | 私钥文件 |
| ssl_protocols | 支持的SSL/TLS版本 |
| ssl_ciphers | 加密套件 |
| ssl_session_cache | 会话缓存 |

---

## 多项目与端口转发

### 多项目配置

```nginx
server {
    listen 8081;
    server_name _;

    # 主项目
    location / {
        root /usr/share/nginx/html/main;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # 子项目
    location /admin {
        alias /usr/share/nginx/html/admin;
        try_files $uri $uri/ /admin/index.html;
    }

    # API代理
    location /api {
        proxy_pass http://backend;
    }
}
```

### TCP端口转发

```nginx
# stream模块需要在nginx.conf最外层（不是http块内）
stream {
    # MongoDB转发
    upstream mongodb {
        hash $remote_addr;
        server 192.168.1.103:27017 max_fails=3 fail_timeout=10s;
    }

    server {
        listen 10009;
        proxy_connect_timeout 20s;
        proxy_timeout 5m;
        proxy_pass mongodb;
    }

    # MySQL转发
    upstream mysql_backend {
        server 192.168.1.104:3306;
    }

    server {
        listen 13306;
        proxy_pass mysql_backend;
    }
}
```

### WebSocket支持

```nginx
location /ws/ {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_read_timeout 3600s;
}
```

---

## 常见配置示例

### Vue/React History模式

```nginx
server {
    listen 80;
    server_name example.com;
    root /usr/share/nginx/html/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # 静态资源
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # API代理
    location /api/ {
        proxy_pass http://backend/api/;
        proxy_set_header Host $host;
    }
}
```

### 图片防盗链

```nginx
server {
    listen 80;
    server_name example.com;

    location ~* \.(jpg|jpeg|png|gif)$ {
        valid_referers none blocked example.com *.example.com;
        if ($invalid_referer) {
            return 403;
        }
    }
}
```

### 限制IP访问

```nginx
location /admin/ {
    allow 192.168.1.0/24;
    allow 10.0.0.0/8;
    deny all;

    proxy_pass http://backend;
}
```

### 请求超时配置

```nginx
location /api/ {
    proxy_connect_timeout 30s;
    proxy_send_timeout 30s;
    proxy_read_timeout 30s;
    proxy_pass http://backend;
}
```

### 日志配置

```nginx
http {
    # 访问日志
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    # 错误日志
    error_log /var/log/nginx/error.log warn;
}
```
