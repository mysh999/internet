## 一、故障描述

一套nginx前端调用后台工程上传文件至OSS存储空间，在上传时间到60S时，报504错误

![企业微信截图_20201111113013.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gkl2q2kp7wj31ac0f341l.jpg)







## 二、解决办法

2.1、在nginx配置里增加超时配置时间，问题依旧

```bash
# cat nginx.conf
... ...
    keepalive_timeout 600;          #新增
    fastcgi_connect_timeout 6000;   #新增
    fastcgi_send_timeout 6000;      #新增
    fastcgi_read_timeout 6000;      #新增

    charset utf-8;
    server_names_hash_bucket_size 128;
    client_header_buffer_size 4k;
    large_client_header_buffers 8 64k;
    client_max_body_size 1g;
    client_body_buffer_size 10m;
    client_header_timeout 10m;
    client_body_timeout 10m;
    send_timeout 10m;  #新增

    proxy_connect_timeout 3000;   #新增
    proxy_read_timeout 3000;       #新增
    proxy_send_timeout 3000;       #新增
... ...



# cat conf/xxx.xxx.conf 

    server {
        listen       80;
        ... ...
        proxy_send_timeout 3000;       #新增
        proxy_read_timeout 3000;        #新增
        proxy_connect_timeout 3000;     #新增
        
        location ~* /xxx/ {
            proxy_next_upstream error timeout http_503 http_500 http_502 http_504;
            proxy_pass http://xxx;
            proxy_redirect off;
            proxy_store off;
            proxy_set_header Host $host;
            proxy_send_timeout 3000;      #新增
            proxy_read_timeout 3000;       #新增
            proxy_connect_timeout 3000;     #新增
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_ignore_client_abort on;
            proxy_intercept_errors on;
        }        

```



2.2、在SLB中查看对应日志

![企业微信截图_20201111113705.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gkl2xfji0sj31fg0jg0to.jpg)







![企业微信截图_20201111115345.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gkl3ekd4o1j31d90pljts.jpg)





2.3、在SLB上修改超时连接

![企业微信截图_20201111115520.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gkl3g7d3h8j31bd0fdq42.jpg)



超时连接设置成180s

![企业微信截图_20201111115612.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gkl3h6x0y9j30va0ptt9h.jpg)



客户端验证，已经不再报504报错