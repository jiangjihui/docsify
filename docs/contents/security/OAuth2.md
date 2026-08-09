# OAuth2 协议详解

> 文章来源：综合自 RFC 6749（The OAuth 2.0 Authorization Framework）、OAuth 2.1 草案、OAuth 官方文档（oauth.net）及公开技术博客，部分由 AI 整理。

OAuth2 是目前最主流的**授权（Authorization）**开放标准。它解决的核心问题是：**在不把用户密码交给第三方应用的前提下，让第三方应用获得有限的、可撤销的访问用户资源的权限**。本文从角色、概念、模式、令牌机制到接入实践，系统性讲清 OAuth2。

---

## 一、概述

设想一个场景：你用「微信」账号登录某个第三方网站（如某小游戏）。如果网站直接索要你的微信密码，会带来巨大风险——网站可以拿你的密码做任意操作，且你无法单独吊销它对微信的访问。OAuth2 就是为这类「委托授权」场景而生的协议。

需要明确的两个易混概念：

- **认证（Authentication, AuthN）**：「你是谁」——验证用户身份。
- **授权（Authorization, AuthZ）**：「你能做什么」——授予应用访问资源的权限。

> **说明**：OAuth2 本质上是**授权框架**，本身不解决「用户是谁」（认证）。在需要身份认证时，通常叠加 **OpenID Connect（OIDC）**，见本文第八节。

---

## 二、核心角色

OAuth2 定义了四个角色，理解它们是理解所有授权流程的基础。

| 角色 | 英文 | 职责 |
| --- | --- | --- |
| 资源所有者 | Resource Owner | 拥有受保护资源的用户（人），能够授予对资源的访问权限 |
| 客户端 | Client | 代表资源所有者请求访问的**第三方应用**（Web 站、App、后端服务等） |
| 授权服务器 | Authorization Server | 负责认证资源所有者、获得其同意，并**签发令牌**的服务器（如微信开放平台、Keycloak） |
| 资源服务器 | Resource Server | 托管受保护资源的服务器，凭令牌**校验并响应**资源请求（如微信的用户信息接口） |

> **图题：OAuth2 四角色关系**：客户端代表资源所有者向授权服务器申请令牌，再凭令牌到资源服务器获取受保护资源。

```mermaid
flowchart LR
    RO[资源所有者<br/>Resource Owner<br/>用户] -->|授权| C[客户端<br/>Client<br/>第三方应用]
    C -->|请求授权| AS[授权服务器<br/>Authorization Server<br/>签发令牌]
    AS -->|返回令牌| C
    C -->|携带令牌访问| RS[资源服务器<br/>Resource Server<br/>提供资源]
    AS -. 可托管用户身份 .-> RS
```

---

## 三、核心概念

### 3.1 令牌（Token）

- **访问令牌（Access Token）**：用于访问资源服务器的短期凭证，通常有效期很短（几分钟到几小时）。
- **刷新令牌（Refresh Token）**：用于获取新的访问令牌的长期凭证，有效期长，且**只发给授权服务器**，不随每个资源请求发送。

> **说明**：访问令牌短期、刷新令牌长期，是「降低令牌泄露危害」与「避免频繁登录」之间的平衡设计。详见第五节。

### 3.2 关键参数

| 参数 | 作用 |
| --- | --- |
| `client_id` / `client_secret` | 客户端的身份标识与密钥；保密客户端（如后端服务）用 `secret` 证明自己，公共客户端（如前端 SPA）无 `secret` |
| `scope` | 申请的**权限范围**（如 `read:profile`、`write:repo`），遵循最小权限原则 |
| `state` | 客户端生成的随机值，授权回调时原样返回，用于**防 CSRF / 防钓鱼** |
| `redirect_uri` | 授权完成后回调的地址，授权服务器会**严格校验**其合法性 |

### 3.3 三类端点

| 端点 | 用途 |
| --- | --- |
| 授权端点（Authorization Endpoint） | 展示登录与同意页，用户在浏览器中完成授权 |
| 令牌端点（Token Endpoint） | 客户端用授权凭证（如 code）换取令牌 |
| 重定向端点（Redirection Endpoint） | 授权服务器把授权结果（code 或 token）回传给客户端 |

---

## 四、四种授权模式

