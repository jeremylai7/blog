# Nacos 源码环境搭建

> 最近在学习`nacos`，通过调式源码查看服务注册和发现流程和原理，本地部署`naos`源码需要一定的步骤，本文主要做`nacos`源码部署。

**nacos版本：2.1.1**

# 下载源码

从[github](https://github.com/alibaba/nacos/releases/tag/2.1.1)上下载源码到本地，下载下图的源码包，地址为`https://github.com/alibaba/nacos/releases/tag/2.1.1`。


![image](https://user-images.githubusercontent.com/11553237/211712931-c25767d1-5783-499c-920a-36000a6fa7a4.png)

解压后用`idea`打开源码，`com.alibaba.nacos.consistency.entity`报红错误：

![image](https://user-images.githubusercontent.com/11553237/211712972-5dae053f-9a02-4c8b-9f43-9f6601a6581c.png)

# 编译

解决`com.alibaba.nacos.consistency.entity`报红问题，编译总项目。在目录`nacos-2.1.1`打开终端编译整个项目:

```
mvn compile
```

# 启动

## 启动类

找到`console`项目中的`Nacos`启动类。

![image](https://user-images.githubusercontent.com/11553237/211713010-09241980-f3e1-4a98-826b-294fd1c2b609.png)

## 设置单机启动

启动类配置`VM options`添加参数，设置成单机启动：

```
-Dnacos.standalone=true
```
## 启动输出

有以下输出，说明项目启动成功：

```

         ,--.
       ,--.'|
   ,--,:  : |                                           Nacos 
,`--.'`|  ' :                       ,---.               Running in stand alone mode, All function modules
|   :  :  | |                      '   ,'\   .--.--.    Port: 8848
:   |   \ | :  ,--.--.     ,---.  /   /   | /  /    '   Pid: 7184
|   : '  '; | /       \   /     \.   ; ,. :|  :  /`./   Console: http://192.168.3.181:8848/nacos/index.html
'   ' ;.    ;.--.  .-. | /    / ''   | |: :|  :  ;_
|   | | \   | \__\/: . ..    ' / '   | .; : \  \    `.      https://nacos.io
'   : |  ; .' ," .--.; |'   ; :__|   :    |  `----.   \
|   | '`--'  /  /  ,.  |'   | '.'|\   \  /  /  /`--'  /
'   : |     ;  :   .'   \   :    : `----'  '--'.     /
;   |.'     |  ,     .-./\   \  /            `--'---'
'---'        `--`---'     `----'

2023-01-11 11:20:45.576  INFO 7184 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8848 (http)
2023-01-11 11:20:45.716  INFO 7184 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1964 ms
2023-01-11 11:20:48.151  INFO 7184 --- [           main] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page: class path resource [static/index.html]
2023-01-11 11:20:48.402  WARN 7184 --- [           main] o.s.s.c.a.web.builders.WebSecurity       : You are asking Spring Security to ignore Ant [pattern='/**']. This is not recommended -- please use permitAll via HttpSecurity#authorizeHttpRequests instead.
2023-01-11 11:20:48.402  INFO 7184 --- [           main] o.s.s.web.DefaultSecurityFilterChain     : Will not secure Ant [pattern='/**']
2023-01-11 11:20:48.418  INFO 7184 --- [           main] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@637c8632, org.springframework.security.web.context.SecurityContextPersistenceFilter@7c4a5ef2, org.springframework.security.web.header.HeaderWriterFilter@3055f310, org.springframework.security.web.csrf.CsrfFilter@7901a5ab, org.springframework.security.web.authentication.logout.LogoutFilter@7a2fd94c, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@34d3409d, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@2f64f99f, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@156eeff1, org.springframework.security.web.session.SessionManagementFilter@7d7c05fa, org.springframework.security.web.access.ExceptionTranslationFilter@288b8663]
2023-01-11 11:20:48.433  INFO 7184 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 1 endpoint(s) beneath base path '/actuator'
2023-01-11 11:20:48.465  INFO 7184 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8848 (http) with context path '/nacos'
2023-01-11 11:20:48.480  INFO 7184 --- [           main] c.a.n.c.l.StartingApplicationListener    : Nacos started successfully in stand alone mode. use embedded storage
2023-01-11 11:20:48.752  INFO 7184 --- [)-192.168.3.181] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2023-01-11 11:20:48.752  INFO 7184 --- [)-192.168.3.181] o.s.web.servlet.DispatcherServlet        : Completed initialization in 0 ms
```

## 查看控制台

请求`http://127.0.0.1:8848/nacos`,查看控制台。

![image](https://user-images.githubusercontent.com/11553237/211713039-8250362a-4f37-4647-b847-eae6d2de9f01.png)
