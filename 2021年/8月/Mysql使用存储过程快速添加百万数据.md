# Mysql使用存储过程快速添加百万数据

## 前言
为了测试不加索引和添加索引的区别，需要使用百万级的数据，但是百万数据的表，如果使用一条条添加，特别繁琐又麻烦，这里使用存储过程快速添加数据，用时大概4个小时。
创建一个用户表
```
CREATE TABLE `t_sales` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT '用户名',
  `password` varchar(64) COLLATE utf8_bin DEFAULT NULL COMMENT '密码 MD5存储',
  `register_time` timestamp NULL DEFAULT NULL COMMENT '注册时间',
  `type` int(1) DEFAULT NULL COMMENT '用户类型 1,2,3,4 随机',
  PRIMARY KEY (`id`),
  KEY `idx_username` (`username`) USING BTREE
)
```
然后创建存储过程，批量添加数据。
* 用户名以常量和数字拼接
* 密码是MD5密码
* 注册时间是当前时间随机往前推几天
* type是取1-4随机范围值
```
create procedure salesAdd()
begin 
 declare i int default 11;
   while i <= 4000000 do
         insert into blog.t_sales
         (`username`,`password`,`register_time`,type) values
         (concat("jack",i),MD5(concat("psswe",i)),from_unixtime(unix_timestamp(now()) - floor(rand() * 800000)),floor(1 + rand() * 4)); 
         set i = i + 1; 
   end while; 
end
```
然后调用存储过程
```
call salesAdd()
```


## 改进版
虽然使用存储过程添加数据相对一个个添加更加便捷，快速，但是添加几百万数据要花几个小时时间也是很久的，后面在网上找到不少资料，发现mysql每次执行一条语句都默认自动提交，这个操作非常耗时，所以在在添加去掉自动提交。设置 SET AUTOCOMMIT = 0;
```
create procedure salesAdd()
begin 
 declare i int default 1;
 set autocommit = 0;   
   while i <= 4000000 do
         insert into blog.t_sales
         (`username`,`password`,`register_time`,type) values
         (concat("jack",i),MD5(concat("psswe",i)),from_unixtime(unix_timestamp(now()) - floor(rand() * 800000)),floor(1 + rand() * 4)); 
         set i = i + 1; 
   end while;
 set autocommit = 1;     
end
```
执行时间387秒，约为六分钟，其中还有一半时间用于md5、随机数的计算。
```
[SQL]
call salesAdd();
受影响的行: 0
时间: 387.691s
```



