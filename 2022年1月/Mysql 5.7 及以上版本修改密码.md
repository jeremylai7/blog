# Mysql 5.7 及以上版本修改密码

登录数据后。选择 mysql 数据库
```
use mysql;
```

修改密码
```
update user set authentication_string=PASSWORD("mynewpassword") where user='root';
```
更新密码直接报错
```
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```
设置密码安全机制
```
set global validate_password_policy=0;
```
再更新密码，然后刷新会话
```
flush privileges;
```
