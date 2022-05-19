# idea mapper xml 文件报红

在使用 idea 打开 mapper 文件，出现一下报红错误：

![image](https://user-images.githubusercontent.com/11553237/169225438-dd39e69d-5513-4c89-8519-d99267bd72f5.png)

可以看到数据表和字段都是红色的。

## 解决方案
* 打开设置，window版本是打开Settings：

![image](https://user-images.githubusercontent.com/11553237/169225489-e72a87f5-1bbf-4a69-a5d3-4bc5b8122776.png)

* 找到 Languages & Frameworks  —> SQL Dialects：

![image](https://user-images.githubusercontent.com/11553237/169225523-39588529-fd90-4025-b2e4-b7ce5637c264.png)

* 设置 Global SQL Dialect 和 Project SQL Dialect 都设置成 None。

![Uploading image.png…]()
