# Spring Cloud 微服务实战——nacos 服务注册中心搭建(附源码)

> 作为微服务的基础功能之一的注册中心担任重要的角色。微服务将单体的服务拆分成不同的模块下的服务，而不同的模块的服务如果进行通信调用呢？这就需要服务注册与发现。本文将使用阿里开源项目 nacos 搭建服务中心。

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。
## 下载以及安装 nacos
在 [nacos官网](https://github.com/alibaba/nacos/releases)找到 nacos 安装包，下载下图的 gz 后缀文件。

![image](https://user-images.githubusercontent.com/11553237/169220240-d6e9ad14-045e-4dc2-8d7d-fb07542978a5.png)

解压文件进入bin目录，执行下面命令。
```
sh startup.sh -m 
```
启动成功
新建一个 springboot 项目
## 搭建服务生产者
* 添加maven依赖
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
 <dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```

* 配置application.yml 文件
```
server:
  port: 8020
spring:
  application:
    name: service-provider
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```
* 在Application.java 启动文件添加 @EnableDiscoveryClient 注解
```

@SpringBootApplication
@EnableDiscoveryClient
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```
* 添加服务提供的接口 
```
@RestController
public class ProviderController {

	@Autowired
	private Environment environment;

	@GetMapping("/hello")
	public String hello(String name){
		return "hello4  " + name + " port:" + environment.getProperty("local.server.port");
	}
}
```
>启动上面的服务（Application.java 里面的main方法），启动成功以后，登录 nacos 控制台[http://127.0.0.1:8848/nacos](http://127.0.0.1:8848/nacos), 默认登录名和密码都是nacos，可以发现服务列表多了一个服务。说明服务提供者已经成功注册进了服务中心。

![image](https://user-images.githubusercontent.com/11553237/169220295-419320c9-5767-45c2-8a17-7979dab4998b.png)

请求访问 [http://127.0.0.1:8020/hello](http://127.0.0.1:8020/hello),成功返回数据，说明服务能正常访问。

## 搭建服务消费者
* 添加maven依赖
```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
        <!--如果只使用 ribbon 不需要下面两个依赖，如果使用 feign ,下面两个依赖都需要。-->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-openfeign</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
```

这里相对于服务提供者，多添加了ribbon和feign依赖。
* 配置 application.yml 文件
```
server:
  port: 8030
spring:
  application:
    name: service-consume
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```
* 修改 Application.java 文件，添加注册
```
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class NacosConsumeApplication {

	public static void main(String[] args) {
		SpringApplication.run(NacosConsumeApplication.class, args);
	}

}
```
>这里添加 @EnableDiscoveryClient 注解和 @EnableFeignClients，其中 @EnableDiscoveryClient 是将服务注册进注册中心，@EnableFeignClients 搭配feign客户端使用。

* 启动上面的服务（Application.java 里面的main方法），启动成功以后，登录 nacos 控制台，发现消费服务也注册进了注册中心。

![image](https://user-images.githubusercontent.com/11553237/169220337-84a81978-50fe-4169-bf1a-11be180b06eb.png)


## 服务调用有两种方式:feign 和 ribbon

### 1. 使用 feign 调用服务
* 配置客户端
```
@FeignClient(value = "service-provider")
public interface ProductClient {

    @GetMapping("/hello")
    String product(@RequestParam("name") String name);
}
```
>这里的 @FeignClient 里面的配置 value 对应的是服务提供者的服务名称，@GetMapping里面的value对应服务提供者的 @GetMapping 路径。
* 调用服务端
```
@RestController
public class FeignController {


    //这个错误是编译器问题。 因为这个Bean是在程序启动的时候注入的，编译器感知不到，所以报错。
    @Autowired
    private ProductClient productClient;

    @GetMapping("/feign")
    public String feign(String name){
        return productClient.product(name);
    }

}
```
#### 使用 ribbon 调用服务员
* 配置 RestTemplate bean
```
@Configuration
public class RibbonConfig {

    @LoadBalanced
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```
* 调用服务
```
@RestController
public class RibbonController {

	@Autowired
	private RestTemplate restTemplate;

	@GetMapping("/ribbon")
	public String ribbon(String name){
		String result = restTemplate.getForObject("http://service-provider/hello?name="+name,String.class);
		return result;
	}
}
```
### 源码
 * [github源码](https://github.com/jeremylai7/spring-cloud-demo)
