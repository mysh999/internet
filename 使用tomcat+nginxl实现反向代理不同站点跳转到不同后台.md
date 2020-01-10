[TOC]





## 一、概念

1、目的

Tomcat+Nginx实现反向代理、负载均衡

**新增：在nginx上访问tomcat1站点，跳转到tomcat1;访问tomcat2站点，跳转到tomcat2**



2、什么叫反向代理

指服务器根据客户端的请求，从其关系的一组或多组后端服务器（如Web服务器）上获取资源，然后再将这些资源返回给客户端，客户端只会得知反向代理的IP地址，而不知道在代理服务器后面的服务器簇的存在。



正向代理和反向区别：

正向代理的典型用途是为在防火墙内的局域网客户端提供访问Internet的途径。正向代理还可以使用缓冲特性减少网络使用率；

反向代理的典型用途是将防火墙后面的服务器提供给Internet用户访问。反向代理还可以为后端的多台服务器提供负载平衡，或为后端较慢的服务器提供缓冲服务。



## 二、环境准备



| 主机名  | IP              | 角色               | 操作系统 |
| ------- | --------------- | ------------------ | -------- |
| nginx1  | 192.168.159.101 | 反向代理、负载均衡 | Centos7  |
| tomcat1 | 192.168.159.102 | tomcat集群服       | Centos7  |
| tomcat2 | 192.168.159.103 | tomcat集群服       | Centos7  |

说明：以下在哪台服上操作，就以主机名说明



## 三、nginx配置

3.1 安装关联包

```bash
#yum -y install gcc-c++  pcre pcre-devel  zlib zlib-devel  openssl openssl--devel 
```



3.2  卸载原生nginx

```bash
# yum remove nginx  
# yum remove nginx-filesystem-1.12.2-2.el7.noarch
```



3.3 安装nginx

```bash
[root@nginx1 src]# tar -zxvf nginx-1.14.0.tar.gz 
[root@nginx1 src]# cd nginx-1.14.0/
[root@nginx1 nginx-1.14.0]# ./configure --with-http_stub_status_module
[root@nginx1 nginx-1.14.0]# make
[root@nginx1 nginx-1.14.0]# make install
[root@nginx1 nginx-1.14.0]# whereis nginx
nginx: /usr/lib64/nginx /usr/local/nginx /usr/share/nginx


/usr/local/nginx/sbin/nginx             //启动
/usr/local/nginx/sbin/nginx -s stop     //关闭
/usr/local/nginx/sbin/nginx -s reload   //重启
/usr/local/nginx/sbin/nginx -t   //测试配置文件是否有语法错误

```



3.4 添加防火墙规则

```bash
[root@nginx1 nginx-1.14.0]# firewall-cmd --zone=public --add-port=80/tcp --permanent
[root@nginx1 nginx-1.14.0]# firewall-cmd --reload
```



## 四、部署tomcat

说明：2台tomcat服配置一样



4.1 卸载自带的java

```bash
[root@tomcat1 ~]# yum -y remove java*

```



4.2 安装java

解压缩jdk包
```bash
tar -zxvf jdk-8u151-linux-x64.tar.gz 
```



重命名和拷贝
```bash
# mv –f jdk1.8.0_151 jdk

# mv –f jdk /usr/local/
```



配置root用户环境变量
```bash
# vim /etc/profile 

export JAVA_HOME=/usr/local/jdk
export JRE_HOME=/usr/local/jdk/jre
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```



立即生效
```bash
source  /etc/profile
ln -s /usr/local/jdk/bin/java /usr/bin/java
```





验证java
```bash
# which java

/usr/local/jdk/bin/java

# java -version
java version "1.8.0_151"
Java(TM) SE Runtime Environment (build 1.8.0_151-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.151-b12, mixed mode)
```



4.3、安装tomcat

```bash
[root@tomcat1 src]# tar -zxvf apache-tomcat-8.0.47.tar.gz -C /usr/local/
[root@tomcat1 src]# cd /usr/local/
[root@tomcat1 local]# ln -sv apache-tomcat-8.0.47 tomcat
‘tomcat’ -> ‘apache-tomcat-8.0.47’

[root@tomcat1 local]# vim /etc/profile.d/tomcat.sh  #配置tomcat环境变量
export CATALINA_HOME=/usr/local/tomcat
export PATH=$CATALINA_HOME/bin:$PATH

[root@tomcat1 local]# source  /etc/profile.d/tomcat.sh 

[root@tomcat1 local]# catalina.sh start   #启动服务s
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk/jre
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Tomcat started.
```



