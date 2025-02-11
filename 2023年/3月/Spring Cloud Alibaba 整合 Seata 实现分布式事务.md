# Spring Cloud Alibaba 整合 Seata 实现分布式事务

> 在`Spring Boot`单体服务中，添加`@Transactional`注解就能实现事务。在单体服务中，执行事务都是在同一个数据库下进行。但是随着业务越来越复杂，数据量越来越大会进行分库分表。在微服务场景下，每个服务都有自己的数据库。之前的单体事务无法处理跨库的事务，这个时候就需要使用分布式事务。

前面 [Seata 环境搭建](https://juejin.cn/post/7159090795960598565) 介绍了`seata`的安装，安装后就需要结合实战项目介绍分布式事务的应用。

# 版本

* `spring-cloud-starter-alibaba:2.1.1.RELEASE`
* `spring-cloud-alibaba-dependencies:2.1.1.RELEASE`             
* `Seata Server 1.5.2`
* `Nacos Server:2.0.1`

不同的版本实现可能不太相同，尽量保持版本一致。

# 效果展现

用户下单的业务逻辑，整个逻辑有两个微服务提供支持：

* 仓库服务：给对应的商品扣减库存
* 订单服务：创建订单

![](https://files.mdnice.com/user/29864/e3f41f1f-1678-4be8-83bf-8c90c0283d51.png)

用户下单购买商品。
* 先创建订单，创建订单之后，再扣库存。
* 如果库存不够，就回滚订单。


# 搭建服务

## 项目结构

![](https://files.mdnice.com/user/29864/6962e741-83ba-463f-9dff-6ce512b43d60.png)

有三个项目,仓库服务`stock`、订单服务`order`、`seata`服务。`seata`服务先调用订单服务，然后调用仓库服务。

## maven 依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.1.1.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
<!--Spring Cloud Alibaba-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.1.1.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>$Greenwich.SR3</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<dependency>
   <groupId>com.alibaba.cloud</groupId>
   <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!--seata-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
</dependency>
```

>三个项目的依赖都是一样的。

## yml 配置

`seata`需要分别配置`配置中心`和`注册中心`。

* `配置中心`内部放置着各种配置文件,你可以通过自己所需进行获取配置加载到对应的客户端.比如Seata Client端(TM,RM),Seata Server(TC),会去读取全局事务开关,事务会话存储模式等信息。

* `注册中心`可以说是微服务架构中的”通讯录“，它记录了服务和服务地址的映射关系。在分布式架构中，服务会注册到这里，当服务需要调用其它服务时，就到这里找到服务的地址，进行调用.比如Seata Client端(TM,RM),发现Seata Server(TC)集群的地址,彼此通信.

从官网文档找到[Nacos 配置中心](https://seata.io/zh-cn/docs/user/configuration/nacos.html)，在`application.yml`加入对应的配置：

```
seata:
  config:
    type: nacos
    nacos:
      server-addr: xxxx
      group: SEATA_GROUP
      namespace: xxxxxx
      username: nacos
      password: nacos
```

>`group`、`namespace`在`Nacos`控制中心页面可以找到，分别对应下图：

![](https://files.mdnice.com/user/29864/50276226-32c5-49e8-a4cd-74692ccb7393.png)

[Nacos注册中心](https://seata.io/zh-cn/docs/user/registry/nacos.html)同样可以获取如下配置：

```
seata:
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      group : SEATA_GROUP
      namespace: xxxxx
      username: nacos
      password: nacos
```

`application`对应的是`Seata Server`配置的服务名。`Seata Server`将自己封装成一个微服务注册到`Nacos`注册中心。其他配置和配置中心配置一致。

## 仓库服务

全部贴代码，文章也比较繁琐。所以使用接口的方式，简单声明代码即可。

**仓库扣减库存:**

```
public interface StockService {

    /**
     * 减库存
     * @param id    商品id
     * @param num   扣减数量
     */
    void reduceStock(Long id, BigDecimal num);
}
```

## 订单服务

**创建订单:**

```
public interface OrderService {

    /**
     * 创建订单
     * @param num     下单数量
     * @param price   商品价格
     * @param goodsId 商品id
     */
    Order create(BigDecimal num,BigDecimal price,Long goodsId);
}
```

## 下单操作：

**先创建订单，然后扣减库存**

```
@GlobalTransactional(rollbackFor = Exception.class)
public void order(Long goodsId) {
    orderService.create(BigDecimal.TEN,BigDecimal.TEN,goodsId);
    stockService.reduceStock(goodsId,BigDecimal.TEN);
}
```

# 数据不回滚

调用下单服务，库存不够，报错了，但是创建订单不会回滚。

# 分析原因

事务没有回滚，从[官网示例](https://github.com/seata/seata-samples/tree/master/springcloud-nacos-seata)下载到本地运行，却可以回滚，对比两者控制台输出之后，发现官网示例使用了`mybatis-plus`自动添加了`DataSourceProxy`数据源代理。

所以需要配置`seata`代理数据源。

## 配置数据源代理

配置`Druid`数据源代理：

```
@Configuration
public class DataSourceProxyConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }

    @Bean
    public DataSourceProxy dataSourceProxy(DataSource dataSource) {
        return new DataSourceProxy(dataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath*:/mapper/*.xml"));
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        return sqlSessionFactoryBean.getObject();
    }

}
```

同时需要关闭数据源自动代理:
```
seata.enable-auto-data-source-proxy: false
```

**再调用下单接口，仓库报错报错，创建订单回滚**。订单服务控制台也提示了数据回滚：

```
2023-03-01 23:49:32.162  WARN 66509 --- [nio-8040-exec-3] c.a.c.seata.web.SeataHandlerInterceptor  : xid in change during RPC from 192.168.31.115:8091:1864853653168447501 to null
2023-03-01 23:49:33.653  INFO 66509 --- [h_RMROLE_1_3_24] i.s.c.r.p.c.RmBranchRollbackProcessor    : rm handle branch rollback process:xid=192.168.31.115:8091:1864853653168447501,branchId=1864853653168447502,branchType=AT,resourceId=jdbc:mysql://34.80.215.211:3306/test,applicationData=null
2023-03-01 23:49:33.653  INFO 66509 --- [h_RMROLE_1_3_24] io.seata.rm.AbstractRMHandler            : Branch Rollbacking: 192.168.31.115:8091:1864853653168447501 1864853653168447502 jdbc:mysql://34.80.215.211:3306/test
2023-03-01 23:49:36.005  INFO 66509 --- [h_RMROLE_1_3_24] i.s.r.d.undo.AbstractUndoLogManager      : xid 192.168.31.115:8091:1864853653168447501 branch 1864853653168447502, undo_log deleted with GlobalFinished
2023-03-01 23:49:36.393  INFO 66509 --- [h_RMROLE_1_3_24] io.seata.rm.AbstractRMHandler            : Branch Rollbacked result: PhaseTwo_Rollbacked
```

>`PhaseTwo_Rollbacked`表示两阶段回滚。

# 报错 can not get cluster name 

控制台报错:

```
can not get cluster name in registry config 'service.vgroupMapping.nacos-provide-order-seata-service-group', please make sure registry config correct
can not get cluster name in registry config 'service.vgroupMapping.nacos-provide-order-seata-service-group', please make sure registry config correct
can not get cluster name in registry config 'service.vgroupMapping.nacos-provide-order-seata-service-group', please make sure registry config correct
```

服务在`Nacos`配置中心找不到`service.vgroupMapping.nacos-provide-order-seata-service-group`，这是在找分组事务，官网文档有如下介绍：

![](https://files.mdnice.com/user/29864/8ee132f3-5b48-41dc-8f9e-befc0edd5531.png)

配置中心有`service.vgroupMapping.default_tx_group`配置文件,根据上图讲解，需要在`application.yml`配置:

```
seata:
   tx-service-group: default_tx_group
```

配置后还是报错。定位到报错的源码，向上调式源码，在`application.yml`配置：

```
spring:
  cloud:
    alibaba:
      seata:
        tx-service-group: default_tx_group
```

# 总结

本文介绍了`Spring Cloud`整合分布式事务`seta`，主要有：

* 添加相关依赖
* 配置`application.yml`配置，主要添加`nacos`配置中心和注册中心的配置。
* 实现一个下单服务，先创建订单，然后扣减库存。库存不够，创建订单回滚。

搭建服务完成之后，事务不回滚，对比官网实例项目。需要添加数据源代理，同时关闭据源自动代理。分布式事务就生效了。

控制台一直报错`can not get cluster name in registry config`，通过调试源码，找到问题的根源，这里学到了通过源码解决问题。**以前前段时间一直在看源码，这次通过源码解决问题，努力还是会有收获的**。

# 源码

* [Github 源码](https://github.com/jeremylai7/spring-cloud-demo)
