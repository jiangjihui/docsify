# HTTP

HTTP（HyperText Transfer Protocol）是分布式、协作式、超媒体信息系统的应用层协议，是现代互联网的基础协议。

## HTTP 状态码

服务器返回的响应报文中第一行为状态行，包含了状态码以及原因短语，用来告知客户端请求的结果。

| 状态码 |              类别              |          原因短语          |
| :----: | :----------------------------: | :------------------------: |
|  1XX   |  Informational(信息性状态码)   |     接收的请求正在处理     |
|  2XX   |      Success(成功状态码)       |      请求正常处理完毕      |
|  3XX   |   Redirection(重定向状态码)    | 需要进行附加操作以完成请求 |
|  4XX   | Client Error(客户端错误状态码) |     服务器无法处理请求     |
|  5XX   | Server Error(服务器错误状态码) |     服务器处理请求出错     |

### 常见状态码

| 状态码 | 说明 |
|--------|------|
| 200 | 请求成功 |
| 204 | 请求成功但无内容返回 |
| 301 | 永久重定向 |
| 302 | 临时重定向 |
| 304 | 资源未修改，使用缓存 |
| 400 | 请求语法错误 |
| 401 | 需要认证 |
| 403 | 服务器拒绝请求 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |
| 502 | 网关错误 |
| 503 | 服务不可用 |

## HTTP/1.1 特性

### 持久连接（Keep-Alive）

HTTP/1.1 默认使用持久连接，减少 TCP 连接建立的开销。

```
Connection: Keep-Alive
```

### 管道化（Pipeline）

允许在同一个 TCP 连接上发送多个请求，但响应必须按顺序返回。

### 分块传输编码

```
Transfer-Encoding: chunked
```

### 缓存控制

```
Cache-Control: max-age=3600, no-cache, no-store
```

## HTTP/2

HTTP/2 于 2015 年发布，在 HTTP/1.1 的基础上带来了重大改进。

### 二进制分帧

HTTP/2 使用二进制分帧层，将消息分解为更小的帧进行传输。

### 多路复用

在单个 TCP 连接上可以并行传输多个请求和响应，无需阻塞。

```
# HTTP/1.1 需要多个 TCP 连接
连接1: ----请求1---->
连接2: ----请求2---->
连接3: ----请求3---->

# HTTP/2 单连接多路复用
连接: >请求1
       >请求2
       >请求3
       <响应1
       <响应2
       <响应3
```

### 头部压缩

HTTP/2 使用 HPACK 算法压缩头部，减少传输开销。

```
# 静态表 + 动态表 + 哈夫曼编码
```

### 服务器推送

服务器可以主动向客户端推送资源，无需客户端显式请求。

```
# 服务器推送示例
Push-Promise: /style.css
Push-Promise: /script.js
```

### 流的优先级

客户端可以指定流的优先级，让服务器优先处理重要资源。

### HPACK 头部压缩

HPACK 是 HTTP/2 特有的头部压缩算法：

- **静态表**：常用的 61 个头部字段
- **动态表**：在连接过程中逐步构建
- **哈夫曼编码**：对字符串进行高效编码

## HTTP/3

HTTP/3 于 2022 年发布，基于 QUIC 协议（UDP）。

### QUIC 协议

QUIC（Quick UDP Internet Connections）是 HTTP/3 的底层传输协议：

- **基于 UDP**：避免 TCP 的队头阻塞问题
- **0-RTT 连接建立**：减少连接延迟
- **连接迁移**：网络切换时保持连接
- **内置 TLS 1.3**：加密与传输层整合

### 主要特性

1. **无队头阻塞**：各流独立传输，一个流丢包不影响其他流
2. **更快的连接建立**：0-RTT 和 1-RTT 握手
3. **连接迁移**：使用连接 ID 跨网络切换
4. **改进的拥塞控制**：可插拔的拥塞控制算法

### HTTP/3 与 HTTP/2 对比

| 特性 | HTTP/2 | HTTP/3 |
|------|--------|--------|
| 传输层 | TCP | QUIC (UDP) |
| 多路复用 | ✓ | ✓ |
| 头部压缩 | HPACK | QPACK |
| 队头阻塞 | TCP 队头阻塞 | 无 |
| 连接建立 | 2-RTT | 0/1-RTT |
| TLS | 独立 | 内置 |

