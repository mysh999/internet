说明：nginx从*1.9.0*开始，新增加了一个stream模块，用来实现四层协议的转发、代理或者负载均衡等



1、下载源码包

```bash
# wget http://nginx.org/download/nginx-1.21.0.tar.gz
```



2、安装依赖包

```bash
# yum -y install gcc gcc-c++ autoconf automake
# yum -y install zlib zlib-devel openssl openssl-devel pcre-devel
```



3、编译

```bash
# ./configure \
--prefix=/usr/local/nginx \
--user=nginx \
--group=nginx \
--sbin-path=/usr/local/nginx/sbin/nginx \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--with-pcre \
--with-http_ssl_module \
--with-http_realip_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-stream
```



4、安装

```bash
# make && make install
```



5、检查版本

```bash
# /usr/local/nginx/sbin/nginx -v
nginx version: nginx/1.21.0


# 查看加载模块
# nginx  -V
nginx version: nginx/1.21.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --user=nginx --group=nginx --sbin-path=/usr/local/nginx/sbin/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --with-pcre --with-http_ssl_module --with-http_realip_module --with-http_flv_module --with-http_mp4_module --with-http_gzip_static_module --with-http_stub_status_module --with-stream
```



6、设置开机启动

```bash
# ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx

# vi /etc/init.d/nginx
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemin
#
# chkconfig:  - 85 15
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#              proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /usr/local/nginx/conf/nginx.conf
# pidfile:    /run/nginx/nginx.pid
# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.

[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"

lockfile=/var/lock/nginx.lock
nginx_run_path="/var/run/nginx/"
nginx_temp_path="/var/tmp/nginx/"

start() {
    if [ ! -d $nginx_run_path ];then
        mkdir -p  $nginx_run_path
        chown -R nginx.nginx $nginx_run_path
    fi

     if [ ! -d $nginx_temp_path ];then
      mkdir -p ${nginx_temp_path}"{client,proxy,fastcgi,uwsgi,scgi}"
      chown -R nginx:nginx $nginx_temp_path
    fi

    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
        ;;
    *)
    echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
    exit 2
esac
```



赋予执行权限

```bash
# chmod a+x /etc/init.d/nginx

# chkconfig --add /etc/init.d/nginx 
# chkconfig nginx on
```



7、启动 nginx

```bash
# nginx
```



8、配置四层代理

```bash
# cat nginx.conf
worker_processes auto;
worker_rlimit_nofile 65535;
pid             /run/nginx.pid;
error_log       /var/log/nginx/error.log error;

include /usr/share/nginx/modules/*.conf;

events {
    accept_mutex on;  # 预防惊群现象
    multi_accept on;  # 打开同时接受多个新网络连接请求的功能
    use epoll;
    worker_connections 65535;
}
http {
    charset utf-8;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    server_tokens off;
    log_not_found off;
    types_hash_max_size 2048;
    
    server_names_hash_bucket_size 128;
    client_header_buffer_size 4k;
    large_client_header_buffers 8 64k;
    client_max_body_size 1g;
    client_body_buffer_size 1m;
    client_header_timeout 3m;
    client_body_timeout 3m;
    send_timeout 3m;

    proxy_connect_timeout 120;
    proxy_read_timeout 120;
    proxy_send_timeout 120;
    proxy_buffer_size       8k;
    proxy_buffers           8 8k;
    proxy_busy_buffers_size 16k;
    proxy_temp_file_write_size 128k;
    
    # gzip
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript text/x-component application/json application/javascript application/x-javascript application/xml application/xhtml+xml application/rss+xml application/atom+xml application/x-font-ttf application/vnd.ms-fontobject image/svg+xml image/x-icon font/opentype;

    # MIME
    include mime.types;
    types {
        # here are additional types
        application/javascript mjs;
    }
    default_type application/octet-stream;

    # logging
    log_format main '{"@timestamp":"$time_iso8601",'
        '"host":"$server_addr",'
        '"clientip":"$remote_addr",'
        '"waf":"$http_User_waf",'
        '"size":"$body_bytes_sent",'
        '"scheme":"$scheme",'
        '"request_time":"$request_time",'
        '"upstream_response_time":"$upstream_response_time",'
        '"upstream_addr":"$upstream_addr",'
        '"upstream_status": "$upstream_status",'
        '"http_host":"$host",'
        '"url":"$uri",'
        '"domain":"$host",'
        '"xff":"$http_x_forwarded_for",'
        '"referer":"$http_referer",'
        '"agent":"$http_user_agent",'
        '"status":"$status"}';
        
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log warn;
    underscores_in_headers on; 

    server {
        listen       80 default_server;
        server_name  _;
        return 403;
    }
    # load configs
    include conf.d/*.conf;
}
    include conf.d/*.stream;   # 新增4层配置加载
    
    
```



```bash
# 对ssh 服务进行4层转发
# cat ./conf.d/aaa.stream 
stream {

   upstream ssh {
        least_conn;
        server 10.10.17.22:22 weight=1;
        }

  server {
        # listen 16666 ssl proxy_protocol;
        listen 8888;
        proxy_pass ssh;
        proxy_buffer_size 3M;
        }

}
```





9、四层验证

```bash
# ssh root@127.0.0.1 -p 8888
# 可以登陆
```

