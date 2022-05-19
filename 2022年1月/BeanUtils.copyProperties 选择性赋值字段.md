# BeanUtils.copyProperties 选择性赋值字段

>BeanUtils.copyProperties 在字段赋值上有强大的功能，如果有两个的类，如果需要将相同的字段赋值，就可以直接赋制。而不需要每个字段都需要一个一个赋制。

## BeanUtils.copyProperties 用法全赋制
先创建一个实体类
```
public class User {

	private String name;
	
	private Integer age;
        // 省略get/set方法
}
```
再赋制数据
```
User use = new User();
use.setName("jeremy");
use.setAge(23);
User newUser = new User();
BeanUtils.copyProperties(use,newUser);
System.out.println("赋制后的数据,姓名："+ newUser.getName() + " 年龄：" + newUser.getAge() );
```
控制台输出如下数据，说明数据赋制成功。
```
赋制后的数据,姓名：jeremy 年龄：23
```
## 选择性赋制字段
在工作中不要全部赋制数据，需要有选择性赋制字段。
比如有三个字段 user1 , user2 , user3。
* user1 的 name 有值
* user2 的 age 有值
* user3 都没值

要将user1 的 name 值和user2 的 age 值赋值给user3。

**BeanUtils.copyProperties 提供了忽略字段接口**，源码如下：
```
public static void copyProperties(Object source, Object target, String... ignoreProperties) throws BeansException {
        copyProperties(source, target, (Class)null, ignoreProperties);
    }
```
>其中 ignoreProperties，将字段排除在外。

以下是排除规则，如果字段为空，字段就不赋值。
```
	public static String[] getNullPropertyNames (Object source) {
		final BeanWrapper src = new BeanWrapperImpl(source);
		PropertyDescriptor[] pds = src.getPropertyDescriptors();
		Set<String> emptyNames = new HashSet<String>();
		for(PropertyDescriptor pd : pds) {
			Object srcValue = src.getPropertyValue(pd.getName());
			if (srcValue == null) {
				emptyNames.add(pd.getName());
			}
		}

		String[] result = new String[emptyNames.size()];
		return emptyNames.toArray(result);
	}
``` 
实现上面的 user1，user2 的字段全都复制给 user3。
```
public static void main(String[] args) {
    User user1 = new User();
    user1.setName("jeremy");
    User user2 = new User();
    user2.setAge(23);
    User user3 = new User();
    BeanUtils.copyProperties(user1,user3);
    System.out.println("user1 赋值给 user3 数据，姓名:" + user3.getName() + " 年龄：" + user3.getAge());
    BeanUtils.copyProperties(user2,user3,getNullPropertyNames(user2));
    System.out.println("user2 赋值给 user3 数据，姓名:" + user3.getName() + " 年龄：" + user3.getAge());
}
```
控制台输出如下数据，说明赋值成功。
```
user1 赋值给 user3 数据，姓名:jeremy 年龄：null
user2 赋值给 user3 数据，姓名:jeremy 年龄：23
```
