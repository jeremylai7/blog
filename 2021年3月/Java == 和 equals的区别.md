# Java == 和 equals的区别

1.  == 是操作符，equals是方法。
2. 对于基本类型变量来说，只能使用 == ，因为基本类型的变量没有方法。使用==比较是值比较。
3. 对于引用类型的变量来说，==比较的两个引用对象的地址是否相等。所有类都是继承objcet类，而object类是equals方法比较的也是对象的地址是否相等，如果类没有重写equals方法，使用 == 和equals方法效果是一样的。
4. string类重写了equals方法，首先判断地址是否一致，如果是返回true，如果不是在比较两者值是否一致。代码如下
```
    public boolean equals(Object anObject) {
       //判断对象地址是否一致
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            //值比较
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```
5. Java 八种基本类型的封装类的equals方法，思路基本是一致的:
    * 判断是否是相同的类型，否直接返回false
    * 比较他们对应的值
举例Long类的equals方法
```
public boolean equals(Object obj) {
     if (obj instanceof Long) {
         return value == ((Long)obj).longValue();
     }
     return false;
 }
```
