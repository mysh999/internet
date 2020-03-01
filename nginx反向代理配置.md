[TOC]





## 一、目标说明

此文档模拟使用2台nginx做反向代理，访问后台另外2台nginx搭建的web服（或者java工程）



## 二、架构说明

![工程流程图.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1gcc21libz7j30nc0fomxn.jpg)

这里由于条件所限，只模拟2台nginx代理服务器和2台工程服务器（用nginx做web服务）



## 三、环境描述

| 服务器名 | IP              | 说明              |
| -------- | --------------- | ----------------- |
| nginx1   | 192.168.230.200 | 配置反省代理到web |
| nginx2   | 192.168.230.201 | 配置反省代理到web |
| web1     | 192.168.230.202 | 使用nginx实现web  |
| web2     | 192.168.230.203 | 使用nginx实现web  |

操作系统使用CentOS7.6



## 四、安装nginx

4台都安装，采用yum安装

4.1、卸载自带nginx

```bash
# yum -y remove nginx  
```



4.2、配置nginx的yum

```bash
# cat /etc/yum.repos.d/nginx.repo 
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```



4.3、安装nginx

```bash
# yum -y install nginx
```



4.4、查看版本和和编译信息

```bash
# nginx -v
nginx version: nginx/1.17.8

# nginx -V
nginx version: nginx/1.17.8
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```



## 五、nginx服配置

### 5.1、nginx1服主配置如下：

```bash
[root@nginx1 ~]# cat /etc/nginx/nginx.conf 
user root;    										# 定义Nginx运行的用户和用户组
worker_processes auto;    							# nginx进程数
pid             /run/nginx.pid;   				    # pid(进程标识符): 存放路径
error_log       /var/log/nginx/error.log error;     # 错误日志: 存放路径

include /usr/share/nginx/modules/*.conf;            # 加载动态模块 : 存放路径

worker_rlimit_nofile 102400;						# 指定进程可以打开的最大描述符：数目
events {											#events模块中包含nginx中所有处理连接的设置
    use epoll;										# 使用epoll模型
    worker_connections 102400;						# 每个进程允许的最多连接数
}
http {
    include mime.types;						# 文件扩展名与文件类型映射表
    default_type application/octet-stream;	# 默认文件类型
    sendfile on;						# 允许sendfile方式传输文件，默认为off
    tcp_nopush on;						# 等数据包累积到一定大小才发送
    keepalive_timeout 65;				# 指定服务端为每个 TCP 连接最多可以保持多长时间
    server_tokens     off;
    gzip on;							# 开启gzip
    gzip_min_length   1k;				# 启用gzip压缩的最小文件，小于设置值的文件将不会压缩
    gzip_buffers      4 16k;			# 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流
    gzip_http_version 1.0;
    gzip_comp_level   2;
    gzip_types        text/plain application/x-javascript text/css application/xml text/javascript application/javascript; # 进行压缩的文件类型
    gzip_vary         on;			# 是否在http header中添加Vary: Accept-Encoding，建议开启
    									# 自定义日志格式
    log_format  main  '$remote_addr - [waf: $http_User_waf] - [$upstream_addr $upstream_response_time] - $remote_user [$time_local] "$request" [ body: $request_body ] '
                      '$status $body_bytes_sent "$http_referer"  [ appid: $http_x_ubt_appid ] [ deviceid: $http_x_ubt_deviceid ] [ sign: $http_x_ubt_sign ] '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    charset utf-8;					# 默认编码
    server_names_hash_bucket_size 128;
    client_header_buffer_size 4k;
    large_client_header_buffers 8 64k;
    client_max_body_size 1g;
    client_body_buffer_size 1m;
    client_header_timeout 3m;
    client_body_timeout 3m;
    send_timeout 3m;

    proxy_connect_timeout 3000;
    proxy_read_timeout 3000;
    proxy_send_timeout 3000;
    proxy_buffer_size       16k;
    proxy_buffers           4 64k;
    proxy_busy_buffers_size 128k;
    proxy_temp_file_write_size 128k;

    include conf.d/*.conf;
}
```



### 5.2、nginx1 反代配置文件