添加防火墙规则

```bash
[root@tomcat1 local]# firewall-cmd --zone=public --add-port=8080/tcp --permanent
success
[root@tomcat1 local]# firewall-cmd --reload
success       
```





验证

```bash
[root@tomcat1 local]# netstat -tnlp|grep -i 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      2311/java   
[root@tomcat1 local]# curl http://localhost:8080
```



设置默认虚拟主机，并增加jvmRoute

```bash
# vim /usr/local/tomcat/conf/server.xml 
 <Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat1">  #jvmRoute是jvm标识，就是页面最顶部的标签，在实际生产环境中，所有的后台tomcat标识都要一样，这里为了实验的说明性，两台tomcat的标识改成不一样的，分别为tomcat1和tomcat2
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <Context docBase="/Data/webapps1" path="" reloadable="true" /> #修改默认虚拟主机，并将网站文件路径指向/Data/webapps1
```



tomcat1服上创建站点页面  #   tomcat1/web目录结构和nginx上保持一致

```bash


[root@tomcat1 local]## cat /usr/local/apache-tomcat-8.0.47/webapps/tomcat1/web/index.html 
tomcat1
```



tomcat2服上创建站点页面#   tomcat2/web目录结构和nginx上保持一致

```bash
[root@tomcat2 local]## cat /usr/local/apache-tomcat-8.0.47/webapps/tomcat2/web/index.html 
tomcat2
```





测试配置，并启动

```bash
[root@tomcat1-server-1 local]# catalina.sh stop
[root@tomcat1-server-1 local]# catalina.sh configtest
[root@tomcat1-server-1 local]# catalina.sh start
```

使用类似方法配置tomcat2服



## 五、配置nginx实现不同站点不同指向

```bash
[root@nginx1 ~]# cat /usr/local/nginx/conf/nginx.conf
worker_processes  auto;#根据CPU核数自动分配进程

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    gzip  on;
    gzip_min_length 1k;
    gzip_buffers 4 8k;
    gzip_http_version 1.1;
    # 注意欲压缩文件content-type类型: application/xml application/json 
    gzip_types text/plain application/x-javascript text/css application/xml;
        upstream tomcat1-web { server 192.168.159.102:8080; }
        upstream tomcat2-web { server 192.168.159.103:8080; }
    server {
        listen       80;
        server_name  localhost;
        location /tomcat1/web {
            root   html;
            index  index.html index.htm index.jsp;
                        # 健康检测,默认开启
                        proxy_next_upstream error timeout http_503 http_500 http_502 http_504;
                        proxy_pass  http://tomcat1-web;
                        proxy_redirect  default;
                        proxy_set_header Host  $http_host;
                        proxy_set_header Cookie $http_cookie;
                        proxy_set_header X-Real-IP $remote_addr;
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                        proxy_set_header X-Forwarded-Proto $scheme;
                        client_max_body_size  100m;
                        proxy_connect_timeout 10;

        }
                location /tomcat2/web {
            root   html;
            index  index.html index.htm index.jsp;
                        proxy_pass  http://tomcat2-web;
                        proxy_redirect  default;
                        proxy_set_header Host  $http_host;
                        proxy_set_header Cookie $http_cookie;
                        proxy_set_header X-Real-IP $remote_addr;
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                        proxy_set_header X-Forwarded-Proto $scheme;
                        client_max_body_size  100m;
                        proxy_connect_timeout 10;

        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
  include conf.d/*.conf;  #加载conf.d目录下的所有conf文件
}
```



重新载入

```bash
[root@nginx1 nginx-1.14.0]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful    

如果报
nginx: [emerg] unknown directive "stub_status" in /usr/local/nginx/conf/nginx.conf:43

原因是Nginx没有添加modules/ngx_http_stub_status_module.o模块
编译时加上
./configure  --with-http_stub_status_module

[root@nginx1 nginx-1.14.0]# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf 

[root@nginx1 nginx-1.14.0]# /usr/local/nginx/sbin/nginx -s reload  

```



验证测试

http://192.168.159.101/tomcat1/web/

或者

http://192.168.159.101/tomcat2/web/

或者用域名(域名添加到hosts文件中)



效果：

从结果能看出，nginx根据不同的站点请求，解析到不同的后台
