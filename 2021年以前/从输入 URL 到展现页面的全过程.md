# 从输入 URL 到展现页面的全过程

## 总体分为以下几个过程
1.  DNS解析
2. TCP连接
3. 发送HTTP请求
4. 服务器处理请求并返回HTTP报文
5. 浏览器解析渲染页面
6. 连接结束

### DNS解析
域名到ip地址转换

### TCP连接
HTTP连接是基于TCP连接

https 协议就是http +ssl协议，ssl协议采用非对称加密

### 发送HTTP请求
http请求报文是由三部分组成：请求行，请求报头和请求正文

请求行: 格式如下
Method Request-URL HTTP-Version CRLF
比如：
Method Request-URL HTTP-Version CRLF

请求报头
常见的请求报头有: Accept, Accept-Charset, Accept-Encoding, Accept-Language, Content-Type, Authorization, Cookie, User-Agent等。
请求正文:
使用post，put方法请求，就会把请求信息存储在正文中，请求数据格式一般为json。这时就需要Content-Type: application/json

### 服务器处理请求并返回HTTP报文
http响应报文也是由三部分组成：状态码、响应报头和响应报文
状态码
* 1xx：指示信息–表示请求已接收，继续处理。
* 2xx：成功–表示请求已被成功接收、理解、接受
* 3xx：重定向–要完成请求必须进行更进一步的操作。
* 4xx：客户端错误–请求有语法错误或请求无法实现。
* 5xx：服务器端错误–服务器未能实现合法的请求。
平时遇到比较常见的状态码有:200, 204, 301, 302, 304, 400, 401, 403, 404, 422, 500(分别表示什么请自行查找)。

响应报头
服务器返回给浏览器的文本信息，通常html、css、js、图片等文件
### 浏览器解析渲染页面

浏览器是一个边解析边渲染的过程。首先浏览器解析HTML文件构建DOM树，然后解析CSS文件构建渲染树，等到渲染树构建完成后，浏览器开始布局渲染树并将其绘制到屏幕上。

