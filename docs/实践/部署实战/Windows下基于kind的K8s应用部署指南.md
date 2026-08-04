# Windows + Docker Desktop 下基于 kind 的 K8s 应用部署指南

> 从零到一部署一套 K8s 应用环境：kind 集群 + KubePi 管理面板 + Jenkins CI/CD + 前后端应用。**按步骤操作即可部署成功**，所有配置为实测通过的完整文件，可直接复制使用。全程实测可行。

> 📦 **验证示例项目**：本文所有命令/清单对应的真实项目是公开仓库 **https://github.com/jiangjihui/online-ordering**（Spring Boot + Vue 的点餐系统）。验证部署时直接 `git clone` 它作为被测项目即可；替换为自己的项目时，只需改动镜像名（`online-ordering`）、命名空间与清单路径。

## 1. 部署架构

```
浏览器 http://localhost:80 ──> Ingress (nginx) ──> frontend Pod（页面）
                                        └───────> backend Pod（/api /ws /images，SQLite 挂 PVC）
                                                   ↑ kind 集群内，KubePi 统一管理

Jenkins（宿主机 Docker）→ docker push → 本地 registry:5000 → kind 节点 mirror 拉取
                    └──> kubectl apply / rollout → 部署到 kind 集群
```

| 组件 | 端口 | 说明 |
|------|------|------|
| kind 集群 local-dev | API 随机 | 单节点 control-plane，80/443 映射给 Ingress |
| 本地镜像仓库 registry:2 | 5000 | 宿主机，kind 节点经 containerd mirror 拉镜像 |
| ingress-nginx | 80/443 | 集群统一入口，按路径路由前后端 |
| KubePi | 30080 | 集群管理面板（镜像 `1panel/kubepi:v2.0.0`） |
| Jenkins | 8888 | CI/CD，push 自动触发构建（Poll SCM 每 2 分钟） |

## 2. 前置条件

安装并验证：

```bash
winget install Kubernetes.kind
winget install Kubernetes.kubectl

docker version      # Docker daemon 正常
kind version
kubectl version
```

准备被测项目（验证用公开仓库）：

```bash
git clone https://github.com/jiangjihui/online-ordering
cd online-ordering
```

## 3. 创建 kind 集群

**第1步**，编写 kind-config.yaml（完整文件，含 80/443 端口映射 + containerd mirror）：

```yaml
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: local-dev
nodes:
  - role: control-plane
    extraPortMappings:
      - { containerPort: 80, hostPort: 80, protocol: TCP }       # Ingress http
      - { containerPort: 443, hostPort: 443, protocol: TCP }     # Ingress https
      - { containerPort: 30080, hostPort: 30080, protocol: TCP } # KubePi
    labels:
      ingress-ready: "true"
containerdConfigPatches:
  - |-
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]
      endpoint = ["http://host.docker.internal:5000"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."localhost:5000"]
      endpoint = ["http://host.docker.internal:5000"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
      endpoint = ["http://host.docker.internal:5000"]
```

> ⚠️ `extraPortMappings` 只能集群创建时配置，之后改动需重建集群。**重建前先释放宿主机 80 端口**（停掉占用它的容器，否则报 `Bind for 0.0.0.0:80 failed`）。

**第2步**，创建集群并验证：

```bash
kind create cluster --config kind-config.yaml
kubectl get nodes          # 期望: local-dev-control-plane  Ready
kubectl get storageclass   # 期望: standard (local-path)，PVC 用它
```

## 4. 启动本地镜像仓库

**第3步**，启动 registry:2（kind 节点经 mirror 从这里拉镜像）：

```bash
docker run -d --name local-registry --restart=unless-stopped -p 5000:5000 registry:2
```

**第4步**，把集群要用的镜像推到本地仓库（ingress-nginx 两个镜像）：

```bash
docker pull registry.k8s.io/ingress-nginx/controller:v1.15.1
docker pull registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.6.9

docker tag registry.k8s.io/ingress-nginx/controller:v1.15.1 localhost:5000/ingress-nginx/controller:v1.15.1
docker tag registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.6.9 localhost:5000/ingress-nginx/kube-webhook-certgen:v1.6.9

docker push localhost:5000/ingress-nginx/controller:v1.15.1
docker push localhost:5000/ingress-nginx/kube-webhook-certgen:v1.6.9

curl -s http://localhost:5000/v2/_catalog   # 期望列出以上镜像
```

