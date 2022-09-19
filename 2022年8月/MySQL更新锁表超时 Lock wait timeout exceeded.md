# MySQL更新锁表超时 Lock wait timeout exceeded

# 背景

最近在做一个订单的钉钉审批功能，钉钉审批通过之后，订单更新审核状态，然后添加一条付款，并且更新付款状态：
```
// 订单审批通过
@Transactional(rollbackFor = Exception.class)
public void orderPass() {
	// 更新订单审核状态
	updateOrderAuditStatus(id);
	// 添加入库
	addPutInStorage(id);
	// 更新订单入库状态
	updateOrderStorageStatus(id);
}
```

其中的`添加入库`是远程`ERP`入库，添加出库之后`更新出库状态`。因为`ERP`可能因为库存不足，会`入库失败`。但此时审批流程已经结束，不可能再发起一遍审批流程。当`添加入库失败`时`订单审核状态`正常更新，`添加入库`和`更新入库状态`失败。这里的解决方案是：
> 拆分成两个方法，一个是更新订单审核状态，另一个添加入库和更新入库状态。添加入库和更新入库状态开启一个事务，也就是添加`嵌套事务 REQUIRES_NEW`,`REQUIRES_NEW`表示**无论是否有事务，都会创建一个新的事务。**    

修改后的代码如下：
```
// 订单审批通过
@Transactional(rollbackFor = Exception.class)
public void orderPass() {
    // 更新订单审核状态
    updateOrderAuditStatus(id);
    try {
        // 更新出库  
        updatePutInStorage(id);
    } catch (Exception e) {
        System.out.println("更新出库失败");
    }

}

// 更新出库
@Transactional(rollbackFor = Exception.class,propagation = Propagation.REQUIRES_NEW)
public void updatePutInStorage(Long id) throws Exception{
    // 添加入库
    addPutInStorage(id);
    // 更新订单入库状态
    updateOrderStorageStatus(id);
    System.out.println("更新出库成功");
}
```

上面讲代码拆分成`更新订单审核状态`和`更新入库`,其中`更新入库`报错会被`try catch`异常捕获，不会影响到`订单审核状态更新`。而`添加入库`和`更新订单入库状态`处于同一个事务下，要么同时成功，要么同时失败。上述问题也解决了。


**然而运行结果**：
```
com.mysql.cj.jdbc.exceptions.MySQLTransactionRollbackException: Lock wait timeout exceeded; try restarting transaction
```

# 原因分析
锁超时了，为什么会有锁呢？主要是这里添加了`REQUIRES_NEW`。
* 外层事务对表的更新锁住了表的行，外层事务还没有提交，就调用了内层事务`updatePutInStorage`,内层事务调用了`updatePutInStorage`。
* `updatePutInStorage`需要`更新订单的入库状态`,此时外层事务锁住了该表，所以`更新订单的入库状态`无法更新。
* `更新订单的入库状态`等待`更新订单的审核状态`,而`REQUIRES_NEW`又会让`更新订单的审核状态`等待`更新订单的入库状态`。造成相互等待，也就造成`死锁`。


# 解决方案

>死锁：两个线程为了保护两个不同的共享资源而使用了两个互斥锁，那么这两个互斥锁应用不当的时候，可能会造成两个线程都在等待对方释放锁，在没有外力的作用下，这些线程会一直相互等待，就没办法继续运行，这种情况就是发生了死锁。

上面锁超时原因，就是死锁的一种原因。所以需要把`更新订单审核状态`方法放在最后:

```
// 订单审批通过
@Transactional(rollbackFor = Exception.class)
public void orderPass() {
    
    try {
        // 更新出库  
        updatePutInStorage(id);
    } catch (Exception e) {
        System.out.println("更新出库失败");
    }
    // 更新订单审核状态
    updateOrderAuditStatus(id);

}

// 更新出库
@Transactional(rollbackFor = Exception.class,propagation = Propagation.REQUIRES_NEW)
public void updatePutInStorage(Long id) throws Exception{
    // 添加入库
    addPutInStorage(id);
    // 更新订单入库状态
    updateOrderStorageStatus(id);
    System.out.println("更新出库成功");
}
```

# 总结
* 添加嵌套事务需要考虑到`死锁`的问题。
* 一个事务只有等全部方法执行完毕之后才会提交事务。
* 含有嵌套的事务的更新，需要按照相同的顺序更新，不然可能会出现锁相互等待的情况。

# 参考
[业务上第一次遇到MySQL更新锁表超时（ Lock wait timeout exceeded; try restarting transaction）](https://www.codenong.com/cs105584216/)


