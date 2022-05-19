# Springboot 打war包

## 打成jar包
如果你使用的是maven来管理项目，执行以下命令既可以

cd 项目跟目录（和pom.xml同级）
 ```
mvn clean package
```
## 或者执行下面的命令
## 排除测试代码后进行打包
```
mvn clean package  -Dmaven.test.skip=true
```
打包完成后jar包会生成到target目录下，命名一般是 项目名+版本号.jar

启动jar包命令
```
java -jar  target/spring-boot-scheduler-1.0.0.jar
```
这种方式，只要控制台关闭，服务就不能访问了。下面我们使用在后台运行的方式来启动:

nohup java -jar target/spring-boot-scheduler-1.0.0.jar &
也可以在启动的时候选择读取不同的配置文件

java -jar app.jar --spring.profiles.active=dev
也可以在启动的时候设置jvm参数

java -Xms10m -Xmx80m -jar app.jar &

## 打成war包
打成war包
打成war包一般可以分两种方式来实现，第一种可以通过eclipse这种开发工具来导出war包，另外一种是使用命令来完成，这里主要介绍后一种

1、maven项目，修改pom包

将
```
<packaging>jar</packaging>  
```
改为
```
<packaging>war</packaging>
```
2、打包时排除tomcat.
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-tomcat</artifactId>
	<scope>provided</scope>
</dependency>
```
在这里将scope属性设置为provided，这样在最终形成的WAR中不会包含这个JAR包，因为Tomcat或Jetty等服务器在运行时将会提供相关的API类。

3、注册启动类

创建ServletInitializer.java，继承SpringBootServletInitializer ，覆盖configure()，把启动类Application注册进去。外部web应用服务器构建Web Application Context的时候，会把启动类添加进去。
```
public class ServletInitializer extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }
}
```
最后执行
```
mvn clean package  -Dmaven.test.skip=true
```
会在target目录下生成：项目名+版本号.war文件，拷贝到tomcat服务器中启动即可。
