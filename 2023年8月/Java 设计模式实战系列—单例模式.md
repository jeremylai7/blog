> 本文首发公众号：小码A梦

单例模式是设计模式中最简单一个设计模式，该模式属于创建型模式，它提供了一种创建实例的最佳方式。

单例模式的定义也比较简单：一个类只能允许创建一个对象或者实例，那么这个类就是单例类，这种设计模式就叫做单例模式。

单例模式有哪些好处：

* 类的创建，特别是一个大型的类，只创建一个类，避免内存和 CPU 的开销。
* 降低内存使用，减少 GC 次数，避免 GC 的压力。
* 避免对资源的重复请求。
* 避免创建多个实例引起系统的混乱或者系统数据冲突。


# ID 生成器单例类实战

单例模式，其中的 "例" 表示 "实例" ，一个类需要保证仅有一个实例，并提供一个访问它的全局访问点,实现一个单例，需要符合以下几点要求：

* 构造函数需要设置 private 权限，避免外部通过 new 创建实例，通过一个静态方法给其他类获取实例。
* 对象创建需要考虑线程安全问题。
* 需要考虑延迟加载问题。
* 外部类获取实例需要考虑性能方法。

在电商系统的订单模块，每次下单都需要生成新的订单号。就需要调用订单号生成器。

