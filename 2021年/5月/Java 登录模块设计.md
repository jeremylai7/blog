# Java 登录模块设计

## 登录流程
* 前端登录传输用户名和md5加密后的密码
* 后端对密码在进行md5加密，或者使用md5加密的密码 + id 进行盐加密，增加密码被破解的难度。
* 登录成功后，这里分成单体，或者分布式的情况
    * 单体
        * 单体比较简单，登录成功后，在后端在 session 里面 setAttribute 存入用户信息。
        * 后续登录在拦截器获取session ，查询session。
    * 服务器集群、服务器分布式
        * 存在 **session 共享**的问题。使用 jwt 解决问题，使用ip，用户信息生成 token 值，返回给前端。
        * 在登录拦截器，解析每次传过来的token，匹配ip 和用户信息，用户信息与数据库校验。但是都查数据库，会增加数据库压力，所以也可以把 token 存入redis里面，解析后用户信息和 redis 里的token 信息检验。
## 有选择性使用验证登录信息
* 对于http请求来说，有的请求需要登录后才能访问，有的不需要登录后就能直接访问，这里使用注解的方式，在需要登录才能访问的接口使用自定义注解 @login，注解在类或方法上。当客户端请求后端接口，如果类或方法上没有login注解，则不会坐用户信息校验。

## 总结
前端密码使用 md5 密码传输到后端，后端对密码进行盐加密。针对分布式服务，或者集群服务，使用jwt生成token，进行后续数据校验，后续数据校验一般使用拦截器校验。针对不是所有接口都需数据校验，在需要登录后才能访问的接口，在类上添加login注解，没有添加注解的类不进行数据校验。


[github 登录源码](https://github.com/jeremylai7/springboot-project-seed/blob/master/demo-admin/src/main/java/com/jeremy/admin/controller/LoginController.java)
[github 拦截拦截器源码](https://github.com/jeremylai7/springboot-project-seed/blob/master/demo-admin/src/main/java/com/jeremy/admin/interceptor/MyInterceptor.java)