OAuth2 定义了多种「授权许可（Grant）」类型，对应不同场景。其中两种是**现代推荐/常用**的，两种在 **OAuth 2.1 中已被弃用**（仅作历史了解）。

### 4.1 授权码模式（Authorization Code）+ PKCE ⭐

**最常用、最安全**的模式，适用于有后端的 Web 应用、移动端、单页应用。核心思想：先拿一个一次性的「授权码（code）」，再在后端用 code 换令牌，避免令牌直接暴露在浏览器/前端。

为防范公共客户端（无法安全存储 `client_secret`）被拦截授权码，**强烈建议搭配 PKCE（Proof Key for Code Exchange）**。

> **图题：授权码模式（含 PKCE）时序**：用户先被引导到授权端点，授权后拿到 code；客户端后端用 code + `code_verifier` 向令牌端点换取令牌。

```mermaid
sequenceDiagram
    autonumber
    participant U as 资源所有者(用户)
    participant B as 客户端(浏览器/App)
    participant S as 客户端后端
    participant AS as 授权服务器
    participant RS as 资源服务器
    B->>U: 点击「第三方登录」
    B->>AS: 重定向到授权端点(带 code_challenge, state, scope, redirect_uri)
    U->>AS: 登录并点击「同意授权」
    AS-->>B: 重定向回 redirect_uri, 携带 authorization_code + state
    B->>S: 把 code 交给后端
    S->>AS: 用 code + code_verifier 请求令牌端点
    AS-->>S: 返回 access_token + refresh_token
    S->>RS: 携带 Bearer access_token 访问资源
    RS-->>S: 返回受保护资源
    S-->>B: 展示结果
```

**① 引导用户到授权端点**

```
https://auth.example.com/oauth2/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=https://client.example.com/callback&
  scope=read:profile&
  state=xyzABC&
  code_challenge=BASE64URL(SHA256(verifier))&
  code_challenge_method=S256
```

**② 后端用 code 换取令牌**

```bash
curl -X POST https://auth.example.com/oauth2/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "code=AUTH_CODE_HERE" \
  -d "redirect_uri=https://client.example.com/callback" \
  -d "client_id=CLIENT_ID" \
  -d "code_verifier=CODE_VERIFIER_HERE"
```

**③ 令牌端点返回**

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "v2.DEF456...",
  "scope": "read:profile"
}
```

### 4.2 客户端凭证模式（Client Credentials）

适用于**没有用户参与**的场景：后端服务（机器）直接以自己的身份访问另一个服务/API。客户端用自身的 `client_id` + `client_secret` 换令牌，**不涉及资源所有者**，因此**没有刷新令牌**（需要时重新申请即可）。

> **图题：客户端凭证模式时序**：客户端用自身凭据向授权服务器换取令牌，再调用资源服务器。

```mermaid
sequenceDiagram
    autonumber
    participant C as 客户端(后端服务)
    participant AS as 授权服务器
    participant RS as 资源服务器
    C->>AS: 用 client_id+client_secret 请求令牌(grant_type=client_credentials)
    AS-->>C: 返回 access_token
    C->>RS: 携带 Bearer access_token 调用 API
    RS-->>C: 返回结果
```

```bash
curl -X POST https://auth.example.com/oauth2/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -u "CLIENT_ID:CLIENT_SECRET" \
  -d "grant_type=client_credentials" \
  -d "scope=service:read"
