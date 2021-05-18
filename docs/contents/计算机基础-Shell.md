## **Authentication Failure（认证失败）**

当第一次使用su命令时，可能出现这种情况，遇到这种情况时，使用sudo passwd root重置密码即可。



 

## **Ubuntu设置允许root远程登陆**

Ubuntu默认是不开启root远程登录的，需要设置：

编辑配置文件：

```shell
sudo vim /etc/ssh/sshd_config
```

修改参数：PermitRootLogin without-password -》PermitRootLogin yes

重启ssh：

```shell
service ssh restart
```

 

 

## **VMware平台安装ubuntu**

安装ubuntu的时候不选择简易安装，在新建完虚拟机之后再去CD-ROM挂载需要的镜像。否则开机可能会一直出现：

'SMBus Host Controller not enabled'(还未进入系统）

或者一直卡在：

installing open-vm-tools

 

 

## **更换镜像源**

更换国内软件源，推荐中国科技大学的源，稳定速度快

| 备份镜像列表   | sudo cp  /etc/apt/sources.list /etc/apt/sources.list.bak     |
| -------------- | ------------------------------------------------------------ |
| 替换镜像源链接 | sudo sed -i  's/cn.archive.ubuntu.com/mirrors.ustc.edu.cn/' /etc/apt/sources.list  sudo sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g'  /etc/apt/sources.list |

 

 

## CentOS 7 防火墙设置

centos安装后，默认是开启[防火墙](https://www.linuxidc.com/Linux/2016-12/138979.htm)的，外部访问不了centos的端口。

**查看防火墙状态**

```
firewall-cmd --state
systemctl status firewalld.service
```

**关闭防火墙**

```
systemctl stop firewalld.service
```

**开启防火墙**

```
systemctl start firewalld.service
```

**停止开机启动**

```
systemctl disable firewalld.service
```

**查看已经开放的端口**

```
firewall-cmd --list-ports
```

**开启端口**

```
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

**对指定IP开放指定**[**端口**](https://blog.csdn.net/qguanri/article/details/51673845)

```
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.142.166" port protocol="tcp" port="6379" accept"
```

**删除规则**

```
firewall-cmd --permanent --remove-rich-rule="rule family="ipv4" source address="192.168.142.166" port protocol="tcp" port="11300" accept"

systemctl restart firewalld.service
```

 

 

## CentOS 7 设置静态IP

如果linux操作系统通过dhcp无法自动获取IP地址，需要手动[设置](https://jingyan.baidu.com/article/a501d80c3c9b8aec630f5e8c.html)静态IP地址

进入到网卡配置目录：

```
cd /etc/sysconfig/network-scripts/
```

ifconfig查看网卡信息并获取到网卡的名称位ens33，编辑对应的配置文件[ifcfg-ens33](https://www.jianshu.com/p/70f0a3ae4643?utm_source=oschina-app)：

vi ifcfg-ens33：

```
BOOTPROTO=static
IPADDR=192.168.1.149
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
ONBOOT=yes
```

**重启network**

```
systemctl restart network.service
# 或者
service network restart
```





## **CentOS 7** **替换镜像**[**源**](https://blog.csdn.net/z13615480737/article/details/78945623)

```
# 备份
yum install wget  -y
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# 下载
CentOS 5
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo
CentOS 6
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
CentOS 7
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 生成缓存
yum makecache
# 更新yum
yum -y update
```





## **CentOS 7** **桌面操作**

```
# 关闭图形界面
 [root@bogon ~]# init 3
# 开启图形界面
 [root@bogon ~]# init 5
```





## [**SSH使用前的准备**](http://www.linuxidc.com/Linux/2015-03/115056.htm)

| **被控制端** | SSH 服务器的安装 | sudo apt-get install  openssh-server | 出现问题，可重启 SSH 服务器：sudo  service ssh restart |
| ------------ | ---------------- | ------------------------------------ | ------------------------------------------------------ |
| **控制端**   | SSH 客户端的安装 | sudo apt-get install  openssh-client |                                                        |

 

## **查看IP地址**

```shell
ifconfig
```

 

 

## **远程桌面**

| sudo apt-get install  xrdp            | 安装xrdp       |
| ------------------------------------- | -------------- |
| sudo apt-get install  vnc4server      | 安装vnc4server |
| sudo apt-get install  xubuntu-desktop | 安装xfce4      |

Tip:可在windows的远程桌面（运行：mstsc）打开远程桌面

 

## **查看系统信息**

| free -h | 快速查看内存使用情况                     |                                                              |
| ------- | ---------------------------------------- | ------------------------------------------------------------ |
| htop    | 显示每个进程的内存实时使用率（需要安装） |                                                              |
| top     | 实时的运行中的程序的资源使用统计         | centos:  yum -y install epel-release  / yum -y  install htop |
| du -sh  | 显示当前目录所占用的空间大小             |                                                              |
| df -h   | 显示磁盘使用量和占用率                   | du -h --max-depth=1                                          |

 

 

## **查看网络流量**
```  
# 安装
apt install iftop
yum install iftop

# 使用
iftop
```




## **查看网络连接状态**

只查看有效连接数：
```shell
netstat -nat|grep ESTABLISHED|wc -l 
```
对服务器各种状态下的连接数分组并查询得到结果：
```shell
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```


状态描述： 

- CLOSED：无连接是活动的或正在进行

- LISTEN：服务器在等待进入呼叫

- SYN_RECV：一个连接请求已经到达，等待确认

- SYN_SENT：应用已经开始，打开一个连接

- ESTABLISHED：正常数据传输状态

- FIN_WAIT1：应用说它已经完成

- FIN_WAIT2：另一边已同意释放

- ITMED_WAIT：等待所有分组死掉

- CLOSING：两边同时尝试关闭

- TIME_WAIT：另一边已初始化一个释放

- LAST_ACK：等待所有分组死掉


> **注：**
>
> TIME_WAIT    处理完毕，等待超时结束的连接数
>
> ESTABLISHED    有效的连接数





## **打包文件**

| tar   | tar czvf  My.tar File1                                       | 文件                                                     |
| ----- | ------------------------------------------------------------ | -------------------------------------------------------- |
|       | tar czvf  My.tar dir1                                        | 目录                                                     |
|       | tar czvf  My.tar dir1 dir2                                   | 目录1 目录2                                              |
|       | tar czvf  My.tar dir1   --exclude=dir1/subdir2   --exclude=dir1/subdir3 | 打包目录1，排除（不打包）目录1下面的subdir2和subdir3目录 |
| unzip | unzip [test](http://man.linuxde.net/test).zip                | 解压到当前目录                                           |
|       | unzip  test.zip -d /tmp                                      | 解压到指定目录                                           |
| zip   | zip -r  mydata.zip mydata                                    | 把/home目录下面的mydata目录压缩为mydata.zip              |
|       | unzip mydata.zip -d mydatabak                                | 把/home目录下面的mydata.zip解压到mydatabak目录里面       |
| jar   | jar  -xvf absbank.war                                        | 解压war包                                                |
|       | jar -cvf absbank.war /absbank                                | 打war包                                                  |

> **注意：**unzip可以解压war包

 

 

## **复制移动文件**

```shell
#将文件fileName拷贝到dirName目录下
cp fileName dirName

#将文件夹dirName1拷贝到dirName2目录下
cp -r dirName1 dirName2

#将文件fileName移动到dirName目录下
mv fileName dirName
```

 

## **删除文件**

```shell
# 删除文件或目录
rm -rf {fileName}
```

 


## **查看文件内容**

```shell
#查看所有内容
cat {fileName}

#实时输出文件最新内容
tail -f {fileName}

#查看当前目录下各目录占用的磁盘空间大小
sudo du -ah  --max-depth=1
```

 



## **光标移动命令**

| 命令        | 作用       |
| ------------- | ---------------------- |
| Ctrl+U        | 剪切光标前的内容       |
| Ctrl+K        | 剪切光标前至行末的内容 |
| Ctrl+Y        | 粘贴                   |
| Ctrl+E        | 移动光标到行末         |
| Ctrl+A        | 移动光标到行首         |
| Ctrl+W        | 剪切光标前一个单词     |
| Alt+F         | 跳向下一个空格         |
| Alt+B         | 跳回上一个空格         |
| Alt+BackSpace | 删除前一个单词         |
| Shift+Insert  | 向终端内粘贴文本       |

 

## **暂停程序（比如编辑文件的时候）**

```shell
# 暂停程序
Ctrl+Z
# 重新唤起这个程序
fg
```




## **后台运行程序**

```shell
# 不打印日志
nohup ./program >/dev/null 2>&1 &

# 打印日志
nohup java -jar app.jar > /saas/app.log 2>&1 &
```

  

## **用户管理**

```shell
# 进入超级用户
su -
```




## **安装软件**

| apt-get -f  install                        | 修复安装                                                 |
| ------------------------------------------ | -------------------------------------------------------- |
| apt-get  autoclean                         | 这个命令将已经删除了的软件包的.deb安装文件从硬盘中删除掉 |
| sudo apt-get  --purge remove <programname> | 卸载某个软件（purge表示彻底删除）                        |
| sudo apt-get  install <programname>        | 安装软件                                                 |
| sudo apt-get  clean                        | 清理安装包                                               |

 

## ubuntu 安装gcc g++ make

一个命令全部搞定：

```sh
apt install build-essential
```





## **控制台编码**

```shell
# 设置编码
export  LANG=zh_CN.gbk
# 查看编码
echo $LANG
```

 

## **VIM**

| 搜索              | 命令模式下,按‘/’然后输入要查找的字符之后再按Enter。          |
| ----------------- | ------------------------------------------------------------ |
| dd                | 删除一行                                                     |
| dw                | 删除光标后边的单词                                           |
| Shift+Backspace   | 删除光标前的一个字符                                         |
| 跳到某一行        | 在命令模式下输入行号n : n                                    |
| 撤销              | 命令模式 按 u（uu将恢复文本到最开始的状态）                  |
| 增加一行（Enter） | o (小写)                                                     |
| 翻页              | Ctrl+F（前一页） Ctrl+B（后一页）                            |
| 查找              | /SEARCH 注：正向查找，按n键把光标移动到下一个符合条件的地方； |
|                   | ?SEARCH 注：反向查找，按shift+n 键，把光标移动到下一个符合条件的 |
| 跳到文件头        | :1                                                           |
| 跳到文件尾        | G                                                            |
| readOnly保存文件  | wq!                                                          |
| 复制一行          | yy                                                           |
| 粘贴              | p                                                            |

 

## **nano编辑器**

| ctrl+o | 保存 |
| ------ | ---- |
| ctrl+x | 退出 |



## **查看软件信息**

| 命令          | 作用                             |
| ----------------- | ------------------------------------------------------------ |
| uname -a          | 可显示电脑以及操作系统的相关信息                             |
| cat /proc/version | 说明正在运行的内核版本                                       |
| cat /etc/issue    | 显示的是发行版本信息                                         |
| lsb_release -a    | 适用于所有的linux，包括Redhat、SuSE、Debian等发行版，但是在debian下要安装lsb |
| man [命令]        | 如果不知道命令的意思.可以通过  "man 命令"可以查看它的使用方式.及详细信息 |

 

## **查看硬件信息**

| lscpu   | 查看的是cpu的统计信息 |
| ------- | --------------------- |
| free -h | 以可读方式显示内存    |

 

## **扫描**[**端口**](https://blog.51cto.com/990487026/1907044)

| nmap   | Nmap  localhost             |
| ------ | --------------------------- |
| nc     | nc 192.168.20.166 8080      |
| telnet | telnet  192.168.20.166 8080 |



 

## **端口转发（iptables）**

\# 添加转发（本地22222端口转发到x.x.x.x:22）

```shell
iptables -t nat -A PREROUTING -p tcp --dport 22222 -j DNAT --to x.x.x.x:22
```

\# 查看转发

```shell
iptables -L -n --line-number
```

\# 删除转发（删除查看到的第几行）

```shell
iptables -D INPUT 1
```

 

 

## **网络抓包**

**tcpdump**

采用命令行方式对接口的数据包进行筛选抓取，其丰富特性表现在灵活的表达式上。

命令行直接输入：tcpdump 即可查看抓包情况。

 

## **查找文件**

| 查找目录 | find /（查找范围） -name  '查找关键字' -type d |
| -------- | ---------------------------------------------- |
| 查找文件 | find /（查找范围） -name  查找关键字 -print    |

> 示例：查找目录下的xml：find /home/ubuntu/soft/apache-tomcat-9.0.0.M13/ -name *.xml




## **查看(超)链接的目标文件**

```shell
# ls -al {文件名}
ls -al default
```

 

## **查看进程/端口**

```shell
# 查找系统进程
ps -ef | grep java

# 查找端口情况
netstat  -ant |grep 4243
```

 

## **软件安装**

```shell
# 安装RPM软件包
rpm -i 需要安装的包文件名
```




## **后台运行**

```shell
#! /bin/sh
nohup java -server -jar ./app.jar >/dev/null 2>&1 &
```

> 注：>/dev/null 2>&1 表示不输出运行日志

 

## **执行.sh文件**

执行文件之前必须使用 chmod +x 或者 chmod 777 命令使得文件可以被运行，否则提示权限不够或者命令找不到。

 

## **建立shell脚本**

Linux下有很多不同的shell,但我们通常使用bash(bourne again shell)进行编程，因为bash是免费的并且很容易使用

程序必须以下面的行开始（必须方在文件的第一行）：

```shell
#! /bin/sh
```

符号#!用来告诉系统它后面的参数是用来执行该文件的程序。在这个例子中我们使用/bin/sh来执行程序。

当编辑好脚本时，要想执行脚本，**必须**使脚本可以执行下面的命令，可以使脚本可以执行：

```shell
chmod +x filename
```

然后可以输入./filename来执行脚本。

> **注**：在shell编程时，#符号表示注释，只该行结束为止。在编写程序时，最好使用注释。



## **变量**

shell下所有变量都以字符串表示，变量不需要声明，直接使用。直接对变量进行赋值

```
A="hello world"
```

取出变量用$符号，如:

```shell
#! /bin/sh
A="hello world"
echo "A is:"
echo $A
```

执行该脚本输出结果如下：

```
A is :
hello world
```

 

## **shell** **技巧和流程控制**

**管道和重定向**

这些不是系统命令，但他们经常使用，很重要：

管道符号| 将一个命令的输出作为另外一个命令的输入

grep -qa compat | more

**重定向**：将命令的结果输出到文件，而不是标准输出(屏幕)

〉写入文件并覆盖旧文件

〉〉输出追加到文件的尾部，保留旧文件。

 

**流程控制**

```shell
if ... ; then
...
else if ...;then
...
else
...
fi
```

通常情况下，可以通过测试命令来对条件进行测试，比如可以比较字符串，判断文件是否存在及是否有执行权限等等

通常用“ [ ] “来表示条件测试，注意这里空格很重要，要确保方括号空格

- [ -f "somefile" ] ：判断是否是一个文件

- [ -x "/bin/ls" ] ：判断/bin/ls是否存在并有可执行权限
- [ -n "$var" ] ：判断$var变量是否有值

- [ "$a" = "$b" ] ：判断$a和$b是否相等

 

**调试**

最简单的调试命令当然是使用echo命令。您可以使用echo在任何怀疑出错的地方打印任何变量值。这也是绝大多数的shell程序员要花费80% 的时间来调试程序的原因。Shell程序的好处在于不需要重新编译，插入一个echo命令也不需要多少时间。

shell也有一个真实的调试模式。如果在脚本"strangescript" 中有错误，您可以这样来进行调试：

```shell
sh -x strangescript
```

这将执行该脚本并显示所有变量的值。shell还有一个不需要执行脚本只是检查语法的模式。可以这样使用：

```shell
sh -n your_script
```

这将返回所有语法错误。

 

 

## **定时任务**

**cron调度进程**

cron是系统主要的调度进程，可以在无需人工干预的情况下运行作业。有一个叫做crontab的命令允许用户提交、编辑或删除相应的作业。每一个用户都可以有一个crontab文件来保存调度信息。可以使用它运行任意一个shell脚本或某个命令，每小时运行一次，或一周三次，这完全取决于你。每一个用户都可以有自己的crontab文件，但在一个较大的系统中，系统管理员一般会禁止这些文件，而只在整个系统保留一个这样的文件。系统管理员是通过cron.deny和cron.allow这两个文件来禁止或允许用户拥有自己的crontab文件。

 

**crontab的域**

为了能够在特定的时间运行作业，需要了解crontab文件每个条目中各个域的意义和格式。下面就是这些域：

- 第1列 分钟1～59

- 第2列 小时1～23（0表示子夜）

- 第3列 日1～31

- 第4列 月1～12

- 第5列 星期0～6（0表示星期天）

- 第6列 要运行的命令




**范例格式**

```
分< >时< >日< >月< >星期< >要运行的命令
```

> 其中< >表示空格

crontab文件的一个条目是从左边读起的，第一列是分，最后一列是要运行的命令，它位于星期的后面。

在这些域中，可以用横杠-来表示一个时间范围，例如你希望星期一至星期五运行某个作业，那么可以在星期域使用1 - 5来表示。还可以在这些域中使用逗号“,”，例如你希望星期一和星期四运行某个作业，只需要使用1 , 4来表示。可以用星号*来表示连续的时间段。如果你对某个表示时间的域没有特别的限定，也应该在该域填入*。该文件的每一个条目必须含有5个时间域，而且每个域之间要用空格分隔。该文件中所有的注释行要在行首用#来表示。

**crontab条目举例**

```shell
30 21* * * /apps/bin/cleanup.sh
# 上面的例子表示每晚的21 : 30运行/apps/bin目录下的cleanup.sh。

45 4 1,10,22 * * /apps/bin/backup.sh
# 上面的例子表示每月1、1 0、2 2日的4 : 45运行/apps/bin目录下的backup.sh。

10 1 * * 6,0 /bin/find -name "core" -exec rm {} ;
# 上面的例子表示每周六、周日的1 : 10运行一个find命令。

0,30 18-23 * * * /apps/bin/dbcheck.sh
# 上面的例子表示在每天18 : 00至23 : 00之间每隔3 0分钟运行/ apps/bin目录下的dbcheck.sh。

0 23 * * 6 /apps/bin/qtrend.sh
# 上面的例子表示每星期六的11 : 00 p m运行/ apps/bin目录下的qtrend.sh。

```

```shell
每隔5秒执行一次：*/5 * * * * ?

每隔1分钟执行一次：0 */1 * * * ?

每天23点执行一次：0 0 23 * * ?

每天凌晨1点执行一次：0 0 1 * * ?

每月1号凌晨1点执行一次：0 0 1 1 * ?

每月最后一天23点执行一次：0 0 23 L * ?

每周星期六凌晨1点实行一次：0 0 1 ? * L

在26分、29分、33分执行一次：0 26,29,33 * * * ?

每天的0点、13点、18点、21点都执行一次：0 0 0,13,18,21 * * ?
```


> **注意**：你可能已经注意到上面的例子中，每个命令都给出了绝对路径。当使用crontab运行shell脚本时，要由用户来给出脚本的绝对路径，设置相应的环境变量。记住，既然是用户向cron提交了这些作业，就要向cron提供所需的全部环境。不要假定cron知道所需要的特殊环境，它其实并不知道。所以你要保证在shell脚本中提供所有必要的路径和环境变量，除了一些自动设置的全局变量。

 

**命令形式**

crontab命令的一般形式为：

```shell
crontab [-u user] -e -l -r
```

其中：

- -u 用户名。
- -e 编辑crontab文件。

- -l 列出crontab文件中的内容。

- -r 删除crontab文件。

 **-u** 

如果使用自己的名字登录，就不用使用-u选项，因为在执行crontab命令时，该命令能够知道当前的用户建一个新的crontab文件。

 **-l **

为了列出crontab文件，可以用：

```
crontab -l
```

你将会看到和上面类似的内容。可以使用这种方法在$HOME目录中对crontab文件做一备份：

$ crontab -l > $HOME/mycron

这样，一旦不小心误删了crontab文件，可以用上一节所讲述的方法迅速恢复。

  **-e** 

如果希望添加、删除或编辑crontab文件中的条目，而EDITOR环境变量又设置为vi，那么就可以用vi来编辑crontab文件，相应的命令为：

```
crontab -e
```

可以像使用vi编辑其他任何文件那样修改crontab文件并退出。如果修改了某些条目或添加了新的条目，那么在保存该文件时， cron会对其进行必要的完整性检查。如果其中的某个域出现了超出允许范围的值，它会提示你。

保存并退出。最好在crontab文件的每一个条目之上加入一条注释，这样就可以知道它的功能、运行时间，更为重要的是，知道这是哪位用户的作业。可以使用前面讲过的crontab -l命令列出它的全部信息。

  **-r** 

为了删除crontab文件，可以用：

```
crontab -r
```

 

 

## **开启cron日志**

有好多小伙伴可能找不到cron的[日志](http://www.letuknowit.com/post/101.html)，那是因为Ubuntu系统默认是不打开cron日志的，不信你cd 到/var/log目录下是找不到cron.log文件的。

\# 如何打开，很简单，控制台输入以下命令，打开文件，在文件中找到cron.*，把前面的#去掉，保存退出

```shell
vi /etc/rsyslog.d/50-default.conf
```

\# 重启系统日志

```shell
sudo service rsyslog restart
```

 

 

## **创建一个定时任务**

1、新建一个shell命令，内包含需要定时执行的命令语句，例如外面的backup.sh；【注意给新建的.sh文件赋予权限 chmod 777 **xxx**.sh】

2、新建crontab文件，例如：crontab，并编辑执行内容为上面所创建的shell命令文件，注意：crontab中使用的所有路径都用绝对路径；

3、将crontab提交给cron进程： #crontab 【crontab文件名】

**注意：**

**权限问题：**遇到**没有权限问题**时。切换至root用户 ，执行chmod 755 XXX.sh即可

**无法识别：**如果遇到**执行时无法识别**的命令，则在编写的shell命令内添加各种环境变量即可，例如db2初始化环境：

```
#初始化db2环境
if [ -f ${HOME}/sqllib/db2profile ]; then
  . ${HOME}/sqllib/db2profile
fi
```

等等

**ubuntu环境：**ubuntu环境下注意

1. ubuntu下crontab的服务程序是cron，并且默认cron服务的log是没有的，我们必须手动开启

​     a. sudo vim /etc/rsyslog.d/50-default.conf

​     b. 找到cron.*那一行把注释去掉

​     c. 然后重启cron服务 sudo service cron restart

​     d. 这样就可以在/var/log里面发现有cron的日志文件了，我们就可以通过查看日志文件找到问题所在

2. ubuntu下，用户home目录下是没有.bash_profile文件的，并且会自动去执行.bashrc文件，只要写成下面这样即可

```
*/2 * * * * sh /home/chenguolin/tmp/s.sh >/dev/null 2>&1
```

 

**Shell脚本代码片段**

```shell
#日期相关
DATE=`date +%Y%m%d`
hh=`date +%H`
mm=`date +%M`
ss=`date +%S`
now=$DATE$hh$mm$ss
**echo** $now
```

 

```shell
# 打包14天以前的记录
tar -czvf `date +%Y%m%d`_TASK_log_backup.tar `find . -mtime +14
```

 

```shell
# 删除14天以前的记录
find . -mtime +14 | grep $DBNAME | xargs rm -f
```

 

```shell
# 列出指定进程号所打开的文件
lsof -p 进程号
```


## **shell脚本示例**

**日志备份和删除**

 [05 01 * * 0     /home/app/TaskLogBack.sh >> /home/app/TaskLogBack.log] **[每周日1点05分]**

```shell
#!/bin/sh
logFromPath=/home/tomcat/task/logs/
logToPath=/home/tomcat/backup/log/
backupFileName=`date +%Y%m%d`_TASK_log_backup.tar
#当前时间
now=$DATE$hh
cd $logFromPath
echo "-------------------------------------------------------------"
echo "当前日期为："`date +%Y%m%d`
echo "开始备份14天之前的文件："
echo "待备份的文件："`find . -mtime +14`|tr ' ' '\n'
#清除压缩文件
#rm $backupFileName
tar -czvf $backupFileName `find . -mtime +14`
mv $logFromPath$backupFileName $logToPath
echo "备份完成，备份到的路径和文件名为："$logToPath$backupFileName
#删除14天以前的记录
find . -mtime +14 | xargs rm -f
echo "已清除14天以前的日志文件"
echo "-------------------------------------------------------------"
```



**日志备份和清空** 

[00 00 * * * /home/ubuntu/app/rmlog.sh >> /home/ubuntu/app/task.log] **[每天00点00分]**

```shell
#!/bin/sh
DATE=`date +%Y%m%d`
hh=`date +%H`
mm=`date +%M`
ss=`date +%S`
logFromPath=/home/ubuntu/soft/apache-tomcat-9.0.0.M13/logs/
logToPath=/home/ubuntu/app/logs/
#当前时间
now=$DATE$hh
#日志路径
logName="catalina.out"
cp $logFromPath$logName $logToPath$now$logName
cd $logToPath
tar -czvf $now$logName.tar $now$logName
rm $now$logName
echo "文件"$logToPath$now$logName"已备份压缩"
cd $logFromPath
>$logName
echo $logFromPath$logName"已清空"
#删除14天以前的记录
find . -mtime 7 | grep $logName | xargs rm -f
```



**造数据到文件（redis）**

```shell
#/bin/bash
for((i=0;i<10000;i++))
do
echo -en "set user"$i "modhello"$i "\n">>redis.log
done
echo "done"
```

 

**自动部署**

```shell
#!/bin/sh
# 备份和替换生产工程
_prod_home_path=/home/tomcat
_prod_path=$_prod_home_path"/appfile-uat/loan/"
_restart_sh_path=$_prod_home_path"/appfile-uat/"
_prod_back_path=$_prod_home_path"/backup/"
_prod_target_file=cfs.war
_prod_back_file=`date +%Y%m%d``date +%H%M%S`_loan_backup.tar
echo "-------------------------------------------------------------"
echo "备份文件路径："$_prod_path
cd $_prod_path
echo "开始备份生产web程序"
tar -czf $_prod_back_file * --exclude=$_prod_target_file
echo "移动备份文件到备份目录："$_prod_back_path$_prod_back_file
mv $_prod_back_file $_prod_back_path
echo "当前目录："`pwd`

echo "开始解压目标文件："$_prod_target_file
unzip -o $_prod_target_file
echo "目标文件解压完成"
echo "清理目标文件"
rm $_prod_target_file
echo "生产工程已备份替换完成"
echo "-------------------------------------------------------------"
# 重启web服务
echo "开始重启192.168.10.3 web服务器"
_SERVER_PATH=/home/tomcat/apache-tomcat-7.0.68-uat
# 重启服务
echo "开始准备重启web服务"
cd $_SERVER_PATH/bin
# 关闭所有tomcat进程
while [ `ps -ef | grep $_SERVER_PATH | wc -l` != '1' ]; do
	echo "正在关闭tomcat服务"
	./shutdown.sh
	sleep 2
done
echo "开始启动tomcat服务"
./startup.sh
echo "服务重启完成"
echo "重启192.168.10.3 web服务器完成"
echo "--------------------------------------------------------------"
```

