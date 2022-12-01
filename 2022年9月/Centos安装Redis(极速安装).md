# Centos安装Redis(极速安装)

# 下载
从[官网](https://redis.io/download/#redis-downloads)找到下载文件，我下载的是`redis-6.0.16.tar.gz`。 

# 安装

## 1. 解压文件

解压文件然后，进入解压文件夹：
```
tar -zxvf redis-6.0.16.tar.gz
cd redis-6.0.16
```

## 2. 安装

编译
```
make
```

安装

```
make install PREFIX=/usr/local/redis
```

从解压文件复制`redis.conf` 到 `usr/local/redis/bin`：
```
cp redis.conf usr/local/redis/bin
```

# 修改 redis.conf

## 1. 开启后台启动

`Redis`默认前台启动，窗口就不能关闭，否则`Redis`服务就停止，这很不方便，所以需要开启后台启动。
```
daemonize yes
```

## 2. 开启远程连接

* 注释`bind`
```
bind 127.0.0.1
```

* 关闭安全保护
```
protected-mode no
```

* 添加密码
```
requirepass 密码
```

# 配置 systemctl 服务

* 1. 在/usr/lib/systemd/system创建redis.service文件：
```
vim /usr/lib/systemd/system/redis.service
```

* 2. 添加配置内容:

```
[Unit]
Description=redis
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/redis/bin/redis-server /etc/redis.conf
ExecStop=/usr/local/redis/bin/redis-cli -p 6379 shutdown
PrivateTmp=true

[Install]
```

* 3. `systemctl`服务命令：

```
# 查询状态
systemctl status redis
# 启动
systemctl start redis
# 关闭
systemctl stop redis
# 设置开机启动
systemctl enable redis
```

# 启动服务

```
systemctl start redis
```

执行命令之后，就能成功远程连接`Redis`服务了。