```bash
[root@nginx1 ~]# cat /etc/nginx/conf.d/web.conf 
 upstream web1.com {						# 和location中的上下文proxy_pass 匹配
         server 192.168.230.202 weight=1;	# web1权重为1低
         server 192.168.230.203 weight=100; # web2权重为100，优先访问
   }
 upstream web2.com {						# 和location中的上下文proxy_pass 匹配
         server 192.168.230.202 weight=100;	# web1权重为100，优先访问
         server 192.168.230.203 weight=1;	# web2权重为1低
   }
    
            server {
                listen 80;
        server_name web.com minikorean.ubtrobot.com;
        access_log  /var/log/nginx/web-access.log main;		# 访问日志路径
        error_log  /var/log/nginx/web-error.log;			# 错误日志路径

        location ^~ /web/ {		# 前缀字符串匹配, 匹配任何以 /web/ 开头的请求，匹配成功以后，会停止								   # 搜索后面的正则表达式匹配
            root   html;
            index  index.html index.htm index.jsp;
                        # 健康检测,默认开启
                        proxy_next_upstream error timeout http_503 http_500 http_502 http_504;
                        proxy_pass  http://web1.com;	# 匹配upstream的上下文
                        proxy_redirect  default;
                        # 即允许重新定义或添加字段传递给代理服务器的请求头。
                        proxy_set_header Host  $http_host;
                        proxy_set_header Cookie $http_cookie;
                        
                        # 当字段不在请求头中就无法传递了，可通过设置Host变量，将需传递值赋给Host变量
                        proxy_set_header X-Real-IP $remote_addr;
                        
                        # 服务器名称通过代理服务器传递
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                        proxy_set_header X-Forwarded-Proto $scheme;
                        client_max_body_size  100m;
                        proxy_connect_timeout 10;

        }
        
        location ^~ /web {
            root   html;
            index  index.html index.htm index.jsp;
                        proxy_pass  http://web2.com;
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
```



### 5.3、启动和加载nginx配置

```bash
[root@nginx1 ~]# systemctl start nginx
[root@nginx1 ~]# nginx -t
[root@nginx1 ~]# nginx -s reload
```



### 5.4、nginx2服主配置文件

和5.1章节nginx1主配置文件一样



### 5.5、nginx2反代配置文件

```bash
[root@nginx2 ~]# cat /etc/nginx/conf.d/web.conf 
 upstream web1.com {
         server 192.168.230.202 weight=100;
         server 192.168.230.203 weight=1;
   }
 upstream web2.com {
         server 192.168.230.202 weight=1;
         server 192.168.230.203 weight=100;
   }
    
            server {
                listen 80;
        server_name web.com minikorean.ubtrobot.com;
        access_log  /var/log/nginx/web-access.log main;
        error_log  /var/log/nginx/web-error.log;

        location ^~ /web/ {
            root   html;
            index  index.html index.htm index.jsp;
                        # 健康检测,默认开启
                        proxy_next_upstream error timeout http_503 http_500 http_502 http_504;
                        proxy_pass  http://web1.com;
                        proxy_redirect  default;
                        proxy_set_header Host  $http_host;
                        proxy_set_header Cookie $http_cookie;
                        proxy_set_header X-Real-IP $remote_addr;
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                        proxy_set_header X-Forwarded-Proto $scheme;
                        client_max_body_size  100m;
                        proxy_connect_timeout 10;

        }
        
        location ^~ /web {
            root   html;
            index  index.html index.htm index.jsp;
                        proxy_pass  http://web2.com;
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
```



### 5.6、启动和加载nginx配置

过程同5.3章节



## 六、web服配置

2台web服配置一样

### 6.1、创建nginx主目录

```bash
# mkdir /web
```



### 6.2、生成web文件

web1:

```bash
#echo ““this is web1 webserver”” >/web/index.html
```



web2:

```bash
#echo ““this is web2 webserver”” >/web/index.html
```

### 6.3、nginx配置

web1:

```bash
[root@web01 ~]# egrep -v "#|^$" /etc/nginx/conf.d/web1.conf 
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```



web2:

```bash
[root@web02 ~]# egrep -v "#|^$" /etc/nginx/conf.d/web2.conf    
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```



### 6.4、启动加载nginx

```bash
#systemctl start nginx
#nginx -s reload
```



## 七、访问测试

### 7.1、访问nginx1上的web

因为nginx1配置的web2权重大，所以优先访问web2服

http://192.168.230.200/web/

![20200301_2249_001.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1gcetd6pijxj30od04fjs8.jpg)

### 7.2、访问nginx2上的web

因为nginx2配置的web1权重大，所以优先访问web1服

http://192.168.230.201/web/

![20200301_2251_001.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1gcetee5cqjj30e405f0th.jpg)

### 7.3、关闭web2服务器

http://192.168.230.200/web/

浏览器按F5刷新缓存，因为web2被关闭，所以指向web1

![20200301_2253_001.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1gcetginu7aj30ee03paao.jpg)

http://192.168.230.201/web/仍是访问web1不变



### 7.3、关闭web1服务器

http://192.168.230.200/web/

仍是访问web2



http://192.168.230.201/web/

浏览器按F5刷新缓存，因为web1被关闭，所以指向web2

![20200301_2256_001.jpg](http://ww1.sinaimg.cn/large/007Xg1efgy1gcetjfq2nkj30e0048js4.jpg)