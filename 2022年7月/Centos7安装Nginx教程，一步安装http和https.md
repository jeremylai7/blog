# Centos7安装Nginx教程，一步安装http和https

> nginx是一款轻量级web服务器，主要有负载均衡和反向代理的特性。

# 安装准备

`nginx`一些模块需要依赖`lib`库，所以先安装`lib`库，执行以下命令：
```
[root@localhost local]# yum -y install gcc-c++  pcre pcre-devel  zlib zlib-devel  openssl openssl-devel
```
# 下载

1. 在[官网](https://nginx.org/en/download.html)下载安装包

![image](https://user-images.githubusercontent.com/11553237/182284178-2f174157-44bf-4748-b4fa-bfa5de7fe654.png)

# 安装
1. 解压文件:
```
tar -zxvf nginx-1.20.2.tar.gz
```

2. 解压之后进入到nginx目录:
```
cd nginx-1.20.2
```

3. 默认配置模块:
```
./configure
```

需要添加`https`配置模块:
```
./configure --prefix=/usr/local/nginx --with-http_ssl_module
```

4. 编译
```
make
```

5. 安装
```
make install [路径，默认安装在usr/local路径下]
```



# 启动、关闭

```
# 启动
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
# 重启
/usr/local/nginx/sbin/nginx -s reload
# 关闭
/usr/local/nginx/sbin/nginx -s stop
```

>每次启动或者重启服务都需要输入一大串路径，为了简化操作，配置一下`systemctl`服务。

## 配置`systemctl`服务

1. 在`/usr/lib/systemd/system`创建`nginx.service`文件：
```
vim /usr/lib/systemd/system/nginx.service
```
* 添加配置内容:
```
[Unit]
Description=nginx
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
```

* 配置systemctl之后的启动方式
```
# 查询状态
systemctl status nginx
# 启动
systemctl start nginx
# 重启
systemctl restart nginx
# 关闭
systemctl stop nginx
# 设置开机启动
systemctl enable nginx
```
* 启动`nginx`，访问`http://127.0.0.1`,出现如下页面说明`nginx`启动成功：

![image](https://user-images.githubusercontent.com/11553237/182284209-7514bbc4-784b-4353-8f48-906ea633e135.png)



# 配置

修改`conf/nginx.conf`配置文件。

找到`server`模块下的`80`端口。
* 修改`server_name`后面的域名，我这里改成`www.jeremy7.cn`。
* 在`location /`下添加`proxy_pass`反向代理:
```
proxy_pass http://wwwtomcat; 
```
* 对应添加一个`upstream`模块,添加转发的路径：
```
upstream wwwtomcat {
   server 127.0.0.1:8080;
}
```

```
server {
    listen       80;
    server_name  www.jeremy7.cn;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        proxy_pass http://wwwtomcat; 
        root   html;
        index  index.html index.htm;
    }
}
```

# 配置https

开启 443端口
```
server {
    listen 443 ssl;
    #配置HTTPS的默认访问端口为443。
    #如果未在此处配置HTTPS的默认访问端口，可能会造成Nginx无法启动。
    #如果您使用Nginx 1.15.0及以上版本，请使用listen 443 ssl代替listen 443和ssl on。
    server_name yourdomain.com; #需要将yourdomain.com替换成证书绑定的域名。
    root html;
    index index.html index.htm;
    ssl_certificate cert/cert-file-name.pem;  #需要将cert-file-name.pem替换成已上传的证书文件的名称。
    ssl_certificate_key cert/cert-file-name.key; #需要将cert-file-name.key替换成已上传的证书密钥文件的名称。
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    #表示使用的加密套件的类型。
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #表示使用的TLS协议的类型。
    ssl_prefer_server_ciphers on;
    location / {
        root html;  #站点目录。
        index index.html index.htm;
    }
}
```

* `server_name` 改成自己的申请的域名
* `ssl_certificate` 替换成`.pem`后缀的证书文件
* `ssl_certificate_key` 替换成`.key`后缀的证书文件
* `location /` 里面添加反向代理,也是上面的反向代理：
```
proxy_pass http://wwwtomcat; 
```

测试配置是否正确:
```
/usr/local/nginx/sbin/nginx -t 
```

## http强转https

`server`模块添加配置
```
rewrite ^(.*)$  https://$host$1 permanent;#将http转成https
```

