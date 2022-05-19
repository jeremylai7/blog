# item2 报错 a session ended very soon after starting. check that the command in profile default

周末修改了阿里云 ecs 实例密码，再次用item2 远程连接服务器时，报一下的错误：

![image](https://user-images.githubusercontent.com/11553237/169222838-6ea189dd-bcb5-443c-97c6-ba1cae4204c4.png)


## 原因
每次使用ssh 远程新的连接，都会在 `~/.ssh/known_hosts` 文件上生成 ssh 秘钥对，更新了远程密码后，需要删除对应的 ssh 秘钥。不然会报错。

## 解决方案
找到文件 `~/.ssh/known_hosts`,找到你要远程连接的 ip，删除对应的秘钥，比如我远程连接 `47.98.202.133` ,将下图的框起来的删除即可：

![image](https://user-images.githubusercontent.com/11553237/169222892-e03d6b7c-e2c6-46e5-a6d0-ed2dde35741c.png)
