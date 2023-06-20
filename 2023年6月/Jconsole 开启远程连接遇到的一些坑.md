>最近在学习 JVM，其中涉及到性能、内存等指标分析需要使用工具分享，Java 提供了几个可视化工具来监控和管理 Java 应用，比如 Jconsole、JVisual、JMC，他们以图形化的界面实时的监控程序各种性能指标以及内存、CPU 的使用情况。

Jconsole、JVisual、JMC 可视化工具，调用本地监控直接使用对应的命令行即可，但 Linux 无法使用可视化工具，Java 程序基本都部署到 Linux 服务器。需要本地**远程调用服务器**，本文记录一下远程调用的一些步骤和遇到的坑。

# JMX

JXM(Java Management Extensions) 是 Java 提供的一套标准 API,用于管理和监控 Java 应用程序的各种性能指标和使用情况。这里主要使用远程访问的功能。

JMX 启动参数：
* -Dcom.sun.management.jmxremote 远程开启开关
* -Dcom.sun.management.jmxremote.port=1808  jmx远程调用端口
* -Dcom.sun.management.jmxremote.authenticate=false 不开启验证
* -Dcom.sun.management.jmxremote.ssl=false 不为ssl连接
* -Djava.rmi.server.hostname=34.126.141.21 服务器所在ip或者域名

# 配置远程连接

启动 Java 程序一般有两种方式：

* 一是打成 jar 包，使用 java -jar 运行程序。
* 一种是打成 war 包，放在 tomcat 上运行。

无论是 jar 还是 war 包，都是将上面的配置参数用空格拼接起来，比如将上面的参数拼接：

```
Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=18088 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=34.126.141.21
```

添加到配置文件或者启动参数中。

## Java 程序启动

jar 包程序启动一般为：

```
java -jar xxx.jar
```

添加参数后：

```
java -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1808 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=34.126.141.21 -jar xxx.jar
```

## tomcat 启动

在启动文件 catalina.sh 里面添加:

```
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1808 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=34.126.141.21"
```

添加上面的配置之后，重启 tomcat，再使用 Jconsole 远程连接

使用 jconsole 远程连接，**一直连接不上**：

![](https://files.mdnice.com/user/29864/0904b820-883c-4523-8cd3-d710d1559782.png)

## 无法远程问题排查
 
先查看本地端口是否开启：

```
netstat -ntlp
```

端口已开启：

```
tcp6     0     0 :::1808          :::*              LISTEN      9087/java
```
再查看是否是防火墙问题，使用[端口扫描](https://tool.chinaz.com/port)查看，端口也开启了：

![](https://files.mdnice.com/user/29864/281658bd-cc0e-4f4e-9088-3a30ec8a1201.png)

**端口开启了，但是还是无法连接**

找了很多网上的答案，大家都是抄来抄去的，都是上面的配置。最后才发现少了 rmi 配置。

## 解决方案

添加 rmi 端口：

```
-Dcom.sun.management.jmxremote.rmi.port=1808
```

>JMX 和 RMI，是两种相关联的技术，JMX 使用 RMI 作为远程管理工具来管理和监控 Java 程序，RMI 为 JMX 提供了远程连接所需的远程调用和通信机制。

添加了上面的配置，就能远程监控 Java 服务了。

![](https://files.mdnice.com/user/29864/e8b051ac-1c8a-4a0f-9ff3-94fe721dadff.png)

# 关闭 tomcat 报错

tomcat 启动添加了配置之后，关闭 tomcat 服务时，就报错了：

```
sun.management.AgentConfigurationError: java.rmi.server.ExportException: Port already in use: 18088; nested exception is: 
	java.net.BindException: Address already in use (Bind failed)
	at sun.management.jmxremote.ConnectorBootstrap.exportMBeanServer(ConnectorBootstrap.java:800)
	at sun.management.jmxremote.ConnectorBootstrap.startRemoteConnectorServer(ConnectorBootstrap.java:468)
	at sun.management.Agent.startAgent(Agent.java:262)
	at sun.management.Agent.startAgent(Agent.java:452)
Caused by: java.rmi.server.ExportException: Port already in use: 18088; nested exception is: 
	java.net.BindException: Address already in use (Bind failed)
	at sun.rmi.transport.tcp.TCPTransport.listen(TCPTransport.java:346)
	at sun.rmi.transport.tcp.TCPTransport.exportObject(TCPTransport.java:254)
	at sun.rmi.transport.tcp.TCPEndpoint.exportObject(TCPEndpoint.java:412)
	at sun.rmi.transport.LiveRef.exportObject(LiveRef.java:147)
	at sun.rmi.server.UnicastServerRef.exportObject(UnicastServerRef.java:237)
	at sun.management.jmxremote.ConnectorBootstrap$PermanentExporter.exportObject(ConnectorBootstrap.java:199)
	at javax.management.remote.rmi.RMIJRMPServerImpl.export(RMIJRMPServerImpl.java:146)
	at javax.management.remote.rmi.RMIJRMPServerImpl.export(RMIJRMPServerImpl.java:122)
	at javax.management.remote.rmi.RMIConnectorServer.start(RMIConnectorServer.java:404)
	at sun.management.jmxremote.ConnectorBootstrap.exportMBeanServer(ConnectorBootstrap.java:796)

```

简单就是端口被占用了，又去网上搜了很多答案，汇总了两个解决方法：

* 把 catalina.sh 添加的配置删掉
* 使用 kill -9 的命令直接杀掉进程

这两种方案都是**治标不治本**的方法，每次都要做多一点的操作，就显得很繁琐。

## 问题分析

无论使用 startup.sh 启动 tomcat 还是使用 shutdown.sh 关闭 tomcat 都会执行 catalina.sh 脚本，所以关闭 tomcat 也会启动端口，而启动 tomcat 的时候已经开启了端口，关闭的时候再开启就报错了。

## 问题解决

只在**启动** tomcat 时添加 jmx 相关的配置，在 catalina.sh 添加判断条件 `if [ "$1" = "start" ]` ：

```
if [ "$1" = "start" ] ; then
  JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=18088 -Dcom.sun.management.jmxremote.rmi.port=18088 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=34.126.141.211"
fi
```

# 总结

*  网上都是相互抄来抄去的，都是缺少 RMI 配置,完整配置如下
   * JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1808 -Dcom.sun.management.jmxremote.rmi.port=1808 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=34.126.141.21"
* JMX 是一套标准 API,用于管理和监控 Java 应用程序。而 RMI 为 JMX 提供了远程连接。
* 关闭的报错的解决方案也是互相抄来抄去，解决方案都是治标不治本。在配置上面添加 if 判断条件 添加判断条件 if [ "$1" = "start" ]。
