# Nacos 版本不一致报错: Request nacos server failed

在做微服务开发中，测试环境使用Nacos没有问题，但是生产环境服务启动一直报错：

```
com.alibaba.nacos.api.exception.NacosException: Request nacos server failed: 
	at com.alibaba.nacos.client.naming.remote.gprc.NamingGrpcClientProxy.requestToServer(NamingGrpcClientProxy.java:279) ~[nacos-client-2.0.3.jar:na]
	at com.alibaba.nacos.client.naming.remote.gprc.NamingGrpcClientProxy.doSubscribe(NamingGrpcClientProxy.java:227) ~[nacos-client-2.0.3.jar:na]
	at com.alibaba.nacos.client.naming.remote.gprc.NamingGrpcClientProxy.subscribe(NamingGrpcClientProxy.java:212) ~[nacos-client-2.0.3.jar:na]
	at com.alibaba.nacos.client.naming.remote.NamingClientProxyDelegate.subscribe(NamingClientProxyDelegate.java:147) ~[nacos-client-2.0.3.jar:na]
	at com.alibaba.nacos.client.naming.NacosNamingService.subscribe(NacosNamingService.java:393) ~[nacos-client-2.0.3.jar:na]
```

# 原因分析 版本不一致

代码没有改动，测试环境没问题，但是生产环境有问题呢？首先看一下两者不同的地方，大多数都是**环境配置**的问题。

查看`Nacos`服务的版本，查看`Nacos`控制台首页左上角就能看到版本号：

![](https://files.mdnice.com/user/29864/26483e86-be5c-41ea-a9ca-057094198185.png)

测试环境版本是`2.0.x.RELEASE `，生产环境版本是`2.1.x.RELEASE`，再看`alibaba.cloud`中的`maven`中的依赖：

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.0.1.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

# 解决方案

将依赖从`2.0.x.RELEASE`改成`2.1.x.RELEASE`。

`Nacos`服务端`maven`依赖对应的**版本号**保持一致。`Spring Boot`依赖的版本号也要保持一致。版本 `2.1.x.RELEASE` 对应的是`Spring Boot 2.1.x`版本。版本`2.0.x.RELEASE`对应的是`Spring Boot 2.0.x`版本，具体查看[官方详解](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)。

![](https://files.mdnice.com/user/29864/e46cf947-9607-484d-ab00-c7fa4daaa2ee.png)

# 总结

* `Nacos`服务端和`Nacos`依赖的版本号要一致
* `Spring Boot`和`Spring Cloud Alibaba`版本号要保持一致，`Spring Cloud`也需要对应匹配。具体查看 `https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E`。
