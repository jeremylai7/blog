# Java 设计模式实战系列—策略模式

# 从优惠打折活动说起

电商平台为了增加销量经常搞一些活动，比如 618、双十一，还有一些节假日活动，根据销量的变化又经常更新不同的活动。最开始为了增加销量，全场都六折：

```
// 打六折
public BigDecimal sixDiscount(BigDecimal amount) {
    BigDecimal discount = BigDecimal.valueOf(0.6);
    return amount.multiply(discount);
}
```

促销了几天之后，发现销量上去了，但是利润又减少了，又改成打八折了：

```
// 打八折
public BigDecimal eightDiscount(BigDecimal amount) {
    BigDecimal discount = BigDecimal.valueOf(0.8);
    return amount.multiply(discount);
}
```

再过几个月，活动每周都可能会改变，类似这周六折，下周八折，每次修改活动，就要改代码，就比较繁琐。后面就根据参数来选择不同的活动：

```
// 根据不同的type，选择不同的打折方案
public BigDecimal simpleDiscount(int type,BigDecimal amount) {
    if (type == 1) {
        amount = amount.multiply(BigDecimal.valueOf(0.6));
    } else if (type == 2) {
        amount = amount.multiply(BigDecimal.valueOf(0.8));
    }
    return amount;
}
```

后面不增加活动方案的前提下，改变方案只需要前端选择对应的方案即可。但**如果每次新增代码，还是需要更新代码，还需要更多繁琐的 if-else 判断**:

```
// 根据不同的type，选择不同的打折方案
public BigDecimal discount(int type,BigDecimal amount) {
    if (type == 1) {
        amount = amount.multiply(BigDecimal.valueOf(0.6));
    } else if (type == 2) {
        amount = amount.multiply(BigDecimal.valueOf(0.8));
    } else if (type == 3) {
        // 满100减20，满200减50
    } else if (type == 4) {
        // 满500减70
    } else if (type == 5) {
        // 满1000减两百
    }
    // 更多的方案 .....
    return amount;
}
```

上面的方法只是一些简单的打折方案，实际上的打折方案更复杂，一个打折方案代码至少要十多行，而上面十多个方案，代码显得非常的臃肿。有如下几个缺点：

* 每次增加方案，只能在方面里面添加，导致方案的代码越来越多，一个方法有几百行代码。耦合度非常高。
* 臃肿的代码每次都要从头看到尾，可读性比较差。
* 使用繁琐的 if-else 或者 switch 分支判断，可读性比较差。

解决上面的几个问题一个比较好的方案就是使用**策略模式**，策略模式可以避免繁琐的 if-else 分支判断，同时降低代码的耦合度，增强代码的可读性。


# 策略模式的定义和实现

定义一系列的算法,把每个算法封装起来, 并且使它们可相互替换。根据调用者传参来调用具体的算法。将策略的定义、创建和使用三个部分解耦。下面就上面的打折方案的方法使用策略模式进行改造。

## 策略的定义

策略的定义包含一个策略接口和一组实现这个接口的策略类，所有的策略类都实现相同的接口。


