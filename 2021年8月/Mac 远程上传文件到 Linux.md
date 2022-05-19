# Mac 远程上传文件到 Linux

1. 打开Mac自带终端：
![mac自带终端](https://upload-images.jianshu.io/upload_images/9624625-4a7b07c2da96eaf8.png)

2. 在最顶端选择Shell ->新建远程连接：
![新建连接](https://upload-images.jianshu.io/upload_images/9624625-a787357d58b7e0cf.png)

3. 选择sftp连接，填好服务器地址:
![填写服务器地址](https://upload-images.jianshu.io/upload_images/9624625-e16304274a94af35.png)

4. 连接成功后。上传文件，使用 put 命令： 
```
put 本地文件路径 远程主机路径
```
