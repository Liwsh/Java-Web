Nginx应用
nginx介绍
Nginx是一款高性能的http 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。官方测试nginx能够支支撑5万并发链接，并且cpu、内存等资源消耗却非常低，运行非常稳定。
nginx应用场景
1、http服务器。Nginx是http服务可以独立提供http服务。可以做网页静态服务器。
2、虚拟主机。可以实现在一台服务器虚拟出多个网站。例如个人网站使用的虚拟主机。
3、反向代理，负载均衡。当网站的访问量达到一定程度后，单台服务器不能满足用户的请求时，需要用多台服务器集群可以使用nginx做反向代理。并且多台服务器可以平均分担负载，不会因为某台服务器负载高宕机而某台服务器闲置的情况。
nginx安装
1、nginx安装编译环境
gcc
编译依赖gcc环境，Centos：yum install gcc，ubunt：apt-get install gcc
pcre
pcre(Perl Compatible Regular Expressions)是Perl的一个库，包括perl兼容的正则表达式库。nginx的http模块使用pcre解析正则表达式。Centos：yum install -y pcre pcre-devel
ubuntu：apt-get install libpcre3 libpcre3-dev
zlib
zlib提供了压缩和解压的多种方式，nginx使用zlib对http包的内容进行gzip，需要zlib库，Centos：yum install -y zlib zlib-devel，ubuntu：apt-get install zlib1g-dev。
openssl
OpenSSL是一个强大的安全套接字密码库，包括密码算法、常用密钥和证书封装管理功能以及SSL协议，Centos：yum install -y openssl openssl-devel，ubuntu：apt-get -insatll openssl libssl-dev。
2、上传nginx压缩包至服务器。
3、解压压缩包tar -zxvf nginx-***.tar.gz，进入nginx目录，cd nginx-***。
4、创建目录
mkdir -p /var/temp/nginx
5、设置参数如下：
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi
6、编译：make
7、安装：make install
nginx使用
1、启动nginx
cd /usr/local/nginx/sbin
./nginx
注意：执行./nginx启动nginx，这里可以-c指定加载的nginx配置文件，如下：
./nginx -c /usr/local/nginx/conf/nginx.conf
如果不指定-c，nginx在启动时默认加载conf/nginx.conf文件，此文件的地址也可以在编译安装nginx时指定./configure的参数（--conf-path= 指向配置文件（nginx.conf））
2、查询nginx进程
root@lwsh:~# ps -ef | grep nginx
root      12804      1  0 10:13 ?        00:00:00 nginx: master process ./nginx
nobody    12805  12804  0 10:13 ?        00:00:00 nginx: worker process
root      13461  13379  0 10:42 pts/3    00:00:00 grep --color=auto nginx
12804是nginx主进程的进程id，12805是nginx工作进程的进程id。
3、访问nginx，在浏览器输入ip地址，出现Welcome to nginx!页面则nginx安装成功。

