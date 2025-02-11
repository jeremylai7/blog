# java.sql.SQLException: MONTH 报错解决方法

`idea`控制台报错:`java.sql.SQLException: MONTH`

```
Error attempting to get column 'xxx' from result set.  Cause: java.sql.SQLException: MONTH
; MONTH; nested exception is java.sql.SQLException: MONTH] with root cause
```

# 分析原因：

`sql`查询结果无法转换成`Java`的`Date`类型，这里是月份无法转换，因为数据库的时间是:
```
2020-00-29 00:00:00
```

> 月份无法转换，所以`Java`就报错了。时间改成正确格式即可。

# 总结

* `java.sql.SQLException: MONTH` 报错。先定位到对应字段，如果无法确认问题原因，先去掉字段，去掉后没有问题，应该就是时间字段的问题。
* 然后查看时间，就能确认是格式问题，报错中`MONTH`说明是时间无法转换。
