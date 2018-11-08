
**概要：**
1. Nginx 简介
2. Nginx 架构说明
3. Nginx 基础配置与使用
## 一、Nginx 简介与安装

官网文档：
https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/

---
### 知识点：
1. Nginx 简介
2. Nginx 编译与安装
### 1、Nginx简介：
Nginx是一个高性能WEB服务器，除它之外Apache、Tomcat、Jetty、IIS，它们都是Web服务器，或者叫做WWW（World Wide Web）服务器，相应地也都具备Web服务器的基本功能。Nginx  相对基它WEB服务有什么优势呢？
1. Tomcat、Jetty 面向java语言，先天就是重量级的WEB服务器，其性能与Nginx没有可比性。
2. IIS只能在Windows操作系统上运行。Windows作为服务器在稳定性与其他一些性能上都不如类UNIX操作系统，因此，在需要高性能Web服务器的场合下IIS并不占优。
3. Apache的发展时期很长，而且是目前毫无争议的世界第一大Web服务器，其有许多优点，如稳定、开源、跨平台等，但它出现的时间太长了，在它兴起的年代，互联网的产业规模远远比不上今天，所以它被设计成了一个重量级的、不支持高并发的Web服务器。在Apache服务器上，如果有数以万计的并发HTTP请求同时访问，就会导致服务器上消耗大量内存，操作系统内核对成百上千的Apache进程做进程间切换也会消耗大量CPU资源，并导致HTTP请求的平均响应速度降低，这些都决定了Apache不可能成为高性能Web服务器，这也促使了Lighttpd和Nginx的出现。 下图可以看出07年到17 年强劲增长势头。

