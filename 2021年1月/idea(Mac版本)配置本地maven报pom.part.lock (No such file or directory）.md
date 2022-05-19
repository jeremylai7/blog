## 详情报错信息:
```
Could not transfer artifact junit:junit:pom:4.13.1 from/to central (https://repo.maven.apache.org/maven2): /Repository/junit/junit/4.13.1/junit-4.13.1.pom.part.lock (No such file or directory)
```
## 问题原因:
配置本地仓库地址没有创建。
![idea maven配置](https://user-images.githubusercontent.com/11553237/169195277-9b71440d-9bcd-4adc-9e82-b1da16931d43.png)

将上面的Local repository换成已创建文件夹地址
