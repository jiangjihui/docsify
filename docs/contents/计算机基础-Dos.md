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

### 对开发者的好处

- **统一开发体验**：在 Windows 上使用与 Linux 服务器相同的命令，减少跨平台开发的学习成本
- **强大的管道能力**：可以将多个命令组合起来，处理复杂的文本和数据操作
- **Shell 脚本自动化**：编写脚本实现重复性任务的自动化
- **Git 原生支持**：Git Bash 内置 Git 支持，命令行操作 Git 更加高效
- **SSH/SCP 远程操作**：方便连接远程服务器进行操作
- **兼容 Linux 生态**：许多 Linux 工具和脚本可以直接在 Git Bash 中运行

