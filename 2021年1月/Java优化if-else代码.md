# Java优化if-else代码

## 前言
开发系统一些状态，比如订单状态：数据库存储是数字或字母，但是需要显示中文或英文,一般用到if-else代码判断，但这种判断可读性比较差，也会影响后期维护，也比较容易出现bug。比如：
假设状态对应关系：1:agree 2:refuse 3:finish
```
int status;
String statusStr = null;
if (status == 1) {
   status = "agree";
} else if (status == 2) {
   status = "refuse";
}else if(status == 3) {
    status = “finish”;
}
```
## 方案一: 数组
这种仅限通过数字获取到字母或者中文。
首先设置数组
```
String[] statusArray = {"","agree","refuse","finish"};
```
通过数组的位置获取数组的值
```
int status；
String statusStr =  statusArray[status];

```
优点： 占用内存少
缺点： 状态值只能是数字，而且还需要考虑数组越界情况
## 方案二:HashMap
创建和添加map:
```
    private static final Map<Integer,String> map = new HashMap<>();

    static {
        map.put(1,"agree");
        map.put(2,"refuse");
        map.put(3,"finish");
    }

```
这种有两种求解方式，通过 key 获取 value 以及通过 value 获取 key，
## 由 key 获取 value 
直接使用 get 方法即可。这里的key相对于数组解法，不限制 key 的类型。
 ```
int status;
map.get(status);
```
## 由 value 获取 key
使用map遍历:
 ```
int status;
for(Map.Entry<Integer, String> vo : map.entrySet()){
      if (vo.getValue().equals(result)) {
            status = vo.getKey();
            break;
       }
 }
```
优点：状态值不限制数字
缺点：占用空间大     
### 解决方案三、枚举
先定义一个枚举类
```
public enum TestEum {

    agree(1,"agree"),
    refuse(2,"refuse");

    private int code;

    private String capation;

    TestEum(int code,String capation){
        this.code = code;
        this.capation = capation;
    }

    public int getCode() {
        return code;
    }

   public String getCapation() {
        return capation;
   }
  
   String of(int code){
        for (TestEum testEum : TestEum.values()) {
            if (testEum.getCode() == code) {
                return testEum.getCapation();
            }
        }
        return null;
    } 
}

```
有了枚举以后，if-else 代码块可以优化成一行代码
```
String statusStr = TestEum.of(status);
``` 
## 总结
1. 如果通过数字获取描述，使用数组即可。
2. 如果通过描述获取数字，使用枚举和HashMap都可以。



