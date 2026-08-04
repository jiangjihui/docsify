# 命令行工具指南

## DOS 命令

### 目录操作

#### 进入文件目录

Windows 下进入某个盘的文件目录前，需要确定当前是否已经在这个盘里，如果不在则需要输入盘符之后再使用 cd 命令进入：

```powershell
Microsoft Windows [版本 10.0.15063]
(c) 2017 Microsoft Corporation。保留所有权利。
C:\Users\xqlsr>cd D:\Temp
C:\Users\xqlsr>
```

上面操作无效，正确操作如下：

```powershell
Microsoft Windows [版本 10.0.15063]
(c) 2017 Microsoft Corporation。保留所有权利。
C:\Users\xqlsr>D:
D:\>cd D:\Temp
D:\Temp>
```

#### 切换盘符

直接输入盘符即可切换到对应盘：

```powershell
D:     # 切换到 D 盘
E:     # 切换到 E 盘
```

---

### 文件操作

#### 常用文件操作命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `dir` | 列出目录内容 | `dir` / `dir /w` |
| `md` 或 `mkdir` | 创建目录 | `md newfolder` |
| `rd` 或 `rmdir` | 删除目录（空目录） | `rd oldfolder` |
| `del` | 删除文件 | `del filename.txt` |
| `copy` | 复制文件 | `copy a.txt b.txt` |
| `move` | 移动/重命名文件 | `move a.txt folder\` |
| `type` | 查看文件内容 | `type readme.txt` |
| `cls` | 清屏 | `cls` |

#### 使用示例

```powershell
# 查看当前目录内容
dir

# 创建新目录
md myproject

# 复制文件
copy D:\file.txt D:\backup\file.txt

# 移动文件
move D:\old.txt D:\newfolder\new.txt

# 查看文件内容
type config.ini

# 删除文件（确认提示）
del test.txt
```

---

### 系统信息

#### 常用系统信息命令

| 命令 | 说明 |
|------|------|
| `ver` | 查看 Windows 版本 |
| `date` | 查看/设置日期 |
| `time` | 查看/设置时间 |
| `tasklist` | 查看所有进程 |
| `systeminfo` | 查看系统详细信息（需管理员权限） |

#### 使用示例

```powershell
# 查看 Windows 版本
ver

# 查看当前日期
date

# 查看当前时间
time

# 查看所有运行中的进程
tasklist

# 查看详细系统信息（内存、CPU、操作系统等）
systeminfo
```

> **注意**：`systeminfo` 命令输出信息较多，可以通过管道过滤：`systeminfo | findstr "OS"`

---

### 网络命令

#### 常用网络命令

| 命令 | 说明 |
|------|------|
| `ipconfig` | 查看 IP 配置 |
| `ipconfig /all` | 查看详细的网络配置信息 |
| `ping` | 测试网络连接 |
| `tracert` | 路由追踪 |
| `nslookup` | DNS 查询 |

#### 使用示例

```powershell
# 查看 IP 地址
ipconfig

# 查看详细网络信息
ipconfig /all

# 测试网络连通性
ping www.baidu.com

# 追踪路由
tracert www.baidu.com

# DNS 查询
nslookup www.baidu.com
```

---

### 进程与网络

#### 查看占用指定端口的程序

当你在用 Tomcat 发布程序时，经常会遇到端口被占用的情况，我们想知道是哪个程序或进程占用了端口，可以用该命令：

```powershell
netstat -ano|findstr "8080"
```

---

### 环境变量

#### set 命令

- `set`：查看所有环境变量
- `set PATH`：查看某一个环境变量

```powershell
> set JAVA_HOME
> JAVA_HOME=C:\Program Files\Java\jdk1.8.0_111
```

#### echo 命令

`echo %VAR%`：查看某一环境变量

```powershell
> echo %JAVA_HOME%
> JAVA_HOME=C:\Program Files\Java\jdk1.8.0_111
```

---

### 脚本示例

#### 运行 Java 程序

```powershell
chcp 65001

@echo off