> 💡 国内网络 `registry.k8s.io` 拉不动时，用加速源拉好再 retag 推送，例如 `docker pull docker.1ms.run/registry.k8s.io/ingress-nginx/controller:v1.15.1`。实测可用加速源：`docker.1ms.run`（推荐）、`m.daocloud.io`。

## 5. 部署 ingress-nginx

**第5步**，应用官方 manifest（**去掉 `@sha256` digest**，本地仓库只有单平台，按多平台 digest 拉会 404）：

```bash
curl -sSL https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml -o ingress-nginx-kind.yaml
sed -E 's|@sha256:[a-f0-9]{64}||g' ingress-nginx-kind.yaml | kubectl apply -f -
kubectl wait --namespace ingress-nginx --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller --timeout=180s
kubectl get pods -n ingress-nginx   # 期望: ingress-nginx-controller-xxx  1/1 Running
```

> ⚠️ 若节点拉镜像报 `proxyconnect tcp: dial tcp 127.0.0.1:7890: connection refused`：kind 节点继承了宿主机代理环境变量。进入节点清掉 containerd 代理并重启：
> ```bash
> docker exec local-dev-control-plane sh -c 'mkdir -p /etc/systemd/system/containerd.service.d && printf "[Service]\nEnvironment=\"HTTP_PROXY=\" \"HTTPS_PROXY=\" \"ALL_PROXY=\" \"http_proxy=\" \"https_proxy=\" \"all_proxy=\" \"NO_PROXY=*\"\n" > /etc/systemd/system/containerd.service.d/proxy-off.conf && systemctl daemon-reload && systemctl restart containerd'
> kubectl wait --for=condition=Ready node/local-dev-control-plane --timeout=120s
> ```

## 6. 部署 KubePi 并导入集群

**第6步**，先推送 KubePi 镜像到本地仓库：

```bash
docker pull docker.1ms.run/1panel/kubepi:v2.0.0   # Docker Hub 直连慢，用加速源
docker tag docker.1ms.run/1panel/kubepi:v2.0.0 localhost:5000/1panel/kubepi:v2.0.0
docker push localhost:5000/1panel/kubepi:v2.0.0
```

**第7步**，编写 kubepi.yaml（完整文件：Namespace + PVC + ServiceAccount + cluster-admin + Deployment + NodePort Service）：

```yaml
# kubepi.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kubepi-system
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: kubepi-data
  namespace: kubepi-system
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kubepi
  namespace: kubepi-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubepi-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: kubepi
    namespace: kubepi-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubepi
  namespace: kubepi-system
spec:
  replicas: 1
  selector:
    matchLabels: { app: kubepi }
  template:
    metadata:
      labels: { app: kubepi }
    spec:
      serviceAccountName: kubepi
      containers:
        - name: kubepi
          image: 1panel/kubepi:v2.0.0
          ports: [{ containerPort: 80 }]
          env:
            - name: KUBEPI_SESSION_TIMEOUT
              value: "3600"
          volumeMounts:
            - name: data
              mountPath: /var/lib/kubepi
          resources:
            requests: { cpu: 100m, memory: 128Mi }
            limits: { cpu: 500m, memory: 512Mi }
      volumes:
        - name: data
          persistentVolumeClaim: { claimName: kubepi-data }
---
apiVersion: v1
kind: Service
metadata:
  name: kubepi
  namespace: kubepi-system
spec:
  type: NodePort
  selector: { app: kubepi }
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

**第8步**，部署并验证：

```bash
kubectl apply -f kubepi.yaml
kubectl rollout status deployment/kubepi -n kubepi-system --timeout=120s
kubectl get pods -n kubepi-system     # 期望: kubepi-xxx  1/1 Running
curl -o /dev/null -w "%{http_code}\n" http://localhost:30080   # 期望: 302（跳转登录页）
```

浏览器 `http://localhost:30080`，默认账号 `admin/kubepi`（首次登录强制改密）。

