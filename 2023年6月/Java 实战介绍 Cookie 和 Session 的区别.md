# Java 实战介绍 Cookie 和 Session 的区别

HTTP 是一种不保存状态的协议，即无状态协议，HTTP 协议不会保存请求和响应之间的通信状态，协议对于发送过的请求和响应都不会做持久化处理。

![](https://files.mdnice.com/user/29864/ceffeffc-2dc1-4adf-8cfd-b5b644e597f1.png)

无状态协议减少了对服务压力，如果一个服务器需要处理`百万级用户`的请求状态，对服务器的压力无疑的是巨大的。

无状态的 HTTP 由于其简单和易用性，应用比较管广泛。而且早期的 Web 服务对于状态的需求也很低，所以应用场景也比较广泛。

随着 Web 的不断发展，越来越多的服务需要记录用户的登录状态，比如购物、聊天、论坛服务，请求都是无状态的服务，服务器就无法识别是 HTTP 请求的用户信息，所以就需要一种技术保存用户的状态，也就 Cookie 技术，有了 Cookie 的 HTTP 协议通信，就能保存状态了。

# Cookie 

在无状态协议不受影响的基础上，通过引入 Cookie 来记录状态，这样既不会影响原有的功能，也可以解决请求状态问题。Cookie 是当你浏览网页，通过服务器记录你的用户名，密码等网页信息。



**Cookie 是由服务端创建**，客户端向服务发送请求后，服务端通过响应报文的Set-Cookie 字段将 Cookie 信息返回给客户端，客户端自动保存 Cookie。Cookie 会标记来源、有效期、路径等信息。客户端再次请求该服务端时，会**自动**将 Cookie 添加到请求报文中(Request Header),服务端就能通过传递的 Cookie 识别客户端的信息。

**1.没有 Cookie 信息的请求**：


![](https://files.mdnice.com/user/29864/6fd6e75c-f611-48ce-92dc-0f034b094f94.png)


**2.再次（有了Cookie信息后）发送请求**：


![](https://files.mdnice.com/user/29864/80ab3637-efce-4dcd-8de0-5b6fd574fe47.png)


## Cookie 简单实践

使用 Spring Boot 创建简单的 Controller，当客户端传递的参数 a 有值时，服务端才添加 Cookie：

```
@GetMapping("/cookie")
@ResponseBody
public String cookie(String a, HttpServletRequest request, HttpServletResponse response) {
    if (a != null) {
        Cookie cookie = new Cookie("name",a);
        response.addCookie(cookie);
    }
    return "ok";
}
```

首先使用 Chrome 浏览器发送请求`http://localhost:8080/cookie`:

![](https://files.mdnice.com/user/29864/5d923e11-7e91-4e5c-9ca6-a6433317528a.png)

**返回结果没有 Cookie。**

再发送带有 a 参数的请求`http://localhost:8080/cookie?a=jeremy`:

![](https://files.mdnice.com/user/29864/669daeb3-4f7a-4a5b-b2f9-0c60bad1af26.png)


**返回 Cookie 都存在 Set-Cookie 字段中，客户端会自动保存 Cookie。**

再次发送相同的请求`http://localhost:8080/cookie?a=jeremy`:

![](https://files.mdnice.com/user/29864/0327fd8e-cae9-4df6-8173-e41c291df328.png)

**请求会将客户端的 Cooike 自动添加到请求报文中，**此时服务端也能接收到 Cookie信息：

![](https://files.mdnice.com/user/29864/5b0c3163-454b-428f-8e61-75798fa34d9d.png)



# Session 

Session 是服务端保存用户状态的一种机制，当用户访问网站时，服务端会为每个用户创建唯一个会话标识，并根据用户登录请求创建和存储会话信息，客户端再次请求时，就能从服务端获取会话信息了。

## Session 简单实践

在 Java 中的 Servlet 提供 HttpSession 的接口来操作会话信息，只要有以下几个方法：

* public HttpSession getSession() 获取会话信息，如果不存在就创建会话信息。
* public String getId() 获取唯一的会话 id。
* public void invalidate() 将会话信息失效，一般注销时候使用。

HttpSession 接口通过 getAttribute() 和 setAttribute() 来获取和设置会话信息，
下面创建两个方法，session() 方法获取会话判断用户是否登录，login() 方法添加会话信息。

```
@GetMapping("/session")
@ResponseBody
public String session(HttpServletRequest request, HttpServletResponse response) {
    HttpSession session = request.getSession();
    Boolean login =(Boolean) session.getAttribute("login");
    String loginInfo;
    if (login == null) {
        loginInfo = "未登录";
    } else {
        loginInfo = "已登录";
    }
    return "session id ：" + session.getId() + ":" + loginInfo;
}

@GetMapping("/login")
@ResponseBody
public String login(HttpServletRequest request) {
    HttpSession session = request.getSession();
    session.setAttribute("login",true);
    return "ok";
}
```

先请求 `http://localhost:8080/session`，返回如下信息：

```
session id ：F3C560208A54E3D5B465CDEBE7419817:未登录
```

多次发送请求，session id 是一致的，说明 session id是在会话周期之内（浏览器不关闭）都是不变的。

然后请求登录接口 `http://localhost:8080/login`，设置了会话信息之后，再请求 `http://localhost:8080/session`：返回如下信息：

```
session id ：F3C560208A54E3D5B465CDEBE7419817:已登录
```

> 同一个用户请求，服务端会创建唯一的会话，在请求的生命周期之内，会话 id 一直不改变。session 会话由服务端添加后，后续请求就能获取到会话信息，会话信息只存储在服务端。 

# Cookie 和 Session 的区别

Cookie 是存储在客户端上小型文本，是由服务端创建，然后通过响应报文的 Set-Cookie 字段返回给客户端。客户端每次请求服务端吗，浏览器都会将 Cookie 信息发送给服务端，服务端根据 Cookie 来识别用户的会话信息。Cookie 有如下几个特点：
    
* 存储在客户端
* 可以被客户端修改和删除
* 数据比较小，例如用户基本信息、购物信息
* 可以设置过期时间。

Session 是服务端存储会话信息，当客户端请求服务端时，服务端会被每个用户创建一个唯一的会话（Session id）标识，并在服务端设置和存储会话信息，并在后续的请求，可以获取到会话信息，主要有如下特点：
    
* 存储在服务端，客户端无法修改和删除
* 数据比较大，比如用户的信息，登录记录。
* 通常依赖 Cookie 和客户端进行数据交换。
    
Cookie 和 Session 的主要区别：

* 储存位置：Cookie 存储在客户端，Session 存储在服务端上。
* 数据大小：Cookie 通常比较小，Session 通常存储较大的数据。
* 安全性：Cookie 可以通过 js 设置和修改，也可能通过抓包工具修改，可以轻易的被修改，Cookie 安全性差，很多浏览器也禁用了 Cookie，Cookie 使用也不多。Session 的设置和修改都在服务端，安全性相对 Cookie 高很多。

在实际的使用场景上，Cookie 和 Session 也会结合使用，服务端使用Session记录用户的会话信息，而将会话信息存储在 Cookie 中，这样可以减少服务端的压力。
