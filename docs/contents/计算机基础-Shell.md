# Linux 常用命令与 Shell 脚本

## 终端快捷键

日常终端操作中高频使用的快捷键：

| 快捷键 | 功能 |
| ------ | ---- |
| `Ctrl+A` | 光标移到行首 |
| `Ctrl+E` | 光标移到行末 |
| `Ctrl+U` | 剪切光标前全部内容 |
| `Ctrl+K` | 剪切光标后至行末 |
| `Ctrl+W` | 删除光标前一个单词 |
| `Ctrl+Y` | 粘贴被剪切的内容 |
| `Ctrl+R` | 搜索历史命令 |
| `Ctrl+L` | 清屏（等同 `clear`） |
| `Ctrl+C` | 终止当前命令 |
| `Ctrl+Z` | 挂起当前命令（用 `fg` 恢复） |
| `Ctrl+D` | 退出当前 Shell（等同 `logout`） |
| `Alt+.` | 插入上一条命令的最后一个参数 |

## 系统信息

```bash
# 系统发行版
cat /etc/os-release
lsb_release -a

# 内核版本
uname -a

# CPU 信息
lscpu
nproc                      # CPU 核心数

# 内存
free -h

# 磁盘
df -h                      # 各分区使用量
du -sh *                   # 当前目录各条目大小
du -h --max-depth=1        # 按层级显示子目录大小

# 运行时间与负载
uptime
```

> `uptime` 输出：当前时间、运行时长、登录用户数、1/5/15 分钟平均负载

### 包管理器

| 包管理器 | 安装 | 搜索 | 卸载 | 更新索引 | 代表系统 |
| -------- | ---- | ---- | ---- | -------- | -------- |
| apt | `apt install` | `apt search` | `apt remove` | `apt update` | Debian、Ubuntu |
| yum | `yum install` | `yum search` | `yum remove` | `yum makecache` | CentOS 7 |
| dnf | `dnf install` | `dnf search` | `dnf remove` | `dnf makecache` | Fedora、CentOS 8+ |
| pacman | `pacman -S` | `pacman -Ss` | `pacman -R` | `pacman -Sy` | Arch、Manjaro |

## 用户与权限

```bash
# 切换用户
su - username              # 切换到指定用户（加载环境变量）
su -                       # 切换到 root

# sudo
sudo command               # 以 root 权限执行命令
sudo -u user command       # 以指定用户执行

# 首次使用 su 报 Authentication Failure 时
sudo passwd root
```

### 文件权限

```
-rwxr-xr--  1  owner  group  1024  Jan 1 00:00  filename
│├──┤├──┤├──┤
│ │   │   └── 其他用户: r-- (4)
│ │   └────── 所属组:   r-x (5)
│ └────────── 所有者:   rwx (7)
└──────────── 文件类型: - 普通文件, d 目录, l 链接
```

```bash
# 修改权限
chmod 755 script.sh        # rwxr-xr-x
chmod +x script.sh         # 添加可执行权限
chmod -w file              # 移除写权限

# 修改所有者
chown user:group file
chown -R user:group dir/   # 递归修改

# 查看权限
ls -la
stat file
```

## 文件操作

### 目录与文件导航

```bash
pwd                        # 显示当前目录
cd -                       # 返回上一次的目录
ls -la                     # 详细列表（含隐藏文件）
tree -L 2                  # 树形显示目录结构（2层）
```

### 查找文件

```bash
# 按名称查找
find /path -name "*.log"
find /path -iname "*.log"  # 忽略大小写

# 按类型查找
find /path -type d -name "dir"    # 目录
find /path -type f -name "file"   # 文件

# 按时间查找
find /path -mtime +7       # 7天前修改的文件
find /path -mtime -7       # 7天内修改的文件
find /path -mmin -60       # 60分钟内修改的文件

# 按大小查找
find /path -size +100M     # 大于 100MB 的文件

# 查找后执行操作
find /path -name "*.log" -delete           # 删除找到的文件
find /path -name "*.log" -exec gzip {} \;  # 压缩找到的文件

# 快速定位（需提前建立索引，比 find 快）
locate filename
```

