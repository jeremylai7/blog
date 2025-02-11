# Mysql 使用 group by 不对 null 做分组

>在项目开发查询数据需要将相同的数据做合并处理，但是字段为null，不做合并。

## 创建表以及添加数据
```
create table t_student(
       `id` int not null primary key auto_increment,
       `name` varchar(32) ,
       `age` int   
)
insert into t_student(`name`,age) values("aa",11);
insert into t_student(`name`,age) values('bb',12);
insert into t_student(`name`,age) values('cc',13);
insert into t_student(`name`,age) values('cc',14);
insert into t_student(`name`,age) values('cc',15);
insert into t_student(`name`,age) values(null,16);
insert into t_student(`name`,age) values(null,17);
```
查询数据一共有7条数据
```
select * from t_student
```
结果:

![image](https://user-images.githubusercontent.com/11553237/169219593-7d31c5f5-bd06-4d96-9d83-f7d4f62283ec.png)

再做**name合并**
```
select * from t_student group by name
```
结果:

![image](https://user-images.githubusercontent.com/11553237/169219642-9676427e-e8ef-41c7-83da-91a9126ffd75.png)

结果把**全部null**合并在一起了。

### 解决方案 使用替换UUID()
在 [https://stackoverflow.com/questions/4588935/group-by-do-not-group-null](https://stackoverflow.com/questions/4588935/group-by-do-not-group-null)上看到了一个方法。
做分组的时候如果name为null时，对null设置成一个**随机值UUID()**,这样就避免了null会合并的情况。
使用**UUID()**:
```
select * from t_student group by IFNULL(name,UUID())
```
结果：

![image](https://user-images.githubusercontent.com/11553237/169219687-0733b528-2c58-42c9-b9c2-3a2100bf5054.png)
