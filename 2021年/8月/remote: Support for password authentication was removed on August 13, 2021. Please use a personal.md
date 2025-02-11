# remote: Support for password authentication was removed on August 13, 2021. Please use a personal...

周末提交代码，把代码push到github上，控制台报了下面的错误:
```
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for
 more information
```
表示远程推送不再支持密码验证了，改成 token 验证了。
## 解决方案
1. 首先要生成token,在 [github](https://github.com) 上找到setting -> Developer settings ->Personal access tokens->Generate new token
![找到生成token路径](https://upload-images.jianshu.io/upload_images/9624625-5f294173eb5a3b4f.png)

在配置页面配置好权限后，即可生成token，**注意这里需要保存好token，因为只显示一次**。

2. 设置token，这里分成两种情况，代码已经有的，远程仓库地址添加token；没有代码的，在git clone添加token

* 修改远程仓库添加token
```
git remote set-url origin https://<your_token>@github.com/<USERNAME>/<REPO>.git
```
* git clone 添加token
```
git clone https://<your_token>@github.com/<USERNAME>/<REPO>.git
```
## 添加好token就可以推送和下拉代码了。

## 遇到的坑
1. 在idea上的github上设置token没效果，这个具体原因未知
2. 网上一大堆介绍如果生成token，但是重点是第二步，添加或者更新token。