## HTTPS 加密机制

HTTPs 采用混合加密机制，结合非对称加密和对称加密的优点。

### 加密流程

1. **客户端问候**：发送支持的加密算法列表和随机数
2. **服务器问候**：选择加密算法，发送证书和随机数
3. **证书验证**：客户端验证服务器证书
4. **密钥交换**：使用非对称加密交换预主密钥
5. **生成会话密钥**：双方使用预主密钥生成会话密钥
6. **加密通信**：使用会话密钥进行对称加密通信

### TLS 1.3 改进

TLS 1.3（2018 年发布）相比 TLS 1.2 有重大改进：

- **更快的握手**：1-RTT（TLS 1.2 需要 2-RTT）
- **0-RTT 模式**：对重连请求可实现 0-RTT
- **禁用不安全算法**：移除 MD5、RC4、3DES 等
- **简化密码套件**：仅保留 AEAD 加密算法
- **前向安全性**：使用 ECDHE 密钥交换

```
# TLS 1.3 密码套件
TLS_AES_256_GCM_SHA384
TLS_CHACHA20_POLY1305_SHA256
TLS_AES_128_GCM_SHA256
```

## Cookie 和 Session

### Cookie

Cookie 是服务器发送到用户浏览器并保存在本地的一小块数据。

```
# 服务器设置 Cookie
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

**Cookie 属性**：

- `HttpOnly`：禁止 JavaScript 访问
- `Secure`：仅 HTTPS 传输
- `SameSite`：防止 CSRF 攻击
  - `Strict`：完全禁止跨站 Cookie
  - `Lax`：允许部分 GET 请求携带
  - `None`：允许跨站请求（需 Secure）

### Session

Session 存储在服务器端，安全性更高。

```
# 使用 Session 的流程
1. 客户端发送登录请求
2. 服务器验证后创建 Session，存储在 Redis
3. 服务器返回 Session ID（通过 Cookie）
4. 客户端后续请求携带 Session ID
5. 服务器根据 Session ID 获取用户信息
```

## 缓存机制

### 强缓存

```
# Expires（HTTP/1.1 之前）
Expires: Wed, 21 Oct 2025 07:28:00 GMT

# Cache-Control（HTTP/1.1）
Cache-Control: max-age=3600, public
```

### 协商缓存

```
# Last-Modified / If-Modified-Since
Last-Modified: Wed, 21 Oct 2025 07:28:00 GMT
If-Modified-Since: Wed, 21 Oct 2025 07:28:00 GMT

# ETag / If-None-Match
ETag: "abc123"
If-None-Match: "abc123"
```

### 缓存层级

```
浏览器缓存
    ↓ (未命中)
Service Worker 缓存
    ↓ (未命中)
CDN 缓存
    ↓ (未命中)
源服务器缓存
    ↓ (未命中)
数据库/后端服务
```

## CORS 跨域资源共享

### 简单请求

```
# 请求
Origin: https://example.com

# 响应
Access-Control-Allow-Origin: https://example.com
```

### 预检请求

```
# 预检请求
OPTIONS /api HTTP/1.1
Origin: https://example.com
Access-Control-Request-Method: PUT

# 预检响应
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```

## HTTP 方法

| 方法 | 说明 | 幂等 |
|------|------|------|
| GET | 获取资源 | ✓ |
| POST | 创建资源 | ✗ |
| PUT | 更新资源（整体） | ✓ |
| PATCH | 更新资源（部分） | ✗ |
| DELETE | 删除资源 | ✓ |
| HEAD | 获取头部信息 | ✓ |
| OPTIONS | 获取支持的 HTTP 方法 | ✓ |

## 常见安全问题

### CSRF 攻击

跨站请求伪造，利用用户已登录的身份发起恶意请求。

**防御措施**：
- 使用 SameSite Cookie
- CSRF Token
- 验证 Referer 或 Origin 头

### XSS 攻击

跨站脚本攻击，在页面注入恶意脚本。

**防御措施**：
- 输出编码
- HTTPOnly Cookie
- Content Security Policy (CSP)

### 中间人攻击

攻击者拦截并篡改通信内容。

**防御措施**：
- 使用 HTTPS
- 证书固定（Certificate Pinning）
- HSTS（HTTP Strict Transport Security）
