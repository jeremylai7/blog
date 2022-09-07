# 阿里云ECS后台CPU占用100%，top却找不到

上周公司阿里云服务器后台报警，`CPU`占用瞬间飙升到`100%`:

![image](https://user-images.githubusercontent.com/11553237/188775159-17ecbe4e-05f4-4678-a19f-2cda7724c537.png)

首先想到使用`top`命令查询`CPU`占用详情:

![image](https://user-images.githubusercontent.com/11553237/188775168-8e6f3d3f-a8ef-47f8-a850-98266f26aff4.png)

发现进程占用`CPU`都比较低，在`CPU`占用一栏发现只有`ni`的占用比较高。

先了解一下`CPU`相关监控项：
* us: 用户空间占用`CPU`百分比。
* sy: 内核空间占用`CPU`百分比。
* ni：用户进程空间改变优先级的进程占用`CPU`百分比。
* id: 空闲`CPU`百分比。

`ni`表示**用户进程改变优先级占用**,这个解释有些抽象，简单来说就是**优先进程分配时间片占用总进程`CPU`的百分比**。

> `ni`越高表示某个进程优先级越高，占用的`CPU`占比也就越高。

这么诡异的程序使用`top`命令找不到，再使用`htop`查找，没有安装先使用命令安装：
```
yum -y install epel-release
yum -y install htop
```

然后输入`htop`查询：

![image](https://user-images.githubusercontent.com/11553237/188775191-122184be-c91e-474d-8f40-d8b5c1010830.png)

发现`cryto`相关字段的进程`CPU`特别高，首先用`kill -9`杀死进程。`kill`进程之后，过了几秒，相关的进程又**死灰复燃**了。找了很多文章，发现`cryto`是一个挖矿的病毒。

## 解决方案

* 在`htop`中找到`cryto`进程对应的路径，删除该目录下面所有的文件。
* 全局搜索`cryto`，把出现的文件或者目录全部删除。
* 检查防火墙是否关闭，开启防火墙，安装安全软件查杀，全盘查杀整个服务器。


# 总结

* 后台报警`CPU`占用`100%`，使用`top`命令找不到占用高的进程，但发现`ni`占比过高，`ni`表示优先级进程占用`CPU`的百分比。说明这个进程一直在长时间的占用`CPU`。
* 使用`htop`找到进程，发现是`cryto`占用很高，`cryto`是挖矿病毒，`kill`进程之后，进程又死灰复燃。
* 找到进程对应的目录，以及全局搜索`cryto`关键字，删除所有上述目录，再查看后台，`CPU`占比下降。

# 参考

[linux top中nice的含义](https://www.jianshu.com/p/3c078505fffa)