### 查看文件内容

```bash
cat file                   # 查看全部内容
less file                  # 分页查看（q 退出，/ 搜索）
head -n 20 file            # 前 20 行
tail -n 20 file            # 后 20 行
tail -f file               # 实时追踪（查看日志常用）
tail -f -n 100 file        # 显示最后 100 行并持续追踪

# 统计
wc -l file                 # 行数
wc -w file                 # 单词数
wc -c file                 # 字节数
```

### 复制、移动、删除

```bash
cp file dest/              # 复制文件
cp -r dir1 dir2/           # 递归复制目录
cp -a dir1 dir2/           # 递归复制（保留权限、链接等属性）

mv file dest/              # 移动/重命名
mv old_name new_name       # 重命名

rm file                    # 删除文件
rm -r dir                  # 递归删除目录
rm -rf dir                 # 强制递归删除（谨慎使用）

# 创建链接
ln -s target link_name     # 创建软链接
ln target link_name        # 创建硬链接
ls -al link_name           # 查看链接指向
```

### 打包与解压

```bash
# tar.gz（最常用）
tar czvf archive.tar.gz dir/                    # 压缩
tar czvf archive.tar.gz dir/ --exclude=dir/sub  # 排除目录
tar xzvf archive.tar.gz                         # 解压
tar xzvf archive.tar.gz -C /tmp/                # 解压到指定目录

# tar.bz2（压缩率更高，速度更慢）
tar cjvf archive.tar.bz2 dir/
tar xjvf archive.tar.bz2

# zip
zip -r archive.zip dir/
unzip archive.zip -d /tmp/

# 查看压缩包内容（不解压）
tar tzvf archive.tar.gz
unzip -l archive.zip
```

### 文件上传下载

```bash
# rz/sz（需安装 lrzsz，适用于 XShell 等终端）
rz                         # 上传
sz filename                # 下载

# scp（基于 SSH）
scp local_file user@host:/remote/path       # 上传
scp user@host:/remote/file ./               # 下载
scp -r dir/ user@host:/remote/path          # 传目录

# rsync（增量同步，推荐）
rsync -avz src/ user@host:/dest/            # 同步目录
rsync -avz --delete src/ user@host:/dest/   # 同步并删除目标多余文件
```

## 文本处理

### grep — 文本搜索

```bash
grep "pattern" file              # 基本搜索
grep -i "pattern" file           # 忽略大小写
grep -r "pattern" dir/           # 递归搜索目录
grep -n "pattern" file           # 显示行号
grep -v "pattern" file           # 排除匹配行（反向搜索）
grep -c "pattern" file           # 统计匹配行数
grep -E "a|b" file               # 扩展正则（等同 egrep）
grep -A 3 "pattern" file         # 显示匹配行及后 3 行
grep -B 3 "pattern" file         # 显示匹配行及前 3 行

# 常用组合
ps -ef | grep java | grep -v grep   # 查找进程（排除 grep 自身）
cat file | grep -E "error|fail" -i   # 搜索多个关键词
```

### sed — 流编辑器

```bash
# 替换（默认只替换每行第一个匹配）
sed 's/old/new/' file

# 替换所有匹配
sed 's/old/new/g' file

# 替换第 N 个匹配
sed 's/old/new/2' file

# 删除空行
sed '/^$/d' file

# 删除指定行
sed '3d' file               # 删除第 3 行
sed '2,5d' file             # 删除第 2-5 行

# 直接修改文件（加 -i）
sed -i 's/old/new/g' file

# 多条命令
sed -e 's/old/new/g' -e 's/foo/bar/g' file
```

### awk — 文本分析

