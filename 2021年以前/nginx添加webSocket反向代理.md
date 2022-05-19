# nginx添加webSocket反向代理

nginx是http服务器，默认支持http域名转发。而webSocket是全双工通讯的网络技术。它基于TCP传输协议，并复用HTTP的握手通道。为了支持webSocket，需要在nginx转发上配置Upgrade和Connection转发头信息，如以下示例:
```
location /webSocket/ {
    proxy_pass http://localhost:8080/sell/;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
}
```
配置完成后重启nginx即可。
参考文档: [nginx官方文档]("https://www.nginx.com/blog/websocket-nginx")
