# nginx配置systemctl服务以及systemctl start nginx 卡住

## nginx配置systemctl服务
在/usr/lib/systemd/system/新建nginx.service文件，编辑如下内容:
```
Description=nginx - high performance web server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
ExecQuit=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target

```
重新加载服务
```
systemctl daemon-reload
```
Nginx服务开关命令:
```
Nginx启动: systemctl start nginx
Nginx重启: systemctl restart nginx
Nginx停止: systemctl stop nginx
```
## systemctl start nginx 卡住问题
需要在nginx.conf 配置 nginx.service文件的PIDFile路径。
## 解决方案
修改nginx.conf，设置pid路径，路径自己定义即可。
```
pid    /run/nginx.pid;
```
