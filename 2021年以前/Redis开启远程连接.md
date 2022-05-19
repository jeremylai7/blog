# Redis开启远程连接

redis 后端运行，修改redis.conf
```
daemonize yes
```
重启运行需要带上配置文件 ,否则配置不生效
```
./redis-server ../redis.conf
```
启动redis
```
cd src
./redis-server ../redis.conf
```
+ 注释 bind 127.0.0.1,使所有的ip都能访问
新版本增加了保护模式，所以需要添加如下关闭
+ protected-mode no
+ 安全起见，关闭保护模式最好设置密码 requirepass 密码  
  

## 关闭redis
redis-cli  shutdown

## 带有密码的redis关闭
redis-cli -a 密码 shutdown

需要输入密码验证  Authentication required

填入 auth 密码
