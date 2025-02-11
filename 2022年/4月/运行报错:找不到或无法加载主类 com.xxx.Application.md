# 运行报错:找不到或无法加载主类 com.xxx.Application

> `springboot` 项目下载到本地，用  `idea` 运行报错**找不到或无法加载主类**.

## 原因
项目内还没有编译，所以找不到主类文件，需要先编译项目。

## 解决方案 
执行 `mvn` 编译命令：
```
mvn compile
```

或者点击 `idea` 右侧的 `maven` 菜单栏,点击 `compile`:
![Uploading image.png…]()

## 参考
[找不到或无法加载主类 com.xxx.yyy.Application](https://blog.csdn.net/amoscn/article/details/111466987)