```bash
# 按列提取
awk '{print $1}' file               # 第 1 列
awk '{print $1, $3}' file           # 第 1 和第 3 列
awk -F: '{print $1}' /etc/passwd    # 指定分隔符

# 条件过滤
awk '$3 > 100' file                  # 第 3 列大于 100 的行
awk '$1 == "error"' file             # 第 1 列等于 "error"

# 统计
awk '{sum += $1} END {print sum}' file   # 求和
awk '{count++} END {print count}' file   # 计数

# 按连接状态统计 TCP 连接数
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

### sort / uniq — 排序与去重

```bash
sort file                   # 升序排序
sort -r file                # 降序排序
sort -n file                # 按数值排序
sort -k2 file               # 按第 2 列排序
sort -t: -k3 -n file        # 指定分隔符按第 3 列数值排序

sort file | uniq            # 去重
sort file | uniq -c         # 去重并统计出现次数
sort file | uniq -c | sort -rn   # 按出现次数降序排列
```

### xargs — 参数构建

```bash
# 将 find 结果传给其他命令
find . -name "*.log" | xargs rm -f
find . -name "*.log" | xargs grep "error"

# 控制每行参数数量
find . -name "*.log" | xargs -n 1 rm -f

# 处理含空格的文件名
find . -name "*.log" -print0 | xargs -0 rm -f
```

## 网络与连接

### 网络信息

```bash
ip addr                     # 查看 IP 地址（推荐）
ip link                     # 查看网络接口

# 端口与连接
ss -tlnp                    # 查看监听端口（推荐）
ss -ant                     # 查看所有 TCP 连接
netstat -ant                # 旧版命令

# 查看有效连接数
ss -ant | grep ESTAB | wc -l

# 按状态统计连接数
ss -ant | awk 'NR>1 {++S[$1]} END {for(a in S) print a, S[a]}'
```

**TCP 连接状态：**

| 状态 | 说明 |
| ---- | ---- |
| LISTEN | 等待远程连接 |
| SYN_SENT | 已发起连接请求 |
| SYN_RECV | 收到连接请求，等待确认 |
| ESTABLISHED | 连接已建立，正常传输 |
| FIN_WAIT1/2 | 本端已请求/对端已确认关闭 |
| CLOSE_WAIT | 远端已关闭，等待本地关闭 |
| TIME_WAIT | 等待超时结束 |

### SSH

```bash
# 远程连接
ssh user@host
ssh -p 2222 user@host      # 指定端口

# 免密登录（公钥认证）
ssh-keygen -t ed25519                  # 生成密钥
ssh-copy-id user@host                  # 复制公钥到远程主机
# 或手动复制：cat ~/.ssh/id_ed25519.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# SSH 配置文件（~/.ssh/config）简化连接
Host myserver
    HostName 192.168.1.100
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
# 使用：ssh myserver
```

### 端口扫描与检测

```bash
# 检测端口是否开放
nc -zv host 8080
telnet host 8080

# 扫描端口
nmap host
nmap -p 80,443,8080 host   # 扫描指定端口
```

### 网络诊断

```bash
ping host                  # 测试连通性
traceroute host            # 路由追踪
dig domain                 # DNS 查询
nslookup domain            # DNS 查询（旧版）
host domain                # DNS 查询

# 网络流量
iftop                      # 实时流量监控
tcpdump -i eth0 port 80    # 抓包
```

## 进程与服务管理

### 进程管理

```bash
# 查看进程
ps aux                     # 查看所有进程
ps -ef | grep java         # 查找指定进程
pgrep -f java              # 按名称查找 PID

# 实时监控
top
htop                       # 更友好（需安装）

# 查看进程详情
lsof -p <PID>              # 进程打开的文件
lsof -i :8080              # 占用端口的进程
/proc/<PID>/cmdline        # 完整命令行

