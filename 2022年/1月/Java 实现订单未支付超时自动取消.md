# Java 实现订单未支付超时自动取消

在电商上购买商品后，如果在下单而又没有支付的情况下，一般提示30分钟完成支付，否则订单自动。比如在京东下单为完成支付：

![image](https://user-images.githubusercontent.com/11553237/169225714-ba69c0de-240f-42f7-b6a4-fa8d04244b09.png)

超过24小时，就会自动取消订单，下面使用 Java 定时器实现超时取消订单功能。

* Timer 定时器
Timer 是一个调度任务的执行的工具，任务可以一次性定时执行或者定时重复执行，系统会启动一个线程来执行所有的定时任务。

* TimerTask 定时任务

TimerTask 是一个抽象类，它实现了 Runnable，实现 Runnable 也就是创建了多线程任务。

## 创建 TimerTask 

TimerTask 是抽象类，抽象是为了代码复用，要创建一个类继承 TimerTask：
```
public class CancelOrderTimeTask extends TimerTask {

    private Long id;

    public CancelOrderTimeTask(long id) {
        this.id = id;
    }


    @Override
    public void run() {
        // 执行取消订单
        cancelOrder(id); 
        System.out.println(getCurrentTime() + " 时间取消订单,订单id：" + id);

    }

    private String getCurrentTime() {
        SimpleDateFormat sdf = new SimpleDateFormat();
        sdf.applyPattern("yyyy-MM-dd HH:mm:ss");
        Date date = new Date();
        return sdf.format(date);
    }
}
```
 在 `run` 方法执行订单取消任务。
>因为这个方法不是创建 bean，所以在调用的 dao 或者其他的 bean 时，要使用 `ApplicationContext` 获取 bean。

## Timer 定时器调用 TimerTask
 新建 Timer，使用 schedule 方法调用，方法有两个参数，第一个是任务的实例，另一个是延迟多久后调用任务，单位是毫秒。代码如下：
```
@RestController
public class TimerController {


    @GetMapping("/timer")
    public String timer(long id) {
        Timer timer = new Timer();
        CancelOrderTimeTask timeTask = new CancelOrderTimeTask(id);
        System.out.println("当前时间是" + getCurrentTime());
        //10秒后执行任务
        timer.schedule(timeTask,10 * 1000);
        return "ok";
    }

    private String getCurrentTime() {
        SimpleDateFormat sdf = new SimpleDateFormat();
        sdf.applyPattern("yyyy-MM-dd HH:mm:ss");
        Date date = new Date();
        return sdf.format(date);
    }
}

``` 
调用方法后，控制台输出如下内容，说明定时调用成功。
```
当前时间是2022-01-24 00:05:09
2022-01-24 00:05:19 时间取消订单,订单id：3
```

## 总结
* 首先创建定时任务，继承 TimerTask，在 run 方法里面写业务逻辑。
* 使用 Timer 调用 schedule 方法, schedule 方法写入 TimerTask 实例以及延迟时间。

## 源码
[github源码](https://github.com/jeremylai7/springboot-learning/blob/master/springboot-test/src/main/java/com/test/controller/TimerController.java)
