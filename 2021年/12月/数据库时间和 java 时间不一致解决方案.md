# 数据库时间和 java 时间不一致解决方案

## java添加 date 到数据库，时间不一致
使用 date 添加到数据库，数据库显示的时候和date时间相差 8 个小时，这是由于 mysql 上的时区的问题，这里有两个解决方案:

### 方案一： 设置数据库连接时区
在 springboot 的配置文件中的 `spring.datasource.url` 添加后缀 `serverTimezone=Asia/Shanghai`。设置本地时区。

### 方案二： 设置 mysql 时区
查看 mysql 时区：
```
show variables like "%time_zone%";
```
这里分成修改当前会话和全局会话
* 修改当前会话只对当前会话有效，退出会话就失效
* 修改全局会话是要退出当前会话后才有效

修改当前会话：
```
SET time_zone = "+8:00";
```
修改全局会话：
```
SET global time_zone = "+8:00";
```
这里最好修改**全局会话**。



## java 获取 date 时间和前端展示不一致

后端时间和数据库时间相差 8 个小时

### 原因
springboot 中的@RestController 注解接口返回 json 格式数据，对于 date 类型的数据，会被 spring-boot 默认的Jackson框架转化，而 Jackson 框架默认时区是 GMT（相对于中国少了 8 个小时）。

### 解决方案
在 `application.yml` 添加配置：
```
spring:
  jackson:
    time-zone: GMT+8
```