# 终止进程
kill <PID>                 # 发送 SIGTERM（优雅终止）
kill -9 <PID>              # 发送 SIGKILL（强制终止）
killall java               # 按名称终止
pkill -f "java -jar app"   # 按完整命令匹配终止
```

**常用信号：**

| 信号 | 编号 | 说明 |
| ---- | ---- | ---- |
| SIGHUP | 1 | 挂起（常用于重载配置） |
| SIGINT | 2 | 中断（Ctrl+C） |
| SIGTERM | 15 | 终止（默认，优雅退出） |
| SIGKILL | 9 | 强制终止（无法捕获） |

### 后台运行

```bash
# 后台运行
command &                  # 后台运行（退出终端会终止）
nohup command &            # 退出终端不终止
nohup ./app >/dev/null 2>&1 &          # 不输出日志
nohup java -jar app.jar > app.log 2>&1 &  # 输出到日志

# 查看后台任务
jobs                       # 列出当前 Shell 的后台任务
fg %1                      # 将任务 1 调到前台
bg %1                      # 将任务 1 放到后台继续

# 挂起与恢复
Ctrl+Z                     # 挂起当前前台程序
fg                         # 恢复到前台
bg                         # 恢复到后台
```

### 服务管理（systemctl）

```bash
systemctl start service      # 启动
systemctl stop service       # 停止
systemctl restart service    # 重启
systemctl status service     # 查看状态
systemctl enable service     # 开机自启
systemctl disable service    # 取消开机自启
systemctl daemon-reload      # 重新加载配置文件

# 查看服务日志
journalctl -u service        # 查看服务日志
journalctl -u service -f     # 实时追踪
journalctl -u service --since "1 hour ago"
```

### 应用注册为系统服务

创建 `/etc/systemd/system/myapp.service`：

```ini
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=deploy
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/java -jar /opt/myapp/app.jar
ExecStop=/bin/kill -SIGTERM $MAINPID
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl start myapp
systemctl enable myapp
```

## 系统配置

### 防火墙（firewalld）

```bash
firewall-cmd --state                       # 查看状态
firewall-cmd --list-all                    # 查看所有规则

# 开放端口
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --reload

# 对指定 IP 开放端口
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="6379" accept'

# 删除规则
firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="6379" accept'
firewall-cmd --reload
```

### 环境变量

```bash
# 查看环境变量
echo $PATH
env                               # 查看所有环境变量
printenv HOME                     # 查看指定变量

# 设置环境变量（当前会话）
export MY_VAR="value"
export PATH=$PATH:/new/path

# 持久化（写入配置文件）
echo 'export MY_VAR="value"' >> ~/.bashrc
source ~/.bashrc                  # 立即生效

# 系统级环境变量
sudo vim /etc/environment         # 所有用户生效
sudo vim /etc/profile.d/myapp.sh  # 登录时执行（推荐方式）
```

### 编码设置

```bash
export LANG=zh_CN.UTF-8
export LC_ALL=zh_CN.UTF-8
locale                           # 查看当前编码设置
```

### 更换镜像源

```bash
# Ubuntu — 换为中科大源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo sed -i 's/cn.archive.ubuntu.com/mirrors.ustc.edu.cn/' /etc/apt/sources.list
sudo apt update

# CentOS — 换为阿里云源
sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
sudo curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
```

## 常用工具

### Curl

```bash
# GET
curl https://example.com
curl -s https://example.com                 # 静默模式（不显示进度）
curl -o page.html https://example.com       # 保存到文件
curl -O https://example.com/file.zip        # 使用 URL 文件名保存

# POST
curl -X POST -d 'key=value' https://example.com/api
curl -X POST -d 'key1=val1&key2=val2' https://example.com/api

# JSON
curl -X POST -H "Content-Type: application/json" \
  -d '{"user":"admin","pass":"123"}' \
  https://example.com/api

# 常用选项
curl -v https://example.com                 # 显示请求/响应详情
curl -H "Authorization: Bearer token" url   # 添加请求头
curl -L url                                 # 跟随重定向
curl -C - -O url                            # 断点续传
curl --limit-rate 200k url                  # 限速
curl -w "\nHTTP Code: %{http_code}\n" url   # 显示 HTTP 状态码

# 上传文件
curl -F "file=@local.txt" https://example.com/upload
```

### jq — JSON 处理

```bash
# 安装
apt install jq / yum install jq