```

#### 授权码模式 vs 客户端凭证模式（核心对比）

前两种是现代系统的主线模式，也最容易被混淆。一句话区分：**客户端凭证模式解决「你是谁（应用身份）」，授权码模式解决「你是谁 + 用户是谁、用户同意了哪些权限（用户授权）」**。

| 对比维度 | 客户端凭证模式（client_credentials） | 授权码模式（authorization_code） |
| --- | --- | --- |
| 谁在授权 | **客户端自己**（用 client_id / client_secret 换 token） | **资源所有者（用户）**（浏览器登录后授权） |
| 用户在场 | 没有，纯机器对机器（M2M） | 必须有，用户在浏览器中完成登录与同意 |
| 令牌代表的身份 | 代表**应用本身**（应用级） | 代表**用户**（用户级，携带 scope 范围） |
| token 是否经过浏览器 | 否，全程在后端 | 否，token 经后端 code 交换获取，不经过浏览器地址栏 |
| 典型场景 | 服务间调用、定时任务、开放平台聚合 | 第三方应用替用户访问资源（如微信 / GitHub 登录） |
| 安全前提 | client_secret 不得泄露（仅服务端持有） | 重定向 URI 校验 + state 防 CSRF + PKCE |

> **说明**：两种模式都不会把 token 暴露在浏览器地址栏——区别只在于「是否涉及真人用户授权」。没有用户、只需服务身份时用客户端凭证；需要代表某个具体用户操作时用授权码。

### 4.3 密码模式（Resource Owner Password Credentials）—— ⚠️ OAuth 2.1 已弃用

客户端直接收集用户用户名和密码，向授权服务器换令牌。**要求用户把密码交给客户端**，违背 OAuth2「不暴露密码」的初衷，仅在高度信任的第一方场景中曾经使用。**OAuth 2.1 已正式弃用，不应在新系统中使用。**

### 4.4 隐式模式（Implicit）—— ⚠️ OAuth 2.1 已弃用

早期为无后端的前端应用设计，授权服务器直接通过重定向把 `access_token` 返回到浏览器 URL 片段（fragment）。令牌暴露在浏览器历史和地址栏中，安全性差。**已被「授权码 + PKCE」全面取代，OAuth 2.1 已弃用。**

---

## 五、令牌与刷新机制

访问令牌通常短期有效（如 1 小时）。到期后如何避免用户重新登录？答案是**刷新令牌**。

> **图题：令牌刷新时序**：访问令牌过期 → 客户端用 refresh_token 向授权服务器换取新令牌 → 用新令牌重试。

```mermaid
sequenceDiagram
    autonumber
    participant C as 客户端
    participant AS as 授权服务器
    participant RS as 资源服务器
    C->>RS: 携带 access_token 访问资源
    RS-->>C: 401 令牌已过期
    C->>AS: 用 refresh_token 请求新令牌(grant_type=refresh_token)
    AS-->>C: 返回新的 access_token(可能含新的 refresh_token)
    C->>RS: 用新 access_token 重试
    RS-->>C: 返回受保护资源
```

刷新令牌请求示例：

```bash
curl -X POST https://auth.example.com/oauth2/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=refresh_token" \
  -d "refresh_token=REFRESH_TOKEN_HERE" \
  -d "client_id=CLIENT_ID"
```

> **说明（刷新令牌轮换）**：为提升安全性，许多授权服务器在刷新时会**同时签发新的 refresh_token 并使旧令牌失效**（Rotation）。一旦检测到同一个 refresh_token 被重复使用，服务器会判定令牌可能泄露并吊销整条授权链。

### JWT 与「不透明令牌」

令牌本身有两种形态，OAuth2 并不限定，但实践中二选一：

| 维度 | JWT（自包含令牌） | 不透明令牌（Opaque Token） |
| --- | --- | --- |
| 结构 | 三段式（头部.载荷.签名），可本地解析出 claims | 一串随机字符串，本身无意义 |
| 校验方式 | 资源服务器用授权服务器公钥**本地验签** | 需调用授权服务器 `introspection` 端点查询有效性 |
| 吊销难度 | 较难（靠短期有效期 / 黑名单） | 容易（服务器端直接失效） |
| 性能 | 高（无额外网络往返） | 较低（每次需 introspect） |

> **JWT 详解与示例见同组文档 [JWT](JWT.md)**。OAuth2 的 access_token 既可以是 JWT，也可以是 Opaque Token。

---

## 六、典型应用场景

| 场景 | 推荐模式 | 说明 |
| --- | --- | --- |
| 第三方登录（微信 / GitHub / Google） | 授权码 + PKCE | 用户用既有账号授权新应用，最常见 |
| 后端服务调用开放 API（无用户） | 客户端凭证 | 服务对服务的机器访问 |
| 第一方移动端 / 单页应用 | 授权码 + PKCE | 公共客户端无 `secret`，PKCE 必备 |
| 微服务间服务调用 | 客户端凭证 | 网关或内部服务以自身身份获取令牌 |
| 遗留高度信任内部系统 | （历史）密码模式 | 不推荐，已弃用 |

除上述「第三方登录」「服务间调用」两大主线外，工程上还有一种常见但有别于标准授权码模式的场景：**登录态传递（免登）**，即受信的合作方系统替用户声明身份、让用户免登进入平台页面，在实践中常用「**服务端代理变体**」实现：中间层以服务端身份向内部 SSO 换取登录态，并把登录态以 URL 参数（`?usertoken=xxx`）方式拼接到 302 重定向地址上，从而可兼容 iframe 场景。

> 详细动机、信任模型对比、网关 + 多环节协作实现流程与风险见独立文档：[免登的服务端代理变体（实践）](../../实践/安全认证/免登服务端代理变体.md)。

---

## 七、接入方式（实践要点）

### 7.1 资源服务器如何校验请求

无论哪种模式，客户端最终都在请求头中携带令牌：

```http
GET /api/resource HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

