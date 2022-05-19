# RabbbitMQ RabbitListener 使用IP动态队列 Attribute value must be constant

> 在`RabbitMQ`消息队列使用 `@RabbitListener` 接收消息，队列名称使用常量命名，但是如果使用动态队列名称，比如根据系统 `ip` 命名队列名称。

## 获取服务器 IP
```
/**
     * 获取服务器ip,解决linux获取ip为127.0.0.1bug
     * @return
     */
    public static String getServerIp() throws SocketException {
        String ip = "";
        for (Enumeration<NetworkInterface> en = NetworkInterface.getNetworkInterfaces(); en.hasMoreElements();) {
            NetworkInterface anInterface = en.nextElement();
            String name = anInterface.getName();
            if (!name.contains("docker") && !name.contains("lo")) {
                for (Enumeration<InetAddress> enumIpAddr = anInterface.getInetAddresses(); enumIpAddr.hasMoreElements();) {
                    //获得IP
                    InetAddress inetAddress = enumIpAddr.nextElement();
                    if (!inetAddress.isLoopbackAddress()) {
                        String ipaddress = inetAddress.getHostAddress().toString();
                        if (!ipaddress.contains("::") && !ipaddress.contains("0:0:") && !ipaddress.contains("fe80")) {
                            if(!"127.0.0.1".equals(ip)){
                                return ipaddress;
                            }
                        }
                    }
                }
            }
        }
        return ip;
    }
```

## 创建队列
```
private String IP_ADDRESS = getServerIp();

@RabbitListener(queues = IP_ADDRESS)
    public void listener(String message) {
        System.out.println(message);
    }
```
编译报错 `Attribute value must be constant`

## 解决方案
* 创建队列
```
private String IP_ADDRESS = getServerIp();
 @Bean
    public Queue queue()  {
        return new Queue(IP_ADDRESS);
    }

```
* 使用 `#{}` 引用 `bean` 名称
```
@RabbitListener(queues= "#{queue.name}")
    public void listener(String message) {
        System.out.println(message);
    }
```
>  `#{queue.name}` 其中 `queue` 是 `bean` 名称，`name` 是固定。