# 格式化
cat data.json | jq .

# 提取字段
cat data.json | jq '.name'
cat data.json | jq '.users[0].name'

# 提取数组
cat data.json | jq '.users[].name'

# 过滤
cat data.json | jq '.users[] | select(.age > 18)'

# 提取多个字段
cat data.json | jq '.users[] | {name, age}'
```

### VIM

**模式：** 普通模式（默认）→ `i` 进入插入模式 → `Esc` 回到普通模式 → `:` 进入命令模式

| 操作 | 命令 |
| ---- | ---- |
| 搜索 | `/keyword`（`n` 下一个，`N` 上一个） |
| 反向搜索 | `?keyword` |
| 删除一行 | `dd` |
| 删除 n 行 | `3dd` |
| 复制一行 | `yy` |
| 粘贴 | `p`（下方）/ `P`（上方） |
| 撤销 | `u` |
| 重做 | `Ctrl+R` |
| 跳到第 n 行 | `:n` 或 `nG` |
| 文件头/尾 | `gg` / `G` |
| 翻页 | `Ctrl+F` / `Ctrl+B` |
| 保存退出 | `:wq` / `ZZ` |
| 强制退出 | `:q!` |
| 全局替换 | `:%s/old/new/g` |
| 显示行号 | `:set nu` |

## Shell 脚本编程

### 脚本基础

```bash
#!/bin/bash
# shebang 行指定解释器，必须放在第一行
```

执行方式：

```bash
chmod +x script.sh && ./script.sh    # 方式一：直接执行
bash script.sh                        # 方式二：指定解释器
source script.sh                      # 方式三：在当前 Shell 执行（可修改当前环境）
. script.sh                           # 同 source
```

调试：

```bash
bash -x script.sh                     # 执行并打印每条命令
bash -n script.sh                     # 仅检查语法
```

### 特殊变量

| 变量 | 含义 |
| ---- | ---- |
| `$0` | 脚本名称 |
| `$1` ~ `$9` | 第 1~9 个参数 |
| `${10}` | 第 10 个及以上参数 |
| `$#` | 参数个数 |
| `$@` | 所有参数（每个独立） |
| `$*` | 所有参数（作为一个整体） |
| `$?` | 上一条命令的退出码（0=成功） |
| `$$` | 当前进程 PID |
| `$!` | 最近一个后台进程 PID |

```bash
#!/bin/bash
echo "脚本名: $0"
echo "第一个参数: $1"
echo "参数个数: $#"
echo "所有参数: $@"
echo "上一条命令退出码: $?"
```

### 变量与字符串

```bash
# 变量赋值（等号两侧不能有空格）
NAME="hello"

# 使用变量
echo $NAME
echo ${NAME}

# 字符串操作
str="Hello World"
echo ${#str}               # 字符串长度：11
echo ${str:0:5}            # 截取：Hello
echo ${str/World/Linux}    # 替换第一个：Hello Linux
echo ${str//l/L}           # 替换所有：HeLLo WorLd
echo ${str#Hello}          # 删除前缀： World
echo ${str%World}          # 删除后缀：Hello

# 默认值
echo ${NAME:-"default"}    # NAME 为空时使用 default
echo ${NAME:="default"}    # NAME 为空时赋值并使用

# 命令替换
DATE=$(date +%Y%m%d)
FILES=$(ls *.txt)
```

### 数组

```bash
# 定义数组
arr=(one two three four)

# 读取
echo ${arr[0]}             # 第一个元素：one
echo ${arr[-1]}            # 最后一个元素：four
echo ${arr[@]}             # 所有元素

# 长度
echo ${#arr[@]}            # 元素个数：4
echo ${#arr[0]}            # 第一个元素的长度：3

# 遍历
for item in "${arr[@]}"; do
    echo "$item"
done

# 添加元素
arr+=("five")

# 切片
echo ${arr[@]:1:2}         # 第二个起取 2 个：two three
```

### 条件测试

