# Mysql 创建索引语句

mysql有哪些索引
* index 普通索引
  * alter table `table_name` add index index_name(`column`) 
  * 最基本的索引，没有任何限制
* primary key 主键索引
  * alter table `table_name` add primary key(`column`)
  *  是一种特殊的唯一索引，不允许有空值
* unique 唯一索引
  *  alter table `table_name` add unique(`column`)
  *  与“普通索引”类似，不同的就是，索引列的值必须是唯一，但允许有空值 
* filltext 全文索引
   * alter table `table_name` add fulltext(`column`)
   * 仅可用于MyISAM表，针对较大的数据，生成全文索引很耗时和耗空间
* 组合索引
    * alter table `table_name` add index index_name(`column1`,`column2`,`column3`)
    * 遵循“最左前缀”原则

## 创建索引
```
create index index_name on table_name(column_name)
```
## 修改表结构（添加索引）
```
alter table table_name add index index_name(column_name)
```
## 创建表时直接指定
```
create table table_name(
  id int not null,
  username varchar(64) not null,
  index [index_name] (username)  
);
```
## 删除索引
```
drop index [index_name] on table_name
```
