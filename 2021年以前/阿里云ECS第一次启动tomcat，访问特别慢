打开$JAVA_PATH/jre/lib/security/java.security这个文件，找到下面的内容：securerandom.source=file:/dev/random 
JAVA_PATH 表示jdk安装路径
替换成securerandom.source=file:/dev/./random
##2020年9月21日更新
###背景
在一次重装阿里云ecs后，安装好jdk和tomcat后启动tomcat，一直启动失败，翻了很多资料也没解决，后面提了工单后解决了。
###解决办法
在tomcat的bin路径的catalina.sh添加下面代码
```
JAVA_OPTS="-Djava.security.egd=file:/dev/./urandom"
```
