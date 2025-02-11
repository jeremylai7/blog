# Java 自定义注解在登录验证的应用

## java注解

从 JDK 5开始，Java 增加了注解的新功能，注解其实是代码里面的特殊标记，这些标记可以在编译、类加载和运行时被读取，在不改变代码原有逻辑下，给源文件嵌入注解信息。再通过返回获取注解信息，根据不同的注解信息处理不同逻辑。其中 Java 有以下几个元Annotation：

### @Retention 

@Retention修饰 Annotation 可以保留多长时间，只包含一个 RetentionPolicy 一个成员变量。
* RetentionPolicy.CLASS  默认值，编译器把 Annotation 记录在 class 文件中。当运行 Java 程序时，JVM 不能获取 Annotation 信息。
* RetentionPolicy.RUNTIME 编译器把 Annotation 记录在 class 文件中，当运行 Java 程序时，JVM 可以获取 Annotation 信息，可以通过反射获取 Annotation 信息，**自定义注解使用此变量比较多**。
* RetentionPolicy.SOURCE Annotation 只保留在源代码（也就是 Java 文件），编译器直接抛弃 Annotation。

### @Target

@Target 修饰一个 Annotation 定义，它表示 Annotation 可以修饰在哪些地方：
* ElementType.TYPE 类、接口以及枚举
* ElementType.FIELD 成员变量 
* ElementType.METHOD 方法
* ElementType.PARAMETER 包定义
* ElementType.CONSTRUCTOR 构造器
* ElementType.ANNOTATION_TYPE Annotation
* ElementType.PARAMETER 参数

## 登录注解 @Logined 

### 注解需求
以电商系统举例，请求后端接口分成两类：需要**登录后才能访问**和**不需要登录访问**,所以就需要根据不同的需求做不同的处理，不需要登录的访问的接口不用做处理，而需要登录的接口需要在每次请求时验证请求，而在 Spring 可以使用拦截器作一个登录信息验证，而是否需要登录验证，这就需要用到注解了。

首先创建一个注解 @Logined，它要实现的功能：在需要登录才能访问的接口上添加该注解，可以添加在类和方法上，如果添加在类上，类下面所以的请求方法都需要进行登录验证。添加到方法上，只针对该方法需要验证。@Logined 注解定义如下:
```
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Logined {
    /**
     * 是否需要已经登录才允许访问
     *
     * @return
     */
    boolean isLogined() default true;

}
```
其中 @Target 设置 ElementType.METHOD 和 ElementType.TYPE 表示注解可以修饰在类和方法上，@Retention 设置 RetentionPolicy.RUNTIME 需要在运行时，JVM 可以获取到注解信息。isLogined 是注解的一个成员变量，这个在后面会讲到。
首先定义一个 Controller 控制器：
```
@RestController
@Logined
public class TestController {

	@GetMapping("/login")
	public String login() {
		return "need login";
	}
}
```
## 在拦截器上获取 @Logined 注解
每次发送一个 http 请求后，都会进入到拦截器中。
```
public class MyInterceptor extends HandlerInterceptorAdapter{

	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		if (!(handler instanceof HandlerMethod)) {
			return true;
		}
		HandlerMethod handlerMethod = (HandlerMethod) handler;
		Method method = handlerMethod.getMethod();
		boolean isLogin = this.isLogin(method);
		if (!isLogin){
			return true;
		}
    // 这里对登录信息验证，比如token验证，cookie验证

		return true;
	}

	private boolean isLogin(Method method) {
		//获取方法头部值
		Logined classLogined = method.getDeclaringClass().getAnnotation(Logined.class);
		Logined methodLogined = method.getAnnotation(Logined.class);
                // 如果方法上有注解返回 isLogined 
		if (classLogined != null && methodLogined == null) {
			System.out.println(classLogined.isLogined());
			return classLogined.isLogined();
		}
                // 方法没有注解，再找类上注解
		if ((classLogined != null && methodLogined != null) || (classLogined == null && methodLogined != null)) {
			return methodLogined.isLogined();
		}
		return false;
	}
}

```
### 拦截器流程：
* 获取请求类对应的方法
* 通过**反射**找到方法上的 @Logined 注解，和类上的 @Logined 注解
   * 如果类上有 @Logined 注解，方法上没有 @Logined 注解，返回类 @Logined 注解的 isLogined
   * 如果类和方法都有 @Logined 注解或者类没有 @Logined  方法有注解，返回方法的 isLogined

经过上述判断，如果返回是false，就不进行后续登录信息验证，否则需要登录信息验证。登录信息验证可以 token 验证、cookie验证。

## 总结
* 在需要请求的接口类或者方法上添加 @Logined，表明需要改请求接口需要登录后才能访问。如果不需要就不添加，如果类添加了，而某个方法不需要登录才能访问，添加 @Logined(isLogined = false) 即可。
* 在拦截器里面获取类或者方法的注解，如果有注解，则需要登录验证，如果没有，就直接通过。
