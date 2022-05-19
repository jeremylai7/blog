# Java反编译工具

## 编译和反编译
编程语言分成高级语言和低级语言。低级语言如机器语言、汇编语言。这类语言直接用计算机指令编写命令，不需要编译。这些语言机器能看到懂，但是程序员读起来很费劲。而我们平时经常用的语言C、Java、Python属于高级语言，这些语言程序员能看的懂。而机器是看不懂的。

简单的总结为：高级语言就是程序员认识的语言，而低级语言是机器认识的语言。而把高级语言转成低级语言这个过程就是编译，而反编译就是把低级语言转成高级语言。
有了反编译，我们就可以看懂Java编译器生成的字节码，比如Synchronized的实现原理（监听器monitor）、枚举、语法糖、泛型，这些都需要用到反编译工具。
## javap
javap是jdk自带的反编译命令，可以对代码进行反编译，但是反编译的并不是java文件。

使用格式
```
javap <options> <classes>
```
常用: javap -c 类名
```
-help  --help  -?        输出此用法消息
-version                 版本信息
-v  -verbose             输出附加信息
-l                       输出行号和本地变量表
-public                  仅显示公共类和成员
-protected               显示受保护的/公共类和成员
-package                 显示程序包/受保护的/公共类和成员 (默认)
-p  -private             显示所有类和成员
-c                       对代码进行反汇编
-s                       输出内部类型签名
-sysinfo                 显示正在处理的类的系统信息 (路径, 大小, 日期, MD5 散列)
-constants               显示最终常量
-classpath <path>        指定查找用户类文件的位置
-cp <path>               指定查找用户类文件的位置
-bootclasspath <path>    覆盖引导类文件的位置
```
下面写一段synchronized代码:
```
public class SynchronizedTest {

	private int count = 0;

	public  void addOne() {
		synchronized (SynchronizedTest.class) {
			count++;
		}
	}
}
```
执行编译和反编译命令
```
javac SynchronizedTest .java
javap -c SynchronizedTest.class
```
直接用记事本打开SynchronizedTest.class文件是一堆乱码文件，用sublime打开是一串数字
```
cafe babe 0000 0034 0017 0a00 0400 1209
0003 0013 0700 1407 0015 0100 0563 6f75
6e74 0100 0149 0100 063c 696e 6974 3e01
0003 2829 5601 0004 436f 6465 0100 0f4c
696e 654e 756d 6265 7254 6162 6c65 0100
0661 6464 4f6e 6501 000d 5374 6163 6b4d
6170 5461 626c 6507 0014 0700 1507 0016
```
反编译后的代码：
```
public class com.SynchronizedTest {
  public com.SynchronizedTest();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: iconst_0
       6: putfield      #2                  // Field count:I
       9: return

  public void addOne();
    Code:
       0: ldc           #3                  // class com/yyw/oil/web/admin/controller/purchase/SynchronizedTest
       2: dup
       3: astore_1
       4: monitorenter
       5: aload_0
       6: dup
       7: getfield      #2                  // Field count:I
      10: iconst_1
      11: iadd
      12: putfield      #2                  // Field count:I
      15: aload_1
      16: monitorexit
      17: goto          25
      20: astore_2
      21: aload_1
      22: monitorexit
      23: aload_2
      24: athrow
      25: return
    Exception table:
       from    to  target type
           5    17    20   any
          20    23    20   any
}

```
javap并没有将字节码反编译成成java文件，而是生成一种另一种能看的懂得字节码。可以看出被synchronized修饰的代码包含 monitorenter 和 monitorexit。synchronized 底层依赖着两个指令来实现同步， 这里看起来比较晦涩难懂。

## CFR

在[官网](https://www.benf.org/other/cfr/)上下载jar，执行如下命令:
```
java -jar cfr-0.151.jar SynchronizedTest.class
```
得到反编译java文件:
```
public class SynchronizedTest {
    private int count = 0;

    /*
     * WARNING - Removed try catching itself - possible behaviour change.
     */
    public void addOne() {
        Class<SynchronizedTest> clazz = SynchronizedTest.class;
        synchronized (SynchronizedTest.class) {
            ++this.count;
            // ** MonitorExit[var1_1] (shouldn't be in output)
            return;
        }
    }
}

```
CFR还带有一些参数:
|  参数 | 注释  |
|  ----  | ----  |
| --decodeenumswitch    (boolean)   | 去除switch对枚举支持的语法糖|
| --decodelambdas         (boolean) | 去除lambda表达式的语法糖 |
|--decodestringswitch    (boolean) |去除switch string支持的语法糖 |
其余参数可使用如下命令查看:
```
 java -jar cfr-0.151.jar --help 
```
## idea 
使用idea生成class文件，用idea打开class文件即可。idea是绝大多数Java程序员使用的编辑器，使用idea打开文件比较方便、快捷。

## 参考
[Java代码的编译与反编译那些事儿](https://www.hollischuang.com/archives/58)
[javap的使用](https://www.cnblogs.com/baby123/p/10756614.html)
