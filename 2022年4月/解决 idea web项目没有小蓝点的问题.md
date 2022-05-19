# 解决 idea web项目没有小蓝点的问题

在idea导入web项目，项目没有显示小蓝点，无法添加 java文件和运行。如下图的`springboot-schedule` 和 `springboot-test` 都没有蓝点：
![image](https://user-images.githubusercontent.com/11553237/169229423-73a0dada-e9ed-412e-945b-03687312cb51.png)

## 解决方案一：
点击 File --> Project Structure,选择下图 add web。改成`web` 项目：
![image](https://user-images.githubusercontent.com/11553237/169229456-41f1b01b-d474-419e-8a0a-c68032c384d3.png)

## 进阶解决方案：
正常创建的项目是有小蓝点，没有蓝点，一般是配置文件的问题。
找到 `.idea` --> `modules.xml`
![image](https://user-images.githubusercontent.com/11553237/169229515-59c9f5b0-7bfd-49c0-8d6c-996997e7254c.png)

没有蓝点一般是因为没添加项目的 `.iml` 文件：
![Uploading image.png…]()

添加路径:
```
<module fileurl="file://$PROJECT_DIR$/项目名称/iml文件路径 " filepath="$PROJECT_DIR$/iml文件路径" />
```

## 总结
* 正常创建的`web`项目有小蓝点，但是如果修改过项目名称或者修改项目路径，会导致 `idea` 不能读取成一个`web`项目。
* 把`.iml`项目路径添加进 `.idea` --> `modules.xml` 即可。
