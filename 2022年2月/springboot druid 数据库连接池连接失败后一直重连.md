# springboot druid 数据库连接池连接失败后一直重连

在使用个人阿里云测试机，在查询实时输出日志时，看到数据库连接失败后，服务器一直在重连服务器。开始以为是遭受重复攻击，后面把服务重启后，就没有出现一直重连的情况。看以下输出日志：
```
2022-02-09 11:04:58.896 ERROR 16876 --- [eate-1550991149] com.alibaba.druid.pool.DruidDataSource   : create connection SQLException, url: jdbc:mysql://47.98.67,98:1234/test?useSSL=false&characterEncoding=UTF-8&serverTimezone=UTC, errorCode 1045, state 28000

java.sql.SQLException: Access denied for user 'root'@'113.90.123.76' (using password: YES)
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:129) ~[mysql-connector-java-8.0.16.jar:8.0.16]
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:97) ~[mysql-connector-java-8.0.16.jar:8.0.16]
	at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:122) ~[mysql-connector-java-8.0.16.jar:8.0.16]
	at com.mysql.cj.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:835) ~[mysql-connector-java-8.0.16.jar:8.0.16]
	at com.mysql.cj.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:455) ~[mysql-connector-java-8.0.16.jar:8.0.16]
	at com.mysql.cj.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:240) ~[mysql-connector-java-8.0.16.jar:8.0.16]
	at com.mysql.cj.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:199) ~[mysql-connector-java-8.0.16.jar:8.0.16]
	at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:156) ~[druid-1.1.10.jar:1.1.10]
	at com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:218) ~[druid-1.1.10.jar:1.1.10]
	at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:150) ~[druid-1.1.10.jar:1.1.10]
	at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1560) ~[druid-1.1.10.jar:1.1.10]
	at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1623) ~[druid-1.1.10.jar:1.1.10]
	at com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:2468) ~[druid-1.1.10.jar:1.1.10]
```
注意上面一直有 `druid` 数据库连接池的提示，**这里就想到可能是 `druid` 连接池的问题，然后去掉 `druid`  maven 依赖后在请求接口就不会出现重连的问题。**

## druid 重连原因
在上图源码找到最后一行 `DruidDataSource.java:2468` 定位到源码上，`CreateConnectionThread`  创建连接线程，看一下 `CreateConnectionThread ` 源码：
```
public class CreateConnectionThread extends Thread {

        public CreateConnectionThread(String name){
            super(name);
            this.setDaemon(true);
        }

        public void run() {
            initedLatch.countDown();

            long lastDiscardCount = 0;
            int errorCount = 0;
            for (;;) {
                // addLast
                try {
                    lock.lockInterruptibly();
                } catch (InterruptedException e2) {
                    break;
                }

                long discardCount = DruidDataSource.this.discardCount;
                boolean discardChanged = discardCount - lastDiscardCount > 0;
                lastDiscardCount = discardCount;

                try {
                    boolean emptyWait = true;

                    if (createError != null
                            && poolingCount == 0
                            && !discardChanged) {
                        emptyWait = false;
                    }

                    if (emptyWait
                            && asyncInit && createCount < initialSize) {
                        emptyWait = false;
                    }

                    if (emptyWait) {
                        // 必须存在线程等待，才创建连接
                        if (poolingCount >= notEmptyWaitThreadCount //
                                && !(keepAlive && activeCount + poolingCount < minIdle)) {
                            empty.await();
                        }

                        // 防止创建超过maxActive数量的连接
                        if (activeCount + poolingCount >= maxActive) {
                            empty.await();
                            continue;
                        }
                    }

                } catch (InterruptedException e) {
                    lastCreateError = e;
                    lastErrorTimeMillis = System.currentTimeMillis();

                    if (!closing) {
                        LOG.error("create connection Thread Interrupted, url: " + jdbcUrl, e);
                    }
                    break;
                } finally {
                    lock.unlock();
                }

                PhysicalConnectionInfo connection = null;

                try {
                    connection = createPhysicalConnection();
                    setFailContinuous(false);
                } catch (SQLException e) {
                    LOG.error("create connection SQLException, url: " + jdbcUrl + ", errorCode " + e.getErrorCode()
                              + ", state " + e.getSQLState(), e);

                    errorCount++;
                    if (errorCount > connectionErrorRetryAttempts && timeBetweenConnectErrorMillis > 0) {
                        // fail over retry attempts
                        setFailContinuous(true);
                        if (failFast) {
                            lock.lock();
                            try {
                                notEmpty.signalAll();
                            } finally {
                                lock.unlock();
                            }
                        }

                        if (breakAfterAcquireFailure) {
                            break;
                        }

                        try {
                            Thread.sleep(timeBetweenConnectErrorMillis);
                        } catch (InterruptedException interruptEx) {
                            break;
                        }
                    }
                } catch (RuntimeException e) {
                    LOG.error("create connection RuntimeException", e);
                    setFailContinuous(true);
                    continue;
                } catch (Error e) {
                    LOG.error("create connection Error", e);
                    setFailContinuous(true);
                    break;
                }

                if (connection == null) {
                    continue;
                }

                boolean result = put(connection);
                if (!result) {
                    JdbcUtils.close(connection.getPhysicalConnection());
                    LOG.info("put physical connection to pool failed.");
                }

                errorCount = 0; // reset errorCount
            }
        }
    }

```

这是一个多线程的类，而 `run` 方法里面设置了没有限制的 for 循环 ` for (;;) {}`, 而日志报错定位的信息:
```
connection = createPhysicalConnection();
```
如果符合条件 `errorCount > connectionErrorRetryAttempts && timeBetweenConnectErrorMillis > 0` 会再次尝试重连，先看一下这几个参数的含义：
### errorCount 错误次数
在 run 方法初始化时为零，每次连接失败，会自动加1 
### connectionErrorRetryAttempts  
连接错误重试次数，默认值为 1。
```
protected int  connectionErrorRetryAttempts  = 1;
```
### timeBetweenConnectErrorMillis 
连接间隔时间，单位毫秒。默认值为 500。
```
protected volatile long timeBetweenConnectErrorMillis = DEFAULT_TIME_BETWEEN_CONNECT_ERROR_MILLIS;
public static final long DEFAULT_TIME_BETWEEN_CONNECT_ERROR_MILLIS = 500;
```
我们在连接数据库失败后，要不在里面 `break` 中断，其中有
```
if (breakAfterAcquireFailure) {
     break;
}
```
将改 `break-after-acquire-failure` 设置成 `true`,在 application.properties 文件如下配置：
```
spring.datasource.druid.break-after-acquire-failure=true
```
如果想多尝试连接几次，需要设置 `connection-error-retry-attempts` ,当 `errorCount ` 大于 `connectionErrorRetryAttempts` 才会进入到 条件内，才会中断循环。在 application.properties 文件如下配置：
```
spring.datasource.druid.connection-error-retry-attempts=3
```

## 总结
druid 数据库连接失败，是因为在使用多线程连接数据时使用了无限制循环连接，需要在连接失败中断连接，需要设置 `break-after-acquire-failure` 为 `true`。设置之后数据库连接不成功也不会不断的重试。如果要设置重连次数要设置 `connection-error-retry-attempts`。
