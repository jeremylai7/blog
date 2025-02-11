# Mac 远程上传文件到 Linux

1. 打开Mac自带终端：

![image](https://user-images.githubusercontent.com/11553237/169199113-0fa15305-3246-4341-b8b1-991d117531d4.png)


2. 在最顶端选择Shell ->新建远程连接：

![image](https://user-images.githubusercontent.com/11553237/169199148-b19d5572-d23d-43a6-92ae-6676635828b6.png)

3. 选择sftp连接，填好服务器地址:

![image](https://user-images.githubusercontent.com/11553237/169199182-39111570-4064-4b49-9419-9bc39fe59e7a.png)

4. 连接成功后。上传文件，使用 put 命令： 
```
put 本地文件路径 远程主机路径
```