![](https://files.mdnice.com/user/29864/c8f30c78-1bdd-4b12-8da1-5dac374cb2a2.png)


接口和实现类如下:

```
// 创建一个接口
public interface Strategy {
    BigDecimal discount(BigDecimal amount);
}

// 打六折
public class SixDiscountStrategy implements Strategy{
    @Override
    public BigDecimal discount(BigDecimal amount) {
        return amount = amount.multiply(BigDecimal.valueOf(0.6));
    }
}

// 打八折
public class EightDiscountStrategy  implements Strategy{
    @Override
    public BigDecimal discount(BigDecimal amount) {
        return amount = amount.multiply(BigDecimal.valueOf(0.8));
    }
}

// 满100减20，满200减50
public class FirstDiscountStrategy implements Strategy{
    @Override
    public BigDecimal discount(BigDecimal amount) {
        // 满100减20，满200减50
        // 省略具体算法....
        return amount;
    }
}
```

将上面一个几百行的方法，拆分成一个一个小的类。类的数量变多了，但是代码也更加简洁了。

## 策略创建和使用

策略模式包含一组策略，一般通过类型 type 来选择创建哪个策略类。将创建策略类的代码封装成一个工具类，通过 type 直接调用：

```
// 策略工具类
public class Context {
  public static Strategy operation(int type) {
      if (type == 1) {
          return new SixDiscountStrategy();
      } else if (type == 2) {
          return new EightDiscountStrategy();
      } else if (type == 3) {
          return new FirstDiscountStrategy();
      }
      throw new IllegalArgumentException("not fond strategy");
  }
}
```

以上根据不同的 type 调用不同的策略类，在执行对应的方法。调用策略方法就简单多了，为了方便测试，直接使用 mian 方法调用：

```
public static void main(String[] args) {
    int type = 1;
    Strategy strategy = Context.operation(type);
    BigDecimal sixDiscount = strategy.discount(BigDecimal.valueOf(100));
    System.out.println("六折优惠价格:" + sixDiscount);
}
```

可能细心的同学发现，还是有很多 if-else 的代码，每次新增一个活动，还需要在 Context 多写一个判断。如果策略类是**无状态的，可以被共享**，那就不需要每次调用都创建一个新的策略对象，实现将创建好的对象缓存到 map 集合中，调用的时候直接，同时也不需要写 if-else 判断条件，优化代码如下：

```
private static Map<Integer,Strategy> map = new HashMap<>();

static {
    map.put(1,new SixDiscountStrategy());
    map.put(2,new EightDiscountStrategy());
    map.put(3,new FirstDiscountStrategy());
}

public static Strategy operation(int type) {
    Strategy strategy = map.get(type);
    if (strategy == null) {
        throw new IllegalArgumentException("not fond strategy");
    }
    return strategy;
}

public static void main(String[] args) {
    // 六折优惠
    int type = 1;
    Strategy strategy = Context.operation(type);
    BigDecimal sixDiscount = strategy.discount(BigDecimal.valueOf(100));
    System.out.println("六折优惠价格:" + sixDiscount);
}
```

重构之后的代码就没有 if-else 分支语句了，主要是利用了策略模式和 Map 集合，通过 type 直接获取 Map 集合上对应的策略类，从而很好的规避了 if-else 判断。通过查表法代替 if-else 判断。

将代码重构之后，代码也不会显得臃肿和复杂，有如下几个好处：

* 每个策略都有自己的对应的类，查看策略只需要查看自己对应的类即可，代码的可读性也大大增加。
* 代码拆分到不同的策略类上，更加的简洁。
* 后续新增策略类，只需要添加对应的策略接口实现以及 Map 集合，代码整体改动量比较小。

# 总结

本文先从电商项目的一个复杂多变的优惠券活动上讲起，优惠券的活动复杂多变，经常要搞不同的活动，可能每天也是不同的活动。针对复杂多变的需求，代码数量比较庞大和复杂，代码可读性差，耦合度高。所以就需要使用一个设计模式简化代码，减少代码改动量，增加代码的可读性。策略模式就应允而生。

* 策略模式包含一个策略接口和一组实现这个接口的策略类，所有的策略类都实现这个接口。策略类可以替换。
* 策略模式用来解耦策略的定义、创建以及使用，完整的策略模式有这三个部分组成:
  * 策略类的定义包含一个接口和一组实现类，后续增加策略只需要添加对应的实现类即可。
  * 策略的创建由工具类，或者工厂方法来完成。
  * 策略的使用通过传入参数来执行使用哪个策略，编译就确定好的状态只需要将不同的策略存储在 Map 集合中查表调用。如果需要运行时动态确定就需要返回一个实例对象，**运行时动态**是比较典型的应用场景。
* 使用策略模式之后，后面代码需要添加新的分支，改动量比较小，代码也比较清晰。
* 策略模式最主要的作用是**解耦策略的定义、创建和使用，控制代码的复杂度，让代码量不多过多，代码逻辑不会过于复杂**。对于复杂的代码来讲，策略模式还能再添加新的策略时，能最小化改动代码。
* 如果代码量比较少，逻辑不太复杂的代码，就不太需要引入策略模式，不然增加系统的复杂性。