![图片](https://images-cdn.shimo.im/6A4GSirFCTcUX0WC/服务器排名.png!thumbnail)

### **2、编译与安装**
**安装环境准备：**
**（1）linux 内核2.6及以上版本:**
只有2.6之后才支持epool ，在此之前使用select或pool多路复用的IO模型，无法解决高并发压力的问题。通过命令uname -a 即可查看。
```
#查看 linux 内核
uname -a  
```
**（2）GCC编译器**
GCC（GNU Compiler Collection）可用来编译C语言程序。Nginx不会直接提供二进制可执行程序,只能下载源码进行编译。
**（3）PCRE库**
PCRE（Perl Compatible Regular Expressions，Perl兼容正则表达式）是由Philip Hazel开发的函数库，目前为很多软件所使用，该库支持正则表达式。
**（4）zlib库**
zlib库用于对HTTP包的内容做gzip格式的压缩，如果我们在nginx.conf里配置了gzip on，并指定对于某些类型（content-type）的HTTP响应使用gzip来进行压缩以减少网络传输量。
**（5）OpenSSL开发库**
如果我们的服务器不只是要支持HTTP，还需要在更安全的SSL协议上传输HTTP，那么就需要拥有OpenSSL了。另外，如果我们想使用MD5、SHA1等散列函数，那么也需要安装它。
上面几个库都是Nginx 基础功能所必需的，为简单起见我们可以通过yum 命令统一安装。
```
#yum 安装nginx 环境
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel pcre pcre-devel
```

**源码获取：**
nginx 下载页：http://nginx.org/en/download.html 。
```
# 下载nginx 最新稳定版本
wget http://nginx.org/download/nginx-1.12.2.tar.gz
#解压
tar -zxvf nginx-1.12.2.tar.gz
```
最简单的安装：
```
# 全部采用默认安装
./configure
make && make install 
```

执行完成之后 nginx 运行文件 就会被安装在 /usr/local/nginx 下。

开机启动
```
vi /etc/rc.local

增加一行 /usr/local/nginx/sbin/nginx
设置执行权限：
chmod +x rc.local

```
![image](https://note.youdao.com/yws/res/8975/385FE55BF806475F93CB32CC2D67F8A1)


基于参数构建
```
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-debug
```


**控制命令：**
```
#默认方式启动：
./sbin/nginx 
#指定配置文件启动 
./sbing/nginx -c /tmp/nginx.conf 
#指定nginx程序目录启动
./sbin/nginx -p /usr/local/nginx/

#快速停止
./sbin/nginx -s stop
#优雅停止
./sbin/nginx -s quit

# 热装载配置文件 
./sbin/nginx -s reload
# 重新打开日志文件
./sbin/nginx -s reopen
```

## 配置conf
```
cd /usr/local/nginx/conf
vim nginx.conf
在里面加入如下参数
include /usr/local/nginx/conf/conf.d/*.conf;

另外在 对应目录下创建 conf.d文件夹 里面放入自定义的配置文件
blackip.conf 

search8.internal.api.gloryholiday.com.conf
(文件名以域名为准便于后续区别;里面日志地址也是同理)
```


更换完配置 需执行 ./sbin/nginx -t 进行配置项检验
 然后无问题后执行./sbin/nginx -s reload 重新加载配置文件

## 配置日志处理
1. 在/etc/logrotate.d目录下创建一个nginx的配置文件"nginx"配置内容如下
```
#vim /etc/logrotate.d/nginx
/usr/local/nginx/logs/*.log {
daily
rotate 5
missingok
notifempty
sharedscripts
postrotate
    if [ -f /usr/local/nginx/logs/nginx.pid ]; then
        kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
    fi
endscript
}

```
2. 执行logrotate
```
/usr/sbin/logrotate -f /etc/logrotate.d/nginx
```
**在/usr/local/nginx/logs目录中会产生
error.log
error.log.1
说明logrotate配置成功。**

4. 让logrotate每天进行一次滚动,在crontab中添加一行定时脚本。
```
#crontab -e
59 23 * * *  /usr/sbin/logrotate -f /etc/logrotate.d/nginx

#crontab -l 查看定时任务
```
6. 配置文件说明
```
home/wwwlogs/*nginx.log 需要轮询日志路径
daily: 日志文件分割频度。可选值为 daily，monthly，weekly，yearly
rotate 7: 一次将存储7个归档日志。对于第8个归档，时间最久的归档将被删除。
missingok: 在日志轮循期间，任何错误将被忽略，例如“文件无法找到”之类的错误。
dateext 使用日期作为命名格式
compress: 在轮循任务完成后，已轮循的归档将使用gzip进行压缩。
nocompress: 如果你不希望对日志文件进行压缩，设置这个参数即可
delaycompress: 总是与compress选项一起用，delaycompress选项指示logrotate不要将最近的归档压缩，压缩将在下一次轮循周期进行。这在你或任何软件仍然需要读取最新归档时很有用。
notifempty: 如果日志文件为空，轮循不会进行。
sharedscripts 表示postrotate脚本在压缩了日志之后只执行一次
create 644 www root: 以指定的权限创建全新的日志文件，同时logrotate也会重命名原始日志文件。
postrotate/endscript: 最通常的作用是让应用重启，以便切换到新的日志文件, 在所有其它指令完成后，postrotate和endscript里面指定的命令将被执行。在这种情况下，rsyslogd 进程将立即再次读取其配置并继续运行。

下面是一个脚本
postrotate
    if [ -f /usr/local/nginx/logs/nginx.pid ]; then
        kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
    fi
endscript

脚本让nginx重新生成日志文件。

```



#### yum安装

```
sudo yum install -y nginx

sudo systemctl start nginx.service

开机启动
sudo systemctl enable nginx.service

以下是Nginx的默认路径： 
(1) Nginx配置路径：/etc/nginx/ 
(2) PID目录：/var/run/nginx.pid 
(3) 错误日志：/var/log/nginx/error.log 
(4) 访问日志：/var/log/nginx/access.log 
(5) 默认站点目录：/usr/share/nginx/html

```

![image](https://note.youdao.com/yws/res/8984/F800AE40F76B4AAFA280C1891CD6EC38)

#### Linux查看公网IP

可以运行以下命令来显示你的服务器的公共IP地址:
```
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```

**模块更新：**
## 二、Nginx 架构说明

---
**Nginx 架构图:**
![图片](https://images-cdn.shimo.im/mgBtXOyOrjkUmSrZ/nginx_架构图.png!thumbnail)

**架构说明：**
1）nginx启动时，会生成两种类型的进程，一个是主进程（Master），一个（windows版本的目前只有一个）和多个工作进程（Worker）。主进程并不处理网络请求，主要负责调度工作进程，也就是图示的三项：加载配置、启动工作进程及非停升级。所以，nginx启动以后，查看操作系统的进程列表，我们就能看到至少有两个nginx进程。
2）服务器实际处理网络请求及响应的是工作进程（worker），在类unix系统上，nginx可以配置多个worker，而每个worker进程都可以同时处理数以千计的网络请求。
3）模块化设计。nginx的worker，包括核心和功能性模块，核心模块负责维持一个运行循环（run-loop），执行网络请求处理的不同阶段的模块功能，如网络读写、存储读写、内容传输、外出过滤，以及将请求发往上游服务器等。而其代码的模块化设计，也使得我们可以根据需要对功能模块进行适当的选择和修改，编译成具有特定功能的服务器。
4）事件驱动、异步及非阻塞，可以说是nginx得以获得高并发、高性能的关键因素，同时也得益于对Linux、Solaris及类BSD等操作系统内核中事件通知及I/O性能增强功能的采用，如kqueue、epoll及event ports。


**Nginx 核心模块：**
![图片](https://images-cdn.shimo.im/pV2dAuFXkaodJB4i/image.png!thumbnail)

## 三、Nginx 配置与使用

---
### **知识点**
1. 配置文件语法格式
2. 配置第一个静态WEB服务
3. 配置案例
  1. 动静分离实现
  2. 防盗链
  3. 多域名站点
  4. 下载限速
  5. IP 黑名单
  6. 基于user-agent分流
4. 日志配置
### **1、配置文件的语法格式：**
先来看一个简单的nginx 配置
```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        location /nginx_status {
    	   stub_status on;
    	   access_log   off;
  	    }
    }
}
```
上述配置中的events、http、server、location、upstream等属于配置项块。而worker_processes 、worker_connections、include、listen  属于配置项块中的属性。   /nginx_status   属于配置块的特定参数参数。其中server块嵌套于http块，其可以直接继承访问Http块当中的参数。
| **配置块**   | 名称开头用大口号包裹其对应属性   | 
|:----|:----|
| **属性**   | 基于空格切分属性名与属性值，属性值可能有多个项 都以空格进行切分 如：  access_log  logs/host.access.log  main   | 
| **参数**   | 其配置在 块名称与大括号间，其值如果有多个也是通过空格进行拆   | 

注意 如果配置项值中包括语法符号，比如空格符，那么需要使用单引号或双引号括住配置项值，否则Nginx会报语法错误。例如：
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                     '$status $body_bytes_sent "$http_referer" '
                     '"$http_user_agent" "$http_x_forwarded_for"';
### 2、配置第一个静态WEB服务
**基础站点演示：**
- [ ] 创建站点目录 mkdir -p /usr/www/luban 
- [ ] 编写静态文件
- [ ] 配置 nginx.conf
    - [ ] 配置server
    - [ ] 配置location

基本配置介绍说明：
（1）监听端口
语法：listen address：
默认：listen 80;
配置块：server

（2）主机名称
语法：server_name name[……];
默认：server_name "";
配置块：server
server_name后可以跟多个主机名称，如server_name [www.testweb.com](http://www.testweb.com)、download.testweb.com;。 支持通配符与正则

（3）location
语法：location[=|～|～*|^～|@]/uri/{……}
配置块：server
1. / 基于uri目录匹配
2. =表示把URI作为字符串，以便与参数中的uri做完全匹配。
3. ～表示正则匹配URI时是字母大小写敏感的。
4. ～*表示正则匹配URI时忽略字母大小写问题。
5. ^～表示正则匹配URI时只需要其前半部分与uri参数匹配即可。

**动静分离演示：**
- [ ] 创建静态站点
- [ ] 配置 location /static
- [ ] 配置 ~* \.(gif|png|css|js)$ 

基于目录动静分离
```
   server {
        listen 80;
        server_name *.luban.com;
        root /usr/www/luban;
        location / {
                index luban.html;
        }
        location /static {
         alias /usr/www/static;
        }
 }
```
基于正则动静分离
```
location ~* \.(gif|jpg|png|css|js)$ {
      root /usr/www/static;
}
```

**防盗链配置演示：**
```
# 加入至指定location 即可实现
valid_referers none blocked *.luban.com;
 if ($invalid_referer) {
       return 403;
}
```

**下载限速：**
```
location /download {
    limit_rate 1m;
    limit_rate_after 30m;
}

```
创建IP黑名单
```
# 创建黑名单文件
echo 'deny 192.168.0.132;' >> balck.ip
#http 配置块中引入 黑名单文件
include       black.ip;
```


### 3、日志配置：
**日志格式：**
```
log_format  main  '$remote_addr - $remote_user [$time_local]   "$request" '
                     '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
access_log  logs/access.log  main;
#基于域名打印日志
access_log logs/$host.access.log main;
```

**error日志的设置**
语法：error_log /path/file level;
默认：error_log logs/error.log error;
level是日志的输出级别，取值范围是debug、info、notice、warn、error、crit、alert、emerg，
**针对指定的客户端输出debug级别的日志**
语法：debug_connection[IP|CIDR]
events {
debug_connection 192.168.0.147; 
debug_connection 10.224.57.0/200;
}
[nginx.conf](https://attachments-cdn.shimo.im/fWYRhqhjvfIxcISK/nginx.conf)