**第9步**，导入集群（kubeconfig server 改为集群内部地址）：

```bash
kind get kubeconfig --name local-dev | sed 's|server: https://127.0.0.1:[0-9]*|server: https://kubernetes.default.svc|'
```

把输出粘贴到 KubePi → 集群管理 → 导入集群。

> ⚠️ KubePi Pod 在集群内，`127.0.0.1` 是它自己；`kubernetes.default.svc` 指向 kube-apiserver，kind 证书 SAN 已含该域名，CA/客户端证书无需改。

## 7. 部署 Jenkins

**第10步**，编写 Dockerfile（完整文件，装 Docker CLI + kubectl + Node 22）：

```dockerfile
# deploy/jenkins/Dockerfile
FROM jenkins/jenkins:lts

USER root

# 安装 Docker CLI（通过 socket 调宿主机 Docker）
# jenkins:lts 基础镜像不含 gpg，直接下载 .asc 格式 key；移除 debian-security 源避免构建超时
RUN curl -fsSL https://download.docker.com/linux/debian/gpg -o /usr/share/keyrings/docker-archive-keyring.asc \
    && chmod a+r /usr/share/keyrings/docker-archive-keyring.asc \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" > /etc/apt/sources.list.d/docker.list \
    && rm -f /etc/apt/sources.list.d/debian.sources /etc/apt/sources.list \
    && echo "deb http://deb.debian.org/debian $(. /etc/os-release && echo "$VERSION_CODENAME") main" > /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends docker-ce-cli docker-compose-plugin \
    && rm -rf /var/lib/apt/lists/*

# 安装 kubectl（K8s 部署用）
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && chmod +x kubectl && mv kubectl /usr/local/bin/

# 安装 Node.js 22 LTS（Vite 8 要求 Node >=20.19），路径必须与 Jenkinsfile 的 /opt/node22 一致
RUN curl -fsSL -o /tmp/node22.tar.gz https://nodejs.org/dist/latest-v22.x/node-v22.23.2-linux-x64.tar.gz \
    && tar -xzf /tmp/node22.tar.gz -C /opt \
    && mv /opt/node-v22.23.2-linux-x64 /opt/node22 \
    && rm -f /tmp/node22.tar.gz

USER jenkins
```

**第11步**，编写 docker-compose.yaml（完整文件）：

```yaml
# deploy/jenkins/docker-compose.yaml
version: '3.8'

services:
  jenkins:
    build: .
    container_name: jenkins
    restart: unless-stopped
    ports:
      - "8888:8080"
      - "50000:50000"
    environment:
      - JAVA_OPTS=-Xmx512m
    group_add:
      - "0"        # Docker Desktop (Windows) 下 socket 属于 root 组 (GID 0)
    volumes:
      - jenkins-home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock

volumes:
  jenkins-home:
```

**第12步**，启动 Jenkins：

```bash
docker compose -f deploy/jenkins/docker-compose.yaml up -d --build
# 等就绪后（约 30s）获取初始密码：
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
# 浏览器 http://localhost:8888 安装推荐插件，创建管理员账号
```

> ⚠️ 容器自带 Temurin JDK21（`/opt/java/openjdk`），用 `-release 17` 编译 Java 17 字节码，**无需安装 JDK17 工具**；Maven 用 Jenkins 的 Global Tool Configuration 添加 3.9+。

**第13步**，给 Jenkins 容器配置访问 kind 集群的 kubeconfig：

```bash
PORT=$(kind get kubeconfig --name local-dev | grep -oE 'https://127.0.0.1:[0-9]+' | grep -oE '[0-9]+$')
kind get kubeconfig --name local-dev | \
  sed -E "s|server: https://127.0.0.1:[0-9]+|server: https://host.docker.internal:${PORT}|; s|    certificate-authority-data:.*|    insecure-skip-tls-verify: true|" \
  > jenkins-kubeconfig.yaml
docker exec jenkins mkdir -p /var/jenkins_home/.kube
docker cp jenkins-kubeconfig.yaml jenkins:/var/jenkins_home/.kube/config
# 验证：docker exec jenkins kubectl get nodes  → 期望 local-dev-control-plane Ready
```

