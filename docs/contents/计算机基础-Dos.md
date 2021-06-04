## **查看占用指定端口的程序** 

当你在用tomcat发布程序时，经常会遇到端口被占用的情况，我们想知道是哪个程序或进程占用了端口，可以用该命令 netstat –ano|findstr “指定端口号” 

```powershell
netstat -ano|findstr "8080"
```

 

 

## **进入文件目录**

windows下进入某个盘的文件目录前，需要确定当前是否已经在这个盘里，如果不在则需要输入盘符之后再使用cd 命令进入：

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

 

 

## **查看环境变量**

**set**：命令可查看所有环境变量

**set** **path**：查看某一个环境变量

```powershell
>set JAVA_HOME
>JAVA_HOME=C:\Program Files\Java\jdk1.8.0_111
```

echo %path%：查看某一环境变量

```powershell
>echo %JAVA_HOME%
>JAVA_HOME=C:\Program Files\Java\jdk1.8.0_111
```





## 脚本文件示例

### 运行Java程序

```powershell
chcp 65001

@echo off

java -server -jar -Duser.timezone=Asia/Shanghai -XX:-HeapDumpOnOutOfMemoryError D:\app\hro\app.jar --spring.profiles.active=prod
```