```bash
# 文件测试
[ -f file ]      # 是否为普通文件
[ -d dir ]       # 是否为目录
[ -e path ]      # 是否存在
[ -r file ]      # 是否可读
[ -w file ]      # 是否可写
[ -x file ]      # 是否可执行
[ -s file ]      # 是否非空

# 字符串测试
[ -z "$str" ]    # 是否为空
[ -n "$str" ]    # 是否非空
[ "$a" = "$b" ]  # 是否相等
[ "$a" != "$b" ] # 是否不等

# 整数比较
[ $a -eq $b ]    # 等于
[ $a -ne $b ]    # 不等于
[ $a -gt $b ]    # 大于
[ $a -ge $b ]    # 大于等于
[ $a -lt $b ]    # 小于
[ $a -le $b ]    # 小于等于

# 推荐使用 [[ ]] （bash 扩展，支持 && 和 ||）
[[ -f file && -r file ]]
[[ "$str" == prefix* ]]     # 支持模式匹配
```

### 流程控制

```bash
# if-elif-else
if [[ $count -gt 100 ]]; then
    echo "too many"
elif [[ $count -gt 50 ]]; then
    echo "enough"
else
    echo "not enough"
fi

# for 循环
for i in 1 2 3; do
    echo $i
done

for item in *.txt; do
    echo "$item"
done

# C 风格 for
for ((i=0; i<10; i++)); do
    echo $i
done

# while 循环
while read line; do
    echo "$line"
done < file.txt

# 无限循环
while true; do
    echo "running..."
    sleep 1
done

# case
case $1 in
    start)  echo "starting"  ;;
    stop)   echo "stopping"  ;;
    restart) echo "restarting" ;;
    *)      echo "usage: $0 {start|stop|restart}" ;;
esac
```

### 函数

```bash
# 定义函数
greet() {
    local name=$1          # local 声明局部变量
    echo "Hello, $name"
    return 0               # 返回值（0-255）
}

# 调用
greet "World"
echo "返回值: $?"          # 获取返回值

# 通过 echo 返回字符串
get_date() {
    echo $(date +%Y%m%d)
}
result=$(get_date)         # 捕获输出
echo "$result"
```

### 管道与重定向

```bash
# 管道
command1 | command2        # command1 的输出作为 command2 的输入

# 输出重定向
command > file             # 覆盖写入
command >> file            # 追加写入
command 2> error.log      # 仅重定向 stderr
command > file 2>&1        # stdout 和 stderr 都写入文件
command &> file            # 同上（bash 简写）
command >/dev/null 2>&1    # 丢弃所有输出

# 输入重定向
command < file             # 从文件读取输入

# Here Document
cat <<EOF
多行文本内容
变量: $DATE
EOF

# Here String
grep "pattern" <<< "$variable"
```

### set 选项

```bash
set -e                     # 遇到错误立即退出
set -u                     # 使用未定义变量时报错
set -o pipefail            # 管道中任一命令失败则整体失败
set -x                     # 打印每条执行的命令

# 推荐在脚本开头添加
#!/bin/bash
set -euo pipefail
```

## 定时任务（cron）

### crontab 语法

```
分  时  日  月  星期  命令
```

| 字段 | 范围 | 特殊符号 |
| ---- | ---- | -------- |
| 分 | 0-59 | `*` 任意值，`,` 枚举，`-` 范围，`/` 步长 |
| 时 | 0-23 | |
| 日 | 1-31 | |
| 月 | 1-12 | |
| 星期 | 0-6（0=周日） | |

### 常用示例

```bash
*/5 * * * *      cmd       # 每 5 分钟
0 * * * *        cmd       # 每小时整点
0 2 * * *        cmd       # 每天凌晨 2 点
0 2 * * 1-5      cmd       # 工作日 2 点
0 0 1 * *        cmd       # 每月 1 号零点
30 6 * * 0       cmd       # 每周日 6:30
0,30 18-23 * * * cmd       # 每天 18:00-23:00 每隔 30 分钟
```