> ⚠️ kind API 端口每次重建集群都变，重建后需重新执行第13步。

## 8. 编写应用 K8s 清单

**第14步**，4 个清单文件放 `deploy/kubernetes/`（完整内容）：

```yaml
# app-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: online-ordering
```

```yaml
# app-backend.yaml：Deployment + Service + PVC（SQLite 持久化到 /app/data）
apiVersion: apps/v1
kind: Deployment
metadata: { name: backend, namespace: online-ordering }
spec:
  replicas: 1
  selector: { matchLabels: { app: online-ordering, tier: backend } }
  template:
    metadata: { labels: { app: online-ordering, tier: backend } }
    spec:
      containers:
        - name: backend
          image: localhost:5000/online-ordering/backend:latest
          imagePullPolicy: IfNotPresent
          ports: [{ containerPort: 8080 }]
          volumeMounts: [{ name: data, mountPath: /app/data }]
          readinessProbe:
            httpGet: { path: /doc.html, port: 8080 }
            initialDelaySeconds: 15
      volumes:
        - name: data
          persistentVolumeClaim: { claimName: backend-data }
---
apiVersion: v1
kind: Service
metadata: { name: backend, namespace: online-ordering }
spec:
  selector: { app: online-ordering, tier: backend }
  ports: [{ port: 8080, targetPort: 8080 }]
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata: { name: backend-data, namespace: online-ordering }
spec:
  accessModes: [ReadWriteOnce]
  resources: { requests: { storage: 1Gi } }
  storageClassName: standard
```

```yaml
# app-frontend.yaml：Deployment + Service
# 注意：前端 nginx.conf 里 proxy_pass http://backend:8080，与 Service 名 backend 匹配
apiVersion: apps/v1
kind: Deployment
metadata: { name: frontend, namespace: online-ordering }
spec:
  replicas: 1
  selector: { matchLabels: { app: online-ordering, tier: frontend } }
  template:
    metadata: { labels: { app: online-ordering, tier: frontend } }
    spec:
      containers:
        - name: frontend
          image: localhost:5000/online-ordering/frontend:latest
          imagePullPolicy: IfNotPresent
          ports: [{ containerPort: 80 }]
---
apiVersion: v1
kind: Service
metadata: { name: frontend, namespace: online-ordering }
spec:
  selector: { app: online-ordering, tier: frontend }
  ports: [{ port: 80, targetPort: 80 }]
```

```yaml
# app-ingress.yaml：不限定 host，按路径路由前后端
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: online-ordering
  namespace: online-ordering
  annotations:
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /api/
            pathType: Prefix
            backend: { service: { name: backend, port: { number: 8080 } } }
          - path: /ws
            pathType: Prefix
            backend: { service: { name: backend, port: { number: 8080 } } }
          - path: /images/
            pathType: Prefix
            backend: { service: { name: backend, port: { number: 8080 } } }
          - path: /
            pathType: Prefix
            backend: { service: { name: frontend, port: { number: 80 } } }
```

> ⚠️ 不限定 host 的原因：限定 `host: localhost` 时，其他 Host 头的请求（如 Jenkins 健康检查用网关地址访问）返回 502。

## 9. 配置 Jenkins 流水线

**第15步**，创建 Pipeline 任务：

1. Jenkins 首页 → New Item → 名称 `online-ordering` → 选 **Pipeline**
2. 勾选 **Pipeline script from SCM**
3. SCM 选 **Git**，Repository URL 填 `https://github.com/jiangjihui/online-ordering`，Credentials 用 GitHub 凭据（Username with password，需 `repo` 权限）
4. Script Path 保持 `Jenkinsfile`
5. Build Triggers 勾选 **Poll SCM**，填 `H/2 * * * *`（每 2 分钟轮询，push 自动构建；本地环境 GitHub 够不着 localhost，Webhook 不可行）

