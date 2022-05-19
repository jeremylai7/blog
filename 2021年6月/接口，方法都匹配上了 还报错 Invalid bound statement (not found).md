> Invalid bound statement (not found)这个问题的实质就是mapper接口和mapper.xml没有映射起来

但是接口和方法都匹配上了，还是报错。可能是 dao 或者 mapper 没有扫描上。

## dao 扫描

在Application使用 MapperScan 注解扫描

## mapper 扫描

在配置文件 application.yml 添加以下配置
```
mybatis:
  mapper-locations: classpath:mapper/*.xml
```
