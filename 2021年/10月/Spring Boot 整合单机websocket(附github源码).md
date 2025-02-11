# Spring Boot 整合单机websocket(附github源码)

## websocket 概念
websocket 是一个通信协议，通过单个 TCP 连接提供全双工通信。websocket 连接成功后，服务端和客户可以进行双向通信。不同于 http 通信协议需要每次由客户端发起，服务响应到客户端。
websocket 相对轮询也能节约带宽，并且能实时的进行通信。

## 整合步骤
### 1.  添加 maven 依赖
```
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-websocket</artifactId>
	<version>2.1.3.RELEASE</version>
</dependency>
<dependency>
        <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-freemarker</artifactId>
	<version>2.1.3.RELEASE</version>
</dependency>
```
添加web、websocket和freemarker依赖。

### 2. 使用 ServerEndpointExporter 创建 bean
这个 bean 会自动注册声明 @ServerEndpoint 注解声明的 websocket endpoint，使用springboot自带tomcat启动需要该配置，使用独立 tomcat 则不需要该配置。
```
@Configuration
public class WebSocketConfig {
    //tomcat启动无需该配置
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
``` 
### 3. 创建服务端端点 (ServerEndpoint) 
```
@Component
@ServerEndpoint(value = "/message")
@Slf4j
public class WebSocket {

	private static Map<String, WebSocket> webSocketSet = new ConcurrentHashMap<>();

	private Session session;

	@OnOpen
	public void onOpen(Session session) throws SocketException {
		this.session = session;
		webSocketSet.put(this.session.getId(),this);
		log.info("【websocket】有新的连接,总数:{}",webSocketSet.size());
	}

	@OnClose
	public void onClose(){
		String id = this.session.getId();
		if (id != null){
			webSocketSet.remove(id);
			log.info("【websocket】连接断开:总数:{}",webSocketSet.size());
		}
	}

	@OnMessage
	public void onMessage(String message){
		if (!message.equals("ping")){
			log.info("【wesocket】收到客户端发送的消息,message={}",message);
			sendMessage(message);
		}
	}

	/**
	 * 发送消息
	 * @param message
	 * @return 全部都发送一遍
	 */
	public void sendMessage(String message){
		for (WebSocket webSocket : webSocketSet.values()) {
			webSocket.session.getAsyncRemote().sendText(message);
		}
		log.info("【wesocket】广播消息,message={}", message);

	}

}
```

### 4. 添加 controller 和 客户端
* 添加 controller
```
@GetMapping({"","index.html"})
public ModelAndView index() {
	ModelAndView view = new ModelAndView("index");
	return view;
}
```
* 添加ftl页面
```
<!DOCTYPE html>
<html>
<head>
    <title>websocket</title>
</head>
<body>
<div>
    <input type="text" name="message" id="message">
    <button id="sendBtn">发送</button>
</div>
<div style="width:100px;height: 500px;" id="content">
</div>
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.js"></script>
<script type="text/javascript">
    var ws = new WebSocket("ws://127.0.0.1:8080/message");
    ws.onopen = function(evt) {
        console.log("Connection open ...");
    };

    ws.onmessage = function(evt) {
        console.log( "Received Message: " + evt.data);
        var p = $("<p>"+evt.data+"</p>")
        $("#content").prepend(p);
        $("#message").val("");
    };

    ws.onclose = function(evt) {
        console.log("Connection closed.");
    };

    $("#sendBtn").click(function(){
        var aa = $("#message").val();
        ws.send(aa);
    })

</script>
</body>
</html>
```

### 5. 展示效果
![image](https://user-images.githubusercontent.com/11553237/169200584-8e08a51c-af46-4d8b-bd81-3f91b96eeb30.png)

## 附录
[github源码](https://github.com/jeremylai7/springboot-learning/tree/master/springboot-websocket)

