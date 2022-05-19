# Spring Cloud Alibaba 使用 feign 和 rebion 进行服务消费

> 微服务的服务消费，一般是使用 feign 和 rebion 调用服务提供，进行服务的消费，本文将实战使用代码讲解服务的消费。

## 微服务环境的搭建
创建一个 springboot 项目，springboot 是将服务进行拆分的一个最小服务单位。

### 添加 maven 依赖
* 基本的maven依赖

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
* rebion 依赖
```
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
            </dependency>
```

* feign 依赖
```
           <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-openfeign</artifactId>
            </dependency>
```
 
>feign依赖包含rebion依赖，如果想同时使用 feign 和 rebion 依赖，只需要引用feign 即可。
### 添加application.yml
```
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```

### 添加Appcation.java启动类

```
@SpringBootApplication
//将服务注册到注册中心
@EnableDiscoveryClient
//扫描和注册feign客户端bean定义
@EnableFeignClients
public class NacosConsumeApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosConsumeApplication.class, args);
    }

}
 ```
其中 @EnableDiscoveryClient 将服务注册到注册中心，@EnableFeignClients 扫描和注册feign客户端bean定义。fegin bean定义是 @FeignClient。

## 服务调用 : feign 和 ribbon
### 1. feign
* 创建feign客户端

```
@FeignClient(value = "service-provider")
public interface ProductClient {

    @GetMapping("/hello")
    String product(@RequestParam("name") String name);
}
```

这里 @FeignClient 就是创建bean 给 @EnableFeignClients 扫描的注册。 @FeignClient 里面的配置 value 对应的是服务提供者的服务名称，@GetMapping里面的value对应服务提供者的 @GetMapping 路径。
> 这样做的好处是这里直接配置好路径和服务名称，不需要调用方做任何的配置。

* 创建请求调用 feign客户端

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

### 2. rebion
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

### feign 和 rebion 优缺点 
* feign 配置比较简单明了，配置好了 FeignClient，直接调用即可，不过后续有多余的配置。
* rebion每次都需要配置请求路径，它的优点在于直接调用即可，不要单独配置客户端。
