# IDEA顺序启动多个Spring Boot微服务

>上个月公司新开发的一个项目，需要使用**微服务**，将单体服务拆分成多个微服务。但是每次修改代码之后都需要启动多个微服务，改个代码，都要**修改五分钟，启动半小时**，但是`idea`可以设置将多个服务依次启动，减少操作时间。

# 详细步骤

## 1. 打开`Services`

在`idea`打开`View` ——> `Tool Windows` ——> `Services`:

![image](https://user-images.githubusercontent.com/11553237/192996660-9883ee63-b558-4736-afe2-8bdab0091e33.png)


## 2. 添加服务

添加服务，选择`Spring Boot`

![image](https://user-images.githubusercontent.com/11553237/192996997-da23d43c-9800-4714-8f29-80f725eed6eb.png)

就会出现如下服务列表：

![image](https://user-images.githubusercontent.com/11553237/192996765-db046304-fb02-4118-9d16-42c35a73e730.png)

如果以上列表不存在服务，先配置启动服务。

## 3. 配置服务

* 如果需要启动的服务不存在，先运行一下服务。
* 需要移除服务，右键`Stop`，移除掉项目。
* 要修改启动顺序，**上下拖拉即可**。

**每次启动服务只需要点击下面按钮就可以了。**

![image](https://user-images.githubusercontent.com/11553237/192996830-cdcf3019-26ba-479b-b0d1-69dd8af6f1c6.png)