**第16步**，Jenkinsfile 关键结构（完整版见示例项目 `https://github.com/jiangjihui/online-ordering` 的 Jenkinsfile）：

```groovy
pipeline {
    agent any
    environment {
        PROJECT_NAME   = 'online-ordering'
        IMAGE_REGISTRY = 'localhost:5000'                     // 本地仓库，kind 经 mirror 拉
        IMAGE_PREFIX   = "${IMAGE_REGISTRY}/${PROJECT_NAME}"
        IMAGE_TAG      = "${BUILD_NUMBER}-${GIT_COMMIT.take(7)}"
    }
    stages {
        stage('Checkout') { steps { checkout scm } }

        stage('Backend Build') {
            steps {
                script {
                    def jdkHome = env.JAVA_HOME ?: '/opt/java/openjdk'   // 容器自带 JDK21，-release 17 编 Java17
                    def mvnHome = tool name: 'Maven', type: 'maven'      // 需先在 Global Tool Configuration 配好
                    sh """
                        export JAVA_HOME="${jdkHome}"
                        export PATH="${jdkHome}/bin:${mvnHome}/bin:\$PATH"
                        mvn clean package -B
                    """
                }
            }
        }

        stage('Frontend Build') {
            steps {
                script {
                    def nodeHome = '/opt/node22'                        // Vite 8 要求 Node >=20.19
                    sh """
                        export PATH="${nodeHome}/bin:\$PATH"
                        npm ci --registry=https://registry.npmmirror.com
                        npm run build
                    """
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                    docker build -t ${IMAGE_PREFIX}/backend:${IMAGE_TAG} -f backend/Dockerfile backend/
                    docker build -t ${IMAGE_PREFIX}/frontend:${IMAGE_TAG} -f frontend/Dockerfile frontend/
                """
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker push ${IMAGE_PREFIX}/backend:${IMAGE_TAG}
                    docker push ${IMAGE_PREFIX}/frontend:${IMAGE_TAG}
                    kubectl apply -f deploy/kubernetes/app-namespace.yaml
                    kubectl apply -f deploy/kubernetes/app-backend.yaml
                    kubectl apply -f deploy/kubernetes/app-frontend.yaml
                    kubectl apply -f deploy/kubernetes/app-ingress.yaml
                    kubectl set image deployment/backend backend=${IMAGE_PREFIX}/backend:${IMAGE_TAG} -n online-ordering
                    kubectl set image deployment/frontend frontend=${IMAGE_PREFIX}/frontend:${IMAGE_TAG} -n online-ordering
                    kubectl rollout status deployment/backend -n online-ordering --timeout=180s
                    kubectl rollout status deployment/frontend -n online-ordering --timeout=120s
                """
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    GW=$(ip route 2>/dev/null | awk '/^default/{print $3; exit}')
                    [ -z "$GW" ] && GW="172.21.0.1"          # Docker 网关 = 宿主机
                    [ "$(curl -s -o /dev/null -w '%{http_code}' --max-time 6 http://$GW:80/)" = "200" ] \
                        || { echo "frontend 检查失败"; exit 1; }
                    [ "$(curl -s -o /dev/null -w '%{http_code}' --max-time 6 http://$GW:80/doc.html)" = "200" ] \
                        || { echo "backend 检查失败"; exit 1; }
                '''
            }
        }
    }
}
```

> Groovy 转义规则：`sh """..."""` 里 **shell 的 `$` 一律 `\$`**（如 `\$PATH`）；`${IMAGE_TAG}` 等是 Groovy 插值（environment 块定义的变量）。Health Check 用 Docker 网关 `172.21.0.1` 访问宿主机（`host.docker.internal` 在 Docker Desktop 下解析到 IPv6，80 端口转发不稳定）。

## 10. 部署与验证

**第17步**，首次部署（或直接 Jenkins 任务点 **Build Now**，自动完成构建+推送+部署）：

