---
layout: post
title: Linux下jdk和tomcat的安装与配置
date: 2017-04-06
tags: 服务器    
---
### 查看环境变量
which java    
echo $JAVA_HOME  
echo $PATH  

### 安装
选择要安装java的位置，如将下载的jdk.tar.gz拷贝到/usr/目录下<br/>
选择要安装tomcat的位置，如将下载的tomcat.tar.gz拷贝到/usr/local目录下<br/>
然后分别解压两个文件：      <br/>
tar -zxvf jdk.tar.gz     <br/>
tar -zxvf tomcat.tar.gz

### 设置变量
vim /etc/profile
在最后面添加如下内容：
```java
# Java
JAVA_HOME=/usr/jdk
CLASSPATH=.:$JAVA_HOME/lib.tools.jar
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH PATH

# Tomcat
TOMCAT_HOME=/usr/local/tomcat
CATALINA_HOME=/usr/local/tomcat
export TOMCAT_HOME CATALINA_HOME
```
使变量生效  <br/>
source /etc/profile

### 验证是否成功
java -version  <br/>
启动tomca后，```ps -ef | grep tomcat```

### 卸载
找到jdk安装目录的_uninst子目录,在shell终端执行命令./uninstall.sh即可卸载jdk。

### tomcat的配置
[开源中国的tomcat配置](https://git.oschina.net/oschina/oschina-config/blob/master/tomcat/catalina.sh?dir=0&filepath=tomcat%2Fcatalina.sh&oid=016f9880e109d18cb6dbb110fba740182b4ad805&sha=9a9089bfb36334045692aaf072f6e009d4fa5211)
```java
# 在 catalina.sh 中加入以下两行
CATALINA_OPTS="-Djava.awt.headless=true -Djava.net.preferIPv4Stack=true"
JAVA_OPTS="-server -Xms4096m -Xmx4096m"
```
看到上面的配置后，对里面的几项有些疑惑，总结下：
* -Djava.net.preferIPv4Stack=true

如果系统中开启了IPV6协议（比如window7），java网络编程经常会获取到IPv6的地址，这明显不是我们想要的结果，搜索发现很多做法是：禁止IPv6协议。其实查看官方文档有详细的说明：
> **java.net.preferIPv4Stack (default: false)**
>If IPv6 is available on the operating system the underlying native socket
>will be an IPv6 socket. This allows Java(tm) applications to connect too, >and accept connections from, both IPv4 and IPv6 hosts.
>If an application has a preference to only use IPv4 sockets then this
>property can be set to true. The implication is that the application will >not be able to communicate with IPv6 hosts.
 
所以可在 catalina.bat 或者 catalina.sh 中加入上面的配置即可。<br/>

* -Djava.awt.headless=true

java在图形处理时调用了本地的图形处理库。在利用Java作图形处理（比如：图片缩放，图片签名，生成报表）时，如果运行在windows上不会出问题。但由于Linux/Unix服务器上，一般没有图形界面，所以会抛出java.awt.HeadlessException，此时加入上面的配置就可以了。

#### **Tomcat Connector**
Tomcat Connector有bio、nio、apr三种运行模式：
 * bio(blocking I/O)，顾名思义，即阻塞式I/O操作，表示Tomcat使用的是传统的Java I/O操作(即java.io包及其子包)。
 * nio(new I/O)，是Java SE 1.4及后续版本提供的一种新的I/O操作方式(即java.nio包及其子包)。Java nio是一个基于缓冲区、并能提供非阻塞I/O操作的Java API，因此nio也被看成是non-blocking I/O的缩写。它拥有比传统I/O操作(bio)更好的并发运行性能。要让Tomcat以nio模式来运行也比较简单，我们只需要在Tomcat安装目录/conf/server.xml文件中将如下配置：   
 
```java
<Connector port="8080" protocol="HTTP/1.1"
connectionTimeout="20000"
redirectPort="8443" />
```

中的protocol属性值改为org.apache.coyote.http11.Http11NioProtocol即可：
```java
<Connector port="8080" 
protocol="org.apache.coyote.http11.Http11NioProtocol"
connectionTimeout="20000"
redirectPort="8443" />
```

* apr(Apache Portable Runtime)，是Apache HTTP服务器的支持库。你可以简单地理解为，Tomcat将以JNI的形式调用Apache HTTP服务器的核心动态链接库来处理文件读取或网络传输操作，从而大大地提高Tomcat对静态文件的处理性能。 Tomcat apr也是在Tomcat上运行高并发应用的首选模式。如果我们的Tomcat不是在apr模式下运行，在启动Tomcat的时候，我们可以在日志信息中看到类似如下信息：
```java
09-Apr-2017 09:34:10.857 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent The APR based Apache Tomcat Native library
 which allows optimal performance in production environments was not found on the java.library.path: /usr/java/packages/lib/amd64:/usr/lib64
:/lib64:/lib:/usr/lib
```

* 区分Tomcat Connector当前的运行模式<br/>
 如果以不同的Connector模式启动，在Tomcat的启动日志信息中一般会包含类似如下的不同内容，我们只需要根据这些信息即可判断出当前Tomcat的运行模式：
```java
bio
信息: Starting ProtocolHandler ["http-bio-8080"] 2013-8-6 16:17:50 org.apache.coyote.AbstractProtocol start
nio
信息: Starting ProtocolHandler ["http-nio-8080"] 2013-8-6 16:59:53 org.apache.coyote.AbstractProtocol start
apr
信息: Starting ProtocolHandler ["http-apr-8080"] 2013-8-6 17:03:07 org.apache.coyote.AbstractProtocol start
```

**我用的tomcat9，默认是nio模式，觉得够用了，就没有改动**

### java的配置
JAVA_OPTS ，顾名思义，是用来设置JVM相关运行参数的变量。        <br/>
JAVA_OPTS="-server -Xms2048m -Xmx2048m"                   <br/>
-Xms：初始Heap大小，使用的最小内存,cpu性能高时此值应设的大一些  <br/>
-Xmx：Java heap最大值，使用的最大内存。                      <br/>

这两个值的大小一般根据需要进行设置。初始化堆的大小执行了虚拟机在启动时向系统申请的内存的大小。一般而言，这个参数不重要。但是有的应用 程序在大负载的情况下会急剧地占用更多的内存，此时这个参数就是显得非常重要，如果虚拟机启动时设置使用的内存比较小而在这种情况下有许多对象进行初始 化，虚拟机就必须重复地增加内存来满足使用。由于这种原因，我们一般把-Xms和-Xmx设为一样大，而堆的最大值受限于系统使用的物理内存。一般使用数 据量较大的应用程序会使用持久对象，内存使用有可能迅速地增长。当应用程序需要的内存超出堆的最大值时虚拟机就会提示内存溢出，并且导致应用服务崩溃。因 此一般建议堆的最大值设置为可用内存的最大值的80%。  

另外需要考虑的是Java提供的垃圾回收机制。虚拟机的堆大小决定了虚拟机花费在收集垃圾上的时间和频度。收集垃圾可以接受的速度与应用有 关，应该通过分析实际的垃圾收集的时间和频率来调整。如果堆的大小很大，那么完全垃圾收集就会很慢，但是频度会降低。如果你把堆的大小和内存的需要一致， 完全收集就很快，但是会更加频繁。调整堆大小的的目的是最小化垃圾收集的时间，以在特定的时间内最大化处理客户的请求。在基准测试的时候，为保证最好的性 能，要把堆的大小设大，保证垃圾收集不在整个基准测试的过程中出现。   
　
如果系统花费很多的时间收集垃圾，请减小堆大小。一次完全的垃圾收集应该不超过 3-5 秒。如果垃圾收集成为瓶颈，那么需要指定代的大小，检查垃圾收集的详细输出，研究 垃圾收集参数对性能的影响。一般说来，你应该使用物理内存的 80% 作为堆大小。当增加处理器时，记得增加内存，因为分配可以并行进行，而垃圾收集不是并行的。
在重启Tomcat服务器之后，这些配置的更改才会有效

### setenv.sh
在tomcat，bin目录下，新建setenv.sh，写入下面命令，效果和上面通过修改catalina.sh一样的：<br/>
export CATALINA_OPTS="$CATALINA_OPTS -server -Xms2G -Xmx2G"
//8g内存分了2g
