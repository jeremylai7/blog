# CentOS7安装nginx

## 安装前准备
由于nginx的一些模块依赖一些lib库，所以在安装nginx之前，必须先安装这些lib库，执行下面的命令:
```
[root@localhost local]# yum -y install gcc-c++  pcre pcre-devel  zlib zlib-devel  openssl openssl-devel  
```
pcre-devel 是使用 pcre 开发的一个二次开发库
zlib zlib-devel 依赖压缩
openssl-devel 支持 https（即在ssl协议上传输http）
## 下载安装包
1. 在[官网](https://nginx.org/en/download.html)下载安装包,放在某个目录下。
2. 解压压缩包后进到nginx目录。
3. 配置模块，这里采用默认配置，输入如下命令
```
[root@localhost nginx-1.13.6]# ./configure
./configure --prefix=/usr/local/nginx --with-http_ssl_module
```

4. make 编译 （make的过程是把各种语言写的源码文件，变成可执行文件和各种库文件）
5. make install  安装绝对路径 安装 （make install是把这些编译出来的可执行文件和库文件复制到合适的地方，默认安装在usr/local路径下） 

6. nginx启动、关闭、重启
 ```
nginx 启动 /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
重启 nginx ./sbin/nginx -s reload -c conf/nginx.conf
关闭 nginx ./sbin/nginx -s stop
```
7. 配置
 
![image](https://user-images.githubusercontent.com/11553237/169191298-d44c93e9-c1a1-45e3-8c98-5e60425066c9.png)

8. 配置https
  在配置文件添加如下配置信息

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
在 Nginx 根目录下，通过执行以下命令
```
./sbin/nginx -t
```
## 报错没有ssl模块
1. 在nginx解压目录执行
```
./configure --with-http_ssl_module
```
如果报错 ./configure: error: SSL modules require the OpenSSL library.则执行
```
yum -y install openssl openssl-devel

./configure

./configure --with-http_ssl_module
```
执行 **make** 
将原来的nginx文件备份
```
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
```
将新的nginx文件覆盖到安装目录
```
cp objs/nginx /usr/local/nginx/sbin/nginx 
```
如果发生Text file busy 错误
```
cp: cannot create regular file ‘/usr/local/nginx/sbin/nginx’: Text file busy
```
就添加-rfp(保持被复制的目录结构)
```
cp -rfp objs/nginx /usr/local/nginx/sbin/nginx
```
最后测试 nginx 是否正确
```
/usr/local/nginx/sbin/nginx -t 
```
在443端口下配置location
```
location / {
	        proxy_pass http://xxxxx;
                index  index.html index.htm;
     	}
```
http 强转https
80端口中配置
```
rewrite ^(.*)$  https://$host$1 permanent;#将http 转成https
```

