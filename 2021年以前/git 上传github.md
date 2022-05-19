# git 上传github

## 快速开始
1. git init
2.  git add 
3.  git commit -m "提交注释" 
4.  git remote add origin '地址'
如果出现错误：fatal: remote origin already exists，则执行以下语句：
```
  git remote rm origin
```
在执行3.
5. git push origin master
如果出现错误failed to push som refs to…….，则执行以下语句，先把远程服务器github上面的文件拉下来，再push 上去。：
```
 $ git pull origin master
```

参考：
[网址](https://blog.csdn.net/sinat_20177327/article/details/76062030)

要想删除文件，直接在本地删除文件即可。

不删除本地文件，删除git上的文件

1. git rm -r --cached some-directory    some-directory 表示路径
2. git commit -m 'Remove the now ignored directory "some-directory"'
3. git push origin master

## 强制覆盖本地文件
1.  git fetch --all
2.  git reset --hard origin/master 
3.  git pull origin master
