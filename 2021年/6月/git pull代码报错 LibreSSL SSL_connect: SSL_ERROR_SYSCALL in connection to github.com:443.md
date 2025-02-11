
执行下面的命令：
```
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global http.sslBackend "openssl" 
## 关闭 ssl 校验
git config --global http.sslVerify false
```
