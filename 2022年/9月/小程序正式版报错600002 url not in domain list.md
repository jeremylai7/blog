# 小程序正式版报错600002 url not in domain list

开发版，体验版都没问题，线上版本报错
```
errMsg: "request:fail url not in domain list"
errno: 600002
```

在[小程序官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/usability/PublicErrno.html)找到错误码`600002`：
 
![image](https://user-images.githubusercontent.com/11553237/190374184-60844e60-c147-4300-b5f1-cbc879f24493.png)


# 解决方案：

登录[小程序后台](https://mp.weixin.qq.com/),找到开发 -> 开发管理 -> 开发设置 -> 服务器域名

![image](https://user-images.githubusercontent.com/11553237/190374237-a0b2e603-6e9b-4f60-8797-9eb6eb73f409.png)

配置对应的域名即可。
