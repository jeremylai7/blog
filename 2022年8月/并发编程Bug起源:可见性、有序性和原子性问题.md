# 并发编程Bug起源:可见性、有序性和原子性问题

以前古老的`DOS`操作系统，是单进行的系统。系统每次只能做一件事情，完成了一个任务才能继续下一个任务。每次只能做一件事情，比如在听歌的时候不能打开网页。所有的任务操作都按照串行的方式依次执行。

![image](https://user-images.githubusercontent.com/11553237/187809305-77f47e67-a1c3-48bf-9f07-1c44cd722dc3.png)

>这类服务器缺点也很明显，等待操作的过长，无法同时操作多个任务，执行效率很差。

现在的操作系统都是多任务的操作系统，比如听歌的时候可以做打开网页，还能打开微信和朋友聊天。这几个任务可以同时进行，大大增加执行效率。

# 并发提高效率

一个完整服务器，都有`CPU`、`内存`、`IO`,三者之间的运行速度存在明显的差异:
* `CPU`相关的操作，执行指令以及读取`CPU`缓存等操作，基本都是`纳秒`级别的。
* `CPU`读取内存，耗时是`CPU`相关操作的**千倍**，基本都是`微秒`级别。`CPU`和内存之间的速度差异。
* `IO`操作基本是毫秒的级别，是内存操作的千倍，`内存`和`IO`之间存在速度的差异。

>CPU -> 内存 -> SSD -> 磁盘 -> 网络  
纳秒 -> 微秒 -> 毫秒 -> 毫秒 -> 秒


程序中大部分的语句都要访问内存，有些还要访问的`IO`读写。为了合理的利用`CPU`的高性能，高效的平衡三者的速度差异，操作系统、编译器主要做了以下改进：

* `CPU`增加了`CPU缓存`，用来均衡`CPU`和`内存`的速度差异。
* 操作系统增加了多进程、多线程，用来分时复用`CPU`,从而均衡`CPU`与`IO`设备之间的差异。
* 编译优化程序执行顺序，充分利用缓存。

做了以上操作之后，`CPU`读取或者修改数据之后，将数据缓存在`CPU缓存`中，`CPU`不需要每次都从内存中获取数据，极大的提高了`CPU`的运行速度。多线程是将时间段切成一个个小段，多个线程在上下文切换中，执行完任务，而不用等前面的线程都执行完毕之后再执行。比如做一个计算，`CPU`耗时`1`纳秒,而从内存读取数据要`1`微秒，没有多线程的话,`N`个线程要耗时`N微秒`，此时`CPU`高效性就无法体现出来。有了多线程之后，操作系统将`CPU`时间段切成一个一个小段，多线程上下文切换，线程执行计算操作，无需等待`内存读取操作`。

虽然并发可以提高程序的运行效率，但是凡事有利也有弊，并发程序也有很多诡异的`bug`,根源有以下几个原因。


# 缓存导致可见性问题

一个线程对共享变量的修改，另外线程能立刻看到，称为**可见性**。

在单核时代，所有的线程都是在同一个`CPU`上运行，所有的线程都是操作同一个线程的`CPU缓存`，一个线程修改缓存，对另外一个线程来说一定是可见的。比如在下图中，`线程A`和`线程B`都是操作同一个`CPU缓存`，所以`线程A`更新了`变量V`的值，`线程B`再访问`变量V`的值，获取的一定是`V`的最新值。所以变量`V`对线程都是**可见的**。

![image](https://user-images.githubusercontent.com/11553237/187809335-35f1579b-3377-4273-a3b6-ca2eb5385f71.png)

在多核`CPU`下，每个`CPU`都有自己的缓存。当多个线程执行在不同的`CPU`时，这些线程的操作也是在对应的`CPU缓存`上。这时候就会出现问题了，在下图中，`线程A`运行在`CPU_1`上，首先从`CPU_1`缓存获取`变量V`，获取不到就获取内存的值，然后操作`变量V`。`线程B`也是同样的方式在`CPU_2`缓存中获取`变量V`。

`线程A`操作的是`CPU_1`的缓存，`线程B`操作的是`CPU_2`的缓存，此时`线程A`对`变量V`的操作对于`线程B`是**不可见的**。多核CPU一方面提高了运行速度，但是另一方面也可能会造成**线程不安全**的问题。


![image](https://user-images.githubusercontent.com/11553237/187809367-fc46e810-d2f7-4962-aa98-4014f4b01745.png)

下面使用一段代码来测试多核场景下的可见性。首先创建一个累加的方法`add10k`方法，循环`10000`次`count+=1`的操作。然后在`test`方法里面创建两个线程，每个线程都调用`add10k`方法，结果是多少呢？

```
public class VisibilityTest {

	private  static int count = 0;

	private void add10k() {
		int index = 0;
		while (index++ < 10000) {
			count += 1;
		}
	}

	@Test
	public void test() throws InterruptedException {
		VisibilityTest test = new VisibilityTest();
		Thread thread1 = new Thread(() -> test.add10k());
		Thread thread2 = new Thread(() -> test.add10k());
		// 启动两个线程
		thread1.start();
		thread2.start();
		// 等待两个线程执行结束
		thread1.join();
		thread2.join();
		System.out.println(count);
	}
}
```

按照直觉来说结果是`20000`,因为在每个线程累加`10000`,两个线程就是`20000`。但是实际结果是介于`10000~20000`的之间，每次执行结果都是这个范围内的随机数。

因为线程A和线程B同时开始执行，第一次都会将`count=0`缓存到自己的`CPU缓存`中，执行完`count += 1`之后，写入自己对应的`CPU缓存`中，同时写入内存中，此时内存中的数是`1`，而不是期望的`2`。之后`CPU`再取到自己的`CPU缓存`再进行计算，最后计算出来的`count`值都是小于`20000`,这就是缓存的可见性问题。

# 线程切换带来的原子性问题

上面提到，由于`CPU`、`内存`、`IO`之间的速度存在很大的差异，在单进程系统中，需要等速度最慢的`IO`操作完成之后，才能接着完成下一个任务，`CPU`的高性能也无法体现出来。但操作系统有了多进程之后，操作系统将`CPU`切成一个一个小片段，在不同的时间片段内执行不同的进程的，而不需要等待速度慢的`IO`操作，在单核或者多核的`CPU`上可以一边的听歌，一边的聊天。

操作系统将时间切成很小片，比例`20`毫秒,开始的`20`毫秒执行一个进程，下一个`20`毫秒切换执行另外一个线程，`20`毫秒成为`时间片`,如下图所示：


![image](https://user-images.githubusercontent.com/11553237/187809397-8902a4c2-9ee5-4aa3-9c01-aa6d95ac0029.png)

`线程A`和`线程B`来回的切换任务。

如果一个进行`IO`操作，例如读取文件，这个时候该进程就把自己标记为`休眠状态`并让出`CPU`的使用权，等完成`IO`操作之后，又需要使用`CPU`时又会把休眠的进程唤醒，唤醒的进程就可以等待`CPU`的调用了。让出`CPU`的使用权之后，`CPU`就可以对其他进程进行操作，这样`CPU`的使用率就提高上了，系统整体的运行速度也快了很多。

并发程序大多数都是基于多线程的，也会涉及到**线程上下文的切换**，线程的切换都是在很短的时间片段内完成的。比如上面代码中`count += 1`虽然有一行语句，但这里面就有三条`CPU`指令。

* 指令 1：把变量V从内存加载到`CPU`寄存器中。
* 指令 2：在寄存器中执行`+1`操作。
* 指令 3：将结果写入内存（也可能是写入`CPU缓存中`）。

任何一条`CPU`指令都可能发生`线程切换`。如果线程A在指令1执行完后做线程切换，线程A和线程B按照下图顺序执行，那么我们会发现两个线程都执行`count += 1`的操作，但是最后结果却是`1`，而不是`2`。

![image](https://user-images.githubusercontent.com/11553237/187809424-1edbcc8d-42e2-4ee5-998e-b42966b985bc.png)


# 编译优化带来的有序性问题

有序性是指程序按照代码的先后顺序执行，编译器为了优化性能，在不影响程序的最终结果的情况下，编译器调整了语句的先后顺序，比如程序中：
```
a = 2;
b = 5;
```
编译器优化后可能变成:
```
b = 5;
a = 2;
```

虽然不影响程序的最后结果，但是也会引起一些意想不到的BUG。

在`Java`中一个常见的例子就是利用双重检验创建单例对象，例如下面的代码：
```

public class Singleton {
  static Singleton instance;
  static Singleton getInstance(){
    if (instance == null) {
      synchronized(Singleton.class) {
        if (instance == null)
          instance = new Singleton();
        }
    }
    return instance;
  }
}
```

在获取实例`getInstance`方法中，首先判断`instance`是否为空，如果为空，则锁定`Singleton.class`并再次检查`instance`是否为空，如果还为空就创建一个`Singleton`实例。

假设两个线程，`线程A`和`线程B`同时调用`getInstance`方法。此时`instance == null`，同时对`Singleton.class`加锁，`JVM`保证只有一个线程能加锁成功，假设是`线程A`加锁成功，另一个线程就会处于等待状态，`线程A`会创建一个实例，然后释放锁，`线程B`被唤醒，再次尝试加锁，此时成功加锁，而此时`instance != null`,已经创建过实例，所以`线程B`就不会创建实例了。


看起来没有什么问题，但实际上也有可能问题出现在`new`操作上，本来`new`操作应该是：

* 1、分配一块内存。
* 2、在内存上初始化对象。
* 3、内存的地址赋值给`instance`变量。

但实际优化后的执行顺序却是如下：

* 1、分配一块内存。
* 2、将内存地址赋值给`instance`变量。
* 3、在内存上初始化对象。

优化之后会发生什么问题呢？首先假设`线程A`先执行`getInstance`方法，也就是先执行`new`操作，当执行完指令`2`时发生了线程切换，切换到`线程B`上，此时线程B执行`getInstance`方法，执行判断时会发现`instance != null`,所以就返回`instance`，而此时的`instance`是没有初始化的，如果这时访问`instance`就可能会触发空指针异常。


![image](https://user-images.githubusercontent.com/11553237/187809262-752eafdd-5683-49e5-aed9-b94113c33efb.png)

# 总结

操作系统进入多核、多进程、多线程时代，这些升级会很大的提高程序的执行效率，但同时也会引发`可见性`、`原子性`、`有序性`问题。
* 多核`CPU`，每个CPU都有各自的CPU缓存，每个线程更新变量会先同步在`CPU缓存`中，而此时其他线程，无法获取最新的`CPU`缓存值，这就是不可见性。
* `count += 1`含有多个`CPU`指令。当发生线程切换，会导致原子问题。
* 编译优化器会调整程序的执行顺序，导致在多线程环境，线程切换带来有序的问题。

开始学习并发，经常会看到`volatile`、`synchronized`等并发关键字，而了解并发编程的有序性、原子性、可见性等问题，就能更好的理解并发场景下的原理。

# 参考

[可见性、原子性和有序性问题：并发编程Bug的源头](https://time.geekbang.org/column/article/83682)

