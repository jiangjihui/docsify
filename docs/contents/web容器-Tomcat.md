## Tomcat处理[资源](https://www.zhihu.com/question/57400909/answer/154753720)

Tomcat访问所有的资源，都是用Servlet来实现的。所以Tomcat又叫Servlet容器嘛，什么都交给Servlet来处理。

在Tomcat看来，资源分3种：

1. 静态资源，如css,html,js,jpg,png等。交由DefaultServlet类处理

2. Servlet，交由InvokerServlet类处理

3. JSP，交由JspServlet类来处理

**那么什么时候调用哪个Servlet呢？**

有一个类叫做org.apache.tomcat.util.http.mapper.Mapper，它一共进行了7个大的规则判断，第7个，就是判断是否是该用DefaultServlet。

简单地说：先看是不是servlet,然后看是不是jsp，如果都不是，那么就是你DefaultServlet的活儿了。

到了DefaultServlet之后，就是一个普通的HttpServlet了，doPost方法会交由doGet处理，doGet又交由一个叫做 serveResource的方法处理，在serveResource方法里又瞎搞八搞了许多事情，最后在一个叫做copy()方法里，把静态资源对应的输入流读取出来，扔到了输出流里，这样你的浏览器就看到数据了。

## 相关概念

### ServletContext

