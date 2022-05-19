# Spring Boot项目微信云托管入门部署

> 微信云托管本身是一个服务器，里面的软件都已经配置好了，直接使用即可，适用于一些简单部署的项目。直接把项目直接上传到服务器即可。无需各种繁琐的软件配置和打包，微信云托管统统给你搞定。而且系统会根据使用量计费，对于一些使用量比较少的系统，也是很划算的。本文从一个 Spring Boot 项目简单部署云托管项目。

## 云托管简介
在 [官网](https://cloud.weixin.qq.com/cloudrun) 显示微信云托管的几个优势：
* 开箱即用
* 支持多种后端语言
* 自动扩容
![image](https://user-images.githubusercontent.com/11553237/169226914-8f8ca832-19d4-4d06-9a87-d364f52d3591.png)

### 云托管相对传统项目的优势
* 发布一个简单的 demo 在linux服务器
  * 创建 springboot 项目
  * 将 springboot 项目打成war 包
  * 在 linux 服务器安装 jdk、tomcat，阿里云或者腾讯云开放对应的端口
  * 安装 mysql 
  * 执行sql 语句
  * 上传 war 包到 tomcat 服务器

* 对应的云托管
  * 创建 springboot 项目
  * 在云托管数据库执行 sql 语句
  * 上传springboot文件

>云托管直接上传项目文件即可。

## 入门
进入控制台后，找到 服务列表 -> 新建服务
![image](https://user-images.githubusercontent.com/11553237/169226910-fde1175e-b4b9-44c2-a8bf-2ecc92983690.png)

写好服务名称后，选择不同方式上传代码，这里有 `github`、`gitlab`、`gitee`、`手动上传代码包` 等等。我这里选择了使用手动上传代码包。
![image](https://user-images.githubusercontent.com/11553237/169227003-eaff4b96-4a74-4845-947e-6a0798408117.png)

此处上传的代码和Spring Boot项目区别在于这里上传的代码需要添加额外的两个文件， `Dockerfile` 和 `settings.xml`，文件在 [https://github.com/WeixinCloud/wxcloudrun-springboot](https://github.com/WeixinCloud/wxcloudrun-springboot) 可以找到：

![image](https://user-images.githubusercontent.com/11553237/169227051-be31b163-7e53-45c8-8c78-02eafe96f16c.png)

### Dockerfile 
Dockerfile 配置 `docker` 环境，里面主要命令是项目打包、运行。
这里的打包是打成一个 `jar` 包,如果项目是原来打成一个 war 包，要改成打成 jar 包。
需要将 `pom.xml`
```
<packaging>jar</packaging>  
```
改成
```
<packaging>war</packaging>  
```
Dockerfile 还有打包和运行的命令，修改下面用红框框起来的数据，改成打包的名称：
![image](https://user-images.githubusercontent.com/11553237/169227124-b17da504-fb2e-4d19-9b2a-716e734fbeed.png)

### settings.xml
 settings.xml是 maven 打包的依赖项配置。默认使用腾讯云maven依赖，不需要改动配置。


为了减少项目线上部署调试时间，先在本地执行打包命令，确保打包成功：
```
mvn clean package
```
如果打包成功，直接上传文件，这里为了减少上传时间，可以先对文件进行压缩。

![image](https://user-images.githubusercontent.com/11553237/169227170-71610b40-05f4-421f-a35d-af2cfa332f02.png)

>上面的端口要和springboot里面配置的端口要一致，最好在 `application.yml` 配置文件设置默认端口80，省去每次发布项目都要修改端口的麻烦。

上传成功之后，点击**发布**。发布成功之后访问公网地址。如下图所示：
![image](https://user-images.githubusercontent.com/11553237/169227204-5d0cb8ef-7746-49c9-b830-6d1d4f5c97d5.png)

走完一遍流程，发现云托管有几个不足的地方：
* 发布时间大概要花7,8分钟，主要是花在下载 maven 依赖的时间比较多。
* 如果 `maven` 依赖在本地配置，就不能在云托管使用依赖。

以上两个问题，如果使用上传打包好的 `jar` 包，就能解决这个问题，期待后续能支持上传 `jar` 包的选项。

## 总结
* 如果部署一些不太复杂的项目，微信云托管是一个不错的选择，可以根据配置使用，自动扩容。
* Spring Boot 添加两个文件 `Dockerfile` 和 `settings.xml`。
    * Dockerfile 需要修改打包名称和运行名称
    * settings.xml 不用修改
* Spring Boot 需要改成 `jar` 包的打包方式。
* 项目端口最好配置成默认端口 `80`。

## 后续

文章也同步发表在公众号上，后面遇到微信云托管的一个运维人员。他当时说可以上传 `jar` 包, 通过**上传压缩包**即可：
![Uploading image.png…]()

后续可以直接上传本地打好的 `jar` 包，这样可以省很多时间。也能解决我上面题的问题。
> 当时以为压缩包只是把文件夹压缩后上传，也没有文档说明。这点有点坑，不过还能在公众号找到我，跟进后续，这个给微信云托管的运营点个赞！！