4、停止nginx
i.快速停止
./nginx -s stop
相当于先查询出nginx的进程id，再使用kill命令强制杀掉进程。
ii.完整停止（建议使用）
./nginx -s quit
待nginx进程处理结束后退出停止。
5、重启nginx
重启命令，修改nginx.conf配置文件，重新加载配置文件。
./nginx -s reload
配置虚拟主机
虚拟主机
虚拟主机可以将一台主机分成多个虚拟主机，每个虚拟主机可以独立对外提供www服务，这样可实现一台主机对外提供多个web服务，每个虚拟主机相互独立，互不影响。nginx实现虚拟主机有三种方式:
1、ip虚拟主机。
2、端口虚拟主机。
3、域名虚拟主机。
nginx配置文件结构
......
events {
    .......
}
http{
   .......
   server{
	.......
	}
   server{
	.......
	}

}
每个server就是一个虚拟主机。
基于IP的虚拟主机
同一个服务器配置多个ip地址。
方法一
使用标准网络配置工具（ifconfig和route命令）添加ip别名：
当前ip
root@lwsh:~# ifconfig 
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.195.132  netmask 255.255.255.0  broadcast 192.168.195.255
        inet6 fe80::f02e:4140:51d0:ba04  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:ba:c7:3f  txqueuelen 1000  (Ethernet)
        RX packets 10877  bytes 3056167 (3.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7899  bytes 1070854 (1.0 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 19  base 0x2000  

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 453  bytes 33192 (33.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 453  bytes 33192 (33.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

在ens33网卡再绑定一个ip：192.168.195.22

/sbin/ifconfig ens33:1 192.168.195.22 broadcast 192.168.195.255 netmask 255.255.255.0 up
/sbin/route add -host 192.168.195.22 dev ens33:1

root@lwsh:~# ifconfig 
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.195.132  netmask 255.255.255.0  broadcast 192.168.195.255
        inet6 fe80::f02e:4140:51d0:ba04  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:ba:c7:3f  txqueuelen 1000  (Ethernet)
        RX packets 11426  bytes 3096722 (3.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8272  bytes 1119349 (1.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 19  base 0x2000  

ens33:1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.195.22  netmask 255.255.255.0  broadcast 192.168.195.255
        ether 00:0c:29:ba:c7:3f  txqueuelen 1000  (Ethernet)
        device interrupt 19  base 0x2000  

方法二
修改网卡配置文件
新增网卡ens33:1
新增ip 192.168.195.22

修改配置文件
复制静态网页cp -r html html128，cp -r html html22，并修改index.html。

#配置虚拟主机
server {
#虚拟主机监听的ip和端口
        listen       80;
        server_name  192.168.195.132;

        #charset koi8-r;
        #access_log  logs/host.access.log  main;
#所有的请求都以/开始，所有的请求都可以匹配此location
        location / {
#使用root指令指定虚拟主机目录即网页存放目录
	    #比如访问http://ip/test.html将找到/usr/local/html132/test.html
            root   html132;
        #指定欢迎页面，按从左到右顺序查找
            index  index.html index.htm;
        }    
}
server {
        listen       80;
        server_name  192.168.195.22;

        #charset koi8-r;
        #access_log  logs/host.access.log  main;

        location / {
            root   html22;
            index  index.html index.htm;
        }  
    }

访问192.168.195.22


访问192.168.195.128

基于端口的虚拟主机
修改配置文件
复制静态网页cp -r html html81，cp -r html html82，并修改index.html。

#配置虚拟主机
server {
        listen       81;
        server_name  192.168.195.132;

        #charset koi8-r;
        #access_log  logs/host.access.log  main;

        location / {
            root   html81;
            index  index.html index.htm;
        }    
}
server {
        listen       82;
        server_name  192.168.195.132;

        #charset koi8-r;
        #access_log  logs/host.access.log  main;

        location / {
            root   html82;
            index  index.html index.htm;
        }  
    }

访问http://192.168.195.132:81


访问http://192.168.195.132:82

基于域名的虚拟主机
一个域名只能绑定一个ip地址，一个ip地址能绑定多个域名。两个域名指向同一台nignx服务器，访问不同的域名显示不同的内容。
修改host文件使www.lwsh.com和www.lwsh.cn对应192.168.195.132虚拟机。可以通过SwitchHosts工具修改。

修改配置文件
复制静态网页cp -r html htmlcom，cp -r html htmlcn，并修改index.html。

#配置虚拟主机
server {
        listen       80;
        server_name  www.lwsh.com;

        #charset koi8-r;
        #access_log  logs/host.access.log  main;

        location / {
            root   htmlcom;
            index  index.html index.htm;
        }    
}
server {
        listen       80;
        server_name  www.lwsh.cn;

        #charset koi8-r;
        #access_log  logs/host.access.log  main;

        location / {
            root   htmlcn;
            index  index.html index.htm;
        }  
    }

访问www.lwsh.com

访问www.lwsh.cn



nginx反向代理
反向代理
通常的代理服务器，只用于代理内部网络对Internet的连接请求，客户机必须指定代理服务器，并将本来要直接发送到Web服务器的http请求发送到代理服务器，由代理服务器向Internet上的Web服务器发起请求，最终达到客户机，该过程即正向代理。
而反向代理（Reverse Proxy）是指以代理服务器来接收Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

nginx+tomcat反向代理
两个tomcat服务通过nginx反向代理。
tomcat1服务器：192.168.195.132:8080
tomcat2服务器：192.168.195.132:8081
如下图：

配置tomcat
修改tomcat1的conf目录下的server.xml文件端口参数。
......
<Server port="8005" shutdown="SHUTDOWN">
......
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
......
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

修改tomcat1的conf目录下的server.xml文件端口参数。
......
<Server port="8006" shutdown="SHUTDOWN">
......
<Connector port="8081" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
......
<Connector port="8010" protocol="AJP/1.3" redirectPort="8443" />

修改tomcat1、tomcat2的webapps/ROOT/index.jsp以作区分。启动tomcat：./bin/startup.sh
nginx反向代理配置
修改配置文件
#配置一个代理即tomcat1服务器
upstream tomcat_server1{
        server 192.168.195.132:8080;
}
#配置一个代理即tomcat2服务器
upstream tomcat_server2{
        server 192.168.195.132:8081;
    }
server {
        listen       80;
        server_name  www.lwsh.com;

        location / {
#域名www.lwsh.com的请求全部转发到tomcat_server1即tomcat1服务上
            proxy_pass http://tomcat_server1;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
server {
        listen       80;
        server_name  www.lwsh.cn;

        location / {
#域名www.lwsh.cn的请求全部转发到tomcat_server2即tomcat2服务上
            proxy_pass http://tomcat_server2;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

分别访问www.lwsh.com、www.lwsh.cn测试反向代理。
访问www.lwsh.com，tomcat1下的index.jsp如下：

访问www.lwsh.cn，tomcat2下的index.jsp如下：