ServletContext 被 Servlet 程序用来与 Web 容器通信。例如写日志，转发请求。每一个 Web 应用程序含有一个Context，被Web应用内的各个**程序共享**。因为Context可以用来保存资源并且共享，所以我所知道的 [ServletContext](https://blog.csdn.net/gavin_john/article/details/51399425) 的最大应用是Web缓存----把不经常更改的内容读入内存，所以服务器响应请求的时候就不需要进行慢速的磁盘I/O了。

**创建**

- WEB容器在启动时，它会为每个Web应用程序都创建一个对应的ServletContext，它代表当前Web应用。并且它被所有客户端共享。
- ServletContext对象可以通过ServletConfig.getServletContext()方法获得对ServletContext对象的引用，也可以通过this.getServletContext()方法获得其对象的引用。
- 由于一个WEB应用中的所有Servlet共享同一个ServletContext对象，因此Servlet对象之间可以通过ServletContext对象来实现通讯。ServletContext对象通常也被称之为context域对象。公共聊天室就会用到它。
- 当web应用关闭、Tomcat关闭或者Web应用reload的时候，ServletContext对象会被销毁

**应用场景** 

1. 网站计数器 

2. 网站的在线用户显示 

3. 简单的聊天系统



### 线程池

tomcat线程池与java线程池不一样，tomcat线程池其实**就是连接池**，主要用于处理网络请求。

**一、线程池参数**

1. 核心线程数：**默认**情况下，Tomcat 的**核心线程数**一般为 **10**。这意味着在没有高负载的情况下，线程池会保持至少 10 个线程处于活动状态，随时准备处理请求。
2. 最大线程数：Tomcat 的默认**最大线程数**通常为 **200**。当请求量增加时，线程池可以创建的最大线程数量为 200。这有助于在高负载情况下处理大量并发请求，但同时也需要考虑系统资源的限制。
3. 线程空闲超时时间：默认情况下，如果一个线程在 60 秒内没有被使用，它将会被回收。这个超时时间可以根据实际应用的需求进行调整，以平衡系统资源的使用和响应时间。

**二、工作原理**

当请求到达 Tomcat 服务器时，Tomcat 会从线程池中分配一个线程来处理该请求。如果当前线程池中没有可用的线程，并且线程数量未达到最大线程数，Tomcat 会创建新的线程来处理请求。这点与java线程池先放入等待队列的逻辑不一样。当请求处理完成后，线程会返回到线程池中，等待下一个请求的到来。如果线程在一段时间内没有被使用，它可能会被回收，以释放系统资源。



## Tomcat调优

[调优](https://blog.csdn.net/wangyonglin1123/article/details/50986524)

Tomcat的优化分成两块：

1. Tomcat启动命令行中的优化参数即JVM优化
2. Tomcat容器自身参数的优化（这块很像ApacheHttp     Server）

**1** **启动行参数优化**

Tomcat 的启动参数位于tomcat的安装目录\bin目录下的catalina.sh文件，在最开始注释文字的最后一段也就是真正有效的shell命令开始的那里加入如下的参数

jdk1.7：

```
export JAVA_OPTS="-server -Xms1400M -Xmx1400M -Xss512k -XX:+AggressiveOpts -XX:+UseBiasedLocking -XX:PermSize=128M -XX:MaxPermSize=256M -XX:+DisableExplicitGC -XX:MaxTenuringThreshold=31 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC  -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m  -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -Djava.awt.headless=true "
```

jdk1.8：

```
export JAVA_OPTS="-server -Xms2G -Xmx2G -Xmn512m -XX:MetaspaceSize=512M -XX:MaxMetaspaceSize=512M -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+HeapDumpOnOutOfMemoryError -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:/appl/gc.log -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly "
```

> **参数解释**
> 
> **-server**  
> 只要你的tomcat是运行在生产环境中的，这个参数必须加上
> 
> **-Xms** **–Xmx**  
> 把Xms与Xmx两个值设成一样是最优的做法
> 
> **–Xmn**  
> 设置年轻代大小为512m。整个堆大小=年轻代大小 + 年老代大小 + 持久代大小。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。
> 
> **-Xss**  
> 是指设定每个线程的堆栈大小。这个就要依据你的程序，看一个线程 大约需要占用多少内存，可能会有多少线程同时运行等。一般不易设置超过1M，要不然容易出现out ofmemory。
> 
> **-XX:+AggressiveOpts**  
> 作用如其名（aggressive），启用这个参数，则每当JDK版本升级时，你的JVM都会使用最新加入的优化技术（如果有的话）
> 
> **-XX:+UseBiasedLocking**  
> 启用一个优化了的线程锁，我们知道在我们的appserver，每个http请求就是一个线程，有的请求短有的请求长，就会有请求排队的现象，甚至还会出现线程阻塞，这个优化了的线程锁使得你的appserver内对线程处理自动进行最优调配。
> 
> **-XX:PermSize=128M-XX:MaxPermSize=256M**  
> XX:PermSize设置非堆内存初始值，默认是物理内存的1/64；在数据量的很大的文件导出时，一定要把这两个值设置上，否则会出现内存溢出的错误。
> XX:MaxPermSize设置最大非堆内存的大小，默认是物理内存的1/4。
> 如果是物理内存4GB，那么64分之一就是64MB，这就是PermSize默认值，也就是永生代内存初始大小；四分之一是1024MB，这就是MaxPermSize默认大小。
> 
> **-XX:+DisableExplicitGC**  
> 在程序代码中不允许有显示的调用”System.gc()”。看到过有两个极品工程中每次在DAO操作结束时手动调用System.gc()一下，觉得这样 做好像能够解决它们的out ofmemory问题一样，付出的代价就是系统响应时间严重降低，就和我在关于Xms,Xmx里的解释的原理一样，这样去调用GC导致系统的JVM大起大落
> 
> **-XX:+UseParNewGC**  
> 对年轻代采用多线程并行回收，这样收得快。
> 
> **-XX:+UseConcMarkSweepGC**  
> 即CMS gc，这一特性只有jdk1.5即后续版本才具有的功能，它使用的是gc估算触发和heap占用触发。
> 
> **-XX:MaxTenuringThreshold**  
> 设 置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一 个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概率。
> 
> 这个值的设置是根据本地的jprofiler监控后得到的一个理想的值，不能一概而论原搬照抄。
> 
> **-XX:LargePageSizeInBytes**  
> 指定Java heap的分页页面大小
> 
> **-XX:+UseFastAccessorMethods**  
> get,set 方法转成本地代码
> 
> **-Djava.awt.headless=true**  
> 这 个参数一般我们都是放在最后使用的，这全参数的作用是这样的，有时我们会在我们的J2EE工程中使用一些图表工具如：jfreechart，用于在web 网页输出GIF/JPG等流，在winodws环境下，一般我们的app server在输出图形时不会碰到什么问题，但是在linux/unix环境下经常会碰到一个exception导致你在winodws开发环境下图片显示的好好可是在linux/unix下却显示不出来，因此加上这个参数以免避这样的情况出现。

上述这样的配置，基本上可以达到：

- 系统响应时间增快
- JVM回收速度增快同时又不影响系统的响应率
- JVM内存最大化利用
- 线程阻塞情况最小化

**2** **容器优化**

前面我们对Tomcat启动时的命令进行了优化，增加了系统的JVM可使用数、垃圾回收效率与线程阻塞情况、增加了系统响应效率等还有一个很重要的指标，我们没有去做优化，就是吞吐量。

打开tomcat安装目录\conf\server.xml文件，定位到这一行：

```
<connector port="8080" protocol="HTTP/1.1" <="" p="" style="word-wrap: break-word;">
```

这一行就是我们的tomcat容器性能参数设置的地方，它一般都会有一个默认值，这些默认值是远远不够我们的使用的，我们来看经过更改后的这一段的配置：

```
<connector  port="8080" protocol="HTTP/1.1" <=""  p="" style="word-wrap: break-word;">         URIEncoding="UTF-8" minSpareThreads="25"  maxSpareThreads="75"         enableLookups="false" disableUploadTimeout="true"  connectionTimeout="20000"         acceptCount="300" maxThreads="300"  maxProcessors="1000" minProcessors="5"         useURIValidationHack="false"                           compression="on" compressionMinSize="2048"                           compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain"           redirectPort="8443"  />  
```

> **URIEncoding=”UTF-8”**  
> 使得tomcat可以解析含有中文名的文件的url
> 
> **maxSpareThreads**  
> maxSpareThreads 的意思就是如果空闲状态的线程数多于设置的数目，则将这些线程中止，减少这个池中的线程总数。
> 
> **minSpareThreads**  
> 最小备用线程数，tomcat启动时的初始化的线程数。
> 
> **enableLookups**  
> 这个功效和Apache中的HostnameLookups一样，设为关闭。
> 
> **connectionTimeout**  
> connectionTimeout为网络连接超时时间毫秒数。
> 
> **maxThreads**  
> maxThreads Tomcat使用线程来处理接收的每个请求。这个值表示Tomcat可创建的最大的线程数，即最大并发数。
> 
> **acceptCount**  
> acceptCount是当线程数达到maxThreads后，后续请求会被放入一个等待队列，这个acceptCount是这个队列的大小，如果这个队列也满了，就直接refuse connection
> 
> **maxProcessors与minProcessors**  
> 在 Java中线程是程序运行时的路径，是在一个程序中与其它控制线程无关的、能够独立运行的代码段。它们共享相同的地址空间。多线程帮助程序员写出CPU最大利用率的高效程序，使空闲时间保持最低，从而接受更多的请求。通常Windows是1000个左右，Linux是2000个左右。
> 
> **useURIValidationHack**  
> 如果把useURIValidationHack设成"false"，可以减少它对一些url的不必要的检查从而减省开销。
> 
>  **enableLookups="false"**  
> 为了消除DNS查询对性能的影响我们可以关闭DNS查询，方式是修改server.xml文件中的enableLookups参数值。
> 
> **disableUploadTimeout**  
> 类似于Apache中的keeyalive一样
> 
> **compression="on" compressionMinSize="2048"** **...**  
> 给Tomcat配置gzip压缩(HTTP压缩)功能，HTTP 压缩可以大大提高浏览网站的速度，它的原理是，在客户端请求网页后，从服务器端将网页文件压缩，再下载到客户端，由客户端的浏览器负责解压缩并浏览。相对 于普通的浏览过程HTML,CSS,Javascript , Text ，它可以节省40%左右的流量。更为重要的是，它可以对动态生成的，包括CGI、PHP , JSP , ASP , Servlet,SHTML等输出的网页也能进行压缩，压缩效率惊人。
> 
> 1. compression="on"  on：表示允许压缩（文本将被压缩）、force：表示所有情况下都进行压缩，默认值为off
> 
> 2. compressionMinSize="2048" 启用压缩的输出内容大小，这里面默认为2KB
> 
> 3. noCompressionUserAgents="gozilla, traviata" 对于以下的浏览器，不启用压缩
> 
> 4. compressableMimeType="text/html,text/xml"　压缩类型

## Tomcat的[运行模式](http://tyrion.iteye.com/blog/2256896)

**bio：** (blocking I/O)，阻塞式I/O操作，一般而言，表示Tomcat使用的是传统的Java I/O操作。bio模式是三种运行模式中性能最低的一种。

**nio：** [nio](https://www.cnblogs.com/nizuimeiabc1/p/8934185.html)(new I/O)，是Java SE 1.4及后续版本提供的一种新的I/O操作方式(即Java.nio包及其子包)。java nio是一个基于缓冲区、并能提供非阻塞I/O操作的Java API，因此nio也被看成是non-blocking I/O的缩写。它拥有比传统I/O操作(bio)更好的并发运行性能。关于nio与bio在tomcat上的差别可以参考这个[简书](https://www.jianshu.com/p/76ff17bc6dea)：NIO只是优化了网络IO的读写，如果系统的瓶颈不在这里，比如每次读取的字节说都是500b，那么BIO和NIO在性能上没有区别。（个人感觉与socket的nio一样，并发小于1000时，区别不大，而并发很多的时候，更多的是使用nginx）

**apr：** (Apache Portable Runtime/Apache可移植运行时)，是Apache HTTP服务器的支持库。可以简单地理解为Tomcat将以JNI的形式调用Apache HTTP服务器的核心动态链接库来处理文件读取或网络传输操作，从而大大地提高Tomcat对静态文件的处理性能。 Tomcat apr也是在Tomcat上运行高并发应用的首选模式。

修改运行模式为NIO模式：修改server.xml里的Connector节点，修改protocol为org.apache.coyote.http11.Http11NioProtocol

> **注：**通过对tomcat几个版本的测试，tomcat7默认启动是bio模式，需要优化；tomcat8及8以上默认是nio模式，无需改变运行模式，如果是windows版本的话，包里面还默认携带一个[tcnative-1.dll](http://www.365mini.com/page/tomcat-connector-mode.htm)，默认就是在Tomcat apr模式下运行。

当用nginx和tomcat做企业级集群的时候，需要禁用掉AJP协议。

## Tomcat版本

通过对比发现，**Linux**版本的Tomcat和**Windows**相比，是可以互换的，windows版本包含全部linux版本的文件，只是在windows的版本中多加了2个.exe文件和一个tcnative-1.dll文件。

## 编码设置

Tomcat 的编码通常在两个地方可以配置，一是conf/server.xml，二是bin/catalina.sh

**conf/server.xml**

```
<Connector executor="tomcatThreadPool"
    port="8088" protocol="HTTP/1.1"
    redirectPort="8443" URIEncoding="GBK"/>
```

**bin/catalina.sh**

```
JAVA_OPTS="${JAVA_OPTS} -Dfile.encoding=GBK"
```

## Tomcat开启Gzip

修改Tomcat目录下的/conf/server.xml，添加[字段](https://www.cnblogs.com/DDgougou/p/8675504.html)：

```xml
<Connector port="9092" protocol="HTTP/1.1"
   connectionTimeout="20000"
   redirectPort="8443" 
   compression="on"
   compressionMinSize="2048"
   compressableMimeType="text/html,text/xml,text/javascript,application/javascript,text/css,text/plain,text/json,application/json"
   />
```

参数说明：

1、compression="on" 开启压缩。可选值："on"开启，"off"关闭，"force"任何情况都开启。

2、compressionMinSize="2048"大于2KB的文件才进行压缩。用于指定压缩的最小数据大小，单位B，默认2048B。注意此值的大小，如果配置不合理，产生的后果是小文件压缩后反而变大了，达不到预想的效果。

3、noCompressionUserAgents="gozilla, traviata"，对于这两种浏览器，不进行压缩（我也不知道这两种浏览器是啥，百度上没找到），其值为正则表达式，匹配的UA将不会被压缩，默认空。

4、compressableMimeType="text/html,text/xml,application/javascript,text/css,text/plain,text/json"会被压缩的MIME类型列表，多个逗号隔，表明支持html、xml、js、css、json等文件格式的压缩（plain为无格式的，但对于具体是什么，我比较概念模糊）。compressableMimeType很重要，它用来告知tomcat要对哪一种文件进行压缩。

**备注：**开启gzip后，返回头中会有 Content-Encoding: gzip，请求端基于Java的话，可直接使用RestTemplate解析，会自动解压gzip，返回解压后的数据。

## Tomcat的几种部署方式

**第一种：**

编辑tomcat/conf/server.xml下的<host/>节点中添加：

```xml
<Context path="/hello"
docBase="D:/eclipse3.2.2/forwebtoolsworkspacehello/WebRoot" debug="0"
privileged="true" >
</Context>
```

将web项目拷贝到tomcat/webapps下

**第三种：**

在conf目录中，新建Catalina\localhost目录，在该目录中新建一个xml文件，名字任意（不与本文件夹中的其他文件名重复），xml内容为：

```
<Context path="/hello"
docBase="D:/eclipse3.2.2/forwebtoolsworkspacehello/WebRoot" debug="0"
privileged="true" >
</Context>
```

注：这个方法的优点是可以定义别名。服务器端运行的项目名称为path，外部访问的URL则使用XML的文件名。这个方法很方便的隐藏了项目的名称，对于一些项目名称被固定不能更换，但外部访问时又想换个路径时非常有效。

**第四种：**

用tomcat在线的后台管理器，直接上传war包。