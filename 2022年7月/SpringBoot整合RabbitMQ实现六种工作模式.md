# SpringBoot整合RabbitMQ实现六种工作模式

> `RabbitMQ`主要有六种种工作模式，本文整合`SpringBoot`分别介绍工作模式的实现。

# 前提概念

## 生产者

消息生产者或者发送者，使用`P`表示：

![image](https://user-images.githubusercontent.com/11553237/181467488-6ec0c4a0-b971-4d28-8efa-f15b074c6714.png)

## 队列

消息从生产端发送到消费端，一定要通过队列转发，使用`queue_name`表示：

![image](https://user-images.githubusercontent.com/11553237/181467526-deed6188-6f93-4e2d-a35a-41d7805867de.png)

## 消费者
消费的消费者或者接收者，使用`C`表示，如果有多个消费者也可以用`C1`、`C2`表示：

![image](https://user-images.githubusercontent.com/11553237/181467557-166b79e8-889c-430b-a759-f59ce8250b28.png)

## SpringBoot整合RabbitMQ基本配置
1. 添加maven依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>2.2.1.RELEASE</version>
</dependency>
```

2. 添加application.yml 配置

```
spring:
  rabbitmq:
    host: 192.168.3.19
    port: 5672
    username: admin
    password: 123456
```
3. 消息生产

生产端发送消息，调用`RabbitTemplate`发送消息，比如：
```
@Autowired
private RabbitTemplate rabbitTemplate;

public String send() {
  rabbitTemplate.convertAndSend("routingKey","send message");
}
```
4. 消费消息

消费消息使用队列监听注解`@RabbitListener`,添加队列名称就能消费发送到队列上的消息了：
```
@RabbitListener(queuesToDeclare = @Queue("queue_name"))
public void consume(String message) {
  // 接收消息
}
```

# 1. 简单（simple）模式


![image](https://user-images.githubusercontent.com/11553237/181467608-28bb1f2f-4a22-42d2-8d9a-f2907eaef5e7.png)

> 最简单的消息发送

## 特点
* 生产者是消费者是一一对应，也叫做`点对点模式`,生产者发送消息经过队列直接发送给消费者。
* 生产者和消费者在发送和接收消息时，只需要指定队列名称，而不需要指定`Exchange `交换机。

## [代码示例](https://github.com/jeremylai7/springboot-learning/blob/master/spring-rabbitmq/src/main/java/com/jeremy/pattern/Simple.java)

生产消息:
```
@GetMapping("/simple-send")
public String simpleSend() {
  rabbitTemplate.convertAndSend("simple","this is news");
  return "ok";
}
```

消费消息
```
@RabbitListener(queuesToDeclare = @Queue("simple"))
public void consume(String message) {
  System.out.println(message);
}
```
输出：
```
this is news
```

> 无需创建交换机和绑定队列，只需要匹配发送端和消费端的队列名称就能成功发送消息。

## 2. 工作模式


![image](https://user-images.githubusercontent.com/11553237/181467772-563a69b5-9836-4c27-8874-65738c0767f6.png)

> 在多个消费者之间分配任务

## 特点
* `工作模式`和`简单模式`差不多，只需要生产端、消费端、队列。
* 不同在于一个生产者、一个队列对应`多个消费者`，也就是一对多的关系。
* 在多个消费者之间分配消息（[竞争消费者模式](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CompetingConsumers.html)），类似轮询发送消息，每个消息都只发给一个消费者。

## [代码示例](https://github.com/jeremylai7/springboot-learning/blob/master/spring-rabbitmq/src/main/java/com/jeremy/pattern/Work.java)

* 生产消息：
```
@GetMapping("/work-send")
public String simpleSend() {
  rabbitTemplate.convertAndSend("work","this is news");
  return "ok";
}
```

* 消费消息：
```
@RabbitListener(queuesToDeclare = @Queue("work"))
public void consume(String message) {
  System.out.println("first:" + message);
}

@RabbitListener(queuesToDeclare = @Queue("work"))
public void consumeSecond(String message) {
  System.out.println("second:" + message);
}
```

创建一个生产者，两个消费者，发送两条消息，两个消费者分别接收到消息，输出：
```
first:this is news
second:this is news
```

两个消费者，轮流消费消息。类似`nginx负载均衡`。

# 3. 发布订阅模式

![image](https://user-images.githubusercontent.com/11553237/181467886-5bfe37fa-3c0f-4849-978b-29f7e33fda7f.png)

> 一次向多个消费者发送消息

## 特点

* 发布订阅类似广播消息，每个消息可以同时发送给订阅该消息的消费者，
* 上图中的`X`表示交换机，使用的`扇形交换机`(fanout),它将发送的消息发送到所有绑定交换机的队列。

## [代码示例](https://github.com/jeremylai7/springboot-learning/blob/master/spring-rabbitmq/src/main/java/com/jeremy/pattern/PublishSubscribe.java)
* 创建队列、交换机以及绑定：
```
@Bean
public FanoutExchange fanoutExchange() {
  return new FanoutExchange("PUBLISH_SUBSCRIBE_EXCHANGE");
}

@Bean
public Queue psFirstQueue() {
  return new Queue("psFirstQueue");
}

@Bean
public Queue psSecondQueue() {
  return new Queue("psSecondQueue");
}

@Bean
public Queue psThirdQueue() {
  return new Queue("psThirdQueue");
}

@Bean
public Binding routingFirstBinding() {
  return BindingBuilder.bind(psFirstQueue()).to(fanoutExchange());
}

@Bean
public Binding routingSecondBinding() {
  return BindingBuilder.bind(psSecondQueue()).to(fanoutExchange());
}

@Bean
public Binding routingThirdBinding() {
  return BindingBuilder.bind(psThirdQueue()).to(fanoutExchange());
}
```

* 上面定义一个交换机`fanoutExchange`。
* 分别绑定三个队列`psFirstQueue`、`psSecondQueue`、`psThirdQueue`。
* 队列绑定交换机不需要`routingKey`，直接绑定即可。


![](https://files.mdnice.com/user/29864/e2ea5537-be5d-4614-80c6-9d27bb339aac.png)


* 生产端：
```
@GetMapping("/publish-sub-send")
public String publishSubSend() {
  rabbitTemplate.convertAndSend("PUBLISH_SUBSCRIBE_EXCHANGE", null, "publish/subscribe hello");
  return "ok";
}
```

无需指定`routingKey`,设置为`null`。

* 消费端：
```
@RabbitListener(queues = "psFirstQueue")
public void pubsubQueueFirst(String message) {
  System.out.println("【first】:" + message);
}

@RabbitListener(queues = "psSecondQueue")
public void pubsubQueueSecond(String message) {
  System.out.println("【second】:" + message);
}

@RabbitListener(queues = "psThirdQueue")
public void pubsubQueueThird(String message) {
  System.out.println("【third】:" + message);
}
```

* **输出：**
```
【first】: publish/subscribe hello
【second】: publish/subscribe hello
【third】: publish/subscribe hello
```

>**发送一条消息，绑定的队列都能接收到消息。**

# 4. 路由模式

![image](https://user-images.githubusercontent.com/11553237/181467950-113ed255-a1c2-4155-aa3d-f133e9c330fb.png)

>根据`routingKey`有选择性的接收消息

## 特点

* 每个队列根据不同`routingKey`绑定交换机
* 消息发送到交换机后通过`routingKey`发送给特定的队列，然后传到消费者消费。
* 交换由`扇形交换机`(fanout)改成`直连交换机`(direct)。

## [代码示例](https://github.com/jeremylai7/springboot-learning/blob/master/spring-rabbitmq/src/main/java/com/jeremy/pattern/Routing.java)
* 创建队列、交换机以及绑定：
```
@Bean
public Queue routingFirstQueue() {
    return new Queue("routingFirstQueue");
}

@Bean
public Queue routingSecondQueue() {
    return new Queue("routingSecondQueue");
}

@Bean
public Queue routingThirdQueue() {
    return new Queue("routingThirdQueue");
}

@Bean
public DirectExchange routingExchange() {
    return new DirectExchange("routingExchange");
}

@Bean
public Binding routingFirstBind() {
    return BindingBuilder.bind(routingFirstQueue()).to(routingExchange()).with("firstRouting");
}

@Bean
public Binding routingSecondBind() {
    return BindingBuilder.bind(routingSecondQueue()).to(routingExchange()).with("secondRouting");
}

@Bean
public Binding routingThirdBind() {
    return BindingBuilder.bind(routingThirdQueue()).to(routingExchange()).with("thirdRouting");
}
```

* 创建一个交换机，根据不同的路由规则匹配不同的队列`routingExchange`,根据不同的`routingKey`绑定不同的队列：
 * `firstRouting`路由键绑定`routingFirstQueue`队列。
 * `secondRouting`路由键绑定`routingSecondQueue`队列。
 * `thirdRouting`路由键绑定`routingThirdQueue`队列。

![image](https://user-images.githubusercontent.com/11553237/181468004-1f1d95e9-0858-4ae1-9f41-2ac67e8d411b.png)

* 生产消息:
```
@GetMapping("/routing-first")
public String routingFirst() {
    // 使用不同的routingKey 转发到不同的队列
    rabbitTemplate.convertAndSend("routingExchange","firstRouting"," first routing message");
    rabbitTemplate.convertAndSend("routingExchange","secondRouting"," second routing message");
    rabbitTemplate.convertAndSend("routingExchange","thirdRouting"," third routing message");
    return "ok";
}
```

* 消费消息：
```
@RabbitListener(queues = "routingFirstQueue")
public void routingFirstListener(String message) {
    System.out.println("【routing first】" + message);
}

@RabbitListener(queues = "routingSecondQueue")
public void routingSecondListener(String message) {
    System.out.println("【routing second】" + message);
}

@RabbitListener(queues = "routingThirdQueue")
public void routingThirdListener(String message) {
    System.out.println("【routing third】" + message);
}
```

**输出：**
```
【routing first】first routing message
【routing second】second routing message
【routing third】third routing message
```

**分析:**
```
rabbitTemplate.convertAndSend("routingExchange","firstRouting"," first routing message");
```
>消息从生产者指定`firstRouting`路由键，找到对应的绑定队列`routingFirstQueue`,就被`routingFirstQueue`队列消费了。

# 5. 主题模式

![image](https://user-images.githubusercontent.com/11553237/181468058-9737b1db-d03a-4d1e-9680-219ccc83f5cc.png)

> 基于某个主题接收消息

## 特点
`路由模式`发送的消息，是需要指定固定的`routingKey`，如果想要针对一类路由。
比如：
* 只接收以`.com `结尾的消息。
* `www.`开头的消息。

`主题模式`就派上场了，`路由模式`和`主题模式`类似，`路由模式`是设置特定的`routingKey`绑定唯一的队列，而`主题模式`的是使用`通配符`匹配`一个或者多个`队列。

## [代码示例](https://github.com/jeremylai7/springboot-learning/blob/master/spring-rabbitmq/src/main/java/com/jeremy/pattern/Topic.java)

* 创建交换机和队列：
```
@Bean
public Queue topicFirstQueue() {
    return new Queue("topicFirstQueue");
}

@Bean
public Queue topicSecondQueue() {
    return new Queue("topicSecondQueue");
}

@Bean
public Queue topicThirdQueue() {
    return new Queue("topicThirdQueue");
}

@Bean
public TopicExchange topicExchange() {
    return new TopicExchange("topicExchange");
}
```

* 使用`通配符`绑定交换机和交换机：

```
@Bean
public Binding topicFirstBind() {
    // .com 为结尾
    return BindingBuilder.bind(topicFirstQueue()).to(topicExchange()).with("*.com");
}

@Bean
public Binding topicSecondBind() {
    // www.为开头
    return BindingBuilder.bind(topicSecondQueue()).to(topicExchange()).with("www.#");
}
```
`通配符`有两种，`*`和`#`,

* `*` 表示可以匹配`一个`。
* `#` 表示可以匹配`多个`。

**比如：**
* `#.com`表示接收`多个`以`.com`结尾的字段。
   * 例如： `taobao.com`、`www.taobao.com`、`www.jd.com`。
* `*.com`表示接收`一个`以`.com`结尾的字段。  
   * 例如： `taobao.com`、`jd.com`。
   * 多个字段是无法匹配的，比如`www.taobao.com`、`cn.taobao.com`。
* `www.#`可以匹配`多个`以`www`开头的字段。
   * 例如`www.taobao`、`www.jd`。
* `www.*`可以匹配`一个`以`www`开头的字段。
   * 例如：`www.taobao`、`www.jd`。
   * 多个字段是无法匹配的,比如`www.taobao.com`、`www.jd.com`。

* 生产消息:
```
@GetMapping("/topic-first-send")
public String topicFirstSend() {
    rabbitTemplate.convertAndSend("topicExchange","www.taobao.com","www.taobao.com");
    rabbitTemplate.convertAndSend("topicExchange","taobao.com","taobao.com");
    rabbitTemplate.convertAndSend("topicExchange","www.jd","www.jd");
    return "topic ok";
}
```
* 消费消息：
```
@RabbitListener(queues = "topicFirstQueue")
public void topicFirstListener(String message) {
    System.out.println("【topic first】" + message);
}

@RabbitListener(queues = "topicSecondQueue")
public void topicSecondListener(String message) {
    System.out.println("【topic second】" + message);
}
```

* **输出：**
```
【topic second】www.taobao.com
【topic first】taobao.com
【topic second】www.jd
```

> `www.#`可以匹配多个以`www.`开头的路由键，例如`www.taobao.com`、`www.jd`。而`*.com`只能匹配一个以`.com`结尾的路由键，例如`taobao.com`，而无法匹配`www.taobao.com`。


# 6. RPC模式


![image](https://user-images.githubusercontent.com/11553237/181468125-461b0221-a5b2-4ff1-bb3b-2e24b29524c7.png)

> 消息有返回值

## 特点
* `PRC`模式和上面的几种模式唯一不同的点在于，该模式可以收到消费端的`返回值`。
* 生成端接收消费端的返回值。

## [代码示例](https://github.com/jeremylai7/springboot-learning/blob/master/spring-rabbitmq/src/main/java/com/jeremy/pattern/Rpc.java)

* 消费端添加返回值：

```
@RabbitListener(queuesToDeclare =@Queue("rpcQueue"))
public String rpcListener(String message) {
  System.out.println("【rpc接收消息】" + message);
  return "rpc 返回" + message;
}
```

* 生产端发送消息：
```
@GetMapping("/rpc-send")
	public void rpcSend() {
		Object receive = rabbitTemplate.convertSendAndReceive("rpcQueue","rpc send message");
		System.out.println("【发送消息消息】" + receive);
	}
```

* 输出：
```
【rpc接收消息】rpc send message
【发送端接收消息】rpc 返回rpc send message
```

# 交换机类型

上面的 `订阅发布模式`、`路由模式`以及`主题模式`使用到了不同的交换机,分别是：

* 直连交换机 Direct
* 扇形交换机 Fanout
* 主题交换器 Topic


![image](https://user-images.githubusercontent.com/11553237/181468798-e6e7eaba-9760-4255-a60d-7161efd778c7.png)

## Direct Exchange（直连）

![image](https://user-images.githubusercontent.com/11553237/181468848-7709e017-786e-4074-b016-36838470025c.png)

`直连交换机`被应用在`路由模式`下，该交换机需要通过特定的`routingKey`来绑定队列，交换机只有接收到了匹配的`routingKey`才会将消息转发到对应的队列中，否则就不会转发消息。

`路由模式`使用`直连交换机`,该模式下根据`routingKey`绑定特定的队列。

## Fanout Exchange(扇形)

![image](https://user-images.githubusercontent.com/11553237/181468897-31bb1d42-fc1f-47a0-902b-75f64d82f50c.png)

`扇形交换机`没有路由键的概念，只需将队列绑定在交换机上，发送到交换机上的消息会转发到交换机所以绑定的队列里面，类似广播，只要打开收音机都能接收到广播消息。`扇形交换机`应用于`发布订阅模式`。

## Topic Exchange(主题)

![image](https://user-images.githubusercontent.com/11553237/181468940-295efa1c-bf10-40ee-8ee2-e55349b2e68d.png)

`主题模式`是将路由键根据一个主题进行分类，和`直连模式`不同的是,`直连模式`绑定`特定`的路由键，而`主题模式`使用通配符绑定路由键,绑定键有两种：
* `*` 表示可以匹配`仅一个`。
* `#` 表示可以匹配`零个或多个`。


# 总结

整合`SpringBoot`实现`RabbitMQ`六种工作模式，并详细讲解`RabbitMQ`六种工作模式：
* 简单模式 
  * 无需创建交换机，匹配生产端和消费的`routingKey`即可。
* 工作模式 
  * 多个消费端公平竞争同一个消息。
* 发布订阅模式 
  * 一次向多个消费者发送消息。
* 路由模式
  * 根据特定的路由键转发消息。
* 主题模式
  * 根据通配符，匹配路由键转发消息。
* RPC模式
  * 生产端接收消费端发送的返回值。 
  
# 源码示例

* [https://github.com/jeremylai7/springboot-learning/tree/master/spring-rabbitmq/src/main/java/com/jeremy/pattern](https://github.com/jeremylai7/springboot-learning/tree/master/spring-rabbitmq/src/main/java/com/jeremy/pattern)

# 参考

* [RabbitMQ简介和六种工作模式详解](https://www.cnblogs.com/Leo_wl/p/15507961.html)
* [RabbitMQ 的四种交换机](https://www.cnblogs.com/zhaosq/p/13986350.html)
