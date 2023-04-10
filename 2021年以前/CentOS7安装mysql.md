# CentOS7安装mysql

## 查看mysql是否安装
```
[root@localhost ~]# rpm -qa|grep mysql
```
如果已安装，而且不是自己需要的版本，卸载，命令如下：
```
[root@localhost ~]# yum remove mysql mysql-server mysql-libs mysql-common
[root@localhost ~]# rm -rf /var/lib/mysql
[root@localhost ~]# rm /etc/my.cnf
```
## 下载和安装mysql的repo源

~~wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm~~
~~yum -y localinstall mysql57-community-release-el7-11.noarch.rpm~~
```
[root@localhost ~]# wget https://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm
[root@localhost ~]# yum -y localinstall mysql80-community-release-el7-3.noarch.rpm
```
## 下载和安装mysql
```
[root@localhost ~]# yum install mysql-community-server
```

安装报错：

```
The GPG keys listed for the "MySQL 5.7 Community Server" repository are already installed but they are not correct for this package.
Check that the correct key URLs are configured for this repository.


 Failing package is: mysql-community-client-5.7.39-1.el7.x86_64
 GPG Keys are configured as: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

解决办法：

```
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```


## 启动mysql
```
[root@localhost ~]# service mysqld start
```
mysql5.7安装之后会在日志中生成一个随机密码,日志默认路径为/var/log/mysqld.log
获取默认密码
```
[root@localhost ~]# grep "password" /var/log/mysqld.log
```
获取密码后登录mysql。
```
[root@localhost ~]# mysql -uroot -p密码
```
登录mysql后查询数据会报错误，提示需要重置密码
```
mysql> select user();
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```
修改简单密码又会报如下错误：
```
mysql> ALTER USER USER() IDENTIFIED BY '123456';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

# mysql 5.7

需要修改Policy
```
mysql> set global validate_password_policy=0;
```
然后设置设置密码(默认为8，最小4位)：
```
mysql> set global validate_password_length=6;
```
最后设置密码
```
mysql>  ALTER USER USER() IDENTIFIED BY '123456';
```

# mysql 8.0

```
mysql> set global validate_password.policy=0;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```

  `mysql 8.0`修改密码之前，不能做任何修改。
  
先设置一个符合要求的密码

```
ALTER USER USER() IDENTIFIED BY 'A_123456'
```
查看修改项：

```
SHOW VARIABLES LIKE 'validate_password%';
```

![image](https://user-images.githubusercontent.com/11553237/223953887-cd85b7b2-ebda-4d20-8b79-f3ab72aa0d8f.png)


需要修改Policy:

```
mysql> set global validate_password.policy=0;
mysql> set global validate_password.length=1;
```

最后修改你想要设置的密码：

```
ALTER USER USER() IDENTIFIED BY '123456'
```


# 开启远程连接
有以下两个方法:
1.如果要直接使用root用户远程连接,直接执行以下sql
```
mysql>update mysql.user set host='%' where user='root';
```
线上环境建议不要开启root用户远程访问，使用下面方法创建一个用户用来远程连接

2.设置固定账号开启远程
```
mysql>GRANT ALL PRIVILEGES ON *.* TO '这里写账号'@'%' IDENTIFIED BY '这里写密码' WITH GRANT OPTION;
```
设置完了后：
```
mysql>flush privileges;  
```
**这句一定要加上！！！**
