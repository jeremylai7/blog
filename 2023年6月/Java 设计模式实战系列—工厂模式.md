# Java 设计模式实战系列—工厂模式

>在 Java 开发中，对象的创建是一个常见的场景，如果对象的创建和使用都写在一起，代码的耦合度高，也不利于后期的维护。我们可以使用工厂模式来解决这个问题，**工厂模式是一个创建型模式**，将对象的创建和使用分离开来，降低代码的耦合度，提高程序的可维护性和扩展性。

# 工厂模式应用场景

* 调用方无需关注对象的创建过程，只需要直接调用即可。
* 具体子类的数目不确定，后续可能会新增或者减少子类的数量。对子类的使用不确定
* 调用方根据参数来调用对应的对象。

# 从多种支付种类说起

电商平台下单之后，支付需要可以选择不同的支付的方式，比如拼多多下单后，就会有以下的支付方式：

![](https://files.mdnice.com/user/29864/18165530-da47-440b-bf82-ccdadddb2a47.png)

工厂模式定义:**定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行**。

## 创建支付接口以及实现类

首先定义一个支付接口，以及几个实现类，比如微信支付，支付宝支付：

```
// 支付接口
public interface Pay {
    void pay();
}

// 支付宝支付
public class AliPay implements Pay{
    @Override
    public void pay() {
        System.out.println("支付宝支付 .....");
    }
}

// 微信支付
public class WechatPay implements Pay{

    @Override
    public void pay() {
        System.out.println("微信支付 .....");
    }
}
```

![](https://files.mdnice.com/user/29864/0dc91167-6f1d-471a-93ad-c3b176a7c9fe.png)



没使用工厂模式之前，一个下单方法里面包含创建订单、支付、更新订单方法，所以代码都冗余到了一起:

```
public void order(int payType) {
    // 创建订单
    // 省略代码......
    // 根据type 调用不同的支付
    Pay pay;
    if (payType == 1) {
        pay = new AliPay();
    } else {
        pay = new WechatPay();
    }
    // 调用支付
    pay.pay();
    // 更新状态 代码省略.....
}
```

以上代码耦合度很高，有以下几个缺点：

* 违反**单一职责原则**，对象的创建散落在多个地方，没有一个统一的地方处理对象创建，代码耦合度高。
* 代码可读性差，后续阅读代码难度增加。
* 难以扩展和修改，后续添加新的支付方法，或者更修支付方式，需要在多个地方修改代码，增加代码的维护成本。

## 引入工厂类

将 order 方法的支付代码抽取到工厂类中:

```
// 支付工厂类
public class PayFactory {

  public static Pay getPay(int payType) {
      Pay pay;
      if (payType == 1) {
          pay = new AliPay();
      } else {
          pay = new WechatPay();
      }
      return pay;
  }

}
```

经过重构之后的 order 方法，支付直接调用 PayFactory 工厂类，代码**逻辑更加清晰，可读性也更强了**:

```
public void order2(int payType) {
    // 创建订单
    // 省略代码......
    // 根据type 调用不同的支付
    Pay pay  = PayFactory.getPay(payType);
    // 调用支付
    pay.pay();
    // 更新状态 代码省略.....

}
```

![](https://files.mdnice.com/user/29864/140d8a4d-a7d6-45a1-8394-092bce69f699.png)


* 引入工厂模式之后，代码有如下的优点：

  * 实现对象的创建和使用解耦，调用方无需关注对象的创建，直接调用即可。
  * 增加的代码的可扩展性和灵活性，添加或者修改对象，只需要在工厂类修改即可，降低代码的维护成本。
  * 代码可读性大大增加。
  
**需要注意的是，上面示例类只有一个方法，仅仅是因为方便，还可以写其他方法，因为工厂模式重点是对象的创建，所以可以包含多个方法**。  
 
# 工厂模式在 JDK 中的应用

工厂模式作为场景的一种设计模式，在 JDK 使用也比较广泛，这里就介绍几种使用示例。

## Calendar.getInstance

Calendar 类提供了大量跟日期相关的功能代码，Calendar 是一个抽象类，通过调用 Calendar.getInstance() 来获取对象，最终会调用 createCalendar方法，来看一下源码：


![](https://files.mdnice.com/user/29864/67efbe33-6ac6-4c40-9b1d-0e8fbbf1b1d1.png)

从代码可以看出 getInstance 根据 TimeZone 和 Locale 的不同的，返回不同的 Calendar 子类对象，比如 BuddhistCalendar、JapaneseImperialCalendar。将对象的创建封装成一个工厂方法，调用只需要传递当时时区和地址，就能获取到对应的对象了，无需关心对象是如何创建的。

# 总结

* 在 Java 开发中如果将对象的创建和使用耦合在一起，不利于后期的维护，这就需要引入工厂模式。
* 工厂模式将对象的创建和使用分离，降低代码的耦合度。
* 定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。
* 引入工厂模式之后，调用方无需关注对象的创建过程，根据传参调用对应的实例对象，后续新增或更新对象只需要修改工厂方法即可，降低代码的维护成本。


# 参考

* [菜鸟教程-工厂模式](https://www.runoob.com/design-pattern/factory-pattern.html)

* [极客时间-工厂模式（上）](https://time.geekbang.org/column/article/197254)

* [开源实战一（上）：通过剖析Java JDK源码学习灵活应用设计模式](https://time.geekbang.org/column/article/229996)
