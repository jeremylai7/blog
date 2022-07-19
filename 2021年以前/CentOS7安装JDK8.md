# CentOS7安装JDK8

## 下载jdk
 从[官网](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)上下载jdk1.8，我下载的是jdk-8u181-linux-x64.tar.gz

## 安装jdk
为了方便对下载的源文件有一个统一的管理，在 /opt新加一个文件夹resource，用于存放下载的文件

## 解压源码包
```
[root@iZbp1fgnu2pf6dztkpeq5dZ resource]# tar -zxvf jdk-8u181-linux-x64.tar.gz
```
## 设置环境变量
```
[root@iZbp1fgnu2pf6dztkpeq5dZ resource]# vim /etc/profile
```
填入一下内容
```
#java环境变量
export JAVA_HOME=/usr/local/resource/jdk1.8.0_181
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```
保存退出，并运行 source /etc/profile让配置生效
```
[root@iZbp1fgnu2pf6dztkpeq5dZ resource]# source /etc/profile
```
完成上述步骤之后就已经完成java的安装，使用java -version 验证是否安装成功
如果出现java版本信息就表示安装已经完成