java -server -jar -Duser.timezone=Asia/Shanghai -XX:-HeapDumpOnOutOfMemoryError D:\app\hro\app.jar --spring.profiles.active=prod
```

---

## Git Bash

### 什么是 Git Bash

Git Bash 是 Git for Windows 附带的一个命令行工具，基于 MinGW（Minimalist GNU for Windows）。它让 Windows 用户可以使用 Unix/Linux 命令，提供了一个类似于 Linux/Unix 的命令行环境。

### 安装与配置

1. **安装**：从 [Git 官网](https://git-scm.com/) 下载 Git for Windows，安装时选择 "Git Bash Here" 选项
2. **配置**：安装完成后，可以在 Git Bash 中配置用户信息：
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

### 常用工具分类

| 类别 | 工具 | 说明 |
|------|------|------|
| 文件操作 | `ls`, `cp`, `mv`, `rm`, `mkdir`, `touch` | 类似 Linux 文件命令 |
| 文本处理 | `cat`, `grep`, `sed`, `awk`, `head`, `tail` | 文本查看、搜索、替换、分析 |
| 进程管理 | `ps`, `kill`, `top` | 进程查看与终止 |
| 网络 | `curl`, `wget`, `ssh`, `scp` | HTTP请求、下载、远程连接 |
| 压缩 | `zip`, `unzip`, `tar` | 文件压缩与解压 |
| 版本控制 | `git` | 版本控制 |
| 构建 | `make`, `gcc`, `g++` | 项目构建与编译 |
| 管道与脚本 | `|`, `&&`, `||`, `for`, `while` | 管道连接与脚本控制 |

### 常用命令示例

#### 文件操作

```bash
# 列出目录内容
ls -la

# 创建目录
mkdir newproject

# 复制文件
cp file.txt backup/

# 移动/重命名
mv oldname.txt newname.txt

# 删除文件
rm unwanted.txt
```

#### 文本处理

```bash
# 查看文件内容
cat readme.txt

# 搜索内容
grep "error" log.txt

# 查看文件前10行
head -n 10 file.txt

# 查看文件后10行
tail -n 10 file.txt

# 统计行数
wc -l file.txt
```

#### 网络操作

```bash
# 发送 HTTP 请求
curl https://api.example.com/data

# 下载文件
wget https://example.com/file.zip

# SSH 远程连接
ssh user@server

# 远程复制文件
scp local.txt user@server:/remote/path/
```

### 与 CMD/PowerShell 的区别

| 特性 | Git Bash | CMD | PowerShell |
|------|----------|-----|------------|
| 命令语法 | Unix/Linux 风格 | Windows 风格 | PowerShell Cmdlet |
| 路径格式 | `/c/path` 或 `/mnt/c/path` | `C:\path` | `C:\path` |
| 换行符 | LF（Unix） | CRLF | CRLF |
| 管道功能 | 强大，支持复杂组合 | 基础 | 强大，对象管道 |
| 环境 | MinGW | cmd.exe | .NET |

#### 路径格式差异

```bash
# Git Bash 中表示 Windows 路径
/c/Users/Admin/Documents
/mnt/c/Users/Admin/Documents

# CMD 中
C:\Users\Admin\Documents

# PowerShell 中
C:\Users\Admin\Documents
```

> **提示**：在 Git Bash 中可以直接使用 `cd /d/D:` 切换到 D 盘，与 CMD 类似。

### Git Bash 实用技巧

#### 常用快捷键

- `Ctrl + L`：清屏（与 `clear` 命令相同）
- `Ctrl + C`：终止当前命令
- `Ctrl + U`：清除当前行输入
- `Tab`：自动补全命令/文件名
- `Ctrl + R`：搜索命令历史

#### 环境变量配置

Git Bash 会自动继承系统 PATH 环境变量。如果需要添加自定义路径，可以在 `.bashrc` 文件中配置：

```bash
# 编辑 ~/.bashrc
export PATH="/c/mytools:$PATH"
```

#### 包管理器

Git Bash 基于 MSYS2，可以使用部分 MSYS2 包：

```bash
# 更新包列表
pacman -Sy

# 安装包（需要管理员权限）
pacman -S package-name
```

> **注意**：部分包可能需要额外配置或管理员权限。

### 对开发者的好处

- **统一开发体验**：在 Windows 上使用与 Linux 服务器相同的命令，减少跨平台开发的学习成本
- **强大的管道能力**：可以将多个命令组合起来，处理复杂的文本和数据操作
- **Shell 脚本自动化**：编写脚本实现重复性任务的自动化
- **Git 原生支持**：Git Bash 内置 Git 支持，命令行操作 Git 更加高效
- **SSH/SCP 远程操作**：方便连接远程服务器进行操作
- **兼容 Linux 生态**：许多 Linux 工具和脚本可以直接在 Git Bash 中运行
