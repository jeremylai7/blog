# mvn 打包报错：no compiler is provided in this environment

最近公司换了电脑，系统也从 `win7` 升级到 `win11`,开发环境都重新安装了一遍，然后在 `idea` 用`mvn` 执行打包命令 `mvn clean package` 报错：

```
no compiler is provided in this environment. perhaps you are running on a jre rather than a jdk
```

# 问题分析

`maven` 没有找到 `Java` 编译环境，先查看 `idea` 编译器配置：

![image](https://user-images.githubusercontent.com/11553237/207997887-2251cc6e-3e25-4483-ada6-5aebd9c6ef42.png)

> `idea`编译环境没有问题。

使用终端执行 `mvn clean package` 同样也报错，就不是 `idea` 的问题。

执行 `maven` 打包命令是需要运行 `mvn.cmd` 文件(`mac`或者`linux`对应的是`mvn.sh`)：

![image](https://user-images.githubusercontent.com/11553237/207997792-1674ed6d-937d-40d7-9ffe-5d6df668eef6.png)

打开文件，看到很多地方是使用了`JAVA_HOME`变量：

![image](https://user-images.githubusercontent.com/11553237/207997928-e5a2aff6-d7e7-4d98-b0e4-96e84eab791e.png)

**`win11`安装 `jdk` 是自动添加 `java`环境变量到 `path` 中,`win7` 是需要手动配置 `java` 的环境变量。 解决方案就是添加 `JAVA_HOME`系统变量。**

# 解决方案 

## 方案一

在 `mvn.cmd` 文件第一行输入： 
```
set JAVA_HOME=C:\Program Files\Java\jdk1.8.0_202
```

> 其中 `C:\Program Files\Java\jdk1.8.0_202` 是 `jdk` 所在的路径。

## 方案二：

新增环境变量`JAVA_HOME`:

![image](https://user-images.githubusercontent.com/11553237/207997837-ef5c8fd2-437c-4f23-994d-1a4c102a8122.png)


# 总结

* `maven` 打包报错，首先需要分析是不是`idea`配置问题。
* 在终端也报同样的错，大概率是`maven`问题，找到启动文件`mvn.cmd`。
* 找打`mvn.cmd`文件找不到`JAVA_HOME`：
   * 在第一行设置`JAVA_HOME`
   * 添加`JAVA_HOME`环境变量
