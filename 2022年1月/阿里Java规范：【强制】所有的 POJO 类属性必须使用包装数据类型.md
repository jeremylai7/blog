# 阿里Java规范：【强制】所有的 POJO 类属性必须使用包装数据类型

在 Java 开发手册中有这一条：

![Uploading image.png…]()

我们知道基本类型和包装类型有很多不同点：
* 封装类型可以调用各种方法，而基本类型没有
* 封装类型声明字段之后可以不设置默认值，而基本类型需要初始化默认值。比如 int 默认值为0，boolean 默认值为 false。

## 为什么要有这种规定
一般 dao 层会有固定的增删改查方法，分别是：
* selectByPrimaryKey
* updateByPrimaryKey
* insertSelective
* updateByPrimaryKeySelective

### selectByPrimaryKey
在调用查询方法 `selectByPrimaryKey`,字段Integer a 本来是 null，但是因为设置成了 int 类型，Java 会自动初始化默认值，结果显示默认值 0。查询结果和数据库不匹配。本来要插入null，反而插入了0，导致数据添加错误。

### updateByPrimaryKeySelective
在调用更新的方法，这里是调用 `updateByPrimaryKeySelective` 选择性更新字段，有值才更新，没有值就不更新，更新字段 b，但是由于字段 a 是 int 类型，在创建对象时，a 设置成了默认值0，误将为 null 的 a 设置成了0。`insertSelective`  方法也是同理。

### 参数传输
在springmvc接收数据，或者使用 RPC 方法传输数据，参数本来是没有赋值，但是 SpringMvc 或者 RPC 都会初始化默认值，但是传输的数据和需要传输的数据值不一致。

## 总结
所有的 POJO 必须设置封装类型，这是因为数据为 null 的情况下，基本类型会有默认值，无论是在添加、修改和查询的时候，都会导致数据和实际要修改的数据不一致。这一点还是需要多多注意。

## 参考
* [Java开发手册（嵩山版）.pdf](https://github.com/alibaba/p3c/blob/master/Java%E5%BC%80%E5%8F%91%E6%89%8B%E5%86%8C%EF%BC%88%E5%B5%A9%E5%B1%B1%E7%89%88%EF%BC%89.pdf)

