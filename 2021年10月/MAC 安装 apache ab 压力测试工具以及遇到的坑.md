# MAC 安装 apache ab 压力测试工具以及遇到的坑

> ab 是apache对 http服务器进行压力测试的工具，它可以测试出服务器每秒可以处理多少请求。本文记录**mac版本**安装 ab 的步骤以及遇到的坑。

## 下载
进入 [apache ab官网](https://httpd.apache.org/download.cgi) 下载页面。

![image](https://user-images.githubusercontent.com/11553237/169219878-21c47da0-13fb-40fc-9c6c-f3913fc8c04b.png)

## 安装
### brew 安装
* 因为笔者的操作系统是mac系统，所以需要先安装brew。进入[brew网站](https://brew.sh/index_zh-cn)。执行下方命令
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
执行命令后报错:
Failed to connect to raw.githubusercontent.com port 443: Connection refused
解决方案:
打开 [https://www.ipaddress.com/](https://www.ipaddress.com/) 查询 raw.githubusercontent.com 对应的 ip 地址。

![image](https://user-images.githubusercontent.com/11553237/169219938-77c0f7c3-3815-4c5e-90ce-372baf6ff927.png)


添加ip到 /etc/hosts,添加以下配置:
```
185.199.108.133 raw.githubusercontent.com
185.199.109.133 raw.githubusercontent.com
185.199.110.133 raw.githubusercontent.com
185.199.111.133 raw.githubusercontent.com
```
再执行
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
执行成功后，使用brew安装apr、apr-util和prce
```
brew install apr
brew install apr-util
brew inatll prce
```

### apache ab安装
解压下载后压缩包，进入 httpd-2.4.51 目录。
执行以下命令：
```
./configure
make
make install
```
执行 ./configure 命令时报错:
```
jeremy@jeremydeMacBook-Pro httpd-2.4.51 % ./configure
checking for chosen layout... Apache
checking for working mkdir -p... yes
checking for grep that handles long lines and -e... /usr/bin/grep
checking for egrep... /usr/bin/grep -E
checking build system type... x86_64-apple-darwin20.2.0
checking host system type... x86_64-apple-darwin20.2.0
checking target system type... x86_64-apple-darwin20.2.0
configure:
configure: Configuring Apache Portable Runtime library...
configure:
checking for APR... no
configure: error: APR not found.  Please read the documentation.
```
**APR not found 没找到**
./configure 改成
```
 ./configure --with-apr=/usr/local/opt/apr --with-apr-util=/usr/local/opt/apr-util --with-pcre=/usr/local/Cellar/pcre/8.45
```
>其中 pcre 的路径可能不同，需要在 **/usr/local/Cellar/pcre** 里面确定路径。

上述命令执行成功后，如果没有报错，表明安装成功，执行ab
```
ab: wrong number of arguments
Usage: ab [options] [http[s]://]hostname[:port]/path
Options are:
    -n requests     Number of requests to perform
    -c concurrency  Number of multiple requests to make at a time
    -t timelimit    Seconds to max. to spend on benchmarking
                    This implies -n 50000
    -s timeout      Seconds to max. wait for each response
                    Default is 30 seconds
    -b windowsize   Size of TCP send/receive buffer, in bytes
    -B address      Address to bind to when making outgoing connections
    -p postfile     File containing data to POST. Remember also to set -T
    -u putfile      File containing data to PUT. Remember also to set -T
    -T content-type Content-type header to use for POST/PUT data, eg.
                    'application/x-www-form-urlencoded'
                    Default is 'text/plain'
    -v verbosity    How much troubleshooting info to print
    -w              Print out results in HTML tables
    -i              Use HEAD instead of GET
```
出现以上界面，说明 ab 已经安装成功。

## 运行 ab
### 主要参数
* -n 请求树
* -c 并发数（访问人数）
* -t 请求时间最大数
```
ab -n 1000 -c 100 http://www.baidu.com
```
>表示请求baidu.com 使用100请求数，请求1000次。

## 总结
* 需要在配置brew和检测configure上花了比较多的时间。
* 其余的按照步骤即可。