资源服务器（或统一入口的网关）的职责：**校验令牌签名/有效性、解析 scope、将用户身份透传下游**。

### 7.2 与本站文档的衔接

- **Spring 集成**：在 Spring 生态中作为 OAuth2 客户端对接第三方登录、或将 JWT 作为资源服务器无状态认证，见 [Spring Security](../../contents/spring/SpringSecurity.md) 的「OAuth2 / JWT」小节。
- **网关统一校验**：在 API 网关层收敛认证逻辑、校验 JWT/OAuth2 令牌并透传身份信息，见 [API 网关](../../contents/middleware/API网关.md) 的「认证与授权」小节。

> **说明**：协议原理（本文）与框架落地（Spring / 网关）应分层阅读——先理解四角色与授权流程，再看具体代码如何对接。

---

## 八、OAuth 2.1 与 OIDC 延伸

### 8.1 OAuth 2.1

OAuth 2.1 是对 OAuth2 的整合与收紧，主要变化：

- **正式弃用**密码模式与隐式模式；
- **强制**公共客户端使用 **PKCE**；
- 强调 `redirect_uri` 精确匹配、刷新令牌轮换等最佳实践。

> 新项目应直接按 **OAuth 2.1** 的规范实现（即「授权码 + PKCE」「客户端凭证」两条主线）。

### 8.2 OpenID Connect（OIDC）

OAuth2 只管「授权」，不管「用户是谁」。**OIDC 在 OAuth2 之上叠加一层身份认证**，额外签发一个 **ID Token（JWT）**，其中包含用户身份声明（sub、name、email 等）。当你需要「用第三方账号登录并知道是谁」时，用的其实是 OIDC（如微信登录、Google 登录的底层）。

---

## 九、安全实践与常见风险

| 风险点 | 防护手段 |
| --- | --- |
| 授权码被拦截 | 公共客户端使用 **PKCE**；全程 HTTPS |
| CSRF / 钓鱼 | 授权请求携带并校验 `state` |
| 重定向 URI 被篡改 | 授权服务器**精确校验**注册的 `redirect_uri` |
| 令牌泄露长期可用 | 访问令牌短期有效；刷新令牌支持吊销与轮换 |
| 前端明文存令牌 | 避免 `localStorage` 明文存储；优先 httpOnly Cookie 或内存，配合 CSRF 防护 |
| 权限过大 | `scope` 遵循**最小权限**原则 |
| 中间人窃听 | 全链路 **HTTPS**，禁止明文传输令牌 |

> **延伸阅读**：Web 攻击与防护（注入、XSS、CSRF 等）见同组文档 [安全防护](安全防护.md)。

---

## 小结

- OAuth2 是**授权框架**，通过「客户端代表用户向授权服务器申请令牌、凭令牌访问资源服务器」实现安全的委托访问。
- 四个角色：**资源所有者、客户端、授权服务器、资源服务器**。
- 现代主线两条：**授权码 + PKCE**（有人参与）、**客户端凭证**（服务间）。密码/隐式模式已弃用。
- 令牌机制：短期 access token + 长期 refresh token，JWT 或 Opaque 二选一。
- 需要「认证用户身份」时叠加 **OIDC**。
- 相关：[JWT](JWT.md) · [Spring Security](../../contents/spring/SpringSecurity.md) · [API 网关](../../contents/middleware/API网关.md) · [安全防护](安全防护.md)