```bash
# 首次手动部署（不依赖 Jenkins）时，先构建并推送应用镜像到本地仓库：
docker build -t localhost:5000/online-ordering/backend:latest -f backend/Dockerfile backend/
docker build -t localhost:5000/online-ordering/frontend:latest -f frontend/Dockerfile frontend/
docker push localhost:5000/online-ordering/backend:latest
docker push localhost:5000/online-ordering/frontend:latest

# 应用 K8s 清单
kubectl apply -f deploy/kubernetes/app-namespace.yaml
kubectl apply -f deploy/kubernetes/app-backend.yaml
kubectl apply -f deploy/kubernetes/app-frontend.yaml
kubectl apply -f deploy/kubernetes/app-ingress.yaml
kubectl rollout status deployment/backend deployment/frontend -n online-ordering --timeout=180s
```

**第18步**，全链路验证：

```bash
# 应用入口
curl -o /dev/null -w "%{http_code}\n" http://localhost:80/            # 期望 200 前端
curl -o /dev/null -w "%{http_code}\n" http://localhost:80/doc.html    # 期望 200 后端 API 文档
curl http://localhost:80/api/categories                               # 期望返回菜品分类 JSON
# 集群状态
kubectl get pods -A                                                    # 期望全部 Running/Completed
kubectl top nodes    # 可选：需先部署 metrics-server，否则报 "Metrics API not available"
# 管理面板
curl -o /dev/null -w "%{http_code}\n" http://localhost:30080          # 期望 302 KubePi
curl -o /dev/null -w "%{http_code}\n" http://localhost:8888/login     # 期望 200 Jenkins
```

**第19步**，可选：本地域名访问（Ingress 不限 host，加 hosts 即可）：

```
# C:\Windows\System32\drivers\etc\hosts（管理员权限）追加：
127.0.0.1 order.test
# 浏览器访问 http://order.test
```

## 附录 A：版本清单

| 组件 | 版本 | 说明 |
|------|------|------|
| kind | 任意较新（实测 v0.2x） | `winget install Kubernetes.kind` |
| kind 节点镜像 | kindest/node:v1.31.0 | kind 自动拉取 |
| KubePi | 1panel/kubepi:v2.0.0 | 唯一可用较新版本（v1.7.2 不存在！latest 反而指向 v1.9.0） |
| ingress-nginx | controller:v1.15.1 + kube-webhook-certgen:v1.6.9 | manifest 需去掉 @sha256 digest |
| registry | registry:2 | 宿主机本地镜像仓库 |
| Jenkins | jenkins/jenkins:lts | 自定义 Dockerfile 装 docker-cli/kubectl/node22 |
| Node.js | v22.23.2（/opt/node22） | Vite 8 要求 Node >=20.19 |
| JDK | 容器自带 Temurin 21 | `-release 17` 编译 Java 17 字节码，无需装 JDK17 |
| 应用后端 | eclipse-temurin:17-jre | 运行 v61 字节码 |
| 应用前端 | node:22-alpine 构建 → nginx:alpine | nginx 反代 backend:8080 |

## 附录 B：一句话排障

| 症状 | 处置 |
|------|------|
| KubePi Pod ImagePullBackOff | 镜像 tag 用 v2.0.0；确认已 push 到 localhost:5000/1panel/kubepi:v2.0.0 |
| 导入集群 connection refused | kubeconfig server 改 `https://kubernetes.default.svc`（见第9步） |
| 节点拉镜像 proxyconnect 7890 | 清节点 containerd 代理（见第5步提示） |
| ingress-nginx 404/Pending | manifest 去 @sha256 digest 后重新 apply（见第5步） |
| kind load digest not found | 不要用 kind load，走本地 registry + mirror（见第3-4步） |
| Health Check 502 | 用 Docker 网关 172.21.0.1 而非 host.docker.internal（见第16步） |
| Jenkins 找不到 JDK17 | 直接用容器自带 JDK21（见第16步），无需 Jenkins JDK 工具 |
| 前端构建失败 | Node ≥20.19（/opt/node22），npm ci 用 npmmirror 源 |
| Jenkins 重跑失败 | 点 Build Now 全量，不要"从失败 stage 继续"（会跳过 Build 导致 Deploy 无镜像） |
| 装了 metrics-server 仍无数据 | kind kubelet 证书不含节点 IP，加参数 `--kubelet-insecure-tls` 后重启 deployment |