### crontab 命令

```bash
crontab -e                  # 编辑
crontab -l                  # 列出
crontab -r                  # 删除
crontab -l > ~/mycron       # 备份
crontab -u user -e          # 指定用户
```

> crontab 中所有路径必须使用绝对路径，且脚本中应设置必要的环境变量，cron 不会继承用户环境。

### 开启 cron 日志（Ubuntu）

```bash
sudo vim /etc/rsyslog.d/50-default.conf
# 取消 cron.* 前的注释
sudo systemctl restart rsyslog
# 日志位于 /var/log/cron.log
```

## Shell 脚本示例

### 日志备份与清理

```bash
#!/bin/bash
set -euo pipefail

LOG_PATH=/home/app/logs/
BACKUP_PATH=/home/app/backup/
DAYS=14
BACKUP_FILE=$(date +%Y%m%d)_log_backup.tar.gz

cd "$LOG_PATH"
echo "[$(date)] 开始备份 ${DAYS} 天前的日志..."

# 查找并打包
find . -mtime +$DAYS -type f | tar -czvf "$BACKUP_FILE" -T -
mv "$BACKUP_FILE" "$BACKUP_PATH"
echo "备份完成: ${BACKUP_PATH}${BACKUP_FILE}"

# 清理原文件
find . -mtime +$DAYS -type f -delete
echo "已清理 ${DAYS} 天前的日志"
```

### 日志备份与清空

```bash
#!/bin/bash
set -euo pipefail

LOG_PATH=/home/app/logs/
BACKUP_PATH=/home/app/backup/
LOG_NAME="catalina.out"
TIMESTAMP=$(date +%Y%m%d%H%M)

# 备份并压缩
cp "${LOG_PATH}${LOG_NAME}" "${BACKUP_PATH}${TIMESTAMP}_${LOG_NAME}"
cd "$BACKUP_PATH"
tar -czvf "${TIMESTAMP}_${LOG_NAME}.tar.gz" "${TIMESTAMP}_${LOG_NAME}"
rm "${TIMESTAMP}_${LOG_NAME}"

# 清空原日志
> "${LOG_PATH}${LOG_NAME}"
echo "[$(date)] 日志已备份并清空"

# 清理过期备份
find "$BACKUP_PATH" -mtime +7 -name "*.tar.gz" -delete
```

### 自动部署

```bash
#!/bin/bash
set -euo pipefail

APP_PATH=/home/tomcat/app/
BACKUP_PATH=/home/tomcat/backup/
TARGET=app.war
BACKUP_FILE=$(date +%Y%m%d%H%M%S)_backup.tar.gz

cd "$APP_PATH"

# 备份当前应用
tar -czf "$BACKUP_FILE" * --exclude="$TARGET"
mv "$BACKUP_FILE" "$BACKUP_PATH"
echo "[$(date)] 备份完成"

# 解压新版本
unzip -o "$TARGET"
rm "$TARGET"
echo "[$(date)] 部署完成"

# 重启服务
SERVER_PATH=/home/tomcat/apache-tomcat
cd "$SERVER_PATH/bin"

# 等待进程关闭
timeout=30
while [ $(ps -ef | grep "$SERVER_PATH" | grep -v grep | wc -l) -gt 0 ] && [ $timeout -gt 0 ]; do
    ./shutdown.sh
    sleep 2
    timeout=$((timeout - 2))
done

# 强制关闭残留进程
if [ $(ps -ef | grep "$SERVER_PATH" | grep -v grep | wc -l) -gt 0 ]; then
    pkill -f "$SERVER_PATH"
    sleep 2
fi

./startup.sh
echo "[$(date)] 服务已重启"
```

### 批量生成测试数据

```bash
#!/bin/bash
# 生成 Redis SET 命令
for ((i=0; i<10000; i++)); do
    echo "SET user${i} value${i}"
done > redis_commands.txt
echo "已生成 10000 条 Redis 命令到 redis_commands.txt"
```
