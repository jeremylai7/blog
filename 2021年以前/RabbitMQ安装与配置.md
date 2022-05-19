# RabbitMQ安装与配置

### 安装RabbitMQ需先安装erlang和socat
1. 安装依赖环境
```
yum install build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel
```
2. 下载软件包及其依赖包：
```
# 过期 wget www.rabbitmq.com/releases/erlang/erlang-18.3-1.el7.centos.x86_64.rpm
wget github.com/rabbitmq/erlang-rpm/releases/download/v23.2.1/erlang-23.2.1-1.el7.x86_64.rpm
wget repo.iotti.biz/CentOS/7/x86_64/socat-1.7.3.2-5.el7.lux.x86_64.rpm
# 过期 wget www.rabbitmq.com/releases/rabbitmq-server/v3.6.5/rabbitmq-server-3.6.5-1.noarch.rpm
wget github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.9/rabbitmq-server-3.8.9-1.el6.noarch.rpm
```
3. 执行安装
```
依次执行
rpm -ivh erlang-18.3-1.el7.centos.x86_64.rpm
rpm -ivh socat-1.7.3.2-5.el7.lux.x86_64.rpm
rpm -ivh rabbitmq-server-3.6.5-1.noarch.rpm
```
安装 erlang如果以前安装了erlang，会出现冲突
```
file /usr/lib64/erlang/bin/epmd from install of erlang-23.2.1-1.el7.x86_64 conflicts with file from package erlang-19.3.6.13-1.el7.centos.x86_64
```
使用
```
yum remove erlang-19.3.6.13-1
```

4.启动
```
## 启动
systemctl start rabbitmq-server
##设置开机启动
systemctl enable rabbitmq-server
##查看状态
systemctl status rabbitmq-server
```
5.添加用户
```
查看当前所有用户
rabbitmqctl list_users

# 查看默认guest用户的权限
rabbitmqctl list_user_permissions guest

# 添加新用户
rabbitmqctl add_user username password

# 设置用户tag
rabbitmqctl set_user_tags username administrator

# 赋予用户默认vhost的全部操作权限
rabbitmqctl set_permissions -p / username ".*" ".*" ".*"

# 查看用户的权限
rabbitmqctl list_user_permissions username
```
6. 插件管理

查看插件列表
```
rabbitmq-plugins list
```
启动management插件

rabbitmq-plugins enable rabbitmq_management
8. 至此已经安装配置完成，访问管控台查看信息
```
http://YourIp:15672/
```
> 如果无法访问，可能是防火墙的问题，可以关闭防火墙试试
```
service firewalld stop  
```
或者
```
systemctl stop firewalld
```
>如果登录不上(Login failed)，可能是没有权限需要设置tag
```
# 设置用户tag
rabbitmqctl set_user_tags username administrator
```