![](https://files.mdnice.com/user/29864/92e90c21-0a20-4d8e-ab20-f8adc3bcbe46.png)


1、 创建一个简单单例类

```
public class SnGenerator {

    private AtomicLong id = new AtomicLong(0);
    // 创建一个 Singleton 对象
    private static SnGenerator instance = new SnGenerator();

    // 构造函数设置为 private，类就无法被实例化
    private SnGenerator() {}

    // 获取唯一实例
    public static SnGenerator getInstance() {
        return instance;
    }

    public long getSn() {
        return id.incrementAndGet();
    }
}
```

2、获取 Singleton 类的唯一实例

```
public class SingletonTest {

    public static void main(String[] args) {
        // 编译报错，因为 Singleton 构造函数是私有的
        //Singleton singleton = new Singleton();

        SnGenerator snGenerator = SnGenerator.getInstance();
        for (int i = 0; i < 10; i++) {
            System.out.println(snGenerator.getSn());
        }

    }
}
```

控制台输出生成的 id：

```
1
2
3
4
5
6
7
8
9
10
```

以上首先创建一个单例类，提供唯一的单例获取方法 getInstance。SingletonTest 类通过 `Singleton.getInstance` 获取实例，获取到实例，也就获取到实例所有的方法。示例中调用 getSn 方法，获取到唯一的订单号了。

![](https://files.mdnice.com/user/29864/cd0c6d0e-5861-4ce4-9656-fc5384866e84.png)


## 饿汉单例

饿汉单例实现起来比较简单，**所谓 "饿汉" 重点在饿，开始就需要创建单例**。在类加载时，就创建好了实例。所以 instance 实例的创建是线程安全，不存在线程安全问题。但是这种方式不支持延迟加载，类加载时占用的内存就比较高。

```
public class SnGenerator {

    private AtomicLong id = new AtomicLong(0);

    private static SnGenerator instance = new SnGenerator();

    private SnGenerator() {}

    public static SnGenerator getInstance() {
        return instance;
    }

    public long getSn() {
        return id.incrementAndGet();
    }
}
```

饿汉单例解决线程安全问题，项目启动时就创建好了实例，就需要考虑创建和获取实例的线程安全问题。但是不支持延迟，如果实例的占用内存比较大，或者实例加载时间比较长，类加载的时候就创建实例，就比较浪费内存或者增加项目启动时间。

>对饿汉单例来说，不支持延迟加载，确实是比较浪费内存。但是一个实例内存相对于一个 Java 项目内存占用影响是微乎其微。部署服务端项目时会分配几倍于项目启动占用的内存，所以饿汉单例占用内存还是可以接受的。而且如果占用内存比较大，初始化实例也可以发现内存不足的问题，并及时的处理。避免程序运行后，再去初始化实例，导致系统内存溢出，影响系统稳定性。

## 懒汉单例

既然饿汉单例单例不支持延迟加载，那我们就介绍一下支持延迟的加载的单例：懒汉单例。**所谓"懒汉"重点在懒，一开始是不会初始化实例，而等到被调用才会初始化单例**。

```
public class LazySnGenerator {

    private AtomicLong id = new AtomicLong(0);

    private static LazySnGenerator instance;

    // 构造函数设置为 private，类就无法被实例化
    private LazySnGenerator() {}

    // 获取唯一实例
    public static LazySnGenerator getInstance() {
        if (instance == null) {
            instance = new LazySnGenerator();
        }
        return instance;
    }

    public long getSn() {
        return id.incrementAndGet();
    }

}
```

上面的懒汉单例最开始不会初始化实例，而且等到 getInstance 方法被调用时，才会时候实例，这样支持懒加载的方式，优点是不占内存。

但是懒汉单例缺点也比较明显，在多线程环境下，getInstance 方法不是线程安全的。

打个比方，多个线程同时执行到 `if (instance == null)`结果都为 true,进而都会创建实例，所以上面的懒汉单例不是线程安全的实例。

## 加同步锁的懒汉单例

懒汉单例存在多线程安全问题，第一想到的就是给 getInstance 添加同步锁，添加锁后，保证了线程的安全。

```

public class LazySnGenerator {

    private AtomicLong id = new AtomicLong(0);

    private static LazySnGenerator instance;

    // 构造函数设置为 private，类就无法被实例化
    private LazySnGenerator() {}

    // 获取唯一实例
    public synchronized static LazySnGenerator getInstance() {
        if (instance == null) {
            instance = new LazySnGenerator();
        }
        return instance;
    }

    public long getSn() {
        return id.incrementAndGet();
    }

}

```

添加同步锁后懒汉单例，并发量下降，如果方法被频繁使用，频繁的加锁、释放锁，有很大的性能瓶颈。

## 双重检验懒汉单例

饿汉单例不支持延迟加载，懒汉单例有性能问题，不支持高并发。就需要一种既支持延迟加载又支持高并发的单例，也就是双重检验懒汉单例。对上面的懒汉单例进行优化之后，得出如下代码。

```
public class LazyDoubleCheckSnGenerator {

    private AtomicLong id = new AtomicLong(0);

    private static LazyDoubleCheckSnGenerator instance;

    // 构造函数设置为 private，类就无法被实例化
    private LazyDoubleCheckSnGenerator() {}

    // 双重检测
    public static LazyDoubleCheckSnGenerator getInstance() {
        if (instance == null) {
            // 类级别锁
            synchronized (LazyDoubleCheckSnGenerator.class) {
                if (instance == null) {
                    instance = new LazyDoubleCheckSnGenerator();
                }
            }
        }
        return instance;
    }

    public long getSn() {
        return id.incrementAndGet();
    }
}
```

双重检测首先判断实例是否为空，如果为空就使用类级别锁锁住整个类，其他线程也只能等待实例新建后，才能执行 synchronized 代码块的代码，而此时 instance 不为空，就不会继续新建实例。从而确保线程安全。getInstance 只会在最开始的时候，性能较差。创建实例之后，后面的线程都不会请求到 synchronized 代码块。后续并发性能也提高了。

> CPU 指令重排可能会导致新建对象并赋值给 instance 之后，还来得及初始化，就会其他线程使用。导致系统报错，为了解决这个问题，就需要给 instance 成员变量添加 volatile 关键字禁止指令重排。

## 静态内部类单例

和双重检测单例一样，静态内部类既支持延迟加载又支持高并发。首先看一下代码实现。

```
public class SnStaticClass {
    private AtomicLong id = new AtomicLong(0);

    private static LazySnGenerator instance;

    // 构造函数设置为 private，类就无法被实例化
    private SnStaticClass() {}

    private static class SingletonHolder{
        private static final SnStaticClass instance = new SnStaticClass();
    }

    // 静态内部类获取实例
    public synchronized static SnStaticClass getInstance() {
        return SingletonHolder.instance;
    }

    public long getSn() {
        return id.incrementAndGet();
    }
}
```

SingletonHolder 是一个静态内部类，当 SnStaticClass 加载时，并不会加载 SingletonHolder 静态内部类，也就不会执行静态内部的代码。在**类加载初始化阶段，不会执行静态内部类的代码**。只有 getInstance 方法，执行 SingletonHolder 静态内部类，才会创建 SnStaticClass 实例。而 instance 创建的安全性，都是由 JVM 保证的。虚拟机使用加锁同步机制，保证实例只会创建一次。这种方式不仅**实现延迟加载，也保证线程安全**。

## 枚举单例

枚举实例单例是一个简答实现方式，这种方式是通过 Java 枚举类性本身的特性，来保证实例的唯一和线程的安全。

```
public enum SnGeneratorEnum {

    instance;

    private AtomicLong id = new AtomicLong(0);

    public long getSn() {
        return id.incrementAndGet();
    }

}
```

# 单例模式的应用

在 Java 开发中，有很多地方使用到了单例模式。比如 JDK、Spring。

## JDK 

Runtime 类封装了 Java 运行信息，可以获取有关运行时环境的信息，每个 JVM 进程只有一个运行环境，只需要一个 Runtime 实例，所以 Runtime 一个单例实现。

```
public class Runtime {
    private static Runtime currentRuntime = new Runtime();

    /**
     * Returns the runtime object associated with the current Java application.
     * Most of the methods of class <code>Runtime</code> are instance
     * methods and must be invoked with respect to the current runtime object.
     *
     * @return  the <code>Runtime</code> object associated with the current
     *          Java application.
     */
    public static Runtime getRuntime() {
        return currentRuntime;
    }

    /** Don't let anyone else instantiate this class */
    private Runtime() {}
    
    ......省略
}
```

由以上代码可知，Runtime 是一个饿汉单例，类加载时就初始化了实例，提供 getRuntime 方法提交单例的调用。

## Spring

大部分 Java 项目都是基于 Spring 框架开发的，Spring 中 bean 简单分成单例和多例，其中 bean 的单例实现既不是饿汉单例也不是懒汉单例。是基于**单例注册表**实现，注册表就就是一个哈希表，使用一个哈希表存储 bean 的信息。

```
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
```

singletonObjects 表示一个单例注册表，key 存储 bean 的名称，value 存储 bean 的实例信息。DefaultSingletonBeanRegistry 类的 getSingleton 方法实现 bean 单例，以下摘取主要的的代码。

```
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(beanName, "Bean name must not be null");
        // 锁住注册表
		synchronized (this.singletonObjects) {
            // 获取 bean 信息，不存在就创建一个 bean
			Object singletonObject = this.singletonObjects.get(beanName);
			if (singletonObject == null) {
				
				beforeSingletonCreation(beanName);
				boolean newSingleton = false;
				boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
				
				try {
                    // 创建 bean
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				}
				catch (IllegalStateException ex) {
					
				}
				catch (BeanCreationException ex) {
					
				}
				finally {
					
				}
                // 创建好的 bean 存进 map 中
				if (newSingleton) {
					addSingleton(beanName, singletonObject);
				}
			}
			return singletonObject;
		}
	}
```

Spring 获取 bean，锁住整个注册表，首先从 map 中获取 bean，如果 bean 不存在，就创建一个 bean，并存入 map 中。后续获取 bean，获取到的都是 map 的 bean。并不会创建新的 bean。

# 总结

单例模式一个最简单的一种设计模式，该设计模式是一种创建型设计模式。规定了一个类只能创建一实例。很多类只需要一个实例，这样的好处，减少内存的占用和 CPU 的开销，减少 GC 的次数。同时也减少对资源的重复使用。

* 以生成订单系统的订单号为例，分别介绍几种单例模式。
  * 饿汉单例：线程安全，但不支持延迟加载，不使用也会占用内存，比较浪费内存。但是类加载时创建实例，可以及时的发现内存不足问题。
  * 懒汉单例：支持延迟加载，但是线程不安全。多线程获取实例，可能会创建多个实例，就需要使用同步锁，锁住获取实例的方法，但是加了锁之后，性能就比较差。
  * 双重检测懒汉单例：针对上面不同同时满足延迟加载和线程安全问题，就设计出来双重检测的懒汉单例，主要将锁的代码块范围缩小，先获取实例，如果实例为空，才使用类级别锁，锁住代码，创建实例。当创建好实例后，后面请求都不会进同步锁的代码块，性能也不会降低。还需要考虑指令重排的问题，需要给成员变量添加 volatile 关键字禁止指令重排。
  * 静态内部类：也同时满足延迟加载和线程安全，延迟加载是在类加载时不会静态内部类的代码，只有调用时候才会执行静态内部类的代码。JVM 使用同步锁的机制保证获取实例是线程安全的。
  *  枚举单例：通过 Java 枚举类性本身的特性，来保证实例的唯一和线程的安全。
* 单例模式的应用
  * JDK: Runtime 封装了 Java 运行信息，可以获取有关运行时环境的信息，一个 JVM 只需要一个 Runtime 实例.Runtime 单例是饿汉单例，在类加载时就初始化实例。
  * Spring: Spring 的 bean 支持单例，使用单例注册表，一种哈希表存储 bean信息，key 是存储 bean 的名称，value 是存储 bean 的实例。获取 bean 首先锁住表，然后获取 bean，如果为空就创建 bean，并存入表中，后续都能从哈希表中获取 bean 实例了。
  
# 参考

* [单例模式（上）：为什么说支持懒加载的双重检测不比饿汉式更优？](https://time.geekbang.org/column/article/194035)

* [Spring学习之路——单例模式和多例模式](https://www.cnblogs.com/nickup/p/9800120.html)
 
